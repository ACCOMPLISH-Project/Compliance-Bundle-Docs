# Sample interactions between Automation Engine and Assessment Service

## Automation Engine -> Assessment Service
These are details required for Assessment Service to identify in scope articles for each regulation

### GDPR
These are the attributes that the ACAS will need to extract from the Scenario Details.
```json
{
    "contains_pii": True,
    "contains_special_categories": True|False,
    "collection_method": "Direct"|"Indirect",
    "user": id or string or something,
    "is_controller": True|False
}
```
### AI Act
```json
{
    "risk-level": "Minimal"|"Limited"|"High"|"Unacceptable"
}
```
### Data Act
```
TBD
```
### Data Goverance Act
```
TBD
```

### Topics
#### acas.assessment_requested
```json
{
    "execution_id": "uuid"
    "assessment_id": "019f7f39-4a77-70de-8464-bb420b5817dc",
    "user_id": "uuid",
    "organization_id": "uuid",
    "scope": {
        "gdpr": false | {
            "contains_pii": true, // true if the user uploaded a form, if not, run pii_check in anonymization <- Hook for Automation Engine
            "contains_special_categories": true, // false, // Comes from Data Ingestion OR pii_check
            "collection_method": "Direct", // | "Indirect", // Comes from Data Ingestion? Forms Manager?
            "is_controller": "boolean" // Comes from Data Ingestion
            "additional_info": {
                // 
            }
        }, 
        "ai-act": false | {
            "risk-level": "high_risk"|"medium_risk"|"low_risk" // Comes from Model Training Service
            "additional_info": {
                // 
            }
        }, 
        "data-act": false,
        "data-governance-act": false
    }
}
```
#### acas.acas.additional_info_available
```
TBD
```

## Assessment Service -> Automation Engine
<!-- Response back to CAAE after completion of compliance assessment.
```json
{
    "assessment_id": 1,
    "status": "Compliant"|"Non-Compliant",
    "details": "",
    "compliant_articles": [],
    "non_compliant_articles": [],
    "non_compliance_explanation": [],
    "non_compliance_risk": "" (BETA)
}
``` -->

### Topics
#### acas.assessment_available
```json
{
    "execution_id": "uuid"
    "assessment_id": "019f7f39-4a77-70de-8464-bb420b5817dc",
    "scope": {
        "gdpr": {
            "is_compliant": "boolean",
            "verdict": {
                "article_6": true | "string", // true or explanation of non-compliance
                "article_9": true | "string", // true or explanation of non-compliance
                // etc.
            },
            "risk": "string",
        },
        "ai-act": {
          "is_compliant": "boolean",
          "verdict": {
              "article_15": true | "string",
          },
          "risk": "string",
        }
    }
}
```

#### acas.additional_info_requested
```
TBD
```
