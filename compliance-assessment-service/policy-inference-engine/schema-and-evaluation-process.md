# ODRL to API: Schema Derivation and Evaluation Process

This document explains two conceptual processes:

1. How an API input schema is derived from ODRL policies.
2. How an input payload is evaluated against those policies.

---

## 1) Schema derivation from policies

### Goal
Produce a user-facing, machine-readable input contract from ODRL JSON-LD files under `src/main/resources/models`.

### Entry point (conceptual)
- Schema discovery endpoint (e.g. `GET /api/schema`).

### Step-by-step

1. **Load all policy sources**
   - Discover `*.jsonld` in classpath `models/`.
   - Parse each file with Jackson.
   - Build `generatedFromPolicies[]` with `id`, `title`, `sourceFile`.

2. **Traverse ODRL structure**
   - Visit:
     - `permission[].constraint[]`
     - `permission[].duty[].constraint[]`
     - nested `consequence[].constraint[]`
     - `prohibition[].constraint[]`
   - Also derive duty action booleans (e.g. `review`, `notify`) from duty actions.

3. **Derive field keys (generic)**
   - Convert `leftOperand` into readable field keys:
     - CURIE/URI token extraction
     - camel-case normalization
     - prefix verb stripping (`has`, `is`, `supports`, …)
     - pluralization for collection operators (`isAnyOf`, `isAllOf`, `in`)
   - Resolve collisions deterministically.

4. **Humanize values**
   - Convert expected policy values to API-readable values (e.g. `ex:paper` -> `paper`).
   - For object operands with `@id`, use `@id` for expected API value.
   - Keep reversible mapping in provenance.

5. **Build field metadata**
   - For each field:
     - `key`, `type`, `required`, `possibleValues`
     - `provenance`:
       - `x-operands`
       - `x-policyIds`
       - `x-appliesTo`
       - `evaluatedAgainst[]` (rule/operator/value/policy/action/path)
       - `valueMappings[]` (`apiValue` <-> `policyValue`)

6. **Type exposure**
   - Inferred from operand shapes.
   - Collection operators force exposed type `array` when applicable.

### Output shape (simplified)

```json
{
  "version": "phase1-v2",
  "inputMode": "flat-fact-map",
  "generatedFromPolicies": [{ "id": "...", "title": "...", "sourceFile": "..." }],
  "fields": [
    {
      "key": "legalBases",
      "type": "array",
      "required": true,
      "possibleValues": ["a61A", "a61B", "a61E", "jointDataControllersAgreement"],
      "provenance": {
        "x-operands": ["dpv:hasLegalBasis"],
        "x-policyIds": ["...article_20"],
        "x-appliesTo": "permission",
        "evaluatedAgainst": [ ... ],
        "valueMappings": [ ... ]
      }
    }
  ]
}
```

---

## 2) Evaluation process

### Goal
Validate and evaluate submitted payload facts against constraints derived from loaded ODRL policies.

### Entry point (conceptual)
- Evaluation endpoint (e.g. `POST /api/inference`).

### Input mode
- Multi-action model:
  - payload contains `actions` (array), e.g. `["obtain", "transfer"]`.
- Remaining facts are submitted as key/value entries matching the derived schema.

### Step-by-step

1. **Build/resolve evaluation model**
   - Resolve policy-derived schema and constraint model for this evaluation.

2. **Find applicable checks**
   - Use selected `actions` to scope which checks apply:
     - permission/prohibition checks are action-scoped.
     - duty/consequence checks are matched by `permissionAction`.

3. **Validate required fields first**
   - Build required field list from applicable checks.
   - If missing fields exist:
     - return `status: "failed-validation"`
     - return `overallVerdict: "unclear"`
     - return `validation.missingRequiredFields`.

4. **Evaluate constraints**
   - Supported operators:
     - `eq`, `neq`
     - `isAnyOf`, `isAllOf`, `in`
     - `lt`, `lte|lteq`, `gt`, `gte|gteq`
   - Matching is normalized for readable values (case-insensitive scalar matching).
   - Array equality and set containment are handled explicitly.

5. **Aggregate verdict**
   - Any fail -> `non-compliant`
   - Else any indeterminate -> `unclear`
   - Else -> `compliant`

6. **Return structured response**
   - Includes:
     - `evaluationId`, `requestId`, `status`, `overallVerdict`
     - `checkedPolicies`
     - `summary` (`checksPassed`, `checksFailed`, `checksIndeterminate`)
     - `results[0].checks[]`
     - `results[0].nextActions.review[]`

### Response style
- One overall verdict plus check-level results and review hints.

### Small example: inference request

```json
{
  "requestId": "eval-001",
  "system": {
    "systemId": "test-system-001",
    "characteristics": {
      "actions": ["obtain", "transfer"],
      "dataSource": "dataSubjectDataSource",
      "legalBases": ["a61A", "jointDataControllersAgreement"],
      "mediaTypes": ["application/json", "text/csv"],
      "technicalMeasures": [
        "accessControlMethod",
        "encryption",
        "activityMonitoring",
        "authorisationProtocols",
        "securityMethod",
        "api"
      ],
      "makeAvailable": true,
      "review": true,
      "notify": true
    }
  }
}
```

### Small example: inference response

```json
{
  "evaluationId": "uuid-v4",
  "requestId": "eval-001",
  "status": "completed",
  "overallVerdict": "compliant",
  "summary": {
    "checksPassed": 12,
    "checksFailed": 0,
    "checksIndeterminate": 0
  },
  "results": [
    {
      "verdict": "compliant",
      "checks": [
        {
          "name": "legalBases",
          "operator": "isAnyOf",
          "status": "pass",
          "expected": ["a61A", "a61B"],
          "actual": ["a61A", "jointDataControllersAgreement"]
        }
      ],
      "nextActions": {
        "review": []
      }
    }
  ]
}
```

---

## Notes

- This document is intentionally process-oriented and endpoint-agnostic.
- Exact endpoint paths and internal classes may change without changing the derivation/evaluation concepts.
- ODRL semantics support is incremental and can be deepened over phases.
