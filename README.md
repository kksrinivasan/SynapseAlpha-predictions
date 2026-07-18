# SynapseAlpha Predictions

**An append-only public record of SynapseAlpha weekly stock selections.**

This repository records each published edition in Git so its contents,
publication commit, and timing can be independently inspected. Historical
files are never edited or deleted, including when a publication was late or a
legacy file exposed a field that newer records omit.

## Publication classes

- `forward_prediction`: published on its edition date and eligible for public
  forward-performance reporting.
- `historical_recovery`: published after its edition date to recover missing
  history. It remains auditable but is never counted as forward evidence.

The authoritative classification of the 11 legacy files is
[`legacy_publication_ledger.json`](legacy_publication_ledger.json). Only Weeks
15, 18, 19, 22, and 29 were published on their recorded edition date and count
as forward evidence. Weeks 17, 20, 21, 23, 24, and 26 are historical
recoveries because their publication date was later. A database-only Week 16
edition has no verified public Git artifact and is therefore excluded.

## Integrity note

The legacy edition files included a model `score` in each pick. That field is
now treated as protected internal data. Removing it would rewrite the public
evidence, so the original files remain unchanged and the exposure is recorded
in the ledger. New records use an explicit allowlist and do not publish model
scores, score breakdowns, or other internal analysis.

New historical recoveries are written to `recoveries/`, while same-day forward
records are written to `editions/`. Each successful publication is accepted by
the application only after the exact JSON blob is verified on `origin/main`.

## Current record format

```json
{
  "edition_id": "week-2026-W30",
  "edition_date": "2026-07-24",
  "scan_date": "2026-07-24",
  "universe_size": 461,
  "published_at": "2026-07-24T12:00:00Z",
  "record_type": "forward_prediction",
  "forward_performance_eligible": true,
  "picks": [
    {
      "rank": 1,
      "ticker": "AAPL",
      "sector": "Technology",
      "decision": "Buy",
      "confidence": "HIGH",
      "price_at_pick": 185.5,
      "prediction_id": "exact-persisted-prediction-id"
    }
  ]
}
```

## How to verify

Inspect the immutable history of edition paths:

```bash
git log --format="%H %aI %s" -- editions/ recoveries/
```

Recompute the canonical picks checksum used by the current publisher:

```bash
python3 - <<'PY'
import hashlib
import json

path = "editions/week-2026-W29.json"
with open(path, encoding="utf-8") as handle:
    picks = json.load(handle)["picks"]
canonical = json.dumps(picks, sort_keys=True, separators=(",", ":")).encode()
print(hashlib.sha256(canonical).hexdigest())
PY
```

For legacy files, compare the result with `picks_sha256` in the ledger and
confirm `git log -1 --format=%H -- <path>` matches its `git_commit`.
`normalized_picks_sha256` covers the same picks after removing the retired
legacy `score` field and keeping the six fields shared with the database; it is
used only to prove that the database edition matches the Git evidence. For new
files, compare the full public-picks checksum with the `Picks-SHA256` trailer
in the publication commit. The checksum is not embedded in the JSON, because
an embedded checksum could be changed together with the file.

## Evaluation

Only `forward_prediction` records are eligible for the public scorecard.
Selections are reviewed against sector-relative returns at defined horizons;
recoveries and unverified database editions are excluded.

The system is evaluated on whether:

1. More than half of reviewed picks outperform their sector.
2. Higher decision tiers outperform lower tiers.
3. Average sector-relative return is positive.

These rankings are model outputs, not investment advice. Past performance is
not indicative of future results. See the
[full disclaimer](https://synapsealpha.ie/disclaimer).

## Repository rules

- No force pushes to `main`.
- No edits or deletions of published edition files.
- Every new publication targets one exact database edition.
- Publication failures are loud and retryable.
- A local file or database row is not public proof; verification on
  `origin/main` is required.

The first verified public edition is Week 15, published 6 April 2026.
