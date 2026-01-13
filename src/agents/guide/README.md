# Guided Learning Module

## Overview

The **Guided Learning** module transforms static notebook content into a dynamic, personalized learning curriculum. By leveraging multiple specialized agents, it breaks down complex records into progressive knowledge points, generates interactive visualizations, and supports context-aware Q&A.

For a detailed architectural deep-dive, including diagrams and data schemas, please refer to [Interactive Learning Architecture](../../docs/guide/interactive_learning_architecture.md).

## Core Features

1.  **Personalized Curriculum Planning**
    - **Agent:** `LocateAgent`
    - **Function:** Analyzes notebook records to identify 3-5 core concepts and organizes them into a logical learning path.

2.  **Interactive Visualizations**
    - **Agent:** `InteractiveAgent`
    - **Function:** Generates custom HTML/JS pages for each knowledge point, featuring mathematical rendering (Katex) and responsive design. Includes a **Fallback Mechanism** to ensure content is always displayable even if generation fails.

3.  **Context-Aware Q&A**
    - **Agent:** `ChatAgent`
    - **Function:** Provides targeted answers by "scoping" the chat history to the current knowledge point, preventing context bleeding between topics.

4.  **Learning Summary**
    - **Agent:** `SummaryAgent`
    - **Function:** Compiles a final review and mastery assessment upon session completion.

## Usage Flow

1.  **Create Session**: `create_session(notebook_id, records)`
    - Parses input and generates the curriculum.
2.  **Start Learning**: `start_learning(session_id)`
    - Generates the visualization for the first topic.
3.  **Interact**: `chat(session_id, message)`
    - Ask questions specific to the current topic.
4.  **Progress**: `next_knowledge(session_id)`
    - Moves to the next topic or generates the final summary.

## Directory Structure

```
guide/
├── guide_manager.py          # Central State Machine & Controller
├── agents/
│   ├── locate_agent.py       # Curriculum Planner
│   ├── interactive_agent.py  # Visual Generator
│   ├── chat_agent.py         # Q&A Handler
│   └── summary_agent.py      # Review Generator
└── prompts/                  # Agent Prompt Templates
```

## Data Persistence

Sessions are persisted as JSON files in `data/user/guide/`. Each session file acts as a self-contained database of the user's learning journey, including the curriculum, chat logs, and generated content.
