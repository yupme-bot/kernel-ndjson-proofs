Kernel v1.1 — Historical NDJSON Proof Artifacts


This repository contains two NDJSON exports produced by Kernel v1.1.

Files

kernel-v1.1-clean.ndjson
A normal run demonstrating deterministic ordering with no gaps.

kernel-v1.1-forced-gap.ndjson
A run where one persisted segment was intentionally removed before export.
The kernel emits an explicit gap record and continues exporting without crashing or silently truncating data.

What this demonstrates

Deterministic event ordering

Segmented capture and export

Explicit, first-class gap markers (no silent data loss)

Successful continuation after missing data

These files are intended to be audited directly as independent evidence.


Live Demonstration (Non-Authoritative)

A live, browser-based demonstration of the deterministic event kernel is available here:
https://deterministic-stream-demo1.pages.dev/

This demo visualizes event capture, stalls, gaps, and recovery in real time.

Important: The demo is illustrative only.
The authoritative proof artifacts are the NDJSON files in this repository, which can be independently verified offline.

Runs in the browser using the deterministic kernel, with persistence via IndexedDB and real-time visualization. No server-side mutation of events.
