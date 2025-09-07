# Complete Theoretical Documentation: Academic Procedures Models - Part 3

## Continuing from Part 2...

## Due Management System

### 1. **Due Model - Student Financial Dues**

```python
class Due(models.Model):
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    amount = models.IntegerField()
    description = models.CharField(max_length=500, null=True)
    due_type = models.CharField(max_length=20, choices=Constants.DueType, null=True)
    status = models.CharField(max_length=20, choices=Constants.DueStatus, default='Unpaid')
    timestamp = models.DateTimeField(auto_now=True)
    semester = models.IntegerField(default=1)
    batch = models.IntegerField(default=2016)
```

#### **Field Analysis:**

#### **student_id** - Student Reference
```python
student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
```
**Student Financial Tracking:**
- Links financial dues to specific student record
- Enables comprehensive student financial management
- Supports student-wise due analysis and payment tracking
- Provides access to student programme and batch for context

#### **amount** - Due Amount
```python
amount = models.IntegerField()
```
**Financial Value:**
- Due amount in institutional currency (typically INR)
- Integer field for precise financial calculations
- Supports positive amounts for dues owed
- Used for financial reporting and analysis

#### **description** - Due Details
```python
description = models.CharField(max_length=500, null=True)
```
**Due Description:**
- Detailed explanation of the due (library fine, hostel charges, etc.)
- Maximum 500 characters for comprehensive description
- Optional field allowing flexible due creation
- Supports clear communication to students about dues

#### **due_type** - Due Classification
```python
due_type = models.CharField(max_length=20, choices=Constants.DueType, null=True)
```

**Due Type Categories:**
```python
DueType = (
    ('Library', 'Library'),
    ('Hostel', 'Hostel'),
    ('Academic', 'Academic'),
    ('Mess', 'Mess'),
    ('Medical', 'Medical'),
    ('Transport', 'Transport'),
    ('Other', 'Other')
)
```

**Due Type Analysis:**
- **Library**: Book fines, overdue charges, damage fees
- **Hostel**: Room charges, maintenance fees, damage costs
- **Academic**: Course fees, exam fees, certificate charges
- **Mess**: Meal charges, mess bill defaults
- **Medical**: Health center charges, medical fees
- **Transport**: Bus fees, parking charges
- **Other**: Miscellaneous institutional charges

#### **status** - Payment Status
```python
status = models.CharField(max_length=20, choices=Constants.DueStatus, default='Unpaid')
```

**Due Status Options:**
```python
DueStatus = (
    ('Paid', 'Paid'),
    ('Unpaid', 'Unpaid'),
    ('Partial', 'Partial'),
    ('Waived', 'Waived'),
    ('Disputed', 'Disputed')
)
```

**Status Workflow:**
- **Unpaid**: Default status, due pending payment
- **Paid**: Due fully settled by student
- **Partial**: Partial payment made, balance remaining
- **Waived**: Due cancelled/forgiven by authority
- **Disputed**: Due contested by student, under review

#### **timestamp** - Record Tracking
```python
timestamp = models.DateTimeField(auto_now=True)
```
**Temporal Management:**
- Automatically updated on every record modification
- Tracks latest status change time
- Supports audit trail and timeline analysis
- Used for due aging and reporting

#### **Academic Context:**
```python
semester = models.IntegerField(default=1)
batch = models.IntegerField(default=2016)
```
**Academic Integration:**
- **semester**: Links due to specific semester
- **batch**: Student batch year for context
- Enables semester-wise due analysis
- Supports batch-based financial reporting

### 2. **Due Management Business Logic**

```python
def create_student_due(student, amount, description, due_type, semester=None, batch=None):
    """Create new due for student"""
    
    # Get student's current semester and batch if not provided
    if not semester:
        semester = get_current_semester(student)
    if not batch:
        batch = student.batch
    
    due = Due.objects.create(
        student_id=student,
        amount=amount,
        description=description,
        due_type=due_type,
        semester=semester,
        batch=batch,
        status='Unpaid'
    )
    
    # Notify student of new due
    send_due_notification(student, due)
    
    return due

def process_due_payment(due_id, payment_amount, payment_method='Online'):
    """Process payment against student due"""
    
    try:
        due = Due.objects.get(id=due_id)
        
        if due.status == 'Paid':
            return False, "Due already paid"
        
        if payment_amount >= due.amount:
            # Full payment
            due.status = 'Paid'
            due.save()
            
            # Create payment record
            create_payment_record(due, payment_amount, payment_method)
            
            return True, "Due paid successfully"
            
        else:
            # Partial payment
            due.status = 'Partial'
            remaining_amount = due.amount - payment_amount
            due.amount = remaining_amount
            due.save()
            
            # Create payment record
            create_payment_record(due, payment_amount, payment_method)
            
            return True, f"Partial payment received. Remaining: {remaining_amount}"
            
    except Due.DoesNotExist:
        return False, "Due not found"

def get_student_dues_summary(student, status=None):
    """Get comprehensive due summary for student"""
    
    dues_query = Due.objects.filter(student_id=student)
    
    if status:
        dues_query = dues_query.filter(status=status)
    
    summary = {
        'total_dues': dues_query.count(),
        'total_amount': dues_query.aggregate(Sum('amount'))['amount__sum'] or 0,
        'by_type': {},
        'by_status': {},
        'by_semester': {}
    }
    
    # Group by due type
    for due_type, _ in Constants.DueType:
        type_dues = dues_query.filter(due_type=due_type)
        summary['by_type'][due_type] = {
            'count': type_dues.count(),
            'amount': type_dues.aggregate(Sum('amount'))['amount__sum'] or 0
        }
    
    # Group by status
    for status, _ in Constants.DueStatus:
        status_dues = dues_query.filter(status=status)
        summary['by_status'][status] = {
            'count': status_dues.count(),
            'amount': status_dues.aggregate(Sum('amount'))['amount__sum'] or 0
        }
    
    return summary
```

---

## Registration Management Systems

### 1. **InitialRegistration Model - Course Pre-Registration**

```python
class InitialRegistration(models.Model):
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    curr_id = models.ForeignKey(Curriculum, on_delete=models.CASCADE)
    semester = models.IntegerField(default=1)
    batch = models.IntegerField(default=2016)
```

#### **Field Analysis:**

#### **student_id** - Student Reference
```python
student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
```
**Student Registration:**
- Links initial course selection to student record
- Enables student-wise course planning and tracking
- Supports academic advising and course load management
- Provides access to student programme for eligibility checks

#### **curr_id** - Course Selection
```python
curr_id = models.ForeignKey(Curriculum, on_delete=models.CASCADE)
```
**Course Integration:**
- References specific curriculum course for registration
- Provides access to course credits, prerequisites, and requirements
- Enables course capacity and section management
- Supports curriculum compliance verification

#### **Academic Context:**
```python
semester = models.IntegerField(default=1)
batch = models.IntegerField(default=2016)
```
**Temporal Context:**
- **semester**: Registration semester (1-8 typically)
- **batch**: Student batch year for cohort tracking
- Enables semester-wise registration analysis
- Supports batch-based course planning and scheduling

### 2. **FinalRegistration Model - Confirmed Course Registration**

```python
class FinalRegistration(models.Model):
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    curr_id = models.ForeignKey(Curriculum, on_delete=models.CASCADE)
    semester = models.IntegerField(default=1)
    batch = models.IntegerField(default=2016)
```

#### **Registration Workflow Business Logic:**

```python
def process_initial_registration(student, course_selections, semester):
    """Process student's initial course selections"""
    
    # Validate student eligibility
    if not is_registration_open(semester):
        return False, "Registration period closed"
    
    # Clear existing initial registrations
    InitialRegistration.objects.filter(
        student_id=student,
        semester=semester,
        batch=student.batch
    ).delete()
    
    # Create new initial registrations
    registrations = []
    for course in course_selections:
        # Validate course eligibility
        if not validate_course_eligibility(student, course):
            continue
            
        registration = InitialRegistration.objects.create(
            student_id=student,
            curr_id=course,
            semester=semester,
            batch=student.batch
        )
        registrations.append(registration)
    
    return True, f"Initial registration completed for {len(registrations)} courses"

def finalize_registration(student, semester, advisor_approval=False):
    """Convert initial registrations to final registrations"""
    
    # Get initial registrations
    initial_regs = InitialRegistration.objects.filter(
        student_id=student,
        semester=semester,
        batch=student.batch
    )
    
    if not initial_regs.exists():
        return False, "No initial registrations found"
    
    # Validate credit requirements
    total_credits = sum(reg.curr_id.course_id.credit for reg in initial_regs)
    
    if not validate_credit_range(total_credits, student.programme):
        return False, f"Invalid credit load: {total_credits}"
    
    # Require advisor approval for certain cases
    if requires_advisor_approval(student, initial_regs) and not advisor_approval:
        return False, "Advisor approval required"
    
    # Create final registrations
    for initial_reg in initial_regs:
        FinalRegistration.objects.get_or_create(
            student_id=initial_reg.student_id,
            curr_id=initial_reg.curr_id,
            semester=initial_reg.semester,
            batch=initial_reg.batch
        )
    
    return True, "Registration finalized successfully"

def validate_course_eligibility(student, course):
    """Validate if student can register for course"""
    
    # Check prerequisites
    if not check_prerequisites(student, course):
        return False
    
    # Check programme compatibility
    if not course.programme_compatibility(student.programme):
        return False
    
    # Check capacity
    if is_course_full(course):
        return False
    
    # Check time conflicts
    if has_time_conflict(student, course):
        return False
    
    return True

def validate_credit_range(total_credits, programme):
    """Validate credit load within programme limits"""
    
    credit_limits = {
        'B.Tech': {'min': 18, 'max': 26},
        'M.Tech': {'min': 16, 'max': 24},
        'PhD': {'min': 8, 'max': 20}
    }
    
    limits = credit_limits.get(programme, {'min': 12, 'max': 24})
    
    return limits['min'] <= total_credits <= limits['max']
```

---

## Course Replacement System

### 1. **CourseRequested Model - Course Change Requests**

```python
class CourseRequested(models.Model):
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    curr_id = models.ForeignKey(Curriculum, on_delete=models.CASCADE)
    semester = models.IntegerField(default=1)
    batch = models.IntegerField(default=2016)
```

#### **Field Analysis:**

#### **student_id** - Requesting Student
```python
student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
```
**Student Course Modification:**
- References student requesting course change
- Enables student-specific course modification tracking
- Supports academic advising for course adjustments
- Links to student programme for eligibility verification

#### **curr_id** - Requested Course
```python
curr_id = models.ForeignKey(Curriculum, on_delete=models.CASCADE)
```
**Course Addition Request:**
- References new course student wants to add
- Provides access to course requirements and prerequisites
- Enables course capacity and scheduling verification
- Supports curriculum compliance for course changes

#### **Academic Context:**
```python
semester = models.IntegerField(default=1)
batch = models.IntegerField(default=2016)
```
**Change Context:**
- **semester**: Semester for which change is requested
- **batch**: Student batch for cohort-specific policies
- Enables temporal tracking of course modifications
- Supports semester-wise change analysis

### 2. **Course Replacement Business Logic**

```python
def request_course_replacement(student, drop_course, add_course, semester, reason):
    """Process course replacement request"""
    
    # Validate replacement timing
    if not is_course_change_allowed(semester):
        return False, "Course change period expired"
    
    # Validate current enrollment in drop course
    current_reg = FinalRegistration.objects.filter(
        student_id=student,
        curr_id=drop_course,
        semester=semester
    ).first()
    
    if not current_reg:
        return False, "Not currently enrolled in course to drop"
    
    # Validate eligibility for new course
    if not validate_course_eligibility(student, add_course):
        return False, "Not eligible for requested course"
    
    # Check if already requested
    existing_request = CourseRequested.objects.filter(
        student_id=student,
        curr_id=add_course,
        semester=semester
    ).first()
    
    if existing_request:
        return False, "Course replacement already requested"
    
    # Create course request
    request = CourseRequested.objects.create(
        student_id=student,
        curr_id=add_course,
        semester=semester,
        batch=student.batch
    )
    
    # Log the replacement request
    log_course_replacement_request(student, drop_course, add_course, reason)
    
    return True, "Course replacement request submitted"

def approve_course_replacement(request_id, approver):
    """Approve and process course replacement"""
    
    try:
        request = CourseRequested.objects.get(id=request_id)
        
        # Find the course to drop (based on business logic)
        drop_course = find_course_to_drop(request)
        
        if not drop_course:
            return False, "Unable to determine course to drop"
        
        # Remove from final registration
        FinalRegistration.objects.filter(
            student_id=request.student_id,
            curr_id=drop_course,
            semester=request.semester
        ).delete()
        
        # Add new course to final registration
        FinalRegistration.objects.create(
            student_id=request.student_id,
            curr_id=request.curr_id,
            semester=request.semester,
            batch=request.batch
        )
        
        # Remove the request
        request.delete()
        
        # Log the approval
        log_course_replacement_approval(request, approver)
        
        return True, "Course replacement approved and processed"
        
    except CourseRequested.DoesNotExist:
        return False, "Course request not found"

def get_pending_course_requests(semester=None, programme=None):
    """Get pending course replacement requests"""
    
    requests = CourseRequested.objects.all()
    
    if semester:
        requests = requests.filter(semester=semester)
    
    if programme:
        requests = requests.filter(student_id__programme=programme)
    
    return requests.select_related('student_id', 'curr_id')
```

---

## Assistantship Management System

### 1. **AssistantshipInfo Model - RA/TA Information**

```python
class AssistantshipInfo(models.Model):
    RESEARCH_ASSISTANT = 'RA'
    TEACHING_ASSISTANT = 'TA'
    ASSISTANTSHIP_TYPE_CHOICES = [
        (RESEARCH_ASSISTANT, 'Research Assistant'),
        (TEACHING_ASSISTANT, 'Teaching Assistant'),
    ]
    
    student = models.ForeignKey(Student, on_delete=models.CASCADE)
    year = models.IntegerField()
    assistantship_type = models.CharField(max_length=2, choices=ASSISTANTSHIP_TYPE_CHOICES)
    remarks = models.TextField(blank=True, null=True)
```

#### **Field Analysis:**

#### **student** - Graduate Student
```python
student = models.ForeignKey(Student, on_delete=models.CASCADE)
```
**Student Assistant:**
- References graduate student (typically M.Tech/PhD)
- Enables assistantship tracking and management
- Provides access to student programme and research area
- Supports financial aid and academic career development

#### **year** - Assistantship Year
```python
year = models.IntegerField()
```
**Academic Year:**
- Year of assistantship appointment
- Enables multi-year assistantship tracking
- Supports annual assistantship renewal and evaluation
- Used for assistantship history and progression analysis

#### **assistantship_type** - Assistant Category
```python
assistantship_type = models.CharField(max_length=2, choices=ASSISTANTSHIP_TYPE_CHOICES)
```

**Assistantship Types:**
- **RA (Research Assistant)**: Research-focused assistantship
- **TA (Teaching Assistant)**: Teaching-focused assistantship
- Enables type-specific responsibilities and evaluation
- Supports different stipend structures and requirements

#### **remarks** - Additional Information
```python
remarks = models.TextField(blank=True, null=True)
```
**Contextual Information:**
- Special conditions, performance notes, or additional details
- Flexible text field for administrative notes
- Supports assistantship evaluation and review
- Used for renewal decisions and recommendations

### 2. **Assistantship Business Logic**

```python
def create_assistantship(student, year, assistantship_type, supervisor=None, remarks=""):
    """Create new assistantship appointment"""
    
    # Validate student eligibility
    if not is_eligible_for_assistantship(student):
        return False, "Student not eligible for assistantship"
    
    # Check for existing assistantship in same year
    existing = AssistantshipInfo.objects.filter(
        student=student,
        year=year
    ).first()
    
    if existing:
        return False, f"Student already has {existing.assistantship_type} for {year}"
    
    # Create assistantship record
    assistantship = AssistantshipInfo.objects.create(
        student=student,
        year=year,
        assistantship_type=assistantship_type,
        remarks=remarks
    )
    
    # Create related assignment records if TA
    if assistantship_type == 'TA' and supervisor:
        create_ta_assignment(student, supervisor, year)
    
    # Setup stipend records
    setup_assistantship_stipend(assistantship)
    
    return True, "Assistantship created successfully"

def is_eligible_for_assistantship(student):
    """Check student eligibility for assistantship"""
    
    # Check programme eligibility
    eligible_programmes = ['M.Tech', 'M.S.', 'PhD']
    if student.programme not in eligible_programmes:
        return False
    
    # Check CGPA requirement
    if student.cgpa < 6.5:  # Minimum CGPA requirement
        return False
    
    # Check academic standing
    if has_academic_probation(student):
        return False
    
    return True

def evaluate_assistantship_performance(assistantship_id, evaluator, performance_score, comments):
    """Evaluate assistantship performance"""
    
    try:
        assistantship = AssistantshipInfo.objects.get(id=assistantship_id)
        
        # Update remarks with evaluation
        evaluation_note = f"Performance Evaluation {assistantship.year}: Score {performance_score}/10. {comments}"
        
        if assistantship.remarks:
            assistantship.remarks += f"\n{evaluation_note}"
        else:
            assistantship.remarks = evaluation_note
        
        assistantship.save()
        
        # Determine renewal recommendation
        renewal_eligible = performance_score >= 7.0
        
        return True, f"Evaluation completed. Renewal: {'Recommended' if renewal_eligible else 'Not Recommended'}"
        
    except AssistantshipInfo.DoesNotExist:
        return False, "Assistantship record not found"

def get_assistantship_statistics(year=None, assistantship_type=None):
    """Get assistantship statistics and analytics"""
    
    assistantships = AssistantshipInfo.objects.all()
    
    if year:
        assistantships = assistantships.filter(year=year)
    
    if assistantship_type:
        assistantships = assistantships.filter(assistantship_type=assistantship_type)
    
    stats = {
        'total_assistantships': assistantships.count(),
        'by_type': {},
        'by_programme': {},
        'by_year': {}
    }
    
    # Statistics by type
    for type_code, type_name in AssistantshipInfo.ASSISTANTSHIP_TYPE_CHOICES:
        count = assistantships.filter(assistantship_type=type_code).count()
        stats['by_type'][type_name] = count
    
    # Statistics by programme
    programmes = assistantships.values_list('student__programme', flat=True).distinct()
    for programme in programmes:
        count = assistantships.filter(student__programme=programme).count()
        stats['by_programme'][programme] = count
    
    return stats
```

---

This concludes Part 3 covering Due Management, Registration Systems, Course Replacement, and Assistantship Management.

Should I continue with **Part 4** covering the remaining models like Progress Examinations, Degree Verification, MTech Specialization, and other final systems?
