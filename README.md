# Fractional Approval Agent

> A symbolic agent that decides whether a fractional-ownership purchase on BASIX.Market should be **approved, flagged, or rejected**, with a confidence level, a full rule-by-rule justification trail, and a human override that never erases the original reasoning.

*(Working title. Rename freely.)*

| | |
|---|---|
| **Hackathon** | SingularityNET x Omega x BASIX.Market Hackathon (SJIT & KU 2026) |
| **Track** | Next Level Devs: *The Agent That Can Be Trusted With Money* |
| **Challenge** | 02: Micro-lending / fractional-ownership approval agent |
| **Team** | `<TEAM NAME>`: `<Member 1>`, `<Member 2>`, `<Member 3>` |
| **Institution** | `<INSTITUTION>` |
| **Demo video** | `<LINK>` |

---

## Problem

Small financial decisions on a fractional-ownership marketplace (can this buyer purchase a share of this asset?) are usually made by opaque scoring or by manual review that doesn't scale. When a decision is wrong, nobody can say *which rule* caused it, and an auditor or compliance officer has nothing to defend.

Real applications also arrive **incomplete**: KYC half-finished, funding source unverified. A naive system either rejects everything incomplete (bad for the business) or approves it anyway (bad for the risk team).

## Solution

A decision agent where **a MeTTa rule engine makes the decision** and an LLM is used only to turn messy applications into structured facts.

- **Deterministic verdicts.** Same inputs, same outcome, every time.
- **Explicit confidence.** Every deduction that lowers confidence is logged with its rule ID.
- **Gap handling, not gap hiding.** Partial data caps confidence and routes to a human instead of silently passing or failing.
- **Human override.** A compliance officer can grant a documented exception. The engine re-evaluates live and the audit trail keeps both the original decision and the override.

## How it works

```
Application (free text / form)
        │
        ▼
 [LLM extraction]  ── structured facts only, no decisions ──┐
        │                                                   │
        ▼                                                   ▼
 [MeTTa Atomspace]  facts + rules  ──►  verdict + confidence + rule trail
        │
        ▼
 [Python layer]  parses atoms, formats reasons, appends to audit log
        │
        ▼
 [API / UI]  "Why?" interrogation · Human override · One-page audit trail
```

**Design principle:** the symbolic layer owns the control flow and the decision. The LLM is a subordinate skill, in the spirit of the Omega architecture.

## Decision rules

| ID | Rule | Effect |
|----|------|--------|
| R1 | Funding source not verified | **Reject** (hard stop) |
| R2 | KYC missing | **Reject** |
| R3 | KYC partial | Cap confidence at 60, **Flag** for human sign-off |
| R4 | Purchase pushes buyer above concentration limit | **Flag** |
| R5 | High-risk asset without verified KYC | **Flag** |
| R6 | All checks pass | **Approve** |

> Thresholds live in `src/rules/approval.metta` and are tunable. This table must match that file. If they diverge, the file wins and this table is a bug.

## Demo scenarios

Planned fixtures in `data/cases.json`. Expected outcomes are **design targets** until `pytest` passes.

| Case | Situation | Target verdict |
|------|-----------|----------------|
| 001 | Verified buyer, low-risk asset, low concentration | Approve |
| 002 | Partial KYC | Flag (confidence capped) |
| 003 | Funding source unverified | Reject |
| 004 | Concentration over limit | Flag |
| 005 | Partial KYC, then compliance override | Flag → Approve (override logged) |

## Sample audit trail

See [`docs/AUDIT_TRAIL_SAMPLE.md`](docs/AUDIT_TRAIL_SAMPLE.md) for the one-page format. Replace the illustrative sample with real output from the running system before submitting.

## Tech stack

- **Symbolic reasoning:** MeTTa via [`hyperon`](https://github.com/trueagi-io/hyperon-experimental) (pre-alpha, version pinned in `requirements.txt`)
- **Extraction:** `<Gemini model used>`
- **Backend:** Python, FastAPI
- **Frontend:** `<Streamlit / React>`
- **Dev tooling:** Google Antigravity (see `AGENTS.md`)

## Quickstart

```bash
git clone <REPO_URL> && cd fractional-approver
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env            # add your Gemini API key
python -m src.smoke_test        # confirms hyperon works before anything else
uvicorn src.api:app --reload
pytest
```

## Repo layout (target)

```
.
├── README.md
├── AGENTS.md              # canonical agent rules (Antigravity, Cursor, Claude Code, Codex)
├── GEMINI.md              # pointer to AGENTS.md
├── .agents/               # rules, workflows, skills for Antigravity
├── src/
│   ├── rules/approval.metta   # the decision logic
│   ├── engine.py              # hyperon bridge + atom parsing
│   ├── extract.py             # LLM extraction (facts only)
│   ├── audit.py               # append-only audit log
│   └── api.py
├── data/cases.json
├── tests/
└── docs/AUDIT_TRAIL_SAMPLE.md
```

## AI Disclosure (mandatory)

**Core reasoning engine:** All approve/flag/reject decisions and confidence scores are produced by deterministic MeTTa rules. No LLM makes or influences a verdict.

**LLMs used, and where:**

| Tool / model | Used for | Not used for |
|---|---|---|
| `<Gemini model>` | Extracting structured fields from free-text applications at runtime | Any decision, confidence value, or override |
| `<Antigravity / Gemini>` | Boilerplate, scaffolding, test generation, README drafting | Rule logic design (human-authored and reviewed) |
| `<Other, e.g. Claude>` | `<e.g. brainstorming, use-case analysis>` | `<...>` |

## What's next

- `<Real KYC provider integration>`
- `<Rule editor so risk teams can change thresholds without code>`
- `<Persistent Atomspace storage across sessions>`
- `<Multi-asset portfolio risk rules>`

## Team

| Name | Role |
|---|---|
| `<Name>` | System / MeTTa lead |
| `<Name>` | Frontend & UX |
| `<Name>` | Data, docs & demo |

## License

`<MIT / other>`
