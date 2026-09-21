---
name: coder
description: Semantically code every valid response to an open-ended survey question with the question directory's codebook.csv and emit MariaDB statements in coded.csv. Use for production coding; do not use to design or silently change the codebook.
---

# Coder

Treat each question directory as an independent coding unit containing raw response data and its own `codebook.csv`. Write MariaDB SQL statements to `coded.csv` in that directory unless the user specifies another path.

## Prepare the run

- Use the directory named by the user. If only one plausible question directory exists, use it without asking.
- Require and read the complete `codebook.csv` before assigning any response. If it is absent, stop and use `$codebook` first.
- Identify the raw CSV by excluding `codebook.csv`, `coded.csv`, and other clearly generated outputs. Never modify the raw file.
- Detect delimiter, quoting, encoding, headers, and response column from the actual file. The participant key column for this MariaDB workflow is `participant`; do not invent a `ParticipantID` column. Ask only if the source does not contain `participant` or the response column remains ambiguous.
- Derive the target table name from the original raw-data filename without its `.csv` extension (for example, `heracles_project_224.csv` becomes `heracles_project_224`). Ask if the database uses a different table name.
- Before coding, ask the user for the exact category-variable name to add to the table. Do not guess it. Validate that it is a safe MariaDB identifier and use the same name in the `ALTER TABLE` and every `UPDATE` statement.
- A row is valid only when both `participant` and the response are present. Treat empty strings and explicit null markers as missing and skip those rows.
- Verify that valid `participant` values are unique in the source. If they are not unique, stop and ask for an additional key because `WHERE participant = ...` could update multiple database rows.

## Assign categories semantically

- Use LLM language understanding on every complete response. Consider its full meaning, context, sentiment, negation, humor, and language.
- Compare the response with all category definitions. Examples guide interpretation but do not override definitions.
- Assign exactly one category ID representing the dominant theme. If two categories fit equally well, choose the lower ID.
- Use the residual category only when no substantive category fits reliably.
- Preserve participant IDs and input order exactly.
- Do not create, merge, rename, or reinterpret categories during coding. If the codebook repeatedly fails to cover the data, finish only if reliable residual coding remains possible; otherwise pause and recommend refinement with `$codebook`.

Category assignment itself must never be delegated to a script, keyword list, regex, lookup rule, clustering model, embedding similarity, or other automated shortcut. Those approaches discard context and are not an acceptable substitute for reading the comments. Mechanical tooling is allowed only to parse CSV losslessly, divide rows into manageable batches, write already-decided assignments, and validate structure.

For large files, work in bounded batches while retaining the full codebook for every batch. Keep a stable mapping from each input row to its assignment so no row is lost, duplicated, or reordered.

## Output

Write `coded.csv` as a UTF-8 MariaDB SQL script, despite the legacy `.csv` filename. The first statement must add the requested integer category column:

```sql
ALTER TABLE `heracles_project_224` ADD COLUMN `category_variable` INT;
```

Follow it with one update statement per valid input row, in original order. `INSERT` is incorrect here because it would create duplicate rows instead of updating the existing participant:

```sql
UPDATE `heracles_project_224` SET `category_variable` = 14 WHERE `participant` = '273821';
```

Use backticks for table and column identifiers. Emit participant values as SQL string literals with embedded single quotes escaped by doubling them; emit category IDs as integers. Do not include comments, translations, confidence scores, or skipped rows in this file.

Before completing the run, verify mechanically that:

- the first statement is the single expected `ALTER TABLE` statement,
- the number of `UPDATE` statements equals the number of valid input rows,
- each valid participant appears exactly once in the original order,
- valid source participants are unique,
- every assigned ID exists in `codebook.csv`, and
- every SQL statement ends with `;` and targets the requested table and category variable.

Report the paths, valid/skipped counts, and validation result in chat after writing the file. Do not dump the entire coded CSV into chat unless the user asks.
