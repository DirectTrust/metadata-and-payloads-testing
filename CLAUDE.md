# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Data-only repository: sample use cases, patients, and payload content for DirectTrust Metadata and Payloads Framework use cases. There is no build, lint, or test tooling. The files are cloned and consumed by the accreditation testing tools (`accreditation-testing-backend`, which pulls this repository at startup, hourly, and on an admin's request) and must stay consistent with the published implementation guides, e.g. https://directtrust.github.io/cc-maternal-health-birth/.

**README.md is the reference for the data format** (`useCase.json`, `patient.json`, scenarios, per-document XD metadata, and the patient-over-use-case precedence rule). Keep it in sync with any schema change here and in the backend's `MetaAndPayloadUseCase` records.

## Structure

`useCases/<useCaseName>/`
- `useCase.json` — use case name, `contextIdentifierTypes`, and `transactions` (each with `metadataTypeCode`, `purpose`, `payloadContainerTypes`, transaction-level `metadata.XD`, and `payloadContents` with `mimeType`, `required` and document-level `metadata.XD`).
- `patients/<patientDir>/patient.json` — one test patient: `patientId`, `demographics`, and `scenarios` (each a `name` plus its own `transactions`; each transaction's `payloadContents` maps a document `name` to a `contentFile`, optionally with `mimeType` and `metadata.XD`).
- Content files live in the same directory as the patient's `patient.json`.

## Rules to keep when editing

- **The guide is the standard, and only TX1 is authoritative right now.** The other transactions in `useCase.json` are being revised; do not "fix" TX1 to match them, and do not present their codes as confirmed.
- The format is *omit, don't default*: leave a code out rather than inventing a placeholder. Class/type/format/confidentiality codes classify the document (e.g. the CDA's LOINC `code`), not the transaction.
- A patient's `metadata.XD` for a document overrides the use case's same-named fields for that patient; for a document the use case doesn't define it is the only source and the patient must give the document's `mimeType`. CDA is `application/cda+xml` (the IANA-registered type), not `application/xml+cda`.
- Payload `name` values in `patient.json` match the use case's names for the same transaction (patient-only documents excepted). A patient is always required for a message; there is no patient-optional flag.
- Content must agree with `patient.json`: HL7 v2 `PID-3` uses `patientId`, name, DOB and gender (HL7 Table 0001 code, e.g. `F`); a CDA's `recordTarget` agrees likewise.
- Human-readable content is `.txt` (`text/plain`) with an optional `.html` twin; HL7 v2 is `.txt` (`text/hl7v2`); CDA is `.xml`.
- **Sender/recipient addresses in content are placeholders**: write `{{sender.address}}` / `{{recipient.address}}` instead of the author's example mailboxes (the sending organization or author, the receiving mailbox) when bringing in new files; the backend swaps in the tool's real addresses at send time and rejects any other `{{...}}` marker. Likewise `{{now}}` / `{{now.hl7}}` for a Task's `authoredOn`-style "created now" time (never clinical event times). Leave unrelated addresses (patient email) alone. See README Conventions.
- tx1/tx2/tx4 HL7 v2 notifications are ADT messages (tx1 is ADT^A03) and carry the pregnancy episode context identifier with its assigning authority. Identifiers, OIDs, addresses, and NPIs in sample content are made-up placeholders.
- Changing the shape of either JSON file requires the backend to understand it first (the backend has no fallback for old shapes), so coordinate the two.
