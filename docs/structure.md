# XSpeak AI skills repository structure

```text
<root>/
    analyzers/
        index.json
        <id>.xanalyzer/   // one folder per analyzer, e.g. x.summary.request.xanalyzer
```

## index.json

`index.json` registers the analyzers in the repository by listing the id of each analyzer:

```json
{
  "ids": [
    "x.summary.request"
  ]
}
```

The layout of an analyzer folder is described in [analyzer-advanced.md](analyzer-advanced.md).
