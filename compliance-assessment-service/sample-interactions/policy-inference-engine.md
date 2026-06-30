# Sample interactions between Policy Infernce Engine and Assessment Service

## Assessment Service <-> Policy Inference Engine
### Applicable Articles Check
#### Input
```json
{
    "applicable_articles": ["regulation-article-#"] // i.e. ["gdpr-article-6", "gdpr-article-9", "aiact-article-10",...]
}
```
#### Response
```json
{
    "characteristics": {
        "dataHandling": {
            "obtainsDataFromDataSubject": None,
            "supportsDataAccessRequest": None,
            "supportsDataExport": None,
            "supportsDirectControllerToControllerTransfer": None,
            "supportedExportFormats": []
        },
        "legalAndGovernance": {
            "legalBases": []
        },
        "securityAndOperations": {
            "usesEncryptionInTransit": None,
            "usesEncryptionAtRest": None,
            "usesAccessControl": None,
            "monitorsSecurityActivity": None,
            "usesAuthorizationProtocols": None,
            "providesApiForDataTransfer": None,
            "usesDataTransferTool": None
        }
    }
}
```
### Compliance Check
#### Input
Currently a draft input. Further extension to more articles is needed for integration.
```json
{
    "requestId": "compliance-check-001",
    "checkedPolicies": [
        "gdpr-article-20"
    ],
    "system": {
        "systemId": "test-system-001",
        "characteristics": {
            "dataHandling": {
                "obtainsDataFromDataSubject": true,
                "supportsDataAccessRequest": true,
                "supportsDataExport": true,
                "supportsDirectControllerToControllerTransfer": true,
                "supportedExportFormats": [
                    "application/json",
                    "text/csv"
                ]
            },
            "legalAndGovernance": {
                "legalBases": [
                    "Consent",
                    "Contract",
                    "JointDataControllersAgreement"
                ]
            },
            "securityAndOperations": {
                "usesEncryptionInTransit": true,
                "usesEncryptionAtRest": true,
                "usesAccessControl": true,
                "monitorsSecurityActivity": false,
                "usesAuthorizationProtocols": false,
                "providesApiForDataTransfer": true,
                "usesDataTransferTool": true
            }
        }
    },
    "options": {
        "decisionView": "detailed"
    }
}
```
#### Output
```json
{
    "evaluationId": "550e8400-e29b-41d4-a716-446655440000",
    "requestId": "compliance-check-001",
    "status": "completed",
    "overallVerdict": "non-compliant",
    "checkedPolicies": [
        "gdpr-article-20"
    ],
    "summary": {
        "compliantPolicies": 0,
        "nonCompliantPolicies": 1,
        "indeterminatePolicies": 0,
        "checksPassed": 6,
        "checksFailed": 1,
        "checksIndeterminate": 0
    },
    "results": [
        {
            "verdict": "non-compliant",
            "checks": [
                {
                    "policyIds": [
                        "gdpr-article-20"
                    ],
                    "name": "Data comes from data subject",
                    "status": "pass",
                    "details": "Confirmed",
                    "source": "dataHandling.obtainsDataFromDataSubject"
                },
                {
                    "policyIds": [
                        "gdpr-article-20"
                    ],
                    "name": "Legal basis supports portability",
                    "status": "pass",
                    "details": "Consent/Contract present",
                    "source": "legalAndGovernance.legalBases"
                },
                {
                    "policyIds": [
                        "gdpr-article-20"
                    ],
                    "name": "Joint controller agreement present",
                    "status": "pass",
                    "details": "Declared",
                    "source": "legalAndGovernance.legalBases"
                },
                {
                    "policyIds": [
                        "gdpr-article-20"
                    ],
                    "name": "Machine-readable export format",
                    "status": "pass",
                    "details": "JSON and CSV supported",
                    "source": "dataHandling.supportedExportFormats"
                },
                {
                    "policyIds": [
                        "gdpr-article-20"
                    ],
                    "name": "Transfer safeguards enabled",
                    "status": "fail",
                    "details": "Monitoring and authorization protocols are missing",
                    "source": "securityAndOperations"
                },
                {
                    "policyIds": [
                        "gdpr-article-20"
                    ],
                    "name": "Transfer tool available",
                    "status": "pass",
                    "details": "Declared",
                    "source": "securityAndOperations.usesDataTransferTool"
                },
                {
                    "policyIds": [
                        "gdpr-article-20"
                    ],
                    "name": "No harmful impact signal",
                    "status": "pass",
                    "details": "No harmful signal detected",
                    "source": "characteristics"
                }
            ],
            "nextActions": {
                "review": [
                    "securityAndOperations.monitorsSecurityActivity",
                    "securityAndOperations.usesAuthorizationProtocols"
                ]
            }
        }
    ]
}
```