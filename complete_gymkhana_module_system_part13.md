# Complete Gymkhana Module System Documentation - Fusion IIIT (Part 13)

## Database Models Analysis with Business Logic (Final Models)

### Change_office Model

```python
class Change_office(models.Model):
    """
    CORE PURPOSE: Office bearer transition management with comprehensive handover processes
    
    BUSINESS LOGIC:
    - Manages seamless transitions between office bearers
    - Tracks handover processes and knowledge transfer
    - Maintains institutional continuity during leadership changes
    - Handles election results and appointment processes
    - Provides audit trails for all office changes
    
    INTEGRATION POINTS:
    - Links with Core_team for leadership structure updates
    - Connects with Club_info for organizational changes
    - Integrates with voting systems for democratic transitions
    - Coordinates with notification systems for announcements
    """
    
    change_id = models.CharField(max_length=20, unique=True)
    club = models.ForeignKey(Club_info, on_delete=models.CASCADE, related_name='office_changes')
    session = models.CharField(max_length=20)
    
    # Transition details
    change_type = models.CharField(max_length=30, choices=[
        ('election', 'Election'),
        ('appointment', 'Appointment'),
        ('resignation', 'Resignation'),
        ('transfer', 'Transfer'),
        ('promotion', 'Promotion'),
        ('demotion', 'Demotion'),
        ('emergency', 'Emergency Change'),
        ('rotation', 'Rotation'),
        ('expansion', 'Team Expansion'),
        ('consolidation', 'Team Consolidation')
    ])
    
    change_reason = models.CharField(max_length=30, choices=[
        ('term_completion', 'Term Completion'),
        ('academic_completion', 'Academic Completion'),
        ('voluntary_resignation', 'Voluntary Resignation'),
        ('academic_issues', 'Academic Issues'),
        ('disciplinary_action', 'Disciplinary Action'),
        ('health_issues', 'Health Issues'),
        ('personal_reasons', 'Personal Reasons'),
        ('performance_issues', 'Performance Issues'),
        ('conflict_of_interest', 'Conflict of Interest'),
        ('institutional_requirement', 'Institutional Requirement'),
        ('emergency_situation', 'Emergency Situation'),
        ('restructuring', 'Organizational Restructuring')
    ])
    
    # Personnel involved
    outgoing_member = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True, related_name='outgoing_positions')
    incoming_member = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True, related_name='incoming_positions')
    
    # Position details
    position_title = models.CharField(max_length=50)
    position_level = models.CharField(max_length=20, choices=[
        ('president', 'President'),
        ('vice_president', 'Vice President'),
        ('secretary', 'Secretary'),
        ('treasurer', 'Treasurer'),
        ('coordinator', 'Coordinator'),
        ('member', 'Member'),
        ('advisor', 'Advisor'),
        ('mentor', 'Mentor')
    ])
    
    previous_position_title = models.CharField(max_length=50, blank=True)
    new_position_title = models.CharField(max_length=50, blank=True)
    
    # Timing and duration
    effective_date = models.DateTimeField()
    announcement_date = models.DateTimeField(null=True, blank=True)
    handover_start_date = models.DateTimeField(null=True, blank=True)
    handover_completion_date = models.DateTimeField(null=True, blank=True)
    
    # Transition period management
    transition_duration_days = models.IntegerField(default=7)
    overlap_period_days = models.IntegerField(default=3)
    
    # Handover process
    handover_checklist = models.JSONField(default=list, blank=True)
    handover_completion_status = models.CharField(max_length=20, choices=[
        ('not_started', 'Not Started'),
        ('in_progress', 'In Progress'),
        ('partially_complete', 'Partially Complete'),
        ('completed', 'Completed'),
        ('delayed', 'Delayed'),
        ('incomplete', 'Incomplete')
    ], default='not_started')
    
    # Knowledge transfer
    documents_transferred = models.JSONField(default=list, blank=True)
    access_rights_transferred = models.JSONField(default=list, blank=True)
    ongoing_projects_briefed = models.JSONField(default=list, blank=True)
    stakeholder_introductions_completed = models.BooleanField(default=False)
    
    # Training and orientation
    orientation_completed = models.BooleanField(default=False)
    training_modules_assigned = models.JSONField(default=list, blank=True)
    mentorship_assigned = models.BooleanField(default=False)
    mentor_assigned = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True, related_name='mentored_transitions')
    
    # Performance and evaluation
    outgoing_performance_rating = models.FloatField(default=0.0)
    handover_quality_rating = models.FloatField(default=0.0)
    transition_smoothness_rating = models.FloatField(default=0.0)
    incoming_readiness_score = models.FloatField(default=0.0)
    
    # Impact assessment
    club_continuity_impact = models.CharField(max_length=20, choices=[
        ('minimal', 'Minimal Impact'),
        ('low', 'Low Impact'),
        ('moderate', 'Moderate Impact'),
        ('high', 'High Impact'),
        ('severe', 'Severe Impact')
    ], default='minimal')
    
    member_morale_impact = models.CharField(max_length=20, choices=[
        ('positive', 'Positive'),
        ('neutral', 'Neutral'),
        ('slightly_negative', 'Slightly Negative'),
        ('negative', 'Negative'),
        ('very_negative', 'Very Negative')
    ], default='neutral')
    
    # Approval and authorization
    approved_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True, related_name='approved_office_changes')
    approval_date = models.DateTimeField(null=True, blank=True)
    approval_comments = models.TextField(blank=True)
    
    # Status and workflow
    change_status = models.CharField(max_length=20, choices=[
        ('proposed', 'Proposed'),
        ('under_review', 'Under Review'),
        ('approved', 'Approved'),
        ('in_transition', 'In Transition'),
        ('completed', 'Completed'),
        ('cancelled', 'Cancelled'),
        ('on_hold', 'On Hold')
    ], default='proposed')
    
    # Communication and notifications
    announcement_made = models.BooleanField(default=False)
    stakeholders_notified = models.BooleanField(default=False)
    public_announcement = models.BooleanField(default=False)
    notification_channels = models.JSONField(default=list, blank=True)
    
    # Documentation and records
    change_documentation = models.TextField(blank=True)
    official_announcement_text = models.TextField(blank=True)
    transition_notes = models.TextField(blank=True)
    lessons_learned = models.TextField(blank=True)
    
    # Emergency provisions
    is_emergency_change = models.BooleanField(default=False)
    emergency_justification = models.TextField(blank=True)
    temporary_arrangement = models.BooleanField(default=False)
    interim_period_days = models.IntegerField(default=0)
    
    # Metadata
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    created_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True, related_name='initiated_office_changes')
    
    class Meta:
        db_table = 'gymkhana_change_office'
        ordering = ['-effective_date']
        verbose_name = 'Office Change'
        verbose_name_plural = 'Office Changes'
        unique_together = [['club', 'position_title', 'effective_date']]
    
    def __str__(self):
        return f"{self.club.club_name} - {self.position_title} Change ({self.effective_date.date()})"

def save(self, *args, **kwargs):
    """Enhanced save method with office transition management logic"""
    # Generate change ID if not provided
    if not self.change_id:
        self.change_id = self._generate_change_id()
    
    # Validate transition parameters
    self._validate_transition_parameters()
    
    # Calculate transition metrics
    self._calculate_transition_metrics()
    
    # Update transition status
    self._update_transition_status()
    
    # Generate handover checklist
    self._generate_handover_checklist()
    
    super().save(*args, **kwargs)
    
    # Post-save actions
    self._handle_status_transitions()

def _generate_change_id(self):
    """Helper method to generate unique change ID"""
    import uuid
    from datetime import datetime
    
    # Format: CHG_CLUB_YEAR_SEQUENCE
    year_suffix = self.effective_date.year if self.effective_date else datetime.now().year
    sequence = Change_office.objects.filter(
        club=self.club,
        effective_date__year=year_suffix
    ).count() + 1
    
    club_prefix = self.club.club_name[:3].upper() if self.club else 'CLB'
    
    return f"CHG_{club_prefix}_{year_suffix}_{sequence:03d}"

def _validate_transition_parameters(self):
    """Helper method to validate transition parameters"""
    from django.core.exceptions import ValidationError
    from django.utils import timezone
    
    # Validate effective date
    if self.effective_date and self.effective_date < timezone.now() - timezone.timedelta(days=30):
        # Allow some flexibility for past dates
        pass
    
    # Validate handover timeline
    if self.handover_start_date and self.handover_completion_date:
        if self.handover_start_date >= self.handover_completion_date:
            raise ValidationError("Handover start date must be before completion date")
    
    # Validate transition duration
    if self.transition_duration_days < 1:
        self.transition_duration_days = 7  # Default minimum

def _calculate_transition_metrics(self):
    """Helper method to calculate transition metrics"""
    # Calculate incoming readiness score
    self.incoming_readiness_score = self._calculate_incoming_readiness()
    
    # Assess continuity impact
    self._assess_continuity_impact()
    
    # Evaluate transition smoothness
    self.transition_smoothness_rating = self._calculate_transition_smoothness()

def _calculate_incoming_readiness(self):
    """Helper method to calculate incoming member readiness"""
    readiness_factors = []
    
    # Previous experience in similar roles
    if self.incoming_member:
        # Check previous positions (simplified)
        previous_changes = Change_office.objects.filter(
            incoming_member=self.incoming_member
        ).count()
        
        if previous_changes > 2:
            readiness_factors.append(90)
        elif previous_changes > 0:
            readiness_factors.append(70)
        else:
            readiness_factors.append(50)
    
    # Training completion
    if self.orientation_completed:
        readiness_factors.append(85)
    
    if len(self.training_modules_assigned) > 0:
        completion_estimate = 70  # Assume 70% for assigned modules
        readiness_factors.append(completion_estimate)
    
    # Mentorship availability
    if self.mentorship_assigned and self.mentor_assigned:
        readiness_factors.append(80)
    
    # Default factors if insufficient data
    if not readiness_factors:
        readiness_factors.append(60)  # Average readiness
    
    return sum(readiness_factors) / len(readiness_factors)

def _assess_continuity_impact(self):
    """Helper method to assess club continuity impact"""
    impact_factors = []
    
    # Position importance
    position_importance = {
        'president': 5,
        'vice_president': 4,
        'secretary': 4,
        'treasurer': 4,
        'coordinator': 3,
        'member': 2,
        'advisor': 3,
        'mentor': 3
    }
    
    importance_score = position_importance.get(self.position_level, 3)
    
    # Change timing
    from django.utils import timezone
    if self.effective_date:
        # Check if change is during critical periods
        month = self.effective_date.month
        if month in [3, 4, 5, 10, 11]:  # Academic transition periods
            impact_factors.append('high')
        else:
            impact_factors.append('moderate')
    
    # Handover quality
    if self.handover_completion_status == 'completed':
        impact_factors.append('minimal')
    elif self.handover_completion_status in ['in_progress', 'partially_complete']:
        impact_factors.append('moderate')
    else:
        impact_factors.append('high')
    
    # Emergency nature
    if self.is_emergency_change:
        impact_factors.append('high')
    
    # Determine overall impact
    if 'high' in impact_factors and importance_score >= 4:
        self.club_continuity_impact = 'severe'
    elif 'high' in impact_factors or importance_score >= 4:
        self.club_continuity_impact = 'high'
    elif 'moderate' in impact_factors:
        self.club_continuity_impact = 'moderate'
    else:
        self.club_continuity_impact = 'minimal'

def _calculate_transition_smoothness(self):
    """Helper method to calculate transition smoothness"""
    smoothness_factors = []
    
    # Handover completion
    handover_scores = {
        'completed': 95,
        'partially_complete': 70,
        'in_progress': 50,
        'delayed': 30,
        'incomplete': 20,
        'not_started': 10
    }
    
    smoothness_factors.append(handover_scores.get(self.handover_completion_status, 50))
    
    # Stakeholder communication
    if self.stakeholders_notified:
        smoothness_factors.append(80)
    else:
        smoothness_factors.append(40)
    
    # Overlap period adequacy
    if self.overlap_period_days >= 5:
        smoothness_factors.append(85)
    elif self.overlap_period_days >= 3:
        smoothness_factors.append(70)
    else:
        smoothness_factors.append(50)
    
    # Emergency vs planned
    if self.is_emergency_change:
        smoothness_factors.append(30)
    else:
        smoothness_factors.append(80)
    
    return sum(smoothness_factors) / len(smoothness_factors)

def _update_transition_status(self):
    """Helper method to update transition status"""
    from django.utils import timezone
    current_time = timezone.now()
    
    if self.change_status not in ['cancelled', 'completed']:
        if current_time < self.effective_date:
            if self.approval_date:
                self.change_status = 'approved'
            else:
                self.change_status = 'under_review'
        elif self.effective_date <= current_time < (self.handover_completion_date or self.effective_date + timezone.timedelta(days=self.transition_duration_days)):
            self.change_status = 'in_transition'
        elif current_time >= (self.handover_completion_date or self.effective_date + timezone.timedelta(days=self.transition_duration_days)):
            if self.handover_completion_status == 'completed':
                self.change_status = 'completed'

def _generate_handover_checklist(self):
    """Helper method to generate comprehensive handover checklist"""
    if not self.handover_checklist:
        checklist_items = []
        
        # Standard handover items
        standard_items = [
            'Transfer access credentials',
            'Handover ongoing project files',
            'Brief on current initiatives',
            'Introduce to key stakeholders',
            'Transfer financial records',
            'Handover club assets inventory',
            'Share contact database',
            'Explain current challenges',
            'Review upcoming deadlines',
            'Transfer social media accounts'
        ]
        
        # Position-specific items
        position_specific = {
            'president': [
                'Strategic planning documents',
                'External partnership agreements',
                'Board meeting minutes',
                'Institutional liaison contacts'
            ],
            'treasurer': [
                'Financial statements',
                'Budget tracking sheets',
                'Vendor contact information',
                'Audit documentation'
            ],
            'secretary': [
                'Meeting minutes templates',
                'Communication protocols',
                'Documentation systems',
                'Record keeping procedures'
            ]
        }
        
        checklist_items.extend(standard_items)
        
        if self.position_level in position_specific:
            checklist_items.extend(position_specific[self.position_level])
        
        # Convert to checklist format
        self.handover_checklist = [
            {
                'item': item,
                'completed': False,
                'completion_date': None,
                'notes': ''
            }
            for item in checklist_items
        ]

def _handle_status_transitions(self):
    """Helper method to handle status transitions"""
    if self.change_status == 'approved' and self._just_approved():
        self._initiate_transition_process()
    elif self.change_status == 'completed' and self._just_completed():
        self._finalize_transition_process()

def _just_approved(self):
    """Helper method to check if change was just approved"""
    if self.pk:
        try:
            old_instance = Change_office.objects.get(pk=self.pk)
            return old_instance.change_status != 'approved'
        except Change_office.DoesNotExist:
            return False
    return False

def _just_completed(self):
    """Helper method to check if change was just completed"""
    if self.pk:
        try:
            old_instance = Change_office.objects.get(pk=self.pk)
            return old_instance.change_status != 'completed'
        except Change_office.DoesNotExist:
            return False
    return False

def _initiate_transition_process(self):
    """Helper method to initiate transition process"""
    # Set handover start date if not set
    from django.utils import timezone
    if not self.handover_start_date:
        self.handover_start_date = timezone.now()
    
    # Notify stakeholders
    self._notify_transition_stakeholders()
    
    # Update club structure
    self._update_club_structure()

def _finalize_transition_process(self):
    """Helper method to finalize transition process"""
    # Generate lessons learned
    self.lessons_learned = self._generate_lessons_learned()
    
    # Update club metrics
    self._update_club_transition_metrics()
    
    # Archive transition records
    self._archive_transition_records()

def _notify_transition_stakeholders(self):
    """Helper method to notify stakeholders"""
    # Implementation depends on notification system
    self.stakeholders_notified = True

def _update_club_structure(self):
    """Helper method to update club structure"""
    # Update Core_team records
    if self.outgoing_member and self.incoming_member:
        # Implementation depends on Core_team model structure
        pass

def _generate_lessons_learned(self):
    """Helper method to generate lessons learned"""
    lessons = []
    
    # Transition smoothness lessons
    if self.transition_smoothness_rating < 70:
        lessons.append("Improve handover planning and extend transition period")
    
    # Readiness lessons
    if self.incoming_readiness_score < 70:
        lessons.append("Enhance pre-transition training and orientation programs")
    
    # Communication lessons
    if not self.stakeholders_notified:
        lessons.append("Establish better stakeholder communication protocols")
    
    # Impact lessons
    if self.club_continuity_impact in ['high', 'severe']:
        lessons.append("Implement succession planning for critical positions")
    
    return '\n'.join(lessons)

def _update_club_transition_metrics(self):
    """Helper method to update club transition metrics"""
    # Update club-level transition statistics
    if hasattr(self.club, 'transition_statistics'):
        # Implementation depends on club model structure
        pass

def _archive_transition_records(self):
    """Helper method to archive transition records"""
    # Implementation for record archival
    pass
```

### Voting_poll Model

```python
class Voting_poll(models.Model):
    """
    CORE PURPOSE: Democratic decision-making system for club elections and policy decisions
    
    BUSINESS LOGIC:
    - Manages elections for club positions and leadership roles
    - Handles policy voting and member decision-making processes
    - Ensures transparent and fair democratic processes
    - Provides comprehensive voting analytics and audit trails
    - Supports multiple voting methods and systems
    
    INTEGRATION POINTS:
    - Links with Club_info for club-specific voting
    - Connects with Change_office for election results implementation
    - Integrates with Core_team for leadership selection
    - Coordinates with notification systems for voting announcements
    """
    
    poll_id = models.CharField(max_length=20, unique=True)
    club = models.ForeignKey(Club_info, on_delete=models.CASCADE, related_name='voting_polls')
    session = models.CharField(max_length=20)
    
    # Poll basic information
    poll_title = models.CharField(max_length=200)
    poll_description = models.TextField()
    poll_purpose = models.TextField()
    
    # Poll categorization
    poll_type = models.CharField(max_length=30, choices=[
        ('election', 'Leadership Election'),
        ('policy_decision', 'Policy Decision'),
        ('budget_approval', 'Budget Approval'),
        ('event_selection', 'Event Selection'),
        ('constitutional_change', 'Constitutional Change'),
        ('disciplinary_action', 'Disciplinary Action'),
        ('partnership_approval', 'Partnership Approval'),
        ('resource_allocation', 'Resource Allocation'),
        ('rule_change', 'Rule Change'),
        ('membership_decision', 'Membership Decision'),
        ('referendum', 'Referendum'),
        ('survey', 'Opinion Survey')
    ])
    
    voting_method = models.CharField(max_length=20, choices=[
        ('simple_majority', 'Simple Majority'),
        ('absolute_majority', 'Absolute Majority'),
        ('qualified_majority', 'Qualified Majority'),
        ('unanimous', 'Unanimous'),
        ('ranked_choice', 'Ranked Choice'),
        ('approval_voting', 'Approval Voting'),
        ('proportional', 'Proportional'),
        ('weighted', 'Weighted Voting')
    ], default='simple_majority')
    
    # Timing and scheduling
    registration_start_date = models.DateTimeField()
    registration_end_date = models.DateTimeField()
    voting_start_date = models.DateTimeField()
    voting_end_date = models.DateTimeField()
    result_announcement_date = models.DateTimeField(null=True, blank=True)
    
    # Eligibility and participation
    eligible_voter_criteria = models.JSONField(default=dict, blank=True)
    minimum_membership_duration = models.IntegerField(default=0)  # in days
    minimum_attendance_percentage = models.FloatField(default=0.0)
    academic_standing_requirement = models.BooleanField(default=False)
    
    # Voting configuration
    max_selections_per_voter = models.IntegerField(default=1)
    allow_abstention = models.BooleanField(default=True)
    secret_ballot = models.BooleanField(default=True)
    require_voter_authentication = models.BooleanField(default=True)
    
    # Quorum and validation
    minimum_quorum_percentage = models.FloatField(default=50.0)
    minimum_participation_required = models.IntegerField(default=0)
    required_majority_percentage = models.FloatField(default=50.0)
    
    # Candidates and options
    candidates = models.JSONField(default=list, blank=True)  # For elections
    voting_options = models.JSONField(default=list, blank=True)  # For policy decisions
    candidate_registration_required = models.BooleanField(default=False)
    
    # Results and statistics
    total_eligible_voters = models.IntegerField(default=0)
    total_registered_voters = models.IntegerField(default=0)
    total_votes_cast = models.IntegerField(default=0)
    valid_votes = models.IntegerField(default=0)
    invalid_votes = models.IntegerField(default=0)
    abstentions = models.IntegerField(default=0)
    
    # Participation metrics
    voter_turnout_percentage = models.FloatField(default=0.0)
    quorum_achieved = models.BooleanField(default=False)
    participation_target_met = models.BooleanField(default=False)
    
    # Results data
    voting_results = models.JSONField(default=dict, blank=True)
    winner_candidate = models.CharField(max_length=100, blank=True)
    winning_option = models.CharField(max_length=200, blank=True)
    margin_of_victory = models.FloatField(default=0.0)
    
    # Transparency and audit
    audit_trail = models.JSONField(default=list, blank=True)
    vote_verification_codes = models.JSONField(default=list, blank=True)
    election_observers = models.JSONField(default=list, blank=True)
    complaints_received = models.JSONField(default=list, blank=True)
    
    # Status and workflow
    poll_status = models.CharField(max_length=20, choices=[
        ('draft', 'Draft'),
        ('registration_open', 'Registration Open'),
        ('registration_closed', 'Registration Closed'),
        ('voting_active', 'Voting Active'),
        ('voting_closed', 'Voting Closed'),
        ('counting', 'Counting Votes'),
        ('results_announced', 'Results Announced'),
        ('contested', 'Contested'),
        ('cancelled', 'Cancelled'),
        ('postponed', 'Postponed')
    ], default='draft')
    
    # Security and integrity
    encryption_enabled = models.BooleanField(default=True)
    blockchain_verification = models.BooleanField(default=False)
    multi_factor_authentication = models.BooleanField(default=False)
    ip_logging_enabled = models.BooleanField(default=True)
    
    # Approval and oversight
    approved_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True, related_name='approved_polls')
    election_commission = models.JSONField(default=list, blank=True)
    chief_returning_officer = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True, related_name='overseen_polls')
    
    # Communication and announcements
    announcement_made = models.BooleanField(default=False)
    reminder_sent = models.BooleanField(default=False)
    result_announcement_text = models.TextField(blank=True)
    post_election_report = models.TextField(blank=True)
    
    # Analysis and insights
    demographic_analysis = models.JSONField(default=dict, blank=True)
    voting_patterns = models.JSONField(default=dict, blank=True)
    engagement_metrics = models.JSONField(default=dict, blank=True)
    satisfaction_scores = models.JSONField(default=dict, blank=True)
    
    # Metadata
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    created_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True, related_name='created_polls')
    
    class Meta:
        db_table = 'gymkhana_voting_poll'
        ordering = ['-voting_start_date']
        verbose_name = 'Voting Poll'
        verbose_name_plural = 'Voting Polls'
        unique_together = [['club', 'poll_title', 'session']]
    
    def __str__(self):
        return f"{self.poll_title} - {self.club.club_name}"

def save(self, *args, **kwargs):
    """Enhanced save method with comprehensive voting management logic"""
    # Generate poll ID if not provided
    if not self.poll_id:
        self.poll_id = self._generate_poll_id()
    
    # Validate voting parameters
    self._validate_voting_parameters()
    
    # Calculate participation metrics
    self._calculate_participation_metrics()
    
    # Update poll status
    self._update_poll_status()
    
    # Process voting results if applicable
    if self.poll_status == 'voting_closed':
        self._process_voting_results()
    
    super().save(*args, **kwargs)
    
    # Post-save actions
    self._handle_status_transitions()

def _generate_poll_id(self):
    """Helper method to generate unique poll ID"""
    import uuid
    from datetime import datetime
    
    # Format: POLL_CLUB_YEAR_SEQUENCE
    year_suffix = self.voting_start_date.year if self.voting_start_date else datetime.now().year
    sequence = Voting_poll.objects.filter(
        club=self.club,
        voting_start_date__year=year_suffix
    ).count() + 1
    
    club_prefix = self.club.club_name[:3].upper() if self.club else 'CLB'
    
    return f"POLL_{club_prefix}_{year_suffix}_{sequence:03d}"

def _validate_voting_parameters(self):
    """Helper method to validate voting parameters"""
    from django.core.exceptions import ValidationError
    from django.utils import timezone
    
    # Validate timeline
    if self.registration_start_date >= self.registration_end_date:
        raise ValidationError("Registration start must be before end date")
    
    if self.voting_start_date <= self.registration_end_date:
        raise ValidationError("Voting must start after registration ends")
    
    if self.voting_start_date >= self.voting_end_date:
        raise ValidationError("Voting start must be before end date")
    
    # Validate quorum and majority requirements
    if self.minimum_quorum_percentage < 0 or self.minimum_quorum_percentage > 100:
        raise ValidationError("Quorum percentage must be between 0 and 100")
    
    if self.required_majority_percentage < 0 or self.required_majority_percentage > 100:
        raise ValidationError("Required majority percentage must be between 0 and 100")

def _calculate_participation_metrics(self):
    """Helper method to calculate participation metrics"""
    # Calculate voter turnout
    if self.total_eligible_voters > 0:
        self.voter_turnout_percentage = (self.total_votes_cast / self.total_eligible_voters) * 100
    else:
        self.voter_turnout_percentage = 0.0
    
    # Check quorum achievement
    self.quorum_achieved = self.voter_turnout_percentage >= self.minimum_quorum_percentage
    
    # Check participation target
    self.participation_target_met = self.total_votes_cast >= self.minimum_participation_required

def _update_poll_status(self):
    """Helper method to update poll status based on current time"""
    from django.utils import timezone
    current_time = timezone.now()
    
    if self.poll_status not in ['cancelled', 'postponed', 'contested']:
        if current_time < self.registration_start_date:
            self.poll_status = 'draft'
        elif self.registration_start_date <= current_time < self.registration_end_date:
            self.poll_status = 'registration_open'
        elif self.registration_end_date <= current_time < self.voting_start_date:
            self.poll_status = 'registration_closed'
        elif self.voting_start_date <= current_time < self.voting_end_date:
            self.poll_status = 'voting_active'
        elif current_time >= self.voting_end_date:
            if self.total_votes_cast > 0:
                if not self.voting_results:
                    self.poll_status = 'counting'
                elif self.result_announcement_date and current_time >= self.result_announcement_date:
                    self.poll_status = 'results_announced'
                else:
                    self.poll_status = 'voting_closed'
            else:
                self.poll_status = 'voting_closed'

def _process_voting_results(self):
    """Helper method to process voting results"""
    if self.total_votes_cast > 0 and not self.voting_results:
        # Process results based on voting method
        if self.poll_type == 'election':
            self._process_election_results()
        else:
            self._process_decision_results()
        
        # Calculate margin of victory
        self._calculate_margin_of_victory()
        
        # Generate analysis
        self._generate_voting_analysis()

def _process_election_results(self):
    """Helper method to process election results"""
    # Simplified result processing - actual implementation would depend on vote storage
    if self.candidates:
        # Mock result processing
        results = {}
        total_valid_votes = max(1, self.valid_votes)
        
        # Simulate candidate results
        for i, candidate in enumerate(self.candidates):
            # Mock vote distribution
            vote_percentage = (100 - sum(results.values())) / (len(self.candidates) - i) if i < len(self.candidates) - 1 else (100 - sum(results.values()))
            results[candidate.get('name', f'Candidate {i+1}')] = round(vote_percentage, 2)
        
        self.voting_results = results
        
        # Determine winner
        if results:
            winner = max(results.items(), key=lambda x: x[1])
            self.winner_candidate = winner[0]

def _process_decision_results(self):
    """Helper method to process decision results"""
    if self.voting_options:
        # Mock result processing for policy decisions
        results = {}
        total_valid_votes = max(1, self.valid_votes)
        
        # Simulate option results
        for i, option in enumerate(self.voting_options):
            vote_percentage = (100 - sum(results.values())) / (len(self.voting_options) - i) if i < len(self.voting_options) - 1 else (100 - sum(results.values()))
            results[option.get('text', f'Option {i+1}')] = round(vote_percentage, 2)
        
        self.voting_results = results
        
        # Determine winning option
        if results:
            winner = max(results.items(), key=lambda x: x[1])
            self.winning_option = winner[0]

def _calculate_margin_of_victory(self):
    """Helper method to calculate margin of victory"""
    if self.voting_results and len(self.voting_results) >= 2:
        sorted_results = sorted(self.voting_results.values(), reverse=True)
        if len(sorted_results) >= 2:
            self.margin_of_victory = sorted_results[0] - sorted_results[1]
        else:
            self.margin_of_victory = sorted_results[0]

def _generate_voting_analysis(self):
    """Helper method to generate comprehensive voting analysis"""
    # Demographic analysis
    self.demographic_analysis = self._analyze_voter_demographics()
    
    # Voting patterns
    self.voting_patterns = self._analyze_voting_patterns()
    
    # Engagement metrics
    self.engagement_metrics = self._calculate_engagement_metrics()

def _analyze_voter_demographics(self):
    """Helper method to analyze voter demographics"""
    # Simplified demographic analysis
    return {
        'total_eligible': self.total_eligible_voters,
        'total_participated': self.total_votes_cast,
        'turnout_rate': self.voter_turnout_percentage,
        'engagement_level': 'high' if self.voter_turnout_percentage > 70 else 'moderate' if self.voter_turnout_percentage > 50 else 'low'
    }

def _analyze_voting_patterns(self):
    """Helper method to analyze voting patterns"""
    patterns = {}
    
    # Competitiveness analysis
    if self.margin_of_victory:
        if self.margin_of_victory < 5:
            patterns['competitiveness'] = 'very_close'
        elif self.margin_of_victory < 15:
            patterns['competitiveness'] = 'competitive'
        else:
            patterns['competitiveness'] = 'decisive'
    
    # Consensus analysis
    if self.voting_results:
        highest_percentage = max(self.voting_results.values()) if self.voting_results.values() else 0
        if highest_percentage > 80:
            patterns['consensus_level'] = 'strong_consensus'
        elif highest_percentage > 60:
            patterns['consensus_level'] = 'moderate_consensus'
        else:
            patterns['consensus_level'] = 'divided_opinion'
    
    return patterns

def _calculate_engagement_metrics(self):
    """Helper method to calculate engagement metrics"""
    metrics = {}
    
    # Registration engagement
    if self.total_eligible_voters > 0:
        registration_rate = (self.total_registered_voters / self.total_eligible_voters) * 100
        metrics['registration_engagement'] = registration_rate
    
    # Voting engagement
    if self.total_registered_voters > 0:
        voting_rate = (self.total_votes_cast / self.total_registered_voters) * 100
        metrics['voting_engagement'] = voting_rate
    
    # Overall engagement score
    engagement_factors = [
        self.voter_turnout_percentage,
        metrics.get('registration_engagement', 0),
        metrics.get('voting_engagement', 0)
    ]
    
    metrics['overall_engagement_score'] = sum(engagement_factors) / len(engagement_factors)
    
    return metrics

def _handle_status_transitions(self):
    """Helper method to handle status transitions"""
    if self.poll_status == 'results_announced' and self._just_announced():
        self._finalize_poll_process()

def _just_announced(self):
    """Helper method to check if results were just announced"""
    if self.pk:
        try:
            old_instance = Voting_poll.objects.get(pk=self.pk)
            return old_instance.poll_status != 'results_announced'
        except Voting_poll.DoesNotExist:
            return False
    return False

def _finalize_poll_process(self):
    """Helper method to finalize poll process"""
    # Generate post-election report
    self.post_election_report = self._generate_post_election_report()
    
    # Implement results if election
    if self.poll_type == 'election' and self.winner_candidate:
        self._implement_election_results()

def _generate_post_election_report(self):
    """Helper method to generate post-election report"""
    report_sections = []
    
    # Executive summary
    report_sections.append(f"Poll Summary: {self.poll_title}")
    report_sections.append(f"Participation: {self.total_votes_cast} out of {self.total_eligible_voters} eligible voters ({self.voter_turnout_percentage:.1f}%)")
    
    # Results summary
    if self.winner_candidate:
        report_sections.append(f"Winner: {self.winner_candidate} (Margin: {self.margin_of_victory:.1f}%)")
    elif self.winning_option:
        report_sections.append(f"Winning Option: {self.winning_option}")
    
    # Engagement analysis
    if self.engagement_metrics:
        engagement_score = self.engagement_metrics.get('overall_engagement_score', 0)
        report_sections.append(f"Overall Engagement Score: {engagement_score:.1f}/100")
    
    # Recommendations
    if self.voter_turnout_percentage < 60:
        report_sections.append("Recommendation: Enhance voter outreach and engagement strategies for future polls")
    
    return '\n'.join(report_sections)

def _implement_election_results(self):
    """Helper method to implement election results"""
    # Create Change_office record for winner
    if self.winner_candidate and self.poll_type == 'election':
        # Implementation would create Change_office record
        pass


### Inventory Model

```python
class Inventory(models.Model):
    """
    CORE PURPOSE: Comprehensive asset and resource management for gymkhana clubs
    
    BUSINESS LOGIC:
    - Manages physical assets, equipment, and resources for all clubs
    - Tracks asset lifecycle from procurement to disposal
    - Handles asset allocation, maintenance, and depreciation
    - Provides comprehensive asset analytics and utilization metrics
    - Supports asset sharing and inter-club resource optimization
    
    INTEGRATION POINTS:
    - Links with Club_info for club-specific asset allocation
    - Connects with Club_budget for asset procurement and maintenance costs
    - Integrates with Event_info for event-specific equipment allocation
    - Coordinates with maintenance and procurement systems
    """
    
    asset_id = models.CharField(max_length=20, unique=True)
    club = models.ForeignKey(Club_info, on_delete=models.CASCADE, related_name='inventory_assets')
    session = models.CharField(max_length=20)
    
    # Basic asset information
    asset_name = models.CharField(max_length=100)
    asset_description = models.TextField()
    asset_category = models.CharField(max_length=30, choices=[
        ('electronics', 'Electronics'),
        ('furniture', 'Furniture'),
        ('sports_equipment', 'Sports Equipment'),
        ('musical_instruments', 'Musical Instruments'),
        ('audio_visual', 'Audio Visual Equipment'),
        ('office_supplies', 'Office Supplies'),
        ('decorative', 'Decorative Items'),
        ('safety_equipment', 'Safety Equipment'),
        ('tools', 'Tools'),
        ('costumes', 'Costumes'),
        ('books', 'Books'),
        ('stationery', 'Stationery'),
        ('consumables', 'Consumables'),
        ('infrastructure', 'Infrastructure'),
        ('other', 'Other')
    ])
    
    asset_subcategory = models.CharField(max_length=50, blank=True)
    brand = models.CharField(max_length=50, blank=True)
    model_number = models.CharField(max_length=50, blank=True)
    serial_number = models.CharField(max_length=50, blank=True)
    
    # Quantity and units
    total_quantity = models.IntegerField(default=1)
    available_quantity = models.IntegerField(default=1)
    allocated_quantity = models.IntegerField(default=0)
    damaged_quantity = models.IntegerField(default=0)
    lost_quantity = models.IntegerField(default=0)
    
    unit_of_measurement = models.CharField(max_length=20, choices=[
        ('pieces', 'Pieces'),
        ('sets', 'Sets'),
        ('pairs', 'Pairs'),
        ('kg', 'Kilograms'),
        ('liters', 'Liters'),
        ('meters', 'Meters'),
        ('boxes', 'Boxes'),
        ('packets', 'Packets')
    ], default='pieces')
    
    # Financial information
    purchase_price = models.DecimalField(max_digits=10, decimal_places=2, default=0)
    current_value = models.DecimalField(max_digits=10, decimal_places=2, default=0)
    depreciation_rate = models.FloatField(default=10.0)  # Annual depreciation percentage
    total_maintenance_cost = models.DecimalField(max_digits=10, decimal_places=2, default=0)
    
    # Procurement details
    supplier_name = models.CharField(max_length=100, blank=True)
    supplier_contact = models.CharField(max_length=20, blank=True)
    purchase_date = models.DateField(null=True, blank=True)
    purchase_order_number = models.CharField(max_length=50, blank=True)
    warranty_period_months = models.IntegerField(default=0)
    warranty_expiry_date = models.DateField(null=True, blank=True)
    
    # Asset status and condition
    asset_condition = models.CharField(max_length=20, choices=[
        ('excellent', 'Excellent'),
        ('good', 'Good'),
        ('fair', 'Fair'),
        ('poor', 'Poor'),
        ('damaged', 'Damaged'),
        ('non_functional', 'Non-Functional')
    ], default='good')
    
    asset_status = models.CharField(max_length=20, choices=[
        ('active', 'Active'),
        ('allocated', 'Allocated'),
        ('maintenance', 'Under Maintenance'),
        ('repair', 'Under Repair'),
        ('storage', 'In Storage'),
        ('disposed', 'Disposed'),
        ('lost', 'Lost'),
        ('stolen', 'Stolen')
    ], default='active')
    
    # Location and storage
    current_location = models.CharField(max_length=100)
    storage_location = models.CharField(max_length=100, blank=True)
    assigned_to = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True, related_name='assigned_assets')
    location_last_updated = models.DateTimeField(auto_now=True)
    
    # Usage and utilization
    total_usage_hours = models.FloatField(default=0.0)
    usage_frequency = models.CharField(max_length=20, choices=[
        ('daily', 'Daily'),
        ('weekly', 'Weekly'),
        ('monthly', 'Monthly'),
        ('seasonal', 'Seasonal'),
        ('occasional', 'Occasional'),
        ('rare', 'Rare')
    ], default='occasional')
    
    last_used_date = models.DateTimeField(null=True, blank=True)
    average_usage_per_month = models.FloatField(default=0.0)
    utilization_rate = models.FloatField(default=0.0)
    
    # Maintenance and service
    last_maintenance_date = models.DateField(null=True, blank=True)
    next_maintenance_due = models.DateField(null=True, blank=True)
    maintenance_frequency_months = models.IntegerField(default=12)
    maintenance_history = models.JSONField(default=list, blank=True)
    
    # Asset lifecycle
    expected_life_years = models.IntegerField(default=5)
    age_in_months = models.IntegerField(default=0)
    remaining_life_percentage = models.FloatField(default=100.0)
    replacement_due_date = models.DateField(null=True, blank=True)
    
    # Security and tracking
    asset_tags = models.JSONField(default=list, blank=True)
    qr_code = models.CharField(max_length=100, blank=True)
    rfid_tag = models.CharField(max_length=50, blank=True)
    insurance_covered = models.BooleanField(default=False)
    insurance_value = models.DecimalField(max_digits=10, decimal_places=2, default=0)
    
    # Allocation and booking
    is_bookable = models.BooleanField(default=True)
    booking_advance_days = models.IntegerField(default=7)
    max_allocation_duration_days = models.IntegerField(default=30)
    current_allocations = models.JSONField(default=list, blank=True)
    allocation_history = models.JSONField(default=list, blank=True)
    
    # Performance metrics
    reliability_score = models.FloatField(default=0.0)
    availability_percentage = models.FloatField(default=100.0)
    user_satisfaction_score = models.FloatField(default=0.0)
    cost_effectiveness_score = models.FloatField(default=0.0)
    
    # Compliance and documentation
    compliance_certificates = models.JSONField(default=list, blank=True)
    safety_certifications = models.JSONField(default=list, blank=True)
    documentation_links = models.JSONField(default=list, blank=True)
    user_manual_available = models.BooleanField(default=False)
    
    # Sharing and collaboration
    shareable_with_other_clubs = models.BooleanField(default=False)
    sharing_conditions = models.TextField(blank=True)
    shared_usage_cost = models.DecimalField(max_digits=8, decimal_places=2, default=0)
    inter_club_allocations = models.JSONField(default=list, blank=True)
    
    # Environmental and sustainability
    eco_friendly = models.BooleanField(default=False)
    energy_efficiency_rating = models.CharField(max_length=5, blank=True)
    carbon_footprint_kg = models.FloatField(default=0.0)
    recyclable_components = models.JSONField(default=list, blank=True)
    
    # Metadata
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    created_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True, related_name='created_assets')
    last_updated_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True, related_name='updated_assets')
    
    class Meta:
        db_table = 'gymkhana_inventory'
        ordering = ['-created_at']
        verbose_name = 'Inventory Asset'
        verbose_name_plural = 'Inventory Assets'
        unique_together = [['asset_name', 'club', 'serial_number']]
    
    def __str__(self):
        return f"{self.asset_name} ({self.club.club_name})"

def save(self, *args, **kwargs):
    """Enhanced save method with comprehensive asset management logic"""
    # Generate asset ID if not provided
    if not self.asset_id:
        self.asset_id = self._generate_asset_id()
    
    # Validate asset parameters
    self._validate_asset_parameters()
    
    # Calculate financial metrics
    self._calculate_financial_metrics()
    
    # Update asset lifecycle metrics
    self._update_lifecycle_metrics()
    
    # Calculate performance metrics
    self._calculate_performance_metrics()
    
    # Update maintenance schedule
    self._update_maintenance_schedule()
    
    super().save(*args, **kwargs)
    
    # Post-save actions
    self._handle_asset_updates()

def _generate_asset_id(self):
    """Helper method to generate unique asset ID"""
    import uuid
    from datetime import datetime
    
    # Format: AST_CLUB_CATEGORY_SEQUENCE
    current_year = datetime.now().year
    sequence = Inventory.objects.filter(
        club=self.club,
        asset_category=self.asset_category,
        created_at__year=current_year
    ).count() + 1
    
    club_prefix = self.club.club_name[:3].upper() if self.club else 'CLB'
    category_prefix = self.asset_category[:3].upper() if self.asset_category else 'AST'
    
    return f"AST_{club_prefix}_{category_prefix}_{current_year}_{sequence:04d}"

def _validate_asset_parameters(self):
    """Helper method to validate asset parameters"""
    from django.core.exceptions import ValidationError
    
    # Validate quantity consistency
    total_accounted = self.available_quantity + self.allocated_quantity + self.damaged_quantity + self.lost_quantity
    if total_accounted != self.total_quantity:
        # Auto-correct available quantity
        self.available_quantity = self.total_quantity - self.allocated_quantity - self.damaged_quantity - self.lost_quantity
    
    # Validate financial data
    if self.purchase_price < 0:
        raise ValidationError("Purchase price cannot be negative")
    
    # Validate dates
    if self.warranty_expiry_date and self.purchase_date:
        if self.warranty_expiry_date < self.purchase_date:
            raise ValidationError("Warranty expiry cannot be before purchase date")

def _calculate_financial_metrics(self):
    """Helper method to calculate financial metrics"""
    # Calculate current value with depreciation
    if self.purchase_date and self.purchase_price > 0:
        from datetime import date
        age_years = (date.today() - self.purchase_date).days / 365.25
        
        # Simple straight-line depreciation
        depreciated_amount = (self.purchase_price * self.depreciation_rate * age_years) / 100
        self.current_value = max(0, self.purchase_price - depreciated_amount)
    
    # Calculate cost effectiveness
    self.cost_effectiveness_score = self._calculate_cost_effectiveness()

def _calculate_cost_effectiveness(self):
    """Helper method to calculate cost effectiveness score"""
    effectiveness_factors = []
    
    # Utilization vs cost ratio
    if self.purchase_price > 0 and self.total_usage_hours > 0:
        cost_per_hour = float(self.purchase_price + self.total_maintenance_cost) / self.total_usage_hours
        
        # Benchmark cost per hour by category (simplified)
        category_benchmarks = {
            'electronics': 50,
            'furniture': 10,
            'sports_equipment': 25,
            'musical_instruments': 40,
            'audio_visual': 60
        }
        
        benchmark = category_benchmarks.get(self.asset_category, 30)
        if cost_per_hour <= benchmark:
            effectiveness_factors.append(90)
        elif cost_per_hour <= benchmark * 1.5:
            effectiveness_factors.append(70)
        else:
            effectiveness_factors.append(50)
    
    # Availability vs reliability
    combined_availability = (self.availability_percentage + self.reliability_score) / 2
    effectiveness_factors.append(min(100, combined_availability))
    
    # Condition impact
    condition_scores = {
        'excellent': 95,
        'good': 85,
        'fair': 65,
        'poor': 40,
        'damaged': 20,
        'non_functional': 10
    }
    effectiveness_factors.append(condition_scores.get(self.asset_condition, 50))
    
    return sum(effectiveness_factors) / len(effectiveness_factors) if effectiveness_factors else 0.0

def _update_lifecycle_metrics(self):
    """Helper method to update asset lifecycle metrics"""
    # Calculate age in months
    if self.purchase_date:
        from datetime import date
        age_days = (date.today() - self.purchase_date).days
        self.age_in_months = int(age_days / 30.44)  # Average days per month
    
    # Calculate remaining life percentage
    if self.expected_life_years > 0:
        age_years = self.age_in_months / 12
        self.remaining_life_percentage = max(0, 100 - (age_years / self.expected_life_years) * 100)
    
    # Calculate replacement due date
    if self.purchase_date and self.expected_life_years > 0:
        from datetime import date, timedelta
        replacement_date = self.purchase_date + timedelta(days=self.expected_life_years * 365)
        self.replacement_due_date = replacement_date

def _calculate_performance_metrics(self):
    """Helper method to calculate performance metrics"""
    # Calculate reliability score
    self.reliability_score = self._calculate_reliability_score()
    
    # Calculate availability percentage
    self.availability_percentage = self._calculate_availability_percentage()
    
    # Calculate utilization rate
    self.utilization_rate = self._calculate_utilization_rate()

def _calculate_reliability_score(self):
    """Helper method to calculate reliability score"""
    reliability_factors = []
    
    # Condition-based reliability
    condition_reliability = {
        'excellent': 95,
        'good': 85,
        'fair': 70,
        'poor': 50,
        'damaged': 30,
        'non_functional': 10
    }
    reliability_factors.append(condition_reliability.get(self.asset_condition, 50))
    
    # Age-based reliability
    if self.remaining_life_percentage > 80:
        reliability_factors.append(90)
    elif self.remaining_life_percentage > 60:
        reliability_factors.append(80)
    elif self.remaining_life_percentage > 40:
        reliability_factors.append(70)
    elif self.remaining_life_percentage > 20:
        reliability_factors.append(60)
    else:
        reliability_factors.append(40)
    
    # Maintenance history impact
    if len(self.maintenance_history) > 0:
        # More maintenance could indicate either good care or frequent issues
        recent_maintenance = len([m for m in self.maintenance_history if 'recent' in str(m)])
        if recent_maintenance > 0:
            reliability_factors.append(80)
        else:
            reliability_factors.append(60)
    else:
        reliability_factors.append(70)  # Neutral for no history
    
    return sum(reliability_factors) / len(reliability_factors) if reliability_factors else 0.0

def _calculate_availability_percentage(self):
    """Helper method to calculate availability percentage"""
    # Calculate based on current status
    if self.asset_status in ['active', 'storage']:
        base_availability = 95
    elif self.asset_status == 'allocated':
        base_availability = 70  # Allocated but could be available soon
    elif self.asset_status in ['maintenance', 'repair']:
        base_availability = 30
    else:
        base_availability = 10
    
    # Adjust for condition
    condition_multipliers = {
        'excellent': 1.0,
        'good': 0.95,
        'fair': 0.85,
        'poor': 0.70,
        'damaged': 0.50,
        'non_functional': 0.10
    }
    
    multiplier = condition_multipliers.get(self.asset_condition, 0.80)
    
    # Adjust for quantity availability
    if self.total_quantity > 0:
        quantity_availability = self.available_quantity / self.total_quantity
        final_availability = base_availability * multiplier * quantity_availability
    else:
        final_availability = 0
    
    return min(100, max(0, final_availability))

def _calculate_utilization_rate(self):
    """Helper method to calculate utilization rate"""
    if self.total_usage_hours > 0 and self.age_in_months > 0:
        # Calculate theoretical maximum usage
        # Assume 8 hours/day max usage, 30 days/month
        max_possible_hours = self.age_in_months * 30 * 8
        
        if max_possible_hours > 0:
            utilization = (self.total_usage_hours / max_possible_hours) * 100
            return min(100, utilization)
    
    return 0.0

def _update_maintenance_schedule(self):
    """Helper method to update maintenance schedule"""
    if self.last_maintenance_date and self.maintenance_frequency_months > 0:
        from datetime import date, timedelta
        next_due = self.last_maintenance_date + timedelta(days=self.maintenance_frequency_months * 30)
        self.next_maintenance_due = next_due
    elif self.purchase_date and not self.last_maintenance_date:
        # First maintenance due after purchase
        from datetime import date, timedelta
        next_due = self.purchase_date + timedelta(days=self.maintenance_frequency_months * 30)
        self.next_maintenance_due = next_due

def _handle_asset_updates(self):
    """Helper method to handle asset updates"""
    # Check for maintenance alerts
    self._check_maintenance_alerts()
    
    # Update club asset statistics
    self._update_club_asset_statistics()

def _check_maintenance_alerts(self):
    """Helper method to check maintenance alerts"""
    from datetime import date, timedelta
    
    if self.next_maintenance_due:
        days_to_maintenance = (self.next_maintenance_due - date.today()).days
        
        # Generate alerts for upcoming maintenance
        if days_to_maintenance <= 7:
            # Implementation would create maintenance alerts
            pass

def _update_club_asset_statistics(self):
    """Helper method to update club asset statistics"""
    # Update club-level asset metrics
    if hasattr(self.club, 'asset_statistics'):
        # Implementation depends on club model structure
        pass

def get_comprehensive_asset_analytics(self):
    """
    CORE LOGIC: Generate comprehensive analytics for asset management and optimization
    
    HOW IT WORKS:
    1. Analyzes asset performance, utilization, and financial efficiency
    2. Provides lifecycle management insights and replacement planning
    3. Generates optimization recommendations for asset utilization
    4. Tracks maintenance patterns and cost effectiveness
    
    BUSINESS PURPOSE:
    - Asset performance optimization and cost management
    - Lifecycle planning and replacement scheduling
    - Resource allocation optimization across clubs
    - Financial efficiency analysis and budget planning
    """
    # Core analytics
    financial_analytics = self._analyze_comprehensive_financial_performance()
    utilization_analytics = self._analyze_comprehensive_utilization_metrics()
    lifecycle_analytics = self._analyze_comprehensive_lifecycle_management()
    
    # Advanced analytics
    optimization_analytics = self._analyze_comprehensive_optimization_opportunities()
    predictive_analytics = self._generate_comprehensive_predictive_insights()
    sustainability_analytics = self._analyze_comprehensive_sustainability_metrics()
    
    return {
        'asset_overview': {
            'asset_id': self.asset_id,
            'asset_name': self.asset_name,
            'asset_category': self.asset_category,
            'club': self.club.club_name,
            'current_value': float(self.current_value),
            'condition': self.asset_condition,
            'status': self.asset_status,
            'analysis_timestamp': datetime.datetime.now().isoformat()
        },
        'financial_analytics': financial_analytics,
        'utilization_analytics': utilization_analytics,
        'lifecycle_analytics': lifecycle_analytics,
        'optimization_analytics': optimization_analytics,
        'predictive_analytics': predictive_analytics,
        'sustainability_analytics': sustainability_analytics,
        'comprehensive_recommendations': self._generate_comprehensive_recommendations(),
        'action_priorities': self._identify_action_priorities()
    }

def _analyze_comprehensive_financial_performance(self):
    """Helper method for comprehensive financial performance analysis"""
    return {
        'cost_analysis': {
            'initial_investment': float(self.purchase_price),
            'current_value': float(self.current_value),
            'total_maintenance_cost': float(self.total_maintenance_cost),
            'total_cost_of_ownership': float(self.purchase_price + self.total_maintenance_cost),
            'depreciation_amount': float(self.purchase_price - self.current_value),
            'depreciation_rate': self.depreciation_rate
        },
        'efficiency_metrics': {
            'cost_effectiveness_score': self.cost_effectiveness_score,
            'cost_per_usage_hour': float(self.purchase_price + self.total_maintenance_cost) / max(1, self.total_usage_hours),
            'maintenance_cost_ratio': float(self.total_maintenance_cost) / max(1, float(self.purchase_price)) * 100,
            'value_retention_percentage': float(self.current_value) / max(1, float(self.purchase_price)) * 100
        },
        'roi_analysis': {
            'utilization_roi': self._calculate_utilization_roi(),
            'maintenance_efficiency': self._calculate_maintenance_efficiency(),
            'replacement_economics': self._analyze_replacement_economics()
        }
    }

def _calculate_utilization_roi(self):
    """Helper method to calculate utilization ROI"""
    if self.total_usage_hours > 0 and self.purchase_price > 0:
        # Estimate value generated through usage
        # Simplified calculation based on category value
        category_value_per_hour = {
            'electronics': 25,
            'audio_visual': 40,
            'musical_instruments': 30,
            'sports_equipment': 15,
            'furniture': 5
        }
        
        value_per_hour = category_value_per_hour.get(self.asset_category, 20)
        total_value_generated = self.total_usage_hours * value_per_hour
        
        roi = ((total_value_generated - float(self.purchase_price)) / float(self.purchase_price)) * 100
        return round(roi, 2)
    
    return 0.0

def _calculate_maintenance_efficiency(self):
    """Helper method to calculate maintenance efficiency"""
    if len(self.maintenance_history) > 0:
        # Simplified efficiency based on maintenance frequency vs condition
        maintenance_count = len(self.maintenance_history)
        condition_scores = {
            'excellent': 100,
            'good': 85,
            'fair': 70,
            'poor': 50,
            'damaged': 30,
            'non_functional': 10
        }
        
        current_condition_score = condition_scores.get(self.asset_condition, 50)
        
        # More maintenance should generally lead to better condition
        expected_condition = min(100, 60 + (maintenance_count * 10))
        efficiency = (current_condition_score / expected_condition) * 100
        
        return min(100, max(0, efficiency))
    
    return 75  # Default efficiency for assets without maintenance history

def _analyze_replacement_economics(self):
    """Helper method to analyze replacement economics"""
    return {
        'remaining_life_percentage': self.remaining_life_percentage,
        'replacement_urgency': 'high' if self.remaining_life_percentage < 20 else 'medium' if self.remaining_life_percentage < 50 else 'low',
        'current_vs_new_cost_ratio': self._estimate_replacement_cost_ratio(),
        'optimal_replacement_timing': self._estimate_optimal_replacement_timing()
    }

def _estimate_replacement_cost_ratio(self):
    """Helper method to estimate replacement cost ratio"""
    # Estimate current market price (assuming 3% annual inflation)
    if self.purchase_date and self.purchase_price > 0:
        from datetime import date
        age_years = (date.today() - self.purchase_date).days / 365.25
        estimated_current_market_price = float(self.purchase_price) * (1.03 ** age_years)
        
        return float(self.current_value) / estimated_current_market_price
    
    return 0.5  # Default assumption

def _estimate_optimal_replacement_timing(self):
    """Helper method to estimate optimal replacement timing"""
    if self.remaining_life_percentage > 60:
        return 'not_yet_optimal'
    elif self.remaining_life_percentage > 30:
        return 'consider_planning'
    elif self.remaining_life_percentage > 10:
        return 'replacement_recommended'
    else:
        return 'immediate_replacement_needed'
```