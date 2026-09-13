---
name: Build a firm's FDA compliance history
description: >-
  Assemble the full public FDA regulatory record for one regulated establishment — its inspection
  outcomes, the CFR citations against it, the compliance actions FDA took, and any shipments
  refused at the border — by joining all four FDA Data Dashboard datasets on the firm's FEI number.
api: openapi/fda-regulations-data-dashboard-openapi.yml
operations:
  - inspectionsClassifications
  - inspectionsCitations
  - complianceActions
  - importRefusals
---

# Build a firm's FDA compliance history

Use this when you need everything FDA publishes about how one establishment has performed against
FDA regulations. The four datasets are separate endpoints with no join server-side; you join them
yourself on `FEINumber`.

## Before you start

- Base URL is `https://api-datadashboard.fda.gov/v1`. TLS 1.2 is required.
- Send both credential headers on every request: `Authorization-User` (your FDA-approved email)
  and `Authorization-Key` (the FDA-generated key). Missing or wrong credentials return
  `statuscode: 401`, `message: "Not Authorized."` — inside an HTTP 200 body.
- Every endpoint is `POST` and every endpoint is a read. Nothing you send here changes anything at
  FDA, so there is nothing to undo and no idempotency key to set.
- `Content-Type: application/json`.

## Read the status code correctly

This API does not use HTTP status. Read `statuscode` from the JSON body:

- `400` — **Success**. Not an error.
- `412` — No results found. Expected, and common on a firm with a clean record.
- `401` — Not authorized.
- `406`, `407`, `410`, `411`, `413`, `414` — your request body is malformed; the response names the
  offending fieldnames in `invalid_columns`, `invalid_filters`, `missing_parameters` or
  `fieldnames_with_invalid_values`. Fix and retry; do not retry unchanged.

Full registry: `errors/fda-regulations-error-codes.yml`.

## Step 1 — inspection outcomes

`POST /inspections_classifications` with `operationId: inspectionsClassifications`.

```json
{
  "start": 1,
  "rows": 100,
  "returntotalcount": true,
  "sort": "InspectionEndDate",
  "sortorder": "DESC",
  "filters": { "FEINumber": [3003378587] },
  "columns": ["FEINumber","LegalName","InspectionID","Classification","ClassificationCode",
              "ProductType","ProjectArea","InspectionEndDate","PostedCitations"]
}
```

`FEINumber` is numeric — send it unquoted. Keep every `InspectionID` you get back; step 2 needs them.

## Step 2 — the citations behind those inspections

`POST /inspections_citations` with `operationId: inspectionsCitations`. Filter by the same
`FEINumber`, and read `ActCFRNumber` — that is the FD&C Act section or Title 21 CFR reference the
investigator cited, which is the actual regulation at issue.

```json
{
  "start": 1,
  "rows": 1000,
  "returntotalcount": true,
  "sort": "InspectionEndDate",
  "sortorder": "DESC",
  "filters": { "FEINumber": [3003378587] },
  "columns": ["FEINumber","LegalName","InspectionID","CitationID","ActCFRNumber",
              "ShortDescription","LongDescription","ProgramArea","InspectionEndDate"]
}
```

Join back to step 1 on `InspectionID`. An inspection with `PostedCitations` set but no rows here
means the citations are not yet posted, not that the inspection was clean.

## Step 3 — compliance actions

`POST /compliance_actions` with `operationId: complianceActions`.

```json
{
  "start": 1,
  "rows": 100,
  "returntotalcount": true,
  "sort": "ActionTakenDate",
  "sortorder": "DESC",
  "filters": { "FEINumber": [3003378587] },
  "columns": ["FEINumber","LegalName","CaseInjunctionID","ActionType","ProductType",
              "Center","Region","ActionTakenDate"]
}
```

## Step 4 — import refusals

`POST /import_refusals` with `operationId: importRefusals`. This dataset calls the firm name
`FirmName`, not `LegalName` — the only field that changes spelling across the four datasets.

```json
{
  "start": 1,
  "rows": 500,
  "returntotalcount": true,
  "sort": "RefusalDate",
  "sortorder": "DESC",
  "filters": { "FEINumber": [3003378587] },
  "columns": ["FEINumber","FirmName","CountryCode","CountryName","DistrictCode",
              "ProductCode","ProductCodeDescription","RefusalDate","RefusalCharges"]
}
```

## Paging

Set `returntotalcount: true` on the **first** request of each dataset only — FDA recomputes it on
every call and the value does not change. Then page by adding the returned `resultcount` to your
previous `start` until `resultcount` drops below your `rows` value. Hard ceiling is 5000 rows per
response; asking for more returns `statuscode: 415`.

## Do not

- Do not treat an HTTP 200 as success. Read `statuscode`.
- Do not quote `FEINumber`, `InspectionID` or `CitationID` — they are numeric and quoting them
  returns `statuscode: 414`.
- Do not send an optional key (`start`, `rows`, `returntotalcount`) with an empty value. Omit it.
  Only the four required keys (`sort`, `sortorder`, `filters`, `columns`) tolerate empty values.
- Do not infer that a firm is compliant from `statuscode: 412`. It means no rows matched your
  filter, which may mean your FEI number is wrong.
