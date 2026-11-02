# Qualitative Survey Autocoder

This repository creates and applies codebooks for multilingual open-ended survey questions. A project can contain multiple question directories; each question directory is an independent coding unit.

## Skills

- Use `/codebook` to create a codebook inductively from a question directory's raw CSV and to inspect, discuss, or refine it collaboratively. The resulting `codebook.csv` belongs in that question directory.
- Use `/coder` to apply an existing question-local `codebook.csv` semantically to all valid responses and write MariaDB SQL statements to `coded.csv` beside it.
- If a request includes both codebook changes and coding, finalize the codebook first and then code against that finalized version.

## Directory convention

```text
<project>/<question>/
|-- <raw-data>.csv
|-- codebook.csv
`-- coded.csv  (MariaDB SQL script)
```

## Project rules

- Never overwrite raw survey data.
- Detect raw-data CSV dialect and columns from the file; `codebook.csv` uses semicolon-delimited UTF-8, while `coded.csv` is UTF-8 SQL text despite its legacy `.csv` name. Existing participant rows are updated with `UPDATE ... WHERE participant = ...`; do not generate inserts that duplicate them.
- The MariaDB table name is the raw CSV basename without `.csv`; the source key column is `participant`. The category column is named `<response column>code`, without an underscore.
- Category discovery and row assignment require LLM-based semantic reasoning. Scripts may support lossless I/O and validation only, never determine a category.
- Keep codebook design separate from production coding. `/coder` must not silently change categories, and `/codebook` must not claim a full coding run has been completed.
- The legacy prompt files are background material, not runtime sources of truth. Do not import their ski-specific categories into unrelated questions.
