# CustodyIQ Interoperability Test Protocol

An evidence-governance and provenance repository for interoperability experiments
involving CustodyIQ and independent external systems or reviewers.

This repository preserves **governed evidence artifacts** so that the record of an
interoperability experiment remains trustworthy, auditable, and honest about its own
limitations. Its purpose is **not** to make experiments look successful. Its purpose is
to make the evidence record trustworthy.

---

## 1. What this repository is

This repository preserves and governs, at minimum:

- issuer artifacts
- independent evidence-side determinations
- independent consequence-side determinations
- reconciliation artifacts
- final reconciliation receipts
- provenance records
- exposure-state tracking
- challenge-stage eligibility records
- artifact hashes
- tamper-evident historical references (Git commit chain — **not** WORM storage; see [PROVENANCE_RULES.md](PROVENANCE_RULES.md) §1)
- governance records

## 2. What this repository is NOT

This repository is **not**:

- a CustodyIQ product-code repository
- a freight-platform implementation repository
- an AI-agent implementation repository
- an implementation repository for any participating system
- a substitute for source-system evidence
- a mechanism for manufacturing missing provenance
- authority to rewrite independent determinations

No product source code, proprietary deterministic logic, scoring/threshold mechanics,
patent-sensitive methods, credentials, production data, or customer-sensitive information
belongs here. See [GOVERNANCE.md](GOVERNANCE.md) §Protected Information.

## 3. The three independent controls

This repository keeps three controls **explicitly separate**. They must never be
collapsed into one another.

| Control | Question it answers | What it does **not** prove |
|---|---|---|
| **A — Storage immutability** | Can this object be silently changed after preservation? | It does not prove the artifact is workflow-eligible. |
| **B — Manifest / provenance identity** | What is this artifact, who issued it, and how is it identified? | It does not prove immutability, and it does not prove eligibility. |
| **C — Workflow eligibility / stage gate** | Is this artifact eligible to advance the protocol? | It does not prove or create immutability or identity. |

An artifact may be **immutable** and **provenance-verifiable** and still be **workflow
INELIGIBLE**. Conversely, storage existence, Git presence, manifest completeness,
hashing, or object locking must **never** automatically advance a challenge. Stage
advancement requires an explicit eligibility determination. See
[EXPOSURE_MODEL.md](EXPOSURE_MODEL.md) and
[GOVERNANCE.md](GOVERNANCE.md).

> **Git history is not WORM storage.** Git commit history contributes provenance and a
> tamper-evident chain, but it is not equivalent to immutable / write-once-read-many
> (WORM) storage. This repository makes no claim of cryptographic immutability beyond
> what is actually implemented (SHA-256 content hashes recorded in manifests, and Git's
> tamper-evident commit chain). See [PROVENANCE_RULES.md](PROVENANCE_RULES.md).

## 4. Authority boundaries

- **CustodyIQ / issuer evidence authority** may establish what an operational evidence
  record establishes, does not establish, contradicts, leaves unresolved, or cannot
  legitimately support by inference — without exposing protected implementation logic.
- **Consequence authority** (a downstream, independent decision authority) may decide
  what action is permitted (e.g. `ALLOW`, `HOLD`, `ESCALATE`, `DENY`,
  `REQUEST_ADDITIONAL_EVIDENCE`) based on governed evidence, policy, authority, and
  consequence scope. It must never rewrite, enlarge, repair, merge, or reinterpret the
  evidence authority's source record.
- **Reconciliation is substantively non-authoritative.** It compares independently
  frozen determinations. It must not become a new source of evidence.

Full detail: [GOVERNANCE.md](GOVERNANCE.md).

## 5. AI assistance boundary

Participants may use LLMs or AI systems during analysis. That does **not** transfer
authority to the model. Each authority-bearing artifact must identify the designated
human/system authority responsible for the determination. AI-generated summaries,
explanations, drafts, narrations, and comparisons remain **non-authoritative** unless a
designated authority explicitly adopts them through the governed artifact process. See
[GOVERNANCE.md](GOVERNANCE.md) §AI Assistance Boundary.

## 6. Repository layout

```
README.md                          This file
GOVERNANCE.md                      Authority boundaries, controls, AI boundary, security
PROVENANCE_RULES.md                Hashing, freezing, immutability claims, chronology
ARTIFACT_SCHEMA.md                 Human-readable manifest field reference
EXPOSURE_MODEL.md                  Exposure-state vocabulary and eligibility gates

challenges/
    _template/                     Stage folders for one challenge (copy per challenge)
        README.md
        00-issuer-artifact/
        01-evidence-side/
        02-consequence-side/
        03-reconciliation/
        04-final-receipt/

schemas/
    artifact-manifest.schema.json  JSON Schema for artifact manifests
    provenance-record.schema.json  JSON Schema for provenance records

```

## 7. Status of this publication

This is the **protocol only**: the governance rules, exposure model, provenance rules,
manifest field reference, JSON schemas and an empty challenge template. No challenge
records, participant determinations or reconciliation material from CustodyIQ's own
interoperability experiments are included.

> **Substantively valuable ≠ formally eligible.** A record can be useful and still fail
> the eligibility gate. That distinction is preserved deliberately and must not be
> "cleaned up" to make a record look complete.

## Acknowledgments

Reviewed informally before release by Ricky Jones of AlvianTech, with his AI review instruments Lala and Auntie Trinity. Their suggestions (neutral lodging of hash commitments, recording shared upstream influences, a defined reference for SHARED_MISS, machine-readable reconciliation and eligibility records, and a plain-English walkthrough with a synthetic example) are offered for anyone building on this protocol. This was not an audit or certification.
