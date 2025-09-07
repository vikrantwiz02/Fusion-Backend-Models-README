# Complete Department System Theory - Fusion IIIT

## System Overview
The Department System manages departmental communication, special requests, and information dissemination within the institute. It facilitates interaction between students, faculty, and department administration through announcements, special requests, and department-specific information management.

---

## Model Analysis

### 1. SpecialRequest Model
**Purpose**: Manages special requests from students/faculty to department administration with workflow tracking and status management.

**Fields Analysis**:
```python
class SpecialRequest(models.Model):
    request_maker = models.ForeignKey(ExtraInfo, on_delete=models.CASCADE)
    request_date = models.DateTimeField(default=date.today)
    brief = models.CharField(max_length=20, default='--')
    request_details = models.CharField(max_length=200)
    upload_request = models.FileField(blank=True)
    status = models.CharField(max_length=50,default='Pending')
    remarks = models.CharField(max_length=300, default="--")
    request_receiver = models.CharField(max_length=30, default="--")
```

**Business Logic Implementation**:

```python
# Enhanced SpecialRequest with workflow management
class SpecialRequest(models.Model):
    REQUEST_TYPES = [
        ('ACADEMIC', 'Academic Request'),
        ('TECHNICAL', 'Technical Support'),
        ('FACILITY', 'Facility Request'),
        ('DOCUMENT', 'Document Request'),
        ('PERMISSION', 'Permission Request'),
        ('OTHER', 'Other')
    ]
    
    STATUS_CHOICES = [
        ('PENDING', 'Pending Review'),
        ('UNDER_REVIEW', 'Under Review'),
        ('APPROVED', 'Approved'),
        ('REJECTED', 'Rejected'),
        ('COMPLETED', 'Completed'),
        ('CANCELLED', 'Cancelled')
    ]
    
    PRIORITY_LEVELS = [
        ('LOW', 'Low Priority'),
        ('MEDIUM', 'Medium Priority'),
        ('HIGH', 'High Priority'),
        ('URGENT', 'Urgent')
    ]
    
    request_maker = models.ForeignKey(ExtraInfo, on_delete=models.CASCADE, 
                                    related_name='department_requests')
    request_date = models.DateTimeField(auto_now_add=True)
    request_type = models.CharField(max_length=20, choices=REQUEST_TYPES, default='OTHER')
    brief = models.CharField(max_length=100, help_text="Brief description of request")
    request_details = models.TextField(max_length=1000, help_text="Detailed request description")
    upload_request = models.FileField(upload_to='department/requests/', blank=True, null=True)
    
    # Status and workflow
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default='PENDING')
    priority = models.CharField(max_length=10, choices=PRIORITY_LEVELS, default='MEDIUM')
    remarks = models.TextField(max_length=500, blank=True, help_text="Admin remarks")
    request_receiver = models.ForeignKey(ExtraInfo, on_delete=models.SET_NULL, 
                                       null=True, blank=True, related_name='received_requests')
    
    # Tracking fields
    assigned_date = models.DateTimeField(null=True, blank=True)
    due_date = models.DateTimeField(null=True, blank=True)
    completion_date = models.DateTimeField(null=True, blank=True)
    estimated_days = models.IntegerField(default=7, help_text="Estimated completion days")
    
    # Approval workflow
    reviewed_by = models.ForeignKey(ExtraInfo, on_delete=models.SET_NULL, 
                                  null=True, blank=True, related_name='reviewed_requests')
    approved_by = models.ForeignKey(ExtraInfo, on_delete=models.SET_NULL, 
                                  null=True, blank=True, related_name='approved_requests')
    
    class Meta:
        ordering = ['-request_date']
        indexes = [
            models.Index(fields=['status', 'priority']),
            models.Index(fields=['request_date']),
            models.Index(fields=['request_maker'])
        ]
    
    def __str__(self):
        return f"{self.request_maker.user.username} - {self.brief}"
    
    def save(self, *args, **kwargs):
        # Auto-assign due date based on priority
        if not self.due_date and self.priority:
            days_map = {'URGENT': 1, 'HIGH': 3, 'MEDIUM': 7, 'LOW': 14}
            self.due_date = self.request_date + timedelta(days=days_map.get(self.priority, 7))
        super().save(*args, **kwargs)
    
    @property
    def is_overdue(self):
        """Check if request is overdue"""
        if self.due_date and self.status not in ['COMPLETED', 'CANCELLED']:
            return timezone.now() > self.due_date
        return False
    
    @property
    def days_pending(self):
        """Calculate days since request creation"""
        return (timezone.now() - self.request_date).days
    
    def assign_to(self, receiver):
        """Assign request to specific receiver"""
        self.request_receiver = receiver
        self.assigned_date = timezone.now()
        self.status = 'UNDER_REVIEW'
        self.save()
    
    def approve(self, approver, remarks=""):
        """Approve the request"""
        self.approved_by = approver
        self.status = 'APPROVED'
        if remarks:
            self.remarks = remarks
        self.save()
    
    def complete(self, completion_remarks=""):
        """Mark request as completed"""
        self.status = 'COMPLETED'
        self.completion_date = timezone.now()
        if completion_remarks:
            self.remarks = completion_remarks
        self.save()
```

**Integration Features**:
- Email notifications for status changes
- Department-specific request routing
- Priority-based SLA management
- File attachment handling with security checks
- Automated escalation for overdue requests

---

### 2. Announcements Model
**Purpose**: Manages department-wide announcements with targeted delivery to specific batches, programs, and departments.

**Fields Analysis**:
```python
class Announcements(models.Model):
    maker_id = models.ForeignKey(ExtraInfo, on_delete=models.CASCADE)
    ann_date = models.DateTimeField(auto_now_add=True)
    message = models.CharField(max_length=200)
    batch = models.CharField(max_length=40,default="Year-1")
    department = models.CharField(max_length=40,default="ALL")
    programme = models.CharField(max_length=10)
    upload_announcement = models.FileField(upload_to='department/upload_announcement', null=True, default=" ")
```

**Business Logic Implementation**:

```python
# Enhanced Announcements with advanced targeting and scheduling
class Announcements(models.Model):
    ANNOUNCEMENT_TYPES = [
        ('GENERAL', 'General Announcement'),
        ('URGENT', 'Urgent Notice'),
        ('ACADEMIC', 'Academic Notice'),
        ('EVENT', 'Event Announcement'),
        ('DEADLINE', 'Deadline Reminder'),
        ('EXAM', 'Examination Notice'),
        ('PLACEMENT', 'Placement Notice')
    ]
    
    VISIBILITY_LEVELS = [
        ('PUBLIC', 'Public'),
        ('STUDENTS', 'Students Only'),
        ('FACULTY', 'Faculty Only'),
        ('STAFF', 'Staff Only'),
        ('DEPARTMENT', 'Department Specific')
    ]
    
    maker_id = models.ForeignKey(ExtraInfo, on_delete=models.CASCADE, 
                                related_name='department_announcements')
    ann_date = models.DateTimeField(auto_now_add=True)
    title = models.CharField(max_length=200, help_text="Announcement title")
    message = models.TextField(max_length=2000, help_text="Announcement content")
    announcement_type = models.CharField(max_length=20, choices=ANNOUNCEMENT_TYPES, default='GENERAL')
    
    # Targeting options
    batch = models.CharField(max_length=50, default="ALL", 
                           help_text="Target batch (Year-1, Year-2, etc.)")
    department = models.CharField(max_length=50, default="ALL",
                                help_text="Target department")
    programme = models.CharField(max_length=20, default="ALL",
                               help_text="Target programme (B.Tech, M.Tech, etc.)")
    visibility = models.CharField(max_length=20, choices=VISIBILITY_LEVELS, default='PUBLIC')
    
    # Content and scheduling
    upload_announcement = models.FileField(upload_to='department/announcements/', 
                                         blank=True, null=True)
    is_urgent = models.BooleanField(default=False)
    is_pinned = models.BooleanField(default=False)
    
    # Publishing controls
    is_published = models.BooleanField(default=True)
    publish_date = models.DateTimeField(null=True, blank=True)
    expiry_date = models.DateTimeField(null=True, blank=True)
    
    # Engagement tracking
    view_count = models.IntegerField(default=0)
    read_receipts = models.ManyToManyField(ExtraInfo, blank=True, 
                                         related_name='read_announcements')
    
    # Approval workflow
    requires_approval = models.BooleanField(default=False)
    approved_by = models.ForeignKey(ExtraInfo, on_delete=models.SET_NULL, 
                                  null=True, blank=True, related_name='approved_announcements')
    approval_date = models.DateTimeField(null=True, blank=True)
    
    class Meta:
        ordering = ['-is_pinned', '-is_urgent', '-ann_date']
        indexes = [
            models.Index(fields=['department', 'batch']),
            models.Index(fields=['announcement_type']),
            models.Index(fields=['is_published', 'expiry_date'])
        ]
    
    def __str__(self):
        return f"{self.title} - {self.maker_id.user.username}"
    
    @property
    def is_active(self):
        """Check if announcement is currently active"""
        now = timezone.now()
        if self.expiry_date and now > self.expiry_date:
            return False
        if self.publish_date and now < self.publish_date:
            return False
        return self.is_published
    
    @property
    def target_audience_count(self):
        """Calculate target audience size"""
        filters = {}
        if self.department != "ALL":
            filters['department__name'] = self.department
        if self.programme != "ALL":
            filters['programme'] = self.programme
        if self.batch != "ALL":
            filters['batch'] = self.batch
        
        return ExtraInfo.objects.filter(**filters).count()
    
    def mark_as_read(self, user_extra_info):
        """Mark announcement as read by user"""
        self.read_receipts.add(user_extra_info)
        self.view_count += 1
        self.save(update_fields=['view_count'])
    
    def get_read_percentage(self):
        """Calculate read percentage"""
        total_target = self.target_audience_count
        if total_target == 0:
            return 0
        read_count = self.read_receipts.count()
        return (read_count / total_target) * 100
    
    def schedule_announcement(self, publish_date, expiry_date=None):
        """Schedule announcement for future publication"""
        self.publish_date = publish_date
        self.expiry_date = expiry_date
        self.is_published = False
        self.save()
    
    def extend_expiry(self, days):
        """Extend announcement expiry"""
        if self.expiry_date:
            self.expiry_date += timedelta(days=days)
        else:
            self.expiry_date = timezone.now() + timedelta(days=days)
        self.save()
```

**Integration Features**:
- Multi-channel delivery (email, SMS, mobile push)
- Advanced targeting with demographic filters
- Read receipt tracking and analytics
- Scheduled publishing with automatic expiry
- Template-based announcement creation

---

### 3. Information Model
**Purpose**: Stores comprehensive department information including contact details, facilities, and laboratory information.

**Fields Analysis**:
```python
class Information(models.Model):
    department = models.OneToOneField(DepartmentInfo, on_delete=models.CASCADE)
    phone_number = models.BigIntegerField()
    email = models.CharField(max_length=200)
    facilites = models.TextField()
    labs = models.TextField()
```

**Business Logic Implementation**:

```python
# Enhanced Information with comprehensive department profile management
class Information(models.Model):
    DEPARTMENT_TYPES = [
        ('ACADEMIC', 'Academic Department'),
        ('ADMINISTRATIVE', 'Administrative Department'),
        ('RESEARCH', 'Research Department'),
        ('SERVICE', 'Service Department')
    ]
    
    # Core information
    department = models.OneToOneField(DepartmentInfo, on_delete=models.CASCADE,
                                    related_name='detailed_info')
    department_type = models.CharField(max_length=20, choices=DEPARTMENT_TYPES, 
                                     default='ACADEMIC')
    
    # Contact information
    phone_number = models.CharField(max_length=15, help_text="Primary contact number")
    alternate_phone = models.CharField(max_length=15, blank=True, 
                                     help_text="Alternate contact number")
    email = models.EmailField(help_text="Official department email")
    alternate_email = models.EmailField(blank=True, help_text="Alternate email")
    website = models.URLField(blank=True, help_text="Department website")
    
    # Physical information
    office_location = models.CharField(max_length=200, blank=True,
                                     help_text="Office location details")
    floor_number = models.IntegerField(null=True, blank=True)
    building_name = models.CharField(max_length=100, blank=True)
    room_numbers = models.TextField(blank=True, help_text="List of room numbers")
    
    # Operational information
    office_hours = models.TextField(blank=True, help_text="Office working hours")
    head_of_department = models.ForeignKey(ExtraInfo, on_delete=models.SET_NULL,
                                         null=True, blank=True, 
                                         related_name='headed_department')
    
    # Facilities and resources
    facilities = models.TextField(help_text="Available facilities description")
    labs = models.TextField(help_text="Laboratory information and equipment")
    equipment_list = models.TextField(blank=True, help_text="Major equipment inventory")
    software_resources = models.TextField(blank=True, help_text="Software and licenses available")
    
    # Academic information (for academic departments)
    total_faculty = models.IntegerField(default=0)
    total_students = models.IntegerField(default=0)
    courses_offered = models.TextField(blank=True, help_text="Courses offered by department")
    research_areas = models.TextField(blank=True, help_text="Major research focus areas")
    
    # Capacity and statistics
    lab_capacity = models.IntegerField(null=True, blank=True, 
                                     help_text="Total lab capacity")
    classroom_capacity = models.IntegerField(null=True, blank=True,
                                           help_text="Total classroom capacity")
    
    # Administrative
    established_year = models.IntegerField(null=True, blank=True)
    accreditation_info = models.TextField(blank=True, help_text="Accreditation details")
    
    # Media and documents
    department_logo = models.ImageField(upload_to='department/logos/', blank=True, null=True)
    brochure = models.FileField(upload_to='department/brochures/', blank=True, null=True)
    
    # Timestamps
    last_updated = models.DateTimeField(auto_now=True)
    created_date = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        verbose_name = "Department Information"
        verbose_name_plural = "Department Information"
        indexes = [
            models.Index(fields=['department_type']),
            models.Index(fields=['last_updated'])
        ]
    
    def __str__(self):
        return f"{self.department.name} - Information"
    
    @property
    def faculty_student_ratio(self):
        """Calculate faculty to student ratio"""
        if self.total_students > 0:
            return f"1:{self.total_students // max(self.total_faculty, 1)}"
        return "N/A"
    
    @property
    def contact_summary(self):
        """Get formatted contact information"""
        return {
            'primary_phone': self.phone_number,
            'primary_email': self.email,
            'website': self.website,
            'office_location': f"{self.building_name}, Floor {self.floor_number}" 
                              if self.building_name and self.floor_number else self.office_location
        }
    
    def get_facility_list(self):
        """Parse facilities into structured list"""
        if self.facilities:
            return [facility.strip() for facility in self.facilities.split('\n') if facility.strip()]
        return []
    
    def get_lab_list(self):
        """Parse labs into structured list"""
        if self.labs:
            return [lab.strip() for lab in self.labs.split('\n') if lab.strip()]
        return []
    
    def update_statistics(self):
        """Update faculty and student counts"""
        # Update total faculty count
        self.total_faculty = ExtraInfo.objects.filter(
            department=self.department,
            user_type='faculty'
        ).count()
        
        # Update total student count
        self.total_students = ExtraInfo.objects.filter(
            department=self.department,
            user_type='student'
        ).count()
        
        self.save(update_fields=['total_faculty', 'total_students'])
    
    def get_capacity_utilization(self):
        """Calculate capacity utilization"""
        utilization = {}
        if self.lab_capacity:
            # Estimate lab utilization based on active courses
            utilization['lab'] = min((self.total_students / self.lab_capacity) * 100, 100)
        if self.classroom_capacity:
            utilization['classroom'] = min((self.total_students / self.classroom_capacity) * 100, 100)
        return utilization
    
    def generate_department_report(self):
        """Generate comprehensive department report"""
        return {
            'basic_info': {
                'name': self.department.name,
                'type': self.get_department_type_display(),
                'established': self.established_year,
                'head': self.head_of_department.user.get_full_name() if self.head_of_department else None
            },
            'statistics': {
                'faculty_count': self.total_faculty,
                'student_count': self.total_students,
                'faculty_student_ratio': self.faculty_student_ratio,
                'facility_count': len(self.get_facility_list()),
                'lab_count': len(self.get_lab_list())
            },
            'capacity': self.get_capacity_utilization(),
            'contact': self.contact_summary
        }
```

**Integration Features**:
- Department directory management
- Resource booking integration
- Capacity planning and optimization
- Faculty and student analytics
- Automated report generation

---

## System Integration and Workflow

### Cross-Module Integration
```python
class DepartmentAnalytics:
    """Advanced analytics for department operations"""
    
    @staticmethod
    def get_request_trends(department=None, days=30):
        """Analyze special request trends"""
        end_date = timezone.now()
        start_date = end_date - timedelta(days=days)
        
        requests = SpecialRequest.objects.filter(
            request_date__range=[start_date, end_date]
        )
        
        if department:
            requests = requests.filter(
                request_maker__department__name=department
            )
        
        return {
            'total_requests': requests.count(),
            'pending_requests': requests.filter(status='PENDING').count(),
            'completion_rate': requests.filter(status='COMPLETED').count() / max(requests.count(), 1) * 100,
            'average_resolution_time': requests.filter(
                status='COMPLETED'
            ).aggregate(
                avg_time=models.Avg(
                    models.F('completion_date') - models.F('request_date')
                )
            )['avg_time']
        }
    
    @staticmethod
    def get_announcement_engagement(department=None, days=30):
        """Analyze announcement engagement metrics"""
        end_date = timezone.now()
        start_date = end_date - timedelta(days=days)
        
        announcements = Announcements.objects.filter(
            ann_date__range=[start_date, end_date],
            is_published=True
        )
        
        if department:
            announcements = announcements.filter(department=department)
        
        total_announcements = announcements.count()
        if total_announcements == 0:
            return {}
        
        return {
            'total_announcements': total_announcements,
            'average_view_count': announcements.aggregate(
                avg_views=models.Avg('view_count')
            )['avg_views'],
            'engagement_rate': announcements.aggregate(
                avg_engagement=models.Avg(
                    models.F('read_receipts__count') * 100.0 / 
                    models.F('view_count')
                )
            )['avg_engagement']
        }

class DepartmentNotificationService:
    """Notification service for department events"""
    
    @staticmethod
    def notify_new_request(request):
        """Send notification for new special request"""
        notification_data = {
            'type': 'SPECIAL_REQUEST',
            'title': f"New Special Request: {request.brief}",
            'message': f"Request from {request.request_maker.user.get_full_name()}",
            'priority': request.priority,
            'recipients': DepartmentNotificationService.get_department_admins(
                request.request_maker.department
            )
        }
        # Send notification through notification service
        return notification_data
    
    @staticmethod
    def notify_announcement_published(announcement):
        """Send notification for new announcement"""
        notification_data = {
            'type': 'ANNOUNCEMENT',
            'title': announcement.title,
            'message': announcement.message[:100] + "..." if len(announcement.message) > 100 else announcement.message,
            'urgent': announcement.is_urgent,
            'recipients': DepartmentNotificationService.get_target_audience(announcement)
        }
        return notification_data
```

---

## Performance Optimization

### Database Optimization
```python
# Optimized querysets for department operations
class DepartmentQueryOptimizer:
    
    @staticmethod
    def get_active_requests_with_makers():
        """Optimized query for active requests"""
        return SpecialRequest.objects.select_related(
            'request_maker__user',
            'request_maker__department',
            'request_receiver__user'
        ).filter(
            status__in=['PENDING', 'UNDER_REVIEW']
        ).order_by('priority', 'request_date')
    
    @staticmethod
    def get_department_announcements_with_engagement():
        """Optimized query for announcements with engagement data"""
        return Announcements.objects.select_related(
            'maker_id__user',
            'maker_id__department'
        ).prefetch_related(
            'read_receipts'
        ).filter(
            is_published=True,
            expiry_date__gt=timezone.now()
        ).order_by('-is_pinned', '-is_urgent', '-ann_date')
    
    @staticmethod
    def get_department_info_complete():
        """Optimized query for complete department information"""
        return Information.objects.select_related(
            'department',
            'head_of_department__user'
        ).all()

# Caching strategies
class DepartmentCache:
    
    @staticmethod
    def get_department_stats(department_id):
        """Cache department statistics"""
        cache_key = f"dept_stats_{department_id}"
        stats = cache.get(cache_key)
        
        if not stats:
            dept_info = Information.objects.get(department_id=department_id)
            stats = dept_info.generate_department_report()
            cache.set(cache_key, stats, 3600)  # Cache for 1 hour
        
        return stats
```

---

## Quality Assurance and Testing

### Automated Testing Suite
```python
class DepartmentTestSuite:
    
    def test_special_request_workflow(self):
        """Test complete special request workflow"""
        # Create test request
        request = SpecialRequest.objects.create(
            request_maker=self.test_user,
            brief="Test Request",
            request_details="Test details",
            priority="HIGH"
        )
        
        # Test assignment
        request.assign_to(self.test_admin)
        assert request.status == 'UNDER_REVIEW'
        assert request.assigned_date is not None
        
        # Test approval
        request.approve(self.test_admin, "Approved for testing")
        assert request.status == 'APPROVED'
        assert request.approved_by == self.test_admin
        
        # Test completion
        request.complete("Testing completed successfully")
        assert request.status == 'COMPLETED'
        assert request.completion_date is not None
    
    def test_announcement_targeting(self):
        """Test announcement targeting accuracy"""
        announcement = Announcements.objects.create(
            maker_id=self.test_admin,
            title="Test Announcement",
            message="Test message",
            department="CSE",
            batch="Year-3",
            programme="B.Tech"
        )
        
        target_count = announcement.target_audience_count
        # Verify targeting logic
        expected_count = ExtraInfo.objects.filter(
            department__name="CSE",
            batch="Year-3",
            programme="B.Tech"
        ).count()
        
        assert target_count == expected_count
    
    def test_department_info_validation(self):
        """Test department information validation"""
        dept_info = Information.objects.create(
            department=self.test_department,
            phone_number="1234567890",
            email="test@dept.edu",
            facilities="Test facilities",
            labs="Test labs"
        )
        
        # Test statistics update
        dept_info.update_statistics()
        assert dept_info.total_faculty >= 0
        assert dept_info.total_students >= 0
        
        # Test report generation
        report = dept_info.generate_department_report()
        assert 'basic_info' in report
        assert 'statistics' in report
        assert 'contact' in report
```
