---
name: drift-audit
description: Use after a coding agent delivers work — a build, a PR, a doc change — to audit what the work claims against what actually shipped. Classifies confirmed defects into five drift classes and appends them to a receipts-backed catch ledger. Triggers on "drift audit", "audit this build", "verify against the spec", "check for drift", or before merging any agent-authored work.
---

# Drift audit

Audits agent-delivered work against its own claims, classifies every confirmed defect into a drift class, and appends it to an append-only catch ledger with receipts. The operational core of the [Structural Mistrust](https://jmill823.github.io/deltascanner-ai-pm/structural-mistrust-brief.html) eval framework.

## Establish the claim surface

Collect every claim the work makes about itself: the spec or build brief, any README or doc it touched, docstrings, commit messages, the PR description. Each claim becomes a checkable assertion. Work that claims nothing checkable is itself a finding — flag it before proceeding.

## Verify with receipts

For each claim, find the receipt that proves or breaks it: file path + line, command output, test result, commit hash.

- A claim without a receipt is UNVERIFIED — never assumed true. Confident-fill is the failure this skill exists to catch.
- When docs and code disagree, the doc is wrong until proven otherwise — never the reverse.
- Sweep for concept-level drift, not just exact-match: changing "10 agents" to "8 agents" in one file doesn't fix "ran a 10-agent fleet" in another — the sentence drifted, not just the token. Search for the concept, not the string.

## Classify

Every confirmed defect gets exactly one class:

- **calibration-drift** — the output claims more certainty than its inputs support
- **process-rot** — a required step was skipped or quietly relaxed, though the output looks clean
- **doc-drift** — the record claims something the code no longer does
- **discipline-drift** — a rule followed by rote; the failure it guards against is no longer stated anywhere it can be read
- **false-verification** — a gate that passes by construction: the test asserts a proxy, not the property that can actually break

## Append to the ledger

Write confirmed defects to `ops/catch-ledger.json`. Append-only: never edit or delete a prior entry; corrections are new entries that supersede by id.

```json
{
  "id": "CATCH-014",
  "date": "2026-07-16",
  "class": "doc-drift",
  "claim": "README states retries are capped at 3",
  "reality": "retry loop in fetch.py:88 is unbounded",
  "receipt": "fetch.py:88 @ commit a1b2c3d",
  "status": "open"
}
```

Bodies only: confirmed defects with receipts. Routine gate passes, style nits, and preferences never enter the ledger — padding the count launders the signal.

## Verdict

End with one line: `PASS` (no confirmed defects) or `CATCH(n)` followed by the entries. Propose fixes; never apply them in the same pass, and never merge. The human decides.

## Rules

- Never mark a claim verified without a receipt you can quote.
- Never tune a check until it passes — a failed check ships as a declared failure, marked and carried, with a revisit locked.
- Never accept the audited code's own tests as the only evidence; a system can't grade its own homework.
- Never edit past ledger entries; supersede with a new one.
- Never auto-fix or auto-merge; report and stop.
- Flag any test that passes by construction as false-verification, even while it passes.
- Sweep concepts, not strings — exact-match search alone misses the drift that matters.
