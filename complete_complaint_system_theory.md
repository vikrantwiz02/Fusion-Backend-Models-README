# Complete Theoretical Documentation: Complaint System Models

## **COMPREHENSIVE COMPLAINT SYSTEM DOCUMENTATION**

### **System Overview**
The Complaint System manages institutional maintenance and service requests through a structured workflow involving students, caretakers, section incharges, workers, and supervisors. This system handles everything from complaint submission to resolution tracking across various campus areas and service types.

---

## Constants and System Configuration

### **Area Classifications**
```python
AREA = (
    ('hall-1', 'hall-1'),
    ('hall-3', 'hall-3'),
    ('hall-4', 'hall-4'),
    ('library', 'CC1'),
    ('computer center', 'CC2'),
    ('core_lab', 'core_lab'),
    ('LHTC', 'LHTC'),
    ('NR2', 'NR2'),
    ('NR3', 'NR3'),
    ('Admin building', 'Admin building'),
    ('Rewa_Residency', 'Rewa_Residency'),
    ('Maa Saraswati Hostel', 'Maa Saraswati Hostel'),
    ('Nagarjun Hostel', 'Nagarjun Hostel'),
    ('Panini Hostel', 'Panini Hostel'),
)
```

**Campus Area Management:**
- **Residential Areas**: hall-1, hall-3, hall-4, hostels
- **Academic Areas**: library, computer center, core_lab, LHTC
- **Administrative Areas**: Admin building, NR2, NR3
- **Guest Facilities**: Rewa_Residency
- **Student Hostels**: Maa Saraswati, Nagarjun, Panini
- Enables location-specific complaint routing and management
- Supports area-wise maintenance planning and resource allocation

### **Complaint Type Classifications**
```python
COMPLAINT_TYPE = (
    ('Electricity', 'Electricity'),
    ('carpenter', 'carpenter'),
    ('plumber', 'plumber'),
    ('garbage', 'garbage'),
    ('dustbin', 'dustbin'),
    ('internet', 'internet'),
    ('other', 'other'),
)
```

**Service Category Management:**
- **Electrical Services**: Power, lighting, electrical appliances
- **Carpentry Services**: Furniture repair, woodwork maintenance
- **Plumbing Services**: Water supply, drainage, sanitation
- **Sanitation Services**: Garbage collection, dustbin maintenance
- **IT Services**: Internet connectivity, network issues
- **General Services**: Other maintenance requirements
- Enables specialized work assignment and expertise matching

---

## Human Resource Management Models

### 1. **Caretaker Model - Area-wise Maintenance Management**

```python
class Caretaker(models.Model):
    staff_id = models.ForeignKey(ExtraInfo, on_delete=models.CASCADE)
    area = models.CharField(choices=Constants.AREA, max_length=20, default='hall-3')
    rating = models.IntegerField(default=0)
    myfeedback = models.CharField(max_length=400, default='this is my feedback')
```

#### **Field Analysis:**

#### **staff_id** - Staff Integration
```python
staff_id = models.ForeignKey(ExtraInfo, on_delete=models.CASCADE)
```
**Staff Management:**
- Links to institutional staff record via ExtraInfo
- Enables access to staff personal and professional details
- Supports staff authentication and authorization
- Used for communication and accountability tracking

#### **area** - Area Responsibility
```python
area = models.CharField(choices=Constants.AREA, max_length=20, default='hall-3')
```
**Area Assignment:**
- Specific campus area under caretaker responsibility
- Enables geographical work distribution and management
- Supports area-specific expertise development
- Used for complaint routing and escalation

#### **rating** - Performance Tracking
```python
rating = models.IntegerField(default=0)
```
**Performance Management:**
- Numerical performance rating (typically 1-5 or 1-10 scale)
- Aggregate rating based on complaint resolution feedback
- Enables performance tracking and improvement identification
- Used for staff evaluation and recognition programs

#### **myfeedback** - Self-Assessment
```python
myfeedback = models.CharField(max_length=400, default='this is my feedback')
```
**Self-Evaluation:**
- Caretaker's self-assessment and feedback
- Enables self-reflection and improvement planning
- Supports two-way communication and development
- Used for performance discussions and goal setting

#### **Caretaker Business Logic:**
```python
def assign_caretaker_to_area(staff_id, area):
    """Assign caretaker to specific area"""
    
    # Check if area already has caretaker
    existing_caretaker = Caretaker.objects.filter(area=area).first()
    if existing_caretaker:
        return False, f"Area {area} already has caretaker: {existing_caretaker.staff_id}"
    
    # Create caretaker assignment
    caretaker = Caretaker.objects.create(
        staff_id=staff_id,
        area=area
    )
    
    # Initialize performance metrics
    initialize_caretaker_metrics(caretaker)
    
    return True, f"Caretaker assigned to {area}"

def update_caretaker_rating(caretaker_id, new_rating):
    """Update caretaker performance rating"""
    
    try:
        caretaker = Caretaker.objects.get(id=caretaker_id)
        old_rating = caretaker.rating
        
        caretaker.rating = new_rating
        caretaker.save()
        
        # Log rating change
        log_performance_change(caretaker, old_rating, new_rating)
        
        # Trigger performance actions if needed
        if new_rating >= 8:
            trigger_recognition_process(caretaker)
        elif new_rating <= 3:
            trigger_improvement_plan(caretaker)
        
        return True, f"Rating updated from {old_rating} to {new_rating}"
        
    except Caretaker.DoesNotExist:
        return False, "Caretaker not found"

def get_area_caretaker(area):
    """Get caretaker responsible for specific area"""
    
    try:
        caretaker = Caretaker.objects.get(area=area)
        return caretaker
    except Caretaker.DoesNotExist:
        return None

def calculate_area_performance_metrics(area):
    """Calculate performance metrics for area"""
    
    caretaker = get_area_caretaker(area)
    if not caretaker:
        return None
    
    # Get area complaints
    area_complaints = StudentComplain.objects.filter(location=area)
    
    metrics = {
        'total_complaints': area_complaints.count(),
        'resolved_complaints': area_complaints.filter(status=2).count(),
        'pending_complaints': area_complaints.filter(status=0).count(),
        'average_resolution_time': calculate_average_resolution_time(area_complaints),
        'caretaker_rating': caretaker.rating,
        'satisfaction_score': calculate_satisfaction_score(area_complaints)
    }
    
    # Update caretaker rating based on metrics
    new_rating = calculate_performance_rating(metrics)
    if abs(new_rating - caretaker.rating) > 1:
        update_caretaker_rating(caretaker.id, new_rating)
    
    return metrics
```

### 2. **SectionIncharge Model - Service Type Management**

```python
class SectionIncharge(models.Model):
    staff_id = models.ForeignKey(ExtraInfo, on_delete=models.CASCADE)
    work_type = models.CharField(choices=Constants.COMPLAINT_TYPE,
                                   max_length=20, default='Electricity')
```

#### **Field Analysis:**

#### **staff_id** - Staff Integration
```python
staff_id = models.ForeignKey(ExtraInfo, on_delete=models.CASCADE)
```
**Leadership Assignment:**
- Links to institutional staff record for section leadership
- Enables access to staff qualifications and experience
- Supports authority and responsibility tracking
- Used for escalation and decision-making processes

#### **work_type** - Specialization Area
```python
work_type = models.CharField(choices=Constants.COMPLAINT_TYPE,
                            max_length=20, default='Electricity')
```
**Technical Specialization:**
- Specific service type under section incharge supervision
- Enables expertise-based complaint routing and management
- Supports specialized team leadership and coordination
- Used for technical decision-making and quality assurance

#### **SectionIncharge Business Logic:**
```python
def assign_section_incharge(staff_id, work_type):
    """Assign section incharge for specific work type"""
    
    # Check if work type already has incharge
    existing_incharge = SectionIncharge.objects.filter(work_type=work_type).first()
    if existing_incharge:
        return False, f"Work type {work_type} already has incharge: {existing_incharge.staff_id}"
    
    # Create section incharge assignment
    incharge = SectionIncharge.objects.create(
        staff_id=staff_id,
        work_type=work_type
    )
    
    # Initialize section metrics
    initialize_section_metrics(incharge)
    
    return True, f"Section incharge assigned for {work_type}"

def get_work_type_incharge(work_type):
    """Get section incharge for specific work type"""
    
    try:
        incharge = SectionIncharge.objects.get(work_type=work_type)
        return incharge
    except SectionIncharge.DoesNotExist:
        return None

def escalate_complaint_to_incharge(complaint_id):
    """Escalate complaint to appropriate section incharge"""
    
    try:
        complaint = StudentComplain.objects.get(id=complaint_id)
        incharge = get_work_type_incharge(complaint.complaint_type)
        
        if not incharge:
            return False, f"No section incharge found for {complaint.complaint_type}"
        
        # Update complaint with escalation
        complaint.flag = 1  # Set escalation flag
        complaint.save()
        
        # Notify section incharge
        notify_section_incharge(incharge, complaint)
        
        # Log escalation
        log_complaint_escalation(complaint, incharge)
        
        return True, f"Complaint escalated to {incharge.staff_id}"
        
    except StudentComplain.DoesNotExist:
        return False, "Complaint not found"

def generate_section_performance_report(work_type, start_date, end_date):
    """Generate performance report for section"""
    
    incharge = get_work_type_incharge(work_type)
    if not incharge:
        return None
    
    # Get section complaints
    section_complaints = StudentComplain.objects.filter(
        complaint_type=work_type,
        complaint_date__range=[start_date, end_date]
    )
    
    report = {
        'section_incharge': incharge.staff_id.user.get_full_name(),
        'work_type': work_type,
        'period': f"{start_date} to {end_date}",
        'total_complaints': section_complaints.count(),
        'resolved_complaints': section_complaints.filter(status=2).count(),
        'pending_complaints': section_complaints.filter(status=0).count(),
        'escalated_complaints': section_complaints.filter(flag=1).count(),
        'average_resolution_time': calculate_average_resolution_time(section_complaints),
        'worker_performance': get_section_worker_performance(work_type),
        'satisfaction_metrics': calculate_section_satisfaction(section_complaints)
    }
    
    return report
```

### 3. **Workers Model - Field Personnel Management**

```python
class Workers(models.Model):
    secincharge_id = models.ForeignKey(SectionIncharge, on_delete=models.CASCADE, null=True)
    name = models.CharField(max_length=50)
    age = models.CharField(max_length=10)
    phone = models.BigIntegerField(blank=True)
    worker_type = models.CharField(choices=Constants.COMPLAINT_TYPE,
                                   max_length=20, default='internet')
```

#### **Field Analysis:**

#### **secincharge_id** - Supervision Hierarchy
```python
secincharge_id = models.ForeignKey(SectionIncharge, on_delete=models.CASCADE, null=True)
```
**Management Structure:**
- Links worker to appropriate section incharge
- Enables hierarchical management and supervision
- Supports work assignment and performance tracking
- Used for escalation and coordination workflows

#### **Personal Information:**
```python
name = models.CharField(max_length=50)
age = models.CharField(max_length=10)
phone = models.BigIntegerField(blank=True)
```
**Worker Profile:**
- **name**: Worker's full name for identification
- **age**: Age information for HR and safety purposes
- **phone**: Contact number for direct communication
- Enables worker identification and communication
- Supports workforce management and contact tracking

#### **worker_type** - Skill Specialization
```python
worker_type = models.CharField(choices=Constants.COMPLAINT_TYPE,
                              max_length=20, default='internet')
```
**Technical Expertise:**
- Worker's specialized skill area and service type
- Enables skill-based work assignment
- Supports expertise matching with complaint requirements
- Used for quality assurance and efficiency optimization

#### **Workers Business Logic:**
```python
def register_worker(name, age, phone, worker_type, section_incharge=None):
    """Register new worker in the system"""
    
    # Get appropriate section incharge if not provided
    if not section_incharge:
        section_incharge = get_work_type_incharge(worker_type)
        if not section_incharge:
            return False, f"No section incharge found for {worker_type}"
    
    # Create worker record
    worker = Workers.objects.create(
        secincharge_id=section_incharge,
        name=name,
        age=age,
        phone=phone,
        worker_type=worker_type
    )
    
    # Initialize worker metrics
    initialize_worker_metrics(worker)
    
    return True, f"Worker {name} registered for {worker_type}"

def assign_worker_to_complaint(complaint_id, worker_id=None):
    """Assign worker to complaint (auto or manual assignment)"""
    
    try:
        complaint = StudentComplain.objects.get(id=complaint_id)
        
        if worker_id:
            # Manual assignment
            worker = Workers.objects.get(id=worker_id)
            
            # Validate worker type matches complaint type
            if worker.worker_type != complaint.complaint_type:
                return False, f"Worker specializes in {worker.worker_type}, complaint is {complaint.complaint_type}"
        else:
            # Auto assignment - find available worker
            worker = find_available_worker(complaint.complaint_type, complaint.location)
            if not worker:
                return False, f"No available worker found for {complaint.complaint_type}"
        
        # Assign worker to complaint
        complaint.worker_id = worker
        complaint.status = 1  # In progress
        complaint.save()
        
        # Notify worker
        notify_worker_assignment(worker, complaint)
        
        # Log assignment
        log_worker_assignment(worker, complaint)
        
        return True, f"Worker {worker.name} assigned to complaint"
        
    except (StudentComplain.DoesNotExist, Workers.DoesNotExist):
        return False, "Complaint or worker not found"

def find_available_worker(work_type, location):
    """Find available worker for specific work type and location"""
    
    # Get workers of the required type
    available_workers = Workers.objects.filter(worker_type=work_type)
    
    # Check worker availability (not assigned to active complaints)
    for worker in available_workers:
        active_assignments = StudentComplain.objects.filter(
            worker_id=worker,
            status=1  # In progress
        ).count()
        
        # If worker has less than 3 active assignments, consider available
        if active_assignments < 3:
            return worker
    
    return None

def get_worker_performance_metrics(worker_id):
    """Calculate worker performance metrics"""
    
    try:
        worker = Workers.objects.get(id=worker_id)
        
        # Get worker's complaint history
        worker_complaints = StudentComplain.objects.filter(worker_id=worker)
        
        metrics = {
            'total_assignments': worker_complaints.count(),
            'completed_assignments': worker_complaints.filter(status=2).count(),
            'pending_assignments': worker_complaints.filter(status=1).count(),
            'average_resolution_time': calculate_worker_resolution_time(worker),
            'satisfaction_rating': calculate_worker_satisfaction(worker),
            'efficiency_score': calculate_worker_efficiency(worker)
        }
        
        return metrics
        
    except Workers.DoesNotExist:
        return None

def update_worker_status(worker_id, new_phone=None, new_section=None):
    """Update worker information and status"""
    
    try:
        worker = Workers.objects.get(id=worker_id)
        
        if new_phone:
            worker.phone = new_phone
        
        if new_section:
            new_incharge = get_work_type_incharge(new_section)
            if new_incharge:
                worker.secincharge_id = new_incharge
                worker.worker_type = new_section
        
        worker.save()
        
        return True, f"Worker {worker.name} updated successfully"
        
    except Workers.DoesNotExist:
        return False, "Worker not found"
```

---

## Complaint Management and Workflow

### 1. **StudentComplain Model - Core Complaint System**

```python
class StudentComplain(models.Model):
    complainer = models.ForeignKey(ExtraInfo, on_delete=models.CASCADE)
    complaint_date = models.DateTimeField(default=timezone.now)
    complaint_finish = models.DateField(blank=True, null=True)
    complaint_type = models.CharField(choices=Constants.COMPLAINT_TYPE,
                                      max_length=20, default='internet')
    location = models.CharField(max_length=20, choices=Constants.AREA)
    specific_location = models.CharField(max_length=50, blank=True)
    details = models.CharField(max_length=100)
    status = models.IntegerField(default='0')
    remarks = models.CharField(max_length=300, default="Pending")
    flag = models.IntegerField(default='0')
    reason = models.CharField(max_length=100, blank=True, default="None")
    feedback = models.CharField(max_length=500, blank=True)
    worker_id = models.ForeignKey(Workers, blank=True, null=True,on_delete=models.CASCADE)
    upload_complaint = models.FileField(blank=True)
    comment = models.CharField(max_length=100,  default="None")
```

#### **Field Analysis:**

#### **complainer** - Complaint Origin
```python
complainer = models.ForeignKey(ExtraInfo, on_delete=models.CASCADE)
```
**User Integration:**
- Links complaint to student/staff record via ExtraInfo
- Enables complaint tracking and user history
- Supports authentication and authorization
- Used for communication and follow-up

#### **Temporal Tracking:**
```python
complaint_date = models.DateTimeField(default=timezone.now)
complaint_finish = models.DateField(blank=True, null=True)
```
**Timeline Management:**
- **complaint_date**: Automatic timestamp of complaint submission
- **complaint_finish**: Completion date (null until resolved)
- Enables resolution time tracking and SLA monitoring
- Supports performance analytics and trend analysis

#### **Complaint Classification:**
```python
complaint_type = models.CharField(choices=Constants.COMPLAINT_TYPE,
                                  max_length=20, default='internet')
location = models.CharField(max_length=20, choices=Constants.AREA)
specific_location = models.CharField(max_length=50, blank=True)
details = models.CharField(max_length=100)
```
**Problem Specification:**
- **complaint_type**: Service category for proper routing
- **location**: Campus area for geographical routing
- **specific_location**: Detailed location within area
- **details**: Problem description and context
- Enables accurate problem identification and resource allocation

#### **Status and Workflow Management:**
```python
status = models.IntegerField(default='0')
remarks = models.CharField(max_length=300, default="Pending")
flag = models.IntegerField(default='0')
reason = models.CharField(max_length=100, blank=True, default="None")
```
**Workflow Control:**
- **status**: Complaint status (0=Pending, 1=In Progress, 2=Resolved)
- **remarks**: Administrative comments and updates
- **flag**: Escalation flag (0=Normal, 1=Escalated)
- **reason**: Detailed reasoning for status changes
- Enables systematic workflow management and audit trail

#### **Resolution and Feedback:**
```python
feedback = models.CharField(max_length=500, blank=True)
worker_id = models.ForeignKey(Workers, blank=True, null=True,on_delete=models.CASCADE)
upload_complaint = models.FileField(blank=True)
comment = models.CharField(max_length=100,  default="None")
```
**Resolution Tracking:**
- **feedback**: User feedback on resolution quality
- **worker_id**: Assigned worker for resolution
- **upload_complaint**: Supporting documents/images
- **comment**: Additional comments and notes
- Enables complete resolution documentation and quality assessment

#### **Complaint Workflow Business Logic:**
```python
def submit_complaint(complainer, complaint_type, location, specific_location, details, upload_file=None):
    """Submit new complaint to the system"""
    
    # Validate complaint details
    if not details or len(details.strip()) < 10:
        return False, "Please provide detailed description (minimum 10 characters)"
    
    # Check for duplicate complaints
    recent_complaints = StudentComplain.objects.filter(
        complainer=complainer,
        complaint_type=complaint_type,
        location=location,
        complaint_date__gte=timezone.now() - timezone.timedelta(hours=24),
        status__in=[0, 1]  # Pending or in progress
    )
    
    if recent_complaints.exists():
        return False, "Similar complaint already exists and is being processed"
    
    # Create complaint
    complaint = StudentComplain.objects.create(
        complainer=complainer,
        complaint_type=complaint_type,
        location=location,
        specific_location=specific_location,
        details=details,
        upload_complaint=upload_file
    )
    
    # Auto-assign based on type and location
    auto_assign_complaint(complaint)
    
    # Notify relevant personnel
    notify_complaint_submission(complaint)
    
    # Generate complaint ID for tracking
    complaint_id = f"COMP{complaint.id:06d}"
    
    return True, f"Complaint submitted successfully. ID: {complaint_id}"

def auto_assign_complaint(complaint):
    """Automatically assign complaint to appropriate personnel"""
    
    # Get area caretaker
    caretaker = get_area_caretaker(complaint.location)
    
    # Get section incharge
    section_incharge = get_work_type_incharge(complaint.complaint_type)
    
    # Assign worker if available
    worker = find_available_worker(complaint.complaint_type, complaint.location)
    if worker:
        complaint.worker_id = worker
        complaint.status = 1  # In progress
        complaint.remarks = f"Assigned to {worker.name}"
    
    complaint.save()
    
    # Notify assignments
    if caretaker:
        notify_caretaker_assignment(caretaker, complaint)
    if section_incharge:
        notify_section_incharge_assignment(section_incharge, complaint)
    if worker:
        notify_worker_assignment(worker, complaint)

def update_complaint_status(complaint_id, new_status, remarks="", reason=""):
    """Update complaint status with proper workflow"""
    
    try:
        complaint = StudentComplain.objects.get(id=complaint_id)
        old_status = complaint.status
        
        # Validate status transition
        if not is_valid_status_transition(old_status, new_status):
            return False, f"Invalid status transition from {old_status} to {new_status}"
        
        # Update complaint
        complaint.status = new_status
        complaint.remarks = remarks or get_default_remarks(new_status)
        complaint.reason = reason
        
        # Set completion date if resolved
        if new_status == 2:  # Resolved
            complaint.complaint_finish = timezone.now().date()
        
        complaint.save()
        
        # Trigger post-status actions
        handle_status_change(complaint, old_status, new_status)
        
        # Notify stakeholders
        notify_status_change(complaint, old_status, new_status)
        
        return True, f"Complaint status updated to {get_status_name(new_status)}"
        
    except StudentComplain.DoesNotExist:
        return False, "Complaint not found"

def submit_complaint_feedback(complaint_id, feedback, rating=5):
    """Submit feedback for resolved complaint"""
    
    try:
        complaint = StudentComplain.objects.get(id=complaint_id)
        
        # Validate complaint is resolved
        if complaint.status != 2:
            return False, "Can only provide feedback for resolved complaints"
        
        # Check if feedback already submitted
        if complaint.feedback:
            return False, "Feedback already submitted for this complaint"
        
        # Update feedback
        complaint.feedback = feedback
        complaint.comment = f"Rating: {rating}/5"
        complaint.save()
        
        # Update worker and caretaker ratings
        update_personnel_ratings(complaint, rating)
        
        # Log feedback submission
        log_feedback_submission(complaint, rating)
        
        return True, "Feedback submitted successfully"
        
    except StudentComplain.DoesNotExist:
        return False, "Complaint not found"

def escalate_complaint(complaint_id, escalation_reason=""):
    """Escalate complaint to higher authority"""
    
    try:
        complaint = StudentComplain.objects.get(id=complaint_id)
        
        # Check if already escalated
        if complaint.flag == 1:
            return False, "Complaint already escalated"
        
        # Check escalation criteria
        if not should_escalate_complaint(complaint):
            return False, "Complaint does not meet escalation criteria"
        
        # Set escalation flag
        complaint.flag = 1
        complaint.reason = escalation_reason or "Escalated due to delay/complexity"
        complaint.save()
        
        # Notify section incharge
        section_incharge = get_work_type_incharge(complaint.complaint_type)
        if section_incharge:
            notify_complaint_escalation(section_incharge, complaint)
        
        # Log escalation
        log_complaint_escalation(complaint, escalation_reason)
        
        return True, "Complaint escalated successfully"
        
    except StudentComplain.DoesNotExist:
        return False, "Complaint not found"

def should_escalate_complaint(complaint):
    """Determine if complaint should be escalated"""
    
    # Time-based escalation (more than 48 hours pending)
    if complaint.status == 0:  # Pending
        time_diff = timezone.now() - complaint.complaint_date
        if time_diff.total_seconds() > 48 * 3600:  # 48 hours
            return True
    
    # Priority-based escalation (critical areas/types)
    critical_areas = ['Admin building', 'library', 'computer center']
    critical_types = ['Electricity', 'internet']
    
    if complaint.location in critical_areas or complaint.complaint_type in critical_types:
        time_diff = timezone.now() - complaint.complaint_date
        if time_diff.total_seconds() > 24 * 3600:  # 24 hours
            return True
    
    return False

def generate_complaint_analytics(start_date, end_date):
    """Generate comprehensive complaint analytics"""
    
    complaints = StudentComplain.objects.filter(
        complaint_date__range=[start_date, end_date]
    )
    
    analytics = {
        'total_complaints': complaints.count(),
        'resolved_complaints': complaints.filter(status=2).count(),
        'pending_complaints': complaints.filter(status=0).count(),
        'in_progress_complaints': complaints.filter(status=1).count(),
        'escalated_complaints': complaints.filter(flag=1).count(),
        'type_distribution': {},
        'location_distribution': {},
        'resolution_metrics': {},
        'satisfaction_metrics': {}
    }
    
    # Type distribution
    for ctype, cname in Constants.COMPLAINT_TYPE:
        count = complaints.filter(complaint_type=ctype).count()
        analytics['type_distribution'][ctype] = count
    
    # Location distribution
    for area, aname in Constants.AREA:
        count = complaints.filter(location=area).count()
        analytics['location_distribution'][area] = count
    
    # Resolution metrics
    resolved_complaints = complaints.filter(status=2, complaint_finish__isnull=False)
    if resolved_complaints.exists():
        resolution_times = []
        for complaint in resolved_complaints:
            if complaint.complaint_finish:
                resolution_time = (complaint.complaint_finish - complaint.complaint_date.date()).days
                resolution_times.append(resolution_time)
        
        if resolution_times:
            analytics['resolution_metrics'] = {
                'average_resolution_time': sum(resolution_times) / len(resolution_times),
                'min_resolution_time': min(resolution_times),
                'max_resolution_time': max(resolution_times)
            }
    
    # Satisfaction metrics
    feedback_complaints = complaints.filter(feedback__isnull=False).exclude(feedback='')
    analytics['satisfaction_metrics']['feedback_rate'] = (
        feedback_complaints.count() / max(resolved_complaints.count(), 1) * 100
    )
    
    return analytics
```

### 2. **Supervisor Model - Administrative Oversight**

```python
class Supervisor(models.Model):
    sup_id = models.ForeignKey(ExtraInfo, on_delete=models.CASCADE)
    type = models.CharField(choices=Constants.COMPLAINT_TYPE, max_length=30,default='Electricity')
```

#### **Field Analysis:**

#### **sup_id** - Supervisor Integration
```python
sup_id = models.ForeignKey(ExtraInfo, on_delete=models.CASCADE)
```
**Administrative Authority:**
- Links to institutional staff record for supervisory role
- Enables access to supervisor credentials and authority
- Supports high-level decision making and oversight
- Used for escalation and policy enforcement

#### **type** - Supervision Domain
```python
type = models.CharField(choices=Constants.COMPLAINT_TYPE, max_length=30,default='Electricity')
```
**Oversight Specialization:**
- Specific service type under supervisor oversight
- Enables specialized supervision and quality assurance
- Supports policy implementation and standards enforcement
- Used for high-level escalation and strategic planning

#### **Supervisor Business Logic:**
```python
def assign_supervisor(staff_id, supervision_type):
    """Assign supervisor for specific complaint type"""
    
    # Check if type already has supervisor
    existing_supervisor = Supervisor.objects.filter(type=supervision_type).first()
    if existing_supervisor:
        return False, f"Supervision type {supervision_type} already has supervisor: {existing_supervisor.sup_id}"
    
    # Create supervisor assignment
    supervisor = Supervisor.objects.create(
        sup_id=staff_id,
        type=supervision_type
    )
    
    return True, f"Supervisor assigned for {supervision_type}"

def escalate_to_supervisor(complaint_id):
    """Escalate complaint to supervisor level"""
    
    try:
        complaint = StudentComplain.objects.get(id=complaint_id)
        supervisor = Supervisor.objects.filter(type=complaint.complaint_type).first()
        
        if not supervisor:
            return False, f"No supervisor found for {complaint.complaint_type}"
        
        # Mark as supervisor escalation
        complaint.flag = 2  # Supervisor level escalation
        complaint.reason = "Escalated to supervisor level"
        complaint.save()
        
        # Notify supervisor
        notify_supervisor_escalation(supervisor, complaint)
        
        return True, f"Complaint escalated to supervisor: {supervisor.sup_id}"
        
    except StudentComplain.DoesNotExist:
        return False, "Complaint not found"

def supervisor_review_complaint(complaint_id, supervisor_id, action, comments=""):
    """Supervisor review and action on complaint"""
    
    try:
        complaint = StudentComplain.objects.get(id=complaint_id)
        supervisor = Supervisor.objects.get(id=supervisor_id)
        
        if action == 'approve_resolution':
            complaint.status = 2  # Resolved
            complaint.complaint_finish = timezone.now().date()
            complaint.remarks = f"Approved by supervisor: {comments}"
            
        elif action == 'reject_resolution':
            complaint.status = 1  # Back to in progress
            complaint.remarks = f"Resolution rejected by supervisor: {comments}"
            
        elif action == 'reassign':
            # Reassign to different worker/section
            reassign_complaint(complaint, comments)
            
        complaint.save()
        
        # Log supervisor action
        log_supervisor_action(supervisor, complaint, action, comments)
        
        return True, f"Supervisor action '{action}' completed"
        
    except (StudentComplain.DoesNotExist, Supervisor.DoesNotExist):
        return False, "Complaint or supervisor not found"

def generate_supervisor_dashboard(supervisor_id):
    """Generate supervisor dashboard with key metrics"""
    
    try:
        supervisor = Supervisor.objects.get(id=supervisor_id)
        
        # Get complaints under supervision
        supervised_complaints = StudentComplain.objects.filter(
            complaint_type=supervisor.type
        )
        
        dashboard = {
            'supervisor_info': {
                'name': supervisor.sup_id.user.get_full_name(),
                'supervision_type': supervisor.type
            },
            'complaint_summary': {
                'total_complaints': supervised_complaints.count(),
                'pending_complaints': supervised_complaints.filter(status=0).count(),
                'in_progress_complaints': supervised_complaints.filter(status=1).count(),
                'resolved_complaints': supervised_complaints.filter(status=2).count(),
                'escalated_complaints': supervised_complaints.filter(flag__gte=1).count()
            },
            'performance_metrics': calculate_supervision_performance(supervisor),
            'critical_issues': get_critical_complaints(supervisor.type),
            'team_performance': get_team_performance_summary(supervisor.type)
        }
        
        return dashboard
        
    except Supervisor.DoesNotExist:
        return None
```

---

## System Integration and Complete Workflow

### **Complete Complaint Management Workflow**

```python
def complete_complaint_lifecycle_workflow(complainer, complaint_type, location, specific_location, details):
    """Complete end-to-end complaint management workflow"""
    
    workflow_log = []
    
    # Step 1: Submit complaint
    success, message = submit_complaint(complainer, complaint_type, location, specific_location, details)
    if not success:
        return False, [message]
    
    complaint_id = extract_complaint_id(message)
    workflow_log.append(f"Complaint submitted: {complaint_id}")
    
    # Step 2: Auto-assignment
    complaint = StudentComplain.objects.get(id=complaint_id)
    auto_assign_complaint(complaint)
    workflow_log.append(f"Auto-assigned to: {complaint.worker_id.name if complaint.worker_id else 'None'}")
    
    # Step 3: Progress tracking
    if complaint.worker_id:
        # Simulate work progress
        update_complaint_status(complaint_id, 1, f"Work started by {complaint.worker_id.name}")
        workflow_log.append("Status updated: In Progress")
    
    # Step 4: Escalation if needed
    if should_escalate_complaint(complaint):
        escalate_complaint(complaint_id, "Auto-escalation due to complexity")
        workflow_log.append("Complaint auto-escalated")
    
    # Step 5: Resolution
    # (This would be triggered by worker action)
    # update_complaint_status(complaint_id, 2, "Issue resolved successfully")
    # workflow_log.append("Status updated: Resolved")
    
    return True, workflow_log

def generate_system_performance_report():
    """Generate comprehensive system performance report"""
    
    today = timezone.now().date()
    last_month = today - timezone.timedelta(days=30)
    
    report = {
        'system_overview': {
            'total_complaints': StudentComplain.objects.count(),
            'active_complaints': StudentComplain.objects.filter(status__in=[0, 1]).count(),
            'resolved_complaints': StudentComplain.objects.filter(status=2).count(),
            'escalated_complaints': StudentComplain.objects.filter(flag__gte=1).count()
        },
        'resource_utilization': {
            'total_caretakers': Caretaker.objects.count(),
            'total_workers': Workers.objects.count(),
            'total_section_incharges': SectionIncharge.objects.count(),
            'total_supervisors': Supervisor.objects.count()
        },
        'performance_metrics': generate_complaint_analytics(last_month, today),
        'area_performance': {},
        'type_performance': {}
    }
    
    # Area-wise performance
    for area, aname in Constants.AREA:
        area_metrics = calculate_area_performance_metrics(area)
        if area_metrics:
            report['area_performance'][area] = area_metrics
    
    # Type-wise performance
    for ctype, cname in Constants.COMPLAINT_TYPE:
        type_report = generate_section_performance_report(ctype, last_month, today)
        if type_report:
            report['type_performance'][ctype] = type_report
    
    return report

def optimize_complaint_routing():
    """Optimize complaint routing based on performance data"""
    
    optimization_results = []
    
    # Analyze worker performance
    workers = Workers.objects.all()
    for worker in workers:
        metrics = get_worker_performance_metrics(worker.id)
        if metrics and metrics['efficiency_score'] < 0.7:
            # Suggest training or reassignment
            optimization_results.append({
                'type': 'worker_improvement',
                'worker': worker.name,
                'suggestion': 'Additional training recommended',
                'current_score': metrics['efficiency_score']
            })
    
    # Analyze area coverage
    for area, aname in Constants.AREA:
        caretaker = get_area_caretaker(area)
        if not caretaker:
            optimization_results.append({
                'type': 'coverage_gap',
                'area': area,
                'suggestion': 'Assign caretaker to area'
            })
    
    # Analyze section coverage
    for ctype, cname in Constants.COMPLAINT_TYPE:
        incharge = get_work_type_incharge(ctype)
        if not incharge:
            optimization_results.append({
                'type': 'management_gap',
                'complaint_type': ctype,
                'suggestion': 'Assign section incharge'
            })
    
    return optimization_results
```

---

## **COMPLAINT SYSTEM SUMMARY**

The Complaint System provides comprehensive institutional maintenance and service request management through a structured hierarchy:

### **Human Resource Models (4):**
1. **Caretaker** - Area-wise maintenance responsibility with performance tracking
2. **SectionIncharge** - Service type leadership and technical oversight
3. **Workers** - Field personnel with specialized skills and assignments
4. **Supervisor** - Administrative oversight and high-level escalation

### **Complaint Management (1):**
1. **StudentComplain** - Core complaint workflow with complete lifecycle management

### **Key System Features:**
- **Hierarchical Management** with clear authority and responsibility chains
- **Geographic Coverage** across all campus areas and facilities
- **Service Specialization** with type-based routing and expertise matching
- **Performance Tracking** with ratings, metrics, and continuous improvement
- **Escalation Workflows** with automatic and manual escalation triggers
- **Quality Assurance** through feedback systems and supervisor oversight
- **Complete Audit Trail** with status tracking and decision logging
- **Resource Optimization** with workload balancing and efficiency monitoring

This system ensures comprehensive institutional maintenance service delivery from complaint submission through resolution tracking, with built-in quality assurance and continuous improvement mechanisms.