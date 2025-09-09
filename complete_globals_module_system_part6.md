# Complete Globals Module System Documentation - Fusion IIIT (Part 6)

## View Functions and Business Logic Implementation

### Overview
The globals module provides core view functions that handle authentication, user management, dashboard functionality, and system-wide utilities for the Fusion IIIT system.

---

### 1. RateLimitedPasswordResetView (Class-Based View)
**Purpose**: Extends Django's built-in PasswordResetView with rate limiting functionality to prevent abuse of password reset requests.

**Business Logic**:
```python
class RateLimitedPasswordResetView(PasswordResetView):
    template_name = 'registration/password_reset_form.html'

    def post(self, request, *args, **kwargs):
        """
        CORE LOGIC: Rate-limited password reset with security enforcement
        
        HOW IT WORKS:
        1. Extracts email from POST request
        2. Creates or retrieves PasswordResetTracker for email
        3. Enforces 24-hour rate limiting rule
        4. Updates tracker timestamp on successful reset
        5. Proceeds with Django's standard reset process
        
        BUSINESS PURPOSE:
        - Prevent password reset abuse and brute force attacks
        - Maintain audit trail for security monitoring
        - Provide user-friendly error messaging for rate limits
        - Integrate seamlessly with Django's authentication system
        """
        email = request.POST.get('email')
        if not email:
            return self.form_invalid(self.get_form())

        # Security checkpoint: Get or create tracker
        tracker, created = PasswordResetTracker.objects.get_or_create(email=email)

        # Rate limiting enforcement (24-hour window)
        if tracker.last_reset and now() - tracker.last_reset < timedelta(days=1):
            return render(request, self.template_name, {
                'form': self.get_form(),
                'error_message': "Password can only be reset once every 24 hours.",
            })

        # Update tracker and proceed
        tracker.last_reset = now()
        tracker.save()
        
        return super().post(request, *args, **kwargs)
```

**Key Features**:
- **Security Rate Limiting**: 24-hour window between reset requests
- **Audit Trail**: Tracks all reset attempts with timestamps
- **User Experience**: Clear error messaging for rate limit violations
- **Integration**: Seamless integration with Django's password reset workflow

---

### 2. index(request)
**Purpose**: Landing page controller that handles initial user routing based on authentication status.

**Business Logic**:
```python
def index(request):
    """
    CORE LOGIC: Smart routing for authenticated and anonymous users
    
    HOW IT WORKS:
    1. Checks user authentication status
    2. Redirects authenticated users to dashboard
    3. Displays landing page for anonymous users
    4. Provides clean entry point to application
    
    BUSINESS PURPOSE:
    - Seamless user experience with automatic routing
    - Clear separation between public and authenticated areas
    - Performance optimization by avoiding unnecessary page loads
    - Consistent application entry point management
    """
    context = {}
    
    # Smart routing logic
    if str(request.user) != "AnonymousUser":
        # Authenticated users go directly to dashboard
        return HttpResponseRedirect('/dashboard/')
    else:
        # Anonymous users see the landing page
        return render(request, "globals/index1.html", context)
```

**Key Features**:
- **Smart Routing**: Automatic redirection based on authentication
- **Performance**: Direct routing without unnecessary template rendering
- **User Experience**: Seamless navigation flow
- **Security**: Clear separation of authenticated and public areas

---

### 3. reset_all_pass(request)
**Purpose**: Development utility function for resetting all user passwords in development environments.

**Business Logic**:
```python
def reset_all_pass(request):
    """
    CORE LOGIC: Bulk password reset for development environments
    
    HOW IT WORKS:
    1. Checks if password reset is allowed in current environment
    2. Retrieves all user accounts from the system
    3. Sets standard development password for all accounts
    4. Returns count of affected accounts
    5. Provides security by environment-based access control
    
    BUSINESS PURPOSE:
    - Development environment setup and testing
    - Quick account access for development teams
    - Consistent development credentials across team
    - Security by restricting to development environments only
    
    SECURITY NOTE: Only available when ALLOW_PASS_RESET setting is True
    """
    # Environment-based security check
    if settings.ALLOW_PASS_RESET:
        UserMod = get_user_model()
        arr = UserMod.objects.all()
        
        # Bulk password reset operation
        for e in arr:
            print(e.username)  # Development logging
            u = User.objects.get(username=e.username)
            u.set_password('user@123')  # Standard dev password
            u.save()
        
        # Return operation summary
        context = {"done": len(arr)}
        return HttpResponse(json.dumps(context), "application/json")
    else:
        # Security: Not allowed in production
        return HttpResponseNotFound("Not allowed")
```

**Key Features**:
- **Environment Security**: Only works in development environments
- **Bulk Operations**: Efficiently handles multiple user accounts
- **Audit Trail**: Logs operations for development tracking
- **JSON Response**: API-compatible response format

---

### 4. about(request)
**Purpose**: Team information and credits page displaying development team details and project attribution.

**Business Logic**:
```python
@login_required(login_url=LOGIN_URL)
def about(request):
    """
    CORE LOGIC: Team information and project credits display
    
    HOW IT WORKS:
    1. Structures team information by functional modules
    2. Organizes developer profiles with roles and images
    3. Provides comprehensive project attribution
    4. Renders organized team information for display
    
    BUSINESS PURPOSE:
    - Project transparency and team recognition
    - Module ownership and responsibility tracking
    - Developer portfolio and contribution visibility
    - Educational value for new team members
    """
    # Module-wise team organization
    teams = {
        'uiTeam': {'teamId': "uiTeam", 'teamName': "Frontend Team"},
        'qaTeam': {'teamId': "qaTeam", 'teamName': "Quality Analysis Team"},
        'academics_a_Team': {'teamId': "academics_a_Team", 'teamName': "Academics (A) Module Team"},
        'academics_b_Team': {'teamId': "academics_b_Team", 'teamName': "Academics (B) Module Team"},
        'spacsTeam': {'teamId': "spacsTeam", 'teamName': "Awards & Scholarship Module Team"},
        'messTeam': {'teamId': "messTeam", 'teamName': "Central Mess Module Team"},
        'complaintTeam': {'teamId': "complaintTeam", 'teamName': "Complaint Module Team"},
        'eisTeam': {'teamId': "eisTeam", 'teamName': "EIS Module Team"},
        'filetrackingTeam': {'teamId': "filetrackingTeam", 'teamName': "File Tracking Module Team"},
        'gymkhanaTeam': {'teamId': "gymkhanaTeam", 'teamName': "Gymkhana Module Team"},
        'leaveTeam': {'teamId': "leaveTeam", 'teamName': "Leave Module Team"},
        'phcTeam': {'teamId': "phcTeam", 'teamName': "Primary Health Center Module Team"},
        'placementTeam': {'teamId': "placementTeam", 'teamName': "Placement Module Team"},
        'vhTeam': {'teamId': "vhTeam", 'teamName': "Visitors Hostel Module Team"},
    }
    
    # Core development team with detailed profiles
    context = {
        'teams': teams,
        'psgTeam': {
            'dev1': {
                'devName': 'Anuraag Singh',
                'devImage': 'team/2015043.jpeg',
                'devTitle': 'Developer'
            },
            'dev2': {
                'devName': 'Kanishka Munshi',
                'devImage': 'team/2015121.jpg',
                'devTitle': 'Head UI Developer'
            },
            'dev3': {
                'devName': 'M. Arshad Siddiqui',
                'devImage': 'team/2015153.jpg',
                'devTitle': 'Database Designer'
            },
            'dev4': {
                'devName': 'Pranjul Shukla',
                'devImage': 'team/2015325.jpg',
                'devTitle': 'Developer'
            },
            'dev5': {
                'devName': 'Saket Patel',
                'devImage': 'team/2015329.jpg',
                'devTitle': 'Head Developer'
            },
        },
        # Additional team structures for UI, QA, and module-specific teams...
    }
    
    return render(request, "globals/about.html", context)
```

**Key Features**:
- **Module Attribution**: Clear ownership and responsibility mapping
- **Team Recognition**: Comprehensive developer profiles and contributions
- **Project Documentation**: Historical development team information
- **User Access Control**: Login required for team information access

---

### 5. dashboard(request)
**Purpose**: Main dashboard controller that provides personalized user interface based on roles and designations.

**Business Logic**:
```python
@login_required(login_url=LOGIN_URL)
def dashboard(request):
    """
    CORE LOGIC: Role-based dashboard with personalized content and navigation
    
    HOW IT WORKS:
    1. Retrieves user's designations and roles from HoldsDesignation model
    2. Analyzes user type (student, faculty, staff) and special roles
    3. Gathers relevant notifications and module access rights
    4. Determines appropriate dashboard template and context
    5. Provides role-specific navigation and content
    
    BUSINESS PURPOSE:
    - Personalized user experience based on institutional roles
    - Efficient access to relevant modules and functions
    - Centralized notification and task management
    - Role-based interface customization and access control
    """
    user = request.user
    notifs = request.user.notifications.all()
    
    # User designation analysis
    desig = list(HoldsDesignation.objects.select_related(
        'user', 'working', 'designation'
    ).filter(working=request.user).values_list('designation'))
    
    b = [i for sub in desig for i in sub]
    design = HoldsDesignation.objects.select_related(
        'user', 'designation'
    ).filter(working=request.user)
    
    # Build designation list for role-based logic
    designation = []
    for i in design:
        designation.append(str(i.designation))
    
    roll_ = []
    for i in b:
        name_ = get_object_or_404(Designation, id=i)
        roll_.append(str(name_.name))
    
    # Hostel management role detection
    hall_caretakers = HallCaretaker.objects.all().select_related()
    hall_wardens = HallWarden.objects.all().select_related()
    
    hall_caretaker_user = [caretaker.staff.id.user for caretaker in hall_caretakers]
    hall_warden_user = [warden.faculty.id.user for warden in hall_wardens]
    
    # Dashboard context preparation
    context = {
        'notifications': notifs,
        'Curr_desig': roll_,
        'club_details': coordinator_club(request),
        'designation': designation,
        'hall_caretaker': hall_caretaker_user,
        'hall_warden': hall_warden_user,
    }
    
    # Role-based dashboard routing
    if request.user.get_username() == 'director':
        return render(request, "dashboard/director_dashboard2.html", {})
    elif "dean_rspc" in designation:
        return render(request, "dashboard/dashboard.html", context)
    elif user.extrainfo.user_type != "student":
        designat = HoldsDesignation.objects.select_related().filter(user=user)
        response = {'designat': designat}
        context.update(response)
        return render(request, "dashboard/dashboard.html", context)
    else:
        return render(request, "dashboard/dashboard.html", context)
```

**Key Features**:
- **Role-Based Routing**: Different dashboards for different user types
- **Designation Management**: Comprehensive role and responsibility tracking
- **Notification Integration**: Centralized notification display
- **Module Access**: Integration with module access rights system
- **Hostel Integration**: Special handling for hostel management roles

---

### 6. profile(request, username=None)
**Purpose**: Comprehensive user profile management with role-based routing and extensive profile editing capabilities.

**Business Logic**:
```python
@login_required(login_url=LOGIN_URL)
def profile(request, username=None):
    """
    CORE LOGIC: Comprehensive profile management with role-based routing
    
    HOW IT WORKS:
    1. Determines target user (provided username or current user)
    2. Retrieves user profile and checks edit permissions
    3. Routes faculty to EIS module, academics to AIMS module
    4. Handles extensive profile editing for students
    5. Manages placement-related profile information
    6. Processes form submissions for all profile sections
    
    BUSINESS PURPOSE:
    - Centralized user profile management across modules
    - Role-appropriate profile interfaces and capabilities
    - Comprehensive student portfolio management
    - Integration with placement and academic systems
    """
    # User resolution and permission checking
    user = get_object_or_404(User, Q(username=username)) if username else request.user
    editable = request.user == user
    profile = get_object_or_404(ExtraInfo, Q(user=user))
    
    # Role-based routing logic
    if str(user.extrainfo.user_type) == 'faculty':
        return HttpResponseRedirect('/eis/profile/' + (username if username else ''))
    
    if str(user.extrainfo.department) == 'department: Academics':
        return HttpResponseRedirect('/aims')
    
    # Student designation normalization
    array = [
        "student", "CC convenor", "Mechatronic convenor", "mess_committee",
        "mess_convener", "alumini", "Electrical_AE", "Electrical_JE",
        "Civil_AE", "Civil_JE", "co-ordinator", "co co-ordinator",
        "Convenor", "Convener", "cc1convener", "CC2 convener",
        "mess_convener_mess2", "mess_committee_mess2"
    ]
    
    # Designation processing for students
    designation_name = ""
    design = False
    current = HoldsDesignation.objects.select_related(
        'user', 'working', 'designation'
    ).filter(Q(working=user))
    
    for obj in current:
        designation_name = obj.designation.name
        if designation_name in array:
            design = True
            break
    
    # Normalize student designations
    if design:
        current = HoldsDesignation.objects.filter(
            working=user, designation__name=designation_name
        )
        for obj in current:
            obj.designation.name = obj.designation.name.replace(designation_name, 'student')
    
    if current:
        student = get_object_or_404(Student, Q(id=profile.id))
        
        # Extensive form processing for profile updates
        if editable and request.method == 'POST':
            # Placement status management
            if 'studentapprovesubmit' in request.POST:
                PlacementStatus.objects.select_related(
                    'notify_id', 'unique_id__id__user', 'unique_id__id__department'
                ).filter(pk=request.POST['studentapprovesubmit']).update(
                    invitation='ACCEPTED', timestamp=timezone.now()
                )
            
            # Education management
            if 'educationsubmit' in request.POST:
                form = AddEducation(request.POST)
                if form.is_valid():
                    Education.objects.create(
                        unique_id=student,
                        degree=form.cleaned_data['degree'],
                        grade=form.cleaned_data['grade'],
                        institute=form.cleaned_data['institute'],
                        stream=form.cleaned_data['stream'],
                        sdate=form.cleaned_data['sdate'],
                        edate=form.cleaned_data['edate']
                    )
            
            # Profile information updates
            if 'profilesubmit' in request.POST:
                extrainfo_obj = ExtraInfo.objects.select_related(
                    'user', 'department'
                ).get(user=user)
                extrainfo_obj.about_me = request.POST.get('about')
                extrainfo_obj.date_of_birth = request.POST.get('age')
                extrainfo_obj.address = request.POST.get('address')
                extrainfo_obj.phone_no = request.POST.get('contact')
                extrainfo_obj.save()
                profile = get_object_or_404(ExtraInfo, Q(user=user))
            
            # Skills management
            if 'skillsubmit' in request.POST:
                form = AddSkill(request.POST)
                if form.is_valid():
                    skill = form.cleaned_data['skill']
                    skill_rating = form.cleaned_data['skill_rating']
                    
                    try:
                        skill_id = Skill.objects.get(skill=skill)
                    except Exception:
                        skill_id = Skill.objects.create(skill=skill)
                        skill_id.save()
                    
                    Has.objects.create(
                        unique_id=student,
                        skill_id=skill_id,
                        skill_rating=skill_rating
                    )
            
            # Achievement, publication, patent, course, project, experience management
            # (Similar form processing for each category)
            
            # Deletion operations for profile sections
            if 'deleteskill' in request.POST:
                hid = request.POST['deleteskill']
                hs = Has.objects.select_related(
                    'skill_id', 'unique_id__id__user', 'unique_id__id__department'
                ).get(Q(pk=hid))
                hs.delete()
            
            # (Similar deletion handlers for other profile sections)
        
        # Form initialization and data retrieval
        forms = {
            'education_form': AddEducation(initial={}),
            'profile_form': AddProfile(initial={}),
            'skill_form': AddSkill(initial={}),
            'course_form': AddCourse(initial={}),
            'achievement_form': AddAchievement(initial={}),
            'publication_form': AddPublication(initial={}),
            'project_form': AddProject(initial={}),
            'patent_form': AddPatent(initial={}),
            'experience_form': AddExperience(initial={}),
        }
        
        # Profile data retrieval
        profile_data = {
            'skills': Has.objects.select_related(
                'skill_id', 'unique_id__id__user', 'unique_id__id__department'
            ).filter(Q(unique_id=student)),
            'education': Education.objects.select_related(
                'unique_id__id__user', 'unique_id__id__department'
            ).filter(Q(unique_id=student)),
            'courses': Course.objects.select_related(
                'unique_id__id__user', 'unique_id__id__department'
            ).filter(Q(unique_id=student)),
            'experiences': Experience.objects.select_related(
                'unique_id__id__user', 'unique_id__id__department'
            ).filter(Q(unique_id=student)),
            'projects': Project.objects.select_related(
                'unique_id__id__user', 'unique_id__id__department'
            ).filter(Q(unique_id=student)),
            'achievements': Achievement.objects.select_related(
                'unique_id__id__user', 'unique_id__id__department'
            ).filter(Q(unique_id=student)),
            'publications': Publication.objects.select_related(
                'unique_id__id__user', 'unique_id__id__department'
            ).filter(Q(unique_id=student)),
            'patents': Patent.objects.select_related(
                'unique_id__id__user', 'unique_id__id__department'
            ).filter(Q(unique_id=student)),
        }
        
        # Context preparation and template routing
        context = {
            'user': user,
            'profile': profile,
            'current': current,
            'editable': editable,
            **profile_data,
            **forms
        }
        
        # Smart template routing based on form submissions
        if 'skillsubmit' in request.POST or 'deleteskill' in request.POST:
            return render(request, "globals/student_profile2.html", context)
        if ('coursesubmit' in request.POST or 'educationsubmit' in request.POST or 
            'deleteedu' in request.POST or 'deletecourse' in request.POST):
            return render(request, "globals/student_profile3.html", context)
        # (Additional template routing logic for other profile sections)
        
        return render(request, "globals/student_profile.html", context)
```

**Key Features**:
- **Role-Based Routing**: Automatic routing to appropriate profile modules
- **Comprehensive Profile Management**: Education, skills, achievements, projects, etc.
- **Placement Integration**: Direct integration with placement system
- **Form Processing**: Extensive form handling for all profile sections
- **Permission Management**: Edit access control and validation
- **Template Routing**: Smart template selection based on user actions

---

This concludes Part 6 covering the main view functions. The profile function is particularly complex, handling comprehensive student portfolio management with integration across multiple modules.
