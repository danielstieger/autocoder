---
name: coder
description: Semantically code every valid response to an open-ended survey question with the question directory's codebook.csv. Use for production coding of multilingual raw CSV data and creation of coded.csv; do not use to design or silently change the codebook.
---

# Coder

Treat each question directory as an independent coding unit containing raw response data and its own `codebook.csv`. Write the completed assignments to `coded.csv` in that directory unless the user specifies another path.

## Prepare the run

- Use the directory named by the user. If only one plausible question directory exists, use it without asking.
- Require and read the complete `codebook.csv` before assigning any response. If it is absent, stop and use `$codebook` first.
- Identify the raw CSV by excluding `codebook.csv`, `coded.csv`, and other clearly generated outputs. Never modify the raw file.
- Detect delimiter, quoting, encoding, headers, participant-ID column, and response column from the actual file. Ask only if multiple columns remain genuinely ambiguous.
- A row is valid only when both participant ID and response are present. Treat empty strings and explicit null markers as missing and skip those rows.

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

Write `coded.csv` as UTF-8 semicolon-delimited data without a header, one line per valid input row:

```text
ParticipantID;CategoryID;-
```

Do not include comments, translations, confidence scores, or skipped rows in this file. Before completing the run, verify mechanically that:

- output rows equal valid input rows,
- each valid participant appears exactly once in the original order,
- every assigned ID exists in `codebook.csv`, and
- every output row has exactly three fields and `-` in the third field.

Report the paths, valid/skipped counts, and validation result in chat after writing the file. Do not dump the entire coded CSV into chat unless the user asks.
