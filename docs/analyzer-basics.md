> Note: it's not possible to create your own analyzers or contribute them to this repository yet. This is planned for the future.

# Creating a simple analyzer

This guide walks through creating the simplest possible analyzer.

Not sure what an analyzer is? See [analyzer-overview.md](analyzer-overview.md). For the full specification of every field and option, see [analyzer-advanced.md](analyzer-advanced.md).

## 1. Create the analyzer folder

Analyzers live under `analyzers/`, which already exists in the repository. You only need to create a folder for your analyzer, named after the analyzer id, with the `.xanalyzer` extension:

```text
analyzers/my.analyzer.xanalyzer/
```

## 2. Add recipe.json

The minimum recipe, with a single step:

```json5
{
  "kit-version": 1, // the version of the XSpeak Skills Kit (the internal engine that uses skills)
  "version": 1, // the version of this skill
  "name": "My Analyzer", // the display name, shown in the AI Actions popup
  "steps": [ // the pipeline of steps
    {
      "id": "my-step" // the step's id, its prompts live in en/my-step/
    }
  ]
}
```

- `kit-version`, `version`, `name`, and `steps` (with each step's `id`) are required.
- `name` is shown in the AI Actions popup.
- `description`, `modes`, the `manual` and `live` sections, and per-step `min.confidence` and `mask.user` are optional. See the Schemas section of [analyzer-advanced.md](analyzer-advanced.md).

## 3. Add the step prompt files

Create a directory named after the step id under the language directory, `en/my-step/core` (only English is supported so far), and add three files:

- `system.md`: the model's instructions: its role and how it should behave.
- `user.md`: the user prompt, typically contains `{{conversation.transcript}}`.
- `result.md`: describes what the model should put into the `result` field of its output.

These files are **instructions to the model, not the output itself**. See the Parameters section of [analyzer-advanced.md](analyzer-advanced.md) for everything you can reference in them.

## 4. Register the analyzer

Add its id to the `ids` array in `analyzers/index.json`:

```json
{
  "ids": [
    "x.summary.manual",
    "my.analyzer"
  ]
}
```

## Example

The `x.summary.manual` analyzer in this repository is a minimal single-step analyzer. It's a good reference when creating your own. To build on it with multiple steps, full context and verification see [Creating an advanced analyzer](analyzer-advanced.md).
