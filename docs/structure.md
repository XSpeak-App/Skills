# XSpeak AI skills repository structure

```text
<root>/
    analyzers/
        index.json
        <id>.xanalyzer/   // one folder per analyzer, e.g. x.summary.manual.xanalyzer
            recipe.json
            <language>/   // only English (en) supported so far
                <step-id>/
                    core/        // core prompts, required
                    manual/      // specifications for manual analysis, required to appear in AI Actions
                    live/        // specifications for live analysis, required to appear in Real-Time Insights
```

## index.json

`index.json` registers the analyzers in the repository by listing the id of each analyzer:

```json
{
  "ids": [
    "x.summary.manual"
  ]
}
```

The layout of an analyzer folder is described in [analyzer-advanced.md](analyzer-advanced.md).
