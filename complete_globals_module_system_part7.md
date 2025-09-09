# Complete Globals Module System Documentation - Fusion IIIT (Part 7)

## API Architecture and Implementation

### Overview
The globals module provides a comprehensive REST API that handles authentication, user management, profile operations, and system-wide functionality for mobile and web applications.

---

## View Functions (Continued from Part 6)

### 7. logout_view(request)
**Purpose**: Handles user logout and session cleanup.

**Business Logic**:
```python
@login_required(login_url=LOGIN_URL)
def logout_view(request):
    """
    CORE LOGIC: Clean user logout with session management
    
    HOW IT WORKS:
    1. Calls Django's logout function to clear session
    2. Invalidates authentication tokens and cookies
    3. Redirects to homepage for clean state
    
    BUSINESS PURPOSE:
    - Secure session termination
    - Clean user state management
    - Security compliance for logout procedures
    """
    logout(request)
    return redirect("/")
```

### 8. feedback(request)
**Purpose**: Comprehensive feedback management system with rating capabilities.

**Business Logic**:
```python
@login_required(login_url=LOGIN_URL)
def feedback(request):
    """
    CORE LOGIC: Feedback collection and rating system with analytics
    
    HOW IT WORKS:
    1. Retrieves and displays top-rated feedback from other users
    2. Handles new feedback submission with star ratings
    3. Calculates system-wide average rating
    4. Manages user's existing feedback updates
    5. Provides visual star rating display
    
    BUSINESS PURPOSE:
    - System improvement through user feedback
    - Quality monitoring and rating analytics
    - User engagement and satisfaction tracking
    - Community-driven improvement insights
    """
    # Display top feedback from other users (excluding current user)
    feeds = Feedback.objects.select_related('user').all().order_by("rating").exclude(user=request.user)
    if feeds.count() > 5:
        feeds = feeds[:5]
    
    # Prepare star ratings for display
    rated = [range(feed.rating) for feed in feeds]
    feeds = zip(feeds, rated)
    
    if request.method == "POST":
        # Handle feedback submission/update
        try:
            feedback = Feedback.objects.select_related('user').get(user=request.user)
        except Exception:
            feedback = None
        
        # Form processing
        form = WebFeedbackForm(request.POST, instance=feedback if feedback else None)
        feedback = form.save(commit=False)
        
        # Rating validation and assignment
        user_rating = request.POST.get("rating")
        feedback.user = request.user
        if int(user_rating) > 0 and int(user_rating) < 6:
            feedback.rating = user_rating
            feedback.save()
        
        # Calculate system average rating
        rating = sum(feed.rating for feed in Feedback.objects.all())
        if Feedback.objects.all().count() > 0:
            rating = round(rating / Feedback.objects.all().count(), 1)
        
        # Prepare response context
        stars = list(range(int(feedback.rating)))
        context = {
            'form': WebFeedbackForm(instance=feedback),
            "feedback": feedback,
            'rating': rating,
            "stars": stars,
            "reviewed": True,
            "feeds": feeds
        }
        return render(request, "globals/feedback.html", context)
    
    # GET request handling
    rating = sum(feed.rating for feed in Feedback.objects.all())
    if Feedback.objects.all().count() > 0:
        rating = round(rating / Feedback.objects.all().count(), 1)
    
    try:
        feedback = Feedback.objects.select_related('user').get(user=request.user)
        form = WebFeedbackForm(instance=feedback)
        stars = list(range(int(feedback.rating)))
        context = {
            "rating": rating,
            "feedback": feedback,
            "stars": stars,
            "form": form,
            "reviewed": True,
            "feeds": feeds
        }
    except Exception:
        form = WebFeedbackForm()
        context = {"form": form, "rating": rating, "feeds": feeds}
    
    return render(request, "globals/feedback.html", context)
```

### 9. issue(request)
**Purpose**: Issue reporting and tracking system with image attachment support.

**Business Logic**:
```python
@login_required(login_url=LOGIN_URL)
def issue(request):
    """
    CORE LOGIC: Comprehensive issue reporting and management system
    
    HOW IT WORKS:
    1. Handles new issue creation with form validation
    2. Processes multiple image attachments with validation
    3. Categorizes issues as open/closed for display
    4. Provides issue tracking and status management
    
    BUSINESS PURPOSE:
    - System maintenance and bug tracking
    - User-reported problem resolution
    - Visual documentation of issues
    - Priority-based issue management
    """
    if request.method == "POST":
        form = IssueForm(request.POST)
        if form.is_valid():
            issue = form.save(commit=False)
            issue.user = request.user
            issue.save()
            
            # Process image attachments with validation
            for image in request.FILES.getlist('images'):
                try:
                    Image.open(image)  # Validate image format
                    image_obj = IssueImage.objects.create(image=image, user=request.user)
                    issue.images.add(image_obj)
                except Exception:
                    pass  # Skip invalid images
            
            issue.save()
    
    # Retrieve and categorize issues
    openissue = Issue.objects.select_related('user').prefetch_related('images', 'support').filter(closed=False)
    closedissue = Issue.objects.select_related('user').prefetch_related('images', 'support').filter(closed=True)
    
    form = IssueForm()
    context = {
        "form": form,
        "openissue": openissue,
        "closedissue": closedissue,
    }
    return render(request, "globals/issue.html", context)
```

### 10. view_issue(request, id) & support_issue(request, id)
**Purpose**: Individual issue management and community support features.

### 11. search(request)
**Purpose**: Advanced user search with intelligent name matching.

**Business Logic**:
```python
@login_required(login_url=LOGIN_URL)
def search(request):
    """
    CORE LOGIC: Intelligent user search with multi-word matching
    
    Algorithm:
    - Searches first 15 users whose first/last name starts with query words
    - All words in query must be matched
    - Supports partial name matching (e.g., 'Atu Gu' matches 'Atul Gupta')
    
    HOW IT WORKS:
    1. Validates minimum query length (3 characters)
    2. Tokenizes search query into individual words
    3. Builds database query with AND logic for all tokens
    4. Returns first 15 matching users
    
    BUSINESS PURPOSE:
    - Directory service for institutional users
    - Contact discovery and networking
    - Administrative user lookup
    - Communication facilitation
    """
    key = request.GET['q']
    
    # Minimum query length validation
    if len(key) < 3:
        return render(request, "globals/search.html", {'sresults': ()})
    
    # Multi-word search logic
    words = (w.strip() for w in key.split())
    name_q = Q()
    for token in words:
        name_q = name_q & (Q(first_name__icontains=token) | Q(last_name__icontains=token))
    
    # Execute search with limit
    search_results = User.objects.filter(name_q)[:15]
    
    context = {'sresults': search_results if search_results else []}
    return render(request, "globals/search.html", context)
```

### 12. update_global_variable(request)
**Purpose**: Session management for role switching and module access rights.

**Business Logic**:
```python
@login_required(login_url=LOGIN_URL)
def update_global_variable(request):
    """
    CORE LOGIC: Dynamic role switching with module access management
    
    HOW IT WORKS:
    1. Captures selected designation from user interface
    2. Updates session with current designation selection
    3. Retrieves corresponding module access rights
    4. Stores access permissions in session for quick lookup
    5. Redirects to dashboard with updated permissions
    
    BUSINESS PURPOSE:
    - Multi-role user experience management
    - Dynamic permission enforcement
    - Session-based access control
    - Performance optimization for permission checks
    """
    if request.method == 'POST':
        selected_option = request.POST.get('dropdown')
        
        # Update session with selected designation
        request.session['currentDesignationSelected'] = selected_option
        
        # Retrieve module access rights for the designation
        module_access = ModuleAccess.objects.filter(designation=selected_option).first()
        
        if module_access:
            access_rights = {}
            field_names = [
                field.name for field in ModuleAccess._meta.get_fields() 
                if field.name not in ['id', 'designation']
            ]
            
            for field_name in field_names:
                access_rights[field_name] = getattr(module_access, field_name)
        
        # Store access rights in session
        request.session['moduleAccessRights'] = access_rights
        
        return HttpResponseRedirect('/dashboard')
    
    return HttpResponseRedirect(reverse('home'))
```

---

## REST API Implementation

### API Authentication and User Management

### 1. login(request)
**Purpose**: Token-based authentication with role-aware login response.

**Business Logic**:
```python
@api_view(['POST'])
@permission_classes([AllowAny])
def login(request):
    """
    CORE LOGIC: Comprehensive authentication with designation management
    
    HOW IT WORKS:
    1. Validates user credentials using custom serializer
    2. Authenticates user and generates authentication token
    3. Retrieves all user designations and roles
    4. Builds comprehensive response with user context
    5. Returns token and role information for client applications
    
    BUSINESS PURPOSE:
    - Mobile and web application authentication
    - Role-based application initialization
    - Token management for API access
    - Multi-designation user support
    """
    serializer = serializers.UserLoginSerializer(data=request.data)
    serializer.is_valid(raise_exception=True)
    user = get_and_authenticate_user(**serializer.validated_data)
    data = serializers.AuthUserSerializer(user).data
    
    # Retrieve user designations
    desig = list(HoldsDesignation.objects.select_related(
        'user', 'working', 'designation'
    ).filter(working=user).values_list('designation'))
    
    b = [i for sub in desig for i in sub]
    design = HoldsDesignation.objects.select_related(
        'user', 'designation'
    ).filter(working=user)
    
    # Build designation list
    designation = []
    if str(user.extrainfo.user_type) == "student":
        designation.append(str(user.extrainfo.user_type))
    
    for i in design:
        if str(i.designation) != str(user.extrainfo.user_type):
            designation.append(str(i.designation))
    
    resp = {
        'success': 'True',
        'message': 'User logged in successfully',
        'token': data['auth_token'],
        'designations': designation
    }
    return Response(data=resp, status=status.HTTP_200_OK)
```

### 2. logout(request)
**Purpose**: Secure token invalidation for API logout.

### 3. auth_view(request)
**Purpose**: User authorization information with module access details.

**Business Logic**:
```python
@api_view(['GET'])
@permission_classes([AllowAny])
def auth_view(request):
    """
    CORE LOGIC: Comprehensive user authorization and module access information
    
    HOW IT WORKS:
    1. Retrieves user's basic information and profile
    2. Gets all designations held by the user
    3. Calculates accessible modules for each designation
    4. Returns comprehensive authorization context
    5. Includes last selected role for state management
    
    BUSINESS PURPOSE:
    - Client application initialization
    - Role-based UI configuration
    - Module access verification
    - User preference persistence
    """
    user = request.user
    name = request.user.first_name + "_" + request.user.last_name
    roll_no = request.user.username
    
    extra_info = get_object_or_404(ExtraInfo, user=user)
    last_selected_role = extra_info.last_selected_role
    
    # Get all user designations
    designation_list = list(HoldsDesignation.objects.filter(
        working=request.user
    ).values_list('designation'))
    
    designation_id = [designation for designations in designation_list for designation in designations]
    designation_info = []
    
    for id in designation_id:
        name_ = get_object_or_404(Designation, id=id)
        designation_info.append(str(name_.name))
    
    # Calculate accessible modules for each designation
    accessible_modules = {}
    for designation in designation_info:
        module_access = ModuleAccess.objects.filter(
            designation__iexact=designation
        ).first()
        
        if module_access:
            filtered_modules = {}
            field_names = [
                field.name for field in ModuleAccess._meta.get_fields()
                if field.name not in ['id', 'designation']
            ]
            
            for field_name in field_names:
                filtered_modules[field_name] = getattr(module_access, field_name)
            
            accessible_modules[designation] = filtered_modules
    
    resp = {
        'designation_info': designation_info,
        'name': name,
        'roll_no': roll_no,
        'accessible_modules': accessible_modules,
        'last_selected_role': last_selected_role
    }
    
    return Response(data=resp, status=status.HTTP_200_OK)
```

### 4. notification(request)
**Purpose**: User notification retrieval for mobile applications.

### 5. update_last_selected_role(request)
**Purpose**: Role preference management for user experience.

### 6. profile(request, username=None)
**Purpose**: Comprehensive user profile API with student portfolio data.

**Business Logic**:
```python
@api_view(['GET'])
def profile(request, username=None):
    """
    CORE LOGIC: Comprehensive student profile with portfolio information
    
    HOW IT WORKS:
    1. Resolves target user (provided username or current user)
    2. Serializes basic profile information
    3. For students, retrieves comprehensive portfolio data
    4. Includes skills, education, projects, achievements, etc.
    5. Returns structured profile response
    
    BUSINESS PURPOSE:
    - Mobile application profile display
    - Portfolio management API
    - Student information system integration
    - Placement and academic data access
    """
    user = get_object_or_404(User, username=username) if username else request.user
    profile = serializers.ExtraInfoSerializer(user.extrainfo).data
    
    if profile['user_type'] == 'student':
        student = user.extrainfo.student
        std_sem = Student.objects.get(id=student.id).curr_semester_no
        
        # Skills with ratings
        skills = list(
            Has.objects.filter(unique_id_id=student)
            .select_related("skill_id")
            .values("skill_id__skill", "skill_rating")
        )
        formatted_skills = [
            {"skill_name": skill["skill_id__skill"], "skill_rating": skill["skill_rating"]}
            for skill in skills
        ]
        
        # Portfolio data serialization
        education = serializers.EducationSerializer(student.education_set.all(), many=True).data
        course = serializers.CourseSerializer(student.course_set.all(), many=True).data
        experience = serializers.ExperienceSerializer(student.experience_set.all(), many=True).data
        project = serializers.ProjectSerializer(student.project_set.all(), many=True).data
        achievement = serializers.AchievementSerializer(student.achievement_set.all(), many=True).data
        publication = serializers.PublicationSerializer(student.publication_set.all(), many=True).data
        patent = serializers.PatentSerializer(student.patent_set.all(), many=True).data
        current = serializers.HoldsDesignationSerializer(user.current_designation.all(), many=True).data
        
        resp = {
            'profile': profile,
            'semester_no': std_sem,
            'skills': formatted_skills,
            'education': education,
            'course': course,
            'experience': experience,
            'project': project,
            'achievement': achievement,
            'publication': publication,
            'patent': patent,
            'current': current
        }
        return Response(data=resp, status=status.HTTP_200_OK)
    else:
        return Response(data={'error': 'User is not a student'}, status=status.HTTP_400_BAD_REQUEST)
```

### 7. profile_update(request)
**Purpose**: API endpoint for updating various profile sections.

---

## API Utilities and Serializers

### Serializers Structure
The API uses Django REST Framework serializers for data validation and transformation:

- **UserLoginSerializer**: Handles authentication credentials
- **AuthUserSerializer**: User authentication response data
- **ExtraInfoSerializer**: User profile information
- **NotificationSerializer**: User notifications
- **EducationSerializer**: Educational background
- **CourseSerializer**: Course certifications
- **SkillSerializer**: Skills and ratings
- **ExperienceSerializer**: Professional experience
- **ProjectSerializer**: Project portfolio
- **AchievementSerializer**: Awards and achievements
- **PublicationSerializer**: Research publications
- **PatentSerializer**: Patent information
- **HoldsDesignationSerializer**: Role and designation data

### Authentication and Security
- **Token-based authentication** for mobile applications
- **Permission classes** for access control
- **Role-based API access** with designation verification
- **Session management** for web applications
- **Rate limiting** for password reset endpoints

### Integration Patterns
- **Cross-module integration** with EIS, Placement, Academic modules
- **Notification system** integration
- **File upload handling** for images and documents
- **Database optimization** with select_related and prefetch_related
- **Error handling** with appropriate HTTP status codes

---

This concludes Part 7 covering the API architecture and implementation. The next part will cover forms, admin interface, and utility components.
