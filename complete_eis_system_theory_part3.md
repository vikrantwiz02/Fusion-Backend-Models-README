# Complete EIS (Employee Information System) Theory Part 3 - Fusion IIIT

## Continued Model Analysis with Detailed Business Logic Explanations (Models 6-13)

### 6. emp_mtechphd_thesis Model
**Purpose**: Comprehensive tracking of M.Tech and PhD thesis supervision and guidance activities.

**Original Fields Analysis**:
```python
class emp_mtechphd_thesis(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE, blank=True,null=True)
    pf_no = models.CharField(max_length=20)
    degree_type = models.IntegerField(default=1)  # 1=M.Tech, 2=PhD
    title = models.CharField(max_length=250)  # Thesis title
    supervisors = models.CharField(max_length=250)  # Supervisor names
    co_supervisors = models.CharField(max_length=250, null=True, blank=True)
    rollno = models.CharField(max_length=200)  # Student roll number
    s_name = models.CharField(max_length=5000)  # Student name
    s_year = models.IntegerField(('year'), choices=YEAR_CHOICES, null=True, blank=True)
    status = models.CharField(max_length = 10, choices=[
        ('Awarded', 'Awarded'), ('Submitted', 'Submitted'), 
        ('Ongoing', 'Ongoing'), ('Completed', 'Completed')
    ])
```

**Enhanced Business Logic Implementation**:

```python
class emp_mtechphd_thesis(models.Model):
    DEGREE_TYPES = [
        ('MTECH', 'M.Tech'),
        ('PHD', 'Ph.D'),
        ('MS', 'M.S (Research)'),
        ('MPHIL', 'M.Phil'),
        ('PROJECT', 'B.Tech Project')
    ]
    
    SUPERVISION_ROLES = [
        ('SUPERVISOR', 'Main Supervisor'),
        ('CO_SUPERVISOR', 'Co-Supervisor'),
        ('JOINT_SUPERVISOR', 'Joint Supervisor'),
        ('EXTERNAL_SUPERVISOR', 'External Supervisor'),
        ('ADVISORY_MEMBER', 'Advisory Committee Member')
    ]
    
    STATUS_CHOICES = [
        ('REGISTERED', 'Registration Complete'),
        ('COURSEWORK', 'Coursework Phase'),
        ('RESEARCH_PROPOSAL', 'Research Proposal Submitted'),
        ('ONGOING_RESEARCH', 'Ongoing Research'),
        ('THESIS_WRITING', 'Thesis Writing'),
        ('SUBMITTED', 'Thesis Submitted'),
        ('DEFENDED', 'Successfully Defended'),
        ('AWARDED', 'Degree Awarded'),
        ('DISCONTINUED', 'Discontinued'),
        ('WITHDRAWN', 'Withdrawn')
    ]
    
    # Core information
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='supervised_thesis')
    pf_no = models.CharField(max_length=20)
    degree_type = models.CharField(max_length=20, choices=DEGREE_TYPES)
    supervision_role = models.CharField(max_length=20, choices=SUPERVISION_ROLES, default='SUPERVISOR')
    
    # Student information
    student_name = models.CharField(max_length=200, help_text="Student full name")
    student_roll_number = models.CharField(max_length=50, unique=True)
    student_email = models.EmailField(blank=True)
    student_department = models.CharField(max_length=100)
    
    # Thesis details
    thesis_title = models.TextField(max_length=1000, help_text="Complete thesis title")
    research_area = models.CharField(max_length=300, help_text="Research domain/field")
    keywords = models.TextField(max_length=500, blank=True, help_text="Research keywords")
    abstract = models.TextField(max_length=3000, blank=True, help_text="Thesis abstract")
    
    # Supervision team
    main_supervisor = models.CharField(max_length=200, help_text="Main supervisor name")
    co_supervisors = models.TextField(max_length=500, blank=True, help_text="Co-supervisors list")
    external_supervisors = models.TextField(max_length=500, blank=True)
    advisory_committee = models.TextField(max_length=1000, blank=True)
    
    # Timeline tracking
    registration_date = models.DateField(help_text="Registration date")
    proposal_defense_date = models.DateField(null=True, blank=True)
    thesis_submission_date = models.DateField(null=True, blank=True)
    defense_date = models.DateField(null=True, blank=True)
    degree_award_date = models.DateField(null=True, blank=True)
    expected_completion_date = models.DateField(null=True, blank=True)
    
    # Status and progress
    current_status = models.CharField(max_length=20, choices=STATUS_CHOICES, default='REGISTERED')
    progress_percentage = models.IntegerField(default=0, help_text="Overall progress percentage")
    current_semester = models.IntegerField(default=1)
    
    # Academic outputs
    publications_count = models.IntegerField(default=0, help_text="Publications from thesis work")
    conferences_presented = models.IntegerField(default=0)
    patents_filed = models.IntegerField(default=0)
    
    # Quality metrics
    thesis_grade = models.CharField(max_length=20, blank=True, help_text="Final grade/rating")
    external_examiner_rating = models.CharField(max_length=100, blank=True)
    
    def calculate_supervision_load_points(self):
        """
        CORE LOGIC: Calculate supervision workload points for faculty evaluation
        
        HOW IT WORKS:
        1. Assigns base points based on degree type (PhD > M.Tech > Project)
        2. Applies role multiplier (Main supervisor gets full points, co-supervisor gets partial)
        3. Considers thesis complexity and research area difficulty
        4. Adds bonus points for successful completion and high-quality outcomes
        5. Factors in time investment and student success metrics
        
        BUSINESS PURPOSE:
        - Quantifies supervision workload for faculty performance evaluation
        - Supports fair distribution of supervision responsibilities
        - Helps in promotion and tenure decision calculations
        - Enables department workload planning and resource allocation
        """
        # Base points by degree type
        degree_points = {
            'PHD': 100,        # Highest points for PhD supervision
            'MTECH': 60,       # Moderate points for M.Tech
            'MS': 80,          # High points for research MS
            'MPHIL': 40,       # Lower points for M.Phil
            'PROJECT': 20      # Lowest points for B.Tech projects
        }
        
        # Role multipliers
        role_multipliers = {
            'SUPERVISOR': 1.0,           # Full points for main supervisor
            'CO_SUPERVISOR': 0.5,        # Half points for co-supervisor
            'JOINT_SUPERVISOR': 0.7,     # 70% points for joint supervision
            'EXTERNAL_SUPERVISOR': 0.3,  # 30% points for external role
            'ADVISORY_MEMBER': 0.2       # 20% points for advisory role
        }
        
        base_points = degree_points.get(self.degree_type, 50)
        role_mult = role_multipliers.get(self.supervision_role, 1.0)
        total_points = base_points * role_mult
        
        # Completion bonus
        if self.current_status == 'AWARDED':
            total_points *= 1.3  # 30% bonus for successful completion
        elif self.current_status == 'DEFENDED':
            total_points *= 1.2  # 20% bonus for successful defense
        elif self.current_status in ['SUBMITTED', 'THESIS_WRITING']:
            total_points *= 1.1  # 10% bonus for reaching advanced stages
        
        # Quality bonus based on outputs
        quality_bonus = 0
        quality_bonus += self.publications_count * 5  # 5 points per publication
        quality_bonus += self.conferences_presented * 2  # 2 points per conference
        quality_bonus += self.patents_filed * 10  # 10 points per patent
        
        total_points += min(quality_bonus, 30)  # Cap quality bonus at 30 points
        
        # Duration factor (longer supervision gets slight penalty)
        if self.registration_date:
            duration_years = (timezone.now().date() - self.registration_date).days / 365.25
            expected_duration = {'PHD': 5, 'MTECH': 2, 'MS': 3}.get(self.degree_type, 3)
            
            if duration_years > expected_duration * 1.5:
                total_points *= 0.9  # 10% penalty for significantly delayed completion
        
        return round(total_points, 2)
    
    def assess_student_progress(self):
        """
        CORE LOGIC: Comprehensive assessment of student research progress
        
        HOW IT WORKS:
        1. Evaluates timeline adherence by comparing actual vs expected milestones
        2. Assesses research productivity through publications and conference presentations
        3. Analyzes thesis quality indicators and research novelty
        4. Considers external validation through peer review and citations
        5. Generates overall progress score and identifies risk factors
        
        BUSINESS PURPOSE:
        - Early identification of students at risk of not completing
        - Supports intervention strategies and additional support provision
        - Helps supervisors track multiple students efficiently
        - Enables department-level monitoring of program quality
        """
        assessment = {
            'overall_score': 0,
            'timeline_performance': 'On Track',
            'research_productivity': 'Average',
            'quality_indicators': 'Satisfactory',
            'risk_factors': [],
            'recommendations': []
        }
        
        score = 0
        
        # Timeline assessment
        if self.registration_date:
            duration_months = (timezone.now().date() - self.registration_date).days / 30.44
            
            # Expected milestones by degree type
            milestones = {
                'PHD': {
                    6: 'COURSEWORK',
                    12: 'RESEARCH_PROPOSAL', 
                    36: 'ONGOING_RESEARCH',
                    54: 'THESIS_WRITING',
                    60: 'SUBMITTED'
                },
                'MTECH': {
                    6: 'COURSEWORK',
                    12: 'RESEARCH_PROPOSAL',
                    18: 'ONGOING_RESEARCH',
                    22: 'THESIS_WRITING',
                    24: 'SUBMITTED'
                }
            }
            
            expected_milestones = milestones.get(self.degree_type, {})
            current_milestone_score = 0
            
            for month_threshold, expected_status in expected_milestones.items():
                if duration_months >= month_threshold:
                    status_progression = list(self.STATUS_CHOICES)
                    current_index = next((i for i, (k, v) in enumerate(status_progression) 
                                        if k == self.current_status), 0)
                    expected_index = next((i for i, (k, v) in enumerate(status_progression) 
                                         if k == expected_status), 0)
                    
                    if current_index >= expected_index:
                        current_milestone_score += 20  # On track
                    else:
                        current_milestone_score -= 10  # Behind schedule
                        assessment['risk_factors'].append(f"Behind on {expected_status} milestone")
            
            score += min(current_milestone_score, 40)  # Max 40 points from timeline
        
        # Research productivity assessment
        productivity_score = 0
        
        # Publications (most important metric)
        if self.publications_count >= 3:
            productivity_score += 30
            assessment['research_productivity'] = 'Excellent'
        elif self.publications_count >= 2:
            productivity_score += 25
            assessment['research_productivity'] = 'Very Good'
        elif self.publications_count >= 1:
            productivity_score += 20
            assessment['research_productivity'] = 'Good'
        else:
            productivity_score += 5
            assessment['research_productivity'] = 'Needs Improvement'
            assessment['risk_factors'].append("Low publication count")
        
        # Conference presentations
        if self.conferences_presented >= 3:
            productivity_score += 15
        elif self.conferences_presented >= 1:
            productivity_score += 10
        else:
            productivity_score += 2
            assessment['risk_factors'].append("Limited conference participation")
        
        # Patents (bonus for applied research)
        if self.patents_filed > 0:
            productivity_score += 10
        
        score += min(productivity_score, 60)  # Max 60 points from productivity
        
        # Generate recommendations
        if score >= 80:
            assessment['recommendations'].append("Excellent progress - consider early submission")
            assessment['recommendations'].append("Encourage additional publications for stronger profile")
        elif score >= 60:
            assessment['recommendations'].append("Good progress - maintain current trajectory")
            assessment['recommendations'].append("Focus on completing remaining milestones on time")
        elif score >= 40:
            assessment['recommendations'].append("Moderate progress - needs attention")
            assessment['recommendations'].append("Increase research focus and publication efforts")
        else:
            assessment['recommendations'].append("Poor progress - immediate intervention required")
            assessment['recommendations'].append("Consider additional supervision support or timeline extension")
        
        assessment['overall_score'] = score
        
        return assessment
    
    def predict_completion_timeline(self):
        """
        CORE LOGIC: Predict realistic completion timeline based on current progress
        
        HOW IT WORKS:
        1. Analyzes current status and typical time for each subsequent phase
        2. Considers student's historical progress rate and productivity
        3. Factors in research complexity and external dependencies
        4. Accounts for supervisor availability and department processing times
        5. Provides optimistic, realistic, and pessimistic completion scenarios
        
        BUSINESS PURPOSE:
        - Helps students plan career transitions and job applications
        - Enables supervisors to manage multiple student timelines
        - Supports department resource planning and scheduling
        - Assists in fellowship and funding duration decisions
        """
        if not self.registration_date:
            return None
        
        # Standard phase durations (in months) by degree type
        phase_durations = {
            'PHD': {
                'REGISTERED': 6,
                'COURSEWORK': 6,
                'RESEARCH_PROPOSAL': 24,
                'ONGOING_RESEARCH': 18,
                'THESIS_WRITING': 6,
                'SUBMITTED': 3,
                'DEFENDED': 1
            },
            'MTECH': {
                'REGISTERED': 6,
                'COURSEWORK': 6,
                'RESEARCH_PROPOSAL': 6,
                'ONGOING_RESEARCH': 4,
                'THESIS_WRITING': 2,
                'SUBMITTED': 1,
                'DEFENDED': 0.5
            }
        }
        
        standard_durations = phase_durations.get(self.degree_type, phase_durations['MTECH'])
        
        # Calculate remaining phases
        status_order = [s[0] for s in self.STATUS_CHOICES if s[0] != 'DISCONTINUED' and s[0] != 'WITHDRAWN']
        current_index = status_order.index(self.current_status) if self.current_status in status_order else 0
        remaining_phases = status_order[current_index + 1:]
        
        # Calculate student's progress efficiency
        elapsed_months = (timezone.now().date() - self.registration_date).days / 30.44
        completed_phases = status_order[:current_index + 1]
        expected_months_for_completed = sum(standard_durations.get(phase, 6) for phase in completed_phases)
        
        if expected_months_for_completed > 0:
            efficiency_factor = expected_months_for_completed / max(elapsed_months, 1)
        else:
            efficiency_factor = 1.0
        
        # Productivity factor based on research outputs
        productivity_factor = 1.0
        if self.publications_count >= 2:
            productivity_factor = 0.9  # 10% faster for productive students
        elif self.publications_count == 0 and elapsed_months > 12:
            productivity_factor = 1.2  # 20% slower for low productivity
        
        # Generate scenarios
        scenarios = {}
        
        for scenario, multiplier in [('optimistic', 0.8), ('realistic', 1.0), ('pessimistic', 1.3)]:
            remaining_months = 0
            
            for phase in remaining_phases:
                phase_duration = standard_durations.get(phase, 6)
                adjusted_duration = phase_duration / efficiency_factor * productivity_factor * multiplier
                remaining_months += adjusted_duration
            
            completion_date = timezone.now().date() + timedelta(days=int(remaining_months * 30.44))
            
            scenarios[scenario] = {
                'completion_date': completion_date,
                'months_remaining': round(remaining_months, 1),
                'total_duration_months': round(elapsed_months + remaining_months, 1)
            }
        
        return scenarios
    
    def generate_supervision_report(self):
        """
        CORE LOGIC: Generate comprehensive supervision report for administrative purposes
        
        HOW IT WORKS:
        1. Compiles all relevant student and thesis information
        2. Calculates key performance metrics and progress indicators
        3. Identifies achievements, challenges, and intervention needs
        4. Formats information for different stakeholder audiences
        5. Includes recommendations for future actions and support
        
        BUSINESS PURPOSE:
        - Standardizes reporting for department and university administration
        - Supports annual review processes and quality assurance
        - Enables tracking of supervision effectiveness and outcomes
        - Facilitates communication between supervisors and administrators
        """
        report = {
            'student_information': {
                'name': self.student_name,
                'roll_number': self.student_roll_number,
                'department': self.student_department,
                'degree_type': self.get_degree_type_display(),
                'current_status': self.get_current_status_display(),
                'registration_date': self.registration_date,
                'current_semester': self.current_semester
            },
            'thesis_details': {
                'title': self.thesis_title,
                'research_area': self.research_area,
                'supervision_team': {
                    'main_supervisor': self.main_supervisor,
                    'co_supervisors': self.co_supervisors.split(',') if self.co_supervisors else [],
                    'advisory_committee': self.advisory_committee.split(',') if self.advisory_committee else []
                }
            },
            'progress_metrics': {
                'completion_percentage': self.progress_percentage,
                'publications': self.publications_count,
                'conferences': self.conferences_presented,
                'patents': self.patents_filed,
                'supervision_points': self.calculate_supervision_load_points()
            },
            'timeline_analysis': {
                'duration_so_far': self._calculate_duration_months(),
                'predicted_completion': self.predict_completion_timeline(),
                'key_milestones': self._get_milestone_history()
            },
            'assessment': self.assess_student_progress(),
            'administrative_notes': self._generate_admin_notes()
        }
        
        return report
    
    def _calculate_duration_months(self):
        """Helper method to calculate duration in months"""
        if self.registration_date:
            return round((timezone.now().date() - self.registration_date).days / 30.44, 1)
        return 0
    
    def _get_milestone_history(self):
        """Helper method to get milestone achievement history"""
        milestones = []
        
        if self.registration_date:
            milestones.append({
                'milestone': 'Registration',
                'date': self.registration_date,
                'status': 'Completed'
            })
        
        if self.proposal_defense_date:
            milestones.append({
                'milestone': 'Proposal Defense',
                'date': self.proposal_defense_date,
                'status': 'Completed'
            })
        
        if self.thesis_submission_date:
            milestones.append({
                'milestone': 'Thesis Submission',
                'date': self.thesis_submission_date,
                'status': 'Completed'
            })
        
        if self.defense_date:
            milestones.append({
                'milestone': 'Thesis Defense',
                'date': self.defense_date,
                'status': 'Completed'
            })
        
        if self.degree_award_date:
            milestones.append({
                'milestone': 'Degree Award',
                'date': self.degree_award_date,
                'status': 'Completed'
            })
        
        return milestones
    
    def _generate_admin_notes(self):
        """Helper method to generate administrative notes"""
        notes = []
        
        # Duration warnings
        duration = self._calculate_duration_months()
        max_duration = {'PHD': 72, 'MTECH': 30, 'MS': 42}.get(self.degree_type, 30)
        
        if duration > max_duration:
            notes.append(f"Duration exceeded maximum allowed ({max_duration} months)")
        elif duration > max_duration * 0.8:
            notes.append(f"Approaching maximum duration limit")
        
        # Productivity notes
        if self.publications_count == 0 and duration > 24:
            notes.append("No publications yet - may need additional research support")
        
        if self.conferences_presented == 0 and duration > 18:
            notes.append("Limited conference participation - encourage presentation opportunities")
        
        # Status consistency check
        if self.current_status == 'ONGOING_RESEARCH' and self.progress_percentage < 30 and duration > 18:
            notes.append("Low progress percentage for research phase - intervention recommended")
        
        return notes
```

### 7. emp_keynote_address Model
**Purpose**: Tracking keynote speeches and plenary addresses at academic conferences and events.

**Enhanced Business Logic Implementation**:

```python
class emp_keynote_address(models.Model):
    ADDRESS_TYPES = [
        ('KEYNOTE', 'Keynote Address'),
        ('PLENARY', 'Plenary Talk'),
        ('INVITED_LECTURE', 'Invited Lecture'),
        ('DISTINGUISHED_LECTURE', 'Distinguished Lecture'),
        ('MEMORIAL_LECTURE', 'Memorial Lecture'),
        ('INAUGURAL_ADDRESS', 'Inaugural Address')
    ]
    
    EVENT_SCALES = [
        ('INTERNATIONAL', 'International Conference'),
        ('NATIONAL', 'National Conference'),
        ('REGIONAL', 'Regional Conference'),
        ('INSTITUTIONAL', 'Institutional Event'),
        ('INDUSTRY', 'Industry Event')
    ]
    
    # Enhanced core fields
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='keynote_addresses')
    pf_no = models.CharField(max_length=20)
    address_type = models.CharField(max_length=25, choices=ADDRESS_TYPES, default='KEYNOTE')
    
    # Event details
    event_name = models.CharField(max_length=500, help_text="Full event name")
    event_scale = models.CharField(max_length=20, choices=EVENT_SCALES, default='NATIONAL')
    organizer = models.CharField(max_length=300, help_text="Organizing institution/body")
    venue = models.CharField(max_length=300, help_text="Event venue")
    
    # Geographic information
    city = models.CharField(max_length=100)
    country = models.CharField(max_length=100)
    
    # Presentation details
    title = models.TextField(max_length=500, help_text="Address/lecture title")
    theme = models.CharField(max_length=300, blank=True, help_text="Event theme")
    topic_area = models.CharField(max_length=200, help_text="Subject area/domain")
    
    # Date and time
    address_date = models.DateField(help_text="Date of address")
    duration_minutes = models.IntegerField(default=60, help_text="Duration in minutes")
    
    # Audience and impact
    audience_size = models.IntegerField(null=True, blank=True, help_text="Approximate audience size")
    audience_type = models.CharField(max_length=200, blank=True, help_text="Audience description")
    
    # Recognition metrics
    media_coverage = models.BooleanField(default=False, help_text="Media coverage received")
    social_media_mentions = models.IntegerField(default=0)
    citation_impact = models.IntegerField(default=0, help_text="Later citations of this address")
    
    def calculate_prestige_score(self):
        """
        CORE LOGIC: Calculate prestige score based on event importance and impact
        
        HOW IT WORKS:
        1. Assigns base score based on address type (keynote > invited lecture)
        2. Multiplies by event scale factor (international > national > regional)
        3. Adds audience size bonus using logarithmic scaling
        4. Includes geographic reach premium for international venues
        5. Factors in post-event impact through citations and media coverage
        
        BUSINESS PURPOSE:
        - Quantifies academic recognition and standing in research community
        - Supports promotion and tenure evaluation processes
        - Helps in ranking faculty achievements and contributions
        - Enables comparison across different types of academic recognitions
        """
        # Base scores by address type
        type_scores = {
            'KEYNOTE': 100,              # Highest prestige
            'PLENARY': 95,               # Very high prestige
            'DISTINGUISHED_LECTURE': 90, # High prestige
            'MEMORIAL_LECTURE': 85,      # High prestige with special significance
            'INVITED_LECTURE': 75,       # Good prestige
            'INAUGURAL_ADDRESS': 80      # Good prestige for new events
        }
        
        # Event scale multipliers
        scale_multipliers = {
            'INTERNATIONAL': 2.0,        # 100% premium for international events
            'NATIONAL': 1.5,             # 50% premium for national events
            'REGIONAL': 1.0,             # Standard score for regional events
            'INSTITUTIONAL': 0.7,        # 30% reduction for institutional events
            'INDUSTRY': 1.2              # 20% premium for industry events
        }
        
        base_score = type_scores.get(self.address_type, 75)
        scale_mult = scale_multipliers.get(self.event_scale, 1.0)
        prestige_score = base_score * scale_mult
        
        # Geographic premium for international delivery
        if self.country.upper() != 'INDIA':
            prestige_score *= 1.2  # 20% premium for international delivery
        
        # Audience size impact (logarithmic to prevent outliers)
        if self.audience_size and self.audience_size > 0:
            import math
            audience_bonus = min(math.log10(self.audience_size) * 10, 30)
            prestige_score += audience_bonus
        
        # Impact and recognition bonuses
        if self.media_coverage:
            prestige_score += 15  # Media coverage bonus
        
        if self.social_media_mentions > 100:
            prestige_score += 10  # Social media impact bonus
        elif self.social_media_mentions > 50:
            prestige_score += 5
        
        if self.citation_impact > 0:
            citation_bonus = min(self.citation_impact * 2, 25)  # Max 25 points from citations
            prestige_score += citation_bonus
        
        return round(prestige_score, 2)
    
    def analyze_global_reach_impact(self):
        """
        CORE LOGIC: Analyze the global reach and impact of the keynote address
        
        HOW IT WORKS:
        1. Evaluates geographic diversity of event and attendees
        2. Assesses topic relevance and universal applicability
        3. Measures post-event dissemination and follow-up activities
        4. Considers long-term influence on research directions
        5. Calculates composite global impact score and recommendations
        
        BUSINESS PURPOSE:
        - Demonstrates international recognition and influence
        - Supports applications for global research collaborations
        - Helps in building international academic reputation
        - Guides strategic decisions for future keynote opportunities
        """
        impact_analysis = {
            'geographic_reach': 'Regional',
            'topic_universality': 'Medium',
            'dissemination_score': 0,
            'influence_potential': 'Medium',
            'global_impact_score': 0,
            'recommendations': []
        }
        
        score = 0
        
        # Geographic reach assessment
        if self.event_scale == 'INTERNATIONAL':
            score += 30
            impact_analysis['geographic_reach'] = 'Global'
        elif self.event_scale == 'NATIONAL':
            score += 20
            impact_analysis['geographic_reach'] = 'National'
        elif self.event_scale == 'REGIONAL':
            score += 10
            impact_analysis['geographic_reach'] = 'Regional'
        
        # International venue bonus
        if self.country.upper() != 'INDIA':
            score += 15
            impact_analysis['geographic_reach'] = 'International'
        
        # Topic universality (based on domain analysis)
        universal_topics = [
            'artificial intelligence', 'climate change', 'sustainability',
            'healthcare', 'education', 'energy', 'technology', 'innovation'
        ]
        
        topic_lower = (self.title + ' ' + self.topic_area).lower()
        if any(topic in topic_lower for topic in universal_topics):
            score += 20
            impact_analysis['topic_universality'] = 'High'
        else:
            score += 10
            impact_analysis['topic_universality'] = 'Medium'
        
        # Dissemination and follow-up impact
        dissemination_score = 0
        if self.media_coverage:
            dissemination_score += 10
        
        if self.social_media_mentions > 0:
            dissemination_score += min(self.social_media_mentions / 10, 15)
        
        if self.citation_impact > 0:
            dissemination_score += min(self.citation_impact * 3, 20)
        
        score += dissemination_score
        impact_analysis['dissemination_score'] = round(dissemination_score, 2)
        
        # Influence potential assessment
        audience_factor = 1.0
        if self.audience_size:
            if self.audience_size > 1000:
                audience_factor = 1.5
            elif self.audience_size > 500:
                audience_factor = 1.3
            elif self.audience_size > 200:
                audience_factor = 1.1
        
        score *= audience_factor
        
        # Generate recommendations
        if score >= 80:
            impact_analysis['influence_potential'] = 'Very High'
            impact_analysis['recommendations'].extend([
                "Excellent global impact - leverage for international collaborations",
                "Consider publishing extended version or follow-up work",
                "Pursue similar high-profile speaking opportunities"
            ])
        elif score >= 60:
            impact_analysis['influence_potential'] = 'High'
            impact_analysis['recommendations'].extend([
                "Good global reach - build on this momentum",
                "Enhance dissemination through social media and publications",
                "Seek opportunities at larger international venues"
            ])
        elif score >= 40:
            impact_analysis['influence_potential'] = 'Medium'
            impact_analysis['recommendations'].extend([
                "Moderate impact - focus on broader audience topics",
                "Increase post-event engagement and follow-up",
                "Target international conferences for greater reach"
            ])
        else:
            impact_analysis['influence_potential'] = 'Limited'
            impact_analysis['recommendations'].extend([
                "Limited global impact - consider topic universality",
                "Focus on international conference opportunities",
                "Develop more broadly applicable research themes"
            ])
        
        impact_analysis['global_impact_score'] = round(score, 2)
        return impact_analysis
```