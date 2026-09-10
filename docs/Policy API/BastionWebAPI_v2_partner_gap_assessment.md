# Bastion Policy Web API Partner Gap Assessment

## Purpose and conclusion

This document assesses the partner requests against the current `BastionWebAPI_v2` contract, the `bastion-website` implementation, and the `octohealth-erp` implementation. It proposes the work required in each project without changing enrollment, pricing, termination, or any application code.

The requested capabilities are feasible, and parts of status, claims, and dependent retrieval already exist internally. The recommended approach is to add a small, vendor-scoped read API in `bastion-website`, backed by purpose-built service endpoints in `octohealth-erp`. Existing endpoints should be reused as implementation references, but not exposed directly without tightening authorization, response contracts, pagination, status normalization, and personally identifiable information handling.

The highest-priority dependency is member lookup. Newer Bastion-created records may contain an `externalRef`, email, and phone locally, but the external reference is not consistently propagated to ERP and older ERP-only records may not exist in Bastion's local member tables. A reliable lookup therefore requires an ERP-side search capability plus a durable vendor-to-member reference mapping.

## Scope

This assessment covers:

- Member lookup by phone, email, or partner external reference
- Live cover status
- Member claims
- Current dependants
- Document upload for an enrolled member
- Required security, data model, testing, and documentation work

It does not propose changes to enrollment, pricing, policy termination, or endorsement behavior.

## Current implementation findings

### Published documentation

`BastionWebAPI_v2.md` documents member details by payment or policy reference, but does not document the already-implemented enrollee status, enrollee list, utilization, visits, or plan endpoints. The policy-details section also shows inconsistent URLs: the documented endpoint uses `/api/v2/webhook/vendors/policy-details`, the example uses a path parameter, while the code implements `GET /api/v2/vendors/policy-details?reference=...`.

The existing member-details operation accepts only `reference`, which is validated as required and is resolved through a vendor-scoped policy payment. It is not a member search operation and cannot recover an internal member identifier from phone, email, or external reference.

### Bastion website

The vendor routes already expose the following useful endpoints in `api/vendor/config/routes.json`:

- `GET /api/v2/vendors/member-details`
- `GET /api/v2/vendors/policy-details`
- `GET /api/v2/enrollee-all`
- `GET /api/v2/enrollee-status`
- `GET /api/v2/enrollee-plan`
- `GET /api/v2/enrollee-utilization`
- `GET /api/v2/enrollee-visits`

All of these routes use vendor authentication and the vendor permission policy. There is a duplicate declaration of `GET /api/v2/vendors/member-details` in the route file, which should be removed when the API surface is revised.

The local `policy-member-details` model already contains `email`, `phone`, `octohealthMemberId`, `status`, `memberGroup`, and `externalRef`. Important limitations are:

- `status` is a Boolean local flag and is not a sufficient live cover state.
- New webhook-created draft policies now carry an explicit `ownerVendor` relation, `creationSource`, and `ownershipAssignedAt`. Legacy ownership remains derivable through member group, draft policy, policy payment, and broker relationships until the backfill is completed.
- `externalRef` is accepted during initial vendor onboarding and can be stored by the generic local create path, but it is not sent to ERP in `api/policy-member/services/policy-member.js`.
- The endorsement validation and local record creation path do not consistently accept or save `externalRef`.
- No lookup indexes or uniqueness rules are defined for normalized phone, normalized email, or vendor plus external reference.

The current member-details service returns complete local member records and decrypts NIN values. A new lookup response must not reuse that unrestricted shape; it should return only the identifiers and summary fields partners need.

### OctoHealth ERP

ERP already provides useful source data:

- `get_member_status.php` calculates `Active`, `Inactive`, `Inactive_expired`, or `Deleted` from live policy, scheme, and member data.
- `get_broker_enrollees` returns paginated principal members, attaches their relations, and includes an equivalent status calculation.
- `view_member_utilization` loads claims and authorizations to calculate used limits and balances.
- `get_member_visits` returns authorization or appointment information.
- Older mobile functions return claim lists and payment information.

These are valuable references, but they are not yet an adequate external partner contract:

- `get_broker_enrollees` searches names and internal member ID, not email, phone, or partner external reference.
- Several queries build SQL with escaped string interpolation rather than bound parameters.
- Claim statuses are spread across approval, audit, lot, and payment fields; there is no single stable partner-facing status.
- `view_member_utilization` obtains claim rows, but `utils/octohealth-erp.js` maps only member summary, provider, total spent, and balance. The claims collection is discarded before Bastion returns the response.
- ERP has member photo upload behavior, but no purpose-built, secure member identity-document API or document metadata model was identified.

## Recommended external API contract

The paths below are recommended canonical paths. They can be adjusted to the team's naming conventions, but one consistent `/api/v2/vendors/...` namespace should be used.

### New consolidated member lookup and details endpoint

`GET /api/v2/vendors/member-lookup`

This must be a new endpoint. The existing `GET /api/v2/vendors/member-details?reference=...` endpoint and its response contract should remain unchanged for existing partner integrations.

Accept exactly one of:

- `reference`
- `externalRef`
- `memberId`
- `phone`
- `email`

Optional query fields:

- `page` and `limit` for phone or email collisions
- `includeInactive`, default `false`; set it to `true` when removed dependants must be included
- `policyId`, optional; use it only to disambiguate an identifier that matches more than one policy owned by the authenticated vendor

The new endpoint performs three operations in one request: resolve the member, retrieve the live ERP cover status, and return the current dependant list when the resolved member is a principal.

Recommended single-match response:

```json
{
  "status": true,
  "data": {
    "memberId": "100020001",
    "policyId": "POL-1248901",
    "policyNumber": "23409",
    "externalRef": "VENDOR-A-012345",
    "firstName": "John",
    "lastName": "Doe",
    "relation": "Self",
    "coverStatus": "active",
    "dependants": [
      {
        "memberId": "100020002",
        "firstName": "Jane",
        "lastName": "Doe",
        "relation": "Spouse",
        "coverStatus": "active"
      }
    ]
  }
}
```

`reference` retains its current meaning as the Bastion payment or policy reference; `externalRef` is the partner's member reference. The two names must not be treated as aliases.

Phone and email are not globally unique. Bastion first resolves local matches through `policy-member-details.memberGroup` to a draft policy and then restricts those policies by `ownerVendor`. When an identifier produces multiple vendor-owned matches, the endpoint returns `409 Conflict` with a minimal `matches` array containing member and policy identifiers; the caller may retry with `policyId`. `externalRef` should be unique within a vendor, not globally. Phone values should be normalized to E.164 where possible, email should be lower-cased and trimmed, and raw search values should not be written to application logs.

The response must be restricted to policies paid for by the authenticated vendor. A vendor must never be able to discover another vendor's member through a shared phone or email.

The response must include a stable member summary and a live `coverStatus`. The status must come from ERP at request time, not from the Bastion Boolean `policy-member-details.status` field.

Recommended external states:

| ERP condition | External state | Decision needed |
| --- | --- | --- |
| Active | `active` | None |
| Policy end date passed | `lapsed` | Confirm whether expired and lapsed are equivalent |
| Member deleted | `terminated` | Confirm whether deleted is always a final termination |
| Leaving date passed or inactive | `suspended` or `terminated` | Business must define whether this condition is temporary or final |

The existing `GET /api/v2/enrollee-status?memberId=...` should remain unchanged for current consumers. The new endpoint includes `coverStatus` so new partners do not need a second round trip.

For a principal member, `dependants` contains the current ERP family list. For a member without dependants, it is an empty array. If the supplied identifier resolves to a dependant, the API should return that member as the main `data` object and an empty `dependants` array unless product explicitly chooses to return the whole family.

### Claims

`GET /api/v2/vendors/members/{memberId}/claims`

Recommended query fields:

- `page`, `limit`
- `status`
- `from`, `to`
- `includeItems`, default `false`

Minimum claim response fields:

- `claimId` and `claimNumber`
- `memberId` and `policyId`
- `providerName`
- `serviceDate` or claim date
- `claimType`
- `status`: `pending`, `approved`, `paid`, or `rejected`
- `requestedAmount`, `approvedAmount`, and `paidAmount`
- `currency`
- `items` when requested and available

ERP must define one deterministic mapping from its approval, audit, lot, and payment columns to the four external states. A suggested precedence is `paid` when confirmed payment exists, `rejected` when the approval outcome is denied or rejected, `approved` when final approval exists, otherwise `pending`. The claims product owner should confirm this rule before implementation.

`view_member_utilization` should not become the claims API as-is. It is optimized for benefit balance calculation, returns unrelated cover-limit data, does not provide a stable claims contract, and Bastion currently discards its claims array. Its `getMemberData` claim query and the mobile `get_claim_list` query can be refactored into a shared ERP claim-query service used by both utilization and the new endpoint.

ERP's `get_broker_enrollees` already attaches `relations` to each principal and is suitable as a query reference for populating `dependants`. Removed dependants should be excluded by default and included only when `includeInactive=true`.

### Member document upload

`POST /api/v2/vendors/policies/{policyId}/members/{memberId}/documents`

Use `multipart/form-data` with:

- `file`, required
- `documentType`, required, for example `national_id`, `passport`, or `other`
- `documentNumber`, optional and treated as sensitive
- `expiresOn`, optional
- `externalDocumentRef`, optional for idempotency

Recommended response fields are `documentId`, `memberId`, `documentType`, `fileName`, `uploadedAt`, and `reviewStatus`.

This requires a new ERP document service and metadata table linked to the member scheme and policy. Identity documents must not be stored in the publicly served member-photo directory. Storage should be private, encrypted at rest, scanned for malware, allowlisted by MIME type and extension, constrained by file size, and accessed through authorization checks or short-lived signed URLs. Replacement and retention behavior must be defined per document type.

Bastion must accept multipart data, validate metadata and file content, verify that the member belongs to the authenticated vendor, and forward the file to ERP. `utils/octohealth-erp.js` currently creates an Axios client with JSON content type; the upload method must use a multipart boundary generated by the form-data implementation rather than reuse the JSON header.

## Required changes by project

### Bastion website changes

1. Add a Joi schema for the new member lookup filters, requiring exactly one of `reference`, `externalRef`, `memberId`, `email`, or `phone`, plus schemas for claims filters and document metadata.
2. Add `GET /api/v2/vendors/member-lookup` with a new controller and service method. Do not reuse or alter the existing `member-details` handler.
3. Add service methods that always derive the allowed ERP policy or policies from the authenticated vendor; do not trust a caller-supplied policy ID by itself.
4. Add `lookupMemberWithDetails`, `getMemberClaims`, and `uploadMemberDocument` methods to `utils/octohealth-erp.js`. The lookup method should return the member, live status, and dependants together.
5. Normalize ERP responses into camelCase, stable status enums, decimal amounts, ISO dates, and one pagination format.
6. Return minimal lookup data. Do not include decrypted NIN, document numbers, addresses, or unrelated member fields.
7. Persist and index normalized vendor/member lookup keys. If a separate mapping table is used, recommended keys are `vendorId`, `externalRef`, `octohealthMemberId`, `policyId`, `normalizedEmail`, and `normalizedPhone`.
8. Propagate `externalRef` through initial onboarding and endorsement flows to ERP after the ERP schema is ready. Add vendor-scoped uniqueness validation and an idempotent reconciliation path.
9. Correct the policy-details URL mismatch in the documentation. The duplicate declaration of the existing member-details route can be removed only after confirming that doing so does not affect Strapi route ordering.
10. Add structured audit logs for lookup and document upload without logging raw PII, API tokens, request files, or full Axios error configuration.
11. Add vendor permission keys for member lookup, claims, and documents so each capability can be granted independently.
12. Bind every policy created through a vendor webhook to the authenticated vendor at creation time. For legacy records, accept only a matching `policy-payment.broker` relationship and backfill the explicit owner relation before rollout.

### OctoHealth ERP changes

1. Add a vendor-scoped member lookup and details endpoint accepting the Bastion reference, normalized email, normalized phone, internal member ID, or external reference. Return the resolved member, live status, and current dependants in one response.
2. Add storage for the partner external reference if no suitable member field exists. Enforce uniqueness within the vendor or integration namespace and backfill where mapping data is available.
3. Refactor the current status calculation into one reusable function or query and return the canonical external status plus the raw internal reason for diagnostics.
4. Add a paginated member claims endpoint using bound SQL parameters. Reuse claim retrieval logic from `getMemberData` and the mobile claim list, then map approval and payment states into the agreed external enum.
5. Add an optional claim-items query only if the line-item schema is reliable and the privacy contract permits it.
6. Add dependant retrieval based on `mm_principal_id` to the new lookup/details operation, scoped by policy, with live status for every returned dependant.
7. Add a private document metadata table and upload service. Validate member-policy ownership, file type, size, malware scan result, retention, replacement, and audit history.
8. Replace raw interpolated search SQL in any reused path with prepared statements or bound parameters.
9. Standardize errors and HTTP status codes: `400` invalid input, `401` invalid service token, `403` vendor not authorized for the member or policy, `404` no member, and `409` ambiguous or conflicting lookup results where applicable.

## Backfill and historical-member strategy

Historical members are the reason lookup cannot depend only on Bastion's local `externalRef`.

Recommended sequence:

1. Build ERP lookup by normalized phone and email under the authenticated vendor's ERP policy scope.
2. Introduce the ERP external-reference field or a dedicated integration mapping table.
3. Export known Bastion mappings of vendor, external reference, OctoHealth member ID, and policy ID.
4. Upsert those mappings into ERP with a reconciliation report for duplicates and missing members.
5. Allow partners to supply a one-time historical mapping file for records predating the integration, subject to validation.
6. Keep phone and email lookup available because not every historical record will receive an external reference.

Ambiguous matches must be returned as multiple minimal results or a `409` requiring additional filters such as date of birth. The API must never silently select the first matching member.

### Vendor-policy ownership rollout

The ownership backfill is dry-run by default:

```bash
node seeds/backfill-vendor-policy-ownership.js
```

Review and resolve every reported conflict or missing relationship before applying it:

```bash
node seeds/backfill-vendor-policy-ownership.js --apply
```

The script derives the owner only from an existing `policy-payment.broker` relationship; it does not guess from a vendor name or automatically claim an arbitrary ERP policy. Production deployment must ensure the Strapi schema update has created the new draft-policy ownership fields before this script runs. The application retains a fail-closed legacy payment fallback during the transition.

## Security and privacy requirements

- Enforce vendor ownership at both Bastion and ERP layers.
- Require `policyId` for member-ID, external-reference, email, and phone lookup. A payment reference may omit it because Bastion derives the policy from the authenticated vendor's payment record.
- Return the same generic `404` for missing and cross-vendor resources to reduce enumeration risk.
- Treat a `404` and an unauthorized cross-vendor lookup consistently enough to prevent member enumeration.
- Rate-limit lookup endpoints and audit repeated searches.
- Mask phone and email in responses unless the partner needs the complete value.
- Never include NIN or identity-document numbers in member lookup or claims responses.
- Do not log request bodies for enrollment, lookup, or document upload; current onboarding logging should be reviewed because it can print complete member data.
- Use prepared statements in new ERP queries.
- Apply least-privilege permission flags separately for member lookup, status, claims, dependants, and documents.
- Define retention and deletion policies for uploaded identity documents before production release.

## Delivery sequence

### Phase 1 Member recovery and status

- ERP lookup by phone and email
- External-reference persistence and mapping
- New Bastion consolidated member lookup and details endpoint
- Canonical live status mapping
- Live status and dependants in the consolidated response
- Historical mapping reconciliation

This phase unlocks all subsequent member-specific operations.

### Phase 2 Claims

- Dedicated paginated claims endpoint
- Status and amount normalization
- Optional itemized claim details after privacy approval

### Phase 3 Documents

- Private document storage and upload workflow
- Additional permission controls and retention rules

## Verification plan

Each feature should have unit, integration, and cross-system contract tests covering:

- Exact match, no match, and multiple matches for phone and email
- Vendor-scoped external-reference uniqueness
- Cross-vendor member enumeration attempts
- Active, inactive, expired, deleted, and boundary-date status cases
- Principal and dependant resolution, including removed dependants
- Claims in pending, approved, rejected, and paid states
- Requested, approved, and paid amount mapping and currency precision
- Pagination and stable ordering
- Empty claims and empty dependant responses returning successful empty lists
- Multipart validation, MIME spoofing, oversize files, and duplicate upload idempotency
- Malware rejection after the production scanning service is connected
- ERP timeout and partial failure behavior
- Backward compatibility for existing v2 endpoints

## Documentation updates completed

`BastionWebAPI_v2.md` now documents the implemented contracts and:

- Corrects the policy-details path and example
- Removes numbering gaps and duplicate or inconsistent endpoint descriptions
- Documents query parameters, pagination, and response enums
- Adds the new consolidated member lookup and details endpoint, including live status and dependants, plus claims and document upload sections
- Clearly states that the existing `GET /api/v2/vendors/member-details` contract remains unchanged
- Uses synthetic examples that do not expose NIN or identity-document values
- Publishes the canonical claim and cover status values
- States the approved upload size and file types; retention and replacement remain deployment-governance items

## Implementation decisions applied

1. A past member leaving date maps to `suspended`, an explicitly deleted member or scheme maps to `terminated`, and an expired policy maps to `lapsed`.
2. ERP is the source of truth for new external references. Bastion retains a vendor-scoped local fallback for historical records already carrying `externalRef` locally.
3. An external reference is unique within an ERP policy. The same value may exist under a different policy.
4. Claims return requested, approved, and paid amounts separately.
5. New vendor webhook policies are explicitly owned by the authenticated vendor. Member lookup derives the policy from the local member and ownership relationships; a caller-supplied policy ID is optional and is used only as an ownership-checked disambiguation filter.
6. Claims use `/api/v2/vendors/members/{memberId}/claims`; Bastion resolves the policy through the indexed local member mapping and verifies vendor ownership. Document uploads remain policy-scoped at `/api/v2/vendors/policies/{policyId}/members/{memberId}/documents`.
5. Claim line items are opt-in through `includeItems=true`.
6. Removed dependants are excluded by default and included with `includeInactive=true`.
7. Uploads accept PDF, JPEG, and PNG up to 5 MB. Retention, replacement, and malware-scanning policies still require production operations approval.

## Source locations reviewed

- `api-docs/docs/Policy API/BastionWebAPI_v2.md`
- `bastion-website/api/vendor/config/routes.json`
- `bastion-website/api/vendor/controllers/vendor.js`
- `bastion-website/api/vendor/services/vendor.js`
- `bastion-website/api/policy-member/services/policy-member.js`
- `bastion-website/api/policy-member-details/services/policy-member-details.js`
- `bastion-website/api/policy-member-details/models/policy-member-details.settings.json`
- `bastion-website/schemas/vendor.js`
- `bastion-website/utils/octohealth.js`
- `bastion-website/utils/octohealth-erp.js`
- `octohealth-erp/docs/webservices/get_member_status.php`
- `octohealth-erp/docs/webservices/get_member_plan.php`
- `octohealth-erp/docs/admin/modules/module_app.php`
