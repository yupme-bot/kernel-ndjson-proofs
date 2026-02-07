# Kernel NDJSON Proofs

**Deterministic audit-log export artifacts with explicit gap semantics**

---

## What this is (please read first)

**NDJSON-GAP is NOT a complete audit system.**

It is a **narrow, single-purpose component** designed to solve one specific problem:

> **Preventing silent data loss in audit log exports by making absence explicit and verifiable.**

This repository contains **example NDJSON artifacts and a verifier** that demonstrate how audit logs can be exported such that missing data cannot be silently erased, ignored, or disguised as failure.

---

## What problem this addresses

In real systems, audit logs are often:

* truncated due to storage failures
* partially exported due to retention limits
* incomplete due to crashes or outages
* transferred across trust boundaries

Most audit verification systems treat missing data as a binary failure:

* one missing segment → entire log is unusable

NDJSON-GAP explores an alternative:

> **Incomplete audit artifacts can still carry useful, verifiable evidence if absence is made explicit.**

**Example scenario:**

> A company’s audit logs span two years. Due to a storage failure in month eight, three days of logs are missing.
> Traditional verifiers reject the entire two-year chain as INVALID.
> NDJSON-GAP marks the three-day gap explicitly and verifies the remaining history as PARTIAL, preserving evidence while documenting absence.

---

## What this does

This format + verifier provides:

* Deterministic ordering verification
* Cryptographic hash chaining (integrity only, not authenticity)
* Explicit representation of missing data ("gaps")
* Stable verification outcomes:

  * **VALID** — integrity and completeness hold
  * **PARTIAL** — integrity holds, completeness does not
  * **INVALID** — integrity is broken

**What “explicit absence” means:**
Missing data is represented by GAP records in the chain itself, not silently omitted or implied by failed verification.

**Key idea:**
Integrity and completeness are separate properties and should be evaluated independently.

---

## What this does NOT do (non-goals)

This project intentionally does **not** provide:

* ❌ Trusted timestamps or temporal anchoring
* ❌ Digital signatures or identity/authenticity guarantees
* ❌ Access control or key management
* ❌ Event capture SDKs or log collection agents
* ❌ Real-time streaming, analytics, or dashboards
* ❌ Compliance certification (SOC 2, HIPAA, PCI-DSS, etc.)

These concerns are **explicitly out of scope** and are expected to be handled by external systems.

---

## How this fits into a real workflow

NDJSON-GAP is designed to **compose with existing audit infrastructure**, not replace it.

Example composition:

```
[Log Source]
      ↓
[NDJSON-GAP Export]
      ↓
[External Signature / PKI]
      ↓
[External Timestamp (RFC 3161, etc.)]
      ↓
[Archive / Transfer]
      ↓
[Auditor]
      ↓
[Verify signature → verify timestamp → verify NDJSON-GAP]
```

This layer answers **one question only**:

> **“Do these bytes truthfully represent what data is present and absent?”**

---

## Why “PARTIAL” exists

Traditional audit verification collapses to **all-or-nothing**:

* any missing data → total failure

NDJSON-GAP introduces **PARTIAL** as a first-class, stable outcome:

* Integrity is preserved
* Absence is explicit and enumerable
* Auditors can reason about *what is missing* rather than losing all signal

Whether PARTIAL is acceptable is a **policy decision**, not a technical one.

---

## What’s in this repository

This repository intentionally contains **artifacts only**:

* Clean audit exports
* Forced-gap exports (intentional missing data)
* Verifier output demonstrating VALID / PARTIAL / INVALID states

No capture code, production kernel, or ingestion tooling is included here.

---

## Verification

Artifacts can be independently verified using the open-source verifier:

**Verifier:** [https://github.com/yupme-bot/ndjson-chain-verifier](https://github.com/yupme-bot/ndjson-chain-verifier)

Verification checks:

* record ordering
* hash chain integrity
* gap representation
* terminal seal presence (if applicable)

Verification is deterministic: the same input always yields the same result.

---

## Scope note (important)

**Note on Scope — Feb 2026**

Earlier versions of this repository were presented more broadly than warranted.
Based on community feedback, this project has been explicitly refocused as:

> **A gap-aware audit export format and verifier — not a complete audit system.**

This change reflects improved clarity of intent, not a change in technical guarantees.

---

## When to use this

Use this approach when:

* ✓ Audit logs must cross organizational trust boundaries
* ✓ Long-term archives may experience partial data loss
* ✓ You need to verify what data is present vs absent
* ✓ Partial evidence is preferable to total loss of auditability

Do **not** use this approach when:

* ✗ You need real-time streaming or analytics
* ✗ Complete audit trails are legally required (check regulations)
* ✗ You need a turnkey compliance solution

---

## Common Questions

**Q: Does this prove the missing data never existed?**
A: No. It proves absence is explicit and verifiable, not whether the data should have existed.

**Q: Can this be used for compliance (SOC 2, HIPAA, PCI, etc.)?**
A: This is a component, not a compliance solution. Consult your auditors.

**Q: Why not use Certificate Transparency, Trillian, or similar systems?**
A: Those systems optimize for different problems (inclusion proofs, distribution). This work focuses specifically on explicit absence semantics.

---

## Intended audience

This project is intended for:

* auditors and reviewers
* compliance and security engineers
* infrastructure engineers
* researchers examining audit failure modes

It is **not** intended as an end-user logging or observability product.

---

## Summary

NDJSON-GAP explores one narrow idea:

> **In audit contexts, explicit and verifiable absence is often more useful than silent failure or total invalidation.**

Everything else is intentionally delegated to composable, external systems.
