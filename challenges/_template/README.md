# Challenge template

Copy this folder to `challenges/challenge-NN/` for each challenge. Each stage folder holds
the artifact for that stage plus its `manifest.json` (see ../../ARTIFACT_SCHEMA.md) and,
where applicable, its `provenance.json`.

| Folder | Holds | Exposure state |
|---|---|---|
| 00-issuer-artifact | The issuer's operational evidence record | NOT_APPLICABLE |
| 01-evidence-side | The independent evidence-side determination, frozen and hashed before cross-exposure | PRE_CROSS_EXPOSURE |
| 02-consequence-side | The independent consequence-side determination, frozen and hashed before cross-exposure | PRE_CROSS_EXPOSURE |
| 03-reconciliation | The non-authoritative comparison of the two frozen determinations | POST_CROSS_EXPOSURE |
| 04-final-receipt | The final reconciliation receipt | POST_CROSS_EXPOSURE |

A stage folder that is empty stays empty. A missing original is recorded as `NOT_PRESENT`,
never reconstructed (see ../../EXPOSURE_MODEL.md).
