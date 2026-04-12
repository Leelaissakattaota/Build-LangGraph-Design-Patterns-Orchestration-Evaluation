# 🧠 LangGraph Design Patterns: Orchestration & Reflection

![Language](https://img.shields.io/badge/Language-Python%203.11-3776AB?style=flat-square&logo=python&logoColor=white)
![Framework](https://img.shields.io/badge/Framework-LangGraph%200.6.6-FF6B35?style=flat-square)
![LLM](https://img.shields.io/badge/LLM-GPT--4o--mini-412991?style=flat-square&logo=openai&logoColor=white)
![Pattern](https://img.shields.io/badge/Pattern-Orchestrator--Worker-0052CC?style=flat-square)
![Pattern](https://img.shields.io/badge/Pattern-Reflection%20Loop-2E7D32?style=flat-square)
![Validation](https://img.shields.io/badge/Validation-Pydantic-6A0DAD?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=flat-square)

---

## 📌 Project Overview

This project demonstrates **two fundamental agentic design patterns** that 
power modern AI workflows — implemented using **LangGraph** and **GPT-4o-mini**.

These patterns are used in production systems at **Anthropic, OpenAI, 
Microsoft, IBM, and Google** — enabling AI agents to collaborate, 
self-improve, and tackle complex multi-step reasoning tasks.

**Pattern 1 — Orchestrator-Worker:** Parallel meal planning with 
specialized chef agents

**Pattern 2 — Reflection Loop:** Iterative investment advisory system 
with 3 AI investor personas (Cathie Wood → Ray Dalio → Warren Buffett)

**Domain:** Agentic AI Design Patterns  
**Language:** Python 3.11  
**LLM:** OpenAI GPT-4o-mini  
**Framework:** LangGraph 0.6.6  

---

## 📂 Project Structure

```
Build-LangGraph-Design-Patterns/
│
├── Build LangGraph Design Patterns Orchestration Evaluation.ipynb
└── README.md
```

---

## 🛠️ Tech Stack

| Component | Technology |
|---|---|
| Workflow Framework | LangGraph 0.6.6 |
| LLM | OpenAI GPT-4o-mini |
| LangChain | langchain-openai 0.3.27 |
| Graph Visualization | pygraphviz 1.14 + Mermaid |
| Data Validation | Pydantic BaseModel |
| Prompts | ChatPromptTemplate (LCEL) |
| Async | Send() API for parallel fan-out |

---

## 🏭 Pattern 1 — Orchestrator-Worker (Parallel Processing)

### Use Case: Multi-Chef Meal Planning System

```
User: "Steak and eggs, tacos, and chili"
           │
           ▼
┌──────────────────────────────────────┐
│         Orchestrator Node            │
│  GPT-4o-mini → Pydantic Dishes       │
│  Breaks meal list into Dish objects  │
└──────────┬───────────────────────────┘
           │ Send() API — Fan-out
    ┌──────┼──────┬──────┐
    ▼      ▼      ▼      ▼
 Chef1  Chef2  Chef3  Chef4   (Parallel Workers)
 Steak  Tacos  Chili  Eggs
    └──────┴──────┴──────┘
           │ operator.add merges outputs
           ▼
┌──────────────────────────────────────┐
│         Synthesizer Node             │
│  Aggregates all chef outputs into    │
│  final_meal_guide                    │
└──────────────────────────────────────┘
```

### Key Components

**Pydantic Structured Output:**
```python
class Dish(BaseModel):
    name: str          # "Spaghetti Bolognese"
    ingredients: List[str]  # ["pasta", "beef", "tomato"]
    location: str      # "Italian"

class Dishes(BaseModel):
    sections: List[Dish]

# LCEL pipeline with structured output
planner_pipe = dish_prompt | llm.with_structured_output(Dishes)
```

**State with Parallel Aggregation:**
```python
class State(TypedDict):
    meals: str
    sections: List[Dish]
    completed_menu: Annotated[List[str], operator.add]  # Auto-merges parallel outputs
    final_meal_guide: str
```

**Fan-out via Send() API:**
```python
def assign_workers(state: State):
    return [Send("chef_worker", {"section": s}) for s in state["sections"]]
```

**Graph Construction:**
```python
builder.add_node("orchestrator", orchestrator)
builder.add_node("chef_worker", chef_worker)
builder.add_node("synthesizer", synthesizer)
builder.add_edge(START, "orchestrator")
builder.add_conditional_edges("orchestrator", assign_workers, ["chef_worker"])
builder.add_edge("chef_worker", "synthesizer")
builder.add_edge("synthesizer", END)
```

---

## 🔄 Pattern 2 — Reflection Loop (Iterative Refinement)

### Use Case: Investment Advisory with 3 AI Personas

```
Investor Profile (Age, Salary, Risk Tolerance)
           │
           ▼
┌──────────────────────────────────┐
│    determine_target_grade        │
│    GPT-4o-mini → target risk     │
│    e.g., "aggressive"            │
└──────────┬───────────────────────┘
           │
           ▼
┌──────────────────────────────────┐    ← Loop starts
│  investment_plan_generator       │
│                                  │
│  No feedback → CATHIE WOOD style │
│  Has feedback → RAY DALIO style  │
└──────────┬───────────────────────┘
           │
           ▼
┌──────────────────────────────────┐
│       evaluate_plan              │
│  WARREN BUFFETT style evaluator  │
│  → grade (conservative/moderate) │
│  → feedback (risk analysis)      │
│  → n++ (iteration count)         │
└──────────┬───────────────────────┘
           │
           ▼
┌──────────────────────────────────┐
│      route_investment            │
│                                  │
│  grade == target? → END ✅       │
│  n > 5 iterations? → END ✅      │
│  grade ≠ target? → back to gen   │
└──────────────────────────────────┘
```

### 3 AI Investor Personas

| Persona | Style | When Used |
|---|---|---|
| **Cathie Wood** | Bold, innovation-driven, high-growth | Initial plan generation |
| **Ray Dalio** | Conservative, macro-aware, adaptive | Refinement after feedback |
| **Warren Buffett** | Value investing, capital preservation | Evaluation & grading |

### Reflection State
```python
class State(TypedDict):
    investment_plan: str      # Generated strategy
    investor_profile: str     # User's age, salary, goals
    target_grade: grades      # Ideal risk level
    feedback: str             # Buffett's critique
    grade: grades             # Assigned risk grade
    n: int                    # Iteration counter
```

### Pydantic Feedback Schema
```python
grades = Literal["ultra-conservative", "conservative", "moderate", "aggressive", "high risk"]

class Feedback(BaseModel):
    grade: grades    # Risk classification
    feedback: str    # Reasoning explanation
```

### Conditional Routing
```python
builder.add_conditional_edges(
    "evaluate_plan",
    lambda state: route_investment(state),
    {
        "Accepted": END,
        "Rejected + Feedback": "investment_plan_generator",
    }
)
```

---

## 🔑 LangGraph Concepts Mastered

| Concept | Implementation |
|---|---|
| `StateGraph` | Shared state schema flowing through all nodes |
| `Annotated[List, operator.add]` | Auto-merging parallel worker outputs |
| `Send()` API | Dynamic fan-out to variable number of workers |
| `add_conditional_edges()` | Dynamic routing based on state values |
| `with_structured_output()` | Pydantic-enforced LLM responses |
| `ChatPromptTemplate` | LCEL prompt + LLM chaining |
| Mermaid Visualization | `draw_mermaid_png()` graph inspection |
| Iteration Control | `n` counter + max iterations check |
| Literal Types | Constrained state values |

---

## 🎓 Skills Demonstrated

- LangGraph Orchestrator-Worker pattern — parallel task execution
- LangGraph Reflection pattern — iterative generate-evaluate-refine loop
- Dynamic fan-out with `Send()` API for variable worker counts
- `Annotated` state fields with `operator.add` for parallel merging
- Pydantic structured output — `Dishes`, `Feedback`, `Recipe` schemas
- LCEL chains — `prompt | llm.with_structured_output(Schema)`
- Conditional routing with `add_conditional_edges()`
- Dual-persona generator — Cathie Wood (bold) → Ray Dalio (adaptive)
- Warren Buffett evaluator — value-investing risk assessment
- `TypedDict` state management across multi-node workflows
- LangGraph graph visualization with Mermaid + pygraphviz
- Loop control with iteration counters and exit conditions
- Real-world use cases: Meal Planning + Investment Advisory

---

## 📜 Certifications

| Certification | Issuer | Platform |
|---|---|---|
| IBM Data Science Professional Certificate | IBM | Coursera |
| IBM Generative AI Professional Certificate | IBM | Coursera |
| IBM Agentic AI with RAG Certificate | IBM | Coursera |
| IBM RAG and Agentic AI Professional Certificate | IBM | Coursera |

---

## 🤝 Connect with Me

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Leela%20A-0077B5?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/leela-a)
[![Gmail](https://img.shields.io/badge/Gmail-attotaleelaissak@gmail.com-D14836?style=flat-square&logo=gmail&logoColor=white)](mailto:attotaleelaissak@gmail.com)
[![GitHub](https://img.shields.io/badge/GitHub-Leelaissakattaota-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/Leelaissakattaota)
