# Complete Theoretical Documentation: Academic Procedures Models in Fusion IIIT

## Table of Contents
1. [Model Architecture Overview](#model-architecture-overview)
2. [Registration Management System](#registration-management-system)
3. [Branch Change System](#branch-change-system)
4. [Course Specialization Management](#course-specialization-management)
5. [Credit Management System](#credit-management-system)
6. [Registration Check System](#registration-check-system)
7. [Thesis Management System](#thesis-management-system)
8. [Fee Payment System](#fee-payment-system)
9. [Teaching Credit System](#teaching-credit-system)
10. [Marks and Grade Management](#marks-and-grade-management)

---

## Model Architecture Overview

### System Purpose
The Academic Procedures system serves as the **operational backbone** for all academic transactions and procedures in Fusion IIIT. It manages student registrations, course enrollments, grade processing, fee payments, thesis supervision, and various academic workflows that support the day-to-day academic operations.

### Core Components
1. **Registration Management**: Student course registration and enrollment tracking
2. **Branch Change System**: Inter-departmental transfer procedures
3. **Course Specialization**: M.Tech specialization course mapping
4. **Credit Management**: Minimum credit requirements per semester
5. **Registration Checks**: Pre and final registration status tracking
6. **Thesis Management**: Research supervision and topic approval workflows
7. **Fee Payment System**: Financial transaction processing and verification
8. **Teaching Credit**: Teaching assistantship and credit allocation
9. **Marks Management**: Grade submission, verification, and announcement
10. **Due Management**: Outstanding fee and penalty tracking

### Architecture Philosophy
- **Workflow-Centric**: Designed around academic procedure workflows
- **State Management**: Tracks various states of academic processes
- **Integration-Ready**: Seamlessly integrates with academic_information and programme_curriculum
- **Legacy Support**: Maintains compatibility with older registration systems
- **Audit Trail**: Comprehensive tracking of academic transactions

---

## Registration Management System

### 1. **Register Model - Basic Course Registration**

```python
class Register(models.Model):
    curr_id = models.ForeignKey(Curriculum, on_delete=models.CASCADE)
    year = models.IntegerField(default=datetime.datetime.now().year)
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    semester = models.IntegerField()
```

#### **Field-by-Field Analysis:**

#### **curr_id** - Course Reference
```python
curr_id = models.ForeignKey(Curriculum, on_delete=models.CASCADE)
```
**Technical Specifications:**
- **Relationship Type**: ForeignKey to academic_information.Curriculum
- **Cascade Behavior**: CASCADE (registration deleted when curriculum deleted)
- **Purpose**: Links registration to specific course offering

**Business Logic:**
- References complete curriculum information including course, credits, semester
- Provides access to course type, programme, and branch information
- Enables course-specific registration validation and tracking
- Supports prerequisite checking and academic progression rules

#### **year** - Academic Year
```python
year = models.IntegerField(default=datetime.datetime.now().year)
```
**Technical Specifications:**
- **Data Type**: Integer (4-digit year)
- **Default**: Current year from system date
- **Purpose**: Tracks academic year for registration

**Business Logic:**
- Determines which academic calendar applies to registration
- Enables year-wise registration analysis and reporting
- Supports multi-year course tracking and academic progression
- Used for graduation requirements and transcript generation

#### **student_id** - Student Reference
```python
student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
```
**Student Integration:**
- Links to complete student profile and academic history
- Provides access to programme, batch, and academic standing
- Enables eligibility validation and prerequisite checking
- Supports student-specific registration rules and restrictions

#### **semester** - Semester Number
```python
semester = models.IntegerField()
```
**Semester Tracking:**
- Indicates which semester the registration applies to
- Enables semester-wise course planning and validation
- Supports academic progression tracking
- Used for graduation requirement verification

#### **Unique Constraint**
```python
class Meta:
    db_table = 'Register'
    unique_together = ['curr_id','student_id']
```
**Data Integrity:**
- Prevents duplicate registrations for same course
- Ensures one registration per student per course
- Maintains clean registration records

### 2. **course_registration Model - Modern Registration System**

```python
class course_registration(models.Model):
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    working_year = models.IntegerField(null=True, blank=True, choices=Year_Choices)
    semester_id = models.ForeignKey(Semester, on_delete=models.CASCADE)
    course_id = models.ForeignKey(Courses, on_delete=models.CASCADE)
    course_slot_id = models.ForeignKey(CourseSlot, null=True, blank=True, on_delete=models.SET_NULL)
    registration_type = models.CharField(max_length=20, choices=REGISTRATION_TYPE_CHOICES, default='Regular')
    session = models.CharField(max_length=9, null=True)
    semester_type = models.CharField(max_length=20, choices=SEMESTER_TYPE_CHOICES, null=True)
```

#### **Enhanced Registration Features:**

#### **working_year** - Academic Working Year
```python
working_year = models.IntegerField(null=True, blank=True, choices=Year_Choices)
```
**Year Choices:**
```python
Year_Choices = [
    (datetime.date.today().year, datetime.date.today().year),
    (datetime.date.today().year-1, datetime.date.today().year-1)
]
```
**Academic Planning:**
- Tracks specific academic year for registration
- Supports current and previous year registrations
- Enables backlog and improvement course handling

#### **semester_id** - Modern Semester Reference
```python
semester_id = models.ForeignKey(Semester, on_delete=models.CASCADE)
```
**Integration Benefits:**
- Links to programme_curriculum.Semester system
- Provides structured semester information with curriculum context
- Enables modern academic planning and scheduling
- Supports batch-wise semester management

#### **course_id** - Modern Course Reference
```python
course_id = models.ForeignKey(Courses, on_delete=models.CASCADE)
```
**Advanced Course Integration:**
- References programme_curriculum.Course with versioning
- Provides access to detailed course information and prerequisites
- Supports modern course management features
- Enables version-specific course registrations

#### **course_slot_id** - Course Slot Assignment
```python
course_slot_id = models.ForeignKey(CourseSlot, null=True, blank=True, on_delete=models.SET_NULL)
```
**Scheduling Integration:**
- Links registration to specific course offering slot
- Provides course type and scheduling information
- Supports capacity management and conflict resolution
- Optional field for flexible registration handling

#### **registration_type** - Registration Classification
```python
REGISTRATION_TYPE_CHOICES = [
    ('Audit', 'Audit'),
    ('Improvement', 'Improvement'),
    ('Backlog', 'Backlog'),
    ('Regular', 'Regular'),
    ('Extra Credits', 'Extra Credits'),
    ('Replacement', 'Replacement'),
]
registration_type = models.CharField(max_length=20, choices=REGISTRATION_TYPE_CHOICES, default='Regular')
```

**Registration Type Analysis:**
- **Regular**: Standard course registration for degree requirements
- **Audit**: Non-credit course attendance for learning purposes
- **Improvement**: Re-taking course to improve grade
- **Backlog**: Failed course being re-attempted
- **Extra Credits**: Additional courses beyond minimum requirements
- **Replacement**: Course substitution with different course

#### **session** - Academic Session
```python
session = models.CharField(max_length=9, null=True)
```
**Session Format:** "2023-2024", "2024-2025"
**Session Management:**
- Tracks multi-year academic sessions
- Enables session-wise academic planning
- Supports session overlap and transition management

#### **semester_type** - Semester Classification
```python
SEMESTER_TYPE_CHOICES = [
    ("Odd Semester", "Odd Semester"),
    ("Even Semester", "Even Semester"),
    ("Summer Semester", "Summer Semester"),
]
semester_type = models.CharField(max_length=20, choices=SEMESTER_TYPE_CHOICES, null=True)
```

**Semester Type Benefits:**
- **Odd Semester**: July-December academic period
- **Even Semester**: January-June academic period  
- **Summer Semester**: Special short-term courses
- Enables semester-specific course offerings and planning

#### **Enhanced Unique Constraint**
```python
class Meta:
    db_table = 'course_registration'
    unique_together = ('course_id', 'student_id', 'semester_id', 'registration_type')
```
**Advanced Data Integrity:**
- Prevents duplicate registrations with same type
- Allows multiple registration types for same course
- Supports complex registration scenarios

---

## Branch Change System

### 1. **BranchChange Model**

```python
class BranchChange(models.Model):
    c_id = models.AutoField(primary_key=True)
    branches = models.ForeignKey(DepartmentInfo, on_delete=models.CASCADE)
    user = models.ForeignKey(Student, on_delete=models.CASCADE)
    applied_date = models.DateField(default=datetime.datetime.now)
```

#### **Field Analysis:**

#### **c_id** - Change Request ID
```python
c_id = models.AutoField(primary_key=True)
```
**Unique Identification:**
- Custom primary key for branch change requests
- Auto-incrementing integer for sequential tracking
- Enables easy reference and tracking of change requests

#### **branches** - Target Department
```python
branches = models.ForeignKey(DepartmentInfo, on_delete=models.CASCADE)
```
**Department Integration:**
- Links to globals.DepartmentInfo for complete department details
- Provides access to department name, head, and policies
- Enables department-specific branch change rules
- Supports inter-departmental approval workflows

#### **user** - Student Applicant
```python
user = models.ForeignKey(Student, on_delete=models.CASCADE)
```
**Student Context:**
- References student academic record and current programme
- Provides access to current branch and academic performance
- Enables eligibility validation for branch change
- Supports student-specific change history tracking

#### **applied_date** - Application Date
```python
applied_date = models.DateField(default=datetime.datetime.now)
```
**Timeline Tracking:**
- Records when branch change request was submitted
- Enables deadline and processing time tracking
- Supports academic calendar alignment
- Used for statistical analysis and reporting

### 2. **Branch Change Workflow**

#### **Business Logic Implementation:**
```python
def validate_branch_change_eligibility(student, target_department):
    """Validate student eligibility for branch change"""
    
    # Check current academic standing
    if student.cpi < 7.0:  # Minimum CPI requirement
        return False, "Minimum CPI of 7.0 required for branch change"
    
    # Check if in eligible semester
    if student.curr_semester_no > 2:
        return False, "Branch change only allowed after first year"
    
    # Check if target department has capacity
    current_batch_size = Student.objects.filter(
        batch=student.batch,
        programme=student.programme,
        id__department=target_department
    ).count()
    
    if current_batch_size >= target_department.max_capacity:
        return False, "Target department at full capacity"
    
    return True, "Eligible for branch change"
```

---

## Course Specialization Management

### 1. **CoursesMtech Model**

```python
class CoursesMtech(models.Model):
    c_id = models.ForeignKey(Course, on_delete=models.CASCADE)
    specialization = models.CharField(max_length=40, choices=Constants.MTechSpecialization)
```

#### **Field Analysis:**

#### **c_id** - Course Reference
```python
c_id = models.ForeignKey(Course, on_delete=models.CASCADE)
```
**Course Integration:**
- Links to academic_information.Course for basic course details
- Provides access to course name and description
- Enables course-specialization mapping
- Supports specialization-specific curriculum design

#### **specialization** - M.Tech Specialization
```python
specialization = models.CharField(max_length=40, choices=Constants.MTechSpecialization)
```

**Available Specializations:**
```python
MTechSpecialization = (
    ('Power and Control', 'Power and Control'),
    ('Microwave and Communication Engineering', 'Microwave and Communication Engineering'),
    ('Micro-nano Electronics', 'Micro-nano Electronics'),
    ('CAD/CAM', 'CAD/CAM'),
    ('Design', 'Design'),
    ('Manufacturing', 'Manufacturing'),
    ('CSE', 'CSE'),
    ('Mechatronics', 'Mechatronics'),
    ('MDes', 'MDes'),
    ('all', 'all')
)
```

**Specialization Mapping Benefits:**
- **Power and Control**: Electrical engineering power systems focus
- **Microwave and Communication**: High-frequency electronics and communication
- **Micro-nano Electronics**: Semiconductor and nanotechnology
- **CAD/CAM**: Computer-aided design and manufacturing
- **Design**: Product and system design specialization
- **Manufacturing**: Production and industrial engineering
- **CSE**: Computer science and engineering
- **Mechatronics**: Mechanical-electrical systems integration
- **MDes**: Master of Design programme
- **all**: Common courses across all specializations

---

## Credit Management System

### 1. **MinimumCredits Model**

```python
class MinimumCredits(models.Model):
    semester = models.IntegerField()
    credits = models.IntegerField()
```

#### **Field Analysis:**

#### **semester** - Semester Number
```python
semester = models.IntegerField()
```
**Semester Identification:**
- Numeric semester designation (1, 2, 3, ...)
- Enables semester-specific credit requirements
- Supports programme-wise credit planning
- Used for graduation requirement validation

#### **credits** - Minimum Credit Requirement
```python
credits = models.IntegerField()
```
**Credit Requirements:**
- Minimum credits required for semester registration
- Ensures adequate academic load per semester
- Supports degree completion planning
- Used for registration eligibility validation

#### **Credit Management Business Logic:**
```python
def validate_minimum_credits(student, semester, registered_credits):
    """Validate student meets minimum credit requirements"""
    
    try:
        min_credits = MinimumCredits.objects.get(semester=semester)
        
        if registered_credits < min_credits.credits:
            return False, f"Minimum {min_credits.credits} credits required, registered: {registered_credits}"
        
        return True, "Credit requirements met"
        
    except MinimumCredits.DoesNotExist:
        return True, "No minimum credit requirement defined for this semester"

def calculate_registered_credits(student, semester):
    """Calculate total credits student is registered for"""
    
    registrations = course_registration.objects.filter(
        student_id=student,
        semester_id__semester_no=semester,
        registration_type__in=['Regular', 'Extra Credits']
    )
    
    total_credits = sum(reg.course_id.credit for reg in registrations)
    return total_credits
```

---

## Registration Check System

### 1. **StudentRegistrationChecks Model - Modern System**

```python
class StudentRegistrationChecks(models.Model):
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    pre_registration_flag = models.BooleanField(default=False)
    final_registration_flag = models.BooleanField(default=False)
    semester_id = models.ForeignKey(Semester, on_delete=models.CASCADE)
```

#### **Field Analysis:**

#### **student_id** - Student Reference
```python
student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
```
**Student Tracking:**
- Links to complete student academic profile
- Enables student-specific registration status tracking
- Supports personalized registration workflows
- Provides access to student programme and batch information

#### **pre_registration_flag** - Pre-Registration Status
```python
pre_registration_flag = models.BooleanField(default=False)
```
**Pre-Registration Phase:**
- **False**: Pre-registration not completed (default)
- **True**: Pre-registration phase completed
- Tracks initial course selection and preference submission
- Enables staged registration process management

#### **final_registration_flag** - Final Registration Status
```python
final_registration_flag = models.BooleanField(default=False)
```
**Final Registration Phase:**
- **False**: Final registration not completed (default)
- **True**: Final registration phase completed
- Tracks confirmed course enrollment after allocation
- Enables registration completion validation

#### **semester_id** - Semester Context
```python
semester_id = models.ForeignKey(Semester, on_delete=models.CASCADE)
```
**Semester Integration:**
- Links to programme_curriculum.Semester system
- Provides structured semester and curriculum context
- Enables semester-specific registration tracking
- Supports batch-wise registration management

### 2. **Registration Workflow Management**

#### **Registration Phase Tracking:**
```python
def get_registration_status(student, semester):
    """Get comprehensive registration status for student"""
    
    try:
        reg_check = StudentRegistrationChecks.objects.get(
            student_id=student,
            semester_id=semester
        )
        
        status = {
            'pre_registration_complete': reg_check.pre_registration_flag,
            'final_registration_complete': reg_check.final_registration_flag,
            'phase': 'not_started'
        }
        
        if reg_check.final_registration_flag:
            status['phase'] = 'completed'
        elif reg_check.pre_registration_flag:
            status['phase'] = 'final_registration'
        else:
            status['phase'] = 'pre_registration'
            
        return status
        
    except StudentRegistrationChecks.DoesNotExist:
        return {
            'pre_registration_complete': False,
            'final_registration_complete': False,
            'phase': 'not_started'
        }

def update_registration_phase(student, semester, phase_completed):
    """Update student registration phase completion"""
    
    reg_check, created = StudentRegistrationChecks.objects.get_or_create(
        student_id=student,
        semester_id=semester
    )
    
    if phase_completed == 'pre_registration':
        reg_check.pre_registration_flag = True
    elif phase_completed == 'final_registration':
        reg_check.final_registration_flag = True
    
    reg_check.save()
    return reg_check
```

### 3. **Legacy Registration Check System**

#### **StudentRegistrationCheck Model (Legacy)**
```python
class StudentRegistrationCheck(models.Model):
    student = models.ForeignKey(Student, on_delete=models.CASCADE)
    pre_registration_flag = models.BooleanField(default=False)
    final_registration_flag = models.BooleanField(default=False)
    semester = models.IntegerField(default=1)
```

**Legacy vs Modern System:**
| Aspect | Legacy System | Modern System |
|--------|---------------|---------------|
| **Semester Reference** | Integer field | ForeignKey to Semester model |
| **Table Name** | Custom 'StudentRegistrationCheck' | Custom 'StudentRegistrationChecks' |
| **Integration** | Basic semester number | Full curriculum integration |
| **Flexibility** | Limited semester context | Rich semester and batch context |
| **Scalability** | Simple integer-based | Structured relational design |

---

This concludes Part 1 of the Academic Procedures documentation covering Registration Management, Branch Change, Course Specialization, Credit Management, and Registration Check systems. 

Should I continue with Part 2 covering Thesis Management, Fee Payment, Teaching Credit, and Marks Management systems?
