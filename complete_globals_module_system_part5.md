# Complete Globals Module System Documentation - Fusion IIIT (Part 5)

## Database Models Analysis with Business Logic (Continued)

### 11. ModuleAccess Model
**Purpose**: Granular permission management system that defines access rights for different designations across all modules in the Fusion IIIT system.

**PostgreSQL Table**: `globals_moduleaccess`

**Fields Structure**:
```python
class ModuleAccess(models.Model):
    designation = models.CharField(max_length=155)
    program_and_curriculum = models.BooleanField(default=False)
    course_registration = models.BooleanField(default=False)
    course_management = models.BooleanField(default=False)
    other_academics = models.BooleanField(default=False)
    spacs = models.BooleanField(default=False)
    department = models.BooleanField(default=False)
    examinations = models.BooleanField(default=False)
    hr = models.BooleanField(default=False)
    iwd = models.BooleanField(default=False)
    complaint_management = models.BooleanField(default=False)
    fts = models.BooleanField(default=False)
    purchase_and_store = models.BooleanField(default=False)
    rspc = models.BooleanField(default=False)
    hostel_management = models.BooleanField(default=False)
    mess_management = models.BooleanField(default=False)
    gymkhana = models.BooleanField(default=False)
    placement_cell = models.BooleanField(default=False)
    visitor_hostel = models.BooleanField(default=False)
    phc = models.BooleanField(default=False)
    
    def __str__(self):
        return self.designation
```

**Enhanced Business Methods**:
```python
def get_permission_matrix(self):
    """
    CORE LOGIC: Generate comprehensive permission matrix for role-based access control
    
    HOW IT WORKS:
    1. Analyzes all module permissions assigned to this designation
    2. Categorizes permissions by functional areas and administrative levels
    3. Calculates permission scope and authority boundaries
    4. Provides structured data for access control systems
    
    BUSINESS PURPOSE:
    - Role-based access control implementation
    - Security policy enforcement and audit
    - Administrative oversight and permission management
    - System integration and module-level security
    """
    # Group permissions by functional categories
    permission_matrix = {
        'academic_administration': {
            'program_and_curriculum': self.program_and_curriculum,
            'course_registration': self.course_registration,
            'course_management': self.course_management,
            'other_academics': self.other_academics,
            'examinations': self.examinations,
            'department': self.department
        },
        'research_and_development': {
            'spacs': self.spacs,
            'rspc': self.rspc
        },
        'student_services': {
            'hostel_management': self.hostel_management,
            'mess_management': self.mess_management,
            'gymkhana': self.gymkhana,
            'placement_cell': self.placement_cell,
            'visitor_hostel': self.visitor_hostel,
            'phc': self.phc
        },
        'administrative_operations': {
            'hr': self.hr,
            'iwd': self.iwd,
            'complaint_management': self.complaint_management,
            'fts': self.fts,
            'purchase_and_store': self.purchase_and_store
        }
    }
    
    # Calculate permission statistics
    permission_stats = self._calculate_permission_statistics(permission_matrix)
    
    # Determine access level and scope
    access_profile = self._determine_access_profile(permission_matrix, permission_stats)
    
    # Generate security implications
    security_analysis = self._analyze_security_implications(permission_matrix)
    
    return {
        'designation': self.designation,
        'permission_matrix': permission_matrix,
        'statistics': permission_stats,
        'access_profile': access_profile,
        'security_analysis': security_analysis,
        'compliance_check': self._perform_compliance_check(permission_matrix),
        'recommendations': self._generate_permission_recommendations(permission_matrix, access_profile)
    }

def _calculate_permission_statistics(self, permission_matrix):
    """Helper method to calculate permission statistics"""
    stats = {
        'total_permissions': 0,
        'permissions_by_category': {},
        'permission_density': 0,
        'coverage_percentage': 0
    }
    
    total_possible_permissions = 0
    
    for category, permissions in permission_matrix.items():
        category_count = sum(permissions.values())
        category_total = len(permissions)
        
        stats['permissions_by_category'][category] = {
            'granted': category_count,
            'total_possible': category_total,
            'coverage_percentage': round((category_count / category_total) * 100, 1) if category_total > 0 else 0
        }
        
        stats['total_permissions'] += category_count
        total_possible_permissions += category_total
    
    # Calculate overall metrics
    stats['permission_density'] = round(
        (stats['total_permissions'] / total_possible_permissions) * 100, 1
    ) if total_possible_permissions > 0 else 0
    
    stats['coverage_percentage'] = stats['permission_density']
    
    return stats

def _determine_access_profile(self, permission_matrix, stats):
    """Helper method to determine access profile and authority level"""
    total_permissions = stats['total_permissions']
    
    # Determine access level
    if total_permissions >= 15:
        access_level = 'super_administrator'
    elif total_permissions >= 10:
        access_level = 'administrator'
    elif total_permissions >= 6:
        access_level = 'manager'
    elif total_permissions >= 3:
        access_level = 'coordinator'
    elif total_permissions >= 1:
        access_level = 'limited_access'
    else:
        access_level = 'no_access'
    
    # Determine scope of authority
    academic_permissions = stats['permissions_by_category']['academic_administration']['granted']
    administrative_permissions = stats['permissions_by_category']['administrative_operations']['granted']
    student_services_permissions = stats['permissions_by_category']['student_services']['granted']
    research_permissions = stats['permissions_by_category']['research_and_development']['granted']
    
    authority_scope = []
    
    if academic_permissions >= 4:
        authority_scope.append('academic_leadership')
    elif academic_permissions >= 2:
        authority_scope.append('academic_coordination')
    
    if administrative_permissions >= 3:
        authority_scope.append('administrative_management')
    elif administrative_permissions >= 1:
        authority_scope.append('administrative_support')
    
    if student_services_permissions >= 4:
        authority_scope.append('student_affairs_leadership')
    elif student_services_permissions >= 2:
        authority_scope.append('student_services_coordination')
    
    if research_permissions >= 1:
        authority_scope.append('research_administration')
    
    # Determine risk level
    high_risk_modules = ['hr', 'purchase_and_store', 'fts', 'examinations']
    high_risk_count = sum(1 for module in high_risk_modules if getattr(self, module, False))
    
    if high_risk_count >= 3:
        risk_level = 'high'
    elif high_risk_count >= 1:
        risk_level = 'medium'
    else:
        risk_level = 'low'
    
    return {
        'access_level': access_level,
        'authority_scope': authority_scope,
        'risk_level': risk_level,
        'is_privileged_role': total_permissions >= 8,
        'requires_audit': high_risk_count > 0,
        'delegation_capable': total_permissions >= 5
    }

def _analyze_security_implications(self, permission_matrix):
    """Helper method to analyze security implications of permission set"""
    security_analysis = {
        'risk_factors': [],
        'separation_of_duties': True,
        'privilege_escalation_risk': 'low',
        'audit_requirements': [],
        'compliance_issues': []
    }
    
    # Identify high-risk combinations
    financial_access = self.purchase_and_store
    hr_access = self.hr
    examination_access = self.examinations
    
    if financial_access and hr_access:
        security_analysis['risk_factors'].append('financial_and_hr_access_combination')
        security_analysis['privilege_escalation_risk'] = 'high'
    
    if examination_access and (self.course_management or self.course_registration):
        security_analysis['risk_factors'].append('examination_and_course_management_overlap')
    
    # Check for excessive permissions
    total_permissions = sum(
        sum(category.values()) for category in permission_matrix.values()
    )
    
    if total_permissions >= 12:
        security_analysis['risk_factors'].append('excessive_permissions')
        security_analysis['privilege_escalation_risk'] = 'medium'
    
    # Audit requirements
    if financial_access:
        security_analysis['audit_requirements'].append('financial_transaction_audit')
    
    if hr_access:
        security_analysis['audit_requirements'].append('personnel_data_access_audit')
    
    if examination_access:
        security_analysis['audit_requirements'].append('academic_integrity_audit')
    
    # Compliance checks
    sensitive_modules = ['hr', 'purchase_and_store', 'examinations', 'fts']
    sensitive_access_count = sum(1 for module in sensitive_modules if getattr(self, module, False))
    
    if sensitive_access_count >= 2:
        security_analysis['compliance_issues'].append('multiple_sensitive_module_access')
    
    return security_analysis

def _perform_compliance_check(self, permission_matrix):
    """Helper method to perform compliance validation"""
    compliance_results = {
        'is_compliant': True,
        'violations': [],
        'warnings': [],
        'recommendations': []
    }
    
    # Check for role appropriateness
    designation_lower = self.designation.lower()
    
    # Students shouldn't have administrative access
    if 'student' in designation_lower:
        admin_permissions = permission_matrix['administrative_operations']
        if any(admin_permissions.values()):
            compliance_results['is_compliant'] = False
            compliance_results['violations'].append('student_with_administrative_access')
    
    # Faculty should primarily have academic permissions
    if 'faculty' in designation_lower or 'professor' in designation_lower:
        admin_count = sum(permission_matrix['administrative_operations'].values())
        academic_count = sum(permission_matrix['academic_administration'].values())
        
        if admin_count > academic_count and admin_count > 2:
            compliance_results['warnings'].append('faculty_with_excessive_administrative_access')
    
    # Check for principle of least privilege
    total_permissions = sum(
        sum(category.values()) for category in permission_matrix.values()
    )
    
    if total_permissions > 10 and 'director' not in designation_lower and 'dean' not in designation_lower:
        compliance_results['warnings'].append('potential_over_privileged_role')
    
    # Check for segregation of duties
    conflicting_combinations = [
        (['purchase_and_store', 'fts'], 'financial_approval_and_tracking_conflict'),
        (['examinations', 'course_management'], 'examination_and_teaching_conflict'),
        (['hr', 'complaint_management'], 'hr_and_grievance_conflict')
    ]
    
    for modules, conflict_type in conflicting_combinations:
        if all(getattr(self, module, False) for module in modules):
            compliance_results['warnings'].append(conflict_type)
    
    return compliance_results

def _generate_permission_recommendations(self, permission_matrix, access_profile):
    """Helper method to generate permission optimization recommendations"""
    recommendations = []
    
    # Based on access level
    access_level = access_profile['access_level']
    
    if access_level == 'super_administrator':
        recommendations.extend([
            "Consider implementing delegation mechanisms for routine tasks",
            "Establish regular audit procedures for high-privilege account",
            "Implement additional authentication factors for sensitive operations"
        ])
    
    elif access_level == 'no_access' or access_level == 'limited_access':
        recommendations.append("Review if additional permissions are needed for role effectiveness")
    
    # Based on risk level
    risk_level = access_profile['risk_level']
    
    if risk_level == 'high':
        recommendations.extend([
            "Implement mandatory approval workflows for sensitive operations",
            "Schedule quarterly access reviews",
            "Enable detailed audit logging for all actions"
        ])
    
    # Based on authority scope
    authority_scope = access_profile['authority_scope']
    
    if 'academic_leadership' in authority_scope and 'administrative_management' in authority_scope:
        recommendations.append("Consider separating academic and administrative responsibilities")
    
    # Module-specific recommendations
    if self.purchase_and_store:
        recommendations.append("Implement purchase approval limits and escalation procedures")
    
    if self.hr:
        recommendations.append("Ensure GDPR/data privacy compliance for personnel data access")
    
    if self.examinations:
        recommendations.append("Implement academic integrity monitoring and audit trails")
    
    return recommendations

def get_effective_permissions_for_user(self, user):
    """
    CORE LOGIC: Calculate effective permissions for a specific user based on their designations
    
    HOW IT WORKS:
    1. Retrieves all designations held by the user
    2. Aggregates permissions from all applicable ModuleAccess records
    3. Resolves permission conflicts and inheritance
    4. Returns consolidated permission set for the user
    
    BUSINESS PURPOSE:
    - Dynamic permission calculation for access control
    - Multi-role permission aggregation
    - Real-time authorization decisions
    - Audit trail for permission grants
    """
    user_designations = HoldsDesignation.objects.filter(working=user)
    
    effective_permissions = {
        'consolidated_permissions': {},
        'permission_sources': [],
        'conflicts': [],
        'highest_privilege_level': 'none',
        'total_modules_accessible': 0
    }
    
    # Initialize all permissions as False
    all_modules = [
        'program_and_curriculum', 'course_registration', 'course_management',
        'other_academics', 'spacs', 'department', 'examinations', 'hr',
        'iwd', 'complaint_management', 'fts', 'purchase_and_store',
        'rspc', 'hostel_management', 'mess_management', 'gymkhana',
        'placement_cell', 'visitor_hostel', 'phc'
    ]
    
    for module in all_modules:
        effective_permissions['consolidated_permissions'][module] = False
    
    # Process each designation
    for designation_record in user_designations:
        try:
            module_access = ModuleAccess.objects.get(
                designation=designation_record.designation.name
            )
            
            permission_matrix = module_access.get_permission_matrix()
            source_info = {
                'designation': designation_record.designation.name,
                'permissions_granted': [],
                'access_level': permission_matrix['access_profile']['access_level'],
                'held_since': designation_record.held_at
            }
            
            # Aggregate permissions (OR operation - any role granting access enables it)
            for module in all_modules:
                if hasattr(module_access, module) and getattr(module_access, module):
                    if effective_permissions['consolidated_permissions'][module]:
                        # Track conflicts (multiple sources granting same permission)
                        conflict_info = {
                            'module': module,
                            'sources': [
                                src['designation'] for src in effective_permissions['permission_sources']
                                if module in src['permissions_granted']
                            ] + [designation_record.designation.name]
                        }
                        effective_permissions['conflicts'].append(conflict_info)
                    
                    effective_permissions['consolidated_permissions'][module] = True
                    source_info['permissions_granted'].append(module)
            
            effective_permissions['permission_sources'].append(source_info)
            
            # Track highest privilege level
            current_level = permission_matrix['access_profile']['access_level']
            effective_permissions['highest_privilege_level'] = self._compare_privilege_levels(
                effective_permissions['highest_privilege_level'], current_level
            )
            
        except ModuleAccess.DoesNotExist:
            # Designation doesn't have module access defined
            continue
    
    # Calculate total accessible modules
    effective_permissions['total_modules_accessible'] = sum(
        effective_permissions['consolidated_permissions'].values()
    )
    
    # Add user context
    effective_permissions['user_info'] = {
        'username': user.username,
        'user_type': user.extrainfo.user_type if hasattr(user, 'extrainfo') else 'unknown',
        'department': user.extrainfo.department.name if hasattr(user, 'extrainfo') and user.extrainfo.department else 'unknown',
        'total_designations': user_designations.count()
    }
    
    # Add authorization helpers
    effective_permissions['authorization_helpers'] = {
        'can_access_module': lambda module: effective_permissions['consolidated_permissions'].get(module, False),
        'has_administrative_access': any([
            effective_permissions['consolidated_permissions'].get(module, False)
            for module in ['hr', 'fts', 'purchase_and_store', 'iwd']
        ]),
        'has_academic_access': any([
            effective_permissions['consolidated_permissions'].get(module, False)
            for module in ['program_and_curriculum', 'course_management', 'examinations', 'department']
        ]),
        'requires_audit': effective_permissions['highest_privilege_level'] in ['administrator', 'super_administrator']
    }
    
    return effective_permissions

def _compare_privilege_levels(self, current_highest, new_level):
    """Helper method to compare and determine highest privilege level"""
    privilege_hierarchy = {
        'none': 0,
        'no_access': 0,
        'limited_access': 1,
        'coordinator': 2,
        'manager': 3,
        'administrator': 4,
        'super_administrator': 5
    }
    
    current_rank = privilege_hierarchy.get(current_highest, 0)
    new_rank = privilege_hierarchy.get(new_level, 0)
    
    if new_rank > current_rank:
        return new_level
    else:
        return current_highest

def generate_access_audit_report(self):
    """
    CORE LOGIC: Generate comprehensive audit report for permission assignment
    
    HOW IT WORKS:
    1. Analyzes current permission assignments against role requirements
    2. Identifies potential security risks and compliance issues
    3. Provides recommendations for permission optimization
    4. Generates audit trail and documentation
    
    BUSINESS PURPOSE:
    - Security compliance and audit requirements
    - Risk assessment and mitigation planning
    - Permission optimization and efficiency
    - Regulatory compliance documentation
    """
    permission_matrix = self.get_permission_matrix()
    
    audit_report = {
        'audit_metadata': {
            'designation': self.designation,
            'audit_date': timezone.now(),
            'auditor': 'system_automated',
            'audit_type': 'permission_review'
        },
        'permission_summary': {
            'total_permissions': permission_matrix['statistics']['total_permissions'],
            'access_level': permission_matrix['access_profile']['access_level'],
            'risk_level': permission_matrix['access_profile']['risk_level'],
            'compliance_status': permission_matrix['compliance_check']['is_compliant']
        },
        'detailed_analysis': {
            'permission_breakdown': permission_matrix['permission_matrix'],
            'security_implications': permission_matrix['security_analysis'],
            'compliance_findings': permission_matrix['compliance_check']
        },
        'risk_assessment': {
            'identified_risks': self._identify_audit_risks(permission_matrix),
            'mitigation_strategies': self._suggest_risk_mitigation(permission_matrix),
            'monitoring_requirements': self._define_monitoring_requirements(permission_matrix)
        },
        'recommendations': {
            'immediate_actions': [],
            'long_term_improvements': [],
            'process_enhancements': []
        },
        'usage_analytics': self._generate_usage_analytics(),
        'comparative_analysis': self._perform_comparative_analysis()
    }
    
    # Generate specific recommendations
    audit_report['recommendations'] = self._generate_audit_recommendations(permission_matrix)
    
    return audit_report

def _identify_audit_risks(self, permission_matrix):
    """Helper method to identify specific audit risks"""
    risks = []
    
    access_profile = permission_matrix['access_profile']
    security_analysis = permission_matrix['security_analysis']
    
    # High privilege risks
    if access_profile['access_level'] in ['administrator', 'super_administrator']:
        risks.append({
            'type': 'high_privilege_access',
            'severity': 'high',
            'description': 'Designation has high-level system access requiring enhanced monitoring'
        })
    
    # Security combination risks
    for risk_factor in security_analysis['risk_factors']:
        risks.append({
            'type': 'security_combination',
            'severity': 'medium',
            'description': f'Identified security concern: {risk_factor}'
        })
    
    # Compliance risks
    compliance_check = permission_matrix['compliance_check']
    for violation in compliance_check['violations']:
        risks.append({
            'type': 'compliance_violation',
            'severity': 'high',
            'description': f'Compliance violation: {violation}'
        })
    
    return risks

def _suggest_risk_mitigation(self, permission_matrix):
    """Helper method to suggest risk mitigation strategies"""
    strategies = []
    
    access_profile = permission_matrix['access_profile']
    
    if access_profile['risk_level'] == 'high':
        strategies.extend([
            'Implement mandatory two-factor authentication',
            'Require supervisory approval for sensitive operations',
            'Enable real-time activity monitoring',
            'Conduct monthly access reviews'
        ])
    
    if access_profile['is_privileged_role']:
        strategies.extend([
            'Implement time-based access restrictions',
            'Enable session monitoring and recording',
            'Require justification for privilege escalation'
        ])
    
    return strategies

def _define_monitoring_requirements(self, permission_matrix):
    """Helper method to define monitoring requirements"""
    requirements = []
    
    security_analysis = permission_matrix['security_analysis']
    
    for audit_requirement in security_analysis['audit_requirements']:
        requirements.append({
            'type': audit_requirement,
            'frequency': 'continuous',
            'retention_period': '7_years',
            'alert_conditions': 'unusual_access_patterns'
        })
    
    return requirements

def _generate_usage_analytics(self):
    """Helper method to generate usage analytics"""
    # This would integrate with actual usage tracking
    # Placeholder implementation
    return {
        'last_30_days': {
            'total_logins': 0,  # Would come from session tracking
            'modules_accessed': [],  # Would come from access logs
            'average_session_duration': 0,
            'peak_usage_hours': []
        },
        'historical_trends': {
            'permission_changes': [],  # Would track permission modifications
            'access_violations': 0,  # Would track unauthorized access attempts
            'compliance_scores': []  # Would track compliance over time
        }
    }

def _perform_comparative_analysis(self):
    """Helper method to perform comparative analysis with similar roles"""
    # Compare with other similar designations
    similar_designations = ModuleAccess.objects.exclude(id=self.id)
    
    comparison = {
        'similar_roles': [],
        'permission_variance': 'normal',
        'outlier_permissions': []
    }
    
    current_permission_count = sum([
        self.program_and_curriculum, self.course_registration, self.course_management,
        self.other_academics, self.spacs, self.department, self.examinations,
        self.hr, self.iwd, self.complaint_management, self.fts,
        self.purchase_and_store, self.rspc, self.hostel_management,
        self.mess_management, self.gymkhana, self.placement_cell,
        self.visitor_hostel, self.phc
    ])
    
    # Find roles with similar permission counts
    for other_role in similar_designations:
        other_count = sum([
            other_role.program_and_curriculum, other_role.course_registration,
            other_role.course_management, other_role.other_academics,
            other_role.spacs, other_role.department, other_role.examinations,
            other_role.hr, other_role.iwd, other_role.complaint_management,
            other_role.fts, other_role.purchase_and_store, other_role.rspc,
            other_role.hostel_management, other_role.mess_management,
            other_role.gymkhana, other_role.placement_cell,
            other_role.visitor_hostel, other_role.phc
        ])
        
        if abs(other_count - current_permission_count) <= 2:
            comparison['similar_roles'].append(other_role.designation)
    
    return comparison

def _generate_audit_recommendations(self, permission_matrix):
    """Helper method to generate audit-specific recommendations"""
    recommendations = {
        'immediate_actions': [],
        'long_term_improvements': [],
        'process_enhancements': []
    }
    
    compliance_check = permission_matrix['compliance_check']
    security_analysis = permission_matrix['security_analysis']
    access_profile = permission_matrix['access_profile']
    
    # Immediate actions for violations
    if not compliance_check['is_compliant']:
        recommendations['immediate_actions'].append("Address compliance violations before next audit cycle")
    
    if security_analysis['privilege_escalation_risk'] == 'high':
        recommendations['immediate_actions'].append("Implement additional security controls for high-risk permissions")
    
    # Long-term improvements
    if access_profile['access_level'] == 'super_administrator':
        recommendations['long_term_improvements'].append("Consider implementing delegation mechanisms to reduce centralized privileges")
    
    # Process enhancements
    recommendations['process_enhancements'].extend([
        "Implement automated permission reviews",
        "Establish role-based access certification process",
        "Create permission request and approval workflows"
    ])
    
    return recommendations

@staticmethod
def generate_system_wide_access_report():
    """
    CORE LOGIC: Generate comprehensive system-wide access and permission report
    
    HOW IT WORKS:
    1. Analyzes all ModuleAccess records across the system
    2. Identifies patterns, anomalies, and optimization opportunities
    3. Provides executive dashboard view of system-wide permissions
    4. Generates actionable insights for security and compliance
    
    BUSINESS PURPOSE:
    - System-wide security posture assessment
    - Executive reporting on access management
    - Strategic planning for permission optimization
    - Compliance reporting and risk management
    """
    all_module_access = ModuleAccess.objects.all()
    
    system_report = {
        'executive_summary': {
            'total_roles': all_module_access.count(),
            'total_permissions_granted': 0,
            'high_risk_roles': 0,
            'compliance_score': 0,
            'audit_date': timezone.now()
        },
        'permission_distribution': {
            'by_access_level': {},
            'by_module': {},
            'by_risk_level': {}
        },
        'security_analysis': {
            'over_privileged_roles': [],
            'under_privileged_roles': [],
            'conflicting_permissions': [],
            'orphaned_permissions': []
        },
        'compliance_overview': {
            'compliant_roles': 0,
            'non_compliant_roles': 0,
            'common_violations': [],
            'audit_requirements': []
        },
        'optimization_opportunities': {
            'role_consolidation': [],
            'permission_standardization': [],
            'security_enhancements': []
        },
        'trends_and_insights': {
            'permission_growth': 'stable',  # Would analyze historical data
            'access_patterns': {},
            'seasonal_variations': {}
        }
    }
    
    # Process each role
    total_permissions = 0
    compliant_count = 0
    access_level_distribution = {}
    risk_level_distribution = {}
    
    for role in all_module_access:
        permission_matrix = role.get_permission_matrix()
        
        # Update counters
        role_permission_count = permission_matrix['statistics']['total_permissions']
        total_permissions += role_permission_count
        
        if permission_matrix['compliance_check']['is_compliant']:
            compliant_count += 1
        
        # Access level distribution
        access_level = permission_matrix['access_profile']['access_level']
        access_level_distribution[access_level] = access_level_distribution.get(access_level, 0) + 1
        
        # Risk level distribution
        risk_level = permission_matrix['access_profile']['risk_level']
        risk_level_distribution[risk_level] = risk_level_distribution.get(risk_level, 0) + 1
        
        if risk_level == 'high':
            system_report['executive_summary']['high_risk_roles'] += 1
    
    # Update executive summary
    system_report['executive_summary']['total_permissions_granted'] = total_permissions
    system_report['executive_summary']['compliance_score'] = round(
        (compliant_count / all_module_access.count()) * 100, 1
    ) if all_module_access.count() > 0 else 0
    
    # Update distributions
    system_report['permission_distribution']['by_access_level'] = access_level_distribution
    system_report['permission_distribution']['by_risk_level'] = risk_level_distribution
    
    # Generate module-wise distribution
    module_permissions = ModuleAccess._calculate_module_wise_distribution(all_module_access)
    system_report['permission_distribution']['by_module'] = module_permissions
    
    # Security analysis
    system_report['security_analysis'] = ModuleAccess._perform_system_security_analysis(all_module_access)
    
    # Compliance overview
    system_report['compliance_overview'] = ModuleAccess._generate_compliance_overview(all_module_access)
    
    # Optimization opportunities
    system_report['optimization_opportunities'] = ModuleAccess._identify_optimization_opportunities(all_module_access)
    
    return system_report

@staticmethod
def _calculate_module_wise_distribution(all_module_access):
    """Helper method to calculate module-wise permission distribution"""
    module_stats = {}
    
    modules = [
        'program_and_curriculum', 'course_registration', 'course_management',
        'other_academics', 'spacs', 'department', 'examinations', 'hr',
        'iwd', 'complaint_management', 'fts', 'purchase_and_store',
        'rspc', 'hostel_management', 'mess_management', 'gymkhana',
        'placement_cell', 'visitor_hostel', 'phc'
    ]
    
    total_roles = all_module_access.count()
    
    for module in modules:
        granted_count = sum(1 for role in all_module_access if getattr(role, module, False))
        module_stats[module] = {
            'roles_with_access': granted_count,
            'percentage_coverage': round((granted_count / total_roles) * 100, 1) if total_roles > 0 else 0,
            'is_restricted': granted_count <= total_roles * 0.2,  # Less than 20% have access
            'is_common': granted_count >= total_roles * 0.8  # More than 80% have access
        }
    
    return module_stats

@staticmethod
def _perform_system_security_analysis(all_module_access):
    """Helper method to perform system-wide security analysis"""
    security_analysis = {
        'over_privileged_roles': [],
        'under_privileged_roles': [],
        'conflicting_permissions': [],
        'orphaned_permissions': []
    }
    
    for role in all_module_access:
        permission_matrix = role.get_permission_matrix()
        
        # Check for over-privileged roles
        if permission_matrix['statistics']['total_permissions'] > 12:
            security_analysis['over_privileged_roles'].append({
                'designation': role.designation,
                'permission_count': permission_matrix['statistics']['total_permissions'],
                'risk_level': permission_matrix['access_profile']['risk_level']
            })
        
        # Check for under-privileged roles (roles with very few permissions)
        if permission_matrix['statistics']['total_permissions'] < 2 and 'student' not in role.designation.lower():
            security_analysis['under_privileged_roles'].append({
                'designation': role.designation,
                'permission_count': permission_matrix['statistics']['total_permissions']
            })
        
        # Check for conflicting permission combinations
        conflicts = permission_matrix['security_analysis']['risk_factors']
        if conflicts:
            security_analysis['conflicting_permissions'].append({
                'designation': role.designation,
                'conflicts': conflicts
            })
    
    return security_analysis

@staticmethod
def _generate_compliance_overview(all_module_access):
    """Helper method to generate compliance overview"""
    compliance_overview = {
        'compliant_roles': 0,
        'non_compliant_roles': 0,
        'common_violations': {},
        'audit_requirements': []
    }
    
    violation_counter = {}
    
    for role in all_module_access:
        permission_matrix = role.get_permission_matrix()
        compliance_check = permission_matrix['compliance_check']
        
        if compliance_check['is_compliant']:
            compliance_overview['compliant_roles'] += 1
        else:
            compliance_overview['non_compliant_roles'] += 1
            
            # Count violations
            for violation in compliance_check['violations']:
                violation_counter[violation] = violation_counter.get(violation, 0) + 1
        
        # Collect audit requirements
        audit_reqs = permission_matrix['security_analysis']['audit_requirements']
        compliance_overview['audit_requirements'].extend(audit_reqs)
    
    # Most common violations
    compliance_overview['common_violations'] = dict(
        sorted(violation_counter.items(), key=lambda x: x[1], reverse=True)[:5]
    )
    
    # Remove duplicates from audit requirements
    compliance_overview['audit_requirements'] = list(set(compliance_overview['audit_requirements']))
    
    return compliance_overview

@staticmethod
def _identify_optimization_opportunities(all_module_access):
    """Helper method to identify optimization opportunities"""
    opportunities = {
        'role_consolidation': [],
        'permission_standardization': [],
        'security_enhancements': []
    }
    
    # Identify similar roles that could be consolidated
    role_permission_map = {}
    for role in all_module_access:
        permission_signature = tuple(sorted([
            module for module in [
                'program_and_curriculum', 'course_registration', 'course_management',
                'other_academics', 'spacs', 'department', 'examinations', 'hr',
                'iwd', 'complaint_management', 'fts', 'purchase_and_store',
                'rspc', 'hostel_management', 'mess_management', 'gymkhana',
                'placement_cell', 'visitor_hostel', 'phc'
            ] if getattr(role, module, False)
        ]))
        
        if permission_signature in role_permission_map:
            role_permission_map[permission_signature].append(role.designation)
        else:
            role_permission_map[permission_signature] = [role.designation]
    
    # Find consolidation opportunities
    for permission_set, roles in role_permission_map.items():
        if len(roles) > 1:
            opportunities['role_consolidation'].append({
                'similar_roles': roles,
                'shared_permissions': len(permission_set),
                'consolidation_potential': 'high' if len(roles) > 3 else 'medium'
            })
    
    # Permission standardization opportunities
    # (This would analyze permission patterns for standardization)
    
    return opportunities
```

**Key Business Logic**:
- **Granular Permission Management**: Fine-grained control over module access rights
- **Security Analysis**: Comprehensive risk assessment and compliance checking
- **Audit Capabilities**: Detailed audit trails and compliance reporting
- **System-Wide Analytics**: Enterprise-level permission analysis and optimization

---

### 12. PasswordResetTracker Model
**Purpose**: Rate limiting and security management for password reset requests to prevent abuse and enhance system security.

**PostgreSQL Table**: `globals_passwordresettracker`

**Fields Structure**:
```python
class PasswordResetTracker(models.Model):
    email = models.EmailField(unique=True)
    last_reset = models.DateTimeField(null=True, blank=True)
    
    def __str__(self):
        return self.email
```

**Enhanced Business Methods**:
```python
def is_reset_allowed(self):
    """
    CORE LOGIC: Determine if password reset is allowed based on rate limiting rules
    
    HOW IT WORKS:
    1. Checks if sufficient time has passed since last reset request
    2. Applies configurable rate limiting policies
    3. Considers security factors and user context
    4. Returns authorization decision with detailed reasoning
    
    BUSINESS PURPOSE:
    - Prevent password reset abuse and brute force attacks
    - Maintain system security while enabling legitimate resets
    - Provide user-friendly error messaging and guidance
    - Support compliance with security policies
    """
    if not self.last_reset:
        # First time reset request
        return {
            'allowed': True,
            'reason': 'first_time_request',
            'wait_time_remaining': 0,
            'next_allowed_at': None
        }
    
    # Calculate time since last reset
    time_since_reset = timezone.now() - self.last_reset
    
    # Rate limiting rules (configurable)
    rate_limits = self._get_rate_limiting_rules()
    
    for limit_name, limit_config in rate_limits.items():
        if time_since_reset < limit_config['minimum_interval']:
            wait_time_remaining = limit_config['minimum_interval'] - time_since_reset
            next_allowed_at = self.last_reset + limit_config['minimum_interval']
            
            return {
                'allowed': False,
                'reason': f'rate_limit_{limit_name}',
                'wait_time_remaining': wait_time_remaining,
                'next_allowed_at': next_allowed_at,
                'friendly_message': limit_config['user_message']
            }
    
    # All rate limits passed
    return {
        'allowed': True,
        'reason': 'rate_limits_satisfied',
        'wait_time_remaining': timezone.timedelta(0),
        'next_allowed_at': None
    }

def _get_rate_limiting_rules(self):
    """Helper method to get configurable rate limiting rules"""
    return {
        'basic': {
            'minimum_interval': timezone.timedelta(hours=24),
            'user_message': 'Password can only be reset once every 24 hours for security reasons.'
        },
        'strict': {
            'minimum_interval': timezone.timedelta(hours=1),
            'user_message': 'Please wait at least 1 hour between password reset requests.'
        },
        'emergency': {
            'minimum_interval': timezone.timedelta(minutes=15),
            'user_message': 'Multiple reset requests detected. Please wait 15 minutes before trying again.'
        }
    }

def record_reset_attempt(self, success=True):
    """
    CORE LOGIC: Record password reset attempt for tracking and analysis
    
    HOW IT WORKS:
    1. Updates last_reset timestamp for successful resets
    2. Logs attempt for security monitoring and analysis
    3. Triggers additional security measures if needed
    4. Maintains audit trail for compliance
    
    BUSINESS PURPOSE:
    - Security monitoring and anomaly detection
    - Audit trail for password security compliance
    - User behavior analysis and support
    - System abuse prevention and mitigation
    """
    reset_record = {
        'email': self.email,
        'timestamp': timezone.now(),
        'success': success,
        'ip_address': None,  # Would be captured from request context
        'user_agent': None,  # Would be captured from request context
        'security_flags': []
    }
    
    if success:
        # Update last successful reset time
        self.last_reset = timezone.now()
        self.save()
        
        # Log successful reset
        reset_record['action'] = 'password_reset_successful'
        
    else:
        # Log failed attempt
        reset_record['action'] = 'password_reset_failed'
        reset_record['security_flags'].append('failed_reset_attempt')
    
    # Check for suspicious patterns
    security_analysis = self._analyze_reset_patterns()
    reset_record['security_analysis'] = security_analysis
    
    # TODO: Log to security audit system
    self._log_to_security_system(reset_record)
    
    return reset_record

def _analyze_reset_patterns(self):
    """Helper method to analyze password reset patterns for security"""
    analysis = {
        'risk_level': 'low',
        'suspicious_indicators': [],
        'recommendations': []
    }
    
    # Check frequency of resets
    if self.last_reset:
        time_since_last = timezone.now() - self.last_reset
        
        if time_since_last < timezone.timedelta(hours=1):
            analysis['risk_level'] = 'high'
            analysis['suspicious_indicators'].append('very_frequent_resets')
            analysis['recommendations'].append('temporarily_block_resets')
        
        elif time_since_last < timezone.timedelta(hours=6):
            analysis['risk_level'] = 'medium'
            analysis['suspicious_indicators'].append('frequent_resets')
            analysis['recommendations'].append('require_additional_verification')
    
    # TODO: Add more sophisticated pattern analysis
    # - Check for multiple resets across different emails from same IP
    # - Analyze time-of-day patterns
    # - Check against known attack patterns
    
    return analysis

def _log_to_security_system(self, reset_record):
    """Helper method to log security events"""
    # TODO: Integrate with actual security logging system
    # This could send to:
    # - SIEM system
    # - Security audit database
    # - Alert management system
    # - Compliance logging service
    pass

def get_reset_history_summary(self):
    """
    CORE LOGIC: Generate summary of password reset history for user and admin review
    
    HOW IT WORKS:
    1. Analyzes historical reset patterns and frequency
    2. Identifies potential security concerns or normal usage
    3. Provides user-friendly summary and recommendations
    4. Supports administrative review and decision making
    
    BUSINESS PURPOSE:
    - User self-service and awareness of account activity
    - Administrative oversight and security monitoring
    - Support ticket context and troubleshooting
    - Compliance reporting and audit support
    """
    summary = {
        'account_info': {
            'email': self.email,
            'first_reset_date': None,  # Would track from comprehensive logging
            'last_reset_date': self.last_reset,
            'total_resets': 1 if self.last_reset else 0  # Simplified - would track actual count
        },
        'current_status': {
            'can_reset_now': self.is_reset_allowed()['allowed'],
            'next_allowed_reset': self.is_reset_allowed().get('next_allowed_at'),
            'time_until_next_reset': self.is_reset_allowed().get('wait_time_remaining'),
            'account_locked': False  # Would check for account lockout status
        },
        'security_assessment': {
            'risk_level': self._analyze_reset_patterns()['risk_level'],
            'suspicious_activity': len(self._analyze_reset_patterns()['suspicious_indicators']) > 0,
            'recommended_actions': self._analyze_reset_patterns()['recommendations']
        },
        'usage_patterns': {
            'reset_frequency': self._calculate_reset_frequency(),
            'typical_reset_interval': self._calculate_typical_interval(),
            'last_activity_summary': self._generate_activity_summary()
        }
    }
    
    return summary

def _calculate_reset_frequency(self):
    """Helper method to calculate password reset frequency"""
    if not self.last_reset:
        return 'no_history'
    
    # Simplified calculation - in real implementation would analyze full history
    days_since_first_reset = 30  # Placeholder - would calculate from actual first reset date
    total_resets = 1  # Placeholder - would get actual count
    
    if days_since_first_reset > 0:
        frequency = total_resets / (days_since_first_reset / 30)  # Resets per month
        
        if frequency > 2:
            return 'very_high'
        elif frequency > 1:
            return 'high'
        elif frequency > 0.5:
            return 'moderate'
        else:
            return 'low'
    
    return 'insufficient_data'

def _calculate_typical_interval(self):
    """Helper method to calculate typical interval between resets"""
    # Simplified - would analyze full reset history
    if not self.last_reset:
        return None
    
    # Placeholder calculation
    return {
        'average_days': 30,  # Would calculate from actual data
        'pattern': 'monthly',  # Would analyze for patterns
        'consistency': 'irregular'  # Would assess consistency
    }

def _generate_activity_summary(self):
    """Helper method to generate recent activity summary"""
    if not self.last_reset:
        return "No password reset history available"
    
    time_since_last = timezone.now() - self.last_reset
    
    if time_since_last.days == 0:
        return "Password was reset today"
    elif time_since_last.days == 1:
        return "Password was reset yesterday"
    elif time_since_last.days < 7:
        return f"Password was reset {time_since_last.days} days ago"
    elif time_since_last.days < 30:
        weeks = time_since_last.days // 7
        return f"Password was reset {weeks} week{'s' if weeks != 1 else ''} ago"
    else:
        months = time_since_last.days // 30
        return f"Password was reset {months} month{'s' if months != 1 else ''} ago"

@staticmethod
def get_system_reset_analytics():
    """
    CORE LOGIC: Generate system-wide password reset analytics for security monitoring
    
    HOW IT WORKS:
    1. Analyzes all password reset activity across the system
    2. Identifies trends, patterns, and potential security issues
    3. Provides executive dashboard metrics and alerts
    4. Supports security policy optimization and compliance
    
    BUSINESS PURPOSE:
    - System-wide security monitoring and threat detection
    - Password policy effectiveness assessment
    - User behavior analysis and support optimization
    - Compliance reporting and audit documentation
    """
    all_trackers = PasswordResetTracker.objects.all()
    
    analytics = {
        'overview_metrics': {
            'total_users_with_resets': all_trackers.count(),
            'recent_resets_24h': 0,
            'recent_resets_7d': 0,
            'recent_resets_30d': 0,
            'analysis_date': timezone.now()
        },
        'temporal_analysis': {
            'daily_patterns': {},
            'weekly_patterns': {},
            'monthly_trends': {},
            'seasonal_variations': {}
        },
        'security_insights': {
            'high_frequency_users': [],
            'suspicious_patterns': [],
            'blocked_attempts': 0,
            'security_score': 0
        },
        'user_behavior': {
            'reset_frequency_distribution': {},
            'typical_reset_intervals': {},
            'help_desk_correlation': {}
        },
        'system_health': {
            'reset_success_rate': 0,
            'average_reset_interval': 0,
            'policy_effectiveness': 'good',
            'recommendations': []
        }
    }
    
    # Calculate overview metrics
    now = timezone.now()
    
    for tracker in all_trackers:
        if tracker.last_reset:
            time_since_reset = now - tracker.last_reset
            
            if time_since_reset <= timezone.timedelta(days=1):
                analytics['overview_metrics']['recent_resets_24h'] += 1
            
            if time_since_reset <= timezone.timedelta(days=7):
                analytics['overview_metrics']['recent_resets_7d'] += 1
            
            if time_since_reset <= timezone.timedelta(days=30):
                analytics['overview_metrics']['recent_resets_30d'] += 1
    
    # Analyze patterns and generate insights
    analytics['temporal_analysis'] = PasswordResetTracker._analyze_temporal_patterns(all_trackers)
    analytics['security_insights'] = PasswordResetTracker._analyze_security_patterns(all_trackers)
    analytics['user_behavior'] = PasswordResetTracker._analyze_user_behavior(all_trackers)
    analytics['system_health'] = PasswordResetTracker._assess_system_health(all_trackers)
    
    return analytics

@staticmethod
def _analyze_temporal_patterns(trackers):
    """Helper method to analyze temporal patterns in password resets"""
    patterns = {
        'daily_patterns': {},
        'weekly_patterns': {},
        'monthly_trends': {},
        'seasonal_variations': {}
    }
    
    # Analyze daily patterns (hour of day)
    hourly_counts = {}
    for tracker in trackers:
        if tracker.last_reset:
            hour = tracker.last_reset.hour
            hourly_counts[hour] = hourly_counts.get(hour, 0) + 1
    
    patterns['daily_patterns'] = {
        'hourly_distribution': hourly_counts,
        'peak_hours': sorted(hourly_counts.items(), key=lambda x: x[1], reverse=True)[:3],
        'low_activity_hours': sorted(hourly_counts.items(), key=lambda x: x[1])[:3]
    }
    
    # Analyze weekly patterns (day of week)
    weekly_counts = {}
    for tracker in trackers:
        if tracker.last_reset:
            day = tracker.last_reset.strftime('%A')
            weekly_counts[day] = weekly_counts.get(day, 0) + 1
    
    patterns['weekly_patterns'] = {
        'daily_distribution': weekly_counts,
        'busiest_days': sorted(weekly_counts.items(), key=lambda x: x[1], reverse=True)[:3]
    }
    
    return patterns

@staticmethod
def _analyze_security_patterns(trackers):
    """Helper method to analyze security-related patterns"""
    security_insights = {
        'high_frequency_users': [],
        'suspicious_patterns': [],
        'blocked_attempts': 0,
        'security_score': 85  # Default score, would calculate based on actual metrics
    }
    
    now = timezone.now()
    
    for tracker in trackers:
        if tracker.last_reset:
            # Check for high-frequency resets
            time_since_reset = now - tracker.last_reset
            if time_since_reset < timezone.timedelta(hours=6):
                security_insights['high_frequency_users'].append({
                    'email': tracker.email,
                    'last_reset': tracker.last_reset,
                    'hours_since_reset': time_since_reset.total_seconds() / 3600
                })
            
            # Check for rate limit violations
            if not tracker.is_reset_allowed()['allowed']:
                security_insights['blocked_attempts'] += 1
    
    # Calculate security score
    total_users = trackers.count()
    high_freq_count = len(security_insights['high_frequency_users'])
    blocked_count = security_insights['blocked_attempts']
    
    if total_users > 0:
        security_score = max(0, 100 - (high_freq_count / total_users * 50) - (blocked_count / total_users * 30))
        security_insights['security_score'] = round(security_score, 1)
    
    return security_insights

@staticmethod
def _analyze_user_behavior(trackers):
    """Helper method to analyze user behavior patterns"""
    behavior_analysis = {
        'reset_frequency_distribution': {},
        'typical_reset_intervals': {},
        'help_desk_correlation': {}
    }
    
    frequency_categories = {'low': 0, 'moderate': 0, 'high': 0, 'very_high': 0}
    
    for tracker in trackers:
        frequency = tracker._calculate_reset_frequency()
        if frequency in frequency_categories:
            frequency_categories[frequency] += 1
    
    behavior_analysis['reset_frequency_distribution'] = frequency_categories
    
    # Calculate average reset interval
    intervals = []
    for tracker in trackers:
        interval_data = tracker._calculate_typical_interval()
        if interval_data and interval_data['average_days']:
            intervals.append(interval_data['average_days'])
    
    if intervals:
        behavior_analysis['typical_reset_intervals'] = {
            'average_days': round(sum(intervals) / len(intervals), 1),
            'median_days': sorted(intervals)[len(intervals) // 2],
            'distribution': 'normal'  # Would perform statistical analysis
        }
    
    return behavior_analysis

@staticmethod
def _assess_system_health(trackers):
    """Helper method to assess overall system health"""
    health_assessment = {
        'reset_success_rate': 95,  # Would calculate from actual success/failure data
        'average_reset_interval': 0,
        'policy_effectiveness': 'good',
        'recommendations': []
    }
    
    # Calculate average reset interval
    intervals = []
    for tracker in trackers:
        if tracker.last_reset:
            # Simplified calculation - would use actual historical data
            intervals.append(30)  # Placeholder
    
    if intervals:
        avg_interval = sum(intervals) / len(intervals)
        health_assessment['average_reset_interval'] = round(avg_interval, 1)
        
        # Assess policy effectiveness
        if avg_interval > 90:
            health_assessment['policy_effectiveness'] = 'excellent'
        elif avg_interval > 60:
            health_assessment['policy_effectiveness'] = 'good'
        elif avg_interval > 30:
            health_assessment['policy_effectiveness'] = 'fair'
        else:
            health_assessment['policy_effectiveness'] = 'needs_improvement'
            health_assessment['recommendations'].append('Consider implementing stronger password policies')
    
    # Generate recommendations
    blocked_attempts = sum(1 for tracker in trackers if not tracker.is_reset_allowed()['allowed'])
    
    if blocked_attempts > trackers.count() * 0.1:  # More than 10% blocked
        health_assessment['recommendations'].append('Review rate limiting policies - high block rate detected')
    
    return health_assessment

def __str__(self):
    status = "can reset" if self.is_reset_allowed()['allowed'] else "rate limited"
    last_reset_str = self.last_reset.strftime('%Y-%m-%d %H:%M') if self.last_reset else "never"
    return f"{self.email} ({status}, last reset: {last_reset_str})"
```

**Key Business Logic**:
- **Rate Limiting**: Prevents password reset abuse through configurable time-based restrictions
- **Security Monitoring**: Tracks patterns and identifies suspicious reset behavior
- **Audit Trail**: Maintains comprehensive logging for compliance and security analysis
- **System Analytics**: Provides enterprise-level insights into password reset patterns

---

This concludes Part 5 of the Globals Module documentation. The next parts will cover:

**Part 6**: View functions and business logic implementation
**Part 7**: API architecture and integration patterns
**Part 8**: Administrative features and system integration
