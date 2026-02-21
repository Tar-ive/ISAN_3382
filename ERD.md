# University Course Management System — ERD

---
## 1) Entities & Attributes (with attribute types)

Legend:
- **PK** = Primary Key
- **AK** = Alternate Key / Unique Identifier
- **(C)** = Composite attribute
- **(M)** = Multivalued attribute *(modeled as separate entity/table)*
- **(D)** = Derived attribute

### 1.1 Core academic structure

| Entity | Primary Key | Alternate keys / unique IDs | Selected attributes | Notes (C/M/D) |
|---|---|---|---|---|
| Department | department_id | code | name, chair_instructor_id (FK) | — |
| Course (Catalog) | course_id | (subject_code, catalog_number) *(candidate AK)* | department_id (FK), title, description, units_min, units_max, grading_basis_default, level | — |
| Term | term_id | — | academic_year, quarter, start_date, end_date | — |
| CourseOffering / Section | offering_id | class_number | course_id (FK), term_id (FK), section_number, capacity, waitlist_capacity, modality, enrolled_count, waitlist_count | enrolled_count **(D)**, waitlist_count **(D)** |
| Classroom | room_id | (building, room_number) *(candidate)* | capacity | — |
| Meeting | meeting_id | — | offering_id (FK), room_id (FK, optional), day_of_week, start_time, end_time | room optional for online/hybrid |

### 1.2 People

| Entity | Primary Key | Alternate keys / unique IDs | Selected attributes | Notes (C/M/D) |
|---|---|---|---|---|
| Student | student_uuid | student_id_number; sunet_id | legal_name, preferred_name, date_of_birth, age, level, admit_term_id (FK, optional), status | legal_name **(C)**; emails/phones **(M)**; age **(D)** |
| Instructor | instructor_uuid | sunet_id | name, title, office_location | name **(C)**; emails **(M)** |

### 1.3 Relationship / associative entities (M:N resolution)

| Associative entity | Primary Key | Foreign Keys | Relationship captured | Relationship attributes / notes |
|---|---|---|---|---|
| Enrollment | enrollment_id | student_uuid → Student; offering_id → CourseOffering | Student **M:N** CourseOffering | status (enrolled/waitlisted/dropped), grading_basis, units_taken, enrolled_at, dropped_at, final_grade; rule: unique (student_uuid, offering_id) |
| TeachingAssignment | teaching_assignment_id | instructor_uuid → Instructor; offering_id → CourseOffering | Instructor **M:N** CourseOffering | role (instructor_of_record/co-instructor/TA), percent_responsibility |
| CoursePrerequisite | prereq_id | course_id → Course; prereq_course_id → Course | Course **M:N** Course (self) | min_grade; rule: course_id != prereq_course_id |
| CrossListing | crosslist_id | offering_id → CourseOffering; crosslisted_course_id → Course | Offering **M:N** Course | optional: used for cross-listed offerings |

### 1.4 Multivalued attribute entities (examples)

| Entity (multivalued attribute table) | Primary Key | Foreign Keys | Attributes | Notes |
|---|---|---|---|---|
| StudentEmail | student_email_id | student_uuid → Student | email, is_primary | models Student.emails **(M)** |
| StudentPhone | student_phone_id | student_uuid → Student | phone, type | models Student.phones **(M)** |
| InstructorEmail | instructor_email_id | instructor_uuid → Instructor | email, is_primary | models Instructor.emails **(M)** |

---

## 2) Conceptual, Logical, and Physical Model

### 2.1 Conceptual Model (high-level ER view)

| Item | Details |
|---|---|
| Entities | Department, Course, Term, CourseOffering, Meeting, Classroom, Student, Instructor |
| Key Relationships (cardinalities) | Department **1**—**N** Course; Course **1**—**N** CourseOffering; Term **1**—**N** CourseOffering; CourseOffering **1**—**N** Meeting; Classroom **1**—**N** Meeting *(room optional for online)*; Student **M**—**N** CourseOffering *(via Enrollment)*; Instructor **M**—**N** CourseOffering *(via TeachingAssignment)*; Course **M**—**N** Course *(via CoursePrerequisite)*; CourseOffering **M**—**N** Course *(via CrossListing, optional)* |
| Attribute types used | **Composite:** Student.legal_name, Instructor.name; **Multivalued:** emails/phones *(modeled as separate entities)*; **Derived:** Student.age, CourseOffering.enrolled_count/waitlist_count |

### 2.2 Logical Model (relational design)

| Table | Primary Key | Foreign Keys | Key constraints / notes |
|---|---|---|---|
| Department | department_id | — | code **UNIQUE** |
| Course | course_id | department_id → Department | (subject_code, catalog_number) **UNIQUE** *(optionally scoped by catalog_year)* |
| Term | term_id | — | — |
| CourseOffering | offering_id | course_id → Course; term_id → Term | class_number **UNIQUE** |
| Classroom | room_id | — | — |
| Meeting | meeting_id | offering_id → CourseOffering; room_id → Classroom *(NULL allowed)* | supports multiple meetings per offering |
| Student | student_uuid | admit_term_id → Term *(optional)* | student_id_number **UNIQUE**; sunet_id **UNIQUE** |
| Instructor | instructor_uuid | — | sunet_id **UNIQUE** |
| Enrollment | enrollment_id | student_uuid → Student; offering_id → CourseOffering | (student_uuid, offering_id) **UNIQUE** *(one record per student per offering)* |
| TeachingAssignment | teaching_assignment_id | instructor_uuid → Instructor; offering_id → CourseOffering | (instructor_uuid, offering_id, role) **UNIQUE** *(recommended)* |
| CoursePrerequisite | prereq_id | course_id → Course; prereq_course_id → Course | prevent course_id = prereq_course_id |
| CrossListing | crosslist_id | offering_id → CourseOffering; crosslisted_course_id → Course | optional cross-listing |
| StudentEmail | student_email_id | student_uuid → Student | multivalued attribute table |
| StudentPhone | student_phone_id | student_uuid → Student | multivalued attribute table |
| InstructorEmail | instructor_email_id | instructor_uuid → Instructor | multivalued attribute table |

### 2.3 Physical Model (implementation-oriented decisions; no DDL requested)

| Concern | Recommendation | Why |
|---|---|---|
| Primary keys | Use immutable **UUIDs** for Student/Instructor PKs | safer merges, privacy-friendly, avoids exposing ID numbers |
| Alternate identifiers | Enforce **UNIQUE** on Student.student_id_number, Student.sunet_id, Instructor.sunet_id, CourseOffering.class_number | matches registrar/campus identity patterns |
| Indexing | Index all FKs (student_uuid, offering_id, course_id, term_id, department_id) | improves joins and common queries |
| Enrollment lifecycle | Use status/soft-delete fields (enrolled/waitlisted/dropped) instead of deleting rows | auditability + transcript integrity |
| Derived fields | Compute age, enrolled_count, waitlist_count via views/materialized views (cache only if needed) | avoids inconsistency and stale counts |

---

## 3) ER Diagrams
### 3.1 Core Catalog & Scheduling (Department → Course → Offering → Meetings)

```mermaid
erDiagram
    DEPARTMENT ||--o{ COURSE : offers
    COURSE ||--o{ COURSE_OFFERING : offered_as
    TERM ||--o{ COURSE_OFFERING : occurs_in

    COURSE_OFFERING ||--o{ MEETING : has
    CLASSROOM ||--o{ MEETING : hosts

    DEPARTMENT {
        int department_id PK
        string code UK
        string name
    }

    COURSE {
        int course_id PK
        int department_id FK
        string subject_code
        string catalog_number
        string title
        string description
        int units_min
        int units_max
        string grading_basis_default
        string level
    }

    TERM {
        int term_id PK
        string academic_year
        string quarter
        string start_date
        string end_date
    }

    COURSE_OFFERING {
        int offering_id PK
        int course_id FK
        int term_id FK
        string section_number
        string class_number UK
        int capacity
        int waitlist_capacity
        string modality
        %% Derived in implementation: enrolled_count, waitlist_count
    }

    CLASSROOM {
        int room_id PK
        string building
        string room_number
        int capacity
    }

    MEETING {
        int meeting_id PK
        int offering_id FK
        int room_id FK
        string day_of_week
        string start_time
        string end_time
    }
```
---

### 3.2 Student Enrollment (Student ↔ Offering)

```mermaid
erDiagram
    STUDENT ||--o{ ENROLLMENT : registers
    COURSE_OFFERING ||--o{ ENROLLMENT : has

    STUDENT {
        string student_uuid PK
        string student_id_number UK
        string sunet_id UK
        string legal_first
        string legal_middle
        string legal_last
        string preferred_name
        string date_of_birth
        %% Derived in implementation: age
        string level
        int admit_term_id FK
        string status
    }

    COURSE_OFFERING {
        int offering_id PK
        int course_id FK
        int term_id FK
        string section_number
        string class_number UK
    }

    ENROLLMENT {
        int enrollment_id PK
        string student_uuid FK
        int offering_id FK
        string status
        string grading_basis
        int units_taken
        string enrolled_at
        string dropped_at
        string final_grade
    }
```

---

### 3.3 Teaching Assignments (Instructor ↔ Offering)

```mermaid
erDiagram
    INSTRUCTOR ||--o{ TEACHING_ASSIGNMENT : teaches
    COURSE_OFFERING ||--o{ TEACHING_ASSIGNMENT : staffed_by

    INSTRUCTOR {
        string instructor_uuid PK
        string sunet_id UK
        string first_name
        string middle_name
        string last_name
        string title
        string office_location
    }

    COURSE_OFFERING {
        int offering_id PK
        int course_id FK
        int term_id FK
        string class_number UK
    }

    TEACHING_ASSIGNMENT {
        int teaching_assignment_id PK
        string instructor_uuid FK
        int offering_id FK
        string role
        float percent_responsibility
    }
```

---

### 3.4 Curriculum Rules (Prerequisites + Cross-Listing)

```mermaid
erDiagram
    COURSE ||--o{ COURSE_PREREQUISITE : has
    COURSE ||--o{ COURSE_PREREQUISITE : is_required

    COURSE_OFFERING ||--o{ CROSS_LISTING : crosslists
    COURSE ||--o{ CROSS_LISTING : listed_as

    COURSE {
        int course_id PK
        int department_id FK
        string subject_code
        string catalog_number
        string title
    }

    COURSE_PREREQUISITE {
        int prereq_id PK
        int course_id FK
        int prereq_course_id FK
        string min_grade
    }

    COURSE_OFFERING {
        int offering_id PK
        int course_id FK
        int term_id FK
        string class_number UK
    }

    CROSS_LISTING {
        int crosslist_id PK
        int offering_id FK
        int crosslisted_course_id FK
    }
```

---

### 3.5 Multivalued Contact Attributes (modeled as entities)

```mermaid
erDiagram
    STUDENT ||--o{ STUDENT_EMAIL : has
    STUDENT ||--o{ STUDENT_PHONE : has
    INSTRUCTOR ||--o{ INSTRUCTOR_EMAIL : has

    STUDENT {
        string student_uuid PK
        string student_id_number UK
        string sunet_id UK
    }

    INSTRUCTOR {
        string instructor_uuid PK
        string sunet_id UK
    }

    STUDENT_EMAIL {
        int student_email_id PK
        string student_uuid FK
        string email
        boolean is_primary
    }

    STUDENT_PHONE {
        int student_phone_id PK
        string student_uuid FK
        string phone
        string type
    }

    INSTRUCTOR_EMAIL {
        int instructor_email_id PK
        string instructor_uuid FK
        string email
        boolean is_primary
    }
```

---
