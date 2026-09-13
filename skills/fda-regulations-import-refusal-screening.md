---
name: Screen imports for FDA refusal risk
description: >-
  Search FDA import refusals by country, industry, product code and date range to see what FDA has
  turned away at the border, and pull the refusal charges cited so a supply-chain or regulatory
  team can judge exposure before a shipment ships.
api: openapi/fda-regulations-data-dashboard-openapi.yml
operations:
  - importRefusals
---

# Screen imports for FDA refusal risk

One operation: `POST /import_refusals`, `operationId: importRefusals`, base
`https://api-datadashboard.fda.gov/v1`. Read-only. Credentials are the `Authorization-User` and
`Authorization-Key` headers; TLS 1.2 required.

## What the dataset holds

One row per refused shipment: the firm (`FEINumber`, `FirmName` and address), the origin
(`CountryCode`, `CountryName`), FDA's own routing (`DistrictCode`, `DistrictDescription`),
FDA's coded product taxonomy (`IndustryCode`, `IndustryCodeDescription`, `ProductCategory`,
`ProductCode`, `ProductCodeDescription`), the date (`RefusalDate`), the analysis path
(`FDASampleAnalysis`, `PrivateLabAnalysis`) and the charges cited (`RefusalCharges`).

Full field list and match types: https://datadashboard.fda.gov/oii/api/api-definitions-refusals.htm

## Screen by country and product category

```json
{
  "start": 1,
  "rows": 500,
  "returntotalcount": true,
  "sort": "RefusalDate",
  "sortorder": "DESC",
  "filters": {
    "CountryCode": ["JP","MX"],
    "ProductCategory": ["Devices"],
    "RefusalDateFrom": ["2024-01-01"]
  },
  "columns": ["FEINumber","FirmName","CountryCode","ProductCode","ProductCodeDescription",
              "RefusalDate","RefusalCharges","DistrictCode"]
}
```

## Filter semantics that will bite you

- **String fields match `Partial`, as SQL `LIKE %term%`.** `"ProductCodeDescription": ["RECEIVER"]`
  matches every description containing the word. Narrow with `ProductCode` when you want exact.
- **Dates are not filtered on the fieldname.** Use `RefusalDateFrom` and/or `RefusalDateTo` as the
  key, never `RefusalDate`. Only the first value in a date array is honoured — the rest are
  silently ignored, which is how a range quietly becomes a single bound.
- **Accepted date input** is `MM-DD-YYYY`, `MM/DD/YYYY` or `YYYY-MM-DD`. Responses are always ISO
  `YYYY-MM-DD` with no time component.
- **`FEINumber` and `ShipmentID` are numeric** — unquoted. Everything else is a quoted string.
- **`null` and `""` are different.** `"AddressLine2": [null, ""]` matches both "no data" and
  "empty text"; either one alone matches only itself.

## Checking a specific supplier

```json
{
  "start": 1,
  "rows": 100,
  "returntotalcount": true,
  "sort": "RefusalDate",
  "sortorder": "DESC",
  "filters": {
    "FEINumber": [3003378587, 1000117386],
    "RefusalDateFrom": ["2020-01-01"]
  },
  "columns": ["FEINumber","FirmName","CountryCode","ProductCode","RefusalDate","RefusalCharges"]
}
```

If the supplier also has inspection or compliance history, pivot on the same `FEINumber` into
`inspectionsClassifications`, `inspectionsCitations` and `complianceActions` — see the
"Build a firm's FDA compliance history" skill.

## Reading the response

`statuscode: 400` is success. `statuscode: 412` means no refusals matched — which for this dataset
is a meaningful, reportable result, not an error. Page with `start` + `resultcount` until
`resultcount` is below your `rows`; `rows` above 5000 returns `statuscode: 415`.

## Do not

- Do not present an empty result as "this supplier is cleared". It means FDA published no refusal
  matching your filter over the window you asked for.
- Do not fabricate a refusal reason. `RefusalCharges` is the only field that states one, and it is
  sometimes null.
