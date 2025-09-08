# Complete Globals Module System Documentation - Fusion IIIT (Part 1)

## System Overview
The Globals Module serves as the foundational infrastructure for the entire Fusion IIIT system. It provides core user management, authentication, authorization, profile management, feedback systems, and cross-module utilities. This module acts as the central hub that connects all other modules through shared models, utilities, and services.

---

## Database Models Analysis with Business Logic

### 1. Constants Class
**Purpose**: Centralized configuration class containing all system-wide enumerations, choices, and constants for consistent data validation across modules.

**Core Constants Structure**:
```python
class Constants:
    # Gender classification for user profiles
    SEX_CHOICES = (
        ('M', 'Male'),
        ('F', 'Female'),
        ('O', 'Other')
    )

    # User role classification for access control
    USER_CHOICES = (
        ('student', 'student'),
        ('staff', 'staff'),
        ('compounder', 'compounder'),
        ('faculty', 'faculty')
    )

    # Rating system for feedback and evaluation
    RATING_CHOICES = (
        (1, 1), (2, 2), (3, 3), (4, 4), (5, 5)
    )

    # Module identification for issue tracking and access control
    MODULES = (
        ("academic_information", "Academic"),
        ("central_mess", "Central Mess"),
        ("complaint_system", "Complaint System"),
        ("eis", "Employee Information System"),
        ("file_tracking", "File Tracking System"),
        ("health_center", "Health Center"),
        ("leave", "Leave"),
        ("online_cms", "Online Course Management System"),
        ("placement_cell", "Placement Cell"),
        ("scholarships", "Scholarships"),
        ("visitor_hostel", "Visitor Hostel"),
        ("other", "Other"),
    )

    # Issue classification for bug tracking and feature requests
    ISSUE_TYPES = (
        ("feature_request", "Feature Request"),
        ("bug_report", "Bug Report"),
        ("security_issue", "Security Issue"),
        ("ui_issue", "User Interface Issue"),
        ("other", "Other than the ones listed"),
    )

    # Professional titles for user profiles
    TITLE_CHOICES = (
        ("Mr.", "Mr."), ("Mrs.", "Mrs."), ("Ms.", "Ms."),
        ("Dr.", "Dr."), ("Professor", "Prof."),
        ("Shreemati", "Shreemati"), ("Shree", "Shree")
    )

    # Designation categories for role classification
    DESIGNATIONS = (
        ('academic', 'Academic Designation'),
        ('administrative', 'Administrative Designation'),
    )

    # User status for account lifecycle management
    USER_STATUS = (
        ("NEW", "NEW"),
        ("PRESENT", "PRESENT"),
    )
```

**Key Business Logic**:
- **Centralized Configuration**: Single source of truth for all system constants
- **Data Integrity**: Ensures consistent choices across all modules
- **Extensibility**: Easy addition of new choices without code changes
- **Validation**: Enforces standardized data entry patterns

---

### 2. ExtraInfo Model
**Purpose**: Core user profile extension that augments Django's built-in User model with institute-specific information and comprehensive user management.

**PostgreSQL Table**: `globals_extrainfo`

**Fields Structure**:
```python
class ExtraInfo(models.Model):
    id = models.CharField(max_length=20, primary_key=True)                    # Institute ID (Student/Employee ID)
    user = models.OneToOneField(User, on_delete=models.CASCADE)               # Link to Django User
    title = models.CharField(max_length=20, choices=Constants.TITLE_CHOICES, default='Dr.')
    sex = models.CharField(max_length=2, choices=Constants.SEX_CHOICES, default='M')
    date_of_birth = models.DateField(default=datetime.date(1970, 1, 1))
    user_status = models.CharField(max_length=50, choices=Constants.USER_STATUS, default='PRESENT')
    address = models.TextField(max_length=1000, default="")
    phone_no = models.BigIntegerField(null=True, default=9999999999)
    user_type = models.CharField(max_length=20, choices=Constants.USER_CHOICES)
    department = models.ForeignKey(DepartmentInfo, on_delete=models.CASCADE, null=True, blank=True)
    profile_picture = models.ImageField(null=True, blank=True, upload_to='globals/profile_pictures')
    about_me = models.TextField(default='NA', max_length=1000, blank=True)
    date_modified = models.DateTimeField('date_updated', blank=True, null=True)
    last_selected_role = models.CharField(max_length=20, null=True, blank=True)
```

**Core Business Methods with Enhanced Logic**:
```python
@property
def age(self):
    """
    CORE LOGIC: Calculate user age for age-restricted features and statistics
    
    HOW IT WORKS:
    1. Calculates time difference between current date and birth date
    2. Converts timedelta to years using integer division
    3. Provides real-time age calculation for dynamic content
    
    BUSINESS PURPOSE:
    - Age verification for restricted features
    - Statistical analysis and demographics
    - Automatic eligibility determination
    - User experience personalization
    """
    timedelta = timezone.now().date() - self.date_of_birth
    return int(timedelta.days / 365)

def get_user_role_hierarchy(self):
    """
    CORE LOGIC: Determine user's role hierarchy and permissions within institute
    
    HOW IT WORKS:
    1. Analyzes user_type and department combination
    2. Checks for held designations and administrative roles
    3. Calculates permission levels and access rights
    4. Returns structured role information
    
    BUSINESS PURPOSE:
    - Role-based access control across modules
    - Dynamic UI rendering based on permissions
    - Workflow routing and approval chains
    - Module access determination
    """
    base_roles = {
        'student': 1,
        'staff': 2,
        'faculty': 3,
        'compounder': 2
    }
    
    role_level = base_roles.get(self.user_type, 0)
    
    # Check for administrative designations
    admin_designations = HoldsDesignation.objects.filter(
        working=self.user,
        designation__type='administrative'
    )
    
    if admin_designations.exists():
        role_level += 2
    
    # Check for academic leadership roles
    academic_designations = HoldsDesignation.objects.filter(
        working=self.user,
        designation__type='academic'
    )
    
    if academic_designations.exists():
        role_level += 1
    
    return {
        'base_type': self.user_type,
        'level': role_level,
        'department': self.department.name if self.department else 'None',
        'administrative_roles': list(admin_designations.values_list('designation__name', flat=True)),
        'academic_roles': list(academic_designations.values_list('designation__name', flat=True)),
        'can_admin_modules': role_level >= 4
    }

def calculate_profile_completeness(self):
    """
    CORE LOGIC: Assess profile completion percentage for user engagement
    
    HOW IT WORKS:
    1. Checks completion status of all profile fields
    2. Assigns weights to different field types
    3. Calculates overall completion percentage
    4. Identifies missing critical information
    
    BUSINESS PURPOSE:
    - User onboarding completion tracking
    - Profile quality assessment
    - Personalized improvement suggestions
    - System reliability through complete data
    """
    fields_weight = {
        'profile_picture': 15,
        'about_me': 10,
        'phone_no': 20,
        'address': 15,
        'date_of_birth': 20,
        'department': 20
    }
    
    completed_score = 0
    total_possible = sum(fields_weight.values())
    
    # Check each field completion
    if self.profile_picture and self.profile_picture.name != 'default.jpg':
        completed_score += fields_weight['profile_picture']
    
    if self.about_me and self.about_me != 'NA' and len(self.about_me.strip()) > 0:
        completed_score += fields_weight['about_me']
    
    if self.phone_no and self.phone_no != 9999999999:
        completed_score += fields_weight['phone_no']
    
    if self.address and len(self.address.strip()) > 0:
        completed_score += fields_weight['address']
    
    if self.date_of_birth and self.date_of_birth != datetime.date(1970, 1, 1):
        completed_score += fields_weight['date_of_birth']
    
    if self.department:
        completed_score += fields_weight['department']
    
    completion_percentage = round((completed_score / total_possible) * 100, 1)
    
    return {
        'percentage': completion_percentage,
        'score': completed_score,
        'total_possible': total_possible,
        'is_complete': completion_percentage >= 80,
        'missing_fields': self._get_missing_fields(fields_weight, completed_score)
    }

def _get_missing_fields(self, fields_weight, completed_score):
    """Helper method to identify incomplete profile fields"""
    missing = []
    
    if not (self.profile_picture and self.profile_picture.name != 'default.jpg'):
        missing.append('profile_picture')
    
    if not (self.about_me and self.about_me != 'NA' and len(self.about_me.strip()) > 0):
        missing.append('about_me')
    
    if not (self.phone_no and self.phone_no != 9999999999):
        missing.append('phone_no')
    
    if not (self.address and len(self.address.strip()) > 0):
        missing.append('address')
    
    if not (self.date_of_birth and self.date_of_birth != datetime.date(1970, 1, 1)):
        missing.append('date_of_birth')
    
    if not self.department:
        missing.append('department')
    
    return missing

def get_accessible_modules(self):
    """
    CORE LOGIC: Determine which modules user can access based on role and permissions
    
    HOW IT WORKS:
    1. Checks user type and designation-based permissions
    2. Queries ModuleAccess for specific role permissions
    3. Combines default access with special permissions
    4. Returns comprehensive module access map
    
    BUSINESS PURPOSE:
    - Dynamic navigation menu generation
    - Module-level security enforcement
    - Role-based feature availability
    - Administrative access control
    """
    role_hierarchy = self.get_user_role_hierarchy()
    accessible_modules = {}
    
    # Default access for students
    if self.user_type == 'student':
        accessible_modules = {
            'academic_information': True,
            'course_registration': True,
            'examination': True,
            'hostel_management': True,
            'mess_management': True,
            'gymkhana': True,
            'placement_cell': True,
            'library': True,
            'feeds': True,
            'complaint_system': True
        }
    
    # Enhanced access for faculty
    elif self.user_type == 'faculty':
        accessible_modules = {
            'academic_information': True,
            'course_management': True,
            'examination': True,
            'eis': True,
            'research_procedures': True,
            'department': True,
            'feeds': True
        }
    
    # Administrative access
    elif self.user_type == 'staff':
        accessible_modules = {
            'file_tracking': True,
            'establishment': True,
            'finance_accounts': True,
            'hr2': True,
            'eis': True
        }
    
    # Check for designation-based additional access
    designations = HoldsDesignation.objects.filter(working=self.user)
    for designation in designations:
        try:
            module_access = ModuleAccess.objects.get(designation=designation.designation.name)
            
            # Add specific module permissions
            module_permissions = {
                'program_and_curriculum': module_access.program_and_curriculum,
                'course_registration': module_access.course_registration,
                'course_management': module_access.course_management,
                'other_academics': module_access.other_academics,
                'spacs': module_access.spacs,
                'department': module_access.department,
                'examinations': module_access.examinations,
                'hr': module_access.hr,
                'iwd': module_access.iwd,
                'complaint_management': module_access.complaint_management,
                'fts': module_access.fts,
                'purchase_and_store': module_access.purchase_and_store,
                'rspc': module_access.rspc,
                'hostel_management': module_access.hostel_management,
                'mess_management': module_access.mess_management,
                'gymkhana': module_access.gymkhana,
                'placement_cell': module_access.placement_cell,
                'visitor_hostel': module_access.visitor_hostel,
                'phc': module_access.phc
            }
            
            # Merge with existing permissions
            accessible_modules.update({k: v for k, v in module_permissions.items() if v})
            
        except ModuleAccess.DoesNotExist:
            continue
    
    return accessible_modules

def update_last_activity(self):
    """
    CORE LOGIC: Track user activity for session management and analytics
    
    HOW IT WORKS:
    1. Updates date_modified field with current timestamp
    2. Saves the model to persist activity tracking
    3. Can be called on any user action for activity monitoring
    
    BUSINESS PURPOSE:
    - User engagement analytics
    - Session timeout management
    - Active user statistics
    - System usage patterns analysis
    """
    self.date_modified = timezone.now()
    self.save(update_fields=['date_modified'])

def __str__(self):
    return '{} - {}'.format(self.id, self.user.username)
```

**Key Business Logic**:
- **One-to-One User Extension**: Seamlessly extends Django User with institute data
- **Institute ID Integration**: Links academic/employee IDs with user accounts
- **Role-Based Access**: Determines user permissions across modules
- **Profile Management**: Comprehensive user profile with media support
- **Activity Tracking**: Monitors user engagement and system usage

---

### 3. DepartmentInfo Model
**Purpose**: Master registry of all academic departments and administrative units within the institute.

**PostgreSQL Table**: `globals_departmentinfo`

**Fields Structure**:
```python
class DepartmentInfo(models.Model):
    name = models.CharField(max_length=100, unique=True)     # Department name (CSE, ECE, Finance, etc.)
    
    def __str__(self):
        return 'department: {}'.format(self.name)
```

**Enhanced Business Methods**:
```python
def get_department_statistics(self):
    """
    CORE LOGIC: Generate comprehensive department analytics and metrics
    
    HOW IT WORKS:
    1. Aggregates user counts by type within department
    2. Calculates department-specific metrics
    3. Analyzes activity patterns and engagement
    4. Returns structured statistical data
    
    BUSINESS PURPOSE:
    - Administrative reporting and analytics
    - Resource allocation planning
    - Department performance monitoring
    - Institutional decision making support
    """
    department_users = ExtraInfo.objects.filter(department=self)
    
    statistics = {
        'total_users': department_users.count(),
        'students': department_users.filter(user_type='student').count(),
        'faculty': department_users.filter(user_type='faculty').count(),
        'staff': department_users.filter(user_type='staff').count(),
        'active_users': department_users.filter(
            date_modified__gte=timezone.now() - timezone.timedelta(days=30)
        ).count() if department_users.filter(date_modified__isnull=False).exists() else 0
    }
    
    # Calculate engagement metrics
    total_active = statistics['active_users']
    total_users = statistics['total_users']
    statistics['engagement_rate'] = round(
        (total_active / total_users * 100) if total_users > 0 else 0, 2
    )
    
    return statistics

def get_department_hierarchy(self):
    """
    CORE LOGIC: Retrieve departmental organizational structure
    
    HOW IT WORKS:
    1. Identifies department heads and administrative roles
    2. Maps reporting structures and chains of command
    3. Provides organizational context for workflows
    
    BUSINESS PURPOSE:
    - Workflow routing and approvals
    - Administrative chain visualization
    - Authority determination for actions
    """
    designations = HoldsDesignation.objects.filter(
        user__extrainfo__department=self
    ).select_related('designation', 'user')
    
    hierarchy = {
        'heads': [],
        'administrators': [],
        'faculty_coordinators': [],
        'all_roles': []
    }
    
    for designation in designations:
        role_info = {
            'user': designation.user,
            'designation': designation.designation.name,
            'type': designation.designation.type
        }
        
        hierarchy['all_roles'].append(role_info)
        
        if 'head' in designation.designation.name.lower():
            hierarchy['heads'].append(role_info)
        elif designation.designation.type == 'administrative':
            hierarchy['administrators'].append(role_info)
        elif designation.designation.type == 'academic':
            hierarchy['faculty_coordinators'].append(role_info)
    
    return hierarchy
```

**Key Business Logic**:
- **Organizational Structure**: Defines institute's departmental organization
- **User Categorization**: Groups users by academic/administrative units
- **Resource Management**: Enables department-wise resource allocation
- **Reporting Hierarchy**: Supports organizational workflow systems

---

This concludes Part 1 of the Globals Module documentation. The next parts will cover:

**Part 2**: Designation, HoldsDesignation, Staff, and Faculty models
**Part 3**: Feedback, Issue, and ModuleAccess systems
**Part 4**: View functions and business logic implementation
**Part 5**: API architecture and integration patterns

Would you like me to continue with Part 2?
