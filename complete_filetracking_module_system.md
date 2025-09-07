# Complete File Tracking Module System Documentation

## Overview
The File Tracking Module provides a comprehensive digital file management and workflow system that enables employees to create, forward, track, and manage documents throughout an organization. It implements a hierarchical tracking system where files move through different designations with complete audit trails, supporting drafts, inbox/outbox management, and archival functionality.

## Database Models

### 1. File Model

**Purpose**: Core file entity representing documents in the tracking system with metadata and source integration.

**PostgreSQL Table**: `File`

```python
class File(models.Model):
    uploader = models.ForeignKey(ExtraInfo, on_delete=models.CASCADE, related_name='uploaded_files')
    designation = models.ForeignKey(Designation, on_delete=models.CASCADE, null=True, related_name='upload_designation')
    subject = models.CharField(max_length=100, null=True, blank=True)
    description = models.CharField(max_length=400, null=True, blank=True)
    upload_date = models.DateTimeField(auto_now_add=True)
    upload_file = models.FileField(blank=True)
    is_read = models.BooleanField(default=False)
    
    # API and module integration fields
    src_module = models.CharField(max_length=100, default='filetracking')
    src_object_id = models.CharField(max_length=100, null=True) 
    file_extra_JSON = models.JSONField(null=True)
```

**Fields**:
- `uploader`: Foreign key to ExtraInfo (file creator)
- `designation`: Foreign key to Designation (uploader's role)
- `subject`: File title/subject (100 chars)
- `description`: Detailed file description (400 chars)
- `upload_date`: Auto-generated creation timestamp
- `upload_file`: Optional file attachment
- `is_read`: Archive status flag
- `src_module`: Source module identifier for integration
- `src_object_id`: Source object reference for cross-module tracking
- `file_extra_JSON`: Additional metadata in JSON format

**Enhanced Business Methods**:

```python
def get_current_owner(self):
    """
    CORE LOGIC: Determine current file owner based on latest tracking entry
    HOW IT WORKS: Queries most recent Tracking record to find current holder
    BUSINESS PURPOSE: Identify who currently has responsibility for the file
    """
    latest_tracking = self.tracking_set.order_by('-receive_date').first()
    return latest_tracking.receiver_id if latest_tracking else self.uploader.user

def get_current_designation(self):
    """
    CORE LOGIC: Get current designation holding the file
    HOW IT WORKS: Retrieves receive_design from latest tracking entry
    BUSINESS PURPOSE: Determine appropriate workflow and permissions context
    """
    latest_tracking = self.tracking_set.order_by('-receive_date').first()
    return latest_tracking.receive_design if latest_tracking else self.designation

def can_be_forwarded_by(self, user, designation):
    """
    CORE LOGIC: Verify if user has permission to forward this file
    HOW IT WORKS: Checks if user is current owner with specified designation
    BUSINESS PURPOSE: Enforce workflow permissions and prevent unauthorized forwarding
    """
    current_owner = self.get_current_owner()
    current_designation = self.get_current_designation()
    return (current_owner == user and 
            current_designation.name == designation)

def can_be_archived_by(self, user):
    """
    CORE LOGIC: Determine if user can archive this file
    HOW IT WORKS: Checks if user is both current owner and original uploader
    BUSINESS PURPOSE: Allow archiving only by appropriate authority to maintain workflow integrity
    """
    return (self.get_current_owner() == user and 
            self.uploader.user == user)

def get_workflow_history(self):
    """
    CORE LOGIC: Generate complete workflow history with timing and actors
    HOW IT WORKS: Orders all tracking entries chronologically with user details
    BUSINESS PURPOSE: Provide audit trail and workflow transparency
    """
    history = []
    for tracking in self.tracking_set.order_by('receive_date'):
        history.append({
            'date': tracking.receive_date,
            'from_user': tracking.current_id.user.username,
            'from_designation': tracking.current_design.designation.name,
            'to_user': tracking.receiver_id.username,
            'to_designation': tracking.receive_design.name,
            'remarks': tracking.remarks,
            'has_attachment': bool(tracking.upload_file)
        })
    return history

def calculate_processing_time(self):
    """
    CORE LOGIC: Calculate total time file has been in workflow
    HOW IT WORKS: Computes difference between creation and latest activity
    BUSINESS PURPOSE: Track efficiency and identify bottlenecks in workflow
    """
    from django.utils import timezone
    latest_tracking = self.tracking_set.order_by('-receive_date').first()
    end_time = latest_tracking.receive_date if latest_tracking else timezone.now()
    return end_time - self.upload_date

def archive(self, archiver_user):
    """
    CORE LOGIC: Archive file and update all related tracking entries
    HOW IT WORKS: Sets is_read=True and logs archival action
    BUSINESS PURPOSE: Mark file as completed and remove from active workflows
    """
    if self.can_be_archived_by(archiver_user):
        self.is_read = True
        self.save()
        # Mark all tracking entries as read
        self.tracking_set.update(is_read=True)
        return True
    return False
```

### 2. Tracking Model

**Purpose**: Records each step in the file workflow with complete audit trail and metadata.

**PostgreSQL Table**: `Tracking`

```python
class Tracking(models.Model):
    file_id = models.ForeignKey(File, on_delete=models.CASCADE, null=True)
    current_id = models.ForeignKey(ExtraInfo, on_delete=models.CASCADE)
    current_design = models.ForeignKey(HoldsDesignation, null=True, on_delete=models.CASCADE)
    receiver_id = models.ForeignKey(User, null=True, on_delete=models.CASCADE, related_name='receiver_id')
    receive_design = models.ForeignKey(Designation, null=True, on_delete=models.CASCADE, related_name='rec_design')
    
    receive_date = models.DateTimeField(auto_now_add=True)
    forward_date = models.DateTimeField(auto_now_add=True)
    remarks = models.CharField(max_length=250, null=True, blank=True)
    upload_file = models.FileField(blank=True)
    is_read = models.BooleanField(default=False)
    
    # API integration
    tracking_extra_JSON = models.JSONField(null=True)
```

**Fields**:
- `file_id`: Foreign key to File being tracked
- `current_id`: ExtraInfo of current file holder
- `current_design`: HoldsDesignation of current holder
- `receiver_id`: User receiving the file
- `receive_design`: Designation of receiver
- `receive_date`: Timestamp when file was received
- `forward_date`: Timestamp when file was forwarded
- `remarks`: Comments/notes for this tracking step (250 chars)
- `upload_file`: Optional additional attachment
- `is_read`: Processing status flag
- `tracking_extra_JSON`: Additional metadata for API integration

**Enhanced Business Methods**:

```python
def get_step_duration(self):
    """
    CORE LOGIC: Calculate time file spent at this workflow step
    HOW IT WORKS: Computes difference between receive_date and forward_date
    BUSINESS PURPOSE: Measure processing efficiency at each workflow stage
    """
    return self.forward_date - self.receive_date

def is_current_step(self):
    """
    CORE LOGIC: Determine if this is the current/active tracking step
    HOW IT WORKS: Checks if this is the latest tracking entry for the file
    BUSINESS PURPOSE: Identify current file location in workflow
    """
    latest = Tracking.objects.filter(file_id=self.file_id).order_by('-receive_date').first()
    return latest and latest.id == self.id

def get_next_possible_actions(self):
    """
    CORE LOGIC: Determine available actions for current tracking step
    HOW IT WORKS: Analyzes current state and user permissions
    BUSINESS PURPOSE: Guide users on available workflow operations
    """
    actions = []
    if self.is_current_step():
        actions.append('forward')
        if self.file_id.can_be_archived_by(self.receiver_id):
            actions.append('archive')
    return actions

def validate_forward_permissions(self, forwarding_user, target_designation):
    """
    CORE LOGIC: Validate if user can forward file to specified designation
    HOW IT WORKS: Checks workflow rules and designation hierarchy
    BUSINESS PURPOSE: Ensure proper workflow adherence and authorization
    """
    if not self.is_current_step():
        return False, "File is not at current step"
    
    if self.receiver_id != forwarding_user:
        return False, "Only current file holder can forward"
    
    # Additional business rule validations can be added here
    return True, "Forward permission granted"

def create_forward_entry(self, receiver_username, receiver_designation, remarks="", attachment=None):
    """
    CORE LOGIC: Create next tracking entry for file forwarding
    HOW IT WORKS: Creates new Tracking record with validated parameters
    BUSINESS PURPOSE: Maintain continuous audit trail through workflow
    """
    from django.contrib.auth.models import User
    
    receiver_user = User.objects.get(username=receiver_username)
    receiver_desig = Designation.objects.get(name=receiver_designation)
    
    new_tracking = Tracking.objects.create(
        file_id=self.file_id,
        current_id=self.receiver_id.extrainfo,
        current_design=HoldsDesignation.objects.get(
            user=self.receiver_id, 
            designation=self.receive_design
        ),
        receiver_id=receiver_user,
        receive_design=receiver_desig,
        remarks=remarks,
        upload_file=attachment
    )
    
    # Mark current tracking as read
    self.is_read = True
    self.save()
    
    return new_tracking
```

## View Functions with Enhanced Business Logic

### 1. filetracking (File Creation)

**Purpose**: Main interface for creating new files and drafts with designation-based permissions.

```python
@login_required(login_url="/accounts/login/")
@user_is_student
@dropdown_designation_valid
def filetracking(request):
    """
    CORE LOGIC: Handle file creation with dual modes (save draft vs send immediately)
    HOW IT WORKS:
    1. Validates user permissions and file size (10MB limit)
    2. 'save' action: Creates draft file without tracking entry
    3. 'send' action: Creates file and initial tracking entry, sends notification
    4. Enforces designation-based access control throughout
    BUSINESS PURPOSE: Centralized file creation with flexible workflow initiation
    """
```

### 2. inbox_view (Received Files)

**Purpose**: Display and manage files received by the current user in their designation capacity.

```python
@login_required(login_url="/accounts/login")
@user_is_student
@dropdown_designation_valid
def inbox_view(request):
    """
    CORE LOGIC: Retrieve and display files in user's inbox with filtering and pagination
    HOW IT WORKS:
    1. Gets files received by user in current designation capacity
    2. Adds metadata like receive_date and forwarding status
    3. Implements search/filter by subject, sender, and date
    4. Provides pagination for large file lists
    BUSINESS PURPOSE: Organized view of pending files requiring user attention
    """
```

### 3. outbox_view (Sent Files)

**Purpose**: Track and manage files sent by the current user to other employees.

```python
@login_required(login_url="/accounts/login")
@user_is_student
@dropdown_designation_valid
def outbox_view(request):
    """
    CORE LOGIC: Display files sent by user with recipient and status information
    HOW IT WORKS:
    1. Retrieves files where user was the sender in tracking entries
    2. Shows recipient details and send timestamps
    3. Provides search functionality and pagination
    4. Indicates current file status and location
    BUSINESS PURPOSE: Track outgoing files and monitor workflow progress
    """
```

### 4. forward (File Forwarding)

**Purpose**: Forward received files to other users with remarks and optional attachments.

```python
@login_required(login_url="/accounts/login")
@user_is_student
@dropdown_designation_valid
def forward(request, id):
    """
    CORE LOGIC: Handle file forwarding with validation and notification
    HOW IT WORKS:
    1. Validates user has permission to forward specific file
    2. Creates new tracking entry with recipient and remarks
    3. Optionally adds new attachments to the workflow
    4. Sends notification to recipient and updates file status
    BUSINESS PURPOSE: Enable controlled file movement through organizational hierarchy
    """
```

### 5. drafts_view (Draft Management)

**Purpose**: Manage and edit draft files before sending them through the workflow.

```python
@login_required(login_url="/accounts/login")
@user_is_student
@dropdown_designation_valid
def drafts_view(request, id):
    """
    CORE LOGIC: Display user's draft files with editing and sending capabilities
    HOW IT WORKS:
    1. Shows files created but not yet sent (no tracking entries)
    2. Allows editing of file metadata and attachments
    3. Provides option to send drafts through workflow initiation
    4. Maintains version control for draft modifications
    BUSINESS PURPOSE: Prepare and refine files before formal workflow submission
    """
```

### 6. archive_view (Archived Files)

**Purpose**: View and manage archived/completed files with history.

```python
@login_required(login_url="/accounts/login")
@user_is_student
@dropdown_designation_valid
def archive_view(request, id):
    """
    CORE LOGIC: Display archived files with complete workflow history
    HOW IT WORKS:
    1. Shows files marked as completed (is_read=True)
    2. Provides complete audit trail and processing history
    3. Allows unarchiving for reopening workflows if needed
    4. Generates reports and analytics on file processing
    BUSINESS PURPOSE: Historical record keeping and workflow analysis
    """
```

### 7. view_file (File Details)

**Purpose**: Detailed view of individual files with workflow history and action options.

```python
@login_required(login_url="/accounts/login")
@user_is_student
@dropdown_designation_valid
def view_file(request, id):
    """
    CORE LOGIC: Comprehensive file view with conditional action permissions
    HOW IT WORKS:
    1. Displays file details, attachments, and complete tracking history
    2. Shows available actions based on user permissions and file status
    3. Provides context-sensitive forms for forwarding or archiving
    4. Maintains security through permission-based UI rendering
    BUSINESS PURPOSE: Central hub for file interaction and workflow management
    """
```

## SDK Methods (Core Business Logic)

### File Management Methods

```python
def create_file(uploader, uploader_designation, receiver, receiver_designation, 
               subject="", description="", src_module="filetracking", 
               src_object_id="", file_extra_JSON={}, attached_file=None) -> int:
    """
    CORE LOGIC: Programmatic file creation with automatic tracking initiation
    HOW IT WORKS: Creates File object and initial Tracking entry atomically
    BUSINESS PURPOSE: Enable other modules to integrate with file tracking system
    """

def create_draft(uploader, uploader_designation, src_module="filetracking",
                src_object_id="", file_extra_JSON={}, attached_file=None) -> int:
    """
    CORE LOGIC: Create draft file without workflow initiation
    HOW IT WORKS: Creates File object without Tracking entry for later sending
    BUSINESS PURPOSE: Support draft workflow for file preparation
    """

def forward_file(file_id, receiver, receiver_designation, file_extra_JSON,
                remarks="", file_attachment=None) -> int:
    """
    CORE LOGIC: Programmatic file forwarding with validation
    HOW IT WORKS: Creates new tracking entry after validating current ownership
    BUSINESS PURPOSE: Enable API and module-based file forwarding
    """
```

### Inbox/Outbox Methods

```python
def view_inbox(username, designation, src_module) -> list:
    """
    CORE LOGIC: Retrieve user's received files for specific designation and module
    HOW IT WORKS: Filters tracking entries by receiver and designation
    BUSINESS PURPOSE: Support inbox functionality across different modules
    """

def view_outbox(username, designation, src_module) -> list:
    """
    CORE LOGIC: Retrieve user's sent files for specific designation and module
    HOW IT WORKS: Filters tracking entries by sender and designation
    BUSINESS PURPOSE: Support outbox functionality with sent file tracking
    """

def view_drafts(username, designation, src_module) -> dict:
    """
    CORE LOGIC: Retrieve user's draft files that haven't been sent
    HOW IT WORKS: Finds File objects without associated Tracking entries
    BUSINESS PURPOSE: Support draft management across modules
    """
```

### Archive and History Methods

```python
def archive_file(file_id) -> bool:
    """
    CORE LOGIC: Mark file as archived/completed
    HOW IT WORKS: Sets is_read=True on File object
    BUSINESS PURPOSE: Remove files from active workflow tracking
    """

def view_history(file_id) -> dict:
    """
    CORE LOGIC: Retrieve complete workflow history for a file
    HOW IT WORKS: Orders all tracking entries chronologically
    BUSINESS PURPOSE: Provide audit trail and workflow analysis
    """

def view_archived(username, designation, src_module) -> dict:
    """
    CORE LOGIC: Retrieve user's archived files
    HOW IT WORKS: Finds files marked as read that user has interacted with
    BUSINESS PURPOSE: Support archived file management and retrieval
    """
```

## URL Patterns

```python
urlpatterns = [
    url(r'^$', views.filetracking, name='filetracking'),
    url(r'^draftdesign/$', views.draft_design, name='draft_design'),
    url(r'^drafts/(?P<id>\d+)$', views.drafts_view, name='drafts_view'),
    url(r'^outbox/(?P<id>\d+)$', views.outbox_view, name='outbox_view'),
    url(r'^inbox/$', views.inbox_view, name='inbox_view'),
    url(r'^outward/$', views.outbox_view, name='outward'),
    url(r'^inward/$', views.inbox_view, name='inward'),
    url(r'^archive/(?P<id>\d+)/$', views.archive_view, name='archive_view'),
    url(r'^finish/(?P<id>\d+)/$', views.archive_file, name='finish_file'),
    url(r'^viewfile/(?P<id>\d+)/$', views.view_file, name='view_file_view'),
    url(r'^forward/(?P<id>\d+)/$', views.forward, name='forward'),
    url(r'^forward_inward/(?P<id>\d+)/$', views.forward_inward, name='forward_inward'),
    url(r'^delete/(?P<id>\d+)$', views.delete, name='delete'),
    url(r'^editdraft/(?P<id>\w+)/$', views.edit_draft_view, name="edit_draft"),
    url(r'^download_file/(?P<id>\w+)/$', views.download_file, name="download_file"),
    url(r'^api/', include(urls))
]
```

## API Integration

### REST API Views

```python
class CreateFileView(APIView):
    """
    CORE LOGIC: RESTful file creation with authentication
    HOW IT WORKS: Accepts JSON payload and creates file with tracking
    BUSINESS PURPOSE: Enable external systems to integrate with file tracking
    """

class ViewInboxView(APIView):
    """
    CORE LOGIC: API access to user inbox with filtering
    HOW IT WORKS: Returns paginated inbox files in JSON format
    BUSINESS PURPOSE: Support mobile apps and external dashboards
    """

class ForwardFileView(APIView):
    """
    CORE LOGIC: API-based file forwarding with validation
    HOW IT WORKS: Validates permissions and creates tracking entries
    BUSINESS PURPOSE: Enable automated workflow processing
    """

class CreateDraftFile(APIView):
    """
    CORE LOGIC: API draft creation for later sending
    HOW IT WORKS: Creates File objects without tracking initiation
    BUSINESS PURPOSE: Support API-based draft workflow
    """

class ArchiveFileView(APIView):
    """
    CORE LOGIC: API file archiving with permission validation
    HOW IT WORKS: Marks files as archived with proper authorization
    BUSINESS PURPOSE: Enable programmatic workflow completion
    """
```

## Decorators and Utilities

### Access Control Decorators

```python
@user_is_student
def check_user_type(view_func):
    """
    CORE LOGIC: Ensure only non-student users access file tracking
    HOW IT WORKS: Validates user type before allowing view access
    BUSINESS PURPOSE: Restrict file tracking to employees only
    """

@dropdown_designation_valid
def validate_designation(view_func):
    """
    CORE LOGIC: Verify user has valid designation context
    HOW IT WORKS: Checks session for current designation selection
    BUSINESS PURPOSE: Ensure designation-based permissions are maintained
    """
```

### Utility Functions

```python
def get_designation(user):
    """
    CORE LOGIC: Retrieve all designations held by user
    HOW IT WORKS: Queries HoldsDesignation for user's active roles
    BUSINESS PURPOSE: Support designation dropdown and permission checking
    """

def get_current_file_owner(file_id):
    """
    CORE LOGIC: Identify current owner of a file in workflow
    HOW IT WORKS: Finds latest tracking entry receiver
    BUSINESS PURPOSE: Determine file permissions and current location
    """
```

## Admin Configuration

```python
class FileAdmin(admin.ModelAdmin):
    list_display = ('id', 'uploader', 'designation', 'subject', 'upload_date', 'is_read')
    search_fields = ('uploader__user__username', 'subject', 'description')
    list_filter = ('is_read',)

class TrackingAdmin(admin.ModelAdmin):
    list_display = ('file_id', 'current_id', 'receiver_id', 'receive_date', 'forward_date', 'is_read')
    search_fields = ('file_id__subject', 'current_id__user__username', 'receiver_id__username')
    list_filter = ('is_read',)
```

## System Dependencies

### Internal Dependencies
- **applications.globals.models**: ExtraInfo, HoldsDesignation, Designation
- **notification.views**: file_tracking_notif for workflow notifications
- **Django Auth**: User model, authentication decorators

### External Dependencies
- **Django Core**: File handling, pagination, serializers
- **REST Framework**: API views, authentication, permissions
- **ReportLab**: PDF generation for file reports
- **JSON**: Metadata storage and API communication

## Business Workflow

### File Creation Workflow
1. **Draft Creation**: User creates draft file with metadata and attachments
2. **Review and Edit**: Draft can be modified before sending
3. **Workflow Initiation**: Sending creates initial tracking entry
4. **Notification**: Receiver gets notification of new file
5. **Active Tracking**: File enters active workflow system

### Forwarding Workflow
1. **Permission Validation**: System verifies user can forward file
2. **Recipient Selection**: User selects receiver and designation
3. **Tracking Creation**: New tracking entry records forwarding
4. **Status Update**: Previous tracking marked as read
5. **Notification**: Recipient notified of incoming file

### Archive Workflow
1. **Completion Check**: Verify file processing is complete
2. **Permission Validation**: Ensure user can archive file
3. **Status Update**: Mark file and tracking as archived
4. **Workflow Removal**: Remove from active inbox/outbox views
5. **Historical Record**: Maintain in archived files for audit

## Integration Points

### Notification System
- Integrates with notification.views.file_tracking_notif
- Sends real-time notifications for file forwarding
- Supports cross-module notification triggering

### Cross-Module Integration
- Supports src_module and src_object_id for external module files
- Provides SDK methods for programmatic integration
- Maintains foreign key relationships with other modules

### Permission System
- Integrates with HoldsDesignation for role-based access
- Supports designation-specific file visibility
- Enforces workflow permissions throughout system

### File Management
- Handles file uploads with size validation
- Supports multiple attachments per tracking step
- Provides download capabilities with access control

This File Tracking Module provides a comprehensive document workflow management system with robust permission controls, complete audit trails, and extensive integration capabilities for organizational file processing and tracking.
