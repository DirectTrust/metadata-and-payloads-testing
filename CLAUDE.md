# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Data-only repository: sample transactions, patients, and payload content for DirectTrust Metadata and Payloads Framework use cases. There is no build, lint, or test tooling. The files are consumed by external tooling (e.g. the accreditation testing tools) and must stay consistent with the published implementation guides, e.g. https://directtrust.github.io/cc-maternal-health-birth/.

## Structure

`useCases/<useCaseName>/`
- `useCase.json` — defines the use case's transactions (name, `metadataTypeCode`, `purpose`, `payloadContainerTypes`, and the list of `payloadContents` with `mimeType` and `required`). See "XD submission-set and document metadata" below for the `metadata` field.
- `patients/<patientName>/patient.json` — one test patient: `patientId`, `demographics` (`firstName`, `lastName`, `fullName`, `dob`, `gender`), and the transactions that patient participates in. Each transaction's `payloadContents` entries map a payload `name` to a `contentFile`.
- Content files (`tx1-notif.txt`, `tx1-human-readable.txt`, `tx1-human-readable.html`, ...) live in the same directory as the patient's `patient.json`.

## XD submission-set and document metadata

Each transaction in `useCase.json` may carry a `metadata` object keyed by payload container type (only `XD` exists today — other container types would get their own sibling key, e.g. `metadata.FHIR`, if/when added):

- **Transaction level** (`transaction.metadata.XD`) — the static portion of the IHE XDS SubmissionSet for that transaction: `availabilityStatus` (fixed), `contentTypeCode`/`purpose` (the canonical transaction code, per the Metadata and Payload Framework's SubmissionSet.purpose extension — SHALL match the transaction `name`), `title`, and `referenceIdContextTypes` (the Context Instance identifier types the transaction carries, e.g. `pregnancy`, `visit`). TX8 (`send-workflow-status`) uses `referenceIdCorrelation: "originatingSubmissionSetUniqueId"` instead, since its referenceIdList is the initiating transaction's `SubmissionSet.uniqueId`, not a static context type.
- **Document level** (`payloadContents[].metadata.XD`) — the static portion of that item's XDS DocumentEntry: `classCode` (always equal to the transaction's `contentTypeCode`), and, where the implementation guide defines one, `typeCode`/`typeCodeDisplay`/`typeCodeSystem`.
- **Deliberately excluded**: anything the implementation guide marks as runtime/instance data rather than a fixed use-case value — `uniqueId`, `entryUUID`, `submissionTime`, `intendedRecipient`, `patientId`, `sourcePatientId`/`sourcePatientInfo`, `author`, `hash`, `size` — these come from the actual sender/recipient/patient/content at test-run time, not from this repo. `formatCode`, `confidentialityCode`, `healthcareFacilityTypeCode`, and `practiceSettingCode` are also omitted: the guide leaves `formatCode` "TBD" for every document type used here, and the other three are guide text that says to populate from the sending HISP's own facility/event data, not a fixed use-case value — a consuming builder should fall back to synthetic/configured defaults for those.
- **Source and confidence**: values come from https://directtrust.github.io/cc-maternal-health-birth/tx1-... through .../tx8-send-workflow-status. Where the guide gives an explicit value (e.g. TX1's `98145-6` LOINC typeCode, TX8's FHIR `Task` typeCode) it's used verbatim. Where the guide only shows the pattern for one transaction (e.g. TX4 explicitly ties `56444-3` to the human-readable DocumentEntry) it's extrapolated to the structurally identical human-readable/HTML items in the other transactions — a reasonable best guess, not a guide-confirmed value for those specific transactions. TX3's CDA `typeCode` (`DT-Birth`) is explicitly called out by the guide as temporary, pending LOINC assignment. TX5's document has no `typeCode` at all in the guide, so none is set. Titles for TX2/TX4/TX5 replace unfilled template strings in the guide's own tables with constructed display names.

## Conventions

- Payload `name` values in `patient.json` must match the names in `useCase.json` for the same transaction. Not every patient has every transaction; `contentFile` names follow `tx<N>-...` (tx1 outpatient visit, tx2 admission, tx3 birth, tx4 discharge, tx5 documents, tx8 workflow status).
- Content must agree with `patient.json`: the HL7 v2 PID uses `patientId` (PID-3), the name, DOB, and gender (HL7 Table 0001 code, e.g. `F`).
- Each transaction's human-readable payload is a `.txt` plus an optional `.html` with the same content; HL7 v2 payloads are `.txt` (`text/hl7v2`), CDA payloads are `.xml`.
- The tx1/tx2/tx4 HL7 v2 notifications are ADT messages (tx1 is ADT^A03 for an outpatient visit) and must carry a pregnancy episode context identifier with its assigning authority.
- Identifiers, OIDs, addresses, and NPIs in sample content are made-up placeholders.
