---
name: coder
description: Semantically code every valid response to an open-ended survey question with the question directory's codebook.csv and emit MariaDB statements in coded.sql. Use for production coding; do not use to design or silently change the codebook.
---

# Coder

Treat each question directory as an independent coding unit containing raw response data and its own `codebook.csv`. Write MariaDB SQL statements to `coded.sql` in that directory unless the user specifies another path.

## Prepare the run

- Use the directory named by the user. If only one plausible question directory exists, use it without asking.
- Require and read the complete `codebook.csv` before assigning any response. If it is absent, stop and use `/codebook` first.
- Identify the raw CSV by excluding `codebook.csv`, `coded.sql`, and other clearly generated outputs. Never modify the raw file.
- Detect delimiter, quoting, encoding, headers, and response column from the actual file. The participant key column for this MariaDB workflow is `participant`; do not invent a `ParticipantID` column. Ask only if the source does not contain `participant` or the response column remains ambiguous. A question may span several response columns, for example `association1` to `association5`; all of them are coded with the directory's one codebook.
- Derive the target table name from the original raw-data filename without its `.csv` extension (for example, `heracles_project_224.csv` becomes `heracles_project_224`). Ask if the database uses a different table name.
- Name the category column `<response column>code`, without an underscore, for example `likecode` or `association1code`. If a question has several response columns, add one category column per response column. Validate that each name is a safe MariaDB identifier and use the same name in the `ALTER TABLE` and every `UPDATE` statement.
- A row is valid only when both `participant` and the response are present. Treat empty strings and explicit null markers as missing and skip those rows. With several response columns, a row is valid when `participant` and at least one response are present.
- Verify that valid `participant` values are unique in the source. If they are not unique, stop and ask for an additional key because `WHERE participant = ...` could update multiple database rows.

## Assign categories semantically

- Use LLM language understanding on every complete response. Consider its full meaning, context, sentiment, negation, humor, and language.
- Compare the response with all category definitions. Examples guide interpretation but do not override definitions.
- Assign exactly one category ID. If a response mentions several themes, assign the category of the first-mentioned theme (top of mind), regardless of how much space later themes take up and regardless of category IDs. Skip openers without a codable theme, such as general praise or "Nein, aber ...", and use the first concrete theme after them. A phrase that merely qualifies one theme (for example "Spaß für die ganze Familie") counts as one theme and is coded by its meaning. If several categories could fit the first-mentioned theme, choose by the category definitions, never by ID.
- With several response columns, treat each filled cell as one response and assign it its own category ID. Code each cell by its own content; repeated IDs within a participant are allowed.
- Use the residual category only when no substantive category fits reliably.
- Preserve participant IDs and input order exactly.
- Do not create, merge, rename, or reinterpret categories during coding. If the codebook repeatedly fails to cover the data, finish only if reliable residual coding remains possible; otherwise pause and recommend refinement with `/codebook`.

Category assignment itself must never be delegated to a script, keyword list, regex, lookup rule, clustering model, embedding similarity, or other automated shortcut. Those approaches discard context and are not an acceptable substitute for reading the comments. Mechanical tooling is allowed only to parse CSV losslessly, divide rows into manageable batches, write already-decided assignments, and validate structure.

For large files, work in bounded batches while retaining the full codebook for every batch. Keep a stable mapping from each input row to its assignment so no row is lost, duplicated, or reordered.

## Output

Write `coded.sql` as a UTF-8 MariaDB SQL script. The first statement must add the integer category column:

```sql
ALTER TABLE `heracles_project_224` ADD COLUMN `likecode` INT;
```

Follow it with one update statement per valid input row, in original order. `INSERT` is incorrect here because it would create duplicate rows instead of updating the existing participant:

```sql
UPDATE `heracles_project_224` SET `likecode` = 14 WHERE `participant` = '273821';
```

With several response columns, one `ALTER TABLE` statement adds all category columns, and each participant still gets one `UPDATE` that sets only the columns of filled cells:

```sql
ALTER TABLE `heracles_project_224` ADD COLUMN `association1code` INT, ADD COLUMN `association2code` INT, ADD COLUMN `association3code` INT;
UPDATE `heracles_project_224` SET `association1code` = 1, `association2code` = 2 WHERE `participant` = '273710';
```

Use backticks for table and column identifiers. Emit participant values as SQL string literals with embedded single quotes escaped by doubling them; emit category IDs as integers. Do not include comments, translations, confidence scores, or skipped rows in this file.

Before completing the run, verify mechanically that:

- the first statement is the single expected `ALTER TABLE` statement,
- the number of `UPDATE` statements equals the number of valid input rows,
- each valid participant appears exactly once in the original order,
- valid source participants are unique,
- every assigned ID exists in `codebook.csv`,
- with several response columns, each `UPDATE` sets exactly the category columns of that participant's filled cells, and
- every SQL statement ends with `;` and targets the expected table and category column or columns.

Report the paths, valid/skipped counts, and validation result in chat after writing the file. Do not dump the entire SQL file into chat unless the user asks.
