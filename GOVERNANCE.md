# Governance

This document defines the governing authority boundaries, the three independent
controls, the AI-assistance boundary, and the protected-information rules for the
CustodyIQ Interoperability Test Protocol.

This is an **evidence-governance** document. It governs how artifacts are preserved,
identified, and advanced. It does not itself confer evidentiary or consequence
authority on any party.

---

## 1. Governing authority boundaries

Three authorities are recognized and kept strictly separate.

### 1.1 CustodyIQ / issuer evidence authority

CustodyIQ (the issuer) may establish what an operational evidence record:

- establishes
- does not establish
- contradicts
- leaves unresolved
- cannot legitimately support by inference

Evidence authority may include statements about: source provenance, timing semantics,
lineage, deterministic relationships, contradiction preservation, missing-evidence
preservation, non-inference boundaries, and as-of-time reconstruction.

> **"As-of-time reconstruction" is descriptive, not generative.** It means a statement
> about *what was knowable, or what the record established, as of a past time*. It does
> **not** authorize recreating a missing authority-bearing source artifact, does not
> yield a `PRE_CROSS_EXPOSURE` determination, and is strictly subordinate to the
> no-back-inference rule in [PROVENANCE_RULES.md](PROVENANCE_RULES.md) §4 and
> [EXPOSURE_MODEL.md](EXPOSURE_MODEL.md) §3. A genuinely absent artifact is recorded as
> `NOT_PRESENT`, never reconstructed into existence.

**Constraint.** Evidence authority is exercised *without* exposing protected
deterministic implementation logic, scoring mechanics, proprietary correlation logic,
patent-sensitive methods, credentials, production data, or customer-sensitive
information. A governed artifact states *conclusions and their provenance*, not the
protected mechanics that produced them. See §5.

### 1.2 Consequence authority

A downstream, **independent** consequence authority may decide what action is permitted
based on governed evidence, policy, authority, and consequence scope. Recognized
consequence outcomes include at minimum:

- `ALLOW`
- `HOLD`
- `ESCALATE`
- `DENY`
- `REQUEST_ADDITIONAL_EVIDENCE`

**Hard constraint.** The consequence authority must **never** rewrite, enlarge, repair,
merge, or reinterpret the evidence authority's source record. It consumes evidence; it
does not author or amend it.

### 1.3 Reconciliation (non-authoritative)

Reconciliation is **substantively non-authoritative**. It compares two independently
frozen determinations. It must not become a new source of evidence, and it must not
modify either underlying determination. See [PROVENANCE_RULES.md](PROVENANCE_RULES.md)
and §Reconciliation receipt model below.

---

## 2. The three independent controls

These three controls MUST NOT be collapsed. Satisfying one says nothing about the
others.

### Control A — Storage immutability

Proves an object cannot be silently changed after preservation. Future implementations
may include immutable / WORM cloud storage and object-lock retention.

- **Currently implemented:** none beyond Git's tamper-evident commit chain, which is
  provenance-contributing but is **NOT** WORM storage and must never be described as
  equivalent to WORM storage.
- **Future (conceptual only):** immutable object storage with retention locks. This
  repository does not provision any such infrastructure.

### Control B — Manifest / provenance identity

Establishes what the artifact is, who issued it, which challenge it belongs to, its
artifact role, its version, its hashes, its timestamps, its source references, and its
exposure state. Implemented via artifact manifests conforming to
[schemas/artifact-manifest.schema.json](schemas/artifact-manifest.schema.json) and
provenance records conforming to
[schemas/provenance-record.schema.json](schemas/provenance-record.schema.json).

### Control C — Workflow eligibility / stage gate

Determines whether an artifact is eligible to advance the protocol. Eligibility is an
**explicit determination**, recorded as a governance record — never an automatic
consequence of the other two controls.

### 2.1 Non-collapse rule (normative)

- An artifact may be immutable **and** provenance-verifiable **and still** workflow
  **ineligible**.
- Storage existence, Git presence, manifest completeness, hashing, or object locking
  must **never** automatically advance a challenge.
- Stage advancement requires an explicit eligibility determination recorded under
  Control C.

---

## 3. Controlled challenge sequence

Future challenges follow this controlled sequence. History is never rewritten to
manufacture independence.

1. Issuer artifact finalized.
2. Issuer artifact frozen.
3. SHA-256 generated.
4. Manifest / provenance identity recorded (Control B).
5. Evidence-side participant receives **only** the issuer artifact.
6. Independent evidence-side determination returned.
7. Evidence-side determination frozen and hashed.
8. Exposure state recorded as `PRE_CROSS_EXPOSURE`.
9. Consequence-side participant independently receives **only** the issuer artifact.
10. Independent consequence-side determination returned.
11. Consequence-side determination frozen and hashed.
12. Exposure state recorded as `PRE_CROSS_EXPOSURE`.
13. Only after **both** independent determinations are frozen may cross-exposure occur.
14. Reconciliation may then compare the two determinations.
15. Reconciliation must reference immutable artifact IDs and hashes.
16. Final reconciliation receipt records the outcome.
17. Later corrections become a new version or a new challenge.
18. History is **never** rewritten to manufacture independence.

A determination created or revealed **after** cross-exposure cannot retroactively become
a `PRE_CROSS_EXPOSURE` determination. Independence lost cannot be restored by re-labeling.

---

## 4. Reconciliation receipt model

A reconciliation receipt records the comparison of two independently frozen
determinations. At minimum it must be able to record one of these outcomes:

- `AGREEMENT_SAME_REASON`
- `AGREEMENT_DIFFERENT_REASON`
- `DISAGREEMENT`
- `SHARED_MISS`

It must also support recording:

- evidence relied upon
- policy / authority basis
- inference explicitly refused
- unresolved conditions
- provenance references
- source hashes
- timestamps
- exposure-state history

**Hard constraint.** A reconciliation receipt MUST NOT modify either underlying
determination. It references them by immutable artifact ID and SHA-256; it never edits
them.

---

## 5. Protected information

The following MUST NOT be placed into this repository, in any file, commit, or record:

- passwords, API keys, SSH private keys, access tokens
- environment-variable values
- cloud credentials
- production customer data
- operator PII
- protected deterministic CustodyIQ logic
- proprietary correlation implementation
- scoring mechanics
- threshold mechanics
- patent-sensitive implementation details
- private source code from another repository

The repository-specific SSH **private** key must remain outside this repository and must
never be inspected, copied, printed, or referenced by content. A `.gitignore` guards
against accidental ingestion of common secret-bearing files, but the primary control is
authoring discipline: artifacts state conclusions and provenance, not protected
mechanics.

---

## 6. AI assistance boundary

Participants may use LLMs or AI systems during analysis. **Authority is not
transferred to the model.**

- Each authority-bearing artifact must identify the designated human/system authority
  responsible for the determination (`designated_authority` in the manifest).
- AI-generated summaries, explanations, drafts, narrations, and comparisons remain
  **non-authoritative** (`artifact_role: non_authoritative_summary`) unless a designated
  authority explicitly **adopts** them through the governed artifact process. Adoption is
  itself a governed act, recorded in `governance-records/` and reflected by a new
  authority-bearing artifact whose `designated_authority` is the adopting human/system.
- An AI-produced draft must **never** silently become an authority-bearing source
  artifact. Silent substitution of source is a provenance defect and is grounds for a
  STOP.

**Adoption never launders exposure state or manufactures a gate-satisfying original.**
An adopted artifact is a **new** artifact dated at adoption time. It inherits the
worst-case (latest) exposure state of the content it is derived from: if that content
post-dates cross-exposure, the adopted artifact is `POST_CROSS_EXPOSURE` and can **never**
be `PRE_CROSS_EXPOSURE`. Adoption is therefore **never** a route to back-infer an absent
original, and an adopted artifact can **never** satisfy an eligibility gate that requires
an original, pre-cross-exposure independent determination (see
[EXPOSURE_MODEL.md](EXPOSURE_MODEL.md) §5). `adopted_by` recorded on a
`non_authoritative_summary` manifest is informational only and does not elevate that
artifact's role; elevation happens only by issuing a new authority-bearing artifact and a
companion adoption governance record.

---

## 7. Governance records

`governance-records/` holds governance decisions that are not themselves evidence
artifacts: eligibility determinations (Control C), adoption acts (§6), STOP records, and
scope decisions. Governance records are dated, attributed to a designated authority, and
never rewritten; corrections are appended as new records.

---

## 8. Amendment

This governance document is versioned through Git history. Substantive changes are made
as new commits with clear messages and, where they change how authority or eligibility
works, an accompanying governance record. Governance changes never retroactively alter
the truthful status of an existing challenge.
