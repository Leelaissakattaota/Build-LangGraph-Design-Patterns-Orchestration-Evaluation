# 🦜🔗 Advanced Agentic Design Patterns with LangGraph

![LangChain](https://img.shields.io/badge/Framework-LangChain-blue?style=for-the-badge&logo=langchain)
![LangGraph](https://img.shields.io/badge/Library-LangGraph-orange?style=for-the-badge)
![Python](https://img.shields.io/badge/Python-3.9+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![OpenAI](https://img.shields.io/badge/Model-GPT--4o--Mini-412991?style=for-the-badge&logo=openai)

This repository demonstrates the implementation of advanced **Agentic Design Patterns** using **LangGraph**. By moving beyond linear LLM chains, these patterns allow for sophisticated, stateful, and iterative AI workflows including parallel execution, intent-based routing, and self-reflection loops.

---

## 🏗️ Implemented Design Patterns

### 🔄 1. Orchestrator-Worker Pattern
*Designed for complex, dynamic task decomposition.*
- **Concept:** A lead "Orchestrator" agent analyzes a request and breaks it into multiple sub-tasks.
- **Parallel Execution:** Specialized "Worker" agents process sub-tasks simultaneously (fan-out).
- **Synthesis:** Results are aggregated into a final, comprehensive output.
- **Use Case:** Automated meal planning with specialized cuisine chefs.

### 🔁 2. Reflection & Evaluation Pattern
*Designed for iterative quality improvement.*
- **Generator:** Creates an initial proposal (e.g., an investment plan).
- **Evaluator:** Critically reviews the plan based on specific personas (e.g., Warren Buffett style).
- **Refinement:** The system loops back to the generator with feedback until the target criteria are met.
- **Use Case:** Iterative Investment Advisor.

### 🚦 3. Intent-Based Routing
*Designed for specialized task handling.*
- **Router:** Classifies user intent (e.g., translation vs. summarization).
- **Logic:** Directs the workflow to the most appropriate specialized node.
- **Use Case:** Multi-skill AI Assistant.

### 🔗 4. Prompt Chaining
- **Logic:** Sequential execution where the output of one LLM call is the input for the next.
- **Use Case:** Job Application Assistant (Resume Summary ➡️ Cover Letter).

---

## 🛠️ Tech Stack & Concepts

| Feature | Description |
| :--- | :--- |
| **State Management** | Using `TypedDict` and `Annotated` to manage global workflow context. |
| **Cyclic Graphs** | Implementing loops for self-correction and reflection. |
| **Structured Output** | Utilizing `Pydantic` models and `.with_structured_output()` for reliable parsing. |
| **Parallelism** | Using the `Send` API to trigger concurrent worker executions. |
| **Visualization** | Workflow graphs rendered using `Mermaid.js`. |

---

## 🚀 Getting Started

### Prerequisites
- Python 3.9+
- OpenAI API Key

### Installation
```bash
pip install langchain-openai langgraph pygraphviz pydantic
