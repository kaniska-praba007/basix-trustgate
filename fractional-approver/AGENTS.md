# AGENTS.md

Canonical instructions for any AI coding agent working in this repo (Antigravity, Cursor, Claude Code, Codex). Read this fully before making changes.

## Mission

Build a **working alpha** of one workflow, end to end:

> Buyer application → fact extraction → MeTTa rule evaluation → verdict + confidence + rule trail → human override → audit trail.

One solid workflow beats a general platform. Do not add features outside this loop.

## Non-negotiables

1. **MeTTa decides. Nothing else does.** Verdicts, confidence values and overrides come from rules in `src/rules/approval.metta`. Never compute a verdict in Python, and never let an LLM produce one.
2. **The LLM extracts facts only.** `src/extract.py` returns structured fields. It must not output approve/reject language, scores, or recommendations.
3. **Every decision cites rule IDs.** Each rule that fires (R1, R2, ...) must appear in the trail with the reason it fired and any confidence deduction.
4. **The audit log is append-only.** An override adds an entry. It never edits or deletes the original decision.
5. **Missing data is handled explicitly.** Incomplete input must produce a visible, explained outcome (cap, flag, or reject), never a silent default.
6. **Never invent API behavior.** `hyperon` is pre-alpha. If you are not certain how a call behaves, write a 5-line test against the installed version and read the output before building on it.

## Architecture boundaries

| Layer | File | Owns | Must not |
|---|---|---|---|
| Extraction | `src/extract.py` | Free text → structured facts | Decide anything |
| Reasoning | `src/rules/approval.metta` | Rules, verdict, confidence | Format prose for humans |
| Bridge | `src/engine.py` | Run queries, parse atoms into dicts | Contain business rules |
| Audit | `src/audit.py` | Append-only log, one-page trail | Modify past entries |
| API/UI | `src/api.py`, `ui/` | Presentation, override trigger | Re-implement rules |

Human-readable reason strings are composed in Python from parsed atoms. Do not build prose inside MeTTa.

## Commands

```bash
pip install -r requirements.txt
python -m src.smoke_test        # verify hyperon before anything else
uvicorn src.api:app --reload    # run the API
pytest                          # run all tests
```

## Conventions

- Python 3.11+, type hints on public functions, `ruff` formatting if configured.
- One rule = one clearly named MeTTa definition with a comment header `;; R<N>: <one-line description>`.
- Rule IDs are stable. Never renumber a rule; deprecate it instead.
- Thresholds (concentration limit, confidence caps) live at the top of `approval.metta` as named facts, not magic numbers.
- Pin dependency versions. Run `pip freeze` after any install and commit `requirements.txt`.
- Secrets go in `.env` (git-ignored). Commit only `.env.example`.
- Keep README's rule table in sync with `approval.metta`. The `.metta` file is the source of truth.

## Definition of done (for any change)

- [ ] `pytest` passes, including the five demo fixtures in `data/cases.json`
- [ ] New or changed rule has a test for firing and for not firing
- [ ] Trail output shows the rule ID and reason for the new behavior
- [ ] README rule table matches the `.metta` file
- [ ] No verdict logic outside `approval.metta`

## Verify first (known unknowns)

- How `metta.run()` results map to Python objects in the pinned `hyperon` version. Confirm before writing `engine.py` parsing.
- Whether runtime `add-atom` overrides persist across separate `metta.run()` calls in one instance.
- Whether the installed version needs a specific import path for space/atom utilities.

Record what you learn in `docs/HYPERON_NOTES.md` so teammates don't rediscover it.

## Do not

- Do not add a database, auth system, or multi-tenant features.
- Do not add more than the planned rules without asking the human.
- Do not rewrite the README AI Disclosure to be vaguer. It is a graded submission requirement.
- Do not commit API keys, `.env`, or generated audit logs containing personal data.

## When unsure

Stop and ask the human. State the assumption you would make, and why. Prefer a short plan over a large unreviewed diff.

See `.agents/` for detailed rules, workflows and skills.
