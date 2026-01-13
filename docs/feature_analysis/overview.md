# Knowledge Reinforcement with Practice Exercise Generator

## 🎯 Overview

The **Question Generator** is an intelligent engine for creating educational content on-demand. It solves the problem of limited practice materials by leveraging Generative AI to produce infinite variations of high-quality questions, grounded in course materials (RAG) or modeled after specific reference exams.

This documentation is split into the following detailed guides:

1.  **[Custom Practice Mode](./req_custom_practice.md)**: Knowledge-driven generation using RAG.
2.  **[Exam Mimic Mode](./req_exam_mimic.md)**: Reference-driven generation using structural analysis.

## 🏗️ Architecture

The system employs a **Coordinator-Agent Pattern** combined with **Retrieval-Augmented Generation (RAG)**.

-   **Agent Coordinator**: The central brain that orchestrates the workflow, manages state, and delegates tasks to specialized agents.
-   **Agents**:
    -   `QuestionGenerationAgent`: Specialized in writing educational content.
    -   `QuestionValidationWorkflow`: Ensures quality and relevance.
-   **Tools**:
    -   **RAG (Retrieval)**: Fetches background context from the knowledge base.
    -   **MinerU**: Parses PDF documents for the mimic mode.

### Tech Stack & Core Dependencies

| Component | Technology | Purpose |
| :--- | :--- | :--- |
| **LLM Service** | OpenAI API (GPT-4o) | Reasoning, planning, and content generation. |
| **Retrieval** | LightRAG / RagAnything | Retrieving accurate background knowledge. |
| **PDF Parsing** | MinerU | High-fidelity parsing (formulas, tables, layout). |
| **Concurrency** | Python `asyncio` | Managing parallel generation tasks for speed. |
| **Orchestration**| Custom Agent Framework | Managing the "Plan-Execute" lifecycle. |

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
