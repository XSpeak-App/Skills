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

```json
{
  "sdk-version": 1,
  "version": 1,
  "name": "My Analyzer",
  "steps": [
    {
      "id": "my-step"
    }
  ]
}
```

- The analyzer id is the folder name without the `.xanalyzer` extension.
- `name` is shown in the AI Actions popup.
- Everything else (`description`, `modes`, `context`, `verification`, and per-step `ui` / `min.confidence`) is optional. See the Schemas section of [analyzer-advanced.md](analyzer-advanced.md).

## 3. Add the step prompt files

Create a directory named after the step id under the language directory, `en/my-step/` (only English is supported so far), and add three files:

- `system.md`: the model's instructions: its role and how it should behave.
- `user.md`: the user prompt, typically contains `{{conversation.transcript}}`.
- `result.md`: describes what the model should put into the `result` field of its output.

These files are **instructions to the model, not the output itself**. See the Parameters section of [analyzer-advanced.md](analyzer-advanced.md) for everything you can reference in them.

## 4. Register the analyzer

Add its id to the `ids` array in `analyzers/index.json`:

```json
{
  "ids": [
    "summary.request",
    "my.analyzer"
  ]
}
```

## Example

The `summary.request` analyzer in this repository is a minimal single-step analyzer. It's a good reference when creating your own. To build on it with multiple steps, full context and verification see [Creating an advanced analyzer](analyzer-advanced.md).
