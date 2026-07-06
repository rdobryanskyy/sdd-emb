<!-- Template for `embroidery-qa` — copied to docs/embroidery/<design>/_qa/qa-report-<date>.md. -->
---
status: Final
updated_at: "<YYYY-MM-DD>"
---

# QA report — <design>

## Findings

<!-- One row per finding. No row for a rule that simply passed — the "rules checked" count below
     is the evidence coverage happened. -->

| Severity | Region | Rule | Value found | Threshold (source) | Fix via |
|---|---|---|---|---|---|
| `blocks-production \| warning \| informational` | `<region id>` | `<rule name>` | `<n>` | `<n, doc:<file>>` | `<embroidery-digitize \| embroidery-optimize \| embroidery-export>` |

## Rules checked

- **Ran:** `<list of rule numbers/names that ran>`
- **Skipped (missing domain doc):** `<list, or "none">`

## Verdict

`PASS | ISSUES-FOUND`

<!-- On PASS with open warnings/informational findings, note the user's acknowledgement here rather
     than silently dropping them from view. -->
