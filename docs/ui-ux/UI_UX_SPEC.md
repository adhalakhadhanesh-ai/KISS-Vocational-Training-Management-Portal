# KISS VMP — UI/UX Specification

**KISS Vocational Management Portal (KISS VMP)** is an institutional, ERP-style web portal that manages the vocational training lifecycle for **Student**, **Teacher** and **Admin** users. V1 targets a workable, end-to-end MVP, not a visually experimental product.

This document is the source of truth for frontend developers. It describes UI/UX and workflows, not backend implementation. An interactive reference prototype exists (see [README](./README.md)); where the two differ, this document wins.

**Tags used in this document**

| Tag | Meaning |
|---|---|
| **[FROZEN V1]** | Fixed requirement |
| **[V2]** | Deferred. Do not build in V1 |
| **Not specified** | The source material does not define this. Needs a team decision before it is built. Do not invent it |

---

## Table of Contents

1. [Design Principles](#1-design-principles)
2. [Visual Language](#2-visual-language)
3. [Navigation Model](#3-navigation-model)
4. [Global Header](#4-global-header)
5. [Sidebar](#5-sidebar)
6. [Home Launcher](#6-home-launcher)
7. [Module Cards and Sub-module Pages](#7-module-cards-and-sub-module-pages)
8. [Page Headings](#8-page-headings)
9. [Tables](#9-tables)
10. [Forms](#10-forms)
11. [System States](#11-system-states)
12. [Confirmation Dialogs](#12-confirmation-dialogs)
13. [Unsaved Changes](#13-unsaved-changes)
14. [Caching](#14-caching)
15. [Responsive Behaviour](#15-responsive-behaviour)
16. [Accessibility](#16-accessibility)
17. [Animation Rules](#17-animation-rules)
18. [Terminology](#18-terminology)
19. [Role Structures](#19-role-structures)
20. [Student Workflows](#20-student-workflows)
21. [Teacher Workflows](#21-teacher-workflows)
22. [Admin Workflows](#22-admin-workflows)
23. [Screen Reference](#23-screen-reference)
24. [V1 / V2 Boundaries](#24-v1--v2-boundaries)
25. [Open Items (Not Specified)](#25-open-items-not-specified)
26. [UI/UX Status](#26-uiux-status)

---

## 1. Design Principles

**[FROZEN V1]**

The portal should feel like a real institutional management portal, similar in structure and usability to a university ERP/SAP system, with cleaner modern UI. Approximate balance: **70% institutional ERP/SAP, 30% modern UI polish.**

The interface is:

- Institutional
- Clean
- Practical
- Compact
- Information-focused
- Easy to navigate
- Desktop-first
- Minimal in visual decoration

It has no unnecessary animation, no excessive dashboards and no startup/SaaS aesthetic.

---

## 2. Visual Language

**[FROZEN V1]**

### 2.1 Colors

| Element | Specification |
|---|---|
| Primary identity | Green + White |
| Header | Green background, white text and icons |
| Workspace | White |
| Cards | White with a subtle border |
| Buttons / clickable actions | Blue may be used selectively. The dominant identity stays green and white |
| Exact color values (hex) | Not specified |

Do **not** use: gradients, glassmorphism, excessive shadows, neon colors, excessive rounded cards, flashy illustrations, animated dashboards.

### 2.2 Typography

Font: **Inter**. Typography is restrained.

| Use | Weight |
|---|---|
| Page heading | SemiBold |
| Welcome text | Medium / SemiBold |
| Card labels | Medium |
| Body / table text | Regular |
| Buttons | Medium / SemiBold |

Font sizes: Not specified. Keep them compact; avoid oversized typography.

### 2.3 Spacing and density

The interface is **compact but not cramped**. Avoid giant hero sections, excessive vertical padding, oversized cards and huge empty spaces. Keep enough spacing for readable, comfortable interaction.

---

## 3. Navigation Model

**[FROZEN V1]**

```
Login
 → Role detected from backend/database
   → Role-specific Home
     → Module
       → Sub-module
         → Feature page
```

- There is **no manual role selection** during login.
- There is **no intermediate "Student Self Service Portal" page**.
- After login the user goes directly to their role-specific Home.
- A module opens either a feature page directly or a page of sub-module cards. Some sub-modules contain a further level (for example Teacher → Mentoring → Requests).

---

## 4. Global Header

**[FROZEN V1]**

```
+-----------------------------------------------------------+
| [☰]  KISS Vocational Management Portal                    |   green bar
+-----------------------------------------------------------+
```

- Used on every authenticated page and visually identical throughout.
- Green background, white text, white hamburger icon.
- Top-left: hamburger icon, then "KISS Vocational Management Portal". The hamburger is always present.
- Do **not** replace it with page-specific headers.
- Do **not** add breadcrumbs.
- Do **not** add a large profile/dashboard header.
- The page title lives inside the workspace, not in the header.
- Login is pre-authentication. Its header treatment (no hamburger, no sidebar) is described in [Login](#login).

---

## 5. Sidebar

**[FROZEN V1]**

| Behaviour | Rule |
|---|---|
| Default state | Hidden |
| Open | Click the hamburger icon. The sidebar appears immediately |
| Layout effect | **Pushes** the workspace to the right. Does **not** overlay it |
| Animation | None. No slide animation |
| Collapse-to-icons mode | Not supported |
| On module selection | Navigate to the module. Sidebar disappears immediately and the workspace expands again |
| Active / selected highlight | Not required. Items remain visually consistent |

Contents: Home, role-specific modules, and Profile/Logout where appropriate. The exact Profile entry per role is Not specified.

This behaviour is the same in V1 and future versions.

---

## 6. Home Launcher

**[FROZEN V1]**

Home is a **module launcher**, not a data-heavy analytics dashboard.

Contents, in order:

1. `Welcome, [Username]!` (for example `Welcome, Dhanesh!`)
2. Module cards

Do **not** put any of these on Home (they belong inside their modules): attendance percentages, current course, grades, certificate status, notifications, statistics, roll number, status widgets.

---

## 7. Module Cards and Sub-module Pages

**[FROZEN V1]**

### 7.1 Card style

| Property | Value |
|---|---|
| Background | White |
| Border | Subtle |
| Shadow | None |
| Gradient | None |
| Hover animation / lift | None |
| Dynamic status badges | None |
| Proportions | Square / tall |
| Icon | Left |
| Label | Beside the icon |
| Dimensions, icon size, spacing | Identical for every card |

```
+--------------------------------+
|  [icon]   My Information       |
|                                |
|                                |
|                                |
+--------------------------------+
```

Cards are clickable but visually static. **The same card component** is used on Home and on every sub-module page.

### 7.2 Grid

- Desktop: **3 columns**, fixed-size cells.
- Cards are placed left to right, then wrap to the next row.
- Fewer cards than columns: show only those cards and leave the rest of the row empty.
- One card: keep it **left-aligned**. Do not center it.
- Cards never stretch because fewer cards exist.

### 7.3 Hierarchy

```
Home → Module → Sub-module → Feature page
```

Example:

```
Progress

[ Grades ]          [ Course Progress ]
```

---

## 8. Page Headings

**[FROZEN V1]**

Each page shows **only the page heading**, then immediately the relevant content.

Do **not** add breadcrumbs, subtitles, explanatory hero sections or large introductory text. Keep pages compact.

---

## 9. Tables

**[FROZEN V1]**

Tables are the primary interface for data management. Used for attendance, gradebook, student lists, enrollment records, course management, grievance lists, leave requests, admin records and reports where appropriate.

Style: clean, compact, readable, institutional, white, bordered.

**Pagination:**

```
Showing 1–25 of 184 students

[Previous] [1] [2] [3] [Next]
```

| Feature | Version |
|---|---|
| Pagination | **[FROZEN V1]** |
| Search | **[V2]** |
| Advanced filters | **[V2]** |
| Advanced table navigation | **[V2]** |

Page size: the example shows 25 rows; the final page size is Not specified.

> **Views are not filters.** Navigation destinations such as Admin → Grievance Management → Students or Teachers are separate pages reached through cards. They are not a V2 search/filter feature, and V1 does not add filter controls.

---

## 10. Forms

Forms are used for submitting grievances, leave applications, feedback and admin configuration.

**[FROZEN V1]**

- Every field has a proper visible label.
- Controls are keyboard accessible with visible focus states.
- The submit button shows an inline loading state while saving (for example `[ Submitting... ]`).
- A successful submission shows a short toast (see [System States](#11-system-states)).
- Normal student submissions do not use a confirmation dialog.

**Not specified:** field lists for every form, validation messages, required-field markers, and which fields are mandatory.

---

## 11. System States

### 11.1 Loading — skeleton

**[FROZEN V1]**

- Page and data loading uses **skeleton loading**. No generic full-page spinners.
- Actions such as saving use a small inline loading indicator inside the button.

```
Course       Attendance
████████     ████████
████████     ████████
████████     ████████

[ Saving... ]
```

### 11.2 Empty

**[FROZEN V1]**

Simple, contextual, no illustrations. Include a CTA only when an action is relevant.

Certificates (no CTA):

```
No certificates available yet.
Your certificate will appear here after successful course completion.
```

Grievance (actionable):

```
You haven't submitted any grievances yet.

[ Submit Grievance ]
```

Empty-state wording for other screens: Not specified. Use the same simple, contextual pattern.

### 11.3 Error

**[FROZEN V1]**

```
Unable to load

Something went wrong while loading this page.

[ Retry ]
```

If there is no data and the request itself failed, show the **error state, not the empty state**. No complicated error-state systems.

### 11.4 Success

**[FROZEN V1]**

A short confirmation message/toast, not a large modal success screen.

- ✓ Enrollment successful.
- ✓ Attendance saved successfully.
- ✓ Grievance submitted successfully.
- ✓ Grades saved successfully.

Success wording for leave actions and other flows: Not specified. Follow the same short pattern.

---

## 12. Confirmation Dialogs

**[FROZEN V1]**

| Role | Rule |
|---|---|
| Student | Normal actions need **no** confirmation dialog (for example Enrollment → action → backend → success/error message) |
| Teacher | Confirmation **required** for important attendance and grade submission actions |
| Admin | Confirmation only for selected important, destructive or high-impact actions. Not everywhere |

```
Confirm Submission

Are you sure you want to submit the attendance?

[Cancel] [Confirm]
```

Whether other teacher or admin actions (for example handling leave requests or grievances) need confirmation: Not specified.

---

## 13. Unsaved Changes

**[FROZEN V1]**

- No auto-save and no drafts for teacher attendance or grades.
- If a teacher edits attendance or grades and leaves without saving, the changes are discarded and must be entered again.
- No unsaved-changes modal in V1.

---

## 14. Caching

**[FROZEN V1]**

| Situation | Behaviour |
|---|---|
| First visit | Fetch data → skeleton → display → cache |
| Return shortly afterwards | Use cached data |
| Cached data stale or unused for a significant time | Fetch fresh data |
| Highly dynamic operations (for example enrollment availability) | Always validate against the backend before the action |

Cache lifetime: Not specified.

---

## 15. Responsive Behaviour

**[FROZEN V1]**

- Desktop-first. Primary target is desktop/laptop.
- No elaborate mobile-first system in V1.
- The layout must not completely break on smaller screens.
- Complex tables may scroll horizontally.
- Better mobile optimisation is **[V2]**.

---

## 16. Accessibility

**[FROZEN V1]**

- Readable text
- Sufficient contrast
- Keyboard-accessible controls
- Visible focus states
- Proper form labels
- Sufficiently large clickable areas
- Icons are never the only way to understand an action

---

## 17. Animation Rules

**[FROZEN V1]**

Extremely minimal.

- No bouncing, parallax, card lifting, hover animations, animated dashboards or decorative motion.
- Sidebar: instant appearance.
- Module navigation: instant transition.
- Buttons may have normal browser/UI interaction feedback, nothing elaborate.

---

## 18. Terminology

Use these terms consistently.

| Term | Meaning |
|---|---|
| **Grievance** | A complaint raised by a Student, Teacher or Admin. Shown as "Grievances / Complaints" in the Teacher mentoring request view |
| **Leave Application** | The Student's term for requesting leave |
| **Apply Leave** (Teacher) | The Teacher's **own** leave |
| **Leave Requests** (Teacher → Mentoring → Requests) | Leave requests from the teacher's **mentees** |
| **Apply Leave** (Admin) | The Admin's **own** leave |
| **Mentoring** | Teacher module for mentee students and their requests |

**Leave is not a grievance.** Leave and grievances are separate flows and are never mixed in the same list.

**Grievance views are not separate entities.** In Admin → Grievance Management, "Grievances", "Students" and "Teachers" are different views of the same grievance data, not different kinds of grievance.

---

## 19. Role Structures

**[FROZEN V1]**

### 19.1 Student

| # | Module | Sub-modules |
|---|---|---|
| 1 | My Information | |
| 2 | Enrollment | |
| 3 | Attendance | Separate top-level module. Do **not** merge into Progress |
| 4 | Progress | Grades, Course Progress / Performance |
| 5 | Certificates | |
| 6 | Grievance | |
| 7 | Feedback | |

The student's **Leave Application** is a defined student flow. Its position in Student navigation (separate module card or elsewhere) is **Not specified**.

### 19.2 Teacher (locked)

```
Teacher Home
├── Manage Course
├── Mentoring
│   ├── My Students
│   └── Requests
│       ├── Grievances / Complaints
│       └── Leave Requests
├── Raise Grievance
└── Apply Leave
```

Manage Course workflow:

```
Manage Course
 → Select Course
   → Select Batch/Section
     → Choose: Attendance | Gradebook
```

Attendance and Gradebook open as table-based interfaces.

> Earlier drafts listed "Student Details" and "Assigned Sections" as Teacher modules. They are **not** part of the locked structure above. Where their content now lives is **Not specified**.

### 19.3 Admin (locked)

```
Admin Home
├── Course Management
├── Enrollment
├── Allocation
├── Students
├── Teachers
├── Grievance Management
│   ├── Grievances
│   ├── Students
│   ├── Teachers
│   └── Add Grievance
├── Feedback
├── Reports
└── Apply Leave
```

Grievance Management:

| Card | Shows |
|---|---|
| Grievances | All incoming Student + Teacher grievances |
| Students | Student-raised grievances |
| Teachers | Teacher-raised grievances |
| Add Grievance | Admin raises their own grievance |

Admin configuration functionality (course seats, batches, teachers, grading schemes, feedback forms, sections) exists. Its exact placement within the admin modules is **Not specified**.

---

## 20. Student Workflows

All student actions are simple, with no confirmation dialogs.

| Workflow | Flow |
|---|---|
| Enrollment | Enrollment → table of available courses → Enroll → backend validates availability → toast `✓ Enrollment successful.` or error |
| Attendance | Attendance → read-only attendance table (Course / Attendance) |
| Progress | Progress → Grades or Course Progress → read-only table |
| Certificates | Certificates → table of certificates, or the empty state if none |
| Grievance | Grievance → table of own grievances, or the empty state with `[ Submit Grievance ]` → form → toast `✓ Grievance submitted successfully.` |
| Leave Application | Student submits a leave application. Form fields, status display and navigation position: Not specified |
| Feedback | Feedback → feedback form → submit → success toast |
| My Information | Read-only information. Editing: Not specified |

Student workflow for mentoring (viewing a mentor, for example): Not specified.

---

## 21. Teacher Workflows

### 21.1 Manage Course — Attendance

1. Manage Course → select course → select batch/section.
2. Choose **Attendance**. An editable attendance table opens.
3. Teacher marks attendance, then submits.
4. Confirmation: `Confirm Submission — Are you sure you want to submit the attendance? [Cancel] [Confirm]`.
5. On confirm: inline `[ Saving... ]`, then toast `✓ Attendance saved successfully.`
6. Leaving without saving discards the edits (no drafts, no modal).

### 21.2 Manage Course — Gradebook

1. Same selection path, then choose **Gradebook**. An editable gradebook table opens.
2. Teacher enters grades and submits.
3. Confirmation dialog, then saving, then toast `✓ Grades saved successfully.`
4. Leaving without saving discards the edits.

Wide gradebooks may scroll horizontally.

### 21.3 Mentoring

```
Mentoring
├── My Students
└── Requests
    ├── Grievances / Complaints
    └── Leave Requests
```

- **My Students:** table of the teacher's mentees.
- **Requests → Grievances / Complaints:** grievances raised by mentees.
- **Requests → Leave Requests:** leave requests from mentees.
- Available actions on requests (for example approve, reject, respond), request statuses and table columns: Not specified.

### 21.4 Raise Grievance

The teacher raises their own grievance. Form fields: Not specified. Success shows a toast. The grievance becomes visible to the Admin under Grievance Management → Teachers (and Grievances).

### 21.5 Apply Leave

The teacher applies for their **own** leave. Form fields and approval flow: Not specified. This is separate from Mentoring → Requests → Leave Requests, which handles **mentee** leave.

---

## 22. Admin Workflows

| Module | Workflow |
|---|---|
| Course Management | Table of courses with create/edit. Includes configuration such as seats, batches, sections and grading schemes (placement Not specified) |
| Enrollment | Table of enrollment records |
| Allocation | Allocation table and actions. Confirmation for the high-impact allocation action |
| Students | Table of student records |
| Teachers | Table of teacher records and teacher configuration |
| Grievance Management | Card page with Grievances, Students, Teachers, Add Grievance (see below) |
| Feedback | Feedback form configuration and feedback records |
| Reports | Report tables. No charts or analytics dashboards |
| Apply Leave | Admin applies for their **own** leave. Fields and approval flow: Not specified |

**Grievance Management**

- **Grievances:** table of all incoming Student + Teacher grievances.
- **Students:** table of Student-raised grievances.
- **Teachers:** table of Teacher-raised grievances.
- **Add Grievance:** the Admin raises their own grievance (form).
- Admin actions on a grievance (respond, change status, close) and statuses: Not specified.

Confirmation dialogs for admin are used only for selected important, destructive or high-impact actions.

---

## 23. Screen Reference

Every authenticated screen inherits these global rules:

```
Header:    Green bar, white text. Hamburger + "KISS Vocational Management Portal". No breadcrumbs.
Sidebar:   Hidden by default. Hamburger shows it instantly and pushes the workspace right.
           Choosing a module hides it instantly. No active highlight.
Workspace: White. Page heading only, then content.
Cards:     Single shared card component, 3-column left-aligned grid.
Tables:    White, bordered, compact, paginated. No search or filters.
States:    Skeleton loading, "Unable to load" + Retry, toast on success, inline button loading.
Font:      Inter.
```

Table columns shown are indicative; final columns are Not specified and follow the data model.

### Login

- **Purpose:** Authenticate the user. The role is detected by the backend.
- **Layout:** White page with a compact login form.
- **Header:** Green bar with the portal name. No hamburger, no sidebar (not authenticated).
- **Components:** Heading "Login", labelled credential fields, `[ Login ]` button.
- **Navigation:** On success, goes to the user's role-specific Home.
- **States:** Inline button loading, inline error for invalid credentials or failure.

### Student screens

| Screen | Purpose | Main content | Actions | Key states |
|---|---|---|---|---|
| **Student Home** | Module launcher | "Welcome, [Username]!" and 7 cards: My Information, Enrollment, Attendance, Progress, Certificates, Grievance, Feedback | Card click opens module | None (no data on Home) |
| **My Information** | Show student details | Read-only label/value block | None specified | Skeleton, error |
| **Enrollment** | Enroll in courses | Table: Course, Availability, Action | `[ Enroll ]` | Skeleton, inline loading, success toast, error, empty |
| **Attendance** | View attendance | Table: Course, Attendance | None | Skeleton, empty, error |
| **Progress** | Sub-module launcher | Cards: Grades, Course Progress | Card click | None |
| **Grades** | View grades | Table: Course, Grade/Result | None | Skeleton, empty, error |
| **Course Progress** | View progress/performance | Table: Course, Progress/Performance | None | Skeleton, empty, error |
| **Certificates** | List certificates | Table of certificates | Per-row view/download where available | Skeleton, empty (no CTA), error |
| **Grievance** | Submit and view grievances | Table of own grievances, submission form | `[ Submit Grievance ]` | Skeleton, actionable empty, success toast, error |
| **Feedback** | Submit feedback | Feedback form | `[ Submit ]` | Skeleton, success toast, error |
| **Leave Application** | Apply for leave | Form. Details Not specified | Submit | Not specified |

### Teacher screens

| Screen | Purpose | Main content | Actions | Key states |
|---|---|---|---|---|
| **Teacher Home** | Module launcher | "Welcome, [Username]!" and 4 cards: Manage Course, Mentoring, Raise Grievance, Apply Leave | Card click | None |
| **Manage Course** | Select course and batch/section | Course selection, batch/section selection, then Attendance and Gradebook cards | Card click | Skeleton, empty, error |
| **Teacher Attendance** | Record attendance | Editable table: Student, Attendance status | `[ Submit ]` then confirm dialog | Skeleton, `[ Saving... ]`, success toast, error, edits discarded on leaving |
| **Teacher Gradebook** | Enter grades | Editable table: Student, grade columns | `[ Submit ]` then confirm dialog | Same as above |
| **Mentoring** | Sub-module launcher | Cards: My Students, Requests | Card click | None |
| **My Students** | View mentees | Table of mentees | None specified | Skeleton, empty, error |
| **Requests** | Sub-module launcher | Cards: Grievances / Complaints, Leave Requests | Card click | None |
| **Grievances / Complaints** | View mentee grievances | Table of grievances | Request actions: Not specified | Skeleton, empty, error |
| **Leave Requests** | View mentee leave requests | Table of leave requests | Request actions: Not specified | Skeleton, empty, error |
| **Raise Grievance** | Teacher raises own grievance | Form | Submit | Inline loading, success toast, error |
| **Apply Leave** | Teacher's own leave | Form. Details Not specified | Submit | Not specified |

### Admin screens

| Screen | Purpose | Main content | Actions | Key states |
|---|---|---|---|---|
| **Admin Home** | Module launcher | "Welcome, [Username]!" and 9 cards: Course Management, Enrollment, Allocation, Students, Teachers, Grievance Management, Feedback, Reports, Apply Leave | Card click | None |
| **Course Management** | Manage courses and configuration | Courses table, create/edit form | Add, Edit, Save. Confirm only for destructive actions | Skeleton, empty with Add CTA, success toast, error |
| **Enrollment** | Manage enrollment records | Enrollment records table | As required | Skeleton, empty, error, success toast |
| **Allocation** | Manage allocation | Allocation table | Allocation actions, confirm for high-impact action | Skeleton, inline loading, success toast, error |
| **Students** | Manage student records | Students table | As required | Skeleton, empty, error |
| **Teachers** | Manage teacher records | Teachers table | As required | Skeleton, empty, error |
| **Grievance Management** | Sub-module launcher | Cards: Grievances, Students, Teachers, Add Grievance (4th card wraps to row 2, left-aligned) | Card click | None |
| **Grievances** | All incoming grievances | Table of Student + Teacher grievances | Admin actions: Not specified | Skeleton, empty, error |
| **Grievances → Students** | Student-raised grievances | Table | Admin actions: Not specified | Skeleton, empty, error |
| **Grievances → Teachers** | Teacher-raised grievances | Table | Admin actions: Not specified | Skeleton, empty, error |
| **Add Grievance** | Admin raises own grievance | Form | Submit | Inline loading, success toast, error |
| **Feedback** | Feedback forms and records | Table and configuration form | Add, Edit, Save | Skeleton, empty, success toast, error |
| **Reports** | Reports | Report tables. No charts | As each report needs | Skeleton, empty, error |
| **Apply Leave** | Admin's own leave | Form. Details Not specified | Submit | Not specified |

---

## 24. V1 / V2 Boundaries

### V1 (frozen)

- Role-based login
- Student, Teacher and Admin Home
- Module and sub-module navigation
- Enrollment
- Attendance
- Progress / Grades
- Certificates
- Grievances (Student, Teacher, Admin)
- Leave (Student Leave Application, Teacher and Admin Apply Leave, Teacher mentee Leave Requests)
- Teacher Mentoring
- Feedback
- Teacher attendance management
- Teacher grade management
- Admin management workflows (including Allocation and Reports)
- Pagination
- Loading, empty, error and success states
- Basic caching
- Desktop-first UI

### V2 (not in V1)

- Search
- Advanced filters
- Advanced table navigation
- Better mobile optimisation
- Additional convenience features
- Other enhancements identified after V1

---

## 25. Open Items (Not Specified)

These need a team decision before implementation. Do not invent them.

| # | Item |
|---|---|
| 1 | Student Leave Application: position in Student navigation, form fields, status display |
| 2 | Student-facing view of mentoring, if any |
| 3 | Teacher Mentoring: actions on grievances and leave requests, statuses, table columns |
| 4 | Teacher and Admin Apply Leave: form fields and approval flow |
| 5 | Admin actions on grievances (respond, status, close) and grievance statuses |
| 6 | Placement of admin configuration (seats, batches, teachers, grading schemes, feedback forms, sections) |
| 7 | Where former Teacher modules "Student Details" and "Assigned Sections" now live |
| 8 | Field lists and validation for all forms |
| 9 | Exact color values, font sizes, table page size, cache lifetime |
| 10 | Profile entry in the sidebar per role |
| 11 | Confirmation dialogs for teacher request handling and admin actions beyond those listed |

---

## 26. UI/UX Status

**The V1 UI/UX direction is frozen and ready for frontend implementation. Items listed under [Open Items](#25-open-items-not-specified) are the only undefined areas and need a decision before the affected screens are built.**
