# Artifact Manifest — Field Reference

This is the human-readable reference for artifact manifests. The machine-checkable
definition is [schemas/artifact-manifest.schema.json](schemas/artifact-manifest.schema.json);
where this document and the schema differ, the schema governs structure and the
governance documents govern meaning.

Every governed artifact carries a manifest (Control B — provenance identity). A manifest
identifies an artifact; it does **not** by itself make the artifact immutable (Control A)
or eligible to advance (Control C).

---

## 1. Fields

| Field | Required | Type | Meaning |
|---|---|---|---|
| `artifact_id` | yes | string | Stable unique identifier for this artifact. |
| `challenge_id` | yes | string | The challenge this artifact belongs to (e.g. `challenge-01`). |
| `artifact_role` | yes | enum | The artifact's governed role. See §2. |
| `designated_authority` | yes | string | The human/system authority responsible for the determination. Never an AI model on its own. |
| `source_author` | yes | string | Who produced the artifact bytes (may note AI assistance). |
| `created_at` | no | date-time \| null | When the artifact was created (RFC 3339). |
| `received_at` | no | date-time \| null | When this repository/custodian received it. |
| `frozen_at` | no* | date-time \| null | When the artifact was frozen. Required once `status` is `FROZEN`. |
| `sha256` | no* | 64-hex \| null | SHA-256 of the frozen bytes. Required once `status` is `FROZEN`; `null` when `NOT_PRESENT`. |
| `version` | yes | integer ≥ 1 | Version of this artifact. Corrections increment or supersede. |
| `status` | yes | enum | Lifecycle status. See §3. |
| `exposure_state` | yes | enum | Cross-exposure state. See [EXPOSURE_MODEL.md](EXPOSURE_MODEL.md) and §4. |
| `supersedes` | no | string \| null | `artifact_id` this one supersedes, if any. |
| `source_artifact_refs` | no | array of string | `artifact_id`s this artifact references (e.g. the issuer artifact a determination was based on). |
| `artifact_file_ref` | no | string \| null | Repo-relative path of the file whose exact bytes `sha256` covers. Binds the hash to specific content. |
| `notes` | no | string | Free-text notes. Must not contain protected information. |

\* Conditionally required (enforced by the schema): a `FROZEN` **or** `SUPERSEDED`
artifact must have `frozen_at` and `sha256`; any artifact with a non-null `frozen_at`
must have a non-null `sha256`; a `NOT_PRESENT` artifact has `sha256: null` **and**
`frozen_at: null`.

Additional optional fields (e.g. an `ai_assistance` record) are permitted by the schema
to strengthen auditability; they never grant authority. When `ai_assistance.adopted_by`
is set, `ai_assistance.adoption_record_ref` (a governance-record reference) is required —
adoption is a governed act, never a free-text claim (see [GOVERNANCE.md](GOVERNANCE.md) §6).

---

## 2. Artifact roles (`artifact_role`)

At minimum, the following roles are distinguished:

| Role value | Authority-bearing? | Description |
|---|---|---|
| `issuer_artifact` | yes (issuer) | The operational evidence record issued by CustodyIQ. |
| `evidence_side_determination` | yes (evidence) | Independent evidence-side determination. |
| `consequence_side_determination` | yes (consequence) | Independent consequence-side determination. |
| `reconciliation` | no | Non-authoritative comparison of two frozen determinations. |
| `final_receipt` | no (records outcome) | Final reconciliation receipt / audit artifact. |
| `governance_artifact` | governance only | Eligibility determinations, adoptions, STOP records, scope decisions. |
| `non_authoritative_summary` | no | AI or human summary/explanation/draft — not a source of authority. |
| `correspondence` | no | Emails, messages, and other correspondence. |

An artifact's role determines whether it can carry evidentiary or consequence authority.
`reconciliation`, `final_receipt`, `non_authoritative_summary`, and `correspondence` are
never sources of evidence.

---

## 3. Status (`status`)

| Value | Meaning |
|---|---|
| `DRAFT` | In progress; not frozen; not citable as final. |
| `FINALIZED` | Declared final by its authority but not yet frozen/hashed. |
| `FROZEN` | Final, hashed, and manifested. Bytes not edited in place thereafter. |
| `SUPERSEDED` | Replaced by a newer version/artifact; retained unchanged for provenance. |
| `WITHDRAWN` | Retracted by its authority; retained unchanged for provenance. |
| `NOT_PRESENT` | The artifact does not exist / was never captured. `sha256` is `null`. Not to be fabricated. |

`NOT_PRESENT` is a first-class, truthful status. It is the correct status for an
original artifact that is genuinely unavailable (see EXPOSURE_MODEL.md §6).

---

## 4. Exposure state (`exposure_state`)

Full model: [EXPOSURE_MODEL.md](EXPOSURE_MODEL.md). Vocabulary at minimum:

| Value | Meaning |
|---|---|
| `PRE_CROSS_EXPOSURE` | Frozen before either side saw the other's determination. |
| `POST_CROSS_EXPOSURE` | Created, revealed, or reconstructed after cross-exposure. |
| `NOT_APPLICABLE` | Cross-exposure is not a meaningful axis for this artifact (e.g. issuer or governance artifacts). |

A `POST_CROSS_EXPOSURE` artifact can never be relabeled `PRE_CROSS_EXPOSURE`.

**Role ↔ exposure coupling (schema-enforced):**

- `evidence_side_determination`, `consequence_side_determination` → `PRE_CROSS_EXPOSURE`
  or `POST_CROSS_EXPOSURE` (never `NOT_APPLICABLE` — they sit on the independence axis).
- `reconciliation`, `final_receipt` → `POST_CROSS_EXPOSURE` (inherently after
  cross-exposure).
- `issuer_artifact`, `governance_artifact` → `NOT_APPLICABLE`.
- `non_authoritative_summary`, `correspondence` → unconstrained.

**Not schema-enforced (governance-enforced):** timestamp ordering
(`created_at ≤ frozen_at`, reconciliation after both freezes) and the one-way
`POST → PRE` prohibition across a history cannot be expressed in JSON Schema and are
verified by governance review. Schema-valid does **not** mean chronology-verified. See
[PROVENANCE_RULES.md](PROVENANCE_RULES.md) §4.

---

## 5. Example (illustrative only — not a real determination)

```json
{
  "artifact_id": "EXAMPLE-DO-NOT-INGEST",
  "challenge_id": "challenge-01",
  "artifact_role": "issuer_artifact",
  "designated_authority": "CustodyIQ (issuer)",
  "source_author": "CustodyIQ issuing system",
  "created_at": "2026-08-12T00:00:00Z",
  "received_at": "2026-08-12T00:05:00Z",
  "frozen_at": "2026-08-12T00:10:00Z",
  "sha256": "0000000000000000000000000000000000000000000000000000000000000000",
  "version": 1,
  "status": "FROZEN",
  "exposure_state": "NOT_APPLICABLE",
  "supersedes": null,
  "source_artifact_refs": [],
  "artifact_file_ref": "challenges/challenge-01/00-issuer-artifact/EXAMPLE.txt",
  "notes": "Illustrative example. Not a real artifact."
}
```

The `sha256` shown is an all-zero **non-real placeholder**; it is not the hash of any
artifact and must never be reused. A real manifest records the SHA-256 recomputed over
the actual frozen bytes. The `artifact_id` is a sentinel (`EXAMPLE-DO-NOT-INGEST`) so this
block cannot be mistaken for a real artifact if copied.
