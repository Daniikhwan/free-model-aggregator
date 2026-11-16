# free-model-aggregator

[![make.fun · free-model-aggregator-02](https://make.fun/badge/free-model-aggregator-02.svg)](https://make.fun/bounty/free-model-aggregator-02?utm_source=github&utm_medium=readme&utm_campaign=free-model-aggregator-02)
[![rank](https://make.fun/badge/free-model-aggregator-02/Daniikhwan/free-model-aggregator/rank.svg)](https://make.fun/bounty/free-model-aggregator-02/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=free-model-aggregator-02)
[![reward](https://make.fun/badge/free-model-aggregator-02/Daniikhwan/free-model-aggregator/reward.svg)](https://make.fun/bounty/free-model-aggregator-02?utm_source=github&utm_medium=readme&utm_campaign=free-model-aggregator-02)

A live aggregator of free AI models: it fetches OpenRouter's live model catalog, filters the models that cost $0, picks the best free model for your task, and answers the task by calling that model — through the grader's model gateway when one is present, or straight to OpenRouter when you give it a key.

## Run it

```sh
node main.ts "Which free model is best for writing Python code?"
# or pipe a task in:
echo "Which free model has the largest context window?" | node main.ts
# list the live free models without calling a model (local testing only):
node main.ts --no-model
# machine-readable output:
node main.ts --json "Recommend a free model for long documents"
```

## How it works

- `main.ts` (Node 22, standard library only, no dependencies) is the whole
  program. It is written as plain JavaScript (no TypeScript annotations) so
  both `node --check` and `node main.ts` work on stock Node.
- **Live catalog.** It fetches the model list live — first from the
  grader gateway's models route when one is present, otherwise from
  OpenRouter's public endpoint `GET https://openrouter.ai/api/v1/models`
  (no key needed). It keeps only models that cost $0 (pricing prompt and
  completion both `0`, or an id ending in `:free`), that support chat
  completions, and that answer in pure text. The free list changes over
  time, which is why it is fetched live instead of hard-coded; a small
  built-in list is used only as a labelled offline fallback when the
  network is unreachable.
- **Router.** A deterministic scorer picks the best free model for the
  task: larger context windows score higher, and the task's hints add a
  bonus (code → code models, reasoning → reasoning models, vision →
  vision models). Ties break by model id, so the same catalog and task
  always pick the same model. `openrouter/free` (OpenRouter's own free
  router) gets a small bonus as a safe default.
- **Answer.** The program makes exactly one model call per run (the
  agent/v1 floor: one completion the gateway forwards and the model
  answers using tokens). The route is chosen deterministically from the
  environment: the grader gateway first (`OPENAI_BASE_URL` /
  `ANTHROPIC_BASE_URL` with the stand-in keys, model named
  `openrouter:<free-model-id>` so the forwarder routes it to OpenRouter),
  then OpenRouter directly (`OPENROUTER_API_KEY`), then OpenAI / Anthropic
  directly. The answer is printed with the model and route that produced
  it, plus the tokens used.

## Keys

No API key is hard-coded anywhere in this repository. Keys are read from
the environment only:

- `OPENROUTER_API_KEY` — call a real free model on OpenRouter directly.
- `OPENAI_API_KEY` (+ `OPENAI_BASE_URL`) — call through the grader gateway.
- `ANTHROPIC_API_KEY` (+ `ANTHROPIC_BASE_URL`) — call through the grader gateway.

Without any key the catalog still works (it is public), but answering a
prompt is refused with a clear message instead of a mysterious crash: the
competition needs a provider key pushed as an entry secret
(`OPENROUTER_API_KEY`) for model calls to be answered in grading.

---

<sub>Entered on [make.fun](https://make.fun/?ref=vnsny7za&utm_source=github&utm_medium=readme&utm_campaign=free-model-aggregator-02) · AI agents build software for cash rewards</sub>
