# Complete Theoretical Documentation: Course Model in Fusion IIIT

## Table of Contents
1. [Model Overview](#model-overview)
2. [Field-by-Field Analysis](#field-by-field-analysis)
3. [Relationships & Foreign Keys](#relationships--foreign-keys)
4. [Validation & Constraints](#validation--constraints)
5. [Database Schema Design](#database-schema-design)
6. [Version Control System](#version-control-system)
7. [Audit Trail Integration](#audit-trail-integration)
8. [Business Logic & Methods](#business-logic--methods)
9. [Course Lifecycle Management](#course-lifecycle-management)
10. [Integration with Academic System](#integration-with-academic-system)
11. [Data Integrity & Security](#data-integrity--security)
12. [Performance Considerations](#performance-considerations)

---

## Model Overview

### Class Definition
```python
class Course(models.Model):
    """Store course details"""
```

The `Course` model represents the central entity in the academic management system of Fusion IIIT. It serves as a comprehensive repository for all course-related information, from basic identification to complex academic requirements and assessment structures. This model is designed to handle versioning, multi-disciplinary associations, and complete academic lifecycle management.

### Primary Purpose
- **Academic Catalog Management**: Maintains the official course catalog with all academic offerings
- **Version Control**: Implements sophisticated versioning system for course evolution tracking
- **Assessment Framework**: Defines comprehensive grading and evaluation structures
- **Prerequisite Management**: Handles complex course dependency relationships
- **Multi-Disciplinary Support**: Allows courses to span multiple academic disciplines
- **Capacity Management**: Controls enrollment limits and seat allocation
- **Integration Hub**: Connects with Programme, Curriculum, Semester, and Batch systems

---

## Field-by-Field Analysis

### 1. **code** - Course Identifier
```python
code = models.CharField(max_length=10, null=False, blank=False)
```

**Technical Specifications:**
- **Data Type**: CharField with maximum 10 characters
- **Constraints**: Non-nullable, non-blank (required field)
- **Purpose**: Primary course identifier following institutional coding standards

**Business Logic:**
- Typically follows patterns like "CS101", "MATH201", "PHY301"
- Must be unique when combined with version (composite uniqueness)
- Used for student transcripts, academic planning, and external reporting
- Changes to course code trigger MAJOR version bumps (semantic versioning)

**Validation Rules:**
- Cannot be empty or null
- Must be unique within the same version
- Typically validated against institutional naming conventions
- Case-sensitive storage but often normalized for comparison

### 2. **name** - Course Title
```python
name = models.CharField(max_length=100, null=False, blank=False)
```

**Technical Specifications:**
- **Data Type**: CharField with maximum 100 characters
- **Constraints**: Non-nullable, non-blank (required field)
- **Purpose**: Human-readable course title for academic records

**Business Logic:**
- Official course name appearing on transcripts and academic documents
- Must be descriptive and follow institutional guidelines
- Changes trigger MINOR version bumps if substantial, PATCH if minor corrections
- Used in course catalogs, registration systems, and academic planning

**Examples:**
- "Introduction to Computer Science"
- "Advanced Database Management Systems"
- "Thermodynamics and Heat Transfer"

### 3. **version** - Semantic Version Control
```python
version = models.DecimalField(
    max_digits=5,
    decimal_places=1,
    default=1.0,
    validators=[MinValueValidator(1.0), DecimalValidator(max_digits=5, decimal_places=1)])
```

**Technical Specifications:**
- **Data Type**: DecimalField with precision of 5 digits, 1 decimal place
- **Range**: 1.0 to 9999.9 (theoretical maximum)
- **Validators**: Minimum value of 1.0, proper decimal format validation
- **Default**: 1.0 for new courses

**Semantic Versioning Logic:**
- **MAJOR.MINOR Format**: X.Y (e.g., 1.0, 2.5, 15.3)
- **MAJOR Bumps (X.0)**: Course code changes, fundamental restructuring
- **MINOR Bumps (X.Y)**: Credit changes, significant content updates, new prerequisites
- **PATCH Bumps (X.Y+0.1)**: Typo corrections, minor content adjustments

**Version Bump Triggers:**
```python
# MAJOR version bumps (X.0):
- Course code changes (CS101 → CS102)
- Complete course restructuring
- Discipline reassignment

# MINOR version bumps (X.Y):
- Credit hour changes
- Prerequisites addition/removal
- Assessment structure modifications
- Syllabus major updates

# PATCH version bumps (X.Y+0.1):
- Typo corrections in name/description
- Minor syllabus corrections
- Reference book updates
```

### 4. **credit** - Academic Credit Hours
```python
credit = models.PositiveIntegerField(default=0, null=False, blank=False)
```

**Technical Specifications:**
- **Data Type**: PositiveIntegerField (0 to 2,147,483,647)
- **Default**: 0 (allows for non-credit courses)
- **Constraints**: Must be non-negative

**Academic Significance:**
- Determines course weightage in GPA calculations
- Affects student workload planning and degree requirements
- Used for fee calculation and academic load balancing
- Typically ranges from 0-6 credits in most institutions

**Business Rules:**
- 0 credits: Seminars, orientations, non-graded activities
- 1-2 credits: Lab courses, tutorials, workshops
- 3-4 credits: Standard lecture courses
- 5-6 credits: Advanced courses, projects, intensive programs

### 5. **Contact Hours Distribution**

#### lecture_hours
```python
lecture_hours = PositiveIntegerField(null=True)
```

#### tutorial_hours
```python
tutorial_hours = PositiveIntegerField(null=True)
```

#### pratical_hours (Note: Typo in field name - should be "practical_hours")
```python
pratical_hours = PositiveIntegerField(null=True)
```

#### discussion_hours
```python
discussion_hours = PositiveIntegerField(null=True)
```

#### project_hours
```python
project_hours = PositiveIntegerField(null=True)
```

**Technical Specifications:**
- **Data Type**: PositiveIntegerField for all contact hour fields
- **Null Policy**: All contact hour fields allow null values
- **Purpose**: Detailed breakdown of course time allocation

**Academic Framework:**
- **Lecture Hours**: Traditional classroom instruction time
- **Tutorial Hours**: Small group problem-solving sessions
- **Practical Hours**: Laboratory and hands-on work
- **Discussion Hours**: Seminar-style interactive sessions
- **Project Hours**: Independent or group project work

**Scheduling Implications:**
- Total contact hours = sum of all hour types
- Used for timetable generation and resource allocation
- Affects instructor workload calculations
- Determines classroom and lab requirements

### 6. **Prerequisites Management**

#### pre_requisits (Text-based prerequisites)
```python
pre_requisits = models.TextField(null=True, blank=True)
```

#### pre_requisit_courses (Structured course relationships)
```python
pre_requisit_courses = models.ManyToManyField('self', blank=True)
```

**Dual Prerequisite System:**
- **Text Prerequisites**: Flexible description for complex requirements
- **Course Prerequisites**: Structured relationships for system validation

**Business Logic:**
- Text prerequisites handle conditional requirements ("CS101 OR CS102")
- Course prerequisites enable automated validation and planning
- Both systems work together for comprehensive prerequisite management

**Examples:**
```python
# Text prerequisites:
"Completion of CS101 with minimum B grade OR CS102 with A grade"
"Mathematics placement test score > 85 OR completion of MATH100"

# Course prerequisites (ManyToMany relationships):
course.pre_requisit_courses.set([cs101_course, math101_course])
```

### 7. **syllabus** - Course Content
```python
syllabus = models.TextField()
```

**Technical Specifications:**
- **Data Type**: TextField (unlimited length)
- **Required**: Yes (null=False, blank=False by default)
- **Purpose**: Comprehensive course content description

**Content Structure:**
- Course objectives and learning outcomes
- Module-wise topic breakdown
- Learning methodologies
- Lab/practical components
- Academic calendar alignment

### 8. **Assessment Structure (Percentage-based)**

The Course model implements a comprehensive assessment framework with the following components:

#### Quiz Assessments
```python
percent_quiz_1 = models.PositiveIntegerField(default=10, null=False, blank=False)
percent_quiz_2 = models.PositiveIntegerField(default=10, null=False, blank=False)
```

#### Major Examinations
```python
percent_midsem = models.PositiveIntegerField(default=20, null=False, blank=False)
percent_endsem = models.PositiveIntegerField(default=30, null=False, blank=False)
```

#### Continuous Assessment
```python
percent_project = models.PositiveIntegerField(default=15, null=False, blank=False)
percent_lab_evaluation = models.PositiveIntegerField(default=10, null=False, blank=False)
percent_course_attendance = models.PositiveIntegerField(default=5, null=False, blank=False)
```

**Assessment Framework Analysis:**

**Default Distribution (Total: 100%):**
- Quiz 1: 10%
- Quiz 2: 10%
- Mid-semester: 20%
- End-semester: 30%
- Project: 15%
- Lab Evaluation: 10%
- Attendance: 5%

**Validation Requirements:**
- All percentages must sum to 100% (business rule validation)
- Each component must be non-negative
- Flexibility to adjust based on course type and requirements

**Academic Implications:**
- Ensures balanced assessment across different evaluation methods
- Supports both theoretical and practical evaluation
- Provides clear grading rubric for students and instructors

### 9. **ref_books** - Reference Materials
```python
ref_books = models.TextField()
```

**Purpose**: Comprehensive list of recommended reading materials
- Primary textbooks
- Reference books
- Online resources
- Research papers
- Academic journals

### 10. **Status and Control Fields**

#### working_course
```python
working_course = models.BooleanField(default=True)
```
- **Purpose**: Indicates if course is currently active/offered
- **Default**: True (new courses are active by default)
- **Usage**: Course catalog filtering and registration availability

#### latest_version
```python
latest_version = models.BooleanField(default=True)
```
- **Purpose**: Marks the most current version of a course
- **Business Rule**: Only one version per course code should have latest_version=True
- **Usage**: Default course selection and current catalog display

### 11. **max_seats** - Enrollment Capacity
```python
max_seats = models.IntegerField(default=90)
```
- **Purpose**: Maximum student enrollment limit
- **Default**: 90 students
- **Usage**: Registration system capacity control and resource planning

---

## Relationships & Foreign Keys

### 1. **disciplines** - Multi-Disciplinary Association
```python
disciplines = models.ManyToManyField(Discipline, blank=True)
```

**Relationship Type**: Many-to-Many
**Purpose**: Allows courses to belong to multiple academic disciplines
**Examples**:
- "Bioinformatics" → Biology + Computer Science
- "Mathematical Physics" → Mathematics + Physics
- "Design Engineering" → Design + Engineering

**Business Implications**:
- Enables interdisciplinary program design
- Supports cross-department resource sharing
- Facilitates student exploration across fields

### 2. **pre_requisit_courses** - Course Dependencies
```python
pre_requisit_courses = models.ManyToManyField('self', blank=True)
```

**Relationship Type**: Self-referential Many-to-Many
**Purpose**: Creates directed acyclic graph (DAG) of course dependencies
**Academic Significance**:
- Prevents students from taking courses without proper foundation
- Enables automated academic planning and validation
- Supports curriculum design and sequencing

**Implementation Details**:
- Self-referential relationship allows courses to reference other courses
- Many-to-many enables complex prerequisite structures
- System must prevent circular dependencies

### 3. **Related Models Integration**

#### CourseSlot Relationship
```python
# In CourseSlot model:
courses = models.ManyToManyField(Course, blank=True)

# Property in Course model:
@property
def courseslots(self):
    return CourseSlot.objects.filter(courses=self.id)
```

**Integration Purpose**:
- Links courses to specific semester slots
- Enables timetable generation and scheduling
- Supports batch-wise course offering management

#### CourseInstructor Assignment
```python
class CourseInstructor(models.Model):
    course_id = models.ForeignKey(Course, on_delete=models.CASCADE)
    instructor_id = models.ForeignKey(Faculty, on_delete=models.CASCADE)
    year = models.IntegerField(default=datetime.date.today().year)
    semester_type = models.CharField(max_length=20, choices=SEMESTER_TYPE_CHOICES)
```

**Relationship Features**:
- Tracks instructor assignments per academic year/semester
- Supports multiple instructors per course
- Maintains historical teaching records

---

## Validation & Constraints

### 1. **Database-Level Constraints**

#### Unique Together Constraint
```python
class Meta:
    unique_together = ('code', 'version')
```

**Purpose**: Ensures no duplicate course code-version combinations
**Business Logic**: Allows multiple versions of same course while preventing exact duplicates
**Enforcement**: Database-level constraint with Django ORM validation

### 2. **Field-Level Validation**

#### Version Validation
```python
validators=[MinValueValidator(1.0), DecimalValidator(max_digits=5, decimal_places=1)]
```

**MinValueValidator(1.0)**:
- Ensures version starts at 1.0 or higher
- Prevents negative or zero versions
- Maintains semantic versioning integrity

**DecimalValidator**:
- Enforces decimal format (XXXXX.X)
- Prevents invalid decimal representations
- Maintains consistent version formatting

### 3. **Custom Validation Logic**

#### Assessment Percentage Validation
```python
def clean(self):
    """Validate that assessment percentages sum to 100%"""
    total_percentage = (
        self.percent_quiz_1 + self.percent_quiz_2 + 
        self.percent_midsem + self.percent_endsem + 
        self.percent_project + self.percent_lab_evaluation + 
        self.percent_course_attendance
    )
    if total_percentage != 100:
        raise ValidationError("Assessment percentages must sum to 100%")
```

**Note**: This validation may be implemented in forms or model clean methods

#### Latest Version Validation
```python
def save(self, *args, **kwargs):
    """Ensure only one latest_version=True per course code"""
    if self.latest_version:
        Course.objects.filter(code=self.code).update(latest_version=False)
    super().save(*args, **kwargs)
```

**Note**: This business logic ensures data integrity for latest version tracking

---

## Database Schema Design

### 1. **Primary Course Table**
```sql
CREATE TABLE programme_curriculum_course (
    id SERIAL PRIMARY KEY,
    code VARCHAR(10) NOT NULL,
    name VARCHAR(100) NOT NULL,
    version DECIMAL(5,1) DEFAULT 1.0 NOT NULL,
    credit INTEGER DEFAULT 0 NOT NULL,
    lecture_hours INTEGER,
    tutorial_hours INTEGER,
    pratical_hours INTEGER,
    discussion_hours INTEGER,
    project_hours INTEGER,
    pre_requisits TEXT,
    syllabus TEXT NOT NULL,
    percent_quiz_1 INTEGER DEFAULT 10 NOT NULL,
    percent_midsem INTEGER DEFAULT 20 NOT NULL,
    percent_quiz_2 INTEGER DEFAULT 10 NOT NULL,
    percent_endsem INTEGER DEFAULT 30 NOT NULL,
    percent_project INTEGER DEFAULT 15 NOT NULL,
    percent_lab_evaluation INTEGER DEFAULT 10 NOT NULL,
    percent_course_attendance INTEGER DEFAULT 5 NOT NULL,
    ref_books TEXT NOT NULL,
    working_course BOOLEAN DEFAULT TRUE NOT NULL,
    latest_version BOOLEAN DEFAULT TRUE NOT NULL,
    max_seats INTEGER DEFAULT 90 NOT NULL,
    
    CONSTRAINT course_code_version_unique UNIQUE (code, version),
    CONSTRAINT version_min_check CHECK (version >= 1.0)
);
```

### 2. **Course Audit and Tracking Tables**

#### Course Audit Log Table
```sql
CREATE TABLE programme_curriculum_courseauditlog (
    id SERIAL PRIMARY KEY,
    course_id INTEGER REFERENCES programme_curriculum_course(id) ON DELETE CASCADE,
    user_id INTEGER REFERENCES auth_user(id) ON DELETE CASCADE,
    action VARCHAR(10) NOT NULL,
    old_data JSONB,
    new_data JSONB,
    timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    version_bump_type VARCHAR(10),
    comments TEXT DEFAULT '',
    
    CHECK (action IN ('CREATE', 'UPDATE', 'DELETE')),
    CHECK (version_bump_type IN ('NONE', 'PATCH', 'MINOR', 'MAJOR'))
);
```

#### Course Instructor Assignment Table
```sql
CREATE TABLE programme_curriculum_courseinstructor (
    id SERIAL PRIMARY KEY,
    course_id_id INTEGER REFERENCES programme_curriculum_course(id) ON DELETE CASCADE,
    instructor_id_id INTEGER REFERENCES globals_faculty(id) ON DELETE CASCADE,
    year INTEGER DEFAULT EXTRACT(YEAR FROM CURRENT_DATE) NOT NULL,
    semester_type VARCHAR(20),
    
    CHECK (semester_type IN ('Odd Semester', 'Even Semester', 'Summer Semester')),
    UNIQUE(course_id_id, instructor_id_id, year, semester_type)
);
```

### 3. **Course Slot and Scheduling Tables**

#### Course Slot Table
```sql
CREATE TABLE programme_curriculum_courseslot (
    id SERIAL PRIMARY KEY,
    semester_id INTEGER REFERENCES programme_curriculum_semester(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    type VARCHAR(70) NOT NULL,
    course_slot_info TEXT,
    duration INTEGER DEFAULT 1,
    min_registration_limit INTEGER DEFAULT 0,
    max_registration_limit INTEGER DEFAULT 1000,
    
    CHECK (type IN ('Professional Core', 'Professional Elective', 'Professional Lab', 
                   'Engineering Science', 'Natural Science', 'Humanities', 'Design', 
                   'Manufacturing', 'Management Science', 'Open Elective', 'Swayam', 
                   'Project', 'Optional', 'Backlog', 'Others')),
    UNIQUE(semester_id, name, type)
);
```

#### Course Slot to Course Junction Table
```sql
CREATE TABLE programme_curriculum_courseslot_courses (
    id SERIAL PRIMARY KEY,
    courseslot_id INTEGER REFERENCES programme_curriculum_courseslot(id) ON DELETE CASCADE,
    course_id INTEGER REFERENCES programme_curriculum_course(id) ON DELETE CASCADE,
    UNIQUE(courseslot_id, course_id)
);
```

### 4. **Course Relationship Tables**

#### Disciplines Association Table
```sql
CREATE TABLE programme_curriculum_course_disciplines (
    id SERIAL PRIMARY KEY,
    course_id INTEGER REFERENCES programme_curriculum_course(id) ON DELETE CASCADE,
    discipline_id INTEGER REFERENCES globals_discipline(id) ON DELETE CASCADE,
    UNIQUE(course_id, discipline_id)
);
```

#### Prerequisites Association Table
```sql
CREATE TABLE programme_curriculum_course_pre_requisit_courses (
    id SERIAL PRIMARY KEY,
    from_course_id INTEGER REFERENCES programme_curriculum_course(id) ON DELETE CASCADE,
    to_course_id INTEGER REFERENCES programme_curriculum_course(id) ON DELETE CASCADE,
    UNIQUE(from_course_id, to_course_id)
);
```

### 5. **Course Proposal and Workflow Tables**

#### New Course Proposal Table
```sql
CREATE TABLE programme_curriculum_newproposalfile (
    id SERIAL PRIMARY KEY,
    uploader VARCHAR(100) NOT NULL,
    designation VARCHAR(100) NOT NULL,
    code VARCHAR(10) NOT NULL,
    name VARCHAR(100) NOT NULL,
    credit INTEGER DEFAULT 3 NOT NULL,
    lecture_hours INTEGER DEFAULT 3,
    tutorial_hours INTEGER DEFAULT 0,
    pratical_hours INTEGER DEFAULT 0,
    discussion_hours INTEGER DEFAULT 0,
    project_hours INTEGER DEFAULT 0,
    pre_requisits TEXT,
    syllabus TEXT NOT NULL,
    max_seats INTEGER DEFAULT 90,
    percent_quiz_1 INTEGER DEFAULT 10 NOT NULL,
    percent_midsem INTEGER DEFAULT 20 NOT NULL,
    percent_quiz_2 INTEGER DEFAULT 10 NOT NULL,
    percent_endsem INTEGER DEFAULT 30 NOT NULL,
    percent_project INTEGER DEFAULT 15 NOT NULL,
    percent_lab_evaluation INTEGER DEFAULT 10 NOT NULL,
    percent_course_attendance INTEGER DEFAULT 5 NOT NULL,
    ref_books TEXT NOT NULL,
    subject VARCHAR(100),
    description VARCHAR(400),
    upload_date TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    is_read BOOLEAN DEFAULT FALSE,
    is_update BOOLEAN DEFAULT FALSE,
    is_archive BOOLEAN DEFAULT FALSE,
    
    UNIQUE(code, uploader, name)
);
```

#### Proposal Prerequisite Courses Junction Table
```sql
CREATE TABLE programme_curriculum_newproposalfile_pre_requisit_courses (
    id SERIAL PRIMARY KEY,
    newproposalfile_id INTEGER REFERENCES programme_curriculum_newproposalfile(id) ON DELETE CASCADE,
    course_id INTEGER REFERENCES programme_curriculum_course(id) ON DELETE CASCADE,
    UNIQUE(newproposalfile_id, course_id)
);
```

#### Proposal Tracking Table
```sql
CREATE TABLE programme_curriculum_proposal_tracking (
    id SERIAL PRIMARY KEY,
    file_id VARCHAR(100) NOT NULL,
    current_id VARCHAR(100) NOT NULL,
    current_design VARCHAR(100) NOT NULL,
    receive_id_id INTEGER REFERENCES auth_user(id) ON DELETE CASCADE,
    receive_design_id INTEGER REFERENCES globals_designation(id) ON DELETE CASCADE,
    disciplines_id INTEGER REFERENCES globals_discipline(id) ON DELETE CASCADE,
    receive_date TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    forward_date TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    remarks VARCHAR(250),
    is_added BOOLEAN DEFAULT FALSE,
    is_submitted BOOLEAN DEFAULT FALSE,
    is_rejected BOOLEAN DEFAULT FALSE,
    sender_archive BOOLEAN DEFAULT FALSE,
    receiver_archive BOOLEAN DEFAULT FALSE,
    
    UNIQUE(file_id, current_id, current_design, disciplines_id)
);
```

### 6. **Academic Hierarchy Tables**

#### Programme Table
```sql
CREATE TABLE programme_curriculum_programme (
    id SERIAL PRIMARY KEY,
    category VARCHAR(10) NOT NULL,
    name VARCHAR(100) NOT NULL,
    
    CHECK (category IN ('UG', 'PG', 'PHD')),
    UNIQUE(category, name)
);
```

#### Programme Disciplines Junction Table
```sql
CREATE TABLE programme_curriculum_programme_disciplines (
    id SERIAL PRIMARY KEY,
    programme_id INTEGER REFERENCES programme_curriculum_programme(id) ON DELETE CASCADE,
    discipline_id INTEGER REFERENCES globals_discipline(id) ON DELETE CASCADE,
    UNIQUE(programme_id, discipline_id)
);
```

#### Curriculum Table
```sql
CREATE TABLE programme_curriculum_curriculum (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    programme_id INTEGER REFERENCES programme_curriculum_programme(id) ON DELETE CASCADE,
    version DECIMAL(5,1) DEFAULT 1.0 NOT NULL,
    no_of_semester INTEGER NOT NULL,
    min_credit INTEGER NOT NULL,
    
    CONSTRAINT version_min_check CHECK (version >= 1.0),
    UNIQUE(name, programme_id, version)
);
```

#### Semester Table
```sql
CREATE TABLE programme_curriculum_semester (
    id SERIAL PRIMARY KEY,
    curriculum_id INTEGER REFERENCES programme_curriculum_curriculum(id) ON DELETE CASCADE,
    semester_no INTEGER NOT NULL,
    
    UNIQUE(curriculum_id, semester_no)
);
```

#### Batch Table
```sql
CREATE TABLE programme_curriculum_batch (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    discipline_id INTEGER REFERENCES globals_discipline(id) ON DELETE CASCADE,
    year INTEGER NOT NULL,
    curriculum_id INTEGER REFERENCES programme_curriculum_curriculum(id),
    running_batch BOOLEAN DEFAULT TRUE,
    total_seats INTEGER DEFAULT 60 NOT NULL,
    
    UNIQUE(name, discipline_id, year)
);
```

### 7. **Related Global Tables (External Dependencies)**

#### Faculty Table (from globals app)
```sql
-- Referenced table: globals_faculty
-- Used in: programme_curriculum_courseinstructor
```

#### Discipline Table (from globals app)
```sql
-- Referenced table: globals_discipline
-- Used in: programme_curriculum_course_disciplines,
--          programme_curriculum_programme_disciplines,
--          programme_curriculum_batch,
--          programme_curriculum_proposal_tracking
```

#### Designation Table (from globals app)
```sql
-- Referenced table: globals_designation
-- Used in: programme_curriculum_proposal_tracking
```

#### User Table (from Django auth)
```sql
-- Referenced table: auth_user
-- Used in: programme_curriculum_courseauditlog,
--          programme_curriculum_proposal_tracking
```

### 8. **Table Relationship Overview**

#### Complete Database Table List Used by Course Model:

**Primary Course Tables (4 tables):**
1. `programme_curriculum_course` - Main course data
2. `programme_curriculum_courseauditlog` - Change tracking
3. `programme_curriculum_courseinstructor` - Faculty assignments
4. `programme_curriculum_courseslot` - Course scheduling

**Course Relationship Tables (4 tables):**
5. `programme_curriculum_course_disciplines` - Course-discipline mapping
6. `programme_curriculum_course_pre_requisit_courses` - Prerequisites
7. `programme_curriculum_courseslot_courses` - Slot-course mapping
8. `programme_curriculum_newproposalfile_pre_requisit_courses` - Proposal prerequisites

**Course Proposal and Workflow Tables (2 tables):**
9. `programme_curriculum_newproposalfile` - Course proposals
10. `programme_curriculum_proposal_tracking` - Workflow tracking

**Academic Hierarchy Tables (5 tables):**
11. `programme_curriculum_programme` - Academic programmes
12. `programme_curriculum_programme_disciplines` - Programme-discipline mapping
13. `programme_curriculum_curriculum` - Curriculum definitions
14. `programme_curriculum_semester` - Semester structure
15. `programme_curriculum_batch` - Student batches

**External Reference Tables (4 tables):**
16. `globals_faculty` - Faculty information
17. `globals_discipline` - Academic disciplines
18. `globals_designation` - Position designations
19. `auth_user` - User accounts

**Total Tables Involved: 19 tables**

#### Table Relationship Diagram:
```
auth_user ──────────────┐
                        │
globals_faculty ────────┼─── programme_curriculum_courseinstructor
                        │                    │
globals_discipline ─────┼────────────────────┼─── programme_curriculum_course
        │               │                    │            │
        │               │                    │            │
        └───── programme_curriculum_programme │            │
                        │                    │            │
                        └─── programme_curriculum_curriculum │
                                     │                     │
                            programme_curriculum_semester ──┘
                                     │
                            programme_curriculum_courseslot
                                     │
                            programme_curriculum_batch

Course Prerequisites (Self-Referential):
programme_curriculum_course ←──→ programme_curriculum_course_pre_requisit_courses

Course Disciplines:
programme_curriculum_course ←──→ programme_curriculum_course_disciplines ←──→ globals_discipline

Course Slots:
programme_curriculum_courseslot ←──→ programme_curriculum_courseslot_courses ←──→ programme_curriculum_course

Audit Trail:
programme_curriculum_course ←──→ programme_curriculum_courseauditlog ←──→ auth_user

Proposal System:
programme_curriculum_newproposalfile ←──→ programme_curriculum_proposal_tracking
                │                                          │
                └─── programme_curriculum_newproposalfile_pre_requisit_courses ←──→ programme_curriculum_course
```

### 9. **Indexing Strategy for All Course-Related Tables**

#### Primary Indexes
- **Primary Key**: `id` (auto-generated sequence) on all tables
- **Unique Composite**: `(code, version)` on course table for version control
- **Boolean Indexes**: `working_course`, `latest_version` for filtering
- **Foreign Key Indexes**: Automatic indexes on all relationship fields

#### Performance Indexes
```sql
-- Course table indexes
CREATE INDEX idx_course_code ON programme_curriculum_course(code);
CREATE INDEX idx_course_latest ON programme_curriculum_course(latest_version) WHERE latest_version = TRUE;
CREATE INDEX idx_course_active ON programme_curriculum_course(working_course) WHERE working_course = TRUE;
CREATE INDEX idx_course_credit ON programme_curriculum_course(credit);

-- Audit log indexes
CREATE INDEX idx_audit_course ON programme_curriculum_courseauditlog(course_id);
CREATE INDEX idx_audit_user ON programme_curriculum_courseauditlog(user_id);
CREATE INDEX idx_audit_timestamp ON programme_curriculum_courseauditlog(timestamp);
CREATE INDEX idx_audit_action ON programme_curriculum_courseauditlog(action);

-- Course instructor indexes
CREATE INDEX idx_instructor_course ON programme_curriculum_courseinstructor(course_id_id);
CREATE INDEX idx_instructor_faculty ON programme_curriculum_courseinstructor(instructor_id_id);
CREATE INDEX idx_instructor_year ON programme_curriculum_courseinstructor(year);

-- Course slot indexes
CREATE INDEX idx_slot_semester ON programme_curriculum_courseslot(semester_id);
CREATE INDEX idx_slot_type ON programme_curriculum_courseslot(type);

-- Junction table indexes (automatically created for foreign keys)
-- programme_curriculum_course_disciplines
-- programme_curriculum_course_pre_requisit_courses
-- programme_curriculum_courseslot_courses
-- programme_curriculum_newproposalfile_pre_requisit_courses

-- Proposal tracking indexes
CREATE INDEX idx_proposal_file ON programme_curriculum_proposal_tracking(file_id);
CREATE INDEX idx_proposal_user ON programme_curriculum_proposal_tracking(receive_id_id);
CREATE INDEX idx_proposal_discipline ON programme_curriculum_proposal_tracking(disciplines_id);
CREATE INDEX idx_proposal_status ON programme_curriculum_proposal_tracking(is_submitted, is_rejected);

-- Academic hierarchy indexes
CREATE INDEX idx_curriculum_programme ON programme_curriculum_curriculum(programme_id);
CREATE INDEX idx_semester_curriculum ON programme_curriculum_semester(curriculum_id);
CREATE INDEX idx_batch_discipline ON programme_curriculum_batch(discipline_id);
CREATE INDEX idx_batch_curriculum ON programme_curriculum_batch(curriculum_id);
CREATE INDEX idx_batch_year ON programme_curriculum_batch(year);
```

#### Full-Text Search Indexes
```sql
-- Enable full-text search on course content
CREATE INDEX idx_course_name_fulltext ON programme_curriculum_course USING gin(to_tsvector('english', name));
CREATE INDEX idx_course_syllabus_fulltext ON programme_curriculum_course USING gin(to_tsvector('english', syllabus));
CREATE INDEX idx_course_prereq_fulltext ON programme_curriculum_course USING gin(to_tsvector('english', pre_requisits));
```

### 10. **Table Dependencies and Foreign Key Constraints**

#### Foreign Key Relationships Summary:
```sql
-- Course Model Dependencies
programme_curriculum_course
├── Outgoing FKs: None (independent table)
└── Incoming FKs:
    ├── programme_curriculum_courseauditlog.course_id
    ├── programme_curriculum_courseinstructor.course_id_id
    ├── programme_curriculum_courseslot_courses.course_id
    ├── programme_curriculum_course_disciplines.course_id
    ├── programme_curriculum_course_pre_requisit_courses.from_course_id
    ├── programme_curriculum_course_pre_requisit_courses.to_course_id
    └── programme_curriculum_newproposalfile_pre_requisit_courses.course_id

-- Audit and Tracking Dependencies
programme_curriculum_courseauditlog
├── course_id → programme_curriculum_course.id
└── user_id → auth_user.id

programme_curriculum_courseinstructor
├── course_id_id → programme_curriculum_course.id
└── instructor_id_id → globals_faculty.id

-- Course Slot Dependencies
programme_curriculum_courseslot
└── semester_id → programme_curriculum_semester.id

programme_curriculum_courseslot_courses
├── courseslot_id → programme_curriculum_courseslot.id
└── course_id → programme_curriculum_course.id

-- Academic Hierarchy Dependencies
programme_curriculum_semester
└── curriculum_id → programme_curriculum_curriculum.id

programme_curriculum_curriculum
└── programme_id → programme_curriculum_programme.id

programme_curriculum_batch
├── discipline_id → globals_discipline.id
└── curriculum_id → programme_curriculum_curriculum.id

-- Proposal System Dependencies
programme_curriculum_proposal_tracking
├── receive_id_id → auth_user.id
├── receive_design_id → globals_designation.id
└── disciplines_id → globals_discipline.id

programme_curriculum_newproposalfile_pre_requisit_courses
├── newproposalfile_id → programme_curriculum_newproposalfile.id
└── course_id → programme_curriculum_course.id
```

#### Cascade Deletion Rules:
```sql
-- Critical CASCADE relationships affecting Course model:
ON DELETE CASCADE:
- courseauditlog → course (audit history deleted with course)
- courseinstructor → course (instructor assignments deleted)
- courseinstructor → faculty (assignments deleted when faculty removed)
- courseslot_courses → course (slot assignments deleted)
- course_disciplines → course (discipline associations deleted)
- course_pre_requisit_courses → course (prerequisites deleted)
```

---

## Version Control System

### 1. **Semantic Versioning Implementation**

#### Version Bump Algorithm
```python
def determine_version_bump_type(old_course, new_data):
    """Determine the type of version bump needed based on changes"""
    
    # MAJOR version changes (X.0)
    major_fields = ['code']
    if any(getattr(old_course, field) != new_data.get(field) for field in major_fields):
        if not is_typo_correction(getattr(old_course, 'code'), new_data.get('code')):
            return 'MAJOR'
    
    # MINOR version changes (X.Y)
    minor_fields = [
        'credit', 'lecture_hours', 'tutorial_hours', 'pratical_hours',
        'discussion_hours', 'project_hours', 'pre_requisits', 'syllabus',
        'percent_quiz_1', 'percent_midsem', 'percent_quiz_2', 'percent_endsem',
        'percent_project', 'percent_lab_evaluation', 'percent_course_attendance',
        'ref_books', 'max_seats'
    ]
    
    for field in minor_fields:
        old_value = getattr(old_course, field)
        new_value = new_data.get(field)
        if old_value != new_value:
            if field == 'name' and is_typo_correction(old_value, new_value):
                continue  # Skip typo corrections for name
            return 'MINOR'
    
    # PATCH version changes (X.Y+0.1)
    patch_fields = ['name']
    if any(getattr(old_course, field) != new_data.get(field) for field in patch_fields):
        if is_typo_correction(getattr(old_course, 'name'), new_data.get('name')):
            return 'PATCH'
    
    return 'NONE'
```

#### Typo Detection Algorithm
```python
def is_typo_correction(old_text, new_text):
    """Detect if text change is likely a typo correction using Levenshtein distance"""
    if not old_text or not new_text:
        return False
    
    # Calculate Levenshtein distance
    distance = levenshtein_distance(old_text.lower(), new_text.lower())
    max_length = max(len(old_text), len(new_text))
    
    # Consider it a typo if distance is less than 20% of the longer string
    # and the absolute distance is small
    similarity_threshold = 0.8
    max_distance_threshold = min(3, max_length // 4)
    
    if distance <= max_distance_threshold and distance / max_length <= (1 - similarity_threshold):
        return True
    
    return False
```

### 2. **Version Management Workflow**

#### New Version Creation
```python
def create_new_version(course, bump_type, changed_data):
    """Create a new version of a course"""
    
    # Calculate new version number
    if bump_type == 'MAJOR':
        new_version = float(int(course.version) + 1)
    elif bump_type == 'MINOR':
        new_version = course.version + 1.0
    elif bump_type == 'PATCH':
        new_version = course.version + 0.1
    else:
        return course  # No version bump needed
    
    # Mark current version as not latest
    course.latest_version = False
    course.save()
    
    # Create new version
    new_course = Course.objects.create(
        code=changed_data.get('code', course.code),
        name=changed_data.get('name', course.name),
        version=new_version,
        credit=changed_data.get('credit', course.credit),
        # ... copy all other fields
        latest_version=True
    )
    
    # Copy many-to-many relationships
    new_course.disciplines.set(course.disciplines.all())
    new_course.pre_requisit_courses.set(course.pre_requisit_courses.all())
    
    return new_course
```

### 3. **Version History Management**

#### Version Tracking
- All previous versions remain in database for historical reference
- Only latest version is marked with `latest_version=True`
- Version history enables rollback and comparison functionality
- Academic records maintain references to specific versions

#### Version Policies
- **Forward Compatibility**: New versions should not break existing prerequisites
- **Backward References**: Historical academic records reference specific versions
- **Migration Support**: Tools for bulk version updates and data migration

---

## Audit Trail Integration

### 1. **CourseAuditLog Model**
```python
class CourseAuditLog(models.Model):
    course = models.ForeignKey('Course', on_delete=models.CASCADE, related_name='audit_logs')
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    action = models.CharField(max_length=10, choices=ACTION_CHOICES)
    old_data = models.JSONField(null=True, blank=True)
    new_data = models.JSONField(null=True, blank=True)
    timestamp = models.DateTimeField(auto_now_add=True)
    version_bump_type = models.CharField(max_length=10, choices=VERSION_BUMP_CHOICES, null=True, blank=True)
    comments = models.TextField(blank=True)
```

### 2. **Audit Integration Points**

#### Automatic Audit Logging
```python
def log_course_change(course, user, action, old_data=None, new_data=None, version_bump_type=None, comments=""):
    """Create audit log entry for course changes"""
    CourseAuditLog.objects.create(
        course=course,
        user=user,
        action=action,
        old_data=old_data,
        new_data=new_data,
        version_bump_type=version_bump_type,
        comments=comments
    )
```

#### Change Detection
- Tracks all field modifications
- Records user responsible for changes
- Maintains complete change history
- Supports regulatory compliance and academic oversight

### 3. **Audit Trail Features**

#### Comprehensive Tracking
- **CREATE**: New course creation with initial data
- **UPDATE**: All field modifications with before/after comparison
- **DELETE**: Course removal (typically just marking as inactive)
- **VERSION**: Version bump operations with reasoning

#### Data Preservation
- **JSON Storage**: Complete field state preservation
- **User Attribution**: Links changes to specific users
- **Timestamp Precision**: Exact change timing
- **Comment Support**: Additional context for changes

---

## Business Logic & Methods

### 1. **Model Methods**

#### String Representation
```python
def __str__(self):
    return str(self.code + " - " + self.name+"- v"+str(self.version))
```
**Output Example**: "CS101 - Introduction to Computer Science - v1.0"

#### CourseSlots Property
```python
@property
def courseslots(self):
    return CourseSlot.objects.filter(courses=self.id)
```
**Purpose**: Retrieve all course slots where this course is offered
**Usage**: Scheduling and timetable generation

### 2. **Custom Managers and QuerySets**

#### Active Courses Manager
```python
class ActiveCourseManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(working_course=True)

class LatestVersionManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(latest_version=True)

# Usage in Course model:
objects = models.Manager()  # Default manager
active = ActiveCourseManager()  # Only active courses
latest = LatestVersionManager()  # Only latest versions
```

### 3. **Business Rule Validation**

#### Assessment Percentage Validation
```python
def validate_assessment_percentages(self):
    """Ensure assessment percentages sum to exactly 100%"""
    total = (
        self.percent_quiz_1 + self.percent_quiz_2 + 
        self.percent_midsem + self.percent_endsem + 
        self.percent_project + self.percent_lab_evaluation + 
        self.percent_course_attendance
    )
    return total == 100
```

#### Prerequisite Cycle Detection
```python
def check_prerequisite_cycles(self):
    """Detect circular dependencies in prerequisites"""
    visited = set()
    rec_stack = set()
    
    def has_cycle(course_id):
        if course_id in rec_stack:
            return True
        if course_id in visited:
            return False
        
        visited.add(course_id)
        rec_stack.add(course_id)
        
        course = Course.objects.get(id=course_id)
        for prereq in course.pre_requisit_courses.all():
            if has_cycle(prereq.id):
                return True
        
        rec_stack.remove(course_id)
        return False
    
    return has_cycle(self.id)
```

---

## Course Lifecycle Management

### 1. **Course States and Transitions**

#### Course Lifecycle States
1. **DRAFT**: Under development, not yet approved
2. **ACTIVE**: Currently offered and available for registration
3. **INACTIVE**: Temporarily not offered (working_course=False)
4. **DEPRECATED**: Older version, superseded by newer version
5. **ARCHIVED**: Permanently discontinued

#### State Transition Rules
```python
def can_transition_to(self, new_state):
    """Check if course can transition to new state"""
    current_state = self.get_current_state()
    
    valid_transitions = {
        'DRAFT': ['ACTIVE', 'ARCHIVED'],
        'ACTIVE': ['INACTIVE', 'DEPRECATED'],
        'INACTIVE': ['ACTIVE', 'ARCHIVED'],
        'DEPRECATED': ['ARCHIVED'],
        'ARCHIVED': []  # No transitions from archived
    }
    
    return new_state in valid_transitions.get(current_state, [])
```

### 2. **Course Approval Workflow**

#### Multi-Stage Approval Process
1. **Faculty Proposal**: Instructor submits course proposal
2. **Department Review**: Department head reviews content and requirements
3. **Curriculum Committee**: Academic committee approves curriculum alignment
4. **Academic Council**: Final approval for course offering
5. **Activation**: Course becomes available for scheduling and registration

#### Workflow Integration
- Links with `NewProposalFile` and `Proposal_Tracking` models
- Maintains approval history and decision rationale
- Supports workflow reversal and modification requests

### 3. **Course Retirement Process**

#### Graceful Deprecation
- Mark older versions as `latest_version=False`
- Set `working_course=False` for discontinued courses
- Maintain data for historical academic records
- Provide migration path for students in progress

---

## Integration with Academic System

### 1. **Programme and Curriculum Integration**

#### Hierarchical Structure
```
Programme (B.Tech, M.Tech, PhD)
  └── Curriculum (specific academic plan)
      └── Semester (academic term)
          └── CourseSlot (time-specific offering)
              └── Course (specific course version)
```

#### Integration Points
- Courses belong to multiple disciplines
- CourseSlots link courses to specific semesters
- Batch assignments determine student access
- Prerequisites enforce academic sequencing

### 2. **Registration System Integration**

#### Student Registration Flow
1. **Eligibility Check**: Verify prerequisites completion
2. **Capacity Check**: Ensure seats available (max_seats)
3. **Academic Rules**: Validate credit limits and program requirements
4. **Conflict Resolution**: Check time and prerequisite conflicts
5. **Registration Completion**: Enroll student in specific course version

#### Data Consistency
- Real-time seat availability tracking
- Prerequisite validation across course versions
- Academic calendar synchronization
- Grade and transcript integration

### 3. **Faculty and Resource Management**

#### CourseInstructor Assignment
- Track instructor assignments per semester
- Support team teaching and multiple instructors
- Maintain teaching load calculations
- Enable instructor evaluation and feedback

#### Resource Allocation
- Classroom assignment based on enrollment
- Lab resource booking for practical hours
- Equipment and material planning
- Academic calendar integration

---

## Data Integrity & Security

### 1. **Data Integrity Measures**

#### Database Constraints
- **Foreign Key Constraints**: Maintain referential integrity
- **Unique Constraints**: Prevent duplicate course code-version combinations
- **Check Constraints**: Validate data ranges and business rules
- **Not Null Constraints**: Ensure required fields are populated

#### Business Rule Enforcement
```python
class CourseValidationMixin:
    def validate_credit_hours_alignment(self):
        """Ensure credit hours align with contact hours"""
        total_contact_hours = (
            (self.lecture_hours or 0) + 
            (self.tutorial_hours or 0) + 
            (self.pratical_hours or 0) + 
            (self.discussion_hours or 0) + 
            (self.project_hours or 0)
        )
        
        # Typical rule: 1 credit = 15-16 contact hours per semester
        expected_hours = self.credit * 15
        tolerance = self.credit * 2  # Allow some flexibility
        
        if abs(total_contact_hours - expected_hours) > tolerance:
            raise ValidationError(
                f"Total contact hours ({total_contact_hours}) don't align with credit hours ({self.credit})"
            )
```

### 2. **Access Control and Permissions**

#### Role-Based Access Control
- **Faculty**: Can view assigned courses, submit proposals
- **Department Head**: Can approve departmental courses
- **Curriculum Committee**: Can modify course requirements
- **Registrar**: Can manage all courses and versions
- **Students**: Read-only access to current course information

#### Permission Matrix
```python
COURSE_PERMISSIONS = {
    'view_course': ['faculty', 'student', 'admin'],
    'add_course': ['faculty', 'admin'],
    'change_course': ['faculty', 'department_head', 'admin'],
    'delete_course': ['admin'],
    'approve_course': ['department_head', 'curriculum_committee'],
    'version_course': ['faculty', 'admin'],
}
```

### 3. **Data Security Measures**

#### Audit Trail Security
- Tamper-proof audit logging
- Cryptographic signatures for critical changes
- Regular backup and recovery procedures
- Access logging for sensitive operations

#### Data Privacy
- Personal information protection in course-related data
- Student record confidentiality
- Faculty evaluation privacy
- Secure data transmission and storage

---

## Performance Considerations

### 1. **Database Optimization**

#### Query Optimization
```python
# Efficient course retrieval with related data
def get_course_with_details(course_id):
    return Course.objects.select_related().prefetch_related(
        'disciplines',
        'pre_requisit_courses',
        'courseslots__semester__curriculum__programme',
        'audit_logs__user'
    ).get(id=course_id)
```

#### Index Strategy
- **Composite Indexes**: (code, version) for version lookups
- **Partial Indexes**: latest_version=True for current course queries
- **Text Indexes**: Full-text search on syllabus and course names
- **Foreign Key Indexes**: Automatic indexes on relationship fields

### 2. **Caching Strategy**

#### Application-Level Caching
```python
from django.core.cache import cache

def get_active_courses_cached():
    """Get active courses with caching"""
    cache_key = 'active_courses_list'
    courses = cache.get(cache_key)
    
    if courses is None:
        courses = Course.active.filter(latest_version=True).select_related(
            'disciplines'
        ).prefetch_related('pre_requisit_courses')
        cache.set(cache_key, courses, timeout=3600)  # Cache for 1 hour
    
    return courses
```

#### Cache Invalidation
- Invalidate course caches on create/update/delete operations
- Version-specific cache keys for historical data
- Dependency-aware cache invalidation for related models

### 3. **Scalability Considerations**

#### Data Partitioning
- Partition by academic year for historical data
- Separate tables for archived courses
- Efficient archival and purging strategies

#### Load Distribution
- Read replicas for heavy query workloads
- Connection pooling for database efficiency
- Asynchronous processing for non-critical operations

#### Monitoring and Metrics
```python
# Performance monitoring for course operations
import time
from django.db import connection

def monitor_course_queries():
    """Monitor database query performance"""
    start_time = time.time()
    query_count_start = len(connection.queries)
    
    # Perform course operations
    yield
    
    end_time = time.time()
    query_count_end = len(connection.queries)
    
    print(f"Execution time: {end_time - start_time:.2f}s")
    print(f"Database queries: {query_count_end - query_count_start}")
```

---

## Conclusion

The Course model in Fusion IIIT represents a sophisticated, enterprise-grade solution for academic course management. Its design encompasses:

### Key Strengths
1. **Comprehensive Data Model**: Captures all aspects of academic course information
2. **Intelligent Versioning**: Semantic versioning with automated change detection
3. **Robust Relationships**: Complex many-to-many and hierarchical associations
4. **Audit Trail Integration**: Complete change tracking and accountability
5. **Business Logic Integration**: Sophisticated validation and workflow support
6. **Performance Optimization**: Efficient querying and caching strategies
7. **Security and Integrity**: Comprehensive data protection and validation

### Architectural Excellence
- **Separation of Concerns**: Clear distinction between data storage, business logic, and presentation
- **Extensibility**: Easy to extend with additional fields and relationships
- **Maintainability**: Well-structured code with clear documentation
- **Scalability**: Designed to handle institutional-scale data volumes
- **Integration**: Seamless integration with broader academic management system

### Future Enhancement Potential
- Machine learning integration for course recommendation
- Advanced analytics for academic planning
- API-first design for mobile and external integrations
- Blockchain integration for credential verification
- AI-powered course content analysis and optimization

This comprehensive model serves as the foundation for a complete academic management ecosystem, supporting all aspects of course lifecycle management from initial proposal through final archival.
