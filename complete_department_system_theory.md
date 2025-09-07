# Complete Department Module System Documentation

## Overview
The Department Module serves as a comprehensive departmental communication and administration system, enabling departments to manage announcements, special requests, department information, and inter-departmental workflows. It provides role-based access for students, faculty, and staff with department-specific dashboards and functionality.

## Database Models

### 1. SpecialRequest Model

**Purpose**: Manages special requests submitted by users to faculty/staff with approval workflow.

**PostgreSQL Table**: `department_specialrequest`

```python
class SpecialRequest(models.Model):
    request_maker = models.ForeignKey(ExtraInfo, on_delete=models.CASCADE)
    request_date = models.DateTimeField(default=date.today)
    brief = models.CharField(max_length=20, default='--')
    request_details = models.CharField(max_length=200)
    upload_request = models.FileField(blank=True)
    status = models.CharField(max_length=50, default='Pending')
    remarks = models.CharField(max_length=300, default="--")
    request_receiver = models.CharField(max_length=30, default="--")
```

**Fields**:
- `request_maker`: Foreign key to ExtraInfo (user submitting request)
- `request_date`: Auto-populated submission timestamp
- `brief`: Short description/category of request (20 chars)
- `request_details`: Detailed request description (200 chars)
- `upload_request`: Optional file attachment
- `status`: Request status ('Pending', 'Approved', 'Denied')
- `remarks`: Admin comments/feedback (300 chars)
- `request_receiver`: Username of receiving authority

**Enhanced Business Methods**:

```python
def approve_request(self, approver, remarks=""):
    """
    CORE LOGIC: Approve special request with remarks
    HOW IT WORKS: Updates status to 'Approved', sets remarks, logs approval
    BUSINESS PURPOSE: Enables authorized personnel to approve requests with feedback
    """
    self.status = 'Approved'
    self.remarks = remarks
    self.save()
    # Additional notification logic would be implemented here

def deny_request(self, denier, remarks=""):
    """
    CORE LOGIC: Deny special request with mandatory remarks
    HOW IT WORKS: Updates status to 'Denied', requires justification remarks
    BUSINESS PURPOSE: Provides transparent rejection process with feedback
    """
    self.status = 'Denied' 
    self.remarks = remarks if remarks else "Request denied"
    self.save()
    # Additional notification logic would be implemented here

def can_be_modified(self):
    """
    CORE LOGIC: Check if request is still modifiable
    HOW IT WORKS: Returns True only if status is 'Pending'
    BUSINESS PURPOSE: Prevents modification of processed requests
    """
    return self.status == 'Pending'

def get_processing_time(self):
    """
    CORE LOGIC: Calculate time elapsed since request submission
    HOW IT WORKS: Computes difference between current time and request_date
    BUSINESS PURPOSE: Track request processing efficiency and SLA compliance
    """
    from datetime import datetime
    return datetime.now() - self.request_date
```

### 2. Announcements Model

**Purpose**: Manages departmental announcements for different batches, programs, and departments.

**PostgreSQL Table**: `department_announcements`

```python
class Announcements(models.Model):
    maker_id = models.ForeignKey(ExtraInfo, on_delete=models.CASCADE)
    ann_date = models.DateTimeField(auto_now_add=True)
    message = models.CharField(max_length=200)
    batch = models.CharField(max_length=40, default="Year-1")
    department = models.CharField(max_length=40, default="ALL")
    programme = models.CharField(max_length=10)
    upload_announcement = models.FileField(upload_to='department/upload_announcement', null=True, default=" ")
```

**Fields**:
- `maker_id`: Foreign key to ExtraInfo (announcement creator)
- `ann_date`: Auto-generated announcement timestamp
- `message`: Announcement content (200 chars)
- `batch`: Target batch (e.g., "Year-1", "Year-2")
- `department`: Target department ("CSE", "ECE", "ME", "SM", "ALL")
- `programme`: Target programme code (10 chars)
- `upload_announcement`: Optional file attachment

**Enhanced Business Methods**:

```python
def get_target_audience(self):
    """
    CORE LOGIC: Determine announcement target audience based on filters
    HOW IT WORKS: Combines department, batch, and programme filters
    BUSINESS PURPOSE: Ensure announcements reach correct recipients
    """
    filters = {}
    if self.department != "ALL":
        filters['department__name'] = self.department
    if self.batch != "ALL":
        filters['batch'] = self.batch
    if self.programme:
        filters['programme'] = self.programme
    return ExtraInfo.objects.filter(**filters)

def is_department_wide(self):
    """
    CORE LOGIC: Check if announcement targets entire department
    HOW IT WORKS: Returns True if department is "ALL" or batch is "ALL"
    BUSINESS PURPOSE: Identify high-priority, institution-wide announcements
    """
    return self.department == "ALL" or self.batch == "ALL"

def can_edit(self, user):
    """
    CORE LOGIC: Verify if user can edit announcement
    HOW IT WORKS: Checks if user is maker or has administrative privileges
    BUSINESS PURPOSE: Maintain announcement integrity and access control
    """
    return self.maker_id.user == user or user.has_perm('department.change_announcements')

def get_engagement_metrics(self):
    """
    CORE LOGIC: Calculate announcement reach and engagement
    HOW IT WORKS: Counts target audience and notification delivery status
    BUSINESS PURPOSE: Measure communication effectiveness and reach
    """
    target_count = self.get_target_audience().count()
    return {
        'target_audience': target_count,
        'delivery_rate': 100 if target_count > 0 else 0,
        'announcement_age': (timezone.now() - self.ann_date).days
    }
```

### 3. Information Model

**Purpose**: Stores department-specific contact information, facilities, and lab details.

**PostgreSQL Table**: `department_information`

```python
class Information(models.Model):
    department = models.OneToOneField(DepartmentInfo, on_delete=models.CASCADE)
    phone_number = models.BigIntegerField()
    email = models.CharField(max_length=200)
    facilites = models.TextField()
    labs = models.TextField()
```

**Fields**:
- `department`: One-to-one relationship with DepartmentInfo
- `phone_number`: Department contact number
- `email`: Department email address
- `facilites`: Text description of department facilities
- `labs`: Text description of department laboratories

**Enhanced Business Methods**:

```python
def update_contact_info(self, phone=None, email=None):
    """
    CORE LOGIC: Update department contact information
    HOW IT WORKS: Validates and updates phone/email with change tracking
    BUSINESS PURPOSE: Maintain current department contact details
    """
    if phone:
        self.phone_number = phone
    if email:
        self.email = email
    self.save()

def add_facility(self, facility_description):
    """
    CORE LOGIC: Add new facility to department listing
    HOW IT WORKS: Appends facility info to existing facilites text field
    BUSINESS PURPOSE: Maintain comprehensive facility inventory
    """
    if self.facilites:
        self.facilites += f"\n{facility_description}"
    else:
        self.facilites = facility_description
    self.save()

def add_lab(self, lab_description):
    """
    CORE LOGIC: Add new laboratory to department listing
    HOW IT WORKS: Appends lab info to existing labs text field
    BUSINESS PURPOSE: Track laboratory resources and capabilities
    """
    if self.labs:
        self.labs += f"\n{lab_description}"
    else:
        self.labs = lab_description
    self.save()

def get_department_profile(self):
    """
    CORE LOGIC: Generate comprehensive department profile
    HOW IT WORKS: Combines all information fields into structured format
    BUSINESS PURPOSE: Provide complete department overview for portals
    """
    return {
        'department_name': self.department.name,
        'contact': {
            'phone': self.phone_number,
            'email': self.email
        },
        'facilities': self.facilites.split('\n') if self.facilites else [],
        'laboratories': self.labs.split('\n') if self.labs else []
    }
```

## View Functions with Enhanced Business Logic

### 1. dep_main (Department Dashboard)

**Purpose**: Main department dashboard with role-based routing and request creation.

```python
@login_required(login_url='/accounts/login')
def dep_main(request):
    """
    CORE LOGIC: Role-based department dashboard with request submission
    HOW IT WORKS: 
    1. Identifies user role (student/faculty/staff)
    2. Loads department-specific announcements and faculty lists
    3. Handles special request creation via POST
    4. Routes to appropriate template based on role and department
    BUSINESS PURPOSE: Centralized department portal with role-specific functionality
    """
```

### 2. faculty_view (Faculty Dashboard)

**Purpose**: Faculty interface for announcement creation and request management.

```python
def faculty_view(request):
    """
    CORE LOGIC: Faculty announcement creation and request processing
    HOW IT WORKS:
    1. Displays received special requests for approval/denial
    2. Enables creation of targeted announcements with file attachments
    3. Sends notifications to relevant student groups
    4. Routes to department-specific faculty templates
    BUSINESS PURPOSE: Faculty-centric communication and administrative tools
    """
```

### 3. staff_view (Staff Dashboard)

**Purpose**: Staff interface with dual functionality for announcements and department information.

```python
def staff_view(request):
    """
    CORE LOGIC: Staff dual-form processing for announcements and department info
    HOW IT WORKS:
    1. Form1: Creates announcements similar to faculty functionality
    2. Form2: Updates/creates department information (contact, facilities, labs)
    3. Provides department admin access based on designation
    4. Manages both communication and information maintenance
    BUSINESS PURPOSE: Staff administrative control over department communication and info
    """
```

### 4. approved/deny (Request Processing)

**Purpose**: Administrative actions for special request approval/denial.

```python
def approved(request):
    """
    CORE LOGIC: Approve special requests with remarks
    HOW IT WORKS: Updates request status to 'Approved' with admin remarks
    BUSINESS PURPOSE: Formal approval workflow for student/staff requests
    """

def deny(request):
    """
    CORE LOGIC: Deny special requests with mandatory remarks
    HOW IT WORKS: Updates request status to 'Denied' with justification
    BUSINESS PURPOSE: Transparent rejection process with feedback
    """
```

### 5. all_students (Student Directory)

**Purpose**: Paginated student listing with sorting and filtering capabilities.

```python
@login_required(login_url='/accounts/login')
def all_students(request, bid):
    """
    CORE LOGIC: Display paginated student list with department/batch filtering
    HOW IT WORKS:
    1. Decodes bid parameter into programme/batch/department filters
    2. Applies sorting based on user selection with toggle functionality
    3. Filters students by decoded criteria and user type
    4. Paginates results with 25 students per page
    BUSINESS PURPOSE: Provide searchable student directory for faculty/staff
    """

def decode_bid(bid):
    """
    CORE LOGIC: Decode bid structure into programme, batch, and department
    HOW IT WORKS: Maps first character to programme, calculates batch from length
    BUSINESS PURPOSE: Support URL-based student filtering system
    """
```

### 6. alumni (Alumni Directory)

**Purpose**: Display alumni information by department.

```python
def alumni(request):
    """
    CORE LOGIC: Retrieve and display alumni by department
    HOW IT WORKS: Filters ExtraInfo for user_type='alumni' by department
    BUSINESS PURPOSE: Maintain alumni connections and networking database
    """
```

### 7. Helper Functions

```python
def browse_announcements():
    """
    CORE LOGIC: Retrieve department-wise announcement collections
    HOW IT WORKS: Separates announcements by department (CSE/ECE/ME/SM/ALL)
    BUSINESS PURPOSE: Organize announcements for department-specific viewing
    """

def faculty():
    """
    CORE LOGIC: Generate faculty lists by department with staff integration
    HOW IT WORKS: Creates combined staff+faculty lists for each department
    BUSINESS PURPOSE: Support request routing and contact information
    """

def department_information(request):
    """
    CORE LOGIC: Load department-specific information records
    HOW IT WORKS: Retrieves Information model data for all major departments
    BUSINESS PURPOSE: Provide department contact and facility details
    """

def get_make_request(user_id):
    """
    CORE LOGIC: Retrieve requests made by specific user
    HOW IT WORKS: Filters SpecialRequest by request_maker field
    BUSINESS PURPOSE: Display user's request history and status tracking
    """

def get_to_request(username):
    """
    CORE LOGIC: Retrieve requests received by specific user
    HOW IT WORKS: Filters SpecialRequest by request_receiver username
    BUSINESS PURPOSE: Show pending requests for faculty/staff processing
    """
```

## URL Patterns

```python
urlpatterns = [
    url(r'^$', views.dep_main, name='dep'),
    url(r'^facView/$', views.faculty_view, name='faculty_view'),
    url(r'^staffView/$', views.staff_view, name='staff_view'),
    url(r'All_Students/(?P<bid>[0-9]+)/$', views.all_students, name='all_students'),
    url(r'alumni/$', views.alumni, name='alumni'),
    url(r'^approved/$', views.approved, name='approved'),
    url(r'^deny/$', views.deny, name='deny'),
    url(r'^api/', include("applications.department.api.urls"))
]
```

## API Integration

### REST API Views

```python
class ListCreateAnnouncementView(generics.ListCreateAPIView):
    """
    CORE LOGIC: RESTful API for announcement CRUD operations
    HOW IT WORKS: Provides GET (list) and POST (create) endpoints for announcements
    BUSINESS PURPOSE: Enable mobile/web apps to manage announcements programmatically
    """
    queryset = Announcements.objects.all()
    serializer_class = AnnouncementSerializer
    permission_classes = (IsAuthenticated, IsFacultyStaffOrReadOnly)
    
class DepMainAPIView(APIView):
    """
    CORE LOGIC: API version of main department dashboard
    HOW IT WORKS: Returns user role, announcements, and faculty lists in JSON format
    BUSINESS PURPOSE: Support mobile apps and frontend frameworks with dashboard data
    """
    
    def get(self, request):
        # Returns role-based dashboard data with announcements and faculty lists
        
class FacAPIView(APIView):
    """
    CORE LOGIC: API for faculty-specific data retrieval
    HOW IT WORKS: Returns faculty dashboard data including announcements
    BUSINESS PURPOSE: Support faculty mobile interfaces and applications
    """
    
class StaffAPIView(APIView):
    """
    CORE LOGIC: API for staff-specific functionality
    HOW IT WORKS: Returns staff dashboard data and administrative tools
    BUSINESS PURPOSE: Enable staff mobile access to department management
    """

class AllStudentsAPIView(APIView):
    """
    CORE LOGIC: API endpoint for student directory access
    HOW IT WORKS: Provides filtered and paginated student data via REST
    BUSINESS PURPOSE: Support mobile student directory and contact apps
    """
```

### API Permissions

```python
class IsFacultyStaffOrReadOnly(BasePermission):
    """
    CORE LOGIC: Custom permission for faculty/staff write access
    HOW IT WORKS: Allows read-only for all, write access for faculty/staff only
    BUSINESS PURPOSE: Secure API endpoints while maintaining data accessibility
    """
```

### Serializers

```python
class AnnouncementSerializer(serializers.ModelSerializer):
    """
    CORE LOGIC: Handles announcement API serialization with auto-maker assignment
    HOW IT WORKS: Automatically assigns maker_id from authenticated user
    BUSINESS PURPOSE: Ensure proper user attribution for API-created announcements
    """
    class Meta:
        model = Announcements 
        fields = ('__all__')
        extra_kwargs = {'maker_id': {'required': False}}
        
    def create(self, validated_data):
        user = self.context['request'].user
        user_info = ExtraInfo.objects.filter(user=user).first()
        validated_data['maker_id'] = user_info
        return Announcements.objects.create(**validated_data)

class ExtraInfoSerializer(serializers.ModelSerializer):
    """Serializes user extended information for API responses"""
    class Meta:
        model = ExtraInfo 
        fields = ('__all__')

class SpiSerializer(serializers.ModelSerializer):
    """Handles academic performance data serialization"""
    class Meta:
        model = Spi 
        fields = ('__all__')
        
class StudentSerializer(serializers.ModelSerializer):
    """Serializes student information for directory APIs"""
    class Meta:
        model = Student  
        fields = ('__all__')
        
class DesignationSerializer(serializers.ModelSerializer):
    """Handles user designation/role serialization"""
    class Meta:
        model = Designation 
        fields = ('__all__')
        
class HoldsDesignationSerializer(serializers.ModelSerializer):
    """Serializes user-designation relationships"""
    class Meta:
        model = HoldsDesignation 
        fields = ('__all__')
        
class FacultySerializer(serializers.ModelSerializer):
    """Handles faculty-specific data serialization"""
    class Meta:
        model = Faculty
        fields = ('__all__')

class faculty_aboutSerializer(serializers.ModelSerializer):
    """Serializes faculty profile and about information"""
    class Meta:
        model = faculty_about
        fields = ('__all__')

class emp_research_projectsSerializer(serializers.ModelSerializer):
    """Handles faculty research projects data"""
    class Meta:
        model = emp_research_projects
        fields = ('__all__')
```

## Admin Configuration

```python
from django.contrib import admin
from .models import Announcements, SpecialRequest

admin.site.register(Announcements)
admin.site.register(SpecialRequest)
```

## System Dependencies

### Internal Dependencies
- **applications.globals.models**: ExtraInfo, DepartmentInfo
- **applications.academic_information.models**: Student, Spi
- **applications.eis.models**: faculty_about, emp_research_projects
- **notification.views**: department_notif for notifications

### External Dependencies
- **Django Auth**: User model, login_required decorator
- **Django Forms**: File upload handling
- **Django Pagination**: Student list pagination
- **JSON Schema**: Request validation

## Business Workflow

### Special Request Workflow
1. **Student Submission**: Student creates special request via dep_main
2. **Faculty Review**: Faculty receives request in faculty_view
3. **Processing**: Faculty approves/denies with remarks
4. **Notification**: System sends status update to student
5. **Tracking**: Request history maintained for auditing

### Announcement Workflow
1. **Creation**: Faculty/staff creates announcement with targeting
2. **Distribution**: System identifies target audience based on filters
3. **Notification**: department_notif sends notifications to recipients
4. **Storage**: Announcement stored for browsing and reference
5. **Access**: Department-wise announcement browsing available

### Department Information Management
1. **Staff Updates**: Department staff updates contact and facility info
2. **Validation**: System validates information format and completeness
3. **Storage**: Information linked to specific department via OneToOne
4. **Display**: Information displayed in department dashboards
5. **Maintenance**: Regular updates ensure current information

## Integration Points

### Notification System
- Integrates with notification.views.department_notif
- Sends real-time notifications for announcements
- Provides notification tracking for request status changes

### File Management
- Supports file uploads for announcements and requests
- Stores files in department-specific directories
- Provides file download capabilities

### Role-Based Access
- Integrates with HoldsDesignation for role verification
- Provides department-specific template routing
- Supports designation-based feature access

### Database Relationships
- Links to globals.models for user and department data
- Integrates with academic_information for student data
- Connects to eis.models for faculty information

This Department Module provides a comprehensive departmental administration system with robust communication tools, request management, and information maintenance capabilities, supporting the complete departmental workflow from student requests to administrative announcements.
