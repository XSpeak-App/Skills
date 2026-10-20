> Note: it's not possible to create your own analyzers or contribute them to this repository yet. This is planned for the future.

# Creating an advanced analyzer

New to analyzers? Start with [analyzer-overview.md](analyzer-overview.md) to learn what an analyzer is, and [analyzer-basics.md](analyzer-basics.md) to create the simplest possible one.

This guide builds on the simple analyzer from [analyzer-basics.md](analyzer-basics.md). An advanced analyzer can:

- run a pipeline of **multiple steps**, where a later step references the output of earlier ones (`steps` in `recipe.json`, see Multi-step pipeline)
- **control how much of the transcript** it analyzes (`context` in `recipe.json`, see Context)
- **shape its output** beyond the required `result` with `title`, `details`, `confidence`, and `quote` (see Step output)
- **stop the pipeline** when the model's confidence is too low (`min.confidence` per step, see Step output)
- **verify the final result** before it is shown (`verification`, see Verification)
- **aggregate full-context analysis** into a single result (`/aggregate`, see Aggregation)

## Multi-step pipeline

The `steps` array in `recipe.json` defines the pipeline. Each step gets a directory named after its `id` under the language directory (`en` — only English is supported so far). A step can reference the output of any previous step with `{{step.<step_id>.*}}` parameters (see Parameters) — for example, a second step can take the `result` of the first step and refine it.

## Context

The `context` object in `recipe.json` controls how much of the transcript the pipeline sees:

- `full`: the whole conversation. It is processed in consecutive parts, so each step's `/aggregate` directory is required.
- `max`: the default.
- `latest`: only the last N statements, where N is `statements`.

## Aggregation

With `full` context, each step has an `/aggregate` directory with `system.md` and `user.md`. The aggregator receives the outputs of the consecutive parts as `{{json}}` (see Aggregator parameters) and produces a single combined result for the whole conversation.

## File Layout

Analysis skills (analyzers) live under `analyzers/` (see `docs/structure.md`).
```text
<id.xanalyzer>/
    recipe.json
    <language>/           // only English supported so far
        step-1-id/
            system.md
            user.md
            result.md
            title.md       (optional)
            details.md     (optional)
            confidence.md  (optional)
            quote.md       (optional)
            aggregate/     (optional, required with full context)
                system.md
                user.md
        step-2-id/
            system.md
            user.md
            result.md
            title.md       (optional)
            details.md     (optional)
            confidence.md  (optional)
            quote.md       (optional)
            aggregate/     (optional, required with full context)
                system.md
                user.md
        ...
        verify/            (only if verification: true)
            question.md
```

## Schemas

### recipe.json

```json5
{
  "kit-version": "<int>", // the version of the XSpeak Skills Kit (the internal engine that uses skills)
  "version": "<int>", // the version of this skill
  "name": "<skill-name>", // used in the AI Actions popup 
  "description": "<skill-description>", // optional, describes what skill is for 
  "modes": [
    "request",
    "live"
  ], // optional, default both
  "context": {
    "transcript": "<full|max|latest>",
    "statements": "<int>" // only for latest, number of statements to take from transcript
  }, // optional, by default max
  "steps": [
    {
      "id": "<step.id>",
      "min.confidence": "<int 1-10>" // optional
    },
    // ...
    // other steps
  ],
  "ui": { // optional
    "result.title": "<result-title>", // optional
    "details.title": "<details-title>", // optional
  },
  "verification": { // optional, no verification if omitted
    "models": "<models-set|all>", // models-set means specific models, all means all models
    "models.ids": [
      "afm",
      "qwen-3-4b",
      "openai-compatible"
    ] // optional, required only when models is set to "models-set"
  } // optional, controls the verify gate, see "Verification"
}
```

## Step output
Every step outputs a JSON object with maximum five fields:
- `result` (required): the primary output of the step. Shown in the first section in AI analysis results and in the middle in Real-Time Insights panel.
- `title` (optional): short human-facing headline for the output, used as a title in AI analysis results in transcript view.
- `details` (optional): optional context output of the step. Shown as a separate section in the analysis results and at the bottom in Real-Time Insights panel.
- `confidence` (optional): the model's self-reported confidence in its own output, used to check whether we should proceed with the pipeline by comparing with the min confidence of the step.
- `quote` (optional): the criteria for the quote from transcript for the model to pick to illustrate its result.

### Output structure description
Each regular step directory contains a mandatory `result.md` and other optional files, each describing the content of one field:
- `result.md`: what to put into `result` (required)
- `title.md`: what to put into `title` (optional)
- `details.md`: what to put into `details` (optional)
- `confidence.md`: what the 1–10 scale means for this step (optional)
- `quote.md`: the criteria for the quote from transcript for the model to pick to illustrate its result (optional).

These files are **instructions to the model, not the output itself**.

XSpeak produces the output contract from them and appends it to the step's system prompt.

## Verification
Optional gate controlled by the `verification` flag in `recipe.json`. It is not part of `steps`: the host runs it after all steps have completed.
It decides whether the output of the last step is taken as the final result of the pipeline.

- Runs only when `verification: true`, the `verify/` directory is then required and contains exactly one file, `question.md`.
- Gates the output of the last step.
- `question.md` is the entire prompt for the step and may reference the gated step's output (e.g. `{{step.transform.result}}`, `{{conversation.transcript}}` and `{{conversation.context.conversation.type}}`).

## Parameters
These are the parameters you can use in your prompts:
- `{{step.<step_id>.json}}`
- `{{step.<step_id>.title}}`
- `{{step.<step_id>.result}}`
- `{{step.<step_id>.details}}`
- `{{step.<step_id>.confidence}}`
- `{{history.last.json}}`
- `{{history.last.title}}`
- `{{history.last.result}}`
- `{{history.last.details}}`
- `{{history.last.confidence}}`
- `{{conversation.transcript}}`
- `{{conversation.spoken.language}}`
- `{{conversation.transcript.language}}`
- `{{conversation.context.conversation.type}}`

## Aggregator parameters
- `{{json}}`
