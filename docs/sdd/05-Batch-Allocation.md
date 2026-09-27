# Batch Allocation Design

## 1. Overview

This document describes the batch allocation process, algorithm, and implementation for assigning students to course batches after enrollment closure.

## 2. Allocation Process Flow

```
Enrollment Window Ends
    ↓
Admin closes enrollment window
    ↓
Admin navigates to Allocation section
    ↓
Admin confirms allocation run
    ↓
AllocationService.runAllocation(courseOfferingId)
    ↓
Process Steps:
├─ Fetch all enrollments
├─ Verify sufficient capacity
├─ Shuffle enrollments randomly
├─ Assign students to batches
├─ Create allocation records
└─ Generate report
    ↓
Send allocation emails to all students
    ↓
Update student dashboard
    ↓
Complete with success message
```

## 3. Allocation Algorithm

### 3.1 Random Shuffle Allocation

**Input**: 
- List of enrollments for course offering
- List of batches with capacities
- Random seed (for reproducibility)

**Algorithm**:

```
FUNCTION runAllocation(courseOfferingId, randomSeed):
    1. FETCH all enrollments for courseOfferingId
    2. FETCH all batches for courseOfferingId with capacities
    3. INITIALIZE randomizer with seed
    4. SHUFFLE enrollments list randomly
    5. FOR each batch:
        a. SET current_capacity = batch.batch_capacity
        b. FOR each enrollment in shuffled list:
            i. IF current_capacity > 0:
                - CREATE allocation record (enrollment → batch)
                - DECREMENT current_capacity
                - REMOVE from shuffled list
            ii. ELSE break to next batch
    6. IF any enrollments remain unallocated:
        a. RAISE AllocationException("Insufficient capacity")
    7. GENERATE allocation report
    8. SEND notification emails
    9. RETURN AllocationReport
```

### 3.2 Example Walkthrough

**Setup**:
```
Course: Java Programming
Total Enrollments: 87
Batches:
- Batch A (Capacity: 30, Teacher: Prof. Smith)
- Batch B (Capacity: 30, Teacher: Prof. Johnson)
- Batch C (Capacity: 30, Teacher: Prof. Williams)
Total Capacity: 90
```

**Process**:

1. **Fetch Enrollments**:
   ```
   Enrollments: [E001, E002, E003, ..., E087]
   ```

2. **Randomize** (seed = 12345):
   ```
   Shuffled: [E042, E015, E067, E001, E089, E033, ...]
   ```

3. **Assign to Batch A** (capacity 30):
   ```
   [E042, E015, E067, E001, E089, E033, E022, E056, E044, E011,
    E078, E029, E055, E066, E007, E031, E018, E092, E041, E009,
    E052, E064, E006, E025, E019, E035, E048, E072, E062, E012]
   ```

4. **Assign to Batch B** (capacity 30):
   ```
   [Remaining 27 students from shuffled list + next 3]
   ```

5. **Assign to Batch C** (capacity 30):
   ```
   [Remaining 30 students]
   ```

**Result**:
```
Batch A: 30 students
Batch B: 30 students
Batch C: 27 students
Total: 87 students allocated
```

---

## 4. API Design

### 4.1 Run Allocation

**Endpoint**: `POST /api/admin/allocation/run`

**Request**:
```json
{
  "courseOfferingId": 1,
  "randomSeed": null
}
```

**Response** (201 Created):
```json
{
  "allocationId": 1,
  "courseOfferingId": 1,
  "courseName": "Java Programming",
  "semester": "Spring 2026",
  "enrollmentCount": 87,
  "batchCount": 3,
  "allocationStatus": "SUCCESS",
  "allocations": [
    {
      "batchId": 1,
      "batchName": "Batch A",
      "capacity": 30,
      "assignedCount": 30,
      "utilization": "100%"
    },
    {
      "batchId": 2,
      "batchName": "Batch B",
      "capacity": 30,
      "assignedCount": 30,
      "utilization": "100%"
    },
    {
      "batchId": 3,
      "batchName": "Batch C",
      "capacity": 30,
      "assignedCount": 27,
      "utilization": "90%"
    }
  ],
  "executionTime": "2.345 seconds",
  "randomSeed": 12345,
  "createdAt": "2026-01-25T15:30:00Z"
}
```

### 4.2 Get Allocation Report

**Endpoint**: `GET /api/admin/allocation/{allocationId}`

**Response**:
```json
{
  "allocationId": 1,
  "courseOfferingId": 1,
  "allocationDate": "2026-01-25",
  "batches": [
    {
      "batchId": 1,
      "batchName": "Batch A",
      "teacher": "Prof. Smith",
      "capacity": 30,
      "assignedStudents": 30,
      "students": [
        {
          "studentId": 42,
          "rollNumber": "042",
          "name": "Student A",
          "email": "studentA@example.com"
        },
        ...
      ]
    }
  ]
}
```

---

## 5. Implementation

### 5.1 Service Layer

```java
@Service
@Transactional
public class AllocationService {
    
    @Autowired
    private EnrollmentRepository enrollmentRepo;
    
    @Autowired
    private BatchRepository batchRepo;
    
    @Autowired
    private AllocationRepository allocationRepo;
    
    @Autowired
    private EmailService emailService;
    
    @Autowired
    private AllocationReportService reportService;
    
    public AllocationReport runAllocation(Integer courseOfferingId, 
                                         Long randomSeed) 
        throws AllocationException {
        
        // Fetch enrollments
        List<Enrollment> enrollments = enrollmentRepo
            .findByCourseOfferingId(courseOfferingId);
        
        if (enrollments.isEmpty()) {
            throw new AllocationException("No enrollments found");
        }
        
        // Fetch batches
        List<Batch> batches = batchRepo
            .findByCourseOfferingId(courseOfferingId)
            .stream()
            .sorted(Comparator.comparing(Batch::getBatchId))
            .collect(Collectors.toList());
        
        if (batches.isEmpty()) {
            throw new AllocationException("No batches configured");
        }
        
        // Check capacity
        int totalCapacity = batches.stream()
                                   .mapToInt(Batch::getBatchCapacity)
                                   .sum();
        
        if (enrollments.size() > totalCapacity) {
            throw new AllocationException(
                String.format("Insufficient capacity: %d enrolled, %d available",
                             enrollments.size(), totalCapacity)
            );
        }
        
        // Shuffle enrollments
        Random random = randomSeed != null ? 
                       new Random(randomSeed) : new Random();
        Collections.shuffle(enrollments, random);
        
        // Allocate to batches
        List<Allocation> allocations = allocateToBatches(
            enrollments, batches, random
        );
        
        // Validate allocation
        if (allocations.size() != enrollments.size()) {
            throw new AllocationException("Allocation incomplete");
        }
        
        // Save allocations
        allocationRepo.saveAll(allocations);
        
        // Send emails
        sendAllocationEmails(allocations, courseOfferingId);
        
        // Generate report
        return reportService.generateReport(courseOfferingId, allocations);
    }
    
    private List<Allocation> allocateToBatches(
        List<Enrollment> enrollments,
        List<Batch> batches,
        Random random) {
        
        List<Allocation> allocations = new ArrayList<>();
        int enrollmentIndex = 0;
        
        for (Batch batch : batches) {
            int capacityRemaining = batch.getBatchCapacity();
            
            while (capacityRemaining > 0 && enrollmentIndex < enrollments.size()) {
                Enrollment enrollment = enrollments.get(enrollmentIndex);
                
                Allocation allocation = new Allocation();
                allocation.setStudentId(enrollment.getStudentId());
                allocation.setBatchId(batch.getBatchId());
                allocation.setStatus("ACTIVE");
                
                allocations.add(allocation);
                
                enrollmentIndex++;
                capacityRemaining--;
            }
        }
        
        return allocations;
    }
    
    private void sendAllocationEmails(
        List<Allocation> allocations,
        Integer courseOfferingId) {
        
        allocations.forEach(allocation -> {
            Student student = studentRepo.findById(allocation.getStudentId())
                                         .orElseThrow();
            Batch batch = batchRepo.findById(allocation.getBatchId())
                                  .orElseThrow();
            CourseOffering course = courseOfferingRepo
                .findById(courseOfferingId)
                .orElseThrow();
            
            emailService.sendAllocationNotification(
                student.getEmail(),
                course,
                batch
            );
        });
    }
}
```

### 5.2 Allocation Repository

```java
@Repository
public interface AllocationRepository extends JpaRepository<Allocation, Integer> {
    
    Optional<Allocation> findByStudentIdAndBatchId(
        Integer studentId, Integer batchId);
    
    List<Allocation> findByBatchId(Integer batchId);
    
    List<Allocation> findByStudentId(Integer studentId);
    
    boolean existsByStudentIdAndBatchId(Integer studentId, Integer batchId);
}
```

---

## 6. Batch Switching

### 6.1 Switch Request Process

```
Student Requests Switch
    ↓
POST /api/student/allocation/{allocationId}/switch-request
{
  "targetBatchId": 3,
  "reason": "Schedule conflict"
}
    ↓
Service validates:
├─ Target batch exists
├─ Target batch has capacity
├─ One switch limit not exceeded
└─ Both in same course
    ↓
Create SwitchRequest record
    ↓
Notification to admin
    ↓
Admin reviews and approves/rejects
    ↓
If approved:
├─ Delete current allocation
├─ Create new allocation
├─ Increment switch count
└─ Notify student
    ↓
If rejected:
├─ Mark request as rejected
└─ Notify student
```

### 6.2 Batch Switching API

**Endpoint**: `POST /api/student/allocation/{allocationId}/switch`

**Request**:
```json
{
  "targetBatchId": 3,
  "reason": "Schedule conflict with work"
}
```

**Response** (201 Created):
```json
{
  "switchRequestId": 1,
  "allocationId": 1,
  "currentBatchId": 1,
  "targetBatchId": 3,
  "status": "PENDING",
  "reason": "Schedule conflict with work",
  "submittedAt": "2026-02-10T10:00:00Z"
}
```

### 6.3 Admin Approval

**Endpoint**: `PATCH /api/admin/switch-requests/{switchRequestId}`

**Request**:
```json
{
  "action": "APPROVE",
  "comment": "Approved - sufficient capacity"
}
```

**Process**:
```java
@Transactional
public void approveSwitchRequest(Integer switchRequestId, String comment) {
    // Fetch switch request
    SwitchRequest request = switchRequestRepo.findById(switchRequestId)
                                            .orElseThrow();
    
    // Check target batch capacity
    if (!hasBatchCapacity(request.getTargetBatchId())) {
        throw new Exception("Target batch at capacity");
    }
    
    // Delete current allocation
    allocationRepo.deleteById(request.getAllocationId());
    
    // Create new allocation
    Allocation newAllocation = new Allocation();
    newAllocation.setStudentId(request.getAllocationId()); // from current
    newAllocation.setBatchId(request.getTargetBatchId());
    newAllocation.setStatus("ACTIVE");
    allocationRepo.save(newAllocation);
    
    // Update switch count
    // Increment switch_count in allocations table
    
    // Update request
    request.setStatus("APPROVED");
    request.setApprovedDate(LocalDateTime.now());
    switchRequestRepo.save(request);
    
    // Send email
    emailService.sendSwitchApprovalNotification(...);
}
```

---

## 7. Allocation Report

### 7.1 Report Contents

```
ALLOCATION REPORT
Course: Java Programming (VOCA-001)
Semester: Spring 2026
Date: 2026-01-25

SUMMARY
Total Enrollments: 87
Total Batches: 3
Total Capacity: 90
Allocation Status: SUCCESS
Execution Time: 2.345 seconds

BATCH DETAILS
Batch A:
  Capacity: 30
  Assigned: 30
  Utilization: 100%
  Teacher: Prof. Smith
  Timing: 10:00 AM - 12:00 PM

Batch B:
  Capacity: 30
  Assigned: 30
  Utilization: 100%
  Teacher: Prof. Johnson
  Timing: 12:00 PM - 2:00 PM

Batch C:
  Capacity: 30
  Assigned: 27
  Utilization: 90%
  Teacher: Prof. Williams
  Timing: 2:00 PM - 4:00 PM

RANDOM SEED: 12345
(Allocation is reproducible with this seed)
```

---

## 8. Error Scenarios

### 8.1 Insufficient Capacity

**Scenario**: 100 enrollments, 90 total capacity

**Handling**:
```
AllocationException: 
  "Insufficient capacity: 100 enrolled, 90 available"
  
Admin options:
1. Increase batch capacities
2. Create additional batches
3. Waitlist extra students
4. Defer allocation
```

### 8.2 Empty Enrollments

**Scenario**: No enrollments for course

**Handling**:
```
AllocationException: "No enrollments found"

Possible causes:
- Enrollment window not opened
- No students enrolled
```

---

## 9. Allocation Verification

### 9.1 Consistency Checks

```
AFTER allocation:
1. Every enrolled student has exactly one allocation ✓
2. No batch exceeds capacity ✓
3. All batches in same course offering ✓
4. No duplicate (student, batch) pairs ✓
5. All students have status = 'ACTIVE' ✓
```

### 9.2 Audit Trail

```
Allocation Event Log:
- Timestamp: 2026-01-25 15:30:00 IST
- User: admin@example.com
- Action: RUN_ALLOCATION
- Course Offering: 1 (Java Programming)
- Enrollments: 87
- Batches: 3
- Random Seed: 12345
- Status: SUCCESS
- Duration: 2.345 seconds
```

---

## 10. Performance Considerations

### 10.1 Scalability

**Test Results** (expected):
- 100 enrollments: < 100 ms
- 1000 enrollments: < 500 ms
- 5000 enrollments: < 2 seconds

**Optimization**:
- Batch insert for allocations
- Database indexes on enrollment and batch IDs
- Avoid N+1 queries

---

## 11. Related Documents

- [System Architecture](01-System-Architecture.md)
- [Database Design](02-Database-Design.md)
- [Enrollment Design](04-Enrollment-Design.md)
- [Functional Requirements - Allocation](../SRS/05-Functional-Requirements.md#fr-adm-010-run-allocation)

---

## 12. Algorithm Justification

See [ADR-002: Random Allocation](../Architecture/ADR-002-RandomAllocation.md) for detailed justification of why random allocation was chosen over preference-based allocation.
