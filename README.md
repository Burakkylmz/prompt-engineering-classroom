# Prompt Engineering Classroom

## Project Overview

This project is a set of minimal, classroom-ready Jupyter Notebook labs for
teaching LLM prompt engineering, using real LLM calls through the Groq API.

- **LAB01** compares four prompt engineering techniques — baseline, context
  + constraints, output contract, and few-shot — on the same customer-support
  classification task. Hypothesis tested: **more prompt components do not
  always mean better performance.**
- **LAB02** compares four approaches to handling untrusted content —
  baseline, naive concatenation, untrusted-content separation, and
  deterministic application-level validation — on a return-decision
  assistant exposed to prompt injection. Hypothesis tested: **labeling
  untrusted content can reduce prompt injection success but is not a
  security guarantee by itself; deterministic application-level checks only
  catch the specific violations they are built for.**
- **LAB03** compares zero-shot prompting against three few-shot example
  selection strategies — no examples, redundant/biased examples, curated
  diverse examples, and the same diverse examples in reversed order — on a
  customer-support category classification task. Research questions
  explored: whether adding few-shot examples always improves accuracy,
  whether redundant examples perform differently from diverse ones, and
  whether example ordering alone changes individual model decisions.

All experiment logic lives directly inside each lab's notebook
(`notebooks/lab01_prompt_anatomy.ipynb`, `notebooks/lab02_instruction_hierarchy.ipynb`,
`notebooks/lab03_zero_shot_few_shot.ipynb`).
There is no `src/` package, no classes, and no custom framework — just small,
readable Python functions meant to be read and modified live during a class.
All three labs share the same `uv` environment, Docker setup, and
`config/settings.toml` model configuration.

## Requirements

- Python 3.10
- [uv](https://docs.astral.sh/uv/) (for local setup)
- Docker and Docker Compose (for container setup, optional)
- A Groq API key ([console.groq.com](https://console.groq.com)) — only
  needed to run live experiments

## Local Setup with uv

```bash
uv sync
uv run jupyter lab
```

This creates a `.venv`, installs all dependencies from `pyproject.toml` /
`uv.lock`, and launches JupyterLab using that environment. Open
`notebooks/lab01_prompt_anatomy.ipynb`.

## Docker Setup

```bash
docker compose up --build
```

JupyterLab starts inside the container and is reachable at
`http://127.0.0.1:8888` (bound to localhost only, with Jupyter's default
token authentication — check the container logs for the access URL/token).
The `notebooks/`, `config/`, and `data/` folders are mounted as volumes, so
local edits are reflected immediately inside the container. The same
container serves both LAB01 and LAB02 — no separate Dockerfile or service
is needed.

## Groq API Configuration

1. Copy `.env.example` to `.env`:

   ```bash
   cp .env.example .env
   ```

2. Add your real key to `.env`:

   ```dotenv
   GROQ_API_KEY=your_actual_key
   ```

3. `.env` is git-ignored and is never read into the notebook's output. For
   Docker, the same variable is passed through from your shell environment
   via `compose.yaml`.

## Running LAB01

Open `notebooks/lab01_prompt_anatomy.ipynb` and run the cells top to bottom.

- All steps up to and including Step 6 (client initialization) run without
  any API key and without any cost.
- Step 7 ("Execute Experiments") only performs real Groq API calls if you
  explicitly set `RUN_LIVE = True` in that cell. Leaving it `False` lets you
  inspect the whole notebook structure safely.
- Model and experiment parameters (model name, temperature, max tokens,
  max cases, repetitions) are defined in `config/settings.toml`.
- Prompt components for each experiment are defined in `config/prompts.yaml`
  and can be edited freely to test variations during class.

## LAB01 Experiment Description

| Experiment | Components |
|---|---|
| `E01_baseline` | instruction only |
| `E02_context_constraints` | instruction + context + constraints |
| `E03_output_contract` | previous + exact output format |
| `E04_few_shot` | previous + few-shot examples |

All four experiments perform the same task: classifying a synthetic Turkish
customer-support message (`data/messages.json`, 15 messages across
`billing`, `technical_issue`, `cancellation`, `complaint`,
`general_inquiry`) into a category and a priority (`low` / `medium` /
`high`).

Measured metrics per experiment: classification accuracy, output format
compliance rate, average prompt/completion tokens, and average client-side
latency — all computed from real API responses, never estimated.

## Running LAB02

Open `notebooks/lab02_instruction_hierarchy.ipynb` and run the cells top to
bottom. Same safety model as LAB01: nothing before Step 6 needs an API key,
and Step 7 only calls Groq if you explicitly set `RUN_LIVE = True`.

- Model/generation parameters come from the same `config/settings.toml` as
  LAB01 (no separate model config).
- Prompt components (policy, untrusted-content framing, output contract,
  user message templates) are defined in `config/lab02_prompts.yaml`.
- Test cases live in `data/lab02_cases.json`.

## LAB02 Experiment Description

A synthetic "Kaya Elektronik" Return Decision Assistant evaluates a customer
message (and, in three of the four experiments, an attached document)
against a fixed return policy and outputs `ONAYLA` / `REDDET` / `ESKALE`.

| Experiment | What changes |
|---|---|
| `E01_baseline` | No attached document; clean requests only. |
| `E02_naive_concatenation` | Document concatenated into the user message with no trust boundary; may contain prompt injection. |
| `E03_untrusted_separation` | Same documents as E02, but wrapped in an `<untrusted_document>` block with an explicit system-level instruction that it is data, not instructions. |
| `E04_deterministic_validation` | Reuses E03's exact prompt and raw model response (no extra API call); applies a simple, narrowly-scoped Python check on trusted CRM fields (`is_defective`, `days_since_purchase`) on top of the model's decision. |

Measured per experiment: model-level and system-level decision accuracy,
invalid-response rate, model-level and system-level prompt-injection attack
success rate (computed only over attack cases), and token/latency usage.
The deterministic check in E04 is deliberately narrow — it only catches one
specific policy violation and makes no broader security claim.

## Running LAB03

Open `notebooks/lab03_zero_shot_few_shot.ipynb` and run the cells top to
bottom. Same safety model as LAB01/LAB02: nothing before the client-setup
step needs an API key, and the "Execute Experiments" step only calls Groq
if you explicitly set `RUN_LIVE = True`.

- Model/generation parameters come from the same `config/settings.toml` as
  LAB01/LAB02 (no separate model config).
- LAB03's own experiment sizing (`LAB03_MAX_CASES`, `LAB03_REPETITIONS`) is
  defined inside the notebook itself, independent of `config/settings.toml`,
  so changing it never affects LAB01 or LAB02.
- Prompt components (identity, instruction, category definitions,
  classification rules, output contract, per-experiment example lists) are
  defined in `config/lab03_prompts.yaml`.
- The evaluation set lives in `data/lab03_evaluation.json` (20 messages,
  balanced across all 5 categories); the few-shot example candidate pool
  lives in `data/lab03_examples.json` (15 candidates) and is kept separate
  from the evaluation set to avoid data leakage.

## LAB03 Experiment Description

A synthetic Customer Support Assistant classifies a Turkish customer
message into one of `billing`, `technical_issue`, `cancellation`,
`complaint`, or `general_inquiry`. Every experiment shares the same model,
temperature, system instruction (identity, category definitions,
classification rules, output contract), evaluation cases, and repetitions;
only the few-shot examples differ.

| Experiment | Example Count | Selection Strategy | Ordering |
|---|---:|---|---|
| `E01_zero_shot` | 0 | No examples | N/A |
| `E02_redundant_few_shot` | 5 | Biased / redundant (4 billing + 1 technical_issue) | Fixed |
| `E03_diverse_few_shot` | 5 | Curated / balanced (one per category) | Fixed |
| `E04_reversed_order` | 5 | Same examples as E03 | Reversed |

Each few-shot example is represented as a `user`/`assistant` message pair
inserted into the `messages` array before the actual classification
question — this is visible directly in the request JSON inspection step,
comparing E01 against E03.

Measured per experiment: label accuracy, format compliance rate, invalid
response rate, API error rate, prompt/completion/total token usage, and
average latency — plus a per-category accuracy breakdown and an explicit
case-by-case comparison between E03 and E04 showing which individual
decisions changed when the same five examples were reordered.
