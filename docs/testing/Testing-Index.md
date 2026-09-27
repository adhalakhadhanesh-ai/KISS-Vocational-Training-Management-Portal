# Testing Documentation Index

## Overview

This is the navigation hub for all QA and testing documentation for the KISS Vocational Training Management Portal.

## Documents in This Folder

| Document | Purpose |
|----------|---------|
| [Test Plan](Test-Plan.md) | Overall testing strategy, scope, levels, environments, and tooling |
| [Functional Test Cases](Functional-Test-Cases.md) | 64 test cases covering all 34 functional requirements |
| [Security Test Cases](Security-Test-Cases.md) | Authentication, authorization, injection, and data-protection test cases |
| [Performance Test Cases](Performance-Test-Cases.md) | Response time, throughput, batch processing, and query performance test cases |

## How These Fit Together

```
Test Plan                    (strategy: what, how, when, in what environment)
    │
    ├── Functional Test Cases    (does it work correctly?)
    ├── Security Test Cases      (is it safe from misuse?)
    └── Performance Test Cases   (is it fast and scalable enough?)
```

Start with [Test Plan](Test-Plan.md) for scope and strategy, then use the three test-case documents as the executable checklists for each testing dimension.

## Traceability

Every test case in this folder cites the functional requirement, module design document, or API contract it verifies. See:
- [Functional Requirements](../SRS/05-Functional-Requirements.md) for the requirements being tested
- [SDD Index](../SDD/SDD-Index.md) for the module designs under test
- [API Index](../API/API-Index.md) for the endpoint contracts under test
- [Error Codes](../API/Error-Codes.md) for the expected error responses referenced throughout

## Related Documents

- [Documentation Home](../../README.md)
- [SRS Index](../SRS/SRS-Index.md)
- [SDD Index](../SDD/SDD-Index.md)
- [API Index](../API/API-Index.md)
