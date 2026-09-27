# Performance Test Cases

## Overview

Expands [Test-Plan §7](Test-Plan.md#7-performance-testing) into concrete performance test cases mapped to the targets in [Non-Functional Requirements §1](../SRS/06-Non-Functional-Requirements.md#1-performance-requirements).

## 1. Response Time (NFR-PER-001)

| ID | Title | Target | Method | Priority |
|----|-------|--------|--------|----------|
| TC-PERF-RT-001 | Student dashboard load | ≤ 2s | Measure server response time for `GET /student/dashboard` under normal load (10+ Mbps simulated) | Critical |
| TC-PERF-RT-002 | Admin dashboard load | ≤ 3s | Measure `GET /admin/dashboard` (aggregate trackers) | Critical |
| TC-PERF-RT-003 | Course/student list pages | ≤ 3s | Measure `GET /admin/course-offerings`, `GET /admin/users` | High |
| TC-PERF-RT-004 | Form submission | ≤ 1s | Measure `POST /student/enroll`, `POST /teacher/attendance` | High |
| TC-PERF-RT-005 | Report generation | ≤ 5s | Measure `GET /admin/reports?type=student&format=csv` for 1000 students | Medium |
| TC-PERF-RT-006 | Progress module aggregation | ≤ 2s | Measure `GET /student/progress/{courseId}` (per [Progress Module §6](../SDD/09-Progress-Module.md#6-update-latency)) | High |

## 2. Throughput & Concurrency (NFR-PER-002)

| ID | Title | Target | Method | Priority |
|----|-------|--------|--------|----------|
| TC-PERF-LOAD-001 | 200 concurrent users, mixed workload | No errors, response times stay within NFR-PER-001 targets | Load test simulating 200 concurrent sessions across student/teacher/admin flows | Critical |
| TC-PERF-LOAD-002 | Concurrent enrollment during window open | No lost/duplicate enrollments, capacity respected | Simulate 200 students enrolling into a 50-capacity offering simultaneously at window open | Critical |
| TC-PERF-LOAD-003 | Concurrent attendance submission | All submissions saved correctly, no race conditions on unique `(allocation_id, attendance_date)` | Simulate 10 teachers submitting attendance for different batches simultaneously | Medium |
| TC-PERF-LOAD-004 | Sustained load, 30-minute soak | No memory growth, no error-rate increase over time | Sustain 100 concurrent users for 30 minutes | Medium |

## 3. Batch Processing Performance (NFR-PER-004)

| ID | Title | Target | Method | Priority |
|----|-------|--------|--------|----------|
| TC-PERF-BATCH-001 | Allocation run, 1000 students | ≤ 5 minutes | Run allocation against a seeded dataset of 1000 enrollments | Critical |
| TC-PERF-BATCH-002 | Certificate generation rate | ≥ 100 certificates/minute | Trigger course completion for a 1000-student batch, measure generation throughput | Critical |
| TC-PERF-BATCH-003 | Certificate generation, 1000 students | ≤ 10 minutes total | End-to-end from course completion to all PDFs available | High |
| TC-PERF-BATCH-004 | Report generation, full dataset | ≤ 5 minutes | Generate a certificate report across a full academic year of records | Medium |
| TC-PERF-BATCH-005 | Off-peak scheduling respected | Batch jobs execute only 10 PM–6 AM unless manually triggered by an admin | Verify scheduler configuration and a manual-trigger override path | Low |

## 4. Database Query Performance (NFR-PER-003)

| ID | Title | Target | Method | Priority |
|----|-------|--------|--------|----------|
| TC-PERF-DB-001 | Simple lookup query | ≤ 100 ms | `EXPLAIN ANALYZE` on `SELECT * FROM students WHERE student_id = ?` | High |
| TC-PERF-DB-002 | Aggregate query (dashboard tracker) | ≤ 500 ms | `EXPLAIN ANALYZE` on enrollment fill-percentage query ([Admin Dashboard §3](../SDD/08-Admin-Dashboard.md#3-command-center-trackers)) | High |
| TC-PERF-DB-003 | Complex join (progress calculation) | ≤ 1s | `EXPLAIN ANALYZE` on the published-grades join ([Progress Module §4](../SDD/09-Progress-Module.md#4-draft-grade-visibility-rule)) | High |
| TC-PERF-DB-004 | Report query, full semester | ≤ 10s | `EXPLAIN ANALYZE` on the report-generation query for one semester | Medium |
| TC-PERF-DB-005 | Index coverage check | No sequential scans on tables > 1000 rows for the above queries | Review query plans confirm index usage per [Database Design §5](../SDD/02-Database-Design.md#5-indexes) | Medium |

## 5. Scalability

| ID | Title | Target | Method | Priority |
|----|-------|--------|--------|----------|
| TC-PERF-SCALE-001 | Degradation curve | Response times degrade gracefully (no cliff/crash) as load increases | Ramp from 50 to 300 concurrent users, plot response time vs. load | Medium |
| TC-PERF-SCALE-002 | Recovery after peak | System returns to baseline response times within 5 minutes of load drop | Ramp to 300 users, then drop to 20, measure recovery | Low |

## Test Environment Notes

Performance tests run against a **Staging** environment provisioned to match Production specifications (per [Test-Plan §9.2](Test-Plan.md#92-test-environment)), using a seeded dataset that mirrors realistic scale (1000+ students, one full semester of historical records) so results are representative rather than optimistic.

## Related Documents

- [Test Plan](Test-Plan.md)
- [Functional Test Cases](Functional-Test-Cases.md)
- [Non-Functional Requirements](../SRS/06-Non-Functional-Requirements.md)
- [Database Design](../SDD/02-Database-Design.md)
