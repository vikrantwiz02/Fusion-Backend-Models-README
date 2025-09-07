# Complete EIS (Employee Information System) Theory Part 4 - Fusion IIIT

## Final Model Analysis with Detailed Business Logic Explanations (Models 8-13)

### 8. emp_expert_lectures Model
**Purpose**: Tracking expert lectures and invited talks delivered by faculty members.

**Enhanced Business Logic Implementation**:

```python
class emp_expert_lectures(models.Model):
    LECTURE_TYPES = [
        ('EXPERT_LECTURE', 'Expert Lecture'),
        ('INVITED_TALK', 'Invited Talk'),
        ('GUEST_LECTURE', 'Guest Lecture'),
        ('TECHNICAL_SEMINAR', 'Technical Seminar'),
        ('WORKSHOP_SESSION', 'Workshop Session'),
        ('MASTERCLASS', 'Masterclass'),
        ('WEBINAR', 'Webinar')
    ]
    
    AUDIENCE_TYPES = [
        ('ACADEMIC', 'Academic Audience'),
        ('INDUSTRY', 'Industry Professionals'),
        ('STUDENTS', 'Students'),
        ('MIXED', 'Mixed Audience'),
        ('GOVERNMENT', 'Government Officials'),
        ('GENERAL_PUBLIC', 'General Public')
    ]
    
    DELIVERY_MODES = [
        ('IN_PERSON', 'In-Person'),
        ('VIRTUAL', 'Virtual/Online'),
        ('HYBRID', 'Hybrid'),
        ('RECORDED', 'Pre-recorded')
    ]
    
    # Core information
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='expert_lectures')
    pf_no = models.CharField(max_length=20)
    lecture_type = models.CharField(max_length=20, choices=LECTURE_TYPES, default='EXPERT_LECTURE')
    
    # Lecture details
    title = models.TextField(max_length=500, help_text="Lecture title")
    topic_area = models.CharField(max_length=200, help_text="Subject domain")
    abstract = models.TextField(max_length=1500, blank=True, help_text="Lecture abstract")
    
    # Event information
    host_institution = models.CharField(max_length=300, help_text="Host organization")
    event_name = models.CharField(max_length=300, blank=True, help_text="Parent event if applicable")
    venue = models.CharField(max_length=200, help_text="Venue location")
    city = models.CharField(max_length=100)
    country = models.CharField(max_length=100)
    
    # Delivery details
    lecture_date = models.DateField(help_text="Date of lecture")
    duration_minutes = models.IntegerField(default=60, help_text="Duration in minutes")
    delivery_mode = models.CharField(max_length=15, choices=DELIVERY_MODES, default='IN_PERSON')
    
    # Audience information
    audience_type = models.CharField(max_length=15, choices=AUDIENCE_TYPES, default='ACADEMIC')
    audience_size = models.IntegerField(null=True, blank=True, help_text="Number of attendees")
    audience_level = models.CharField(max_length=100, blank=True, help_text="Audience expertise level")
    
    # Impact and engagement
    questions_asked = models.IntegerField(default=0, help_text="Number of questions received")
    interaction_rating = models.IntegerField(null=True, blank=True, 
                                           choices=[(i, i) for i in range(1, 6)],
                                           help_text="Audience engagement rating (1-5)")
    
    # Follow-up activities
    recording_available = models.BooleanField(default=False)
    presentation_shared = models.BooleanField(default=False)
    follow_up_requests = models.IntegerField(default=0, help_text="Follow-up collaboration requests")
    
    def calculate_knowledge_dissemination_score(self):
        """
        CORE LOGIC: Calculate score for knowledge dissemination and outreach impact
        
        HOW IT WORKS:
        1. Evaluates audience reach and diversity (size, type, geographic spread)
        2. Assesses content quality and relevance to different stakeholder groups
        3. Measures engagement levels through interaction metrics and follow-ups
        4. Considers delivery effectiveness and accessibility (mode, recording, sharing)
        5. Weights factors to produce composite dissemination impact score
        
        BUSINESS PURPOSE:
        - Quantifies faculty contribution to knowledge transfer and public engagement
        - Supports evaluation of outreach activities for promotion decisions
        - Helps identify effective knowledge dissemination strategies
        - Enables measurement of university's societal impact through faculty expertise
        """
        dissemination_score = 0
        
        # Base score by lecture type
        type_scores = {
            'EXPERT_LECTURE': 80,      # High value for expert lectures
            'INVITED_TALK': 75,        # Good value for invited talks
            'MASTERCLASS': 90,         # Highest value for masterclasses
            'TECHNICAL_SEMINAR': 70,   # Good value for seminars
            'GUEST_LECTURE': 60,       # Moderate value for guest lectures
            'WORKSHOP_SESSION': 85,    # High value for hands-on workshops
            'WEBINAR': 65              # Moderate value for webinars
        }
        
        base_score = type_scores.get(self.lecture_type, 70)
        dissemination_score = base_score
        
        # Audience diversity and reach multiplier
        audience_multipliers = {
            'MIXED': 1.3,              # 30% bonus for diverse audience
            'INDUSTRY': 1.2,           # 20% bonus for industry outreach
            'GOVERNMENT': 1.25,        # 25% bonus for policy impact
            'GENERAL_PUBLIC': 1.4,     # 40% bonus for public engagement
            'STUDENTS': 1.1,           # 10% bonus for educational impact
            'ACADEMIC': 1.0            # Standard score for academic audience
        }
        
        audience_mult = audience_multipliers.get(self.audience_type, 1.0)
        dissemination_score *= audience_mult
        
        # Audience size impact (logarithmic scaling)
        if self.audience_size and self.audience_size > 0:
            import math
            # Logarithmic bonus to prevent outliers from skewing scores
            size_bonus = min(math.log10(self.audience_size) * 15, 40)
            dissemination_score += size_bonus
        
        # Geographic reach premium
        if self.country.upper() != 'INDIA':
            dissemination_score *= 1.15  # 15% premium for international delivery
        
        # Engagement quality bonuses
        if self.interaction_rating:
            if self.interaction_rating >= 4:
                dissemination_score += 20  # High engagement bonus
            elif self.interaction_rating >= 3:
                dissemination_score += 10  # Moderate engagement bonus
        
        if self.questions_asked > 10:
            dissemination_score += 15  # Active Q&A session bonus
        elif self.questions_asked > 5:
            dissemination_score += 10
        elif self.questions_asked > 0:
            dissemination_score += 5
        
        # Accessibility and reach enhancement
        if self.recording_available:
            dissemination_score += 15  # Recording extends reach
        
        if self.presentation_shared:
            dissemination_score += 10  # Sharing materials enhances impact
        
        # Follow-up impact (indicates lasting value)
        if self.follow_up_requests > 0:
            follow_up_bonus = min(self.follow_up_requests * 5, 25)  # Max 25 points
            dissemination_score += follow_up_bonus
        
        # Delivery mode adjustment
        mode_factors = {
            'IN_PERSON': 1.0,          # Standard score for in-person
            'VIRTUAL': 0.9,            # Slight reduction for virtual
            'HYBRID': 1.1,             # Bonus for hybrid reaching more people
            'RECORDED': 0.8            # Reduction for asynchronous delivery
        }
        
        mode_factor = mode_factors.get(self.delivery_mode, 1.0)
        dissemination_score *= mode_factor
        
        return round(dissemination_score, 2)
    
    def analyze_audience_impact_diversity(self):
        """
        CORE LOGIC: Analyze diversity and breadth of audience impact
        
        HOW IT WORKS:
        1. Categorizes audience by professional background and expertise level
        2. Assesses geographic diversity and institutional representation
        3. Evaluates cross-sectoral impact (academic, industry, government, public)
        4. Measures knowledge transfer effectiveness to different stakeholder groups
        5. Generates diversity index and impact breadth metrics
        
        BUSINESS PURPOSE:
        - Demonstrates faculty's ability to communicate across sectors
        - Supports applications for public engagement awards and recognition
        - Helps identify faculty suitable for cross-sector collaboration projects
        - Enables measurement of university's broader societal engagement
        """
        impact_analysis = {
            'diversity_index': 0,
            'sector_reach': [],
            'geographic_spread': 'Local',
            'knowledge_transfer_effectiveness': 'Medium',
            'recommendations': []
        }
        
        diversity_score = 0
        
        # Audience type diversity scoring
        if self.audience_type == 'MIXED':
            diversity_score += 30  # Maximum diversity for mixed audiences
            impact_analysis['sector_reach'] = ['Academic', 'Industry', 'Students', 'Public']
        elif self.audience_type in ['INDUSTRY', 'GOVERNMENT']:
            diversity_score += 25  # High diversity for professional audiences
            impact_analysis['sector_reach'] = ['Professional', 'Academic']
        elif self.audience_type == 'GENERAL_PUBLIC':
            diversity_score += 20  # Good diversity for public engagement
            impact_analysis['sector_reach'] = ['General Public', 'Academic']
        else:
            diversity_score += 10  # Limited diversity for homogeneous audiences
            impact_analysis['sector_reach'] = [self.get_audience_type_display()]
        
        # Geographic spread assessment
        if self.country.upper() != 'INDIA':
            diversity_score += 20
            impact_analysis['geographic_spread'] = 'International'
        elif self.city.lower() not in ['hyderabad', 'secunderabad']:  # Assuming local context
            diversity_score += 10
            impact_analysis['geographic_spread'] = 'National'
        else:
            diversity_score += 5
            impact_analysis['geographic_spread'] = 'Local'
        
        # Delivery mode impact on reach
        if self.delivery_mode in ['VIRTUAL', 'HYBRID']:
            diversity_score += 15  # Virtual delivery increases geographic diversity
        
        if self.recording_available:
            diversity_score += 10  # Recording enables asynchronous diverse access
        
        # Engagement quality indicators
        if self.interaction_rating and self.interaction_rating >= 4:
            diversity_score += 15
            impact_analysis['knowledge_transfer_effectiveness'] = 'High'
        elif self.interaction_rating and self.interaction_rating >= 3:
            diversity_score += 10
            impact_analysis['knowledge_transfer_effectiveness'] = 'Medium'
        else:
            impact_analysis['knowledge_transfer_effectiveness'] = 'Needs Assessment'
        
        # Follow-up diversity (indicates cross-sector interest)
        if self.follow_up_requests > 5:
            diversity_score += 15
        elif self.follow_up_requests > 0:
            diversity_score += 10
        
        impact_analysis['diversity_index'] = round(diversity_score, 2)
        
        # Generate recommendations based on analysis
        if diversity_score >= 80:
            impact_analysis['recommendations'].extend([
                "Excellent audience diversity - model for other faculty",
                "Consider developing this topic into broader outreach program",
                "Leverage cross-sector connections for collaborative projects"
            ])
        elif diversity_score >= 60:
            impact_analysis['recommendations'].extend([
                "Good diversity reach - explore additional audience segments",
                "Consider virtual delivery to expand geographic reach",
                "Document best practices for knowledge transfer"
            ])
        elif diversity_score >= 40:
            impact_analysis['recommendations'].extend([
                "Moderate diversity - target different audience types",
                "Explore partnerships with industry/government organizations",
                "Develop versions of content for different stakeholder groups"
            ])
        else:
            impact_analysis['recommendations'].extend([
                "Limited diversity - focus on cross-sector engagement",
                "Participate in public engagement training programs",
                "Collaborate with outreach offices for broader audience access"
            ])
        
        return impact_analysis
```

### 9. emp_event_organized Model
**Purpose**: Comprehensive tracking of events organized by faculty members including conferences, workshops, and training programs.

**Enhanced Business Logic Implementation**:

```python
class emp_event_organized(models.Model):
    EVENT_TYPES = [
        ('INTERNATIONAL_CONFERENCE', 'International Conference'),
        ('NATIONAL_CONFERENCE', 'National Conference'),
        ('WORKSHOP', 'Workshop'),
        ('TRAINING_PROGRAM', 'Training Program'),
        ('SEMINAR', 'Seminar'),
        ('SYMPOSIUM', 'Symposium'),
        ('SHORT_TERM_COURSE', 'Short Term Course'),
        ('HACKATHON', 'Hackathon'),
        ('COMPETITION', 'Competition'),
        ('WEBINAR_SERIES', 'Webinar Series')
    ]
    
    ORGANIZATION_ROLES = [
        ('CONVENER', 'Convener'),
        ('CO_CONVENER', 'Co-Convener'),
        ('COORDINATOR', 'Coordinator'),
        ('CHAIR', 'Conference Chair'),
        ('PROGRAM_CHAIR', 'Program Chair'),
        ('ORGANIZING_COMMITTEE', 'Organizing Committee Member'),
        ('ADVISORY_COMMITTEE', 'Advisory Committee Member')
    ]
    
    EVENT_SCALES = [
        ('INTERNATIONAL', 'International'),
        ('NATIONAL', 'National'),
        ('REGIONAL', 'Regional'),
        ('INSTITUTIONAL', 'Institutional'),
        ('DEPARTMENTAL', 'Departmental')
    ]
    
    # Core event information
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='organized_events')
    pf_no = models.CharField(max_length=20)
    event_type = models.CharField(max_length=25, choices=EVENT_TYPES)
    organization_role = models.CharField(max_length=25, choices=ORGANIZATION_ROLES, default='COORDINATOR')
    
    # Event details
    event_name = models.CharField(max_length=500, help_text="Complete event name")
    event_theme = models.CharField(max_length=300, blank=True, help_text="Event theme/focus")
    event_scale = models.CharField(max_length=15, choices=EVENT_SCALES, default='INSTITUTIONAL')
    
    # Organization details
    organizing_institution = models.CharField(max_length=300, help_text="Primary organizing body")
    co_organizers = models.TextField(max_length=1000, blank=True, help_text="Co-organizing institutions")
    sponsoring_agencies = models.TextField(max_length=1000, blank=True, help_text="Sponsoring organizations")
    
    # Venue and logistics
    venue = models.CharField(max_length=300, help_text="Event venue")
    city = models.CharField(max_length=100)
    country = models.CharField(max_length=100)
    
    # Timeline
    start_date = models.DateField(help_text="Event start date")
    end_date = models.DateField(help_text="Event end date")
    planning_start_date = models.DateField(null=True, blank=True, help_text="When planning began")
    
    # Participation metrics
    total_participants = models.IntegerField(null=True, blank=True)
    international_participants = models.IntegerField(default=0)
    industry_participants = models.IntegerField(default=0)
    academic_participants = models.IntegerField(default=0)
    student_participants = models.IntegerField(default=0)
    
    # Financial information
    total_budget = models.DecimalField(max_digits=12, decimal_places=2, null=True, blank=True)
    funding_secured = models.DecimalField(max_digits=12, decimal_places=2, default=0)
    registration_revenue = models.DecimalField(max_digits=12, decimal_places=2, default=0)
    
    # Quality indicators
    keynote_speakers = models.IntegerField(default=0, help_text="Number of keynote speakers")
    technical_sessions = models.IntegerField(default=0, help_text="Number of technical sessions")
    papers_presented = models.IntegerField(default=0, help_text="Papers/presentations")
    
    # Impact and outcomes
    proceedings_published = models.BooleanField(default=False)
    media_coverage = models.BooleanField(default=False)
    follow_up_activities = models.TextField(max_length=1000, blank=True)
    
    def calculate_organization_leadership_score(self):
        """
        CORE LOGIC: Calculate leadership and organizational capability score
        
        HOW IT WORKS:
        1. Assigns base points based on event type and scale (international > national > regional)
        2. Applies role multiplier (convener gets full points, committee member gets partial)
        3. Adds participation diversity bonus for international and cross-sector attendance
        4. Includes financial management bonus based on budget size and funding secured
        5. Factors in quality indicators like keynote speakers and proceedings publication
        
        BUSINESS PURPOSE:
        - Quantifies faculty's administrative and leadership capabilities
        - Supports evaluation for administrative positions and leadership roles
        - Demonstrates ability to manage complex projects and stakeholder relationships
        - Helps identify faculty suitable for institutional leadership development
        """
        leadership_score = 0
        
        # Base scores by event type and scale
        type_scale_matrix = {
            ('INTERNATIONAL_CONFERENCE', 'INTERNATIONAL'): 150,
            ('INTERNATIONAL_CONFERENCE', 'NATIONAL'): 120,
            ('NATIONAL_CONFERENCE', 'NATIONAL'): 100,
            ('NATIONAL_CONFERENCE', 'REGIONAL'): 80,
            ('WORKSHOP', 'INTERNATIONAL'): 90,
            ('WORKSHOP', 'NATIONAL'): 70,
            ('TRAINING_PROGRAM', 'NATIONAL'): 80,
            ('SEMINAR', 'INSTITUTIONAL'): 50,
            ('SYMPOSIUM', 'NATIONAL'): 85,
            ('HACKATHON', 'NATIONAL'): 75,
            ('WEBINAR_SERIES', 'INTERNATIONAL'): 60
        }
        
        # Get base score from matrix or calculate default
        base_score = type_scale_matrix.get((self.event_type, self.event_scale))
        if not base_score:
            # Calculate default score
            type_scores = {
                'INTERNATIONAL_CONFERENCE': 100, 'NATIONAL_CONFERENCE': 80,
                'WORKSHOP': 60, 'TRAINING_PROGRAM': 70, 'SEMINAR': 40,
                'SYMPOSIUM': 75, 'HACKATHON': 65, 'WEBINAR_SERIES': 50
            }
            scale_multipliers = {
                'INTERNATIONAL': 1.5, 'NATIONAL': 1.2, 'REGIONAL': 1.0,
                'INSTITUTIONAL': 0.8, 'DEPARTMENTAL': 0.6
            }
            base_score = (type_scores.get(self.event_type, 50) * 
                         scale_multipliers.get(self.event_scale, 1.0))
        
        leadership_score = base_score
        
        # Role responsibility multiplier
        role_multipliers = {
            'CONVENER': 1.0,                    # Full leadership responsibility
            'CO_CONVENER': 0.8,                 # Shared leadership
            'CHAIR': 1.0,                       # Full leadership as chair
            'PROGRAM_CHAIR': 0.9,               # High responsibility for content
            'COORDINATOR': 0.7,                 # Coordination responsibilities
            'ORGANIZING_COMMITTEE': 0.5,        # Committee participation
            'ADVISORY_COMMITTEE': 0.3           # Advisory role
        }
        
        role_mult = role_multipliers.get(self.organization_role, 0.7)
        leadership_score *= role_mult
        
        # Participation scale and diversity bonuses
        if self.total_participants:
            if self.total_participants > 1000:
                leadership_score += 40  # Large-scale event management
            elif self.total_participants > 500:
                leadership_score += 30
            elif self.total_participants > 200:
                leadership_score += 20
            elif self.total_participants > 100:
                leadership_score += 10
        
        # International participation bonus
        if self.international_participants > 0 and self.total_participants:
            intl_percentage = (self.international_participants / self.total_participants) * 100
            if intl_percentage > 30:
                leadership_score += 25  # Significant international draw
            elif intl_percentage > 15:
                leadership_score += 15
            elif intl_percentage > 5:
                leadership_score += 10
        
        # Cross-sector participation diversity
        if (self.industry_participants > 0 and self.academic_participants > 0 and 
            self.student_participants > 0):
            leadership_score += 20  # Excellent diversity
        elif (self.industry_participants > 0 and self.academic_participants > 0):
            leadership_score += 15  # Good industry-academia mix
        
        # Financial management capability
        if self.total_budget and self.funding_secured:
            funding_ratio = self.funding_secured / self.total_budget
            if funding_ratio >= 0.9:
                leadership_score += 25  # Excellent funding management
            elif funding_ratio >= 0.7:
                leadership_score += 20
            elif funding_ratio >= 0.5:
                leadership_score += 15
        
        # Quality and impact indicators
        if self.keynote_speakers > 5:
            leadership_score += 15  # High-profile speakers
        elif self.keynote_speakers > 2:
            leadership_score += 10
        
        if self.proceedings_published:
            leadership_score += 15  # Formal publication outcome
        
        if self.media_coverage:
            leadership_score += 10  # Public visibility
        
        if self.follow_up_activities:
            leadership_score += 10  # Sustained impact
        
        # Duration and complexity factor
        if self.start_date and self.end_date:
            duration_days = (self.end_date - self.start_date).days + 1
            if duration_days > 5:
                leadership_score += 15  # Multi-day event complexity
            elif duration_days > 2:
                leadership_score += 10
        
        return round(leadership_score, 2)
    
    def calculate_financial_efficiency_metrics(self):
        """
        CORE LOGIC: Calculate financial planning and management efficiency
        
        HOW IT WORKS:
        1. Analyzes cost per participant to assess budget efficiency
        2. Evaluates funding diversification and external support mobilization
        3. Calculates revenue generation capability through registrations
        4. Assesses cost recovery ratio and financial sustainability
        5. Benchmarks against industry standards for similar events
        
        BUSINESS PURPOSE:
        - Demonstrates financial management capabilities for future funding
        - Supports applications for larger grant funding and institutional support
        - Helps optimize resource allocation for future events
        - Provides transparency and accountability for sponsored events
        """
        if not self.total_budget or self.total_budget <= 0:
            return {'error': 'No budget information available'}
        
        metrics = {
            'cost_per_participant': 0,
            'funding_efficiency': 0,
            'cost_recovery_ratio': 0,
            'financial_sustainability_score': 0,
            'budget_utilization_assessment': 'Pending'
        }
        
        # Cost per participant analysis
        if self.total_participants and self.total_participants > 0:
            metrics['cost_per_participant'] = float(self.total_budget) / self.total_participants
            
            # Benchmark against event type standards
            cost_benchmarks = {
                'INTERNATIONAL_CONFERENCE': 15000,  # INR per participant
                'NATIONAL_CONFERENCE': 8000,
                'WORKSHOP': 5000,
                'TRAINING_PROGRAM': 12000,
                'SEMINAR': 3000,
                'SYMPOSIUM': 10000
            }
            
            benchmark = cost_benchmarks.get(self.event_type, 8000)
            if metrics['cost_per_participant'] <= benchmark:
                metrics['budget_utilization_assessment'] = 'Efficient'
            elif metrics['cost_per_participant'] <= benchmark * 1.2:
                metrics['budget_utilization_assessment'] = 'Acceptable'
            else:
                metrics['budget_utilization_assessment'] = 'Needs Optimization'
        
        # Funding efficiency (external funding as % of total budget)
        if self.funding_secured > 0:
            metrics['funding_efficiency'] = (float(self.funding_secured) / float(self.total_budget)) * 100
        
        # Cost recovery through registration revenue
        if self.registration_revenue > 0:
            metrics['cost_recovery_ratio'] = (float(self.registration_revenue) / float(self.total_budget)) * 100
        
        # Overall financial sustainability score
        sustainability_score = 0
        
        # Funding diversification score
        if metrics['funding_efficiency'] > 70:
            sustainability_score += 30  # Excellent external funding
        elif metrics['funding_efficiency'] > 50:
            sustainability_score += 25
        elif metrics['funding_efficiency'] > 30:
            sustainability_score += 20
        elif metrics['funding_efficiency'] > 0:
            sustainability_score += 10
        
        # Cost recovery score
        if metrics['cost_recovery_ratio'] > 80:
            sustainability_score += 25  # Nearly self-sustaining
        elif metrics['cost_recovery_ratio'] > 60:
            sustainability_score += 20
        elif metrics['cost_recovery_ratio'] > 40:
            sustainability_score += 15
        elif metrics['cost_recovery_ratio'] > 20:
            sustainability_score += 10
        
        # Participant value score (inverse relationship with cost per participant)
        if metrics['cost_per_participant'] > 0:
            benchmark = cost_benchmarks.get(self.event_type, 8000)
            if metrics['cost_per_participant'] <= benchmark * 0.8:
                sustainability_score += 20  # Very efficient
            elif metrics['cost_per_participant'] <= benchmark:
                sustainability_score += 15  # Efficient
            elif metrics['cost_per_participant'] <= benchmark * 1.2:
                sustainability_score += 10  # Acceptable
        
        # Scale efficiency bonus
        if self.total_participants and self.total_participants > 500:
            sustainability_score += 15  # Economies of scale achieved
        elif self.total_participants and self.total_participants > 200:
            sustainability_score += 10
        
        metrics['financial_sustainability_score'] = sustainability_score
        
        return metrics
```

### 10. emp_consultancy_projects Model
**Purpose**: Managing consultancy projects and industrial collaborations with comprehensive tracking of outcomes and financial aspects.

**Enhanced Business Logic Implementation**:

```python
class emp_consultancy_projects(models.Model):
    PROJECT_TYPES = [
        ('TECHNICAL_CONSULTANCY', 'Technical Consultancy'),
        ('STRATEGIC_ADVISORY', 'Strategic Advisory'),
        ('PRODUCT_DEVELOPMENT', 'Product Development'),
        ('PROCESS_OPTIMIZATION', 'Process Optimization'),
        ('RESEARCH_COLLABORATION', 'Research Collaboration'),
        ('TRAINING_CONSULTANCY', 'Training & Development'),
        ('POLICY_ADVISORY', 'Policy Advisory'),
        ('TECHNOLOGY_TRANSFER', 'Technology Transfer')
    ]
    
    CLIENT_SECTORS = [
        ('MANUFACTURING', 'Manufacturing'),
        ('IT_SOFTWARE', 'IT & Software'),
        ('HEALTHCARE', 'Healthcare'),
        ('ENERGY', 'Energy & Utilities'),
        ('FINANCE', 'Financial Services'),
        ('GOVERNMENT', 'Government'),
        ('STARTUP', 'Startup'),
        ('MNC', 'Multinational Corporation'),
        ('PSU', 'Public Sector Unit'),
        ('NGO', 'Non-Governmental Organization')
    ]
    
    # Core project information
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='consultancy_projects')
    pf_no = models.CharField(max_length=20)
    project_type = models.CharField(max_length=25, choices=PROJECT_TYPES)
    
    # Project details
    title = models.TextField(max_length=500, help_text="Project title")
    description = models.TextField(max_length=2000, help_text="Project description")
    technical_domain = models.CharField(max_length=200, help_text="Technical area")
    objectives = models.TextField(max_length=1500, help_text="Project objectives")
    
    # Client information
    client_name = models.CharField(max_length=300, help_text="Client organization")
    client_sector = models.CharField(max_length=20, choices=CLIENT_SECTORS)
    client_location = models.CharField(max_length=200)
    client_contact_person = models.CharField(max_length=200, blank=True)
    
    # Team and collaboration
    consultant_team = models.TextField(max_length=1000, help_text="Consulting team members")
    external_collaborators = models.TextField(max_length=1000, blank=True)
    student_involvement = models.TextField(max_length=500, blank=True)
    
    # Financial aspects
    total_value = models.DecimalField(max_digits=12, decimal_places=2, help_text="Total project value")
    institute_share = models.DecimalField(max_digits=12, decimal_places=2, help_text="Institute's share")
    faculty_share = models.DecimalField(max_digits=12, decimal_places=2, help_text="Faculty share")
    payments_received = models.DecimalField(max_digits=12, decimal_places=2, default=0)
    
    # Timeline and status
    start_date = models.DateField(help_text="Project start date")
    planned_end_date = models.DateField(help_text="Planned completion date")
    actual_end_date = models.DateField(null=True, blank=True)
    current_status = models.CharField(max_length=15, choices=[
        ('ONGOING', 'Ongoing'),
        ('COMPLETED', 'Completed'),
        ('SUSPENDED', 'Suspended'),
        ('CANCELLED', 'Cancelled')
    ], default='ONGOING')
    
    # Deliverables and outcomes
    deliverables = models.TextField(max_length=2000, help_text="Expected deliverables")
    deliverables_completed = models.IntegerField(default=0, help_text="Number completed")
    total_deliverables = models.IntegerField(default=1, help_text="Total deliverables")
    
    # Impact and success metrics
    client_satisfaction_rating = models.IntegerField(null=True, blank=True, 
                                                   choices=[(i, i) for i in range(1, 6)])
    commercial_impact = models.TextField(max_length=1000, blank=True, 
                                       help_text="Commercial impact achieved")
    publications_resulted = models.IntegerField(default=0)
    patents_filed = models.IntegerField(default=0)
    follow_up_projects = models.IntegerField(default=0, help_text="Additional projects from client")
    
    def calculate_consultancy_value_score(self):
        """
        CORE LOGIC: Calculate comprehensive value score for consultancy engagement
        
        HOW IT WORKS:
        1. Evaluates financial value (project size, payments, cost recovery)
        2. Assesses knowledge transfer and academic impact (publications, patents)
        3. Measures client satisfaction and commercial success
        4. Considers strategic value (sector importance, follow-up potential)
        5. Factors in student involvement and capacity building aspects
        
        BUSINESS PURPOSE:
        - Demonstrates faculty's industry engagement and practical impact
        - Supports applications for industry-academia collaboration funding
        - Helps identify high-value consulting opportunities and partnerships
        - Quantifies contribution to knowledge transfer and economic development
        """
        value_score = 0
        
        # Financial value component (40% weight)
        financial_score = 0
        
        # Project size scoring (relative to typical ranges)
        project_value = float(self.total_value)
        if project_value > 5000000:  # > 50 lakhs
            financial_score += 40
        elif project_value > 2000000:  # > 20 lakhs
            financial_score += 35
        elif project_value > 1000000:  # > 10 lakhs
            financial_score += 30
        elif project_value > 500000:   # > 5 lakhs
            financial_score += 25
        elif project_value > 100000:   # > 1 lakh
            financial_score += 20
        else:
            financial_score += 10
        
        # Payment realization efficiency
        if self.payments_received > 0:
            payment_ratio = float(self.payments_received) / project_value
            if payment_ratio >= 0.9:
                financial_score += 15  # Excellent payment realization
            elif payment_ratio >= 0.7:
                financial_score += 12
            elif payment_ratio >= 0.5:
                financial_score += 8
            elif payment_ratio > 0:
                financial_score += 5
        
        value_score += financial_score * 0.4
        
        # Knowledge transfer and academic impact (25% weight)
        knowledge_score = 0
        
        # Publications from consultancy
        knowledge_score += min(self.publications_resulted * 15, 45)  # Max 45 points
        
        # Patents and IP creation
        knowledge_score += min(self.patents_filed * 20, 40)  # Max 40 points
        
        # Student involvement (capacity building)
        if self.student_involvement:
            knowledge_score += 15  # Bonus for student involvement
        
        value_score += knowledge_score * 0.25
        
        # Client satisfaction and commercial success (20% weight)
        success_score = 0
        
        # Client satisfaction rating
        if self.client_satisfaction_rating:
            if self.client_satisfaction_rating >= 4:
                success_score += 25
            elif self.client_satisfaction_rating >= 3:
                success_score += 20
            else:
                success_score += 10
        
        # Commercial impact evidence
        if self.commercial_impact:
            success_score += 15  # Documented commercial impact
        
        # Follow-up projects (repeat business indicator)
        success_score += min(self.follow_up_projects * 10, 30)  # Max 30 points
        
        # Completion success
        if self.current_status == 'COMPLETED':
            completion_ratio = self.deliverables_completed / max(self.total_deliverables, 1)
            if completion_ratio >= 1.0:
                success_score += 15  # All deliverables completed
            elif completion_ratio >= 0.8:
                success_score += 12
            elif completion_ratio >= 0.6:
                success_score += 8
        
        value_score += success_score * 0.2
        
        # Strategic value and sector importance (15% weight)
        strategic_score = 0
        
        # Client sector strategic importance
        high_impact_sectors = ['HEALTHCARE', 'ENERGY', 'MANUFACTURING', 'GOVERNMENT']
        if self.client_sector in high_impact_sectors:
            strategic_score += 20
        else:
            strategic_score += 10
        
        # Project type strategic value
        high_value_types = ['TECHNOLOGY_TRANSFER', 'PRODUCT_DEVELOPMENT', 'RESEARCH_COLLABORATION']
        if self.project_type in high_value_types:
            strategic_score += 15
        else:
            strategic_score += 10
        
        # Duration and complexity
        if self.start_date and self.planned_end_date:
            duration_months = (self.planned_end_date - self.start_date).days / 30.44
            if duration_months > 12:
                strategic_score += 15  # Long-term strategic engagement
            elif duration_months > 6:
                strategic_score += 10
            else:
                strategic_score += 5
        
        value_score += strategic_score * 0.15
        
        return round(value_score, 2)
    
    def analyze_industry_impact_potential(self):
        """
        CORE LOGIC: Analyze potential for broader industry impact and scalability
        
        HOW IT WORKS:
        1. Evaluates technical innovation and uniqueness of solution
        2. Assesses market size and applicability to other organizations
        3. Analyzes scalability potential and replication possibilities
        4. Considers competitive advantages and differentiation factors
        5. Estimates broader economic and societal impact potential
        
        BUSINESS PURPOSE:
        - Identifies consultancy work with high commercialization potential
        - Supports decisions for developing consulting into products/services
        - Helps prioritize projects with maximum industry transformation potential
        - Guides strategy for building sustainable industry partnerships
        """
        impact_analysis = {
            'innovation_level': 'Medium',
            'market_scalability': 'Limited',
            'replication_potential': 'Medium',
            'competitive_advantage': 'Moderate',
            'industry_transformation_score': 0,
            'commercialization_recommendations': []
        }
        
        transformation_score = 0
        
        # Technical innovation assessment
        innovative_types = ['PRODUCT_DEVELOPMENT', 'TECHNOLOGY_TRANSFER', 'PROCESS_OPTIMIZATION']
        if self.project_type in innovative_types:
            transformation_score += 25
            impact_analysis['innovation_level'] = 'High'
        elif self.project_type in ['RESEARCH_COLLABORATION', 'TECHNICAL_CONSULTANCY']:
            transformation_score += 20
            impact_analysis['innovation_level'] = 'Medium'
        else:
            transformation_score += 10
            impact_analysis['innovation_level'] = 'Standard'
        
        # Market size and sector reach
        broad_impact_sectors = ['MANUFACTURING', 'IT_SOFTWARE', 'ENERGY', 'HEALTHCARE']
        if self.client_sector in broad_impact_sectors:
            transformation_score += 20
            impact_analysis['market_scalability'] = 'High'
        elif self.client_sector in ['GOVERNMENT', 'PSU']:
            transformation_score += 15
            impact_analysis['market_scalability'] = 'Medium'
        else:
            transformation_score += 10
            impact_analysis['market_scalability'] = 'Limited'
        
        # IP and knowledge assets created
        ip_score = 0
        if self.patents_filed > 0:
            ip_score += 20
            impact_analysis['competitive_advantage'] = 'Strong'
        
        if self.publications_resulted > 0:
            ip_score += 10
        
        transformation_score += ip_score
        
        # Client success and satisfaction indicators
        if self.client_satisfaction_rating and self.client_satisfaction_rating >= 4:
            transformation_score += 15
        
        if self.follow_up_projects > 0:
            transformation_score += 10  # Indicates continued value
            impact_analysis['replication_potential'] = 'High'
        
        # Commercial impact evidence
        if self.commercial_impact:
            transformation_score += 15
            impact_analysis['competitive_advantage'] = 'Strong'
        
        # Project value and scalability
        project_value = float(self.total_value)
        if project_value > 2000000:  # Large projects often have broader impact
            transformation_score += 15
        elif project_value > 1000000:
            transformation_score += 10
        else:
            transformation_score += 5
        
        impact_analysis['industry_transformation_score'] = transformation_score
        
        # Generate commercialization recommendations
        if transformation_score >= 80:
            impact_analysis['commercialization_recommendations'].extend([
                "High commercialization potential - consider spin-off or licensing",
                "Develop intellectual property portfolio for broader market",
                "Explore partnerships with multiple industry players",
                "Consider developing platform solution for sector-wide adoption"
            ])
        elif transformation_score >= 60:
            impact_analysis['commercialization_recommendations'].extend([
                "Good commercialization potential - expand client base",
                "Document and standardize processes for replication",
                "Explore similar applications in related sectors",
                "Develop case studies for marketing to similar organizations"
            ])
        elif transformation_score >= 40:
            impact_analysis['commercialization_recommendations'].extend([
                "Moderate potential - focus on process improvement",
                "Build stronger IP position through patents/publications",
                "Identify unique differentiators for competitive advantage",
                "Explore partnerships to enhance market reach"
            ])
        else:
            impact_analysis['commercialization_recommendations'].extend([
                "Limited commercialization potential - focus on learning",
                "Build technical capabilities and domain expertise",
                "Develop relationships for future opportunities",
                "Consider this as stepping stone to larger projects"
            ])
        
        return impact_analysis
```