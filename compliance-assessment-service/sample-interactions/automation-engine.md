# Sample interactions between Automation Engine and Assessment Service

## Automation Engine -> Assessment Service
These are the details required for Assessment Service to identify in scope articles for each regulation
### GDPR
These are the attributes that the ACAS will need to extract from the Scenario Details.
```json
{
    "contains_pii": True,
    "contains_special_catagories": True|False,
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
## Assessment Service -> Automation Engine
Rsponse back to CAAE after completion of compliance assessment.
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
```