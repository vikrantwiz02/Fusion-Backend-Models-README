# Complete Gymkhana Module System Documentation - Fusion IIIT (Part 3)

## Database Models Analysis with Business Logic (Continued)

### 5. Club_member Model
**Purpose**: Club membership management system that tracks student participation, roles, and membership lifecycle across all clubs.

**PostgreSQL Table**: `Club_member`

**Fields Structure**:
```python
class Club_member(models.Model):
    club = models.ForeignKey(Club_info, on_delete=models.CASCADE)
    member = models.ForeignKey(User, on_delete=models.CASCADE)  
    year = models.CharField(max_length=4, default="2020")
    designation = models.CharField(max_length=30, default="member")
```

**Enhanced Business Methods**:
```python
def get_comprehensive_membership_analytics(self):
    """
    CORE LOGIC: Complete membership lifecycle and engagement analytics
    
    HOW IT WORKS:
    1. Analyzes member's participation patterns and engagement levels
    2. Tracks role progression and leadership development
    3. Evaluates contribution to club activities and success
    4. Provides insights for member development and retention
    
    BUSINESS PURPOSE:
    - Member engagement optimization and retention strategies
    - Leadership development pathway tracking
    - Performance evaluation for role assignments
    - Club culture and satisfaction assessment
    """
    # Member profile and engagement analysis
    member_profile = self._analyze_member_profile()
    
    # Role progression and leadership development
    leadership_analysis = self._analyze_leadership_progression()
    
    # Activity participation and contribution metrics
    participation_metrics = self._calculate_participation_metrics()
    
    # Club impact and influence assessment
    impact_assessment = self._assess_member_impact()
    
    # Member development opportunities
    development_opportunities = self._identify_development_opportunities()
    
    # Retention risk analysis
    retention_analysis = self._analyze_retention_risk()
    
    return {
        'member_overview': {
            'member_id': self.member.id,
            'member_name': self.member.get_full_name(),
            'club_name': self.club.club_name,
            'current_designation': self.designation,
            'membership_year': self.year,
            'membership_duration': self._calculate_membership_duration()
        },
        'member_profile': member_profile,
        'leadership_analysis': leadership_analysis,
        'participation_metrics': participation_metrics,
        'impact_assessment': impact_assessment,
        'development_opportunities': development_opportunities,
        'retention_analysis': retention_analysis,
        'recommendations': self._generate_member_recommendations()
    }

def _analyze_member_profile(self):
    """Helper method to analyze comprehensive member profile"""
    # Get member's academic information
    try:
        from applications.academic_information.models import Student
        student_info = Student.objects.get(id=self.member.extrainfo.id)
        academic_profile = {
            'programme': student_info.programme,
            'batch': student_info.batch,
            'cpi': getattr(student_info, 'cpi', 'N/A'),
            'branch': getattr(student_info, 'branch', 'N/A')
        }
    except:
        academic_profile = {'status': 'academic_info_unavailable'}
    
    # Membership history across all clubs
    all_memberships = Club_member.objects.filter(member=self.member)
    membership_history = []
    
    for membership in all_memberships:
        membership_history.append({
            'club_name': membership.club.club_name,
            'designation': membership.designation,
            'year': membership.year,
            'is_current': membership.id == self.id
        })
    
    # Leadership experience calculation
    leadership_roles = all_memberships.filter(
        designation__in=['coordinator', 'convenor', 'secretary', 'head', 'captain']
    ).count()
    
    # Multi-club participation analysis
    unique_clubs = all_memberships.values('club').distinct().count()
    
    return {
        'academic_profile': academic_profile,
        'membership_history': membership_history,
        'leadership_experience': {
            'total_leadership_roles': leadership_roles,
            'leadership_percentage': round((leadership_roles / len(membership_history)) * 100, 1) if membership_history else 0,
            'multi_club_leader': leadership_roles > 1
        },
        'club_diversity': {
            'total_clubs_joined': unique_clubs,
            'club_diversity_score': min(unique_clubs * 25, 100),  # Max 4 clubs for 100%
            'is_multi_club_member': unique_clubs > 1
        }
    }

def _analyze_leadership_progression(self):
    """Helper method to analyze leadership development and progression"""
    # Get chronological membership progression
    member_memberships = Club_member.objects.filter(
        member=self.member
    ).order_by('year')
    
    progression_path = []
    designation_hierarchy = {
        'member': 1,
        'active_member': 2,
        'volunteer': 3,
        'team_member': 4,
        'executive': 5,
        'secretary': 6,
        'coordinator': 7,
        'convenor': 8,
        'head': 9,
        'captain': 10
    }
    
    for membership in member_memberships:
        designation_level = designation_hierarchy.get(membership.designation.lower(), 1)
        progression_path.append({
            'year': membership.year,
            'club': membership.club.club_name,
            'designation': membership.designation,
            'level': designation_level,
            'is_leadership': designation_level >= 6
        })
    
    # Calculate progression metrics
    progression_trend = self._calculate_progression_trend(progression_path)
    leadership_readiness = self._assess_leadership_readiness(progression_path)
    
    # Current role analysis
    current_level = designation_hierarchy.get(self.designation.lower(), 1)
    potential_next_roles = self._identify_potential_next_roles(current_level)
    
    return {
        'progression_path': progression_path,
        'progression_metrics': {
            'current_level': current_level,
            'highest_level_achieved': max([p['level'] for p in progression_path]) if progression_path else 1,
            'progression_trend': progression_trend,
            'years_in_leadership': sum(1 for p in progression_path if p['is_leadership']),
            'leadership_clubs': len(set(p['club'] for p in progression_path if p['is_leadership']))
        },
        'leadership_readiness': leadership_readiness,
        'potential_next_roles': potential_next_roles,
        'development_timeline': self._create_development_timeline(progression_path)
    }

def _calculate_progression_trend(self, progression_path):
    """Helper method to calculate member's progression trend"""
    if len(progression_path) < 2:
        return 'insufficient_data'
    
    levels = [p['level'] for p in progression_path]
    
    # Calculate trend over time
    recent_trend = levels[-3:] if len(levels) >= 3 else levels
    
    if len(set(recent_trend)) == 1:
        return 'stable'
    elif recent_trend[-1] > recent_trend[0]:
        return 'ascending'
    elif recent_trend[-1] < recent_trend[0]:
        return 'descending'
    else:
        return 'fluctuating'

def _assess_leadership_readiness(self, progression_path):
    """Helper method to assess member's readiness for leadership roles"""
    # Leadership readiness factors
    years_experience = len(progression_path)
    leadership_experience = sum(1 for p in progression_path if p['is_leadership'])
    club_diversity = len(set(p['club'] for p in progression_path))
    highest_level = max([p['level'] for p in progression_path]) if progression_path else 1
    
    # Calculate readiness score
    experience_score = min(years_experience * 20, 40)  # Max 40 for 2+ years
    leadership_score = min(leadership_experience * 30, 30)  # Max 30 for 1+ leadership role
    diversity_score = min(club_diversity * 15, 30)  # Max 30 for 2+ clubs
    
    readiness_score = experience_score + leadership_score + diversity_score
    
    # Determine readiness level
    if readiness_score >= 80:
        readiness_level = 'highly_ready'
    elif readiness_score >= 60:
        readiness_level = 'ready'
    elif readiness_score >= 40:
        readiness_level = 'developing'
    else:
        readiness_level = 'early_stage'
    
    return {
        'readiness_score': readiness_score,
        'readiness_level': readiness_level,
        'contributing_factors': {
            'experience_years': years_experience,
            'leadership_positions': leadership_experience,
            'club_diversity': club_diversity,
            'highest_level_achieved': highest_level
        },
        'readiness_assessment': self._generate_readiness_assessment(readiness_level, readiness_score)
    }

def _generate_readiness_assessment(self, readiness_level, readiness_score):
    """Helper method to generate detailed readiness assessment"""
    assessments = {
        'highly_ready': {
            'description': 'Excellent candidate for senior leadership roles',
            'strengths': ['extensive_experience', 'proven_leadership', 'multi_club_expertise'],
            'recommended_roles': ['convenor', 'head', 'captain'],
            'development_focus': ['strategic_planning', 'team_building', 'institutional_representation']
        },
        'ready': {
            'description': 'Good candidate for mid-level leadership positions',
            'strengths': ['solid_experience', 'some_leadership_background'],
            'recommended_roles': ['coordinator', 'secretary', 'executive'],
            'development_focus': ['leadership_skills', 'project_management', 'team_coordination']
        },
        'developing': {
            'description': 'Promising member with development potential',
            'strengths': ['growing_experience', 'commitment_to_activities'],
            'recommended_roles': ['team_member', 'volunteer', 'executive'],
            'development_focus': ['skill_building', 'responsibility_taking', 'initiative_development']
        },
        'early_stage': {
            'description': 'New member with learning and growth opportunities',
            'strengths': ['enthusiasm', 'potential_for_growth'],
            'recommended_roles': ['member', 'active_member', 'volunteer'],
            'development_focus': ['basic_skills', 'club_culture_understanding', 'participation_increase']
        }
    }
    
    return assessments.get(readiness_level, assessments['early_stage'])

def _identify_potential_next_roles(self, current_level):
    """Helper method to identify potential next roles for progression"""
    role_progression_map = {
        1: ['active_member', 'volunteer'],  # member
        2: ['volunteer', 'team_member'],    # active_member
        3: ['team_member', 'executive'],    # volunteer
        4: ['executive', 'secretary'],      # team_member
        5: ['secretary', 'coordinator'],    # executive
        6: ['coordinator', 'convenor'],     # secretary
        7: ['convenor', 'head'],           # coordinator
        8: ['head', 'captain'],            # convenor
        9: ['captain', 'senior_advisor'],   # head
        10: ['senior_advisor', 'mentor']    # captain
    }
    
    next_roles = role_progression_map.get(current_level, [])
    
    return {
        'immediate_next_roles': next_roles,
        'timeline_estimate': '6-12_months' if next_roles else 'current_role_development',
        'requirements_for_progression': self._get_progression_requirements(current_level)
    }

def _get_progression_requirements(self, current_level):
    """Helper method to get requirements for role progression"""
    requirements_map = {
        1: ['attend_regular_meetings', 'participate_in_events', 'show_commitment'],
        2: ['take_initiative', 'volunteer_for_tasks', 'demonstrate_reliability'],
        3: ['lead_small_projects', 'mentor_new_members', 'develop_specialized_skills'],
        4: ['manage_team_activities', 'coordinate_events', 'show_leadership_potential'],
        5: ['demonstrate_project_management', 'build_external_relationships', 'strategic_thinking'],
        6: ['lead_major_initiatives', 'represent_club_externally', 'mentoring_abilities'],
        7: ['institutional_representation', 'strategic_planning', 'crisis_management'],
        8: ['executive_decision_making', 'long_term_vision', 'stakeholder_management'],
        9: ['institutional_leadership', 'policy_development', 'succession_planning'],
        10: ['advisory_role', 'knowledge_transfer', 'strategic_guidance']
    }
    
    return requirements_map.get(current_level, ['continue_current_development'])

def _create_development_timeline(self, progression_path):
    """Helper method to create member development timeline"""
    if not progression_path:
        return {'status': 'no_progression_data'}
    
    timeline = []
    for i, path in enumerate(progression_path):
        milestone = {
            'year': path['year'],
            'achievement': f"Achieved {path['designation']} in {path['club']}",
            'significance': 'leadership_role' if path['is_leadership'] else 'membership_role',
            'progression_step': i + 1
        }
        timeline.append(milestone)
    
    # Add future projections
    current_year = int(datetime.datetime.now().year)
    next_year = current_year + 1
    
    projected_milestone = {
        'year': str(next_year),
        'projected_achievement': 'Potential role progression based on current trajectory',
        'significance': 'projected_milestone',
        'confidence_level': self._calculate_projection_confidence(progression_path)
    }
    
    timeline.append(projected_milestone)
    
    return {
        'historical_timeline': timeline[:-1],
        'projected_timeline': [timeline[-1]],
        'total_milestones': len(timeline) - 1,
        'development_velocity': self._calculate_development_velocity(progression_path)
    }

def _calculate_development_velocity(self, progression_path):
    """Helper method to calculate member's development velocity"""
    if len(progression_path) < 2:
        return 'insufficient_data'
    
    years_span = int(progression_path[-1]['year']) - int(progression_path[0]['year'])
    level_progression = progression_path[-1]['level'] - progression_path[0]['level']
    
    if years_span == 0:
        return 'single_year_data'
    
    velocity = level_progression / years_span
    
    if velocity > 2:
        return 'rapid_development'
    elif velocity > 1:
        return 'steady_development'
    elif velocity > 0:
        return 'gradual_development'
    else:
        return 'stable_or_declining'

def _calculate_participation_metrics(self):
    """Helper method to calculate comprehensive participation metrics"""
    # This would integrate with event attendance, project participation, etc.
    # For now, we'll calculate based on available membership data
    
    member_clubs = Club_member.objects.filter(member=self.member)
    
    # Participation breadth
    total_clubs = member_clubs.count()
    leadership_roles = member_clubs.filter(
        designation__in=['coordinator', 'convenor', 'secretary', 'head', 'captain']
    ).count()
    
    # Participation depth (years in current club)
    current_club_memberships = Club_member.objects.filter(
        member=self.member,
        club=self.club
    ).count()
    
    # Activity level estimation based on roles
    activity_score = self._estimate_activity_level()
    
    return {
        'participation_breadth': {
            'total_club_memberships': total_clubs,
            'leadership_positions': leadership_roles,
            'leadership_ratio': round((leadership_roles / total_clubs) * 100, 1) if total_clubs > 0 else 0,
            'multi_club_engagement': total_clubs > 1
        },
        'participation_depth': {
            'years_in_current_club': current_club_memberships,
            'club_loyalty_score': min(current_club_memberships * 25, 100),
            'long_term_commitment': current_club_memberships >= 2
        },
        'activity_level': {
            'estimated_activity_score': activity_score,
            'activity_category': self._categorize_activity_level(activity_score),
            'engagement_consistency': self._assess_engagement_consistency()
        }
    }

def _estimate_activity_level(self):
    """Helper method to estimate member's activity level"""
    designation_activity_weights = {
        'member': 10,
        'active_member': 20,
        'volunteer': 30,
        'team_member': 40,
        'executive': 50,
        'secretary': 60,
        'coordinator': 70,
        'convenor': 80,
        'head': 90,
        'captain': 95
    }
    
    base_score = designation_activity_weights.get(self.designation.lower(), 10)
    
    # Adjust based on membership duration and club involvement
    all_memberships = Club_member.objects.filter(member=self.member)
    experience_bonus = min(all_memberships.count() * 5, 20)
    
    total_score = min(base_score + experience_bonus, 100)
    return total_score

def _categorize_activity_level(self, activity_score):
    """Helper method to categorize activity level"""
    if activity_score >= 80:
        return 'highly_active'
    elif activity_score >= 60:
        return 'active'
    elif activity_score >= 40:
        return 'moderately_active'
    elif activity_score >= 20:
        return 'somewhat_active'
    else:
        return 'minimally_active'

def _assess_engagement_consistency(self):
    """Helper method to assess engagement consistency"""
    member_memberships = Club_member.objects.filter(member=self.member).order_by('year')
    
    if member_memberships.count() <= 1:
        return 'insufficient_data'
    
    # Check for gaps in membership years
    years = [int(m.year) for m in member_memberships]
    year_gaps = []
    
    for i in range(1, len(years)):
        gap = years[i] - years[i-1]
        if gap > 1:
            year_gaps.append(gap - 1)
    
    if not year_gaps:
        return 'consistent'
    elif sum(year_gaps) <= 1:
        return 'mostly_consistent'
    else:
        return 'inconsistent'

def _assess_member_impact(self):
    """Helper method to assess member's impact on club and activities"""
    # Impact assessment based on role and involvement
    impact_metrics = {
        'leadership_impact': self._calculate_leadership_impact(),
        'club_contribution': self._assess_club_contribution(),
        'peer_influence': self._evaluate_peer_influence(),
        'institutional_visibility': self._assess_institutional_visibility()
    }
    
    # Overall impact score
    impact_score = sum(impact_metrics.values()) / len(impact_metrics)
    
    return {
        'impact_metrics': impact_metrics,
        'overall_impact_score': round(impact_score, 1),
        'impact_level': self._determine_impact_level(impact_score),
        'impact_areas': self._identify_key_impact_areas(impact_metrics)
    }

def _calculate_leadership_impact(self):
    """Helper method to calculate leadership impact"""
    leadership_roles = Club_member.objects.filter(
        member=self.member,
        designation__in=['coordinator', 'convenor', 'secretary', 'head', 'captain']
    ).count()
    
    if leadership_roles == 0:
        return 20  # Base score for participation
    elif leadership_roles == 1:
        return 60  # Single leadership role
    elif leadership_roles <= 3:
        return 80  # Multiple leadership roles
    else:
        return 95  # Extensive leadership experience

def _assess_club_contribution(self):
    """Helper method to assess contribution to club activities"""
    # Based on current designation and tenure
    designation_contribution = {
        'member': 30,
        'active_member': 40,
        'volunteer': 50,
        'team_member': 60,
        'executive': 70,
        'secretary': 75,
        'coordinator': 85,
        'convenor': 90,
        'head': 95,
        'captain': 95
    }
    
    base_contribution = designation_contribution.get(self.designation.lower(), 30)
    
    # Tenure bonus
    club_memberships = Club_member.objects.filter(
        member=self.member,
        club=self.club
    ).count()
    
    tenure_bonus = min(club_memberships * 5, 15)
    
    return min(base_contribution + tenure_bonus, 100)

def _evaluate_peer_influence(self):
    """Helper method to evaluate influence on peers"""
    # Simplified peer influence based on leadership roles and multi-club participation
    all_memberships = Club_member.objects.filter(member=self.member)
    
    leadership_influence = all_memberships.filter(
        designation__in=['coordinator', 'convenor', 'secretary', 'head', 'captain']
    ).count() * 20
    
    multi_club_influence = min(all_memberships.values('club').distinct().count() * 10, 30)
    
    return min(leadership_influence + multi_club_influence + 20, 100)  # Base 20 for participation

def _assess_institutional_visibility(self):
    """Helper method to assess institutional visibility"""
    # Based on high-level roles and multi-club leadership
    high_level_roles = Club_member.objects.filter(
        member=self.member,
        designation__in=['convenor', 'head', 'captain']
    ).count()
    
    if high_level_roles >= 2:
        return 90  # High institutional visibility
    elif high_level_roles == 1:
        return 70  # Moderate visibility
    else:
        return 40  # Limited visibility

def _determine_impact_level(self, impact_score):
    """Helper method to determine overall impact level"""
    if impact_score >= 85:
        return 'transformational'
    elif impact_score >= 70:
        return 'significant'
    elif impact_score >= 55:
        return 'moderate'
    elif impact_score >= 40:
        return 'emerging'
    else:
        return 'developing'

def _identify_key_impact_areas(self, impact_metrics):
    """Helper method to identify key areas of impact"""
    # Sort impact areas by score
    sorted_areas = sorted(impact_metrics.items(), key=lambda x: x[1], reverse=True)
    
    key_areas = []
    for area, score in sorted_areas:
        if score >= 70:
            key_areas.append(area)
    
    return key_areas if key_areas else ['developing_impact_areas']

def _identify_development_opportunities(self):
    """Helper method to identify member development opportunities"""
    opportunities = []
    
    # Leadership development opportunities
    leadership_analysis = self._analyze_leadership_progression()
    if leadership_analysis['leadership_readiness']['readiness_level'] in ['ready', 'highly_ready']:
        opportunities.append('leadership_role_progression')
    
    # Skill development opportunities
    current_designation = self.designation.lower()
    if current_designation in ['member', 'active_member']:
        opportunities.extend(['skill_building_workshops', 'project_participation'])
    elif current_designation in ['volunteer', 'team_member']:
        opportunities.extend(['leadership_training', 'cross_functional_experience'])
    
    # Cross-club opportunities
    member_clubs = Club_member.objects.filter(member=self.member).values('club').distinct().count()
    if member_clubs == 1:
        opportunities.append('explore_other_club_activities')
    
    # Mentoring opportunities
    leadership_roles = Club_member.objects.filter(
        member=self.member,
        designation__in=['coordinator', 'convenor', 'secretary', 'head', 'captain']
    ).count()
    
    if leadership_roles > 0:
        opportunities.append('mentor_junior_members')
    
    return opportunities

def _analyze_retention_risk(self):
    """Helper method to analyze member retention risk"""
    # Factors affecting retention
    risk_factors = []
    protective_factors = []
    
    # Engagement level assessment
    activity_score = self._estimate_activity_level()
    if activity_score < 40:
        risk_factors.append('low_engagement_level')
    else:
        protective_factors.append('good_engagement_level')
    
    # Role satisfaction assessment
    leadership_readiness = self._assess_leadership_readiness([])['readiness_level']
    current_level = self._get_current_role_level()
    
    if leadership_readiness in ['ready', 'highly_ready'] and current_level < 6:
        risk_factors.append('leadership_role_mismatch')
    
    # Multi-club commitment
    club_count = Club_member.objects.filter(member=self.member).values('club').distinct().count()
    if club_count > 3:
        risk_factors.append('overcommitment_risk')
    elif club_count >= 2:
        protective_factors.append('balanced_multi_club_engagement')
    
    # Calculate retention risk score
    risk_score = len(risk_factors) * 25
    protection_score = len(protective_factors) * 20
    
    retention_probability = max(10, 100 - risk_score + protection_score)
    
    return {
        'retention_probability': min(retention_probability, 95),
        'risk_level': self._determine_retention_risk_level(retention_probability),
        'risk_factors': risk_factors,
        'protective_factors': protective_factors,
        'intervention_recommendations': self._generate_retention_interventions(risk_factors)
    }

def _get_current_role_level(self):
    """Helper method to get current role level"""
    designation_hierarchy = {
        'member': 1, 'active_member': 2, 'volunteer': 3, 'team_member': 4,
        'executive': 5, 'secretary': 6, 'coordinator': 7, 'convenor': 8,
        'head': 9, 'captain': 10
    }
    return designation_hierarchy.get(self.designation.lower(), 1)

def _determine_retention_risk_level(self, retention_probability):
    """Helper method to determine retention risk level"""
    if retention_probability >= 85:
        return 'low_risk'
    elif retention_probability >= 70:
        return 'moderate_risk'
    elif retention_probability >= 50:
        return 'high_risk'
    else:
        return 'critical_risk'

def _generate_retention_interventions(self, risk_factors):
    """Helper method to generate retention intervention recommendations"""
    interventions = []
    
    if 'low_engagement_level' in risk_factors:
        interventions.extend([
            'increase_meaningful_task_assignments',
            'provide_skill_development_opportunities',
            'enhance_recognition_programs'
        ])
    
    if 'leadership_role_mismatch' in risk_factors:
        interventions.extend([
            'fast_track_leadership_development',
            'create_project_leadership_opportunities',
            'mentorship_program_enrollment'
        ])
    
    if 'overcommitment_risk' in risk_factors:
        interventions.extend([
            'workload_assessment_and_balancing',
            'time_management_support',
            'priority_setting_guidance'
        ])
    
    return interventions

def _generate_member_recommendations(self):
    """Helper method to generate comprehensive member development recommendations"""
    recommendations = []
    
    # Performance-based recommendations
    activity_level = self._categorize_activity_level(self._estimate_activity_level())
    
    if activity_level == 'minimally_active':
        recommendations.extend([
            'increase_event_participation',
            'volunteer_for_club_activities',
            'attend_skill_building_sessions'
        ])
    elif activity_level in ['somewhat_active', 'moderately_active']:
        recommendations.extend([
            'take_on_project_responsibilities',
            'develop_specialized_skills',
            'consider_leadership_roles'
        ])
    elif activity_level in ['active', 'highly_active']:
        recommendations.extend([
            'mentor_new_members',
            'lead_major_initiatives',
            'represent_club_externally'
        ])
    
    # Role-specific recommendations
    current_level = self._get_current_role_level()
    if current_level <= 3:
        recommendations.append('focus_on_skill_development_and_networking')
    elif current_level <= 6:
        recommendations.append('prepare_for_senior_leadership_roles')
    else:
        recommendations.append('develop_strategic_thinking_and_institutional_perspective')
    
    # Development opportunities
    recommendations.extend([
        'create_personal_development_plan',
        'seek_feedback_from_seniors',
        'document_achievements_and_learnings'
    ])
    
    return recommendations

def calculate_membership_duration(self):
    """
    CORE LOGIC: Calculate and analyze membership duration and tenure
    
    HOW IT WORKS:
    1. Calculates total membership duration across all clubs
    2. Analyzes tenure patterns and consistency
    3. Provides insights on member loyalty and commitment
    4. Compares with club average tenure
    
    BUSINESS PURPOSE:
    - Member loyalty and commitment assessment
    - Tenure-based recognition and benefits
    - Retention pattern analysis
    - Long-term engagement planning
    """
    current_year = datetime.datetime.now().year
    membership_start_year = int(self.year)
    
    # Basic duration calculation
    base_duration = current_year - membership_start_year + 1
    
    # Get all memberships for comprehensive analysis
    all_memberships = Club_member.objects.filter(member=self.member).order_by('year')
    
    # Calculate total institutional involvement
    first_membership_year = int(all_memberships.first().year) if all_memberships.exists() else membership_start_year
    total_institutional_duration = current_year - first_membership_year + 1
    
    # Calculate club-specific tenure
    club_memberships = Club_member.objects.filter(
        member=self.member,
        club=self.club
    ).order_by('year')
    
    club_tenure_years = [int(m.year) for m in club_memberships]
    club_tenure_duration = len(club_tenure_years)
    
    # Analyze tenure consistency
    tenure_analysis = self._analyze_tenure_consistency(club_tenure_years)
    
    # Compare with club averages
    club_comparison = self._compare_with_club_average_tenure()
    
    return {
        'duration_metrics': {
            'current_membership_duration': base_duration,
            'total_institutional_duration': total_institutional_duration,
            'club_specific_tenure': club_tenure_duration,
            'membership_start_year': self.year,
            'first_institutional_involvement': first_membership_year
        },
        'tenure_analysis': tenure_analysis,
        'club_comparison': club_comparison,
        'tenure_category': self._categorize_tenure(club_tenure_duration),
        'loyalty_metrics': self._calculate_loyalty_metrics(club_tenure_duration, total_institutional_duration)
    }

def _analyze_tenure_consistency(self, tenure_years):
    """Helper method to analyze tenure consistency"""
    if len(tenure_years) <= 1:
        return {'status': 'single_year_or_new_member'}
    
    # Check for gaps in tenure
    gaps = []
    for i in range(1, len(tenure_years)):
        gap = tenure_years[i] - tenure_years[i-1]
        if gap > 1:
            gaps.append(gap - 1)
    
    consistency_score = 100 if not gaps else max(0, 100 - sum(gaps) * 20)
    
    return {
        'consistency_score': consistency_score,
        'tenure_gaps': gaps,
        'consecutive_years': len(tenure_years),
        'consistency_level': 'high' if consistency_score >= 80 else 'moderate' if consistency_score >= 60 else 'low'
    }

def _compare_with_club_average_tenure(self):
    """Helper method to compare member tenure with club average"""
    # Calculate club average tenure
    club_members = Club_member.objects.filter(club=self.club)
    
    if not club_members.exists():
        return {'status': 'no_comparison_data'}
    
    current_year = datetime.datetime.now().year
    
    # Calculate each member's tenure
    member_tenures = []
    for member in club_members:
        member_start_year = int(member.year)
        member_tenure = current_year - member_start_year + 1
        member_tenures.append(member_tenure)
    
    average_tenure = sum(member_tenures) / len(member_tenures)
    median_tenure = sorted(member_tenures)[len(member_tenures) // 2]
    
    # Current member's tenure
    current_member_tenure = current_year - int(self.year) + 1
    
    return {
        'club_average_tenure': round(average_tenure, 1),
        'club_median_tenure': median_tenure,
        'member_tenure': current_member_tenure,
        'above_average': current_member_tenure > average_tenure,
        'tenure_percentile': self._calculate_tenure_percentile(current_member_tenure, member_tenures),
        'tenure_rank': sorted(member_tenures, reverse=True).index(current_member_tenure) + 1
    }

def _calculate_tenure_percentile(self, member_tenure, all_tenures):
    """Helper method to calculate tenure percentile"""
    shorter_tenures = sum(1 for t in all_tenures if t < member_tenure)
    percentile = (shorter_tenures / len(all_tenures)) * 100
    return round(percentile, 1)

def _categorize_tenure(self, tenure_duration):
    """Helper method to categorize member tenure"""
    if tenure_duration >= 4:
        return 'veteran_member'
    elif tenure_duration >= 3:
        return 'senior_member'
    elif tenure_duration >= 2:
        return 'experienced_member'
    else:
        return 'new_member'

def _calculate_loyalty_metrics(self, club_tenure, total_tenure):
    """Helper method to calculate member loyalty metrics"""
    # Club loyalty (focus on single club vs multi-club)
    club_loyalty_score = (club_tenure / total_tenure) * 100 if total_tenure > 0 else 0
    
    # Institutional loyalty (overall commitment to extracurricular activities)
    institutional_loyalty_score = min(total_tenure * 25, 100)  # Max 4 years = 100%
    
    # Combined loyalty assessment
    overall_loyalty = (club_loyalty_score * 0.6 + institutional_loyalty_score * 0.4)
    
    return {
        'club_loyalty_score': round(club_loyalty_score, 1),
        'institutional_loyalty_score': round(institutional_loyalty_score, 1),
        'overall_loyalty_score': round(overall_loyalty, 1),
        'loyalty_level': self._determine_loyalty_level(overall_loyalty),
        'loyalty_indicators': self._identify_loyalty_indicators(club_loyalty_score, institutional_loyalty_score)
    }

def _determine_loyalty_level(self, loyalty_score):
    """Helper method to determine loyalty level"""
    if loyalty_score >= 85:
        return 'highly_loyal'
    elif loyalty_score >= 70:
        return 'loyal'
    elif loyalty_score >= 55:
        return 'moderately_loyal'
    else:
        return 'developing_loyalty'

def _identify_loyalty_indicators(self, club_loyalty, institutional_loyalty):
    """Helper method to identify specific loyalty indicators"""
    indicators = []
    
    if club_loyalty >= 80:
        indicators.append('strong_club_commitment')
    elif club_loyalty >= 60:
        indicators.append('good_club_focus')
    else:
        indicators.append('multi_club_engagement')
    
    if institutional_loyalty >= 80:
        indicators.append('long_term_institutional_commitment')
    elif institutional_loyalty >= 60:
        indicators.append('sustained_involvement')
    else:
        indicators.append('developing_institutional_connection')
    
    return indicators

@staticmethod
def get_club_membership_analytics(club_id):
    """
    CORE LOGIC: Comprehensive club membership analytics and insights
    
    HOW IT WORKS:
    1. Analyzes membership composition and diversity
    2. Tracks membership trends and patterns
    3. Evaluates member engagement and retention
    4. Provides strategic insights for club management
    
    BUSINESS PURPOSE:
    - Club membership strategy development
    - Member engagement optimization
    - Retention improvement initiatives
    - Leadership pipeline development
    """
    try:
        club = Club_info.objects.get(id=club_id)
        club_members = Club_member.objects.filter(club=club)
        
        if not club_members.exists():
            return {'status': 'no_members', 'club_name': club.club_name}
        
        # Membership composition analysis
        composition_analysis = Club_member._analyze_membership_composition(club_members)
        
        # Membership trends and growth
        trend_analysis = Club_member._analyze_membership_trends(club_members)
        
        # Member engagement and activity levels
        engagement_analysis = Club_member._analyze_member_engagement(club_members)
        
        # Leadership pipeline analysis
        leadership_analysis = Club_member._analyze_leadership_pipeline(club_members)
        
        # Retention and churn analysis
        retention_analysis = Club_member._analyze_member_retention(club_members)
        
        # Club health and performance metrics
        club_health = Club_member._assess_club_health(composition_analysis, engagement_analysis, leadership_analysis)
        
        return {
            'club_overview': {
                'club_name': club.club_name,
                'total_members': club_members.count(),
                'analysis_date': datetime.datetime.now().strftime('%Y-%m-%d'),
                'club_category': getattr(club, 'category', 'general')
            },
            'composition_analysis': composition_analysis,
            'trend_analysis': trend_analysis,
            'engagement_analysis': engagement_analysis,
            'leadership_analysis': leadership_analysis,
            'retention_analysis': retention_analysis,
            'club_health': club_health,
            'strategic_recommendations': Club_member._generate_club_recommendations(
                composition_analysis, engagement_analysis, leadership_analysis, retention_analysis
            )
        }
        
    except Club_info.DoesNotExist:
        return {'status': 'club_not_found', 'club_id': club_id}
    except Exception as e:
        return {'status': 'error', 'message': str(e)}

@staticmethod
def _analyze_membership_composition(club_members):
    """Helper method to analyze club membership composition"""
    total_members = club_members.count()
    
    # Designation distribution
    designation_distribution = club_members.values('designation').annotate(
        count=Count('id')
    ).order_by('-count')
    
    # Year-wise distribution
    year_distribution = club_members.values('year').annotate(
        count=Count('id')
    ).order_by('-year')
    
    # Leadership vs regular members
    leadership_designations = ['coordinator', 'convenor', 'secretary', 'head', 'captain']
    leadership_count = club_members.filter(designation__in=leadership_designations).count()
    regular_member_count = total_members - leadership_count
    
    # Calculate diversity metrics
    designation_diversity = len(designation_distribution)
    year_diversity = len(year_distribution)
    
    return {
        'total_members': total_members,
        'designation_distribution': list(designation_distribution),
        'year_distribution': list(year_distribution),
        'leadership_composition': {
            'leadership_members': leadership_count,
            'regular_members': regular_member_count,
            'leadership_ratio': round((leadership_count / total_members) * 100, 1) if total_members > 0 else 0
        },
        'diversity_metrics': {
            'designation_diversity': designation_diversity,
            'year_diversity': year_diversity,
            'composition_balance_score': Club_member._calculate_composition_balance_score(
                leadership_count, regular_member_count, designation_diversity
            )
        }
    }

@staticmethod
def _calculate_composition_balance_score(leadership_count, regular_count, designation_diversity):
    """Helper method to calculate composition balance score"""
    total = leadership_count + regular_count
    if total == 0:
        return 0
    
    # Ideal leadership ratio is around 15-25%
    leadership_ratio = leadership_count / total
    leadership_score = 100 - abs(0.2 - leadership_ratio) * 400  # Penalty for deviation from 20%
    leadership_score = max(0, leadership_score)
    
    # Diversity bonus
    diversity_score = min(designation_diversity * 15, 60)  # Max 4 different designations
    
    # Combined balance score
    balance_score = (leadership_score * 0.7 + diversity_score * 0.3)
    return round(max(0, balance_score), 1)
```

This completes Part 3 covering the Club_member model with comprehensive business logic including membership analytics, leadership progression tracking, participation metrics, and retention analysis. The model provides extensive insights for member development and club management.
