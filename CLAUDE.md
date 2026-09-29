# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Data-only repository: sample transactions, patients, and payload content for DirectTrust Metadata and Payloads Framework use cases. There is no build, lint, or test tooling. The files are consumed by external tooling (e.g. the accreditation testing tools) and must stay consistent with the published implementation guides, e.g. https://directtrust.github.io/cc-maternal-health-birth/.

## Structure

`useCases/<useCaseName>/`
- `useCase.json` — defines the use case's transactions (name, `metadataTypeCode`, `purpose`, `payloadContainerTypes`, and the list of `payloadContents` with `mimeType` and `required`).
- `patients/<patientName>/patient.json` — one test patient: `patientId`, `demographics` (`firstName`, `lastName`, `fullName`, `dob`, `gender`), and the transactions that patient participates in. Each transaction's `payloadContents` entries map a payload `name` to a `contentFile`.
- Content files (`tx1-notif.txt`, `tx1-human-readable.txt`, `tx1-human-readable.html`, ...) live in the same directory as the patient's `patient.json`.

## Conventions

- Payload `name` values in `patient.json` must match the names in `useCase.json` for the same transaction. Not every patient has every transaction; `contentFile` names follow `tx<N>-...` (tx1 outpatient visit, tx2 admission, tx3 birth, tx4 discharge, tx5 documents, tx8 workflow status).
- Content must agree with `patient.json`: the HL7 v2 PID uses `patientId` (PID-3), the name, DOB, and gender (HL7 Table 0001 code, e.g. `F`).
- Each transaction's human-readable payload is a `.txt` plus an optional `.html` with the same content; HL7 v2 payloads are `.txt` (`text/hl7v2`), CDA payloads are `.xml`.
- The tx1/tx2/tx4 HL7 v2 notifications are ADT messages (tx1 is ADT^A03 for an outpatient visit) and must carry a pregnancy episode context identifier with its assigning authority.
- Identifiers, OIDs, addresses, and NPIs in sample content are made-up placeholders.
