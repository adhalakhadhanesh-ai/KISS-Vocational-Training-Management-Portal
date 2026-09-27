# Certificate Workflow

## 1. Overview

This document describes the certificate generation, tracking, and distribution workflow for the KISS Vocational Training Management Portal.

## 2. Certificate Generation Flow

```
Teacher marks course as complete
    ↓
POST /api/teacher/course/complete
    ↓
TeacherService.completeCourse(batchId)
    ↓
Validation:
├─ Teacher assigned to batch?
├─ Course end date passed?
└─ Grades published?
    ↓
All checks pass
    ↓
Fetch all students in course
    ↓
For each student:
├─ Generate certificate PDF
├─ Assign unique certificate ID
├─ Save to database
└─ Send email notification
    ↓
Update course status to COMPLETED
    ↓
Certificate available for download
    ↓
Student receives email notification
```

## 3. Certificate Template

### 3.1 Universal Template

```
╔══════════════════════════════════════════════════════════╗
║                                                          ║
║     KIIT VOCATIONAL TRAINING CERTIFICATE               ║
║                                                          ║
║  This certifies that                                    ║
║                                                          ║
║  {STUDENT_NAME}                                         ║
║  Roll Number: {ROLL_NUMBER}                             ║
║                                                          ║
║  has successfully completed the course                 ║
║                                                          ║
║  {COURSE_NAME}                                          ║
║  Batch: {BATCH_NAME}                                    ║
║                                                          ║
║  Instructor: {TEACHER_NAME}                             ║
║  Date of Completion: {COMPLETION_DATE}                  ║
║  Certificate ID: {CERTIFICATE_ID}                       ║
║                                                          ║
║  Issued on: {ISSUE_DATE}                                ║
║                                                          ║
║  ________________________          ________________________ ║
║  Director Signature                Registrar Signature ║
║                                                          ║
╚══════════════════════════════════════════════════════════╝
```

### 3.2 Dynamic Fields

| Field | Source | Example |
|-------|--------|---------|
| {STUDENT_NAME} | students.name | "Rahul Sharma" |
| {ROLL_NUMBER} | students.roll_number | "042" |
| {COURSE_NAME} | courses.course_name | "Java Programming" |
| {BATCH_NAME} | batches.batch_name | "Batch A" |
| {TEACHER_NAME} | teachers.name | "Prof. Smith" |
| {COMPLETION_DATE} | courses end date | "15 February 2026" |
| {CERTIFICATE_ID} | certificates.cert_number | "KISS-2026-000042" |
| {ISSUE_DATE} | certificates.issue_date | "16 February 2026" |

## 4. Certificate ID Generation

### 4.1 Format

```
KISS-{YEAR}-{SEQUENTIAL_NUMBER}
```

**Example**:
```
KISS-2026-000042
KISS-2026-000043
KISS-2026-000044
```

### 4.2 Generation Algorithm

```java
public String generateCertificateId(Integer year) {
    // Get max certificate ID for year
    Integer maxNumber = certificateRepo.getMaxSequentialByYear(year)
                                      .orElse(0);
    
    // Increment and format
    Integer nextNumber = maxNumber + 1;
    String formatted = String.format("KISS-%d-%06d", year, nextNumber);
    
    return formatted;
}
```

### 4.3 Uniqueness

- Certificate ID must be globally unique
- Database constraint: `UNIQUE(cert_number)`
- Format ensures uniqueness (sequential per year)
- Non-reusable (never reissued)

## 5. API Design

### 5.1 Mark Course Complete

**Endpoint**: `POST /api/teacher/course/{batchId}/complete`

**Request**:
```json
{
  "batchId": 1
}
```

**Response** (200 OK):
```json
{
  "batchId": 1,
  "courseOfferingId": 1,
  "courseName": "Java Programming",
  "status": "COMPLETED",
  "studentCount": 30,
  "certificatesGenerated": 30,
  "completedAt": "2026-02-15T10:30:00Z",
  "message": "Course completed. Certificates generated for all 30 students."
}
```

### 5.2 Get Student Certificates

**Endpoint**: `GET /api/student/certificates`

**Response**:
```json
[
  {
    "certificateId": 1,
    "certNumber": "KISS-2026-000042",
    "courseName": "Java Programming",
    "courseCode": "VOCA-001",
    "completionDate": "2026-02-15",
    "issueDate": "2026-02-16",
    "teacher": "Prof. Smith",
    "batch": "Batch A",
    "downloadUrl": "/api/student/certificates/1/download",
    "verificationUrl": "https://verify.kiit.edu/cert/KISS-2026-000042"
  }
]
```

### 5.3 Download Certificate

**Endpoint**: `GET /api/student/certificates/{certificateId}/download`

**Response**: PDF file (application/pdf)

**Process**:
1. Verify ownership (student owns certificate)
2. Generate PDF from template with fields
3. Stream to client
4. Log download in audit trail

### 5.4 Verify Certificate

**Endpoint**: `GET /api/certificates/verify/{certNumber}`

**Response**:
```json
{
  "valid": true,
  "certNumber": "KISS-2026-000042",
  "studentName": "Rahul Sharma",
  "rollNumber": "042",
  "courseName": "Java Programming",
  "issueDate": "2026-02-16",
  "issueStatus": "VALID"
}
```

## 6. Implementation

### 6.1 Service Layer

```java
@Service
@Transactional
public class CertificateService {
    
    @Autowired
    private CertificateRepository certificateRepo;
    
    @Autowired
    private BatchRepository batchRepo;
    
    @Autowired
    private AllocationRepository allocationRepo;
    
    @Autowired
    private StudentRepository studentRepo;
    
    @Autowired
    private PDFGenerationService pdfService;
    
    @Autowired
    private EmailService emailService;
    
    public CourseCompletionResponse completeCourse(Integer batchId) 
        throws CourseCompletionException {
        
        // Validate batch
        Batch batch = batchRepo.findById(batchId)
                              .orElseThrow(() -> 
                                  new CourseCompletionException("Batch not found"));
        
        // Fetch all allocations in batch
        List<Allocation> allocations = allocationRepo.findByBatchId(batchId);
        
        // Generate certificates
        List<Certificate> certificates = new ArrayList<>();
        for (Allocation allocation : allocations) {
            Certificate cert = generateCertificate(allocation, batch);
            certificates.add(cert);
        }
        
        // Save certificates
        certificateRepo.saveAll(certificates);
        
        // Send emails
        sendCertificateEmails(certificates);
        
        // Update batch status
        batch.setStatus("COMPLETED");
        batchRepo.save(batch);
        
        return new CourseCompletionResponse(batch, certificates.size());
    }
    
    private Certificate generateCertificate(Allocation allocation, Batch batch) 
        throws CourseCompletionException {
        
        Student student = studentRepo.findById(allocation.getStudentId())
                                     .orElseThrow();
        
        // Generate unique certificate ID
        String certId = generateCertificateId(LocalDate.now().getYear());
        
        // Create certificate record
        Certificate cert = new Certificate();
        cert.setCertNumber(certId);
        cert.setStudentId(allocation.getStudentId());
        cert.setAllocationId(allocation.getAllocationId());
        cert.setBatchId(batch.getBatchId());
        cert.setTeacherId(batch.getTeacherId());
        cert.setIssueDate(LocalDate.now());
        cert.setCompletionDate(LocalDate.now());
        
        return cert;
    }
    
    private String generateCertificateId(Integer year) {
        Integer maxNumber = certificateRepo
            .getMaxSequentialByYear(year)
            .orElse(0);
        
        Integer nextNumber = maxNumber + 1;
        return String.format("KISS-%d-%06d", year, nextNumber);
    }
    
    private void sendCertificateEmails(List<Certificate> certificates) {
        certificates.forEach(cert -> {
            Student student = studentRepo.findById(cert.getStudentId())
                                        .orElseThrow();
            emailService.sendCertificateNotification(
                student.getEmail(),
                cert.getCertNumber()
            );
        });
    }
    
    public byte[] downloadCertificate(Integer certificateId, 
                                      Integer currentUserId) 
        throws CertificateException {
        
        Certificate cert = certificateRepo.findById(certificateId)
                                          .orElseThrow();
        
        // Verify ownership
        Student student = studentRepo.findById(cert.getStudentId())
                                     .orElseThrow();
        
        User currentUser = userRepo.findById(currentUserId).orElseThrow();
        if (!currentUser.getRole().equals("ADMIN") && 
            !student.getUserId().equals(currentUserId)) {
            throw new CertificateException("Access denied");
        }
        
        // Generate PDF
        return pdfService.generateCertificatePDF(cert);
    }
}
```

### 6.2 PDF Generation

```java
@Service
public class PDFGenerationService {
    
    public byte[] generateCertificatePDF(Certificate certificate) {
        // Fetch all data
        Student student = studentRepo.findById(certificate.getStudentId())
                                     .orElseThrow();
        Batch batch = batchRepo.findById(certificate.getBatchId())
                              .orElseThrow();
        Teacher teacher = teacherRepo.findById(certificate.getTeacherId())
                                     .orElseThrow();
        CourseOffering course = courseOfferingRepo
            .findById(batch.getCourseOfferingId())
            .orElseThrow();
        Course baseCourse = courseRepo.findById(course.getCourseId())
                                     .orElseThrow();
        
        // Create PDF document
        Document document = new Document(PageSize.A4);
        ByteArrayOutputStream buffer = new ByteArrayOutputStream();
        PdfWriter writer = PdfWriter.getInstance(document, buffer);
        
        document.open();
        
        // Add certificate content
        Paragraph title = new Paragraph("KIIT VOCATIONAL TRAINING CERTIFICATE",
                                       createTitleFont());
        title.setAlignment(Paragraph.ALIGN_CENTER);
        document.add(title);
        
        document.add(new Paragraph("\n"));
        
        Paragraph intro = new Paragraph("This certifies that",
                                       createRegularFont());
        intro.setAlignment(Paragraph.ALIGN_CENTER);
        document.add(intro);
        
        // Student name
        Paragraph studentName = new Paragraph(student.getName(),
                                             createBoldFont());
        studentName.setAlignment(Paragraph.ALIGN_CENTER);
        document.add(studentName);
        
        // Roll number
        Paragraph rollNo = new Paragraph(
            "Roll Number: " + student.getRollNumber(),
            createRegularFont());
        rollNo.setAlignment(Paragraph.ALIGN_CENTER);
        document.add(rollNo);
        
        document.add(new Paragraph("\n"));
        
        Paragraph completion = new Paragraph(
            "has successfully completed the course",
            createRegularFont());
        completion.setAlignment(Paragraph.ALIGN_CENTER);
        document.add(completion);
        
        // Course name
        Paragraph courseName = new Paragraph(baseCourse.getCourseName(),
                                            createBoldFont());
        courseName.setAlignment(Paragraph.ALIGN_CENTER);
        document.add(courseName);
        
        // Batch
        Paragraph batchInfo = new Paragraph("Batch: " + batch.getBatchName(),
                                           createRegularFont());
        batchInfo.setAlignment(Paragraph.ALIGN_CENTER);
        document.add(batchInfo);
        
        // Teacher
        Paragraph teacherInfo = new Paragraph(
            "Instructor: " + teacher.getName(),
            createRegularFont());
        teacherInfo.setAlignment(Paragraph.ALIGN_CENTER);
        document.add(teacherInfo);
        
        // Dates
        Paragraph dateInfo = new Paragraph(
            "Date of Completion: " + formatDate(certificate.getCompletionDate()),
            createRegularFont());
        dateInfo.setAlignment(Paragraph.ALIGN_CENTER);
        document.add(dateInfo);
        
        // Certificate ID
        Paragraph certId = new Paragraph(
            "Certificate ID: " + certificate.getCertNumber(),
            createRegularFont());
        certId.setAlignment(Paragraph.ALIGN_CENTER);
        document.add(certId);
        
        // Footer
        Paragraph issued = new Paragraph(
            "Issued on: " + formatDate(certificate.getIssueDate()),
            createSmallFont());
        issued.setAlignment(Paragraph.ALIGN_CENTER);
        document.add(issued);
        
        document.close();
        
        return buffer.toByteArray();
    }
    
    private Font createTitleFont() {
        return new Font(Font.FontFamily.HELVETICA, 24, Font.BOLD);
    }
    
    private Font createBoldFont() {
        return new Font(Font.FontFamily.HELVETICA, 14, Font.BOLD);
    }
    
    private Font createRegularFont() {
        return new Font(Font.FontFamily.HELVETICA, 12, Font.NORMAL);
    }
    
    private Font createSmallFont() {
        return new Font(Font.FontFamily.HELVETICA, 10, Font.NORMAL);
    }
    
    private String formatDate(LocalDate date) {
        DateTimeFormatter formatter = DateTimeFormatter
            .ofPattern("dd MMMM yyyy")
            .withLocale(Locale.ENGLISH);
        return date.format(formatter);
    }
}
```

## 7. Certificate Verification

### 7.1 Public Verification Portal

```
User navigates to: https://example.com/verify
    ↓
Enters certificate number: KISS-2026-000042
    ↓
System queries database by cert_number
    ↓
If found:
├─ Display certificate details
├─ Show student name, course, date
└─ Confirm validity
    ↓
If not found:
└─ Display "Certificate not found"
```

### 7.2 Verification Endpoint

```java
@GetMapping("/verify/{certNumber}")
@Anonymous // No authentication required
public ResponseEntity<?> verifyCertificate(
    @PathVariable String certNumber) {
    
    Certificate cert = certificateRepo.findByCertNumber(certNumber)
                                      .orElseThrow(() ->
                                          new EntityNotFoundException(
                                              "Certificate not found"));
    
    return ResponseEntity.ok(new CertificateVerificationDTO(cert));
}
```

## 8. Batch Certificate Generation

### 8.1 Performance

**Test Results** (expected):
- 30 certificates: < 2 seconds
- 100 certificates: < 5 seconds
- 500 certificates: < 20 seconds

**Optimization**:
- Batch database inserts
- Async PDF generation
- Email job queue

## 9. Error Scenarios

### 9.1 Certificate Generation Fails

**Scenario**: PDF generation fails mid-process

**Handling**:
```
Exception thrown:
  "Certificate generation failed for student {id}"
  
Admin action:
1. Review error logs
2. Manually retry certificate generation
3. Or regenerate for all students
```

### 9.2 Email Delivery Fails

**Scenario**: Email server unavailable

**Handling**:
```
Fallback:
1. Save error in database
2. Retry via background job (exponential backoff)
3. Notify admin if repeated failures
4. Student can still download from dashboard
```

## 10. Certificate Reissuance

### 10.1 Duplicate Certificate Request

**Scenario**: Student requests duplicate certificate

**Process**:
```
Student submits reissuance request
    ↓
Admin reviews request
    ↓
If approved:
├─ Generate new PDF with same cert ID
├─ Update reissue_count in database
└─ Send email
    ↓
If rejected:
└─ Deny with reason
```

### 10.2 Implementation

```java
public byte[] reissueCertificate(Integer certificateId) {
    Certificate cert = certificateRepo.findById(certificateId)
                                      .orElseThrow();
    
    // Increment reissue counter
    cert.setReissueCount(cert.getReissueCount() + 1);
    certificateRepo.save(cert);
    
    // Generate PDF (same cert number)
    return pdfGenerationService.generateCertificatePDF(cert);
}
```

## 11. Certificate Revocation

### 11.1 Revocation Process

**Scenario**: Certificate needs to be revoked (academic misconduct)

**Process**:
```
Admin marks certificate as revoked
    ↓
Database update: status = 'REVOKED'
    ↓
Verification endpoint returns: "Certificate revoked"
    ↓
Student cannot download
    ↓
Audit log records revocation
```

---

## 12. Related Documents

- [System Architecture](01-System-Architecture.md)
- [Database Design](02-Database-Design.md)
- [Functional Requirements - Certificates](../SRS/05-Functional-Requirements.md#fr-stu-007-certificate-download)
