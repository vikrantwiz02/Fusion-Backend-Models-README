# Complete Gymkhana Module System Documentation - Fusion IIIT (Part 4)

## Database Models Analysis with Business Logic (Continued)

### 6. Core_team Model
**Purpose**: Core team management system that tracks leadership structures, roles, and administrative hierarchies within clubs.

**PostgreSQL Table**: `Core_team`

**Fields Structure**:
```python
class Core_team(models.Model):
    club = models.ForeignKey(Club_info, on_delete=models.CASCADE)
    member = models.ForeignKey(User, on_delete=models.CASCADE)
    year = models.CharField(max_length=4, default="2020")
    designation = models.CharField(max_length=30, default="convenor")
```

**Enhanced Business Methods**:
```python
def get_comprehensive_leadership_analytics(self):
    """
    CORE LOGIC: Complete core team leadership and governance analytics
    
    HOW IT WORKS:
    1. Analyzes leadership structure and effectiveness
    2. Tracks governance patterns and decision-making hierarchies
    3. Evaluates team composition and balance
    4. Provides insights for leadership optimization
    
    BUSINESS PURPOSE:
    - Leadership effectiveness assessment and improvement
    - Governance structure optimization
    - Succession planning and leadership development
    - Team dynamics and collaboration enhancement
    """
    # Leadership profile and authority analysis
    leadership_profile = self._analyze_leadership_profile()
    
    # Team structure and hierarchy assessment
    team_structure = self._analyze_team_structure()
    
    # Leadership effectiveness metrics
    effectiveness_metrics = self._calculate_leadership_effectiveness()
    
    # Governance and decision-making analysis
    governance_analysis = self._assess_governance_patterns()
    
    # Leadership development and succession planning
    succession_planning = self._analyze_succession_readiness()
    
    # Inter-team collaboration and coordination
    collaboration_metrics = self._evaluate_inter_team_collaboration()
    
    return {
        'leadership_overview': {
            'member_id': self.member.id,
            'leader_name': self.member.get_full_name(),
            'club_name': self.club.club_name,
            'current_designation': self.designation,
            'leadership_year': self.year,
            'leadership_tenure': self._calculate_leadership_tenure()
        },
        'leadership_profile': leadership_profile,
        'team_structure': team_structure,
        'effectiveness_metrics': effectiveness_metrics,
        'governance_analysis': governance_analysis,
        'succession_planning': succession_planning,
        'collaboration_metrics': collaboration_metrics,
        'strategic_recommendations': self._generate_leadership_recommendations()
    }

def _analyze_leadership_profile(self):
    """Helper method to analyze comprehensive leadership profile"""
    # Leadership background and experience
    all_leadership_roles = Core_team.objects.filter(member=self.member)
    
    # Calculate leadership experience metrics
    total_leadership_years = all_leadership_roles.count()
    leadership_clubs = all_leadership_roles.values('club').distinct().count()
    
    # Designation hierarchy analysis
    designation_hierarchy = {
        'member': 1, 'coordinator': 2, 'secretary': 3, 'convenor': 4, 'head': 5, 'captain': 6
    }
    
    current_level = designation_hierarchy.get(self.designation.lower(), 1)
    highest_level = max([designation_hierarchy.get(role.designation.lower(), 1) for role in all_leadership_roles])
    
    # Leadership progression analysis
    leadership_progression = self._analyze_leadership_progression_pattern()
    
    # Multi-club leadership assessment
    multi_club_leadership = self._assess_multi_club_leadership()
    
    return {
        'experience_metrics': {
            'total_leadership_years': total_leadership_years,
            'leadership_clubs': leadership_clubs,
            'current_level': current_level,
            'highest_level_achieved': highest_level,
            'leadership_breadth_score': min(leadership_clubs * 25, 100)
        },
        'leadership_progression': leadership_progression,
        'multi_club_leadership': multi_club_leadership,
        'leadership_style': self._assess_leadership_style(),
        'authority_scope': self._evaluate_authority_scope()
    }

def _analyze_leadership_progression_pattern(self):
    """Helper method to analyze leadership progression patterns"""
    leadership_roles = Core_team.objects.filter(member=self.member).order_by('year')
    
    progression_path = []
    designation_levels = {
        'coordinator': 2, 'secretary': 3, 'convenor': 4, 'head': 5, 'captain': 6
    }
    
    for role in leadership_roles:
        level = designation_levels.get(role.designation.lower(), 1)
        progression_path.append({
            'year': role.year,
            'club': role.club.club_name,
            'designation': role.designation,
            'level': level,
            'is_promotion': False  # Will be calculated
        })
    
    # Identify promotions and progression
    for i in range(1, len(progression_path)):
        if progression_path[i]['level'] > progression_path[i-1]['level']:
            progression_path[i]['is_promotion'] = True
    
    # Calculate progression metrics
    promotions = sum(1 for p in progression_path if p['is_promotion'])
    progression_rate = promotions / len(progression_path) if progression_path else 0
    
    return {
        'progression_path': progression_path,
        'total_promotions': promotions,
        'progression_rate': round(progression_rate * 100, 1),
        'leadership_trajectory': self._determine_leadership_trajectory(progression_path),
        'career_velocity': self._calculate_leadership_velocity(progression_path)
    }

def _determine_leadership_trajectory(self, progression_path):
    """Helper method to determine leadership trajectory"""
    if len(progression_path) < 2:
        return 'insufficient_data'
    
    levels = [p['level'] for p in progression_path]
    
    # Analyze trend
    if levels[-1] > levels[0]:
        return 'ascending'
    elif levels[-1] < levels[0]:
        return 'descending'
    else:
        # Check for stability vs fluctuation
        if len(set(levels)) == 1:
            return 'stable'
        else:
            return 'fluctuating'

def _calculate_leadership_velocity(self, progression_path):
    """Helper method to calculate leadership development velocity"""
    if len(progression_path) < 2:
        return 'insufficient_data'
    
    years_span = int(progression_path[-1]['year']) - int(progression_path[0]['year'])
    level_change = progression_path[-1]['level'] - progression_path[0]['level']
    
    if years_span == 0:
        return 'single_year_data'
    
    velocity = level_change / years_span
    
    if velocity > 1.5:
        return 'rapid_advancement'
    elif velocity > 0.5:
        return 'steady_advancement'
    elif velocity > 0:
        return 'gradual_advancement'
    else:
        return 'stable_or_declining'

def _assess_multi_club_leadership(self):
    """Helper method to assess multi-club leadership capabilities"""
    leadership_clubs = Core_team.objects.filter(member=self.member).values('club').distinct()
    club_count = leadership_clubs.count()
    
    if club_count <= 1:
        return {
            'multi_club_leader': False,
            'club_diversity_score': 25,
            'leadership_breadth': 'single_club_focus'
        }
    
    # Analyze club categories if available
    club_diversity = self._calculate_club_category_diversity(leadership_clubs)
    
    # Leadership effectiveness across clubs
    cross_club_effectiveness = self._assess_cross_club_effectiveness()
    
    return {
        'multi_club_leader': True,
        'leadership_clubs_count': club_count,
        'club_diversity_score': min(club_count * 25, 100),
        'leadership_breadth': 'multi_club_expertise',
        'club_diversity': club_diversity,
        'cross_club_effectiveness': cross_club_effectiveness
    }

def _calculate_club_category_diversity(self, leadership_clubs):
    """Helper method to calculate diversity across club categories"""
    # This would integrate with club category information
    # For now, return a simplified assessment
    club_count = leadership_clubs.count()
    
    if club_count >= 4:
        return 'highly_diverse'
    elif club_count >= 3:
        return 'moderately_diverse'
    elif club_count == 2:
        return 'limited_diversity'
    else:
        return 'single_club'

def _assess_cross_club_effectiveness(self):
    """Helper method to assess effectiveness across multiple clubs"""
    # Simplified assessment based on designation levels
    leadership_roles = Core_team.objects.filter(member=self.member)
    
    designation_levels = {
        'coordinator': 2, 'secretary': 3, 'convenor': 4, 'head': 5, 'captain': 6
    }
    
    avg_level = sum(designation_levels.get(role.designation.lower(), 1) for role in leadership_roles) / leadership_roles.count()
    
    if avg_level >= 4:
        return 'high_effectiveness'
    elif avg_level >= 3:
        return 'moderate_effectiveness'
    else:
        return 'developing_effectiveness'

def _assess_leadership_style(self):
    """Helper method to assess leadership style based on roles and progression"""
    current_designation = self.designation.lower()
    leadership_history = Core_team.objects.filter(member=self.member)
    
    # Leadership style indicators
    style_indicators = []
    
    # Collaborative vs Hierarchical
    if leadership_history.filter(designation__in=['coordinator', 'secretary']).exists():
        style_indicators.append('collaborative_oriented')
    
    if leadership_history.filter(designation__in=['convenor', 'head', 'captain']).exists():
        style_indicators.append('authority_oriented')
    
    # Multi-club leadership indicates adaptability
    if leadership_history.values('club').distinct().count() > 1:
        style_indicators.append('adaptable_leadership')
    
    # Long tenure indicates stability
    if leadership_history.count() >= 3:
        style_indicators.append('stable_leadership')
    
    return {
        'style_indicators': style_indicators,
        'primary_style': self._determine_primary_leadership_style(style_indicators),
        'leadership_adaptability': 'high' if 'adaptable_leadership' in style_indicators else 'moderate'
    }

def _determine_primary_leadership_style(self, style_indicators):
    """Helper method to determine primary leadership style"""
    if 'authority_oriented' in style_indicators and 'collaborative_oriented' in style_indicators:
        return 'balanced_leadership'
    elif 'authority_oriented' in style_indicators:
        return 'directive_leadership'
    elif 'collaborative_oriented' in style_indicators:
        return 'participative_leadership'
    else:
        return 'developing_leadership_style'

def _evaluate_authority_scope(self):
    """Helper method to evaluate scope of authority and influence"""
    current_designation = self.designation.lower()
    
    authority_levels = {
        'coordinator': {'scope': 'team_level', 'influence': 'moderate', 'decision_power': 'limited'},
        'secretary': {'scope': 'club_level', 'influence': 'moderate', 'decision_power': 'moderate'},
        'convenor': {'scope': 'club_level', 'influence': 'high', 'decision_power': 'high'},
        'head': {'scope': 'institutional_level', 'influence': 'high', 'decision_power': 'high'},
        'captain': {'scope': 'institutional_level', 'influence': 'very_high', 'decision_power': 'very_high'}
    }
    
    authority_profile = authority_levels.get(current_designation, {
        'scope': 'limited', 'influence': 'low', 'decision_power': 'minimal'
    })
    
    # Multi-club authority assessment
    leadership_roles = Core_team.objects.filter(member=self.member)
    total_authority_score = sum(self._get_authority_score(role.designation) for role in leadership_roles)
    
    return {
        'current_authority': authority_profile,
        'total_authority_score': total_authority_score,
        'authority_breadth': leadership_roles.values('club').distinct().count(),
        'institutional_influence': self._assess_institutional_influence(total_authority_score)
    }

def _get_authority_score(self, designation):
    """Helper method to get numerical authority score for designation"""
    authority_scores = {
        'coordinator': 20, 'secretary': 30, 'convenor': 50, 'head': 70, 'captain': 80
    }
    return authority_scores.get(designation.lower(), 10)

def _assess_institutional_influence(self, total_authority_score):
    """Helper method to assess institutional influence level"""
    if total_authority_score >= 100:
        return 'high_institutional_influence'
    elif total_authority_score >= 60:
        return 'moderate_institutional_influence'
    elif total_authority_score >= 30:
        return 'limited_institutional_influence'
    else:
        return 'minimal_institutional_influence'

def _analyze_team_structure(self):
    """Helper method to analyze team structure and composition"""
    club_core_team = Core_team.objects.filter(club=self.club, year=self.year)
    
    # Team size and composition
    team_size = club_core_team.count()
    
    # Designation distribution
    designation_distribution = club_core_team.values('designation').annotate(
        count=Count('id')
    ).order_by('-count')
    
    # Leadership hierarchy analysis
    hierarchy_levels = self._analyze_team_hierarchy(club_core_team)
    
    # Team balance and diversity
    team_balance = self._assess_team_balance(designation_distribution)
    
    # Team coordination and reporting structure
    coordination_structure = self._analyze_coordination_structure(club_core_team)
    
    return {
        'team_composition': {
            'total_team_size': team_size,
            'designation_distribution': list(designation_distribution),
            'team_size_category': self._categorize_team_size(team_size)
        },
        'hierarchy_analysis': hierarchy_levels,
        'team_balance': team_balance,
        'coordination_structure': coordination_structure,
        'team_effectiveness_score': self._calculate_team_effectiveness_score(team_size, hierarchy_levels, team_balance)
    }

def _analyze_team_hierarchy(self, core_team):
    """Helper method to analyze team hierarchy structure"""
    hierarchy_map = {
        'captain': 6, 'head': 5, 'convenor': 4, 'secretary': 3, 'coordinator': 2
    }
    
    hierarchy_distribution = {}
    for member in core_team:
        level = hierarchy_map.get(member.designation.lower(), 1)
        if level not in hierarchy_distribution:
            hierarchy_distribution[level] = 0
        hierarchy_distribution[level] += 1
    
    # Calculate hierarchy metrics
    hierarchy_levels = len(hierarchy_distribution)
    top_level_leaders = hierarchy_distribution.get(6, 0) + hierarchy_distribution.get(5, 0)
    mid_level_leaders = hierarchy_distribution.get(4, 0) + hierarchy_distribution.get(3, 0)
    coordinator_level = hierarchy_distribution.get(2, 0)
    
    return {
        'hierarchy_levels': hierarchy_levels,
        'level_distribution': hierarchy_distribution,
        'leadership_structure': {
            'top_level_leaders': top_level_leaders,
            'mid_level_leaders': mid_level_leaders,
            'coordinators': coordinator_level
        },
        'hierarchy_balance_score': self._calculate_hierarchy_balance(hierarchy_distribution)
    }

def _calculate_hierarchy_balance(self, hierarchy_distribution):
    """Helper method to calculate hierarchy balance score"""
    total_members = sum(hierarchy_distribution.values())
    if total_members == 0:
        return 0
    
    # Ideal distribution: fewer at top, more at middle levels
    ideal_ratios = {6: 0.1, 5: 0.15, 4: 0.25, 3: 0.3, 2: 0.2}
    
    balance_score = 0
    for level, count in hierarchy_distribution.items():
        actual_ratio = count / total_members
        ideal_ratio = ideal_ratios.get(level, 0.2)
        deviation = abs(actual_ratio - ideal_ratio)
        level_score = max(0, 100 - deviation * 200)  # Penalty for deviation
        balance_score += level_score * (count / total_members)  # Weight by proportion
    
    return round(balance_score, 1)

def _assess_team_balance(self, designation_distribution):
    """Helper method to assess team balance and diversity"""
    total_positions = sum(item['count'] for item in designation_distribution)
    
    # Calculate diversity score
    unique_designations = len(designation_distribution)
    diversity_score = min(unique_designations * 20, 100)  # Max 5 designations
    
    # Calculate concentration risk
    max_concentration = max(item['count'] for item in designation_distribution) if designation_distribution else 0
    concentration_ratio = max_concentration / total_positions if total_positions > 0 else 0
    
    # Balance assessment
    if concentration_ratio > 0.6:
        balance_level = 'imbalanced'
    elif concentration_ratio > 0.4:
        balance_level = 'somewhat_imbalanced'
    else:
        balance_level = 'balanced'
    
    return {
        'diversity_score': diversity_score,
        'unique_designations': unique_designations,
        'concentration_ratio': round(concentration_ratio * 100, 1),
        'balance_level': balance_level,
        'balance_recommendations': self._generate_balance_recommendations(balance_level, concentration_ratio)
    }

def _generate_balance_recommendations(self, balance_level, concentration_ratio):
    """Helper method to generate team balance recommendations"""
    recommendations = []
    
    if balance_level == 'imbalanced':
        recommendations.extend([
            'redistribute_responsibilities_across_team',
            'create_additional_specialized_roles',
            'establish_deputy_positions'
        ])
    
    if concentration_ratio > 0.5:
        recommendations.append('reduce_role_concentration_risk')
    
    recommendations.extend([
        'regular_team_structure_review',
        'succession_planning_for_key_roles'
    ])
    
    return recommendations

def _analyze_coordination_structure(self, core_team):
    """Helper method to analyze team coordination and communication structure"""
    # Reporting relationships analysis
    designation_hierarchy = {
        'captain': 6, 'head': 5, 'convenor': 4, 'secretary': 3, 'coordinator': 2
    }
    
    reporting_structure = {}
    communication_channels = 0
    
    for member in core_team:
        level = designation_hierarchy.get(member.designation.lower(), 1)
        if level not in reporting_structure:
            reporting_structure[level] = []
        reporting_structure[level].append(member.designation)
    
    # Calculate coordination complexity
    coordination_complexity = len(reporting_structure) * core_team.count()
    
    return {
        'reporting_structure': reporting_structure,
        'coordination_complexity': coordination_complexity,
        'communication_efficiency': self._assess_communication_efficiency(reporting_structure),
        'coordination_recommendations': self._generate_coordination_recommendations(coordination_complexity)
    }

def _assess_communication_efficiency(self, reporting_structure):
    """Helper method to assess communication efficiency"""
    hierarchy_levels = len(reporting_structure)
    
    if hierarchy_levels <= 2:
        return 'high_efficiency'
    elif hierarchy_levels <= 3:
        return 'moderate_efficiency'
    else:
        return 'complex_structure'

def _generate_coordination_recommendations(self, coordination_complexity):
    """Helper method to generate coordination recommendations"""
    recommendations = []
    
    if coordination_complexity > 50:
        recommendations.extend([
            'simplify_reporting_structure',
            'establish_clear_communication_protocols',
            'implement_team_coordination_tools'
        ])
    
    recommendations.extend([
        'regular_team_coordination_meetings',
        'define_clear_role_boundaries',
        'establish_decision_making_processes'
    ])
    
    return recommendations

def _calculate_team_effectiveness_score(self, team_size, hierarchy_levels, team_balance):
    """Helper method to calculate overall team effectiveness score"""
    # Size effectiveness (optimal range 5-12 members)
    if 5 <= team_size <= 12:
        size_score = 100
    elif 3 <= team_size <= 15:
        size_score = 80
    else:
        size_score = 60
    
    # Hierarchy effectiveness
    hierarchy_score = hierarchy_levels['hierarchy_balance_score']
    
    # Balance effectiveness
    balance_score = team_balance['diversity_score']
    
    # Combined effectiveness score
    effectiveness_score = (size_score * 0.3 + hierarchy_score * 0.4 + balance_score * 0.3)
    
    return round(effectiveness_score, 1)

def _categorize_team_size(self, team_size):
    """Helper method to categorize team size"""
    if team_size <= 3:
        return 'small_team'
    elif team_size <= 8:
        return 'optimal_team'
    elif team_size <= 15:
        return 'large_team'
    else:
        return 'very_large_team'

def _calculate_leadership_effectiveness(self):
    """Helper method to calculate leadership effectiveness metrics"""
    # Performance indicators based on role and tenure
    leadership_tenure = self._calculate_leadership_tenure()
    
    # Effectiveness factors
    experience_factor = min(leadership_tenure * 20, 60)  # Max 3 years = 60 points
    authority_factor = self._get_authority_score(self.designation)
    
    # Multi-club leadership bonus
    multi_club_roles = Core_team.objects.filter(member=self.member).values('club').distinct().count()
    versatility_factor = min(multi_club_roles * 15, 30)
    
    # Team integration score
    team_integration = self._assess_team_integration()
    
    # Overall effectiveness calculation
    effectiveness_score = experience_factor + authority_factor + versatility_factor + team_integration
    effectiveness_score = min(effectiveness_score, 100)  # Cap at 100
    
    return {
        'overall_effectiveness_score': round(effectiveness_score, 1),
        'effectiveness_factors': {
            'experience_contribution': experience_factor,
            'authority_contribution': authority_factor,
            'versatility_contribution': versatility_factor,
            'team_integration_contribution': team_integration
        },
        'effectiveness_level': self._determine_effectiveness_level(effectiveness_score),
        'improvement_areas': self._identify_effectiveness_improvement_areas(effectiveness_score)
    }

def _assess_team_integration(self):
    """Helper method to assess integration within team"""
    # Simplified assessment based on position in hierarchy
    current_designation = self.designation.lower()
    
    integration_scores = {
        'captain': 15,  # High integration as top leader
        'head': 15,
        'convenor': 12,  # Good integration as senior role
        'secretary': 10,  # Moderate integration
        'coordinator': 8   # Basic integration
    }
    
    return integration_scores.get(current_designation, 5)

def _determine_effectiveness_level(self, effectiveness_score):
    """Helper method to determine effectiveness level"""
    if effectiveness_score >= 85:
        return 'highly_effective'
    elif effectiveness_score >= 70:
        return 'effective'
    elif effectiveness_score >= 55:
        return 'moderately_effective'
    else:
        return 'developing_effectiveness'

def _identify_effectiveness_improvement_areas(self, effectiveness_score):
    """Helper method to identify areas for effectiveness improvement"""
    improvement_areas = []
    
    if effectiveness_score < 70:
        improvement_areas.extend([
            'leadership_skill_development',
            'team_building_capabilities',
            'strategic_thinking_enhancement'
        ])
    
    # Role-specific improvements
    current_designation = self.designation.lower()
    
    if current_designation in ['coordinator', 'secretary']:
        improvement_areas.extend([
            'project_management_skills',
            'communication_enhancement'
        ])
    elif current_designation in ['convenor', 'head', 'captain']:
        improvement_areas.extend([
            'strategic_planning_skills',
            'stakeholder_management',
            'organizational_vision_development'
        ])
    
    return improvement_areas

def _calculate_leadership_tenure(self):
    """Helper method to calculate leadership tenure"""
    current_year = datetime.datetime.now().year
    leadership_start_year = int(self.year)
    return current_year - leadership_start_year + 1

def _assess_governance_patterns(self):
    """Helper method to assess governance and decision-making patterns"""
    # Governance assessment based on role and team structure
    club_core_team = Core_team.objects.filter(club=self.club, year=self.year)
    
    # Decision-making authority distribution
    authority_distribution = self._analyze_decision_authority_distribution(club_core_team)
    
    # Governance structure assessment
    governance_structure = self._evaluate_governance_structure()
    
    # Accountability and oversight mechanisms
    accountability_mechanisms = self._assess_accountability_mechanisms()
    
    return {
        'governance_structure': governance_structure,
        'authority_distribution': authority_distribution,
        'accountability_mechanisms': accountability_mechanisms,
        'governance_effectiveness': self._calculate_governance_effectiveness(
            governance_structure, authority_distribution, accountability_mechanisms
        )
    }

def _analyze_decision_authority_distribution(self, core_team):
    """Helper method to analyze decision-making authority distribution"""
    authority_levels = {
        'captain': 'strategic_decisions',
        'head': 'strategic_decisions', 
        'convenor': 'operational_decisions',
        'secretary': 'administrative_decisions',
        'coordinator': 'tactical_decisions'
    }
    
    authority_distribution = {}
    for member in core_team:
        authority_type = authority_levels.get(member.designation.lower(), 'limited_authority')
        if authority_type not in authority_distribution:
            authority_distribution[authority_type] = 0
        authority_distribution[authority_type] += 1
    
    return {
        'authority_levels': authority_distribution,
        'decision_concentration': self._calculate_decision_concentration(authority_distribution),
        'authority_balance': self._assess_authority_balance(authority_distribution)
    }

def _calculate_decision_concentration(self, authority_distribution):
    """Helper method to calculate decision-making concentration"""
    strategic_decisions = authority_distribution.get('strategic_decisions', 0)
    total_decision_makers = sum(authority_distribution.values())
    
    if total_decision_makers == 0:
        return 0
    
    concentration_ratio = strategic_decisions / total_decision_makers
    return round(concentration_ratio * 100, 1)

def _assess_authority_balance(self, authority_distribution):
    """Helper method to assess authority balance across levels"""
    total_authorities = sum(authority_distribution.values())
    
    if total_authorities == 0:
        return 'no_authority_structure'
    
    # Ideal: balanced distribution across different decision levels
    unique_levels = len(authority_distribution)
    
    if unique_levels >= 4:
        return 'well_distributed'
    elif unique_levels >= 3:
        return 'moderately_distributed'
    elif unique_levels >= 2:
        return 'limited_distribution'
    else:
        return 'concentrated_authority'

def _evaluate_governance_structure(self):
    """Helper method to evaluate overall governance structure"""
    current_designation = self.designation.lower()
    club_core_team = Core_team.objects.filter(club=self.club, year=self.year)
    
    # Governance model assessment
    team_size = club_core_team.count()
    
    if team_size <= 3:
        governance_model = 'simple_hierarchy'
    elif team_size <= 8:
        governance_model = 'functional_structure'
    else:
        governance_model = 'complex_organization'
    
    # Role clarity and structure
    unique_designations = club_core_team.values('designation').distinct().count()
    role_clarity_score = min(unique_designations * 20, 100)
    
    return {
        'governance_model': governance_model,
        'team_size': team_size,
        'role_clarity_score': role_clarity_score,
        'structural_complexity': self._assess_structural_complexity(team_size, unique_designations)
    }

def _assess_structural_complexity(self, team_size, unique_designations):
    """Helper method to assess structural complexity"""
    complexity_score = team_size + unique_designations
    
    if complexity_score <= 8:
        return 'low_complexity'
    elif complexity_score <= 15:
        return 'moderate_complexity'
    else:
        return 'high_complexity'

def _assess_accountability_mechanisms(self):
    """Helper method to assess accountability and oversight mechanisms"""
    current_designation = self.designation.lower()
    
    # Accountability level based on position
    accountability_levels = {
        'captain': {'level': 'high', 'scope': 'institutional'},
        'head': {'level': 'high', 'scope': 'institutional'},
        'convenor': {'level': 'moderate', 'scope': 'club'},
        'secretary': {'level': 'moderate', 'scope': 'administrative'},
        'coordinator': {'level': 'basic', 'scope': 'team'}
    }
    
    accountability_profile = accountability_levels.get(current_designation, {
        'level': 'minimal', 'scope': 'individual'
    })
    
    # Oversight responsibilities
    oversight_responsibilities = self._identify_oversight_responsibilities()
    
    return {
        'accountability_profile': accountability_profile,
        'oversight_responsibilities': oversight_responsibilities,
        'accountability_score': self._calculate_accountability_score(accountability_profile)
    }

def _identify_oversight_responsibilities(self):
    """Helper method to identify oversight responsibilities"""
    current_designation = self.designation.lower()
    
    oversight_map = {
        'captain': ['team_performance', 'strategic_direction', 'external_relations'],
        'head': ['team_performance', 'strategic_direction', 'institutional_compliance'],
        'convenor': ['team_coordination', 'project_execution', 'member_development'],
        'secretary': ['documentation', 'communication', 'administrative_compliance'],
        'coordinator': ['task_completion', 'team_coordination', 'basic_oversight']
    }
    
    return oversight_map.get(current_designation, ['individual_responsibilities'])

def _calculate_accountability_score(self, accountability_profile):
    """Helper method to calculate accountability score"""
    level_scores = {
        'high': 80,
        'moderate': 60,
        'basic': 40,
        'minimal': 20
    }
    
    scope_scores = {
        'institutional': 20,
        'club': 15,
        'administrative': 10,
        'team': 8,
        'individual': 5
    }
    
    level_score = level_scores.get(accountability_profile.get('level', 'minimal'), 20)
    scope_score = scope_scores.get(accountability_profile.get('scope', 'individual'), 5)
    
    return level_score + scope_score

def _calculate_governance_effectiveness(self, governance_structure, authority_distribution, accountability_mechanisms):
    """Helper method to calculate overall governance effectiveness"""
    # Structure effectiveness
    structure_score = governance_structure['role_clarity_score'] * 0.3
    
    # Authority distribution effectiveness
    authority_balance = authority_distribution['authority_balance']
    authority_score = {
        'well_distributed': 30,
        'moderately_distributed': 25,
        'limited_distribution': 15,
        'concentrated_authority': 10,
        'no_authority_structure': 0
    }.get(authority_balance, 10)
    
    # Accountability effectiveness
    accountability_score = accountability_mechanisms['accountability_score'] * 0.4
    
    # Combined governance effectiveness
    governance_effectiveness = structure_score + authority_score + accountability_score
    
    return {
        'overall_effectiveness_score': round(governance_effectiveness, 1),
        'effectiveness_level': self._determine_governance_effectiveness_level(governance_effectiveness),
        'improvement_recommendations': self._generate_governance_improvement_recommendations(
            governance_effectiveness, governance_structure, authority_distribution
        )
    }

def _determine_governance_effectiveness_level(self, effectiveness_score):
    """Helper method to determine governance effectiveness level"""
    if effectiveness_score >= 85:
        return 'highly_effective_governance'
    elif effectiveness_score >= 70:
        return 'effective_governance'
    elif effectiveness_score >= 55:
        return 'moderately_effective_governance'
    else:
        return 'governance_needs_improvement'

def _generate_governance_improvement_recommendations(self, effectiveness_score, governance_structure, authority_distribution):
    """Helper method to generate governance improvement recommendations"""
    recommendations = []
    
    if effectiveness_score < 70:
        recommendations.extend([
            'clarify_roles_and_responsibilities',
            'establish_clear_decision_making_processes',
            'improve_accountability_mechanisms'
        ])
    
    if governance_structure['structural_complexity'] == 'high_complexity':
        recommendations.append('simplify_organizational_structure')
    
    if authority_distribution['authority_balance'] in ['concentrated_authority', 'limited_distribution']:
        recommendations.append('redistribute_decision_making_authority')
    
    recommendations.extend([
        'implement_regular_governance_reviews',
        'establish_performance_monitoring_systems',
        'create_feedback_mechanisms'
    ])
    
    return recommendations
```

This completes the first part of the Core_team model documentation. The content is getting quite extensive, so I should continue with the remaining methods in the next part.
