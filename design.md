# FALIS – Design Specification

## 1. System Architecture

### 1.1 Architecture Overview
FALIS follows a modular, offline-first architecture with three primary layers:

- **Presentation Layer**: User interface for topic selection, inspection sessions, and results
- **Application Layer**: Core business logic for inspection orchestration, adaptive questioning, and status determination
- **Data Layer**: Persistent storage for users, topics, questions, attempts, and embeddings

### 1.2 Deployment Model
- **Primary Mode**: Standalone offline application with local database
- **Enhanced Mode**: Optional online connectivity for AI-powered question variation and aggregated intelligence

---

## 2. Data Model

### 2.1 Core Entities

#### User
```
User {
  id: UUID
  name: String
  academic_level: Enum (school, college, competitive)
  board_or_exam: String
  class_or_semester: String
  created_at: Timestamp
  last_active: Timestamp
}
```

#### Topic
```
Topic {
  id: UUID
  name: String
  parent_topic_id: UUID? (nullable for root topics)
  syllabus_id: UUID
  depth_level: Integer
  importance_score: Float (0.0 to 1.0)
  prerequisites: List<UUID> (topic dependencies)
}
```

#### Subtopic
```
Subtopic {
  id: UUID
  topic_id: UUID
  name: String
  concept_description: Text
  importance_score: Float
  failure_rate: Float (aggregated, optional)
}
```

#### Question
```
Question {
  id: UUID
  subtopic_id: UUID
  question_text: Text
  question_type: Enum (mcq, numerical, short_answer)
  difficulty: Enum (basic, intermediate, advanced)
  answer_key: JSON (structured answer data)
  evaluation_rules: JSON (deterministic rules)
  embedding: Vector (for similarity search)
}
```

#### InspectionSession
```
InspectionSession {
  id: UUID
  user_id: UUID
  topic_id: UUID
  started_at: Timestamp
  completed_at: Timestamp?
  status: Enum (in_progress, completed, abandoned)
}
```

#### Attempt
```
Attempt {
  id: UUID
  session_id: UUID
  question_id: UUID
  subtopic_id: UUID
  user_answer: JSON
  is_correct: Boolean
  time_taken_seconds: Integer
  retry_count: Integer
  attempt_number: Integer (1st, 2nd, 3rd attempt)
  timestamp: Timestamp
}
```

#### TopicStatus
```
TopicStatus {
  id: UUID
  user_id: UUID
  topic_id: UUID
  subtopic_id: UUID
  status: Enum (green, red, not_inspected)
  stability_score: Float (0.0 to 1.0)
  last_inspected: Timestamp
  total_attempts: Integer
  correct_attempts: Integer
}
```

### 2.2 Relationships
- User → InspectionSession (1:N)
- Topic → Subtopic (1:N)
- Topic → Topic (parent-child hierarchy)
- Subtopic → Question (1:N)
- InspectionSession → Attempt (1:N)
- User + Topic + Subtopic → TopicStatus (unique constraint)

---

## 3. Core Workflows

### 3.1 Topic Inspection Flow

```
1. User selects topic for inspection
2. System resolves topic → subtopics
3. System identifies high-importance/high-failure subtopics
4. System creates InspectionSession
5. For each selected subtopic:
   a. Generate/select question
   b. Present question to user
   c. Evaluate answer deterministically
   d. If incorrect: trigger adaptive retry
   e. Record attempt
6. Calculate stability scores per subtopic
7. Mark subtopic status (green/red)
8. Persist TopicStatus
9. Generate revision session if needed
10. Present results to user
```

### 3.2 Adaptive Retry Mechanism

```
When user answers incorrectly:
1. Check retry_count for this subtopic in current session
2. If retry_count < MAX_RETRIES (e.g., 2):
   a. Generate concept-consistent question variation
   b. Present new question
   c. Increment retry_count
   d. Record new attempt
3. If retry_count >= MAX_RETRIES:
   a. Mark subtopic as unstable (red)
   b. Move to next subtopic
```

### 3.3 Stability Calculation

```
For each subtopic after inspection:
1. Collect all attempts for this subtopic in session
2. Calculate metrics:
   - first_attempt_correct: Boolean
   - retry_success_rate: correct_retries / total_retries
   - consistency: variance in response quality
3. Apply stability rules:
   - Green: first_attempt_correct AND (no retries OR retry_success_rate >= 0.8)
   - Red: NOT first_attempt_correct OR retry_success_rate < 0.8
4. Calculate stability_score: weighted combination of metrics
5. Persist TopicStatus
```

---

## 4. Component Design

### 4.1 Inspection Engine
**Responsibility**: Orchestrate inspection sessions and manage question flow

**Key Methods**:
- `start_inspection(user_id, topic_id) → InspectionSession`
- `get_next_question(session_id) → Question`
- `submit_answer(session_id, question_id, answer) → EvaluationResult`
- `complete_inspection(session_id) → InspectionReport`

### 4.2 Question Generator
**Responsibility**: Select or generate questions for subtopics

**Key Methods**:
- `select_question(subtopic_id, difficulty, exclude_ids) → Question`
- `generate_variation(question_id, subtopic_id) → Question` (AI-assisted)
- `validate_question_quality(question) → Boolean`

### 4.3 Answer Evaluator
**Responsibility**: Deterministically evaluate user answers

**Key Methods**:
- `evaluate_mcq(user_answer, answer_key) → Boolean`
- `evaluate_numerical(user_answer, answer_key, tolerance) → Boolean`
- `evaluate_short_answer(user_answer, evaluation_rules) → Boolean`

### 4.4 Stability Analyzer
**Responsibility**: Calculate stability scores and determine topic status

**Key Methods**:
- `calculate_stability(attempts) → Float`
- `determine_status(stability_score, attempts) → Enum(green, red)`
- `analyze_failure_patterns(attempts) → FailureInsights`

### 4.5 Revision Session Manager
**Responsibility**: Create and manage revision sessions for unstable topics

**Key Methods**:
- `create_revision_session(user_id, red_topics) → RevisionSession`
- `get_revision_questions(user_id) → List<Question>`
- `track_revision_progress(user_id, topic_id) → Progress`

### 4.6 Syllabus Manager
**Responsibility**: Manage topic hierarchies and syllabus data

**Key Methods**:
- `get_topic_hierarchy(topic_id) → TopicTree`
- `resolve_subtopics(topic_id) → List<Subtopic>`
- `get_prerequisites(topic_id) → List<Topic>`

---

## 5. AI Integration Design

### 5.1 Question Variation Generation
**Approach**: Use LLM with strict constraints

**Input**:
- Original question
- Subtopic context
- Syllabus boundaries
- Difficulty level

**Process**:
1. Embed original question
2. Query vector store for similar questions
3. If online: Use LLM to generate variation with constraints
4. If offline: Select from pre-generated variations
5. Validate variation maintains concept consistency

**Output**: New question with same conceptual focus

### 5.2 Embedding Strategy
**Purpose**: Enable semantic similarity search for questions and concepts

**Implementation**:
- Use sentence transformers for question embeddings
- Store embeddings in vector database (e.g., FAISS, ChromaDB)
- Index by subtopic_id for efficient retrieval
- Support offline embedding generation during data ingestion

---

## 6. Algorithms

### 6.1 Subtopic Prioritization Algorithm

```python
def prioritize_subtopics(topic_id, user_id):
    subtopics = get_subtopics(topic_id)
    
    for subtopic in subtopics:
        # Calculate priority score
        importance = subtopic.importance_score
        failure_rate = get_failure_rate(subtopic.id)  # aggregated data
        user_history = get_user_history(user_id, subtopic.id)
        
        priority_score = (
            importance * 0.5 +
            failure_rate * 0.3 +
            (1 - user_history.past_success_rate) * 0.2
        )
        
        subtopic.priority = priority_score
    
    return sorted(subtopics, key=lambda x: x.priority, reverse=True)
```

### 6.2 Stability Score Calculation

```python
def calculate_stability_score(attempts):
    if not attempts:
        return 0.0
    
    first_correct = 1.0 if attempts[0].is_correct else 0.0
    
    if len(attempts) == 1:
        return first_correct
    
    retry_attempts = attempts[1:]
    retry_correct_count = sum(1 for a in retry_attempts if a.is_correct)
    retry_success_rate = retry_correct_count / len(retry_attempts)
    
    # Penalize retries
    retry_penalty = 0.2 * len(retry_attempts)
    
    stability = (
        first_correct * 0.6 +
        retry_success_rate * 0.4 -
        retry_penalty
    )
    
    return max(0.0, min(1.0, stability))
```

### 6.3 Topic Status Determination

```python
def determine_topic_status(stability_score, attempts):
    GREEN_THRESHOLD = 0.7
    RED_THRESHOLD = 0.5
    
    first_attempt_correct = attempts[0].is_correct
    
    if first_attempt_correct and stability_score >= GREEN_THRESHOLD:
        return "green"
    elif stability_score < RED_THRESHOLD:
        return "red"
    else:
        # Borderline case: check consistency
        if len(attempts) > 1:
            consistency = calculate_consistency(attempts)
            return "green" if consistency > 0.8 else "red"
        return "red"

def calculate_consistency(attempts):
    if len(attempts) < 2:
        return 1.0
    
    correct_count = sum(1 for a in attempts if a.is_correct)
    return correct_count / len(attempts)
```

---

## 7. User Interface Design

### 7.1 Key Screens

#### Topic Selection Screen
- Display topic hierarchy
- Show previously inspected topics with status indicators (green/red)
- Allow topic search and filtering

#### Inspection Session Screen
- Display current question
- Show progress (e.g., "Question 3 of 10")
- Provide answer input (MCQ options, text input, numerical input)
- Show timer (optional, for behavioral data)

#### Results Screen
- Display subtopic-wise status (green/red)
- Show stability scores
- Highlight weak areas
- Provide option to start revision session

#### Revision Session Screen
- Similar to inspection session
- Emphasize previously failed questions
- Track improvement over time

### 7.2 Status Indicators
- **Green**: ✓ Stable understanding
- **Red**: ✗ Unstable understanding
- **Gray**: Not yet inspected

---

## 8. Offline-First Strategy

### 8.1 Local Data Storage
- Use SQLite or similar embedded database
- Store all core data locally
- Sync optional aggregated data when online

### 8.2 Question Bank Management
- Pre-load question banks during installation
- Support incremental updates when online
- Maintain version control for question data

### 8.3 AI Fallback
- Pre-generate question variations offline
- Use rule-based variation when AI unavailable
- Queue AI requests for later processing when offline

---

## 9. Security & Privacy

### 9.1 Data Minimization
- Collect only essential user data
- Avoid storing sensitive personal information
- Use local-only storage by default

### 9.2 Data Anonymization
- Aggregate failure patterns without user identifiers
- Use differential privacy for shared statistics
- Allow users to opt out of data sharing

---

## 10. Testing Strategy

### 10.1 Unit Tests
- Test answer evaluation logic
- Test stability calculation algorithms
- Test question selection logic

### 10.2 Integration Tests
- Test complete inspection flow
- Test adaptive retry mechanism
- Test revision session generation

### 10.3 Property-Based Tests
- Test stability score properties (monotonicity, bounds)
- Test question variation consistency
- Test topic status determination correctness

---

## 11. Performance Considerations

### 11.1 Response Time Targets
- Question loading: < 500ms
- Answer evaluation: < 200ms
- Session completion: < 1s

### 11.2 Scalability
- Support 10,000+ questions per syllabus
- Handle 1,000+ topics in hierarchy
- Maintain performance with years of user history

---

## 12. Future Enhancements

### 12.1 Phase 2 Features
- Spaced repetition scheduling
- Collaborative learning insights
- Mentor dashboard for tracking student progress

### 12.2 Advanced AI Features
- Explanation generation for incorrect answers
- Personalized question difficulty adjustment
- Predictive readiness scoring

---

## 13. Correctness Properties

### 13.1 Inspection Properties
**Property 1.1**: Every inspection session must evaluate at least one question per selected subtopic
**Property 1.2**: Answer evaluation must be deterministic (same answer → same result)
**Property 1.3**: Retry count must never exceed MAX_RETRIES per subtopic per session

### 13.2 Stability Properties
**Property 2.1**: Stability score must be in range [0.0, 1.0]
**Property 2.2**: First attempt correct with no retries → stability score >= 0.6
**Property 2.3**: All attempts incorrect → stability score < 0.5

### 13.3 Status Properties
**Property 3.1**: Topic status must be one of {green, red, not_inspected}
**Property 3.2**: Green status requires first_attempt_correct = true
**Property 3.3**: Status transitions must be logged with timestamps

### 13.4 Data Integrity Properties
**Property 4.1**: Every attempt must reference a valid question and session
**Property 4.2**: Session completion timestamp must be >= session start timestamp
**Property 4.3**: User cannot have multiple active sessions for same topic simultaneously

---

## 14. Implementation Notes

### 14.1 Technology Stack Recommendations
- **Backend**: Python (FastAPI) 
- **Database**: PostgreSQL (optional online)
- **Vector Store**: ChromaDB
- **AI**: OpenAI API or Gemini Api
- **Frontend**: React or next.js + tailwind css

### 14.2 Development Phases
1. **Phase 1**: Core inspection engine + local storage
2. **Phase 2**: AI-powered question variation
3. **Phase 3**: Revision session management
4. **Phase 4**: Online intelligence and aggregation

---

## 15. Success Metrics

### 15.1 System Metrics
- Inspection completion rate > 80%
- Average session duration: 15-30 minutes
- Question evaluation accuracy: 100% (deterministic)

### 15.2 User Metrics
- User-reported confidence alignment with inspection results
- Reduction in false confidence incidents
- Exam performance correlation with inspection outcomes

---

## 16. Design Principles

1. **Truth over comfort**: Always provide honest feedback
2. **Determinism over guesswork**: Use rule-based evaluation
3. **Offline-first**: Never block core functionality on connectivity
4. **Simplicity over complexity**: Prefer explainable algorithms
5. **Privacy by design**: Minimize data collection and exposure

