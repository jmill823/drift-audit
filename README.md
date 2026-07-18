# drift-audit

Audits agent-delivered work against its own claims, classifies every confirmed defect into a drift class, and appends it to an append-only catch ledger with receipts.

## Install

```
npx skills add jmill823/drift-audit
```

This fetches `SKILL.md` from this repo and installs it into your agent's skills directory. No extra flags needed — the repo ships a single skill at its root.

If you'd rather install by hand: copy `SKILL.md` to `~/.claude/skills/drift-audit/SKILL.md`.

## The five drift classes

- **calibration-drift** — the output claims more certainty than its inputs support
- **process-rot** — a required step was skipped or quietly relaxed, though the output looks clean
- **doc-drift** — the record claims something the code no longer does
- **discipline-drift** — a rule followed by rote; the failure it guards against is no longer stated anywhere it can be read
- **false-verification** — a gate that passes by construction: the test asserts a proxy, not the property that can actually break

## Ledger entry schema

Every confirmed defect is appended to `ops/catch-ledger.json` as an object with these keys:

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

Append-only: a correction is a new entry, never an edit to a prior one. See `examples/catch-ledger.json` for a worked set, including a superseded-by chain.

## Verdict

The audit ends with one line: `PASS` (no confirmed defects) or `CATCH(n)` followed by the entries. The skill proposes fixes but never applies them, and never merges. The human decides.

## Part of Structural Mistrust

drift-audit is the operational core of the [Structural Mistrust](https://jmill823.github.io/deltascanner-ai-pm/structural-mistrust-brief.html) eval framework — treat agent-delivered work as claims to be checked, not facts to be trusted.

This repo runs its own skill — see [ops/catch-ledger.json](ops/catch-ledger.json) for the catches from its own build.

## License

MIT — see [LICENSE](LICENSE).

## Contact

Jeff Millett — millett.jeffrey@gmail.com — [LinkedIn](https://www.linkedin.com/in/jeffrey-millett-productmanager1214/)
