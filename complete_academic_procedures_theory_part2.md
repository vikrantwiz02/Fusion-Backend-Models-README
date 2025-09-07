# Complete Theoretical Documentation: Academic Procedures Models - Part 2

## Continuing from Part 1...

## Thesis Management System

### 1. **Thesis Model - Basic Thesis Registration**

```python
class Thesis(models.Model):
    reg_id = models.ForeignKey(ExtraInfo, on_delete=models.CASCADE)
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    supervisor_id = models.ForeignKey(Faculty, on_delete=models.CASCADE)
    topic = models.CharField(max_length=1000)
```

#### **Field Analysis:**

#### **reg_id** - Registration Authority
```python
reg_id = models.ForeignKey(ExtraInfo, on_delete=models.CASCADE)
```
**Administrative Reference:**
- Links to institutional authority who registered the thesis
- Provides access to registration officer details and department
- Enables administrative tracking and accountability
- Supports institutional workflow and approval chains

#### **student_id** - Research Student
```python
student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
```
**Student Integration:**
- References graduate student conducting research
- Provides access to student programme (M.Tech, PhD)
- Enables student-specific thesis requirements and timelines
- Supports academic progression and degree completion tracking

#### **supervisor_id** - Research Supervisor
```python
supervisor_id = models.ForeignKey(Faculty, on_delete=models.CASCADE)
```
**Faculty Supervision:**
- Links to supervising faculty member
- Provides access to faculty qualifications and research areas
- Enables supervisor workload tracking and management
- Supports faculty-student supervision relationship

#### **topic** - Research Topic
```python
topic = models.CharField(max_length=1000)
```
**Research Focus:**
- Detailed description of thesis research topic
- Maximum 1000 characters for comprehensive topic description
- Enables research area classification and analysis
- Supports thesis database and knowledge management

### 2. **ThesisTopicProcess Model - Advanced Thesis Workflow**

```python
class ThesisTopicProcess(models.Model):
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    research_area = models.CharField(max_length=50)
    thesis_topic = models.CharField(max_length=1000)
    curr_id = models.ForeignKey(Curriculum, on_delete=models.CASCADE, null=True)
    supervisor_id = models.ForeignKey(Faculty, on_delete=models.CASCADE, related_name='%(class)s_supervisor')
    co_supervisor_id = models.ForeignKey(Faculty, on_delete=models.CASCADE, related_name='%(class)s_co_supervisor', null=True)
    submission_by_student = models.BooleanField(default=False)
    pending_supervisor = models.BooleanField(default=True)
    member1 = models.ForeignKey(Faculty, on_delete=models.CASCADE, related_name='%(class)s_member1', null=True)
    member2 = models.ForeignKey(Faculty, on_delete=models.CASCADE, related_name='%(class)s_member2', null=True)
    member3 = models.ForeignKey(Faculty, on_delete=models.CASCADE, related_name='%(class)s_member3', null=True)
    approval_supervisor = models.BooleanField(default=False)
    forwarded_to_hod = models.BooleanField(default=False)
    pending_hod = models.BooleanField(default=True)
    approval_by_hod = models.BooleanField(default=False)
    date = models.DateField(default=datetime.datetime.now)
```

#### **Enhanced Thesis Management Features:**

#### **research_area** - Research Domain
```python
research_area = models.CharField(max_length=50)
```
**Research Classification:**
- Broad research domain categorization
- Enables research area analysis and faculty expertise matching
- Supports interdisciplinary research tracking
- Used for research trend analysis and resource allocation

#### **curr_id** - Curriculum Integration
```python
curr_id = models.ForeignKey(Curriculum, on_delete=models.CASCADE, null=True)
```
**Academic Integration:**
- Links thesis to specific curriculum course (thesis credit)
- Provides academic context and credit information
- Enables thesis as course completion requirement
- Supports academic record and transcript integration

#### **Supervision Structure:**
```python
supervisor_id = models.ForeignKey(Faculty, on_delete=models.CASCADE, related_name='%(class)s_supervisor')
co_supervisor_id = models.ForeignKey(Faculty, on_delete=models.CASCADE, related_name='%(class)s_co_supervisor', null=True)
```
**Supervision Hierarchy:**
- **supervisor_id**: Primary research supervisor (mandatory)
- **co_supervisor_id**: Secondary supervisor for interdisciplinary research (optional)
- Related names prevent foreign key conflicts in supervision relationships
- Enables dual supervision for complex research projects

#### **Committee Structure:**
```python
member1 = models.ForeignKey(Faculty, on_delete=models.CASCADE, related_name='%(class)s_member1', null=True)
member2 = models.ForeignKey(Faculty, on_delete=models.CASCADE, related_name='%(class)s_member2', null=True)
member3 = models.ForeignKey(Faculty, on_delete=models.CASCADE, related_name='%(class)s_member3', null=True)
```
**Evaluation Committee:**
- Three committee members for thesis evaluation
- Optional fields allowing flexible committee size
- Related names ensure unique relationships
- Supports comprehensive thesis evaluation process

#### **Workflow State Management:**
```python
submission_by_student = models.BooleanField(default=False)
pending_supervisor = models.BooleanField(default=True)
approval_supervisor = models.BooleanField(default=False)
forwarded_to_hod = models.BooleanField(default=False)
pending_hod = models.BooleanField(default=True)
approval_by_hod = models.BooleanField(default=False)
```

**Thesis Approval Workflow:**
1. **submission_by_student**: Student submits thesis topic proposal
2. **pending_supervisor**: Awaiting supervisor review (default state)
3. **approval_supervisor**: Supervisor approves thesis topic
4. **forwarded_to_hod**: Forwarded to Head of Department
5. **pending_hod**: Awaiting HOD review
6. **approval_by_hod**: Final HOD approval

#### **Workflow Business Logic:**
```python
def get_thesis_workflow_status(thesis_process):
    """Determine current workflow status"""
    
    if not thesis_process.submission_by_student:
        return "awaiting_student_submission"
    elif thesis_process.pending_supervisor and not thesis_process.approval_supervisor:
        return "pending_supervisor_review"
    elif thesis_process.approval_supervisor and not thesis_process.forwarded_to_hod:
        return "approved_by_supervisor"
    elif thesis_process.pending_hod and not thesis_process.approval_by_hod:
        return "pending_hod_review"
    elif thesis_process.approval_by_hod:
        return "fully_approved"
    else:
        return "status_unclear"

def advance_thesis_workflow(thesis_process, action, actor):
    """Advance thesis through approval workflow"""
    
    if action == "student_submit":
        thesis_process.submission_by_student = True
        thesis_process.pending_supervisor = True
        
    elif action == "supervisor_approve":
        thesis_process.approval_supervisor = True
        thesis_process.pending_supervisor = False
        thesis_process.forwarded_to_hod = True
        thesis_process.pending_hod = True
        
    elif action == "hod_approve":
        thesis_process.approval_by_hod = True
        thesis_process.pending_hod = False
        
    thesis_process.save()
    return thesis_process
```

---

## Fee Payment System

### 1. **FeePayments Model - Modern Payment System**

```python
class FeePayments(models.Model):
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    semester_id = models.ForeignKey(Semester, on_delete=models.CASCADE)
    mode = models.CharField(max_length=20, choices=Constants.PaymentMode)
    transaction_id = models.CharField(max_length=40)
    fee_receipt = models.FileField(null=True, upload_to='fee_receipt/')
    deposit_date = models.DateField(default=datetime.date.today)
    utr_number = models.CharField(null=True, max_length=40)
    fee_paid = models.IntegerField(default=0)
    reason = models.CharField(null=True, max_length=20)
    actual_fee = models.IntegerField(default=0)
```

#### **Field Analysis:**

#### **student_id** - Student Reference
```python
student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
```
**Student Financial Tracking:**
- Links payment to specific student account
- Enables student-wise fee tracking and history
- Supports financial aid and scholarship integration
- Provides access to student programme and batch for fee calculation

#### **semester_id** - Semester Context
```python
semester_id = models.ForeignKey(Semester, on_delete=models.CASCADE)
```
**Semester-wise Fee Management:**
- Links payment to specific semester
- Enables semester-wise fee structure and policies
- Supports variable fee structures across semesters
- Provides curriculum and programme context for fee calculation

#### **mode** - Payment Method
```python
mode = models.CharField(max_length=20, choices=Constants.PaymentMode)
```

**Available Payment Methods:**
```python
PaymentMode = (
    ('Axis Easypay', 'Axis Easypay'),
    ('Subpaisa', 'Subpaisa'),
    ('NEFT', 'NEFT'),
    ('RTGS', 'RTGS'),
    ('Bank Challan', 'Bank Challan'),
    ('Edu Loan', 'Edu Loan')
)
```

**Payment Mode Analysis:**
- **Axis Easypay**: Online payment gateway integration
- **Subpaisa**: Alternative online payment platform
- **NEFT**: National Electronic Funds Transfer
- **RTGS**: Real Time Gross Settlement
- **Bank Challan**: Traditional bank deposit method
- **Edu Loan**: Educational loan disbursement

#### **transaction_id** - Transaction Reference
```python
transaction_id = models.CharField(max_length=40)
```
**Transaction Tracking:**
- Unique identifier for payment transaction
- Enables payment verification and reconciliation
- Supports dispute resolution and audit trail
- Links to external payment gateway records

#### **fee_receipt** - Receipt Document
```python
fee_receipt = models.FileField(null=True, upload_to='fee_receipt/')
```
**Document Management:**
- Stores payment receipt documents (PDF, images)
- Organized file storage in dedicated directory
- Supports payment verification and audit
- Enables student access to payment history

#### **deposit_date** - Payment Date
```python
deposit_date = models.DateField(default=datetime.date.today)
```
**Temporal Tracking:**
- Records actual payment date
- Enables payment timeline analysis
- Supports late fee calculation and penalty management
- Used for financial reporting and cash flow analysis

#### **utr_number** - Bank Reference
```python
utr_number = models.CharField(null=True, max_length=40)
```
**Bank Integration:**
- Unique Transaction Reference number for bank transfers
- Enables bank reconciliation and verification
- Supports automated payment matching
- Required for NEFT/RTGS transactions

#### **Financial Tracking Fields:**
```python
fee_paid = models.IntegerField(default=0)
actual_fee = models.IntegerField(default=0)
reason = models.CharField(null=True, max_length=20)
```

**Financial Analytics:**
- **fee_paid**: Amount actually paid by student
- **actual_fee**: Full fee amount due for semester
- **reason**: Reason for partial payment or fee adjustment
- Enables fee variance analysis and financial planning

### 2. **Legacy Fee Payment System**

#### **FeePayment Model (Legacy)**
```python
class FeePayment(models.Model):
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    semester = models.IntegerField(default=1)
    batch = models.IntegerField(default=2016)
    mode = models.CharField(max_length=20, choices=Constants.PaymentMode)
    transaction_id = models.CharField(max_length=40)
```

**Legacy vs Modern Fee System:**
| Aspect | Legacy System | Modern System |
|--------|---------------|---------------|
| **Semester Reference** | Integer field | ForeignKey to Semester |
| **Document Support** | No receipt storage | File upload for receipts |
| **Bank Integration** | Basic transaction ID | UTR number support |
| **Financial Tracking** | Simple payment record | Detailed amount tracking |
| **Audit Trail** | Limited tracking | Comprehensive payment history |

---

## Teaching Credit System

### 1. **TeachingCreditRegistration Model**

```python
class TeachingCreditRegistration(models.Model):
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    curr_1 = models.ForeignKey(Curriculum, on_delete=models.CASCADE, related_name='%(class)s_curr1')
    curr_2 = models.ForeignKey(Curriculum, on_delete=models.CASCADE, related_name='%(class)s_curr2')
    curr_3 = models.ForeignKey(Curriculum, on_delete=models.CASCADE, related_name='%(class)s_curr3')
    curr_4 = models.ForeignKey(Curriculum, on_delete=models.CASCADE, related_name='%(class)s_curr4')
    req_pending = models.BooleanField(default=True)
    approved_course = models.ForeignKey(Curriculum, on_delete=models.CASCADE, related_name='%(class)s_approved_course', null=True)
    course_completion = models.BooleanField(default=False)
    supervisor_id = models.ForeignKey(Faculty, on_delete=models.CASCADE, related_name='%(class)s_supervisor_id', null=True)
```

#### **Field Analysis:**

#### **student_id** - Teaching Assistant
```python
student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
```
**TA Assignment:**
- References graduate student applying for teaching credits
- Typically M.Tech or PhD students eligible for teaching roles
- Enables student academic and teaching integration
- Supports financial aid through teaching assistantships

#### **Course Preference System:**
```python
curr_1 = models.ForeignKey(Curriculum, on_delete=models.CASCADE, related_name='%(class)s_curr1')
curr_2 = models.ForeignKey(Curriculum, on_delete=models.CASCADE, related_name='%(class)s_curr2')
curr_3 = models.ForeignKey(Curriculum, on_delete=models.CASCADE, related_name='%(class)s_curr3')
curr_4 = models.ForeignKey(Curriculum, on_delete=models.CASCADE, related_name='%(class)s_curr4')
```

**Preference Ranking System:**
- **curr_1**: First preference for teaching assignment
- **curr_2**: Second preference course
- **curr_3**: Third preference course  
- **curr_4**: Fourth preference course
- Related names prevent foreign key conflicts
- Enables preference-based assignment algorithm

#### **Approval Workflow:**
```python
req_pending = models.BooleanField(default=True)
approved_course = models.ForeignKey(Curriculum, on_delete=models.CASCADE, related_name='%(class)s_approved_course', null=True)
course_completion = models.BooleanField(default=False)
```

**Teaching Assignment Process:**
- **req_pending**: Request awaiting approval (default True)
- **approved_course**: Final assigned course after approval
- **course_completion**: Teaching duties completed successfully
- Supports complete teaching credit lifecycle

#### **supervisor_id** - Faculty Supervisor
```python
supervisor_id = models.ForeignKey(Faculty, on_delete=models.CASCADE, related_name='%(class)s_supervisor_id', null=True)
```
**Supervision Structure:**
- Faculty member supervising teaching assistant
- Provides guidance and evaluation for teaching performance
- Enables quality control and mentorship
- Supports academic career development

### 2. **Modern TA Assignment System**

#### **Assignment Model**
```python
class Assignment(models.Model):
    ta = models.ForeignKey(Student, on_delete=models.CASCADE)
    faculty = models.ForeignKey(Faculty, on_delete=models.CASCADE)
    start_year = models.IntegerField()
    start_month = models.IntegerField()  # 1–12
    end_year = models.IntegerField()
    end_month = models.IntegerField()
```

**Enhanced TA Management:**
- **Precise Duration**: Month-level assignment periods
- **Faculty Assignment**: Direct faculty-TA relationship
- **Timeline Management**: Clear start and end dates
- **Flexible Terms**: Supports variable assignment durations

#### **StipendRequest Model**
```python
class StipendRequest(models.Model):
    PENDING = 'pending'
    FAC_APPROVED = 'approved_by_faculty'
    HOD_APPROVED = 'approved_by_hod'
    STATUS_CHOICES = [
        (PENDING, 'Pending'),
        (FAC_APPROVED, 'Approved by Faculty'),
        (HOD_APPROVED, 'Approved by HOD'),
    ]
    
    assignment = models.ForeignKey(Assignment, on_delete=models.CASCADE, related_name='stipends')
    year = models.IntegerField()
    month = models.IntegerField()
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default=PENDING)
    faculty_remark = models.TextField(blank=True, null=True)
    hod_remark = models.TextField(blank=True, null=True)
```

**Monthly Stipend Management:**
- **Monthly Tracking**: Individual month stipend requests
- **Approval Workflow**: Faculty → HOD approval chain
- **Remark System**: Comments for approval/rejection
- **Status Tracking**: Clear approval status progression

#### **Automated Stipend Generation:**
```python
@receiver(post_save, sender=Assignment)
def create_monthly_stipends(sender, instance, created, **kwargs):
    if not created: return
    sy, sm = instance.start_year, instance.start_month
    ey, em = instance.end_year, instance.end_month
    year, month = sy, sm
    while (year<ey) or (year==ey and month<=em):
        StipendRequest.objects.create(assignment=instance, year=year, month=month)
        if month==12:
            month, year = 1, year+1
        else:
            month += 1
```

**Automated Workflow:**
- Creates monthly stipend requests automatically
- Covers entire assignment duration
- Handles year transitions correctly
- Reduces administrative overhead

---

## Marks and Grade Management

### 1. **SemesterMarks Model**

```python
class SemesterMarks(models.Model):
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    q1 = models.FloatField(default=None)
    mid_term = models.FloatField(default=None)
    q2 = models.FloatField(default=None)
    end_term = models.FloatField(default=None)
    other = models.FloatField(default=None)
    grade = models.CharField(max_length=5, choices=Constants.GRADE, null=True)
    curr_id = models.ForeignKey(Courses, on_delete=models.CASCADE)
```

#### **Field Analysis:**

#### **student_id** - Student Reference
```python
student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
```
**Student Performance Tracking:**
- Links marks to specific student record
- Enables comprehensive academic performance analysis
- Supports student-wise grade reporting and transcripts
- Provides access to student programme and batch context

#### **Assessment Component Breakdown:**
```python
q1 = models.FloatField(default=None)
mid_term = models.FloatField(default=None)
q2 = models.FloatField(default=None)
end_term = models.FloatField(default=None)
other = models.FloatField(default=None)
```

**Assessment Components:**
- **q1**: Quiz 1 marks (typically 10-15% weightage)
- **mid_term**: Mid-semester examination marks (20-25% weightage)
- **q2**: Quiz 2 marks (typically 10-15% weightage)
- **end_term**: End-semester examination marks (30-40% weightage)
- **other**: Assignment, project, lab marks (remaining weightage)
- **Float Fields**: Support decimal marks for precise grading

#### **grade** - Final Grade
```python
grade = models.CharField(max_length=5, choices=Constants.GRADE, null=True)
```

**Grade Scale:**
```python
GRADE = (
    ('O','O'),     # Outstanding (90-100%)
    ('A+','A+'),   # Excellent (80-89%)
    ('A','A'),     # Very Good (70-79%)
    ('B+','B+'),   # Good (60-69%)
    ('B','B'),     # Above Average (50-59%)
    ('C+','C+'),   # Average (45-49%)
    ('C','C'),     # Below Average (40-44%)
    ('D+','D+'),   # Poor (35-39%)
    ('D','D'),     # Very Poor (30-34%)
    ('F','F'),     # Fail (Below 30%)
    ('S','S'),     # Satisfactory (Pass/Fail courses)
    ('X','X'),     # Incomplete/Absent
)
```

#### **curr_id** - Course Reference
```python
curr_id = models.ForeignKey(Courses, on_delete=models.CASCADE)
```
**Course Integration:**
- Links to programme_curriculum.Course for modern course system
- Provides access to course details, credits, and version information
- Enables course-specific grading policies and assessment methods
- Supports transcript generation with complete course information

### 2. **MarkSubmissionCheck Model**

```python
class MarkSubmissionCheck(models.Model):
    curr_id = models.ForeignKey(Courses, on_delete=models.CASCADE)
    verified = models.BooleanField(default=False)
    submitted = models.BooleanField(default=False)
    announced = models.BooleanField(default=False)
```

#### **Grade Processing Workflow:**

#### **curr_id** - Course Context
```python
curr_id = models.ForeignKey(Courses, on_delete=models.CASCADE)
```
**Course-Level Status:**
- Tracks mark submission status per course
- Enables course-wise grade processing management
- Supports instructor-specific mark submission workflow
- Provides course context for status tracking

#### **Submission Workflow States:**
```python
submitted = models.BooleanField(default=False)
verified = models.BooleanField(default=False)
announced = models.BooleanField(default=False)
```

**Mark Processing Pipeline:**
1. **submitted**: Instructor submits marks (default False)
2. **verified**: Academic office verifies marks (default False)
3. **announced**: Marks published to students (default False)

#### **Grade Processing Business Logic:**
```python
def process_course_marks(course_id, instructor, marks_data):
    """Process and submit course marks"""
    
    # Create or update mark submission check
    submission_check, created = MarkSubmissionCheck.objects.get_or_create(
        curr_id=course_id
    )
    
    # Process individual student marks
    for student_marks in marks_data:
        semester_marks, created = SemesterMarks.objects.update_or_create(
            student_id=student_marks['student'],
            curr_id=course_id,
            defaults={
                'q1': student_marks.get('q1'),
                'mid_term': student_marks.get('mid_term'),
                'q2': student_marks.get('q2'),
                'end_term': student_marks.get('end_term'),
                'other': student_marks.get('other'),
                'grade': calculate_grade(student_marks)
            }
        )
    
    # Mark as submitted
    submission_check.submitted = True
    submission_check.save()
    
    return submission_check

def calculate_grade(marks_data):
    """Calculate final grade from component marks"""
    
    # Calculate total marks (customize weights as per course policy)
    total_marks = (
        (marks_data.get('q1', 0) or 0) * 0.10 +
        (marks_data.get('mid_term', 0) or 0) * 0.25 +
        (marks_data.get('q2', 0) or 0) * 0.10 +
        (marks_data.get('end_term', 0) or 0) * 0.40 +
        (marks_data.get('other', 0) or 0) * 0.15
    )
    
    # Convert to grade based on percentage
    if total_marks >= 90:
        return 'O'
    elif total_marks >= 80:
        return 'A+'
    elif total_marks >= 70:
        return 'A'
    elif total_marks >= 60:
        return 'B+'
    elif total_marks >= 50:
        return 'B'
    elif total_marks >= 45:
        return 'C+'
    elif total_marks >= 40:
        return 'C'
    elif total_marks >= 35:
        return 'D+'
    elif total_marks >= 30:
        return 'D'
    else:
        return 'F'

def verify_course_marks(course_id, verifier):
    """Verify submitted course marks"""
    
    try:
        submission_check = MarkSubmissionCheck.objects.get(curr_id=course_id)
        
        if not submission_check.submitted:
            return False, "Marks not yet submitted"
        
        submission_check.verified = True
        submission_check.save()
        
        return True, "Marks verified successfully"
        
    except MarkSubmissionCheck.DoesNotExist:
        return False, "No marks submitted for this course"

def announce_course_marks(course_id, announcer):
    """Announce verified marks to students"""
    
    try:
        submission_check = MarkSubmissionCheck.objects.get(curr_id=course_id)
        
        if not submission_check.verified:
            return False, "Marks not yet verified"
        
        submission_check.announced = True
        submission_check.save()
        
        return True, "Marks announced to students"
        
    except MarkSubmissionCheck.DoesNotExist:
        return False, "No marks found for this course"
```

---

This concludes Part 2 of the Academic Procedures documentation covering Thesis Management, Fee Payment, Teaching Credit, and Marks Management systems.

Should I continue with Part 3 covering Due Management, Registration Systems (Initial/Final), Course Replacement, Assistantship, Progress Examinations, and other remaining models?
