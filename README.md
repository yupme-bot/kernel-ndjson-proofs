# Kernel NDJSON Proofs — v1.1 and v1.2

This repository contains **deterministic NDJSON evidence exports** produced by the Guardian Kernel under two versions of the system.

The goal of these artifacts is to demonstrate:

* deterministic ordering
* tamper-evident chaining
* explicit handling of missing data
* stable verification outcomes (VALID vs PARTIAL)

This repository contains **outputs only**.
No kernel source code or internal tooling is included.

---

## What this demonstrates

Across both versions, these files show that:

* events are recorded in a deterministic order
* records are cryptographically chained
* missing data cannot be hidden
* partial data can still be verified safely
* verification results are repeatable and deterministic

---

## Contents

### Kernel v1.1 examples

* `kernel_v1_1_clean.ndjson`
* `kernel_v1_1_forced_gap.ndjson`

**Clean**

* all segments present
* deterministic ordering
* verification result: **VALID**

**Forced gap**

* one persisted segment is missing
* the exporter emits an explicit GAP marker
* export continues after the gap
* verification result: **PARTIAL**

This demonstrates that missing data is **explicitly detectable**, not silently ignored.

---

### Kernel v1.2 examples

* `kernel_v1_2_clean.ndjson`
* `kernel_v1_2_forced_gap.ndjson`

These demonstrate the same **clean vs missing-data scenario**, under the newer Kernel v1.2 persistence model.

Key characteristics of v1.2:

* persistence segments are first-class evidence units
* segments are sealed and chained independently
* missing data is represented as an **absent sealed segment range**
* GAPs are expressed structurally (via segment boundaries and gap records)
* the system transitions through degraded → recovered states
* no data is fabricated to fill gaps

**Clean**

* no gaps detected
* verification result: **PARTIAL (MISSING_SEAL)**

**Forced gap**

* one or more segment ranges are missing
* explicit gap records describe the missing ranges
* later segments remain verifiable
* verification result: **PARTIAL**

---

## Why v1.2 results are PARTIAL

The v1.2 examples in this repository are **stress artifacts**.
They intentionally omit a terminal `seal` record.

A `seal` indicates a formally closed run. When present:

* clean runs verify as **VALID**
* runs with gaps verify as **PARTIAL**

When absent:

* verification reports **PARTIAL: MISSING_SEAL**

This is expected behavior and does **not** indicate corruption or invalid data.

---

## Are these the same test?

They represent the **same scenario**, not identical mechanics.

* v1.1 demonstrates missing data at **export time**
* v1.2 demonstrates missing data at **capture / persistence time**

In both versions:

* deterministic ordering is preserved
* missing data is explicitly detectable
* verifiers can distinguish VALID vs PARTIAL outcomes

This reflects an **evolution in representation**, not a change in guarantees.

---

## Independent verification

These NDJSON files are intended to be independently verifiable.

You can verify them using the open-source NDJSON chain verifier:

[https://github.com/yupme-bot/ndjson-chain-verifier](https://github.com/yupme-bot/ndjson-chain-verifier)

### Verifying the v1.2 examples

Because the v1.2 artifacts omit a terminal seal, verification should be run in **allow-partial** mode.

Example (Node):

```bash
node --input-type=module -e "
import { verifyNdjsonStream } from './src/index.js';
const r = await verifyNdjsonStream('kernel_v1_2_clean.ndjson', { allowPartial: true });
console.log(r.status, r.reason_code, 'segments='+r.segments, 'gaps='+r.gaps);
"
```

```bash
node --input-type=module -e "
import { verifyNdjsonStream } from './src/index.js';
const r = await verifyNdjsonStream('kernel_v1_2_forced_gap.ndjson', { allowPartial: true });
console.log(r.status, r.reason_code, 'segments='+r.segments, 'gaps='+r.gaps);
"
```

Expected results:

| File                            | Expected outcome        |
| ------------------------------- | ----------------------- |
| `kernel_v1_2_clean.ndjson`      | `PARTIAL: MISSING_SEAL` |
| `kernel_v1_2_forced_gap.ndjson` | `PARTIAL` (gaps > 0)    |

---

## What this repository does NOT include

This repository does **not** include:

* kernel implementation code
* persistence or recovery logic
* fault-injection tooling
* signing, anchoring, or timestamping mechanisms
* instructions for generating new logs

Only exported evidence artifacts are provided.

---

## Intended audience

These examples are intended for:

* auditors and reviewers
* regulators and compliance teams
* engineers evaluating verification semantics

They are not intended as a usage guide for operating the Kernel.

---

## Summary

Kernel v1.1 and v1.2 both produce deterministic, tamper-evident NDJSON exports.

Kernel v1.2 improves how missing data is represented and proven,
while preserving the same verification guarantees.

These files show that evolution clearly and verifiably.
