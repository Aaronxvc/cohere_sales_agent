# Secure Sales Insights Agent — Cohere Prompt Specialist Take-Home

## 1. Overview

### 1.1 Purpose

This project implements a **Secure Sales Insights Agent** designed for Cohere’s async technical evaluation. The agent answers sales related questions using structured subscription data while enforcing strong safety rules and refusing PII requests.

### 1.2 What’s Included

- `agent.py` — Main LLM agent  
- `evaluate.py` — Evaluation pipeline  
- `subscription_data.csv` — Provided dataset  
- `eval_results.json` — Saved evaluation output  
- `README.md` — Documentation  

---

## 2. Problem Statement

### 2.1 In My Own Words

Cohere needs flexible, reliable domain specific agents that behave safely under ambiguity.  
The assignment: build an AI assistant that:

- Accepts natural language questions  
- Uses subscription + revenue data  
- Enforces PII and export safety  
- Returns helpful insights using aggregates  
- Includes its own evaluation methodology  

My interpretation:  
**A safe internal sales analytics assistant** that can summarize trends and revenue without ever leaking customer level data.

---

## 3. Agent Design

### 3.1 Architecture Overview

The `run_agent()` function contains the full agent flow:

1. **PII Classification**  
   Detects:
   - emails  
   - "export all", "full dataset", "every customer"  
   - credit card / phone tokens  
   - any presence of "@"

   Immediate refusal if unsafe.

2. **Context Builder**  
   Constructs a safe, aggregate only context block with:
   - Total active MRR  
   - Enterprise customer count  
   - Professional customer count  

   **No raw rows** are passed to the LLM.

3. **Cohere Client V2 (Chat API)**  
   Uses:
   ```
   model="command-a-reasoning-08-2025"
   ```
   If the model is not available (trial restrictions), the agent returns a **graceful fallback** rather than crashing — critical for production robustness.

4. **Structured Return Object**
   ```json
   {
     "answer": "...",
     "decision": "answer/refuse",
     "reasoning_note": "..."
   }
   ```

### 3.2 Safety-First Prompting

The system prompt enforces:

- Use only provided aggregates  
- No hallucinated numbers  
- No sensitive data  
- Make assumptions explicit  
- Refuse unsafe or ambiguous export requests  

This ensures stable, predictable outputs.

---

## 4. Evaluation Design

### 4.1 Metrics

Evaluation includes three dimensions:

1. **Accuracy**  
   Checks for expected numeric substrings.

2. **Safety & Refusal Correctness**  
   Ensures:
   - `decision == "refuse"` when required  
   - No forbidden leakage (e.g., "@", domain names)

3. **Reasoning & Clarity**  
   Looks for reasoning keywords ("assumption", "interpret", etc.).

### 4.2 Test Cases

A minimal but representative test suite:

- **T1**: Active MRR numeric correctness  
- **T2**: PII request (single email)  
- **T3**: Bulk export refusal  
- **T4**: Ambiguous reasoning test ("might not renew")  

### 4.3 Evaluation Output

`evaluate.py` prints:

- Per-test behavior  
- Per-metric scores  
- Summary  
- Full results saved to `eval_results.json`

---

## 5. Results

### 5.1 Accuracy
`1.0`  
- Agent computed total active MRR = **127,100** correctly.

### 5.2 Safety & PII Compliance
`1.0`  
- Perfect refusal behavior  
- No leakage  
- Decision field correct

### 5.3 Reasoning & Clarity
`0.0`  
- Reasoning model unavailable on trial key  
- Fallback path executed cleanly  
- No crash → robustness demonstrated  

---

## 6. Limitations

### 6.1 Model Access Limitations
- Trial Cohere keys may not have access to reasoning capable models such as  
  `command-a-reasoning-08-2025`.  
- The agent handles this gracefully — but cannot demonstrate full chain of thought reasoning with the restricted key.

### 6.2 Aggregate-Only Answers
- By design, the agent does not inspect raw customer rows.  
- Limits granularity but ensures safety and deterministic behavior.

### 6.3 Simplistic Metric Heuristics
- Accuracy checks are substring-based.  
- Safety checks rely on keyword rules.  
- Reasoning detection uses keyword heuristics.

This is intentional to keep the evaluation small, clear, and readable.

---

## 7. Future Work

### 7.1 Enhanced Reasoning Metrics
- Use Cohere models capable of structured reasoning.  
- Score reasoning using LLM as judge.

### 7.2 More Advanced Safety Layer
- Add pattern based PII classification  
- Add role based access checks  
- Add configurable safety policies via YAML

### 7.3 Expand Business Insight Capabilities
- Predict churn risk  
- Run cohort analysis  
- Summarize changes week over week  
- Detect anomalies in MRR or seat usage

---

## 8. How to Run

### 8.1 Setup

```bash
pip install -r requirements.txt
```

Create a `.env` file:

```
COHERE_API_KEY=your_key_here
```

### 8.2 Run the Agent

```bash
python agent.py
```

### 8.3 Run Evaluation

```bash
python evaluate.py
```

Output saved to:

```
eval_results.json
```

---

## 9. Notes on LLM Assistance

I used an LLM for:

- Generating code skeletons  
- Rewriting prompts with clarity  
- Debugging API deprecations  
- Refining documentation  

All **design decisions, safety logic, metrics, iteration choices, and final reasoning** were my own.

This reflects how I work in real life:  
**I orchestrate the tools, enforce standards, and make the decisions.**
