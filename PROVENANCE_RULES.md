# Provenance Rules

This document defines how artifacts are hashed, frozen, versioned, and chronologically
ordered, and what immutability claims this repository does and does not make.

---

## 1. Honest immutability claims

This repository makes **exactly** these immutability-related claims, and no more:

1. **Content hashing.** Every frozen artifact records a SHA-256 hash of its exact bytes
   in its manifest and provenance record. Any later change to those bytes produces a
   different hash, making silent content substitution detectable **provided the recorded
   hash itself is trusted**.
2. **Tamper-evident Git chain.** Git commit history provides a tamper-evident,
   append-oriented chain of changes. Rewriting it is detectable to anyone holding a prior
   reference (commit SHA).

This repository does **NOT** claim:

- that Git provides WORM (write-once-read-many) or object-lock storage — **it does not**;
- cryptographic immutability beyond the SHA-256 content hashes actually recorded and
  Git's commit chain;
- that the mere presence of a file, a hash, or a commit renders an artifact immutable in
  the Control-A sense.

**Control A (storage immutability) is presently unimplemented** beyond the Git chain.
Immutable / WORM cloud storage may be designed and documented conceptually in future,
but no such infrastructure is provisioned by this repository. See
[GOVERNANCE.md](GOVERNANCE.md) §2.

---

## 2. Hashing

- **Algorithm:** SHA-256 over the exact byte content of the frozen artifact file.
- **Encoding of hash:** lowercase hexadecimal, 64 characters, matching
  `^[a-f0-9]{64}$`.
- **What is hashed:** the artifact file exactly as frozen. If an artifact is a directory
  or bundle, a deterministic manifest-of-hashes is produced and that manifest is itself
  hashed; the method is recorded in the provenance record.
- **When:** at freeze time (see §3). The hash is recorded in both the artifact manifest
  (`sha256`) and the provenance record.
- **Absent artifacts:** if the byte content does not exist (e.g. an original
  determination that was never captured), `sha256` is `null` and `status` is
  `NOT_PRESENT`. A hash is **never** fabricated for absent content.

> **Line-ending caution.** This repository is used on Windows, where Git may be
> configured with `core.autocrlf`. SHA-256 hashes must be computed over the artifact's
> canonical bytes as frozen and stored, independent of working-tree line-ending
> conversion. Prefer hashing binary-safe, or normalize and record the normalization in
> the provenance record, so a hash is reproducible regardless of checkout platform.

---

## 3. Freezing

An artifact is **frozen** when its authoring authority declares it final and it is
hashed and manifested. After freezing:

- the artifact's bytes are not edited in place; a correction is a **new version** or a
  **new challenge** (see §5);
- `frozen_at` is recorded (RFC 3339 / ISO 8601 date-time, with timezone);
- `sha256` is recorded;
- the manifest `status` becomes `FROZEN`.

Freezing is a provenance act, not a workflow-eligibility act. A frozen artifact is not
thereby eligible to advance the protocol (Control C is separate). See
[EXPOSURE_MODEL.md](EXPOSURE_MODEL.md).

---

## 4. Chronology and exposure ordering

Chronology is load-bearing for independence:

- `created_at` ≤ `frozen_at` for any artifact that records both.
- An evidence-side or consequence-side determination must be **frozen** and recorded as
  `PRE_CROSS_EXPOSURE` **before** cross-exposure occurs. See
  [EXPOSURE_MODEL.md](EXPOSURE_MODEL.md).
- Reconciliation timestamps must be **after** both determinations are frozen.
- A determination first captured, revealed, or reconstructed **after** cross-exposure is
  `POST_CROSS_EXPOSURE` and can never be relabeled `PRE_CROSS_EXPOSURE`.

Backdating, back-inference of an authority-bearing source artifact from later material,
and reconstruction of a "would-have-been" determination are all prohibited. If an
original artifact is unavailable, it is recorded as `NOT_PRESENT` — not reconstructed.

> **Ordering is governance-enforced, not schema-enforced.** JSON Schema cannot compare
> two date-time values, so `created_at ≤ frozen_at`, "reconciliation after both freezes,"
> and the one-way `POST_CROSS_EXPOSURE → PRE_CROSS_EXPOSURE` prohibition across an
> `exposure_state_history` are verified by governance review (and any future validator
> tooling), **not** by the schemas. A schema-valid manifest or provenance record is not,
> by that fact alone, chronology-verified.

---

## 5. Versioning and supersession

- Corrections and revisions are captured as a **new version** (increment `version`) or a
  **new challenge**, never as an in-place edit of a frozen artifact.
- A superseding artifact records `supersedes` = the artifact_id (and ideally version) it
  replaces. The superseded artifact's manifest `status` becomes `SUPERSEDED`; its bytes
  and hash are retained unchanged.
- Supersession never deletes or rewrites history. The full chain remains inspectable.

---

## 6. Provenance records

Each frozen artifact has a provenance record conforming to
[schemas/provenance-record.schema.json](schemas/provenance-record.schema.json). It binds
together: the artifact ID, its SHA-256, its source references, its freeze event, its
exposure-state history, and an honest statement of which storage controls actually apply
(Control A/B/C). Provenance records are append-only in practice: a correction is a new
record that references the prior one.

---

## 7. Source identity and non-substitution

- Every authority-bearing artifact identifies its `designated_authority` and
  `source_author`.
- The `source_author` may have used AI assistance; that does not make the AI the
  authority (see [GOVERNANCE.md](GOVERNANCE.md) §6).
- **No silent source substitution.** The artifact whose hash is recorded is the artifact
  that was actually produced by the identified authority. Swapping in a different
  document (e.g. a later summary, an AI narration, or a reconciliation-derived artifact)
  under an earlier identity is a provenance defect and grounds for a STOP.
