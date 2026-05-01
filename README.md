# AgentsVille Trip Planner — Multi-Agent Travel Assistant

A two-agent travel-planning notebook for the fictional city of AgentsVille.
A **chain-of-thought planner** turns a `VacationInfo` request into a
day-by-day itinerary, then a **ReAct revision agent** loops with four tools
(`calculator_tool`, `get_activities_by_date_tool`, `run_evals_tool`,
`final_answer_tool`) and seven evaluator functions until the plan passes
every constraint, including a traveler-feedback rule.

This is the Multi-Agent Travel Assistant project of Udacity's Agentic AI
nanodegree. The first review round passed with mentor-grade praise; v2 layers
in the three "future helpful resources" the reviewer suggested.

---

## Architecture

```
                ┌────────────────────────────────────────┐
                │  VacationInfo (Pydantic) + weather +   │
                │  activities pre-fetched as JSON        │
                └────────────────────┬───────────────────┘
                                     ▼
                ┌────────────────────────────────────────┐
                │  ItineraryAgent (Chain-of-Thought)     │
                │   role + task + JSON-schema prompt +   │
                │   weather/activity context             │
                └────────────────────┬───────────────────┘
                                     ▼
                       travel_plan_1: TravelPlan
                                     │
                                     ▼
                ┌────────────────────────────────────────┐
                │  Seven evaluators                      │
                │   • dates match                         │
                │   • total_cost accurate / within budget │
                │   • activities exist (no hallucination) │
                │   • interests covered per traveler      │
                │   • weather-compatibility (LLM judge)   │
                │   • traveler feedback incorporated      │
                └────────────────────┬───────────────────┘
                                     ▼
                ┌────────────────────────────────────────┐
                │  ItineraryRevisionAgent (ReAct loop)   │
                │   THOUGHT → ACTION (JSON tool call)    │
                │   → OBSERVATION → … →                  │
                │   final_answer_tool                    │
                └────────────────────┬───────────────────┘
                                     ▼
                       travel_plan_2: TravelPlan ✓
```

**Tools (rubric-required):**
- `calculator_tool` — accurate arithmetic via `numexpr`
- `get_activities_by_date_tool` — single-date wrapper around the activities API
- `run_evals_tool` — runs every evaluator on a given plan
- `final_answer_tool` — exits the ReAct loop with the validated plan

---

## v2 — post-review enhancements

The first review note praised the project as "rubric-aligned … mentor-grade"
and suggested three "future helpful resources" worth folding in. v2
implements all three (the v1 cells stay untouched so the saved evidence the
reviewer accepted is preserved):

| Reviewer suggestion | v2 implementation (notebook Part 8) |
|---|---|
| **Pydantic JSON Schema** in prompts | sanity-check cell that asserts the v1 prompt already contains `TravelPlan.model_json_schema()` verbatim, and reports its size |
| **OpenAI structured outputs** | `ItineraryAgentV2.get_itinerary()` passes `response_format=TravelPlan` through to `client.beta.chat.completions.parse` — the markdown-stripping branch and the `try/except` around `model_validate_json` are no longer needed |
| **ReAct extension** with stricter judges | `WeatherCompatibilityVerdict` and `FeedbackIncorporationVerdict` Pydantic models replace IS_COMPATIBLE / FULLY_INCORPORATED substring parsing; verdicts carry an explicit `reason` field that the next ReAct iteration can read directly |

---

## Repo layout

```
.
├── project_starter.ipynb       # main notebook — Parts 1-7 (v1, passes rubric) + Part 8 (v2)
├── project_lib.py              # provided helpers (ChatAgent, do_chat_completion,
│                                  print_in_box, mocked APIs, narrate_my_trip)
├── requirements.txt
├── README.md
├── .env.example
└── .gitignore
```

---

## Setup

```bash
git clone https://github.com/MelvinJoshua1375/agentsville-trip-planner.git
cd agentsville-trip-planner

python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt

cp .env.example .env
# edit .env and add your OPENAI_API_KEY (Vocareum voc-... or a normal sk-... key)
```

Inside the Udacity / Vocareum cloud workspace the same flow works without
the venv step — the workspace ships with the dependencies already installed.

---

## How to run

Open the notebook and run every cell top-to-bottom:

```bash
jupyter lab project_starter.ipynb
```

The notebook is organised into eight parts:

1. **Initial Setup** — env vars, OpenAI client, model choice
2. **Define Vacation Details** — `VacationInfo` Pydantic model
3. **Review Weather and Activity Schedules** — fetch the mocked APIs
4. **The ItineraryAgent** — chain-of-thought planner → `travel_plan_1`
5. **Evaluating the Itinerary** — seven evaluator functions
6. **Defining the Tools** — four ReAct tools
7. **The ItineraryRevisionAgent** — ReAct loop → `travel_plan_2` ✓
8. **v2 Enhancements** — structured outputs + Pydantic judges (post-review)

---

## Rubric coverage

| Rubric line | Where it's met |
|---|---|
| `VacationInfo` Pydantic model | Cell 8 |
| Weather + activities fetched via mocked APIs | Cells 10-11 |
| `ItineraryAgent` produces a TravelPlan with role / task / output-format / examples / context | Cell 14 (v1) and Cell 42 (v2 with `response_format=TravelPlan`) |
| Seven evaluators (dates, cost, hallucination, interests, weather, traveler feedback, budget) | Cells 18-22 + 33 |
| Four tools defined and used by the ReAct agent | Cells 26-31 |
| ReAct agent revises the plan and passes every evaluator | Cells 34-36 |
| Stand-out: structured outputs for both planner and judges (v2) | Part 8 (cells 41-46) |

---

## Notes

- The notebook expects the mocked `call_weather_api_mocked` and
  `call_activities_api_mocked` helpers in `project_lib.py`. Both return
  deterministic AgentsVille data for `2025-06-10` through `2025-06-15`.
- v2 cells use the same OpenAI proxy (Vocareum endpoint) as v1; no extra
  configuration is required.
- The audio narration cell (Part 7) is optional — it calls
  `client.audio.speech.with_streaming_response.create` to synthesise an
  mp3 of the trip summary. Skip it if you don't want to spend tokens on
  audio.

---

## License

Course materials remain Udacity's property; the student-authored notebook
filling and v2 additions are provided as a learning artefact.
