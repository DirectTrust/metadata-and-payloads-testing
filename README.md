# metadata-and-payloads-testing

Sample use cases, patients, and payload content for the DirectTrust Metadata and Payloads Framework. This is a data-only repository (no build or tests). The DirectTrust accreditation testing tools clone it and use it to build and send framework-compliant Direct messages, so the files must stay consistent with the published implementation guides, e.g. <https://directtrust.github.io/cc-maternal-health-birth/>.

The backend (`accreditation-testing-backend`) clones this repository at startup, pulls it again every hour, and an administrator can pull on demand from the **Refresh Use Cases** button of the Metadata and Payload test. Changes pushed here show up without a redeploy, but the backend must understand the data format below.

## Layout

```
useCases/<useCaseName>/
  useCase.json                      the use case and its transactions
  patients/<patientDir>/
    patient.json                    one test patient, their scenarios, and the documents they have
    <content files>                 the patient's payload content (CDA, HL7 v2, FHIR, text, ...)
```

## `useCase.json`

| Field | Description |
|---|---|
| `useCaseName`, `useCaseDisplayName` | Machine name (matches the directory) and display name |
| `contextIdentifierTypes` | Map from a context short name (`perinatal`, `visit`, ...) to the identifier type URN used in `referenceIdList` |
| `transactions[]` | The use case's transactions (below) |

Each transaction:

| Field | Description |
|---|---|
| `name`, `description` | Transaction code (e.g. `send-documents`) and description |
| `metadataTypeCode`, `purpose` | Sent as `x-direct-metadataTypeCode` and `x-direct-purposeOfUse` |
| `payloadContainerTypes` | Container type; only `XD` is supported |
| `createHumanReadableContent` | Optional, default `false`. When `true` and the selected documents include no human-readable text, the builder creates one (a summary of the package metadata: patient, source and recipient, context, and each document's classification and identifier) as plain text and HTML. The message's first part is then `multipart/alternative`, and both versions are added to the XDM as documents, identical to the message parts. When `false` or absent, a transaction must be given authored human-readable text or message creation fails. `send-documents` sets it, since its documents are all patient-defined CDAs |
| `metadata.XD` | The static part of the XDS SubmissionSet (below) |
| `payloadContents[]` | The documents the transaction can carry: `name`, `required`, `mimeType`, and `metadata.XD` (below). A transaction whose documents are all defined per patient (e.g. `send-documents`) has none required |

**SubmissionSet metadata** (`transaction.metadata.XD`): `availabilityStatus`, `purpose` with `purposeDisplay`/`purposeSystem` (sent as `code^display^system`), `title`, `referenceIdContextTypes` (context names from `contextIdentifierTypes`, or identifier type URNs, the transaction carries), `referenceIdCorrelation` (TX8 only). `contentTypeCode` (+`Display`/`System`) is only needed when it is *not* the `typeCode` of the transaction's principal document, which is the default.

**DocumentEntry metadata** (`payloadContents[].metadata.XD`, also allowed on a patient's document): `title`, `principal` (this document's `typeCode` is the submission set's content type), and `classCode`, `typeCode`, `formatCode`, `confidentialityCode`, each with `...Display` and `...System`. A code that is not listed is left out of the message, never defaulted.

## `patient.json`

```json
{
  "patientId": "NTHS-00003007",
  "demographics": { "firstName": "Mimi", "lastName": "Shell", "fullName": "Mimi Shell", "dob": "1997-08-10", "gender": "F" },
  "scenarios": [
    {
      "name": "Provide Notification of Pregnancy and Risk Factors",
      "transactions": [
        {
          "name": "send-documents",
          "payloadContents": [
            { "name": "Pregnancy Summary Notification", "contentFile": "PregnancySummaryNotification.xml",
              "mimeType": "application/cda+xml", "metadata": { "XD": { "principal": true, "typeCode": "90767-5", "...": "..." } } }
          ]
        }
      ]
    }
  ]
}
```

- A patient has one or more **scenarios**; the testing UI offers the distinct scenario names across a use case's patients, then only the patients that have the chosen scenario. Each scenario lists the **transactions** the patient participates in and, per transaction, the documents (`payloadContents`) the patient has, each mapped to a `contentFile` in the patient's directory.
- A patient is always required: a metadata and payload message is always sent in the context of a patient.
- A patient's document **matches** a use case document by `name`. `mimeType` is inherited from the use case document when omitted; a document the use case does not define must give its own `mimeType` (CDA is `application/cda+xml`, the type HL7 registered with IANA).
- A patient's document may carry `metadata.XD` (same fields as above). When the use case defines the same document, **each field the patient states takes precedence** over the use case's field of the same name; fields the patient leaves out keep the use case's value. For a document the use case does not define, the patient's `metadata.XD` is the only source.
- Everything else a message needs comes from the content itself rather than from either JSON file: event and service times, language, patient identifiers, author, facility, source organization OID, context identifiers, and a CDA's own document id. The content must therefore agree with `patient.json` (e.g. an HL7 v2 `PID-3` uses `patientId`, and the name, birth date and gender match).

## Conventions

- Content files named by transaction number (`tx1-...`) or by document; HL7 v2 is `text/hl7v2` (`.txt`), human-readable content is `text/plain` with an optional `text/html` twin, FHIR Tasks are `application/fhir+xml`.
- **Address placeholders**: the files here are templates, so the Direct address of the sender and of the recipient inside a content file is written as `{{sender.address}}` or `{{recipient.address}}`, never as a real address. The testing tool replaces them with the addresses it actually sends from and to (including in the metadata). Use them wherever a file names the sending organization/author mailbox or the receiving mailbox (e.g. CDA author/custodian/organization telecoms, a FHIR Task's requester/owner emails and endpoint addresses). Other addresses (e.g. a patient's own email) stay as written. Times that mean "created or sent now" (a Task's `authoredOn`) are `{{now}}` (ISO 8601, FHIR) or `{{now.hl7}}` (HL7 timestamp, CDA/HL7 v2); never use them for clinical event times. Any other `{{...}}` marker is an error.
- The tx1/tx2/tx4 HL7 v2 notifications are ADT messages and must carry the pregnancy (perinatal) episode context identifier with its assigning authority.
- Identifiers, OIDs, addresses and NPIs in the sample content are made-up placeholders.
- **Source of truth**: the implementation guide. At present only TX1 (`send-outpatient-visit-notification`) is authoritative; the other transactions in `useCase.json` are being updated to match the revised guide, so do not treat their codes as final.

## Adding a patient or scenario

1. Create `patients/<patientDir>/` with the content files and a `patient.json` as above.
2. For documents the use case does not define, give each a `mimeType` and the `metadata.XD` its DocumentEntry needs (the codes should match what the document states about itself, e.g. a CDA's LOINC `code`).
3. Check the JSON parses, push, and use **Refresh Use Cases** in the testing UI (or wait for the hourly pull).
