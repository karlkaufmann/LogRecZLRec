---
name: public-repo-packaging
description: Use before publishing this repository or before commits that might touch sensitive paths. Verifies allowlist-only staging against data/model/secret denylist.
---

# Public repo packaging

## When to use this skill

- Before publishing or pushing this repo to a public remote.
- Before any commit where the change set might include data, model artifacts, or secrets.

## Checklist

1. Run `git status` (staged and unstaged).
2. Verify no staged path matches the **denylist** below (same patterns as `.gitignore`).
3. Ensure only **allowlist** paths are staged for public-facing commits unless the team explicitly expands the policy.
4. Confirm the final list of files to be committed with `git diff --cached --name-only`.

## Allowlist (typical public commit)

- `*.ipynb`
- `evaluate_model.py`
- `README.md`
- `*evaluation*.md`, `*vyhodnoceni*.md`
- `.gitignore`
- `.cursor/rules/*`
- `.agent/skills/public-repo-packaging/SKILL.md`

## Denylist (never commit)

Same as `.gitignore` for this project, including at minimum:

- `*.csv`, `*.zip`, `*.gz`, `*.7z`
- `*.parquet`, `*.feather`
- `*.pkl`, `*.joblib`, `*.onnx`, `*.pt`, `*.h5`
- `*.db`, `*.sqlite`
- `.env`, `*.key`, `*.pem`, `*.secrets*`
- Directories: `data/`, `datasets/`, `raw/`, `exports/`, `venv/`, `__pycache__/`

If anything on the denylist appears staged, unstage it and abort the commit until the index is clean.
