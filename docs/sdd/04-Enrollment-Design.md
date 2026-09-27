# Enrollment Design

## 1. Overview

This document details the enrollment process, validation rules, and implementation strategy for the KISS Vocational Training Management Portal.

## 2. Enrollment Process Flow

```
Enrollment Window Configured
    ↓
Admin Activates Enrollment Window
    ↓
Enrollment Window Start Date Reached
    ↓
Student navigates to Enrollment
    ↓
System checks if window is active and student is in roll range
    ↓
Display available courses
    ↓
Student selects course
    ↓
Student confirms enrollment
    ↓
Validation checks:
├─ Is enrollment window active?
├─ Is student within roll-number range?
├─ Is student already enrolled in this course?
├─ Is course at capacity?
└─ Is student eligible?
    ↓
All checks pass
    ↓
Create Enrollment record
    ↓
Save to database
    ↓
Increment current_enrollment count
    ↓
Send confirmation email
    ↓
Display success message
    ↓
Course appears in student dashboard
```

## 3. Enrollment Window Management

### 3.1 Window Configuration

**Admin Creates Window**:
```
POST /api/admin/enrollment-windows
{
  "courseOfferingId": 1,
  "windowStart": "2026-01-15",
  "windowEnd": "2026-01-25",
  "rollNumberStart": "001",
  "rollNumberEnd": "100"
}
```

**Database Entry**:
```
enrollment_windows:
- window_id: 1
- course_offering_id: 1
- window_start: 2026-01-15
- window_end: 2026-01-25
- roll_number_start: "001"
- roll_number_end: "100"
- is_active: false
```

### 3.2 Window Activation

**Admin Activates Window**:
```
PATCH /api/admin/enrollment-windows/1/activate
```

**Process**:
1. Admin confirms window dates
2. Confirms roll-number ranges
3. Activates window (is_active = true)
4. Sends notification to students

### 3.3 Window Closure

**Admin Closes Window**:
```
PATCH /api/admin/enrollment-windows/1/deactivate
```

**Process**:
1. Set is_active = false
2. Prevents new enrollments
3. Prepare for allocation run

---

## 4. Staggered Enrollment

### 4.1 Multiple Windows

**Example Setup**:
```
Course: Java Programming (max capacity: 300)

Window 1: Jan 15 - Jan 18 (Roll 001-100)
Window 2: Jan 19 - Jan 22 (Roll 101-200)
Window 3: Jan 23 - Jan 25 (Roll 201-300)
```

### 4.2 Roll Number Range Validation

**Validation Logic**:
```java
private boolean isStudentInWindowRange(String studentRoll, 
                                       String windowStart, 
                                       String windowEnd) {
    int roll = Integer.parseInt(studentRoll);
    int start = Integer.parseInt(windowStart);
    int end = Integer.parseInt(windowEnd);
    
    return roll >= start && roll <= end;
}
```

**Example**:
- Student Roll: "042"
- Window 1 Range: "001" - "100"
- Result: Eligible (42 >= 1 && 42 <= 100) ✓

---

## 5. Enrollment Validation Rules

### 5.1 Pre-Enrollment Checks

**Check 1: Window Active**
```
SELECT is_active FROM enrollment_windows 
WHERE window_id = ? AND window_start <= CURRENT_DATE AND window_end >= CURRENT_DATE;
```

**Error**: "Enrollment window is not currently active"

**Check 2: Student in Range**
```
SELECT s.roll_number FROM students s 
WHERE s.student_id = ? AND 
      s.roll_number >= ew.roll_number_start AND 
      s.roll_number <= ew.roll_number_end;
```

**Error**: "Your roll number is not eligible for this enrollment window"

**Check 3: Not Already Enrolled**
```
SELECT enrollment_id FROM enrollments 
WHERE student_id = ? AND course_offering_id = ?;
```

**Error**: "You are already enrolled in this course"

**Check 4: Capacity Available**
```
SELECT (co.max_capacity - co.current_enrollment) as slots_available
FROM course_offerings co 
WHERE course_offering_id = ?;
```

**Error**: "Course is at capacity"

**Check 5: Student Eligible**
```
-- Check prerequisites (if applicable)
-- Check academic standing
-- Check other eligibility criteria
```

---

## 6. Enrollment Data Model

### 6.1 Enrollments Table Schema

```sql
CREATE TABLE enrollments (
  enrollment_id SERIAL PRIMARY KEY,
  student_id INT NOT NULL REFERENCES students(student_id),
  course_offering_id INT NOT NULL REFERENCES course_offerings(course_offering_id),
  enrollment_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  status VARCHAR(50) DEFAULT 'ACTIVE',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(student_id, course_offering_id)
);
```

### 6.2 Enrollment Status

| Status | Meaning | Transition |
|--------|---------|-----------|
| ACTIVE | Enrolled, attending | → DROPPED, COMPLETED |
| DROPPED | Student withdrew | Terminal |
| COMPLETED | Course finished | Terminal |

---

## 7. Enrollment API

### 7.1 Submit Enrollment

**Endpoint**: `POST /api/student/enroll`

**Request**:
```json
{
  "courseOfferingId": 1
}
```

**Response** (201 Created):
```json
{
  "enrollmentId": 42,
  "studentId": 101,
  "courseOfferingId": 1,
  "courseName": "Java Programming",
  "semester": "Spring 2026",
  "enrollmentDate": "2026-01-15T10:30:45Z",
  "status": "ACTIVE"
}
```

### 7.2 Get Student Enrollments

**Endpoint**: `GET /api/student/enrollments`

**Response**:
```json
[
  {
    "enrollmentId": 42,
    "courseId": 1,
    "courseName": "Java Programming",
    "courseCode": "VOCA-001",
    "semester": "Spring 2026",
    "status": "ACTIVE",
    "enrollmentDate": "2026-01-15T10:30:45Z"
  },
  {
    "enrollmentId": 43,
    "courseId": 2,
    "courseName": "Web Development",
    "courseCode": "VOCA-002",
    "semester": "Spring 2026",
    "status": "DROPPED",
    "enrollmentDate": "2026-01-16T14:20:00Z"
  }
]
```

### 7.3 Get Active Courses for Enrollment

**Endpoint**: `GET /api/student/available-courses`

**Response**:
```json
{
  "enrollmentWindowActive": true,
  "studentRoll": "042",
  "eligibleCourses": [
    {
      "courseOfferingId": 1,
      "courseName": "Java Programming",
      "courseCode": "VOCA-001",
      "maxCapacity": 100,
      "currentEnrollment": 85,
      "slotsAvailable": 15,
      "semester": "Spring 2026"
    }
  ],
  "windowStartDate": "2026-01-15",
  "windowEndDate": "2026-01-25"
}
```

---

## 8. Implementation Strategy

### 8.1 Service Layer

```java
@Service
@Transactional
public class EnrollmentService {
    
    @Autowired
    private EnrollmentRepository enrollmentRepo;
    
    @Autowired
    private CourseOfferingRepository courseOfferingRepo;
    
    @Autowired
    private EnrollmentWindowRepository windowRepo;
    
    @Autowired
    private EmailService emailService;
    
    public EnrollmentResponse submitEnrollment(Integer studentId, 
                                               Integer courseOfferingId) 
        throws EnrollmentException {
        
        // Validate enrollment window
        validateEnrollmentWindow(studentId, courseOfferingId);
        
        // Check capacity
        checkCapacity(courseOfferingId);
        
        // Check duplicate
        checkDuplicateEnrollment(studentId, courseOfferingId);
        
        // Create enrollment
        Enrollment enrollment = new Enrollment();
        enrollment.setStudentId(studentId);
        enrollment.setCourseOfferingId(courseOfferingId);
        enrollment.setStatus("ACTIVE");
        
        // Save
        Enrollment saved = enrollmentRepo.save(enrollment);
        
        // Update capacity
        courseOfferingRepo.incrementEnrollment(courseOfferingId);
        
        // Send email
        Student student = studentRepo.findById(studentId).orElseThrow();
        CourseOffering course = courseOfferingRepo.findById(courseOfferingId)
                                                   .orElseThrow();
        emailService.sendEnrollmentConfirmation(student.getEmail(), course);
        
        return new EnrollmentResponse(saved);
    }
    
    private void validateEnrollmentWindow(Integer studentId, 
                                          Integer courseOfferingId) 
        throws EnrollmentException {
        
        EnrollmentWindow window = windowRepo
            .findByCourseOfferingId(courseOfferingId)
            .orElseThrow(() -> new EnrollmentException("No window found"));
        
        // Check dates
        LocalDate today = LocalDate.now();
        if (today.isBefore(window.getWindowStart()) || 
            today.isAfter(window.getWindowEnd())) {
            throw new EnrollmentException("Enrollment window not active");
        }
        
        // Check roll number range
        Student student = studentRepo.findById(studentId)
                                     .orElseThrow();
        if (!isInRange(student.getRollNumber(), 
                      window.getRollNumberStart(),
                      window.getRollNumberEnd())) {
            throw new EnrollmentException("Not eligible for this window");
        }
    }
    
    private void checkCapacity(Integer courseOfferingId) 
        throws EnrollmentException {
        
        CourseOffering course = courseOfferingRepo.findById(courseOfferingId)
                                                   .orElseThrow();
        if (course.getCurrentEnrollment() >= course.getMaxCapacity()) {
            throw new EnrollmentException("Course at capacity");
        }
    }
    
    private void checkDuplicateEnrollment(Integer studentId, 
                                          Integer courseOfferingId) 
        throws EnrollmentException {
        
        if (enrollmentRepo.existsByStudentIdAndCourseOfferingId(
            studentId, courseOfferingId)) {
            throw new EnrollmentException("Already enrolled in this course");
        }
    }
}
```

### 8.2 Controller

```java
@RestController
@RequestMapping("/api/student")
public class StudentController {
    
    @Autowired
    private EnrollmentService enrollmentService;
    
    @PostMapping("/enroll")
    public ResponseEntity<?> submitEnrollment(
        @RequestBody EnrollmentRequest request,
        Authentication auth) {
        
        try {
            Integer studentId = getStudentIdFromAuth(auth);
            EnrollmentResponse response = enrollmentService
                .submitEnrollment(studentId, request.getCourseOfferingId());
            return ResponseEntity.status(HttpStatus.CREATED).body(response);
        } catch (EnrollmentException e) {
            return ResponseEntity.status(HttpStatus.BAD_REQUEST)
                                .body(new ErrorResponse(e.getMessage()));
        }
    }
}
```

---

## 9. Transaction Management

### 9.1 Transaction Boundaries

**Enrollment Transaction**:
```
BEGIN TRANSACTION
  ├─ Validate window
  ├─ Check capacity
  ├─ Check duplicate
  ├─ Create enrollment
  ├─ Update course offering enrollment count
  └─ END TRANSACTION (commit if all succeed, rollback if any fail)
```

**Isolation Level**: READ_COMMITTED (prevent dirty reads)

### 9.2 Concurrent Enrollments

**Scenario**: Two students try to enroll simultaneously when 1 seat left

**Handling**:
1. Student A acquires lock on course_offerings row
2. Student A checks capacity: 99/100
3. Student A creates enrollment
4. Student A increments count to 100
5. Student A commits
6. Student B acquires lock
7. Student B checks capacity: 100/100
8. Student B gets "at capacity" error

---

## 10. Error Handling

### 10.1 Error Codes

| Code | Message | Status |
|------|---------|--------|
| ENR_001 | Enrollment window not active | 400 |
| ENR_002 | Not eligible for this window | 400 |
| ENR_003 | Already enrolled in course | 409 |
| ENR_004 | Course at capacity | 400 |
| ENR_005 | Invalid course offering | 404 |
| ENR_006 | Student not found | 404 |
| ENR_007 | Enrollment failed | 500 |

---

## 11. Enrollment Cancellation

### 11.1 Drop Course

**Endpoint**: `DELETE /api/student/enrollments/{enrollmentId}`

**Process**:
1. Validate ownership (student owns enrollment)
2. Check if within drop deadline
3. Update status to DROPPED
4. Decrement course offering enrollment count
5. Send confirmation email

**Validation**:
```
if (enrollment.getEnrollmentDate().plus(14, ChronoUnit.DAYS).isAfter(today)) {
    // Within 2-week drop deadline
} else {
    throw new Exception("Cannot drop after 2 weeks");
}
```

---

## 12. Enrollment Report

**Admin Report**: Enrollment Status

```
Course: Java Programming (VOCA-001)
Semester: Spring 2026
Capacity: 100
Current Enrollment: 85
Slots Available: 15

Window 1 (Roll 001-100): 45 students
Window 2 (Roll 101-200): 40 students (in progress)
Window 3 (Roll 201-300): Not started

Enrollment Rate: 85%
Time to Capacity: 3 days (estimated)
```

---

## 13. Testing Strategy

### 13.1 Unit Tests

- Enrollment validation rules
- Window range checking
- Capacity calculation

### 13.2 Integration Tests

- End-to-end enrollment flow
- Window activation/deactivation
- Concurrent enrollment handling

### 13.3 Performance Tests

- Enrollment under peak load (200 concurrent)
- Large course enrollments (1000+ students)
- Window activation with staggered enrollments

---

## 14. Related Documents

- [System Architecture](01-System-Architecture.md)
- [Database Design](02-Database-Design.md)
- [Batch Allocation](05-Batch-Allocation.md)
- [Functional Requirements - Enrollment](../SRS/05-Functional-Requirements.md#fr-stu-003-course-enrollment)
