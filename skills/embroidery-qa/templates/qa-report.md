<!-- Template for `embroidery-qa` — copied to docs/embroidery/<design>/_qa/qa-report-<date>.md. -->
---
status: Final
updated_at: "<YYYY-MM-DD>"
---

# QA report — <design>

## Findings

<!-- One row for each finding. No row for a rule that passed. The "rules checked" count below
     is the evidence of coverage. -->

| Severity | Region | Rule | Value found | Threshold (source) | Fix via |
|---|---|---|---|---|---|
| `blocks-production \| warning \| informational` | `<region id>` | `<rule name>` | `<n>` | `<n, doc:<file>>` | `<embroidery-digitize \| embroidery-optimize \| embroidery-export>` |

## Rules checked

- **Ran:** `<list of rule numbers/names that ran>`
- **Skipped (missing domain doc):** `<list, or "none">`

## Verdict

`PASS | ISSUES-FOUND`

<!-- On PASS with open warnings or informational findings, record here that the user acknowledged
     them. Do not silently remove them from view. -->
