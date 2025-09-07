# Complete Theoretical Documentation: Counselling Cell Models - Part 1

## **COMPREHENSIVE COUNSELLING CELL SYSTEM DOCUMENTATION**

### **System Overview**
The Counselling Cell system provides comprehensive student mental health, academic guidance, and personal support services through a structured framework involving faculty counsellors, student guides, issue management, FAQ systems, and meeting coordination. This system addresses student welfare and psychological support needs across the institutional ecosystem.

---

## Constants and System Configuration

### **Position Classifications**
```python
STUDENT_POSTIONS = (
    ('student_guide', 'Student Guide'),
    ('student_coordinator', 'Student Coordinator'),
)

FACULTY_POSTIONS = (
    ('head_counsellor', 'Head Counsellor'),
    ('faculty_counsellor', 'Faculty Counsellor'),
)
```

**Counselling Team Structure:**
- **Student Guide**: Peer support and initial guidance
- **Student Coordinator**: Student team coordination and management
- **Faculty Counsellor**: Professional counselling and support
- **Head Counsellor**: Leadership and specialized intervention
- Enables hierarchical support structure with peer and professional guidance
- Supports role-based access and responsibility management

### **Issue and Meeting Management**
```python
ISSUE_STATUS = (
    ('status_unresolved', 'Unresolved'),
    ('status_resolved', 'Resolved'),    
    ('status_inprogress', 'InProgress'),
)

MEETING_STATUS = (
    ('status_accepted',"Accepted"),
    ('status_pending','Pending')
)

TIME = (
    ('10', '10 a.m.'),
    ('11', '11 a.m.'),
    ('12', '12 p.m.'),
    ('13', '1 p.m.'),
    ('14', '2 p.m.'),
    ('15', '3 p.m.'),
    ('16', '4 p.m.'),
    ('17', '5 p.m.'),
    ('18', '6 p.m.'),
    ('19', '7 p.m.'),
    ('20', '8 p.m.'),
    ('21', '9 p.m.')
)
```

**Workflow Management:**
- **Issue Status**: Complete lifecycle tracking from unresolved to resolved
- **Meeting Status**: Request approval workflow management
- **Time Slots**: Comprehensive scheduling for counselling sessions
- Enables systematic workflow management and appointment scheduling

---

## Team and Personnel Management Models

### 1. **FacultyCounsellingTeam Model - Faculty Counsellor Management**

```python
class FacultyCounsellingTeam(models.Model):
    faculty = models.ForeignKey(Faculty, on_delete=models.CASCADE)
    faculty_position = models.CharField(max_length=50,choices=CounsellingCellConstants.FACULTY_POSTIONS)

    class Meta:
        unique_together = (('faculty', 'faculty_position'))
```

#### **Field Analysis:**

#### **faculty** - Faculty Integration
```python
faculty = models.ForeignKey(Faculty, on_delete=models.CASCADE)
```
**Professional Counsellor:**
- Links to institutional faculty record
- Enables access to faculty qualifications and expertise
- Supports professional counselling service delivery
- Used for authentication and professional credential validation

#### **faculty_position** - Role Specification
```python
faculty_position = models.CharField(max_length=50,choices=CounsellingCellConstants.FACULTY_POSTIONS)
```
**Position Management:**
- **head_counsellor**: Senior counsellor with leadership responsibilities
- **faculty_counsellor**: Professional counsellor for student support
- Enables role-based access and responsibility assignment
- Supports hierarchical counselling service structure

#### **Constraint Management:**
```python
class Meta:
    unique_together = (('faculty', 'faculty_position'))
```
**Data Integrity:**
- Ensures faculty member holds only one specific position
- Prevents duplicate role assignments
- Maintains clear organizational structure
- Supports consistent role management

#### **Faculty Team Business Logic:**
```python
def assign_faculty_counsellor(faculty_id, position):
    """Assign faculty member to counselling team"""
    
    # Check if faculty already has a counselling position
    existing_assignment = FacultyCounsellingTeam.objects.filter(
        faculty=faculty_id
    ).first()
    
    if existing_assignment:
        return False, f"Faculty already assigned as {existing_assignment.faculty_position}"
    
    # Check if position is already filled
    position_filled = FacultyCounsellingTeam.objects.filter(
        faculty_position=position
    ).first()
    
    if position == 'head_counsellor' and position_filled:
        return False, "Head Counsellor position already filled"
    
    # Create assignment
    assignment = FacultyCounsellingTeam.objects.create(
        faculty=faculty_id,
        faculty_position=position
    )
    
    # Initialize counsellor profile
    initialize_counsellor_profile(assignment)
    
    return True, f"Faculty assigned as {position}"

def get_head_counsellor():
    """Get head counsellor for escalation and leadership"""
    
    try:
        head_counsellor = FacultyCounsellingTeam.objects.get(
            faculty_position='head_counsellor'
        )
        return head_counsellor
    except FacultyCounsellingTeam.DoesNotExist:
        return None

def get_available_faculty_counsellors():
    """Get all available faculty counsellors"""
    
    counsellors = FacultyCounsellingTeam.objects.filter(
        faculty_position='faculty_counsellor'
    )
    
    # Check availability based on current caseload
    available_counsellors = []
    
    for counsellor in counsellors:
        active_cases = CounsellingIssue.objects.filter(
            resolved_by=counsellor.faculty.id,
            issue_status='status_inprogress'
        ).count()
        
        # Consider counsellor available if handling less than 10 active cases
        if active_cases < 10:
            available_counsellors.append({
                'counsellor': counsellor,
                'active_cases': active_cases,
                'availability_score': 10 - active_cases
            })
    
    # Sort by availability
    available_counsellors.sort(key=lambda x: x['availability_score'], reverse=True)
    
    return available_counsellors

def reassign_counsellor_position(faculty_id, new_position):
    """Reassign faculty member to new counselling position"""
    
    try:
        assignment = FacultyCounsellingTeam.objects.get(faculty=faculty_id)
        old_position = assignment.faculty_position
        
        # Validate new position availability
        if new_position == 'head_counsellor':
            existing_head = get_head_counsellor()
            if existing_head and existing_head.faculty != faculty_id:
                return False, "Head Counsellor position already filled"
        
        # Update position
        assignment.faculty_position = new_position
        assignment.save()
        
        # Log position change
        log_position_change(faculty_id, old_position, new_position)
        
        return True, f"Position updated from {old_position} to {new_position}"
        
    except FacultyCounsellingTeam.DoesNotExist:
        return False, "Faculty not found in counselling team"
```

### 2. **StudentCounsellingTeam Model - Student Peer Support Team**

```python
class StudentCounsellingTeam(models.Model):
    student = models.ForeignKey(Student, on_delete=models.CASCADE)
    student_position = models.CharField(max_length=50,choices=CounsellingCellConstants.STUDENT_POSTIONS)

    class Meta:
        unique_together = (('student_id', 'student_position'))
```

#### **Field Analysis:**

#### **student** - Student Integration
```python
student = models.ForeignKey(Student, on_delete=models.CASCADE)
```
**Peer Support Provider:**
- Links to student academic record
- Enables peer-to-peer support and guidance
- Supports student leadership development
- Used for peer counselling and first-level support

#### **student_position** - Role Definition
```python
student_position = models.CharField(max_length=50,choices=CounsellingCellConstants.STUDENT_POSTIONS)
```
**Student Role Management:**
- **student_guide**: Provides peer guidance and initial support
- **student_coordinator**: Coordinates student counselling activities
- Enables peer support hierarchy and role clarity
- Supports student leadership and responsibility development

#### **Student Team Business Logic:**
```python
def recruit_student_guide(student_id, position):
    """Recruit student for counselling team"""
    
    # Validate student eligibility
    if not is_student_eligible_for_counselling_role(student_id):
        return False, "Student does not meet eligibility criteria"
    
    # Check if student already has a counselling position
    existing_position = StudentCounsellingTeam.objects.filter(
        student=student_id
    ).first()
    
    if existing_position:
        return False, f"Student already serves as {existing_position.student_position}"
    
    # Create assignment
    assignment = StudentCounsellingTeam.objects.create(
        student=student_id,
        student_position=position
    )
    
    # Provide training and orientation
    initiate_student_guide_training(assignment)
    
    return True, f"Student recruited as {position}"

def is_student_eligible_for_counselling_role(student_id):
    """Check student eligibility for counselling team"""
    
    try:
        student = Student.objects.get(id=student_id)
        
        # Eligibility criteria
        criteria = {
            'minimum_cgpa': 7.0,
            'minimum_semester': 3,
            'no_disciplinary_issues': True,
            'leadership_experience': False  # Optional
        }
        
        # Check CGPA (assuming CGPA field exists)
        if hasattr(student, 'cgpa') and student.cgpa < criteria['minimum_cgpa']:
            return False
        
        # Check semester (assuming current_semester field exists)
        if hasattr(student, 'current_semester') and student.current_semester < criteria['minimum_semester']:
            return False
        
        # Additional eligibility checks can be added here
        
        return True
        
    except Student.DoesNotExist:
        return False

def assign_student_guide_to_students(guide_id, assigned_students):
    """Assign student guide to specific students"""
    
    try:
        guide = StudentCounsellingTeam.objects.get(id=guide_id)
        
        # Check if guide position is student_guide
        if guide.student_position != 'student_guide':
            return False, "Only student guides can be assigned to students"
        
        # Create counselling info records
        assignments_created = 0
        
        for student_id in assigned_students:
            # Check if student already has a guide
            existing_info = StudentCounsellingInfo.objects.filter(
                student=student_id
            ).first()
            
            if not existing_info:
                StudentCounsellingInfo.objects.create(
                    student_guide=guide,
                    student_id=student_id
                )
                assignments_created += 1
        
        return True, f"{assignments_created} students assigned to guide"
        
    except StudentCounsellingTeam.DoesNotExist:
        return False, "Student guide not found"

def get_student_guide_caseload(guide_id):
    """Get current caseload for student guide"""
    
    try:
        guide = StudentCounsellingTeam.objects.get(id=guide_id)
        
        assigned_students = StudentCounsellingInfo.objects.filter(
            student_guide=guide
        )
        
        caseload_summary = {
            'total_assigned': assigned_students.count(),
            'active_issues': 0,
            'recent_contacts': 0,
            'students_needing_attention': []
        }
        
        # Analyze each assigned student
        for info in assigned_students:
            # Count active issues
            active_issues = CounsellingIssue.objects.filter(
                student=info.student,
                issue_status='status_inprogress'
            ).count()
            
            caseload_summary['active_issues'] += active_issues
            
            # Check for students needing attention
            if active_issues > 2:
                caseload_summary['students_needing_attention'].append({
                    'student': info.student,
                    'active_issues': active_issues
                })
        
        return caseload_summary
        
    except StudentCounsellingTeam.DoesNotExist:
        return None
```

### 3. **StudentCounsellingInfo Model - Student-Guide Assignment**

```python
class StudentCounsellingInfo(models.Model):
    student_guide = models.ForeignKey(StudentCounsellingTeam,on_delete=models.CASCADE)
    student = models.OneToOneField(Student,on_delete=models.CASCADE)
```

#### **Field Analysis:**

#### **student_guide** - Guide Assignment
```python
student_guide = models.ForeignKey(StudentCounsellingTeam,on_delete=models.CASCADE)
```
**Peer Support Assignment:**
- Links student to assigned peer guide
- Enables one-to-many guide-student relationship
- Supports peer counselling coordination
- Used for initial support and guidance routing

#### **student** - Student Record
```python
student = models.OneToOneField(Student,on_delete=models.CASCADE)
```
**Individual Student Profile:**
- One-to-one relationship ensures each student has one guide
- Links to complete student academic and personal record
- Enables comprehensive student support tracking
- Used for personalized counselling service delivery

#### **Student-Guide Business Logic:**
```python
def create_student_counselling_profile(student_id, guide_id=None):
    """Create counselling profile for new student"""
    
    # Check if student already has counselling info
    existing_info = StudentCounsellingInfo.objects.filter(
        student=student_id
    ).first()
    
    if existing_info:
        return False, "Student already has counselling profile"
    
    # Auto-assign guide if not specified
    if not guide_id:
        guide = find_available_student_guide()
        if not guide:
            return False, "No available student guides"
        guide_id = guide.id
    
    # Create counselling info
    counselling_info = StudentCounsellingInfo.objects.create(
        student_guide_id=guide_id,
        student_id=student_id
    )
    
    # Send welcome communication
    send_counselling_welcome_message(counselling_info)
    
    return True, f"Counselling profile created with guide assignment"

def find_available_student_guide():
    """Find student guide with lowest caseload"""
    
    guides = StudentCounsellingTeam.objects.filter(
        student_position='student_guide'
    )
    
    best_guide = None
    min_caseload = float('inf')
    
    for guide in guides:
        current_caseload = StudentCounsellingInfo.objects.filter(
            student_guide=guide
        ).count()
        
        if current_caseload < min_caseload:
            min_caseload = current_caseload
            best_guide = guide
    
    return best_guide

def transfer_student_to_new_guide(student_id, new_guide_id, reason=""):
    """Transfer student to different guide"""
    
    try:
        counselling_info = StudentCounsellingInfo.objects.get(
            student=student_id
        )
        
        old_guide = counselling_info.student_guide
        new_guide = StudentCounsellingTeam.objects.get(id=new_guide_id)
        
        # Update assignment
        counselling_info.student_guide = new_guide
        counselling_info.save()
        
        # Log transfer
        log_guide_transfer(student_id, old_guide, new_guide, reason)
        
        # Notify both guides
        notify_guide_transfer(old_guide, new_guide, student_id)
        
        return True, f"Student transferred from {old_guide.student.id} to {new_guide.student.id}"
        
    except (StudentCounsellingInfo.DoesNotExist, StudentCounsellingTeam.DoesNotExist):
        return False, "Student or guide not found"

def get_student_counselling_history(student_id):
    """Get complete counselling history for student"""
    
    try:
        counselling_info = StudentCounsellingInfo.objects.get(student=student_id)
        
        history = {
            'current_guide': counselling_info.student_guide,
            'issues_raised': CounsellingIssue.objects.filter(
                student=student_id
            ).order_by('-issue_raised_date'),
            'meeting_requests': StudentMeetingRequest.objects.filter(
                student=student_id
            ).order_by('-requested_time'),
            'support_timeline': generate_support_timeline(student_id)
        }
        
        return history
        
    except StudentCounsellingInfo.DoesNotExist:
        return None

def generate_support_timeline(student_id):
    """Generate chronological support timeline"""
    
    timeline_events = []
    
    # Get all counselling-related events
    issues = CounsellingIssue.objects.filter(student=student_id)
    meetings = StudentMeetingRequest.objects.filter(student=student_id)
    
    # Create timeline entries
    for issue in issues:
        timeline_events.append({
            'date': issue.issue_raised_date,
            'type': 'issue_raised',
            'description': f"Issue raised: {issue.issue_category.category}",
            'status': issue.issue_status
        })
    
    for meeting in meetings:
        timeline_events.append({
            'date': meeting.requested_time,
            'type': 'meeting_requested',
            'description': f"Meeting requested: {meeting.description[:50]}...",
            'status': meeting.requested_meeting_status
        })
    
    # Sort by date
    timeline_events.sort(key=lambda x: x['date'], reverse=True)
    
    return timeline_events
```

---

## Issue Management and Knowledge Base

### 1. **CounsellingIssueCategory Model - Issue Classification**

```python
class CounsellingIssueCategory(models.Model):
    category_id = models.CharField(max_length=40,unique=True)
    category = models.CharField(max_length=40)
```

#### **Field Analysis:**

#### **category_id** - Unique Identifier
```python
category_id = models.CharField(max_length=40,unique=True)
```
**Category Management:**
- Unique identifier for each issue category
- Enables systematic categorization and tracking
- Supports category-based routing and specialization
- Used for analytics and trend analysis

#### **category** - Category Description
```python
category = models.CharField(max_length=40)
```
**Issue Classification:**
- Human-readable category description
- Enables user-friendly issue classification
- Supports filtering and search functionality
- Used for reporting and analysis

#### **Category Management Business Logic:**
```python
def create_issue_category(category_id, category_name):
    """Create new counselling issue category"""
    
    # Check if category already exists
    existing_category = CounsellingIssueCategory.objects.filter(
        category_id=category_id
    ).first()
    
    if existing_category:
        return False, f"Category ID {category_id} already exists"
    
    # Create category
    category = CounsellingIssueCategory.objects.create(
        category_id=category_id,
        category=category_name
    )
    
    return True, f"Category '{category_name}' created successfully"

def get_popular_issue_categories():
    """Get most common issue categories"""
    
    from django.db.models import Count
    
    popular_categories = CounsellingIssueCategory.objects.annotate(
        issue_count=Count('counsellingissue')
    ).order_by('-issue_count')[:10]
    
    return popular_categories

def suggest_issue_category(issue_description):
    """Suggest appropriate category based on issue description"""
    
    # Simple keyword-based category suggestion
    category_keywords = {
        'academic': ['study', 'exam', 'grade', 'course', 'assignment'],
        'personal': ['family', 'relationship', 'personal', 'emotional'],
        'financial': ['money', 'fee', 'scholarship', 'financial'],
        'health': ['health', 'medical', 'illness', 'doctor'],
        'career': ['career', 'job', 'internship', 'placement']
    }
    
    issue_lower = issue_description.lower()
    suggestions = []
    
    for category_id, keywords in category_keywords.items():
        match_count = sum(1 for keyword in keywords if keyword in issue_lower)
        if match_count > 0:
            suggestions.append({
                'category_id': category_id,
                'match_score': match_count,
                'confidence': match_count / len(keywords)
            })
    
    # Sort by match score
    suggestions.sort(key=lambda x: x['match_score'], reverse=True)
    
    return suggestions[:3]  # Return top 3 suggestions
```

### 2. **CounsellingIssue Model - Core Issue Management**

```python
class CounsellingIssue(models.Model):
    issue_raised_date = models.DateTimeField(default=datetime.now)
    student = models.ForeignKey(Student, on_delete=models.CASCADE)
    issue_category = models.ForeignKey(CounsellingIssueCategory,on_delete=models.CASCADE)
    issue = models.TextField(max_length=500,)
    issue_status = models.CharField(max_length=20,choices=CounsellingCellConstants.ISSUE_STATUS,default="status_unresolved")
    response_remark = models.TextField(max_length=500,null=True)
    resolved_by = models.ForeignKey(ExtraInfo, on_delete=models.CASCADE,null=True)
```

#### **Field Analysis:**

#### **Temporal and Student Information:**
```python
issue_raised_date = models.DateTimeField(default=datetime.now)
student = models.ForeignKey(Student, on_delete=models.CASCADE)
```
**Issue Origin Tracking:**
- **issue_raised_date**: Automatic timestamp for issue submission
- **student**: Student who raised the issue
- Enables temporal analysis and student-specific tracking
- Supports response time monitoring and SLA management

#### **Issue Classification and Content:**
```python
issue_category = models.ForeignKey(CounsellingIssueCategory,on_delete=models.CASCADE)
issue = models.TextField(max_length=500,)
```
**Issue Specification:**
- **issue_category**: Structured categorization for routing
- **issue**: Detailed issue description and context
- Enables category-based routing and specialized handling
- Supports issue analysis and trend identification

#### **Resolution Tracking:**
```python
issue_status = models.CharField(max_length=20,choices=CounsellingCellConstants.ISSUE_STATUS,default="status_unresolved")
response_remark = models.TextField(max_length=500,null=True)
resolved_by = models.ForeignKey(ExtraInfo, on_delete=models.CASCADE,null=True)
```
**Resolution Management:**
- **issue_status**: Workflow status tracking (unresolved/inprogress/resolved)
- **response_remark**: Counsellor response and action taken
- **resolved_by**: Staff member who handled the resolution
- Enables complete resolution tracking and accountability

#### **Issue Management Business Logic:**
```python
def submit_counselling_issue(student_id, category_id, issue_description):
    """Submit new counselling issue"""
    
    # Validate issue description
    if len(issue_description.strip()) < 20:
        return False, "Please provide detailed description (minimum 20 characters)"
    
    # Get or suggest category
    try:
        category = CounsellingIssueCategory.objects.get(category_id=category_id)
    except CounsellingIssueCategory.DoesNotExist:
        # Auto-suggest category
        suggestions = suggest_issue_category(issue_description)
        if suggestions:
            category = CounsellingIssueCategory.objects.get(
                category_id=suggestions[0]['category_id']
            )
        else:
            return False, "Please select a valid issue category"
    
    # Create issue
    issue = CounsellingIssue.objects.create(
        student_id=student_id,
        issue_category=category,
        issue=issue_description
    )
    
    # Auto-route to appropriate counsellor
    auto_assign_counsellor(issue)
    
    # Notify student guide
    notify_student_guide_of_issue(issue)
    
    # Check for urgent issues
    if is_urgent_issue(issue_description):
        escalate_urgent_issue(issue)
    
    return True, f"Issue submitted successfully. Reference: ISSUE{issue.id:06d}"

def auto_assign_counsellor(issue):
    """Auto-assign issue to appropriate counsellor"""
    
    # Get available counsellors
    available_counsellors = get_available_faculty_counsellors()
    
    if not available_counsellors:
        # No counsellors available, assign to head counsellor
        head_counsellor = get_head_counsellor()
        if head_counsellor:
            issue.resolved_by = head_counsellor.faculty.id
            issue.issue_status = 'status_inprogress'
            issue.save()
    else:
        # Assign to counsellor with lowest caseload
        best_counsellor = available_counsellors[0]['counsellor']
        issue.resolved_by = best_counsellor.faculty.id
        issue.issue_status = 'status_inprogress'
        issue.save()
        
        # Notify assigned counsellor
        notify_counsellor_assignment(best_counsellor, issue)

def update_issue_status(issue_id, new_status, response_remark="", counsellor_id=None):
    """Update issue status and provide response"""
    
    try:
        issue = CounsellingIssue.objects.get(id=issue_id)
        
        # Validate status transition
        if not is_valid_status_transition(issue.issue_status, new_status):
            return False, f"Invalid status transition from {issue.issue_status} to {new_status}"
        
        # Update issue
        old_status = issue.issue_status
        issue.issue_status = new_status
        issue.response_remark = response_remark
        
        if counsellor_id:
            issue.resolved_by_id = counsellor_id
        
        issue.save()
        
        # Trigger post-status actions
        handle_issue_status_change(issue, old_status, new_status)
        
        # Notify student
        notify_student_issue_update(issue)
        
        return True, f"Issue status updated to {new_status}"
        
    except CounsellingIssue.DoesNotExist:
        return False, "Issue not found"

def is_urgent_issue(issue_description):
    """Determine if issue requires urgent attention"""
    
    urgent_keywords = [
        'suicide', 'self-harm', 'depression', 'emergency',
        'urgent', 'critical', 'immediate', 'crisis'
    ]
    
    issue_lower = issue_description.lower()
    
    for keyword in urgent_keywords:
        if keyword in issue_lower:
            return True
    
    return False

def escalate_urgent_issue(issue):
    """Escalate urgent issue to head counsellor"""
    
    head_counsellor = get_head_counsellor()
    if head_counsellor:
        issue.resolved_by = head_counsellor.faculty.id
        issue.issue_status = 'status_inprogress'
        issue.save()
        
        # Send urgent notification
        send_urgent_issue_notification(head_counsellor, issue)
        
        # Log escalation
        log_urgent_escalation(issue)

def generate_issue_analytics(start_date, end_date):
    """Generate comprehensive issue analytics"""
    
    issues = CounsellingIssue.objects.filter(
        issue_raised_date__range=[start_date, end_date]
    )
    
    analytics = {
        'total_issues': issues.count(),
        'resolved_issues': issues.filter(issue_status='status_resolved').count(),
        'pending_issues': issues.filter(issue_status='status_unresolved').count(),
        'in_progress_issues': issues.filter(issue_status='status_inprogress').count(),
        'category_distribution': {},
        'counsellor_performance': {},
        'resolution_metrics': {}
    }
    
    # Category distribution
    from django.db.models import Count
    category_stats = issues.values('issue_category__category').annotate(
        count=Count('id')
    ).order_by('-count')
    
    for stat in category_stats:
        analytics['category_distribution'][stat['issue_category__category']] = stat['count']
    
    # Counsellor performance
    counsellor_stats = issues.filter(
        resolved_by__isnull=False
    ).values('resolved_by__user__first_name', 'resolved_by__user__last_name').annotate(
        handled_count=Count('id'),
        resolved_count=Count('id', filter=models.Q(issue_status='status_resolved'))
    )
    
    for stat in counsellor_stats:
        counsellor_name = f"{stat['resolved_by__user__first_name']} {stat['resolved_by__user__last_name']}"
        analytics['counsellor_performance'][counsellor_name] = {
            'handled': stat['handled_count'],
            'resolved': stat['resolved_count'],
            'resolution_rate': stat['resolved_count'] / stat['handled_count'] * 100 if stat['handled_count'] > 0 else 0
        }
    
    return analytics

def get_student_issue_summary(student_id):
    """Get issue summary for specific student"""
    
    issues = CounsellingIssue.objects.filter(student=student_id)
    
    summary = {
        'total_issues': issues.count(),
        'resolved_issues': issues.filter(issue_status='status_resolved').count(),
        'pending_issues': issues.filter(issue_status='status_unresolved').count(),
        'recent_issues': issues.order_by('-issue_raised_date')[:5],
        'frequent_categories': issues.values('issue_category__category').annotate(
            count=Count('id')
        ).order_by('-count')[:3]
    }
    
    return summary
```

### 3. **CounsellingFAQ Model - Knowledge Base Management**

```python
class CounsellingFAQ(models.Model):
    counselling_question = models.TextField(max_length=1000)
    counselling_answer = models.TextField(max_length=5000)
    counselling_category = models.ForeignKey(CounsellingIssueCategory,on_delete=models.CASCADE)
```

#### **Field Analysis:**

#### **Knowledge Content:**
```python
counselling_question = models.TextField(max_length=1000)
counselling_answer = models.TextField(max_length=5000)
```
**FAQ Content Management:**
- **counselling_question**: Comprehensive question formulation
- **counselling_answer**: Detailed response and guidance
- Enables self-service support and knowledge sharing
- Supports consistency in counselling responses

#### **Category Integration:**
```python
counselling_category = models.ForeignKey(CounsellingIssueCategory,on_delete=models.CASCADE)
```
**Knowledge Organization:**
- Links FAQ to specific issue category
- Enables category-based knowledge retrieval
- Supports organized knowledge base structure
- Used for targeted FAQ suggestions

#### **FAQ Management Business Logic:**
```python
def create_faq_entry(question, answer, category_id):
    """Create new FAQ entry"""
    
    # Validate content
    if len(question.strip()) < 10:
        return False, "Question must be at least 10 characters"
    
    if len(answer.strip()) < 50:
        return False, "Answer must be at least 50 characters"
    
    # Get category
    try:
        category = CounsellingIssueCategory.objects.get(category_id=category_id)
    except CounsellingIssueCategory.DoesNotExist:
        return False, "Invalid category"
    
    # Check for duplicate questions
    existing_faq = CounsellingFAQ.objects.filter(
        counselling_question__icontains=question[:50]
    ).first()
    
    if existing_faq:
        return False, "Similar question already exists in FAQ"
    
    # Create FAQ
    faq = CounsellingFAQ.objects.create(
        counselling_question=question,
        counselling_answer=answer,
        counselling_category=category
    )
    
    return True, f"FAQ entry created successfully"

def search_relevant_faqs(query, category_id=None):
    """Search for relevant FAQ entries"""
    
    faqs = CounsellingFAQ.objects.all()
    
    if category_id:
        faqs = faqs.filter(counselling_category__category_id=category_id)
    
    # Simple text search in questions and answers
    relevant_faqs = faqs.filter(
        models.Q(counselling_question__icontains=query) |
        models.Q(counselling_answer__icontains=query)
    )
    
    # Rank by relevance (simple keyword counting)
    faq_scores = []
    query_words = query.lower().split()
    
    for faq in relevant_faqs:
        content = f"{faq.counselling_question} {faq.counselling_answer}".lower()
        score = sum(content.count(word) for word in query_words)
        
        faq_scores.append({
            'faq': faq,
            'relevance_score': score
        })
    
    # Sort by relevance
    faq_scores.sort(key=lambda x: x['relevance_score'], reverse=True)
    
    return [item['faq'] for item in faq_scores[:10]]

def suggest_faqs_for_issue(issue_description, category_id):
    """Suggest relevant FAQs for reported issue"""
    
    # Search FAQs in the same category
    category_faqs = CounsellingFAQ.objects.filter(
        counselling_category__category_id=category_id
    )
    
    # Find relevant FAQs based on issue description
    relevant_faqs = []
    
    for faq in category_faqs:
        # Simple relevance scoring
        question_words = faq.counselling_question.lower().split()
        issue_words = issue_description.lower().split()
        
        common_words = set(question_words) & set(issue_words)
        relevance_score = len(common_words) / max(len(question_words), 1)
        
        if relevance_score > 0.2:  # 20% relevance threshold
            relevant_faqs.append({
                'faq': faq,
                'relevance_score': relevance_score
            })
    
    # Sort by relevance
    relevant_faqs.sort(key=lambda x: x['relevance_score'], reverse=True)
    
    return [item['faq'] for item in relevant_faqs[:5]]

def update_faq_from_resolved_issues():
    """Generate FAQ entries from frequently resolved issues"""
    
    # Get recently resolved issues
    resolved_issues = CounsellingIssue.objects.filter(
        issue_status='status_resolved',
        response_remark__isnull=False
    ).exclude(response_remark='')
    
    # Group by category and similar issues
    category_issues = {}
    
    for issue in resolved_issues:
        category_id = issue.issue_category.category_id
        
        if category_id not in category_issues:
            category_issues[category_id] = []
        
        category_issues[category_id].append(issue)
    
    # Generate FAQ suggestions
    faq_suggestions = []
    
    for category_id, issues in category_issues.items():
        if len(issues) >= 5:  # At least 5 similar issues
            # Extract common patterns
            common_question = extract_common_question_pattern(issues)
            consolidated_answer = consolidate_responses(issues)
            
            faq_suggestions.append({
                'question': common_question,
                'answer': consolidated_answer,
                'category_id': category_id,
                'source_issues': len(issues)
            })
    
    return faq_suggestions

def get_faq_analytics():
    """Generate FAQ usage and effectiveness analytics"""
    
    analytics = {
        'total_faqs': CounsellingFAQ.objects.count(),
        'category_distribution': {},
        'most_searched_faqs': [],  # Would need search tracking
        'gaps_analysis': []  # Categories with few FAQs but many issues
    }
    
    # Category distribution
    from django.db.models import Count
    category_stats = CounsellingFAQ.objects.values(
        'counselling_category__category'
    ).annotate(faq_count=Count('id'))
    
    for stat in category_stats:
        analytics['category_distribution'][stat['counselling_category__category']] = stat['faq_count']
    
    # Gap analysis - categories with many issues but few FAQs
    issue_stats = CounsellingIssue.objects.values(
        'issue_category__category'
    ).annotate(issue_count=Count('id'))
    
    for issue_stat in issue_stats:
        category = issue_stat['issue_category__category']
        issue_count = issue_stat['issue_count']
        faq_count = analytics['category_distribution'].get(category, 0)
        
        if issue_count > 10 and faq_count < 3:
            analytics['gaps_analysis'].append({
                'category': category,
                'issues': issue_count,
                'faqs': faq_count,
                'gap_score': issue_count / max(faq_count, 1)
            })
    
    return analytics
```

---

This completes Part 1 of the Counselling Cell documentation covering team management, issue management, and knowledge base models.