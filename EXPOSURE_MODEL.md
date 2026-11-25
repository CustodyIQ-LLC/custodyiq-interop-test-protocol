# Exposure Model and Stage Gate

This document defines the exposure-state vocabulary and how it interacts with workflow
eligibility (Control C). Exposure state is central to the independence guarantee of an
interoperability experiment.

---

## 1. Why exposure state exists

The value of comparing an evidence-side determination with a consequence-side
determination depends on the two having been produced **independently** — each from the
issuer artifact alone, neither having seen the other. "Cross-exposure" is the moment
either side is exposed to the other's determination (directly, or through reconciliation
material derived from it).

Exposure state records, truthfully, where an artifact sits relative to that moment. It is
recorded; it is never inferred favorably.

---

## 2. Vocabulary

| State | Meaning |
|---|---|
| `PRE_CROSS_EXPOSURE` | The artifact was frozen and recorded **before** either side was exposed to the other's determination. |
| `POST_CROSS_EXPOSURE` | The artifact was created, revealed, or reconstructed **after** cross-exposure. |
| `NOT_APPLICABLE` | Cross-exposure is not a meaningful axis for this artifact (issuer artifacts, governance artifacts, correspondence). |

`PRE_CROSS_EXPOSURE` and `POST_CROSS_EXPOSURE` are the two required states. `NOT_APPLICABLE`
is provided so that artifacts to which the axis does not apply are not mislabeled.

---

## 3. Transition rules (normative)

1. **One-way.** An artifact recorded `POST_CROSS_EXPOSURE` can **never** be relabeled
   `PRE_CROSS_EXPOSURE`. Independence, once lost, cannot be restored by re-labeling.
2. **Freeze-before-exposure.** For an evidence-side or consequence-side determination to
   be `PRE_CROSS_EXPOSURE`, it must have been **frozen and hashed** before cross-exposure
   occurred. A determination not frozen before exposure is `POST_CROSS_EXPOSURE`.
3. **No back-inference.** A determination that was not captured before cross-exposure is
   not reconstructed and labeled `PRE_CROSS_EXPOSURE`. If it is genuinely unavailable, the
   original is recorded as `NOT_PRESENT` (see [ARTIFACT_SCHEMA.md](ARTIFACT_SCHEMA.md) §3).
4. **Exposure history is preserved.** The provenance record keeps the full
   `exposure_state_history` with timestamps and the party who recorded each state.

---

## 4. Ordering with the controlled sequence

Mapping to the controlled sequence in [GOVERNANCE.md](GOVERNANCE.md) §3:

- Steps 7–8 and 11–12: each determination is frozen, hashed, and recorded
  `PRE_CROSS_EXPOSURE`.
- Step 13: cross-exposure may occur **only after both** determinations are frozen.
- Steps 14–16: reconciliation and the final receipt are inherently `POST_CROSS_EXPOSURE`
  and are non-authoritative (they compare; they do not add evidence).

---

## 5. The stage gate (Control C) is separate from exposure state

Exposure state is a **provenance fact**. Workflow eligibility is a **separate,
explicit determination**. Recording an artifact `PRE_CROSS_EXPOSURE` does not advance a
challenge, and a challenge is not advanced merely because artifacts exist, are hashed, or
are committed to Git.

Stage advancement requires a governance record (Control C) that explicitly finds the
required artifacts present, provenance-verifiable, and eligible. Absent that explicit
finding, the challenge remains **gated**.

### 5.1 Eligibility for the evidence→consequence→reconciliation protocol

At minimum, to advance past the evidence stage a challenge requires:

- a frozen, provenance-verifiable **issuer artifact**; and
- a frozen, provenance-verifiable, **`PRE_CROSS_EXPOSURE`** evidence-side determination.

If the original evidence-side determination is `NOT_PRESENT`, the challenge is **not
eligible** to be treated as protocol-complete, regardless of how substantively valuable
any later (post-exposure) material may be.

> **Substantively valuable ≠ formally eligible.** A later reconciliation record can be
> genuinely useful and still fail the eligibility gate because it is not a
> pre-cross-exposure independent determination. This distinction is preserved
> deliberately.

---

## 6. Worked case: a missing original

Suppose a challenge has no provenance-verifiable original evidence-side determination
frozen **`PRE_CROSS_EXPOSURE`**, and a reviewer later produces a careful reconciliation
after seeing both sides. That reconciliation is `POST_CROSS_EXPOSURE` and
non-authoritative. However useful it is, it does not satisfy the gate: the challenge
stays **GATED**, and the missing original is recorded as `NOT_PRESENT`, never
reconstructed.
