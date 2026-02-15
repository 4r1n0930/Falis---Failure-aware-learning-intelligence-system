# FALIS – Requirements Specification

## 1. Overview
### Purpose
FALIS (Failure-Aware Learning Intelligence System) is a confidence inspection system designed to verify whether a student is genuinely prepared for a topic after studying it. The system focuses on readiness validation rather than content delivery or score-based evaluation.

### Scope
FALIS supports students from:
- School education (various boards)
- College-level education
- Competitive exam preparation

The system operates in an offline-first mode with optional online intelligence enhancements.

---

## 2. Stakeholders
### Primary Users
- Students (school, college, competitive exams)

### Secondary Stakeholders
- Educators and mentors
- Hackathon evaluators
- System developers and maintainers

---

## 3. Core Functional Requirements

### 3.1 User Management
- The system shall allow users to create and manage profiles.
- The system shall capture academic context (level, board/exam, class/semester).
- The system shall maintain session continuity across multiple inspections.

---

### 3.2 Syllabus & Topic Management
- The system shall store syllabus data in a hierarchical structure.
- The system shall support multiple boards, universities, and exams.
- The system shall define topic depth boundaries per academic level.
- The system shall model topic dependencies and prerequisites.

---

### 3.3 Topic Inspection Initiation
- The system shall allow users to select a topic they have studied.
- The system shall resolve the selected topic to its subtopics.
- The system shall identify high-importance and high-failure subtopics.

---

### 3.4 Diagnostic Questioning
- The system shall generate inspection questions aligned with syllabus topics.
- The system shall ensure questions cover critical conceptual areas.
- The system shall evaluate answers deterministically using answer keys or rules.
- The system shall not rely on probabilistic AI for correctness evaluation.

---

### 3.5 Adaptive Retry Mechanism
- The system shall trigger adaptive retries upon incorrect responses.
- The system shall generate concept-consistent question variations.
- The system shall limit retries based on defined stability thresholds.
- The system shall distinguish accidental mistakes from conceptual gaps.

---

### 3.6 Topic Status Marking
- The system shall mark topics as:
  - Green (stable understanding)
  - Red (unstable understanding)
- The system shall base topic status on response stability, not scores.
- The system shall persist topic status across sessions.

---

### 3.7 Revision Session Generation
- The system shall create revision sessions for unstable topics.
- The system shall include previously failed questions and variations.
- The system shall allow users to revisit revision sessions on demand.

---

## 4. Data Requirements

### 4.1 Data Acquisition
- Official syllabi and curriculum documents
- Verified question banks and answer keys
- Student interaction and response data
- Behavioral signals (time taken, retries, consistency)

---

### 4.2 Data Storage
- Structured storage for users, topics, attempts, and states
- Document storage for questions and explanations
- Vector storage for semantic representations

---

### 4.3 Data Processing
- Topic hierarchy normalization
- Question-to-concept mapping
- Stability and variance analysis
- Failure pattern aggregation (optional, online)

---

## 5. Intelligence & AI Requirements

### 5.1 AI Usage Constraints
- AI shall assist only in question variation and explanation generation.
- AI shall not determine answer correctness.
- AI outputs shall be constrained by syllabus context.

---

### 5.2 Vector-Based Intelligence
- The system shall store embeddings for concepts and questions.
- The system shall support similarity search for controlled variation.
- The system shall avoid repetitive or trivial question generation.

---

## 6. Non-Functional Requirements

### 6.1 Reliability
- The system shall produce consistent results for identical inputs.
- Topic marking decisions shall be explainable and auditable.

---

### 6.2 Offline Capability
- The system shall support offline inspection for core functionality.
- Online connectivity shall enhance, not block, system operation.

---

### 6.3 Performance
- Inspection flow shall respond within acceptable latency.
- Question generation and evaluation shall not disrupt user focus.

---

### 6.4 Security & Privacy
- The system shall collect minimal personal data.
- Student data shall not be used for external profiling.
- Aggregated data shall not expose individual identities.

---

## 7. Constraints
- The system shall remain syllabus-bounded at all times.
- The system shall avoid over-dependence on internet connectivity.
- The system shall prioritize explainability over complexity.

---

## 8. Assumptions
- Students have already studied the topic before inspection.
- Syllabus and question data are curated and verified.
- Confidence verification is more valuable than score optimization.

---

## 9. Success Criteria
- Students can clearly identify weak topics before exams.
- False confidence is reduced through inspection.
- Users report increased clarity and exam readiness.
- The system maintains trust through honest feedback.

---

## 10. Out of Scope
- Full content teaching or lecture delivery
- Gamification or entertainment features
- Ranking, leaderboard, or competitive scoring systems

---

## 11. Guiding Principle
> FALIS exists to tell students the truth about their preparation—before the exam does.
