---
name: Find who FDA cited under a regulation
description: >-
  Work the FDA inspection-citation dataset from the regulation end — start with an FD&C Act or
  21 CFR reference and find the establishments FDA cited under it, with the inspection each
  citation came from and what the investigator wrote.
api: openapi/fda-regulations-data-dashboard-openapi.yml
operations:
  - inspectionsCitations
  - inspectionsClassifications
---

# Find who FDA cited under a regulation

`POST /inspections_citations`, `operationId: inspectionsCitations`, base
`https://api-datadashboard.fda.gov/v1`. Read-only; two credential headers; TLS 1.2.

This is the one FDA dataset keyed to the regulation itself. `ActCFRNumber` carries the FD&C Act
section or Title 21 CFR reference the investigator cited, and `ShortDescription` /
`LongDescription` carry FDA's own wording of what the citation means.

## Search by CFR reference

`ActCFRNumber` is a string field and matches partially, so a title-and-part prefix pulls the whole
family of citations beneath it.

```json
{
  "start": 1,
  "rows": 1000,
  "returntotalcount": true,
  "sort": "InspectionEndDate",
  "sortorder": "DESC",
  "filters": {
    "ActCFRNumber": ["211.22"],
    "InspectionEndDateFrom": ["2023-01-01"]
  },
  "columns": ["ActCFRNumber","ShortDescription","LongDescription","FEINumber","LegalName",
              "City","State","CountryName","InspectionID","CitationID","ProgramArea",
              "InspectionEndDate"]
}
```

Check `returntotalcount` on the first call: a broad CFR prefix can match far more than one page.

## Narrow by program area or geography

`ProgramArea` separates the FDA program the inspection ran under. `CountryCode`, `State` and `City`
narrow geographically. All are partial-match string fields.

```json
{
  "start": 1,
  "rows": 500,
  "returntotalcount": true,
  "sort": "LegalName",
  "sortorder": "ASC",
  "filters": {
    "ActCFRNumber": ["820"],
    "ProgramArea": ["Devices"],
    "CountryCode": ["US"]
  },
  "columns": ["ActCFRNumber","ShortDescription","FEINumber","LegalName","State",
              "InspectionID","CitationID","InspectionEndDate"]
}
```

## Pivot to the inspection

Every citation row carries the `InspectionID` it came from. Feed those into
`POST /inspections_classifications` (`operationId: inspectionsClassifications`) to get the outcome
FDA assigned — the `Classification` and `ClassificationCode` — because a citation on its own does
not tell you whether the inspection ended NAI, VAI or OAI.

```json
{
  "start": 1,
  "rows": 100,
  "sort": "InspectionEndDate",
  "sortorder": "DESC",
  "filters": { "FEINumber": [3003378587] },
  "columns": ["FEINumber","LegalName","InspectionID","Classification","ClassificationCode",
              "ProjectArea","ProductType","InspectionEndDate"]
}
```

`/inspections_classifications` is keyed on `InspectionID` but filters on `FEINumber`, so pivot by
firm and match `InspectionID` client-side.

## Reading the response

`statuscode: 400` is success; `412` is no match; `413` means a fieldname in `filters` is not valid
for this dataset — the response names it in `invalid_filters`. Fieldnames are case-sensitive and
differ between datasets.

## Do not

- Do not treat `ActCFRNumber` as a citation count. One inspection produces many citation rows.
- Do not read a citation as a finding of violation on its own. It is what an investigator observed
  and referenced; `Classification` on the inspection record is FDA's outcome.
- Do not invent CFR references. Only the values FDA returns in `ActCFRNumber` are real.
