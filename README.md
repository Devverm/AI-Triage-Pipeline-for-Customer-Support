# Customer Support Ticket Analyzer & Router

An AI-powered system that classifies and routes customer support tickets based on severity and priority, using a multi-agent architecture.

The pipeline reads incoming tickets, runs them through two specialized AI agents, applies rule-based routing logic, logs full reasoning, and surfaces everything in an interactive dashboard.

---

## Features

- **Multi-Agent System** — separate agents evaluate ticket severity (from subject/message content) and customer priority (from account metadata).
- **Structured Reasoning** — each agent returns a 1–10 score, a category label, and an explanation.
- **Rule-Based Routing** — combines both scores to route tickets to the right team (VIP escalation, critical response, dedicated success, standard support, or low-priority queue).
- **Evaluation Framework** — compares routing output against expected results for a test set and reports accuracy.
- **Full Logging** — every agent input, output, and routing decision is appended to `ai_chat_history.txt` with timestamps.
- **Interactive Dashboard** — Streamlit app with KPI tiles, filters, score distributions, and time-trend charts.

---

## Project Structure

```
.
├── agents/
│   ├── severity_agent.py     # Scores ticket severity from subject/message
│   └── priority_agent.py     # Scores customer priority from account metadata
├── pydantic_ai_mock.py       # Lightweight agent framework (OpenRouter-backed, Pydantic-validated)
├── main.py                   # Entry point: runs agents, routes tickets, saves results, launches dashboard
├── evaluation.py             # Runs agents against test_tickets.json and checks routing accuracy
├── dashboard.py              # Streamlit dashboard for exploring results
├── test_tickets.json         # Input tickets
├── results.json              # Output: scores, categories, routing decisions
├── ai_chat_history.txt       # Full agent input/output/reasoning log
├── requirements.txt
└── .env                      # API key (not committed)
```

---

## How It Works

1. **Ticket Input** — `test_tickets.json` holds incoming tickets (subject, message, and customer metadata).
2. **Agent Processing** — each ticket is sent to:
   - `SeverityAgent`, which analyzes the subject and message text.
   - `PriorityAgent`, which evaluates customer tier, revenue, ticket history, and account age.
3. **Model Reasoning** — each agent returns a score (1–10), a category (e.g. "critical", "low"), and an explanation for the decision.
4. **Routing** — `route_ticket()` in `main.py` combines both scores into a routing decision:
   - Severity ≥ 8 and Priority ≥ 8 → VIP Escalation Team
   - Severity ≥ 8 → Critical Response Team
   - Priority ≥ 8 → Dedicated Customer Success Team
   - Severity ≥ 5 or Priority ≥ 5 → Standard Support Team
   - Otherwise → Low Priority Queue
5. **Logging** — inputs, outputs, and reasoning for every agent call are appended to `ai_chat_history.txt`.
6. **Results** — all ticket outcomes are saved to `results.json`.
7. **Dashboard** — `dashboard.py` reads `results.json` and renders KPIs, filters, score distributions, and trends over time.

---

## Setup

```bash
git clone <your-repo-url>
cd <repo-folder>

python -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate

pip install -r requirements.txt
```

Create a `.env` file with your API key:

```
OPENROUTER_API_KEY=your_key_here
```

Run the pipeline:

```bash
python main.py
```

This processes all tickets, saves `results.json`, and automatically launches the dashboard. If it doesn't open automatically:

```bash
streamlit run dashboard.py
```

---

## Model Configuration

This project uses [OpenRouter](https://openrouter.ai/) as a drop-in API layer, so you can plug in different LLMs without changing the agent logic. Update the model name in `agents/severity_agent.py` and `agents/priority_agent.py`, e.g.:

- `mistralai/mistral-7b-instruct`
- `openai/gpt-3.5-turbo`
- `meta-llama/llama-3-8b-instruct`

Any model that returns chat-completion-style JSON responses will work.

---

## Evaluation

```bash
python evaluation.py
```

This runs every ticket in `test_tickets.json` through both agents, compares the resulting routing decision to the `expected` field on each ticket, and prints a pass/fail summary with a final accuracy score. Useful for testing prompt changes or comparing models.

---

## Use Cases

| Use Case | How It Helps |
|---|---|
| Customer Support Automation | Classify, prioritize, and route tickets instantly |
| Product Feedback Analysis | Separate critical bugs from minor UI issues |
| Tiered SaaS Support | Prioritize enterprise/premium customers dynamically |
| LLM Evaluation | Swap in different models and compare routing consistency |
| Agent Research | Extend with additional agents (sentiment, complexity, urgency) |

---

## Notes

- No API keys are bundled — you'll need your own OpenRouter (or compatible) key.
- Model responses are parsed by extracting the first JSON object from the output, so agents are prompted to return clean structured JSON.
- Ticket data is centralized in `test_tickets.json` to avoid duplication across files.

---

## Author

_Add your name and links here._
