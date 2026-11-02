# Qualitative Survey Autocoder

This repository creates and applies codebooks for multilingual open-ended survey questions. A project can contain multiple question directories; each question directory is an independent coding unit.

## Skills

- Use `/codebook` to create a codebook inductively from a question directory's raw CSV and to inspect, discuss, or refine it collaboratively. The resulting `codebook.csv` belongs in that question directory.
- Use `/coder` to apply an existing question-local `codebook.csv` semantically to all valid responses and write MariaDB SQL statements to `coded.sql` beside it.
- If a request includes both codebook changes and coding, finalize the codebook first and then code against that finalized version.

## Directory convention

```text
<project>/<question>/
|-- <raw-data>.csv
|-- codebook.csv
|-- compact_codes.txt  (short category labels for the analysis tool)
`-- coded.sql  (MariaDB SQL script)
```

- If a raw file holds several questions, split it into one question directory per question. Each copy keeps the raw file's name and contains `participant` plus that question's response column or columns. The original file stays unchanged.
- If one question comes in a separate file, such as `heracles_project_224_BRAND.csv`, copy it unchanged into its question directory under the main raw file's name (`heracles_project_224.csv`), so the SQL targets the main table. Check first that all its participants exist in the main file.

## Project rules

- Never overwrite raw survey data.
- Detect raw-data CSV dialect and columns from the file; `codebook.csv` uses semicolon-delimited UTF-8, while `coded.sql` is a UTF-8 SQL script. Existing participant rows are updated with `UPDATE ... WHERE participant = ...`; do not generate inserts that duplicate them.
- The MariaDB table name is the raw CSV basename without `.csv`; the source key column is `participant`. The category column is named `<response column>code`, without an underscore.
- A question may span several response columns, such as `association1` to `association5` for a brand-image question. These columns share one codebook, and each gets its own category column.
- Category IDs start at 1, except for brand-image questions, where they start at 0.
- Category discovery and row assignment require LLM-based semantic reasoning. Scripts may support lossless I/O and validation only, never determine a category.
- Keep codebook design separate from production coding. `/coder` must not silently change categories, and `/codebook` must not claim a full coding run has been completed.
