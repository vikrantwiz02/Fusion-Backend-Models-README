# Complete Globals Module System Documentation - Fusion IIIT (Part 2)

## Database Models Analysis with Business Logic (Continued)

### 4. Designation Model
**Purpose**: Defines all available roles and positions within the institute's organizational hierarchy, supporting both academic and administrative designations.

**PostgreSQL Table**: `globals_designation`

**Fields Structure**:
```python
class Designation(models.Model):
    name = models.CharField(max_length=50, unique=True, blank=False, default='student')
    full_name = models.CharField(max_length=100, default='Computer Science and Engineering')
    type = models.CharField(max_length=30, default='academic', choices=Constants.DESIGNATIONS)
    
    def __str__(self):
        return self.name
```

**Enhanced Business Methods**:
```python
def get_designation_permissions(self):
    """
    CORE LOGIC: Retrieve all permissions and capabilities associated with this designation
    
    HOW IT WORKS:
    1. Queries ModuleAccess table for designation-specific permissions
    2. Aggregates all module access rights for this role
    3. Calculates permission hierarchy and inheritance
    4. Returns comprehensive permission mapping
    
    BUSINESS PURPOSE:
    - Role-based access control implementation
    - Dynamic permission assignment
    - Administrative control over feature access
    - Security policy enforcement
    """
    try:
        module_access = ModuleAccess.objects.get(designation=self.name)
        
        permissions = {
            'administrative': {
                'program_and_curriculum': module_access.program_and_curriculum,
                'course_registration': module_access.course_registration,
                'course_management': module_access.course_management,
                'other_academics': module_access.other_academics,
                'examinations': module_access.examinations,
                'hr': module_access.hr,
                'department': module_access.department,
            },
            'operational': {
                'iwd': module_access.iwd,
                'complaint_management': module_access.complaint_management,
                'fts': module_access.fts,
                'purchase_and_store': module_access.purchase_and_store,
                'hostel_management': module_access.hostel_management,
                'mess_management': module_access.mess_management,
            },
            'specialized': {
                'spacs': module_access.spacs,
                'rspc': module_access.rspc,
                'gymkhana': module_access.gymkhana,
                'placement_cell': module_access.placement_cell,
                'visitor_hostel': module_access.visitor_hostel,
                'phc': module_access.phc,
            }
        }
        
        # Calculate permission summary
        total_permissions = sum(
            sum(category.values()) for category in permissions.values()
        )
        
        return {
            'permissions': permissions,
            'total_count': total_permissions,
            'permission_level': self._calculate_permission_level(total_permissions),
            'has_admin_access': total_permissions > 10
        }
        
    except ModuleAccess.DoesNotExist:
        return {
            'permissions': {},
            'total_count': 0,
            'permission_level': 'basic',
            'has_admin_access': False
        }

def _calculate_permission_level(self, total_permissions):
    """Helper method to categorize permission levels"""
    if total_permissions >= 15:
        return 'super_admin'
    elif total_permissions >= 10:
        return 'admin'
    elif total_permissions >= 5:
        return 'moderator'
    elif total_permissions >= 1:
        return 'limited'
    else:
        return 'basic'

def get_current_holders(self):
    """
    CORE LOGIC: Retrieve all users currently holding this designation
    
    HOW IT WORKS:
    1. Queries HoldsDesignation for active role assignments
    2. Differentiates between permanent and working assignments
    3. Provides holder statistics and activity metrics
    
    BUSINESS PURPOSE:
    - Role assignment tracking and management
    - Organizational chart generation
    - Succession planning and role transitions
    - Administrative oversight of positions
    """
    current_assignments = HoldsDesignation.objects.filter(
        designation=self
    ).select_related('user', 'working')
    
    holders_info = {
        'permanent_holders': [],
        'working_holders': [],
        'total_assignments': current_assignments.count(),
        'active_count': 0
    }
    
    for assignment in current_assignments:
        permanent_info = {
            'user': assignment.user,
            'user_id': assignment.user.extrainfo.id,
            'department': assignment.user.extrainfo.department.name if assignment.user.extrainfo.department else 'None',
            'held_since': assignment.held_at,
            'is_active': assignment.user.extrainfo.user_status == 'PRESENT'
        }
        
        working_info = {
            'user': assignment.working,
            'user_id': assignment.working.extrainfo.id,
            'department': assignment.working.extrainfo.department.name if assignment.working.extrainfo.department else 'None',
            'held_since': assignment.held_at,
            'is_active': assignment.working.extrainfo.user_status == 'PRESENT'
        }
        
        holders_info['permanent_holders'].append(permanent_info)
        holders_info['working_holders'].append(working_info)
        
        if working_info['is_active']:
            holders_info['active_count'] += 1
    
    return holders_info

def get_designation_hierarchy_level(self):
    """
    CORE LOGIC: Determine hierarchical level and authority scope of designation
    
    HOW IT WORKS:
    1. Analyzes designation name patterns for hierarchy indicators
    2. Maps designation types to organizational levels
    3. Calculates authority scope and reporting relationships
    
    BUSINESS PURPOSE:
    - Workflow routing and approval chains
    - Authority validation for administrative actions
    - Organizational structure visualization
    - Permission inheritance modeling
    """
    hierarchy_keywords = {
        'director': 10,
        'dean': 9,
        'registrar': 8,
        'head': 7,
        'chairman': 7,
        'coordinator': 6,
        'convener': 5,
        'member': 4,
        'assistant': 3,
        'clerk': 2,
        'student': 1
    }
    
    level = 1  # Default level
    authority_scope = 'limited'
    
    name_lower = self.name.lower()
    for keyword, keyword_level in hierarchy_keywords.items():
        if keyword in name_lower:
            level = max(level, keyword_level)
    
    # Determine authority scope
    if level >= 8:
        authority_scope = 'institute_wide'
    elif level >= 6:
        authority_scope = 'department_wide'
    elif level >= 4:
        authority_scope = 'committee_level'
    else:
        authority_scope = 'limited'
    
    return {
        'level': level,
        'authority_scope': authority_scope,
        'can_approve_workflows': level >= 5,
        'has_administrative_power': level >= 6,
        'reporting_level': self._get_reporting_level(level)
    }

def _get_reporting_level(self, level):
    """Helper method to determine reporting relationships"""
    if level >= 9:
        return 'reports_to_board'
    elif level >= 7:
        return 'reports_to_dean'
    elif level >= 5:
        return 'reports_to_head'
    else:
        return 'reports_to_coordinator'
```

**Key Business Logic**:
- **Role Definition**: Centralized definition of all organizational positions
- **Permission Mapping**: Links roles to specific system capabilities
- **Hierarchy Management**: Supports organizational structure and authority levels
- **Assignment Tracking**: Monitors who holds which positions

---

### 5. HoldsDesignation Model
**Purpose**: Manages the assignment of designations to users, supporting both permanent assignments and temporary working arrangements.

**PostgreSQL Table**: `globals_holdsdesignation`

**Fields Structure**:
```python
class HoldsDesignation(models.Model):
    user = models.ForeignKey(User, related_name='holds_designations', on_delete=models.CASCADE)
    working = models.ForeignKey(User, related_name='current_designation', on_delete=models.CASCADE)
    designation = models.ForeignKey(Designation, related_name='designees', on_delete=models.CASCADE)
    held_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        unique_together = [['user', 'designation'], ['working', 'designation']]
    
    def __str__(self):
        return '{} - {}'.format(self.user.username, self.designation)
```

**Enhanced Business Methods**:
```python
def get_assignment_duration(self):
    """
    CORE LOGIC: Calculate how long this designation has been held
    
    HOW IT WORKS:
    1. Computes time difference from assignment date to current time
    2. Provides duration in multiple formats for different use cases
    3. Identifies long-term vs temporary assignments
    
    BUSINESS PURPOSE:
    - Assignment history tracking
    - Role rotation planning
    - Experience assessment for users
    - Administrative oversight of long-term positions
    """
    duration = timezone.now() - self.held_at
    
    days = duration.days
    hours = duration.seconds // 3600
    
    return {
        'total_days': days,
        'years': days // 365,
        'months': (days % 365) // 30,
        'remaining_days': (days % 365) % 30,
        'total_hours': (days * 24) + hours,
        'is_long_term': days > 365,
        'is_recent': days < 30,
        'formatted_duration': self._format_duration(days)
    }

def _format_duration(self, days):
    """Helper method to format duration in human-readable format"""
    if days < 1:
        return "Less than a day"
    elif days < 30:
        return f"{days} days"
    elif days < 365:
        months = days // 30
        return f"{months} month{'s' if months != 1 else ''}"
    else:
        years = days // 365
        remaining_months = (days % 365) // 30
        if remaining_months > 0:
            return f"{years} year{'s' if years != 1 else ''}, {remaining_months} month{'s' if remaining_months != 1 else ''}"
        else:
            return f"{years} year{'s' if years != 1 else ''}"

def validate_assignment_conflict(self):
    """
    CORE LOGIC: Check for conflicts in designation assignments
    
    HOW IT WORKS:
    1. Validates that designation isn't already assigned to someone else
    2. Checks for incompatible role combinations
    3. Ensures user eligibility for the designation
    4. Validates department and hierarchy requirements
    
    BUSINESS PURPOSE:
    - Prevents duplicate role assignments
    - Ensures organizational hierarchy integrity
    - Validates role compatibility and prerequisites
    - Maintains data consistency in role management
    """
    conflicts = {
        'has_conflicts': False,
        'conflict_types': [],
        'conflicting_assignments': [],
        'recommendations': []
    }
    
    # Check for existing assignments of this designation
    existing_assignments = HoldsDesignation.objects.filter(
        designation=self.designation
    ).exclude(id=self.id if self.id else None)
    
    if existing_assignments.exists():
        conflicts['has_conflicts'] = True
        conflicts['conflict_types'].append('duplicate_assignment')
        conflicts['conflicting_assignments'] = list(existing_assignments)
        conflicts['recommendations'].append('Review existing assignments before proceeding')
    
    # Check for incompatible role combinations for the working user
    user_roles = HoldsDesignation.objects.filter(
        working=self.working
    ).exclude(id=self.id if self.id else None)
    
    incompatible_combinations = self._check_role_compatibility(user_roles)
    if incompatible_combinations:
        conflicts['has_conflicts'] = True
        conflicts['conflict_types'].append('incompatible_roles')
        conflicts['recommendations'].extend(incompatible_combinations)
    
    # Check department eligibility
    if self.designation.type == 'academic':
        if self.working.extrainfo.user_type not in ['faculty', 'staff']:
            conflicts['has_conflicts'] = True
            conflicts['conflict_types'].append('eligibility_mismatch')
            conflicts['recommendations'].append('Academic designations require faculty or staff status')
    
    return conflicts

def _check_role_compatibility(self, existing_roles):
    """Helper method to check for incompatible role combinations"""
    incompatible_patterns = [
        # Students shouldn't hold administrative positions
        ('student', 'administrative'),
        # Faculty shouldn't hold certain staff positions
        ('faculty', 'clerk'),
        # Specific role conflicts
        ('dean', 'head'),  # Can't be both dean and department head
    ]
    
    recommendations = []
    current_type = self.working.extrainfo.user_type
    current_designation_type = self.designation.type
    
    for user_type, designation_type in incompatible_patterns:
        if current_type == user_type and current_designation_type == designation_type:
            recommendations.append(f"{user_type} users typically don't hold {designation_type} designations")
    
    return recommendations

def get_delegation_chain(self):
    """
    CORE LOGIC: Retrieve delegation and authority chain for this assignment
    
    HOW IT WORKS:
    1. Identifies if this is a delegated assignment (user != working)
    2. Maps the delegation chain and authority flow
    3. Tracks temporary vs permanent authority transfers
    
    BUSINESS PURPOSE:
    - Authority tracking for administrative actions
    - Delegation management and oversight
    - Audit trail for role-based decisions
    - Temporary assignment management
    """
    is_delegated = self.user != self.working
    
    delegation_info = {
        'is_delegated': is_delegated,
        'permanent_holder': self.user,
        'current_working': self.working,
        'delegation_type': 'temporary' if is_delegated else 'permanent',
        'authority_source': 'delegated' if is_delegated else 'direct'
    }
    
    if is_delegated:
        # Check if there are multiple levels of delegation
        related_delegations = HoldsDesignation.objects.filter(
            designation=self.designation,
            user=self.working
        ).exclude(id=self.id if self.id else None)
        
        delegation_info['delegation_chain'] = self._build_delegation_chain(related_delegations)
        delegation_info['requires_approval'] = True
        delegation_info['delegation_duration'] = self.get_assignment_duration()
    
    return delegation_info

def _build_delegation_chain(self, related_delegations):
    """Helper method to build complete delegation chain"""
    chain = []
    for delegation in related_delegations:
        chain.append({
            'from': delegation.user,
            'to': delegation.working,
            'date': delegation.held_at,
            'designation': delegation.designation.name
        })
    return chain

def calculate_role_authority_score(self):
    """
    CORE LOGIC: Calculate numerical authority score for this role assignment
    
    HOW IT WORKS:
    1. Combines designation hierarchy level with duration
    2. Factors in delegation status and approval chains
    3. Considers department scope and module access
    4. Returns normalized authority score
    
    BUSINESS PURPOSE:
    - Priority determination in workflow systems
    - Authority validation for administrative actions
    - Role-based decision making support
    - Conflict resolution in approval processes
    """
    designation_hierarchy = self.designation.get_designation_hierarchy_level()
    assignment_duration = self.get_assignment_duration()
    delegation_info = self.get_delegation_chain()
    
    # Base score from designation level
    base_score = designation_hierarchy['level'] * 10
    
    # Duration bonus (experience factor)
    duration_bonus = min(assignment_duration['total_days'] / 365 * 5, 20)  # Max 20 points for 4+ years
    
    # Delegation penalty
    delegation_penalty = 5 if delegation_info['is_delegated'] else 0
    
    # Department scope multiplier
    scope_multiplier = {
        'institute_wide': 1.5,
        'department_wide': 1.2,
        'committee_level': 1.0,
        'limited': 0.8
    }.get(designation_hierarchy['authority_scope'], 1.0)
    
    # Module access bonus
    permissions = self.designation.get_designation_permissions()
    access_bonus = permissions['total_count'] * 2
    
    final_score = (base_score + duration_bonus + access_bonus - delegation_penalty) * scope_multiplier
    
    return {
        'total_score': round(final_score, 2),
        'base_score': base_score,
        'duration_bonus': round(duration_bonus, 2),
        'access_bonus': access_bonus,
        'delegation_penalty': delegation_penalty,
        'scope_multiplier': scope_multiplier,
        'authority_level': self._categorize_authority_level(final_score)
    }

def _categorize_authority_level(self, score):
    """Helper method to categorize authority level based on score"""
    if score >= 100:
        return 'supreme'
    elif score >= 80:
        return 'high'
    elif score >= 60:
        return 'moderate'
    elif score >= 40:
        return 'basic'
    else:
        return 'limited'
```

**Key Business Logic**:
- **Dual Assignment Model**: Supports both permanent holders and working delegates
- **Temporal Tracking**: Monitors assignment duration and changes
- **Conflict Prevention**: Validates role assignments and prevents conflicts
- **Authority Management**: Tracks delegation chains and authority levels

---

### 6. Staff Model
**Purpose**: Specialized user type for non-academic personnel with extended attributes for administrative and support staff.

**PostgreSQL Table**: `globals_staff`

**Fields Structure**:
```python
class Staff(models.Model):
    id = models.OneToOneField(ExtraInfo, on_delete=models.CASCADE, primary_key=True)
    
    def __str__(self):
        return str(self.id)
```

**Enhanced Business Methods**:
```python
def get_staff_profile_summary(self):
    """
    CORE LOGIC: Generate comprehensive staff profile with role and department information
    
    HOW IT WORKS:
    1. Aggregates staff-specific information from related models
    2. Combines basic profile with administrative role data
    3. Calculates staff-specific metrics and capabilities
    
    BUSINESS PURPOSE:
    - Staff directory and contact management
    - Administrative oversight and management
    - Role assignment and responsibility tracking
    - Performance and engagement monitoring
    """
    extra_info = self.id
    user = extra_info.user
    
    # Get current designations
    current_designations = HoldsDesignation.objects.filter(
        working=user
    ).select_related('designation')
    
    profile_summary = {
        'basic_info': {
            'name': f"{user.first_name} {user.last_name}",
            'employee_id': extra_info.id,
            'email': user.email,
            'phone': extra_info.phone_no,
            'department': extra_info.department.name if extra_info.department else 'Not Assigned',
            'user_status': extra_info.user_status,
            'join_date': user.date_joined
        },
        'role_information': {
            'current_designations': [d.designation.name for d in current_designations],
            'designation_types': [d.designation.type for d in current_designations],
            'total_roles': current_designations.count(),
            'has_administrative_role': any(d.designation.type == 'administrative' for d in current_designations)
        },
        'contact_details': {
            'address': extra_info.address,
            'emergency_contact': extra_info.phone_no,  # Could be extended with separate emergency contact
            'preferred_communication': 'email'  # Could be made configurable
        }
    }
    
    return profile_summary

def get_administrative_capabilities(self):
    """
    CORE LOGIC: Determine staff member's administrative capabilities and module access
    
    HOW IT WORKS:
    1. Analyzes held designations for administrative permissions
    2. Aggregates module access rights across all roles
    3. Calculates overall administrative authority level
    
    BUSINESS PURPOSE:
    - Dynamic permission assignment for system access
    - Administrative workflow routing
    - Capability-based task assignment
    - System security and access control
    """
    user = self.id.user
    designations = HoldsDesignation.objects.filter(working=user)
    
    capabilities = {
        'module_access': {},
        'administrative_level': 'basic',
        'can_manage_users': False,
        'can_approve_workflows': False,
        'financial_authority': False,
        'hr_capabilities': False
    }
    
    total_permissions = 0
    
    for designation in designations:
        designation_permissions = designation.designation.get_designation_permissions()
        total_permissions += designation_permissions['total_count']
        
        # Aggregate module access
        for category, permissions in designation_permissions['permissions'].items():
            for module, has_access in permissions.items():
                if has_access:
                    capabilities['module_access'][module] = True
    
    # Determine administrative level
    if total_permissions >= 15:
        capabilities['administrative_level'] = 'senior_administrator'
        capabilities['can_manage_users'] = True
        capabilities['can_approve_workflows'] = True
    elif total_permissions >= 10:
        capabilities['administrative_level'] = 'administrator'
        capabilities['can_approve_workflows'] = True
    elif total_permissions >= 5:
        capabilities['administrative_level'] = 'coordinator'
    
    # Check for specific capabilities
    if capabilities['module_access'].get('hr', False):
        capabilities['hr_capabilities'] = True
    
    if capabilities['module_access'].get('purchase_and_store', False):
        capabilities['financial_authority'] = True
    
    return capabilities

def get_workload_analysis(self):
    """
    CORE LOGIC: Analyze staff workload based on role assignments and responsibilities
    
    HOW IT WORKS:
    1. Counts number of active designations and responsibilities
    2. Analyzes scope and complexity of assigned roles
    3. Calculates workload score and capacity utilization
    
    BUSINESS PURPOSE:
    - Workload balancing and management
    - Resource allocation optimization
    - Performance assessment support
    - Role assignment planning
    """
    user = self.id.user
    designations = HoldsDesignation.objects.filter(working=user)
    
    workload_metrics = {
        'total_designations': designations.count(),
        'administrative_roles': 0,
        'academic_roles': 0,
        'workload_score': 0,
        'capacity_utilization': 'normal',
        'role_complexity': 'moderate'
    }
    
    complexity_score = 0
    
    for designation in designations:
        hierarchy_info = designation.designation.get_designation_hierarchy_level()
        
        if designation.designation.type == 'administrative':
            workload_metrics['administrative_roles'] += 1
        else:
            workload_metrics['academic_roles'] += 1
        
        # Add complexity based on hierarchy level
        complexity_score += hierarchy_info['level']
        
        # Add complexity based on authority scope
        scope_weights = {
            'institute_wide': 15,
            'department_wide': 10,
            'committee_level': 5,
            'limited': 2
        }
        complexity_score += scope_weights.get(hierarchy_info['authority_scope'], 2)
    
    workload_metrics['workload_score'] = complexity_score
    
    # Determine capacity utilization
    if complexity_score >= 50:
        workload_metrics['capacity_utilization'] = 'overloaded'
    elif complexity_score >= 30:
        workload_metrics['capacity_utilization'] = 'high'
    elif complexity_score >= 15:
        workload_metrics['capacity_utilization'] = 'normal'
    else:
        workload_metrics['capacity_utilization'] = 'underutilized'
    
    # Determine role complexity
    if complexity_score >= 40:
        workload_metrics['role_complexity'] = 'high'
    elif complexity_score >= 20:
        workload_metrics['role_complexity'] = 'moderate'
    else:
        workload_metrics['role_complexity'] = 'low'
    
    return workload_metrics

def generate_staff_performance_indicators(self):
    """
    CORE LOGIC: Generate key performance indicators for staff evaluation
    
    HOW IT WORKS:
    1. Combines profile completeness, role duration, and activity metrics
    2. Analyzes engagement patterns and system usage
    3. Calculates composite performance scores
    
    BUSINESS PURPOSE:
    - Performance evaluation support
    - Professional development planning
    - Administrative efficiency assessment
    - Recognition and incentive programs
    """
    extra_info = self.id
    profile_completeness = extra_info.calculate_profile_completeness()
    administrative_capabilities = self.get_administrative_capabilities()
    workload_analysis = self.get_workload_analysis()
    
    # Calculate engagement score
    engagement_score = 0
    if extra_info.date_modified:
        days_since_activity = (timezone.now() - extra_info.date_modified).days
        engagement_score = max(0, 100 - (days_since_activity * 2))  # 2 points per day of inactivity
    
    performance_indicators = {
        'profile_quality': {
            'completion_percentage': profile_completeness['percentage'],
            'quality_score': min(100, profile_completeness['percentage'] * 1.2)  # Bonus for complete profiles
        },
        'role_effectiveness': {
            'administrative_level': administrative_capabilities['administrative_level'],
            'capability_score': len(administrative_capabilities['module_access']) * 5,
            'authority_appropriate': workload_analysis['capacity_utilization'] in ['normal', 'high']
        },
        'engagement_metrics': {
            'activity_score': engagement_score,
            'system_utilization': 'active' if engagement_score > 70 else 'moderate' if engagement_score > 30 else 'low'
        },
        'overall_performance': {
            'composite_score': round((
                profile_completeness['percentage'] * 0.3 +
                administrative_capabilities.get('capability_score', 0) * 0.4 +
                engagement_score * 0.3
            ), 2),
            'performance_category': self._categorize_performance(
                profile_completeness['percentage'],
                administrative_capabilities.get('capability_score', 0),
                engagement_score
            )
        }
    }
    
    return performance_indicators

def _categorize_performance(self, profile_score, capability_score, engagement_score):
    """Helper method to categorize overall performance"""
    composite = (profile_score * 0.3 + capability_score * 0.4 + engagement_score * 0.3)
    
    if composite >= 80:
        return 'excellent'
    elif composite >= 65:
        return 'good'
    elif composite >= 50:
        return 'satisfactory'
    else:
        return 'needs_improvement'
```

**Key Business Logic**:
- **Administrative Focus**: Specialized methods for non-academic staff management
- **Capability Assessment**: Evaluates administrative permissions and authority
- **Workload Management**: Analyzes role distribution and capacity utilization
- **Performance Tracking**: Comprehensive metrics for staff evaluation

---

This concludes Part 2 of the Globals Module documentation. The next parts will cover:

**Part 3**: Faculty, Feedback, Issue, and IssueImage models
**Part 4**: ModuleAccess and PasswordResetTracker systems  
**Part 5**: View functions and business logic implementation
**Part 6**: API architecture and integration patterns
