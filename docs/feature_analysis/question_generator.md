# Knowledge Reinforcement with Practice Exercise Generator

## 🎯 User Stories

### Learner - Custom Practice Mode
**As a** Learner,
**I want to** generate targeted practice exercises based on my current knowledge level and specific topics,
**So that** I can reinforce my understanding of weak areas and prepare for upcoming assessments.

**Acceptance Criteria:**
- The system accepts natural language requirements (e.g., "medium difficulty questions about calculus limits").
- The system retrieves relevant background knowledge from the course knowledge base.
- The system generates a configurable number of unique questions.
- Each question includes a detailed explanation and correct answer.
- The system validates that the generated questions are relevant to the requested topic.

### Learner - Exam Simulation Mode (Mimic Mode)
**As a** Learner,
**I want to** upload a past exam paper or reference questions,
**So that** I can practice with questions that mimic the exact style, format, and difficulty of the real exam.

**Acceptance Criteria:**
- The system can parse uploaded PDF exam papers (text, images, equations).
- The system identifies and extracts individual questions from the parsed content.
- The system generates new "mimic" questions for each reference question.
- The generated questions maintain the same core concept and difficulty but change the scenario/numbers.
- The system produces a comprehensive output containing the original and generated questions.

## 🏗️ Basic Design (High-Level)

### Purpose
The module provides an intelligent engine for creating educational content on-demand. It solves the problem of limited practice materials by leveraging Generative AI to produce infinite variations of high-quality questions, grounded in course materials (RAG) or modeled after specific reference exams.

### Architecture
The system employs a **Coordinator-Agent Pattern** combined with **Retrieval-Augmented Generation (RAG)**.
- **Coordinator**: Orchestrates the workflow, manages state, and delegates tasks to specialized agents.
- **Dual-Mode Operation**:
    1.  **Custom Mode**: Knowledge-driven generation using RAG to ground questions in course material.
    2.  **Mimic Mode**: Reference-driven generation using structural analysis of uploaded documents.
- **Parallel Execution**: Uses Python's `asyncio` to generate multiple questions concurrently for performance.

### Core Dependencies
- **OpenAI / LLM Service**: For reasoning, planning, and content generation (GPT-4o recommended).
- **MinerU**: For high-fidelity PDF parsing (handling formulas, tables, and layout).
- **RAG Service (LightRAG/RagAnything)**: For retrieving accurate background knowledge.
- **AsyncIO**: For managing concurrent generation tasks.

## 🔧 Detailed Design (Technical Deep-Dive)

### Logic Flow

#### 1. Custom Mode Workflow (Knowledge-Driven)
The `AgentCoordinator` (`src/agents/question/coordinator.py`) manages a streamlined "Plan-Execute" flow:

1.  **Researching (Retrieval)**:
    -   Input: User requirement string.
    -   Action: LLM generates search queries -> Parallel RAG retrieval -> Summarize Knowledge.
    -   Output: `background_knowledge.json`.
2.  **Planning**:
    -   Action: LLM analyzes knowledge & requirements -> Generates a `QuestionPlan`.
    -   Output: List of `focus` objects (e.g., "Test limit definition at x=0").
3.  **Generating (Parallel Execution)**:
    -   Action: For each `focus` in the plan:
        -   `QuestionGenerationAgent` uses the specific focus + background knowledge to write the question.
        -   `QuestionValidationWorkflow` performs a single-pass relevance check ("High" vs "Partial" relevance).
    -   Output: `result.json` and `question.md` for each question.

#### 2. Mimic Mode Workflow (Reference-Based)
The `mimic_exam_questions` tool (`src/agents/question/tools/exam_mimic.py`) handles this flow:

1.  **Ingestion**: Upload PDF -> Parse with MinerU -> Markdown/JSON.
2.  **Extraction**: LLM extracts distinct questions from the parsed text into a structured list.
3.  **Mimicry (Parallel Generation)**:
    -   Action: For each extracted question:
        -   Prompt LLM to: "Identify core concept", "Keep difficulty", "Change scenario/numbers".
        -   Generate a new question that maps 1:1 to the reference.
    -   Output: A batch JSON file mapping original -> generated questions.

### Data Structures

#### Question Object
The core entity exchanged between agents:
```json
{
  "question_type": "choice | written",
  "question": "The actual text of the question...",
  "options": { "A": "...", "B": "..." }, // Optional
  "correct_answer": "The answer key",
  "explanation": "Detailed step-by-step reasoning",
  "knowledge_point": "Topic tag"
}
```

#### Question Plan
The blueprint for a batch of questions:
```json
{
  "focuses": [
    {
      "id": "q_1",
      "focus": "Test continuity at a specific point",
      "type": "choice"
    },
    {
      "id": "q_2",
      "focus": "Calculate limit using L'Hopital's rule",
      "type": "written"
    }
  ]
}
```

#### Validation Result
The outcome of the quality check:
```json
{
  "decision": "approve | request_modification",
  "relevance": "highly_relevant | partially_relevant",
  "kb_coverage": "Explanation of what KB concepts are covered",
  "issues": ["List of any detected quality issues"]
}
```

### API / Interface
The `AgentCoordinator` exposes the primary entry points:
- `generate_questions_custom(requirement, count)`: Main API for knowledge-based practice.
- `generate_questions_parallel(requirement, count)`: (Legacy/Alternative) Parallel generation with shared context.
- `mimic_exam_questions(pdf_path, ...)`: Standalone tool function for the exam simulation workflow.

### Edge Case Handling
- **Missing Knowledge**: If RAG retrieves nothing, the system flags the error early ("knowledge_not_found") rather than hallucinating.
- **LLM Failure**: Each generation step has error handling; if one question in a batch fails, others continue (partial success).
- **Validation Rejection**: (Legacy) Previously used iterative refinement; now focuses on "Relevance Analysis" to accept valid but slightly "extended" questions rather than getting stuck in loops.

## 💡 Contextual Learning Note

### The "Plan-Execute" Pattern in LLM Workflows

One of the most powerful concepts in this code is the separation of **Planning** from **Execution** (found in `src/agents/question/coordinator.py`).

**How it works:**
Instead of asking the LLM to "Generate 3 questions about X" in one go, the system breaks it down:
1.  **Research**: First, it just gathers information (RAG). It doesn't write anything yet.
2.  **Plan**: It asks the LLM, "Given this info, what are 3 distinct *angles* or *focuses* we could test?" (e.g., "Concept A", "Application B", "Edge Case C").
3.  **Execute**: It then launches 3 separate, parallel tasks. Each task is simple: "Write ONE question about [Concept A]".

**Why this matters for Junior Developers:**
-   **Quality**: It prevents the LLM from getting "lazy" and generating 3 repetitive questions.
-   **Reliability**: Smaller, focused tasks (write 1 question) are less error-prone than large tasks (write 3 questions).
-   **Speed**: Once the plan is made, the execution steps can run at the same time (parallelism), making the app feel much faster!
