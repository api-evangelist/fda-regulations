# FDA Regulations (fda-regulations)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Federal regulations governing the safety, efficacy and security of food, human and animal drugs,
biologics, medical devices, cosmetics and tobacco products under FDA jurisdiction — codified at
21 CFR and administered through the agency's inspection, citation, import and compliance-action
programs. The machine-readable surface for that regulatory activity is the **FDA Data Dashboard
API (DDAPI)**, published by the FDA Office of Inspections and Investigations (formerly the Office
of Regulatory Affairs).

**URL:** [Visit APIs.json URL](https://raw.githubusercontent.com/api-evangelist/fda-regulations/refs/heads/main/apis.yml)

## Scope

- **Type:** Index
- **Position:** Producing
- **Access:** 3rd-Party
- **Parent:** [Food and Drug Administration](https://github.com/api-evangelist/food-and-drug-administration) (product)

## APIs

### FDA Data Dashboard API

Four POST search endpoints over FDA's compliance and enforcement datasets. Base URL
`https://api-datadashboard.fda.gov/v1`; credentials are the `Authorization-User` and
`Authorization-Key` headers, issued free by FDA on request. TLS 1.2 required.

| Operation | Endpoint | What it returns |
|---|---|---|
| `inspectionsClassifications` | `POST /inspections_classifications` | Inspection outcomes and classifications by firm |
| `inspectionsCitations` | `POST /inspections_citations` | The FD&C Act / 21 CFR references cited against a firm (`ActCFRNumber`) |
| `complianceActions` | `POST /compliance_actions` | Warning letters, injunctions and seizures |
| `importRefusals` | `POST /import_refusals` | Shipments refused entry, with the charges cited |

- [Documentation](https://datadashboard.fda.gov/oii/api/index.htm)
- [OpenAPI 3.0.0 (first-party)](https://datadashboard.fda.gov/oii/api/ddapi.json) — harvested verbatim to `openapi/_original/`

> Note: this API signals failure with a numeric `statuscode` **inside the JSON body**, not with
> HTTP status. `statuscode` 400 means Success and 401 means Not Authorized. See
> `errors/fda-regulations-error-codes.yml`.

## Artifacts

| Directory | File | What it holds |
|---|---|---|
| `openapi/` | `fda-regulations-data-dashboard-openapi.yml`, `_original/fda-data-dashboard-openapi.json` | FDA's own OpenAPI 3.0.0, harvested verbatim |
| `overlays/` | `fda-regulations-data-dashboard-overlay.yaml` | Overlay 1.0.0 of our enhancements — never mutates the original |
| `authentication/` | `fda-regulations-authentication.yml` | The two apiKey header schemes |
| `conventions/` | `fda-regulations-conventions.yml` | Auth, paging, filtering, error envelope, idempotency (`na`), reversibility (`na`) |
| `errors/` | `fda-regulations-error-codes.yml`, `fda-regulations-problem-types.yml` | The full 400–419 statuscode registry, and what the contract itself declares |
| `data-model/` | `fda-regulations-data-model.yml` | Entity graph and the per-dataset field lists |
| `examples/` | `fda-regulations-examples.yml` | FDA's published request/response examples, made machine-readable |
| `skills/` | 3 Agent Skills + `_index.yml` | Grounded in real operationIds |
| `conformance/` | `fda-regulations-conformance.yml` | Standards asserted, including the 21 CFR / FEI / product-code domain identifiers |
| `lifecycle/` | `fda-regulations-lifecycle.yml` | Versioning, and the absence of a deprecation policy, SLA and status page |
| `rate-limits/` | `fda-regulations-rate-limits.yml` | An honest zero — FDA publishes none |
| `plans/` | `fda-regulations-plans-pricing.yml` | An honest zero — free, credentialed |
| `packages/` | `fda-regulations-packages.yml` | An honest zero — no first-party SDK in any registry |
| `sandbox/` | `fda-regulations-sandbox.yml` | The in-page try-it console; no test mode |
| `well-known/` | `fda-regulations-well-known.yml` | Every named path probed on four hosts; nothing served |
| `mcp/` | `fda-regulations-mcp.yml` | A **candidate** tool list derived from the spec — FDA ships no MCP server |
| `agentic-access/` | `fda-regulations-agentic-access.yml` | Recommended execution contracts, curated to read-only |
| `security/` | `fda-regulations-domain-security.yml` | Probed TLS / HSTS / DNSSEC / CAA / SPF / DMARC |
| `llms/` | `fda-regulations-llms.txt` | Generated — FDA publishes none |

## What FDA does not publish

Probed 2026-09-12 and recorded as absent rather than omitted: no MCP server, no A2A agent card,
no GraphQL, no AsyncAPI, no webhooks, no `/.well-known/` documents on any FDA host (including
`security.txt`), no client SDKs in any registry, no status page, no dated changelog, no
deprecation policy and no published rate limits.

## Tags

Regulatory Compliance, Healthcare, Medical Devices, Pharmaceuticals, Food Safety, Inspections,
Enforcement, Federal Government, Public Data, Imports

## Timestamps

- **Created:** 2025-01-01
- **Modified:** 2026-09-12

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com
