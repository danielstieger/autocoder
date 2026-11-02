<div align="center">

```text
 █████╗ ██╗   ██╗████████╗ ██████╗  ██████╗ ██████╗ ██████╗ ███████╗██████╗
██╔══██╗██║   ██║╚══██╔══╝██╔═══██╗██╔════╝██╔═══██╗██╔══██╗██╔════╝██╔══██╗
███████║██║   ██║   ██║   ██║   ██║██║     ██║   ██║██║  ██║█████╗  ██████╔╝
██╔══██║██║   ██║   ██║   ██║   ██║██║     ██║   ██║██║  ██║██╔══╝  ██╔══██╗
██║  ██║╚██████╔╝   ██║   ╚██████╔╝╚██████╗╚██████╔╝██████╔╝███████╗██║  ██║
╚═╝  ╚═╝ ╚═════╝    ╚═╝    ╚═════╝  ╚═════╝ ╚═════╝ ╚═════╝ ╚══════╝╚═╝  ╚═╝
```

**Open-ended survey answers in. Countable categories out.**

![Claude Code](https://img.shields.io/badge/Claude_Code-skills-D97757)
![MariaDB](https://img.shields.io/badge/MariaDB-SQL_output-003545)
![Languages](https://img.shields.io/badge/answers-any_language-2E7D32)
![Codebook](https://img.shields.io/badge/codebook-max_28_categories-5C6BC0)

</div>

---

Claude Code builds a codebook from the raw answers, then reads and codes every answer and writes the result as MariaDB SQL. It works with any open-ended question: improvement ideas, likes, missing offers, or brand associations.

## How it works

```text
  raw answers       /codebook          /coder           database
┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│  survey.csv │──▶│ categories  │──▶│  coded.sql  │──▶│   MariaDB   │
└─────────────┘   └─────────────┘   └─────────────┘   └─────────────┘
                         ▲
                   you review and
                    refine them
```

## Quick start

Open this repository in Claude Code and put the raw CSV into a project folder. Then, for each question:

| Step | Command | What happens |
|:---:|---|---|
| **1** | `/codebook` | Creates the categories. Review and refine them together. |
| **2** | `/coder` | Applies the finished codebook to all answers. |
| **3** | Run `coded.sql` | Adds the code columns to the database and fills them. |

## Files

Each question has its own folder:

```text
<project>/<question>/
├── <raw-data>.csv       raw answers, never overwritten
├── codebook.csv         categories with definitions and examples
├── compact_codes.txt    short category labels for the analysis tool
└── coded.sql            SQL that adds the code columns and fills them
```

A file that holds several questions is first split into one folder per question.

## Coding rules

| Rule | In short |
|---|---|
| **One code per answer** | If an answer names several themes, the first-mentioned one counts (top of mind). |
| **At most 28 categories** | As few as the content allows, without blurring a distinct theme. |
| **Code columns** | Named `<column>code`, for example `likecode`. |
| **Category IDs** | Start at 1, or at 0 for brand-image questions. |
| **Brand-image questions** | Each association is coded on its own, with one shared codebook for all association columns. |
| **Model reads every answer** | Scripts only handle files and checks, and never pick a category. |

The full rules are in [CLAUDE.md](CLAUDE.md) and in the two skills under [.claude/skills](.claude/skills).

## Data

> [!IMPORTANT]
> Project folders contain survey data and are not committed to this repository, so back them up separately.
