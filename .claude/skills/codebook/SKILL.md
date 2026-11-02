---
name: codebook
description: Create and collaboratively refine a project-local codebook from raw multilingual responses to one open-ended survey question. Use for discovering themes, defining categories, reviewing overlaps or gaps, presenting the codebook as readable tables, and maintaining the question folder's codebook.csv; do not use for production coding of all rows.
---

# Codebook

Treat each question directory as an independent coding unit. Its raw CSV contains the responses to one open-ended question; its canonical codebook is `codebook.csv` in that same directory.

## Locate and understand the data

- Use the directory named by the user. If only one plausible question directory exists, use it without asking.
- Identify the raw-data CSV by excluding generated files such as `codebook.csv` and `coded.csv`. Never overwrite the raw file.
- Detect the delimiter, quoting, headers, ID column, and response column from the file rather than assuming fixed names.
- Preserve the source identifier column and response column names as metadata for downstream coding. The identifier is normally `participant`; never assume a column named `ParticipantID`.
- Treat empty strings and explicit null markers as missing. Read all substantive responses before finalizing an initial codebook; duplicates and frequencies are evidence of prevalence.
- Infer the question context from the directory, column name, data, and user guidance. Ask only when ambiguity would materially change the codebook.

## Build the initial codebook

- Develop categories inductively from the meaning of complete responses, across all represented languages. Do not create language-based categories.
- Give every category a stable positive integer ID, concise name, operational definition, and representative verbatim examples.
- Make definitions mutually exclusive where practical. State relevant inclusion and exclusion boundaries, especially between neighboring categories.
- Cover the full range of substantive responses without creating categories for isolated wording variants. Add one final residual category such as `Other / unclear` for unusable or genuinely unmatched responses.
- Pilot the draft by mentally assigning every valid response to exactly one category. Merge, split, or clarify categories until the scheme is exhaustive, usable, and not needlessly granular.
- Use at most 28 categories including the residual category, and as few as the content allows; 28 is a ceiling, not a target. Fewer categories are easier for people to survey, so add a category only when it clearly makes sense in terms of content. Merge categories that hold only a few responses into a meaningful broader category. Do not merge where it would blur a distinct thematic focus: keep a theme separate when it is prominent in the data or when readers would need to act on or report it on its own.
- Keep category IDs as positive integers so they can be stored in a MariaDB `INT` column. The database column name is set by `/coder` and does not belong in the codebook itself.

The semantic work must be LLM-based. Scripts may be used only for lossless CSV parsing, counting, file writing, or structural validation; never use keyword rules, regexes, clustering, embeddings, or other automated shortcuts to infer or assign themes.

## Collaborate and refine

After creating or reviewing a codebook, show it in chat as a readable Markdown table with ID, name, concise definition, and representative examples. For a long codebook, split the display into logical sections while keeping one canonical file.

When refining an existing codebook:

- Read both `codebook.csv` and the relevant raw responses.
- Discuss proposed merges, splits, renames, boundary changes, or missing themes in concrete terms.
- Show the affected rows as a compact before/after table.
- Preserve IDs whenever possible. Do not renumber or reuse IDs without explicit user approval because existing coded data may depend on them.
- After an agreed change, update `codebook.csv`, recheck neighboring categories, and state the compatibility impact.

## Canonical file

Write `codebook.csv` as UTF-8, semicolon-delimited CSV with this exact column order:

```csv
"Category ID";"Category Name";"Category Definition";"Category Examples"
```

Quote every field and escape embedded double quotes according to standard CSV rules. Separate multiple examples inside the last field with `, `. The file in the question directory—not a file inside this skill—is the source of truth.
