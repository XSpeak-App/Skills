> Note: it's not possible to create your own analyzers or contribute them to this repository yet. This is planned for the future.

# Creating an advanced analyzer

New to analyzers? Start with [analyzer-overview.md](analyzer-overview.md) to learn what an analyzer is, and [analyzer-basics.md](analyzer-basics.md) to create the simplest possible one.

This guide builds on the simple analyzer from [analyzer-basics.md](analyzer-basics.md). An advanced analyzer can:

- run a pipeline of **multiple steps**, where a later step references the output of earlier ones (`steps` in `recipe.json`, see Multi-step pipeline)
- **control how much of the transcript** it analyzes (`context` in `recipe.json`, see Context)
- **redact the user's own statements** from the transcript a step sees (`mask.user` per step, see Schemas)
- **shape its output** beyond the required `result` with `title`, `details`, `confidence`, and `quote` (see Step output)
- **stop the pipeline** when the model's confidence is too low (`min.confidence` per step, see Step output)
- **verify the final result** before it is shown (`verification` in the `live` section, see Verification)
- **aggregate full-context analysis** into a single result (`on-demand/aggregate`, see Aggregation)

## Multi-step pipeline

The `steps` array in `recipe.json` defines the pipeline. Each step gets a directory named after its `id` under the language directory (`en` — only English is supported so far). A step can reference the output of any previous step with `{{step.<step_id>.*}}` parameters (see Parameters) — for example, a second step can take the `result` of the first step and refine it.

## Context

The `context` object controls how much of the transcript the pipeline sees. It is set per mode: in the `on-demand` section for on-demand analysis and in the `live` section for live analysis.

- `full` (on-demand only): the whole conversation. It is processed in consecutive parts, so each step's `on-demand/aggregate/` directory is required.
- `max`: the default for on-demand analysis.
- `latest`: only the last N statements, where N is `statements`. The default for live analysis, with 20 statements.

## Aggregation

Aggregation is supported only by on-demand analysis. With `full` context, each step has an `on-demand/aggregate/` directory with `system.md` and `user.md`. The aggregator receives the outputs of the consecutive parts as `{{json}}` (see Aggregator parameters) and produces a single combined result for the whole conversation.

## File Layout

Analysis skills (analyzers) live under `analyzers/` (see `docs/structure.md`).
```text
<id.xanalyzer>/
    recipe.json
    <language>/           // only English supported so far
        step-1-id/
            core/
                system.md
                user.md
                result.md
            on-demand/
                title.md       (optional)
                aggregate/     (optional, required with full context)
                    system.md
                    user.md
            live/
                details.md     (optional)
                confidence.md  (optional)
                quote.md       (optional)
        step-2-id/
            core/
                system.md
                user.md
                result.md
            on-demand/
                title.md       (optional)
                aggregate/     (optional, required with full context)
                    system.md
                    user.md
            live/
                details.md     (optional)
                confidence.md  (optional)
                quote.md       (optional)
        ...
        verify/            (only if verification is specified in recipe.json)
            system.md
            user.md
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
  "on-demand": { // describes how on-demand analyzer must work, optional
    "context": {
      "transcript": "<full|max|latest>",
      "statements": "<int>" // only for latest, number of statements to take from transcript
    }, // optional, by default max
    "ui": { // optional, describes how the user interface look for on-demand analysis
      "action.name": "<action-name>", // optional, the name of the action in the AI Actions popup, by default name
      "action.image": "<symbol-name>", // optional, the name of a symbol from SF Symbols, the icon of the action in the AI Actions popup
      "result.title": "<result-title>", // optional
    },
  },
  "live": { // describes how live analyzer must work, optional
    "context": {
      "transcript": "<max|latest>",
      "statements": "<int>" // only for latest, number of statements to take from transcript
    }, // optional, by default latest with 20 statements
    "verification": { // optional, no verification if omitted
      "models": "<models-set|all>", // models-set means specific models, all means all models
      "model.ids": [
        "afm",
        "qwen-3-4b",
        "openai-compatible"
      ] // optional, required only when models is set to "models-set"
    }, // optional, controls the verify gate, see "Verification"
    "ui": { // optional, describes how the user interface look for live analysis
      "skill.name": "<skill-name>" // optional, the name of the skill in the Real-Time Insights panel, by default name
    }
  },
  "steps": [
    {
      "id": "<step.id>",
      "min.confidence": "<int 1-10>", // optional, used only for live analysis
      "mask.user": "<bool>" // optional, when true the user's own statements are redacted from the transcript passed to this step
    },
    // ...
    // other steps
  ],
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
Each step directory contains a mandatory `core/result.md` and other optional files each describing the content of one field:
- `core/result.md`: what to put into `result` (required)
- `on-demand/title.md`: what to put into `title` (optional)
- `live/details.md`: what to put into `details` (optional)
- `live/confidence.md`: what the 1–10 scale means for this step (optional)
- `live/quote.md`: the criteria for the quote from transcript for the model to pick to illustrate its result (optional).

These files are **instructions to the model, not the output itself**.

XSpeak produces the output contract from them and appends it to the step's system prompt.

## Verification
Optional gate for live analysis, controlled by the `verification` object in the `live` section of `recipe.json`. It is not part of `steps`: the host runs it after all steps have completed.
It decides whether the output of the last step is taken as the final result of the pipeline.

- Runs only when `verification` is specified in the `live` section. The `verify/` directory is then required and contains exactly two files: `system.md` and `user.md`.
- Can be enabled for all models (`"models": "all"`) or only for a specific set (`"models": "models-set"` with `model.ids` listing the models). The available model ids:
  - `afm`
  - `openai-compatible`
  - `qwen-3-4b`
  - `qwen-3-8b`
  - `qwen-3-14b`
  - `qwen-3.5-4b`
  - `qwen-3.5-9b`
  - `qwen-3.5-27b`
  - `qwen-3.6-27b`
  - `qwen-3.8-27b`
  - `qwen-3.5-35b-a3b`
  - `qwen-3.6-35b-a3b`
  - `gemma-4-e2b`
  - `gemma-4-e4b`
  - `gemma-4-12b`
- Gates the output of the last step.
- The prompt consists of `system.md` and `user.md`: the system prompt describes what exactly the model answers true or false to, and the user prompt is the question to answer with true or false. Both files are instructions to the model, not the output itself, and may reference the same parameters.

## Parameters
These are the parameters you can use in your prompts:
- `{{step.<step_id>.json}}`
- `{{step.<step_id>.title}}`
- `{{step.<step_id>.result}}`
- `{{step.<step_id>.details}}`
- `{{step.<step_id>.confidence}}`
- `{{step.<step_id>.quote}}`
- `{{history.last.json}}`
- `{{history.last.title}}`
- `{{history.last.result}}`
- `{{history.last.details}}`
- `{{history.last.confidence}}`
- `{{history.last.quote}}`
- `{{conversation.transcript}}`
- `{{conversation.spoken.language}}`
- `{{conversation.transcript.language}}`
- `{{conversation.context.conversation.type}}`

## Aggregator parameters
- `{{json}}`
