# What is an analyzer?

An analyzer is an AI skill that analyzes a conversation and produces a structured result.

## Where it runs

An analyzer can run in two places:
- **AI Actions**: the user invokes the analyzer on demand from the AI Actions popup.
- **Real-Time Insights panel**: the analyzer runs during the conversation and shows its insights in real time.

The `modes` field in `recipe.json` declares which of the two an analyzer supports: `"request"`, `"live"`, or both (the default).

## How it works

1. The host takes the conversation transcript and passes a part of it to the analyzer. How much is passed is controlled by the `context` field in `recipe.json`: `full`, `max` (default), or `latest` with a number of statements.
2. The transcript goes through a pipeline of steps. Each step is a prompt to the model. Every step outputs a JSON object with a required `result` field and optional `title`, `details`, `confidence`, and `quote` fields.
3. With `full` context, the conversation is processed in consecutive parts and the outputs are aggregated into a single result.
4. An optional verification gate runs after the last step and decides whether its output is taken as the final result.
5. The result is shown to the user, in the AI analysis results in transcript view, or in the Real-Time Insights panel.

## What it looks like

An analyzer is a folder under `analyzers/` with a `recipe.json` and prompt directories for the supported language (only English, `en`, so far), registered in `analyzers/index.json`:

```text
analyzers/
    my.analyzer.xanalyzer/
        recipe.json
        en/
            my-step/
                system.md
                user.md
                result.md
```

See [Repository structure](structure.md) for the layout of the whole repository.

## Next steps

- [Creating a simple analyzer](analyzer-basics.md): build the simplest possible analyzer.
- [Creating an advanced analyzer](analyzer-advanced.md): multistep pipelines, full context, verification, and the full specification.
