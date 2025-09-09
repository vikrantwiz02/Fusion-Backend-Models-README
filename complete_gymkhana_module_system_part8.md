# Complete Gymkhana Module System Documentation - Fusion IIIT (Part 8)

## Database Models Analysis with Business Logic (Continued)

### Session_info Model

```python
class Session_info(models.Model):
    """
    CORE PURPOSE: Academic session management and timeline coordination
    
    BUSINESS LOGIC:
    - Manages academic session information and timelines
    - Coordinates club activities with academic calendar
    - Provides session-based analytics and reporting
    - Handles session transitions and continuity planning
    
    INTEGRATION POINTS:
    - Links with Club_budget for financial planning cycles
    - Coordinates with Core_team for leadership transitions
    - Integrates with Event_info for session-based event planning
    - Supports Club_member for membership lifecycle management
    """
    
    session = models.CharField(max_length=20, unique=True)
    start_date = models.DateField()
    end_date = models.DateField()
    is_active = models.BooleanField(default=False)
    academic_year = models.CharField(max_length=10)
    semester = models.CharField(max_length=10, choices=[
        ('odd', 'Odd Semester'),
        ('even', 'Even Semester'),
        ('summer', 'Summer Term')
    ])
    registration_start = models.DateTimeField()
    registration_end = models.DateTimeField()
    activity_start = models.DateTimeField()
    activity_end = models.DateTimeField()
    budget_submission_deadline = models.DateTimeField()
    report_submission_deadline = models.DateTimeField()
    
    # Session management fields
    session_coordinator = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True)
    session_status = models.CharField(max_length=20, choices=[
        ('planning', 'Planning Phase'),
        ('registration', 'Registration Phase'),
        ('active', 'Active Phase'),
        ('evaluation', 'Evaluation Phase'),
        ('completed', 'Completed'),
        ('archived', 'Archived')
    ], default='planning')
    
    # Academic integration
    academic_calendar_sync = models.BooleanField(default=False)
    exam_schedule_consideration = models.BooleanField(default=True)
    holiday_adjustments = models.JSONField(default=dict, blank=True)
    
    # Analytics and reporting
    total_clubs_participated = models.IntegerField(default=0)
    total_events_conducted = models.IntegerField(default=0)
    total_budget_allocated = models.DecimalField(max_digits=12, decimal_places=2, default=0)
    total_budget_utilized = models.DecimalField(max_digits=12, decimal_places=2, default=0)
    
    # Session performance metrics
    member_engagement_score = models.FloatField(default=0.0)
    financial_efficiency_score = models.FloatField(default=0.0)
    activity_success_rate = models.FloatField(default=0.0)
    overall_session_rating = models.FloatField(default=0.0)
    
    # Metadata
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        db_table = 'gymkhana_session_info'
        ordering = ['-start_date']
        verbose_name = 'Session Information'
        verbose_name_plural = 'Session Information'
    
    def __str__(self):
        return f"{self.session} ({self.academic_year})"

def save(self, *args, **kwargs):
    """Enhanced save method with session management logic"""
    # Validate session dates
    if self.start_date >= self.end_date:
        raise ValidationError("Session start date must be before end date")
    
    # Validate registration dates
    if self.registration_start >= self.registration_end:
        raise ValidationError("Registration start must be before registration end")
    
    # Ensure only one active session
    if self.is_active:
        Session_info.objects.filter(is_active=True).exclude(id=self.id).update(is_active=False)
    
    # Auto-calculate academic year if not provided
    if not self.academic_year:
        self.academic_year = self._calculate_academic_year()
    
    # Update session status based on current date
    self._update_session_status()
    
    # Calculate performance metrics
    self._calculate_session_metrics()
    
    super().save(*args, **kwargs)
    
    # Post-save actions
    self._handle_session_transitions()

def _calculate_academic_year(self):
    """Helper method to calculate academic year"""
    start_year = self.start_date.year
    end_year = self.end_date.year
    
    if start_year == end_year:
        return str(start_year)
    else:
        return f"{start_year}-{end_year}"

def _update_session_status(self):
    """Helper method to update session status based on current date"""
    from django.utils import timezone
    current_date = timezone.now()
    
    if current_date < self.registration_start:
        self.session_status = 'planning'
    elif self.registration_start <= current_date < self.registration_end:
        self.session_status = 'registration'
    elif self.activity_start <= current_date < self.activity_end:
        self.session_status = 'active'
    elif self.activity_end <= current_date < self.report_submission_deadline:
        self.session_status = 'evaluation'
    elif current_date >= self.report_submission_deadline:
        if self.session_status != 'archived':
            self.session_status = 'completed'

def _calculate_session_metrics(self):
    """Helper method to calculate session performance metrics"""
    # Update club participation count
    self.total_clubs_participated = Club_member.objects.filter(
        session=self.session
    ).values('club').distinct().count()
    
    # Update event count
    self.total_events_conducted = Event_info.objects.filter(
        session=self.session
    ).count()
    
    # Update budget metrics
    budget_data = Club_budget.objects.filter(session=self.session).aggregate(
        total_allocated=Sum('budget_amount'),
        total_utilized=Sum('expenditure')
    )
    
    self.total_budget_allocated = budget_data['total_allocated'] or 0
    self.total_budget_utilized = budget_data['total_utilized'] or 0
    
    # Calculate performance scores
    self._calculate_performance_scores()

def _calculate_performance_scores(self):
    """Helper method to calculate various performance scores"""
    # Member engagement score
    self.member_engagement_score = self._calculate_member_engagement()
    
    # Financial efficiency score
    self.financial_efficiency_score = self._calculate_financial_efficiency()
    
    # Activity success rate
    self.activity_success_rate = self._calculate_activity_success_rate()
    
    # Overall session rating
    self.overall_session_rating = self._calculate_overall_rating()

def _calculate_member_engagement(self):
    """Helper method to calculate member engagement score"""
    total_members = Club_member.objects.filter(session=self.session).count()
    active_members = Club_member.objects.filter(
        session=self.session,
        membership_status='active'
    ).count()
    
    if total_members == 0:
        return 0.0
    
    # Base engagement from active membership ratio
    base_engagement = (active_members / total_members) * 60
    
    # Event participation bonus
    event_participation = self._calculate_event_participation_bonus()
    
    # Leadership engagement bonus
    leadership_engagement = self._calculate_leadership_engagement_bonus()
    
    total_engagement = min(100.0, base_engagement + event_participation + leadership_engagement)
    return round(total_engagement, 2)

def _calculate_event_participation_bonus(self):
    """Helper method to calculate event participation bonus"""
    total_events = self.total_events_conducted
    if total_events == 0:
        return 0.0
    
    # Estimate participation (simplified calculation)
    avg_participation_rate = min(0.7, total_events / max(1, self.total_clubs_participated) * 0.1)
    return avg_participation_rate * 20  # Max 20 points bonus

def _calculate_leadership_engagement_bonus(self):
    """Helper method to calculate leadership engagement bonus"""
    total_leadership_positions = Core_team.objects.filter(session=self.session).count()
    total_clubs = self.total_clubs_participated
    
    if total_clubs == 0:
        return 0.0
    
    leadership_ratio = total_leadership_positions / total_clubs
    return min(20.0, leadership_ratio * 100 * 0.2)  # Max 20 points bonus

def _calculate_financial_efficiency(self):
    """Helper method to calculate financial efficiency score"""
    if self.total_budget_allocated == 0:
        return 0.0
    
    utilization_rate = (self.total_budget_utilized / self.total_budget_allocated) * 100
    
    # Optimal utilization is between 75-95%
    if 75 <= utilization_rate <= 95:
        efficiency_score = 100
    elif 60 <= utilization_rate < 75:
        efficiency_score = 80 + (utilization_rate - 60) * 1.33
    elif 95 < utilization_rate <= 100:
        efficiency_score = 100 - (utilization_rate - 95) * 2
    elif utilization_rate > 100:
        efficiency_score = max(60, 90 - (utilization_rate - 100) * 2)
    else:
        efficiency_score = max(40, utilization_rate * 1.2)
    
    return round(efficiency_score, 2)

def _calculate_activity_success_rate(self):
    """Helper method to calculate activity success rate"""
    if self.total_events_conducted == 0:
        return 0.0
    
    # Simplified success rate calculation
    # In a real implementation, this would be based on event feedback and completion rates
    
    # Base success rate estimation
    base_success = min(90.0, (self.total_events_conducted / max(1, self.total_clubs_participated)) * 20)
    
    # Financial management factor
    financial_factor = self.financial_efficiency_score * 0.1
    
    # Engagement factor
    engagement_factor = self.member_engagement_score * 0.1
    
    total_success_rate = min(100.0, base_success + financial_factor + engagement_factor)
    return round(total_success_rate, 2)

def _calculate_overall_rating(self):
    """Helper method to calculate overall session rating"""
    # Weighted average of all performance metrics
    weights = {
        'engagement': 0.3,
        'financial': 0.3,
        'activity': 0.25,
        'participation': 0.15
    }
    
    participation_score = min(100.0, (self.total_clubs_participated / 20) * 100)  # Assuming 20 clubs is excellent
    
    overall_rating = (
        self.member_engagement_score * weights['engagement'] +
        self.financial_efficiency_score * weights['financial'] +
        self.activity_success_rate * weights['activity'] +
        participation_score * weights['participation']
    )
    
    return round(overall_rating, 2)

def _handle_session_transitions(self):
    """Helper method to handle session transitions"""
    if self.session_status == 'completed' and self._changed_to_completed():
        self._initiate_session_completion_process()
    elif self.session_status == 'active' and self._changed_to_active():
        self._initiate_session_activation_process()

def _changed_to_completed(self):
    """Helper method to check if session just changed to completed"""
    if self.pk:
        try:
            old_instance = Session_info.objects.get(pk=self.pk)
            return old_instance.session_status != 'completed'
        except Session_info.DoesNotExist:
            return False
    return False

def _changed_to_active(self):
    """Helper method to check if session just changed to active"""
    if self.pk:
        try:
            old_instance = Session_info.objects.get(pk=self.pk)
            return old_instance.session_status != 'active'
        except Session_info.DoesNotExist:
            return False
    return False

def _initiate_session_completion_process(self):
    """Helper method to initiate session completion process"""
    # Generate session completion reports
    self._generate_session_completion_report()
    
    # Archive session data
    self._archive_session_data()
    
    # Prepare for next session
    self._prepare_next_session_template()

def _initiate_session_activation_process(self):
    """Helper method to initiate session activation process"""
    # Activate all club budgets for the session
    self._activate_session_budgets()
    
    # Initialize session tracking
    self._initialize_session_tracking()
    
    # Send activation notifications
    self._send_session_activation_notifications()

def _generate_session_completion_report(self):
    """Helper method to generate session completion report"""
    # This would generate comprehensive session reports
    # Implementation depends on reporting requirements
    pass

def _archive_session_data(self):
    """Helper method to archive session data"""
    # Archive old session data
    # Implementation depends on archival requirements
    pass

def _prepare_next_session_template(self):
    """Helper method to prepare next session template"""
    # Create template for next session
    # Implementation depends on planning requirements
    pass

def _activate_session_budgets(self):
    """Helper method to activate session budgets"""
    Club_budget.objects.filter(session=self.session).update(
        is_active=True
    )

def _initialize_session_tracking(self):
    """Helper method to initialize session tracking"""
    # Initialize tracking metrics
    # Implementation depends on tracking requirements
    pass

def _send_session_activation_notifications(self):
    """Helper method to send session activation notifications"""
    # Send notifications to club coordinators
    # Implementation depends on notification system
    pass

def get_session_timeline(self):
    """
    CORE LOGIC: Get comprehensive session timeline with phase information
    
    HOW IT WORKS:
    1. Calculates current phase and remaining time
    2. Provides timeline milestones and deadlines
    3. Assesses phase completion and progress
    4. Generates timeline-based recommendations
    
    BUSINESS PURPOSE:
    - Session planning and coordination
    - Timeline management and tracking
    - Phase-based activity planning
    - Deadline management and alerts
    """
    from django.utils import timezone
    current_date = timezone.now()
    
    # Calculate phase durations and progress
    timeline_phases = self._calculate_timeline_phases(current_date)
    
    # Assess current phase progress
    current_phase_info = self._assess_current_phase_progress(current_date)
    
    # Generate milestone tracking
    milestone_tracking = self._track_session_milestones(current_date)
    
    # Identify upcoming deadlines
    upcoming_deadlines = self._identify_upcoming_deadlines(current_date)
    
    return {
        'session_overview': {
            'session_name': self.session,
            'academic_year': self.academic_year,
            'semester': self.semester,
            'current_status': self.session_status,
            'is_active': self.is_active
        },
        'timeline_phases': timeline_phases,
        'current_phase': current_phase_info,
        'milestone_tracking': milestone_tracking,
        'upcoming_deadlines': upcoming_deadlines,
        'timeline_health': self._assess_timeline_health(timeline_phases, current_phase_info),
        'recommendations': self._generate_timeline_recommendations(current_phase_info, upcoming_deadlines)
    }

def _calculate_timeline_phases(self, current_date):
    """Helper method to calculate timeline phases"""
    phases = [
        {
            'phase_name': 'Planning Phase',
            'start_date': self.created_at,
            'end_date': self.registration_start,
            'duration_days': (self.registration_start - self.created_at).days,
            'status': 'completed' if current_date >= self.registration_start else 'current' if current_date >= self.created_at else 'upcoming'
        },
        {
            'phase_name': 'Registration Phase',
            'start_date': self.registration_start,
            'end_date': self.registration_end,
            'duration_days': (self.registration_end - self.registration_start).days,
            'status': 'completed' if current_date >= self.registration_end else 'current' if current_date >= self.registration_start else 'upcoming'
        },
        {
            'phase_name': 'Active Phase',
            'start_date': self.activity_start,
            'end_date': self.activity_end,
            'duration_days': (self.activity_end - self.activity_start).days,
            'status': 'completed' if current_date >= self.activity_end else 'current' if current_date >= self.activity_start else 'upcoming'
        },
        {
            'phase_name': 'Evaluation Phase',
            'start_date': self.activity_end,
            'end_date': self.report_submission_deadline,
            'duration_days': (self.report_submission_deadline - self.activity_end).days,
            'status': 'completed' if current_date >= self.report_submission_deadline else 'current' if current_date >= self.activity_end else 'upcoming'
        }
    ]
    
    return phases

def _assess_current_phase_progress(self, current_date):
    """Helper method to assess current phase progress"""
    current_phase = None
    progress_percentage = 0
    
    if current_date < self.registration_start:
        current_phase = 'Planning Phase'
        total_duration = (self.registration_start - self.created_at).total_seconds()
        elapsed_duration = (current_date - self.created_at).total_seconds()
        progress_percentage = (elapsed_duration / total_duration) * 100 if total_duration > 0 else 0
    elif current_date < self.registration_end:
        current_phase = 'Registration Phase'
        total_duration = (self.registration_end - self.registration_start).total_seconds()
        elapsed_duration = (current_date - self.registration_start).total_seconds()
        progress_percentage = (elapsed_duration / total_duration) * 100 if total_duration > 0 else 0
    elif current_date < self.activity_end:
        current_phase = 'Active Phase'
        total_duration = (self.activity_end - self.activity_start).total_seconds()
        elapsed_duration = (current_date - self.activity_start).total_seconds()
        progress_percentage = (elapsed_duration / total_duration) * 100 if total_duration > 0 else 0
    elif current_date < self.report_submission_deadline:
        current_phase = 'Evaluation Phase'
        total_duration = (self.report_submission_deadline - self.activity_end).total_seconds()
        elapsed_duration = (current_date - self.activity_end).total_seconds()
        progress_percentage = (elapsed_duration / total_duration) * 100 if total_duration > 0 else 0
    else:
        current_phase = 'Completed'
        progress_percentage = 100
    
    return {
        'current_phase': current_phase,
        'progress_percentage': round(min(100, max(0, progress_percentage)), 2),
        'phase_status': self._determine_phase_status(progress_percentage),
        'phase_assessment': self._assess_phase_performance(current_phase, progress_percentage)
    }

def _determine_phase_status(self, progress_percentage):
    """Helper method to determine phase status"""
    if progress_percentage < 25:
        return 'early_stage'
    elif progress_percentage < 50:
        return 'developing_stage'
    elif progress_percentage < 75:
        return 'mature_stage'
    elif progress_percentage < 100:
        return 'final_stage'
    else:
        return 'completed'

def _assess_phase_performance(self, current_phase, progress_percentage):
    """Helper method to assess phase performance"""
    # Performance assessment based on phase and progress
    performance_factors = []
    
    if current_phase == 'Planning Phase':
        if progress_percentage > 50:
            performance_factors.append('adequate_planning_time')
        else:
            performance_factors.append('early_planning_stage')
    elif current_phase == 'Registration Phase':
        registration_count = Club_member.objects.filter(session=self.session).count()
        if registration_count > 50:
            performance_factors.append('strong_registration_response')
        elif registration_count > 20:
            performance_factors.append('moderate_registration_response')
        else:
            performance_factors.append('low_registration_response')
    elif current_phase == 'Active Phase':
        if self.total_events_conducted > 10:
            performance_factors.append('active_event_execution')
        elif self.total_events_conducted > 5:
            performance_factors.append('moderate_event_execution')
        else:
            performance_factors.append('limited_event_execution')
    
    return {
        'performance_factors': performance_factors,
        'phase_effectiveness': self._calculate_phase_effectiveness(current_phase),
        'improvement_areas': self._identify_phase_improvement_areas(current_phase)
    }

def _calculate_phase_effectiveness(self, current_phase):
    """Helper method to calculate phase effectiveness"""
    if current_phase == 'Planning Phase':
        return min(100, self.total_clubs_participated * 5)  # 5 points per participating club
    elif current_phase == 'Registration Phase':
        total_registrations = Club_member.objects.filter(session=self.session).count()
        return min(100, total_registrations * 2)  # 2 points per registration
    elif current_phase == 'Active Phase':
        return min(100, self.total_events_conducted * 10)  # 10 points per event
    elif current_phase == 'Evaluation Phase':
        return self.activity_success_rate
    else:
        return self.overall_session_rating

def _identify_phase_improvement_areas(self, current_phase):
    """Helper method to identify phase improvement areas"""
    improvement_areas = []
    
    if current_phase == 'Planning Phase':
        if self.total_clubs_participated < 10:
            improvement_areas.append('increase_club_participation')
        improvement_areas.append('optimize_budget_allocation')
    elif current_phase == 'Registration Phase':
        if Club_member.objects.filter(session=self.session).count() < 50:
            improvement_areas.append('enhance_registration_promotion')
        improvement_areas.append('streamline_registration_process')
    elif current_phase == 'Active Phase':
        if self.total_events_conducted < 15:
            improvement_areas.append('increase_event_frequency')
        improvement_areas.append('improve_event_quality')
    
    return improvement_areas

def _track_session_milestones(self, current_date):
    """Helper method to track session milestones"""
    milestones = [
        {
            'milestone': 'Registration Opens',
            'date': self.registration_start,
            'status': 'completed' if current_date >= self.registration_start else 'pending',
            'importance': 'high'
        },
        {
            'milestone': 'Budget Submission Deadline',
            'date': self.budget_submission_deadline,
            'status': 'completed' if current_date >= self.budget_submission_deadline else 'pending',
            'importance': 'critical'
        },
        {
            'milestone': 'Activities Begin',
            'date': self.activity_start,
            'status': 'completed' if current_date >= self.activity_start else 'pending',
            'importance': 'high'
        },
        {
            'milestone': 'Activities End',
            'date': self.activity_end,
            'status': 'completed' if current_date >= self.activity_end else 'pending',
            'importance': 'high'
        },
        {
            'milestone': 'Report Submission Deadline',
            'date': self.report_submission_deadline,
            'status': 'completed' if current_date >= self.report_submission_deadline else 'pending',
            'importance': 'critical'
        }
    ]
    
    # Calculate milestone completion rate
    completed_milestones = sum(1 for m in milestones if m['status'] == 'completed')
    completion_rate = (completed_milestones / len(milestones)) * 100
    
    return {
        'milestones': milestones,
        'completion_rate': round(completion_rate, 2),
        'next_milestone': self._get_next_milestone(milestones, current_date),
        'milestone_health': self._assess_milestone_health(milestones, current_date)
    }

def _get_next_milestone(self, milestones, current_date):
    """Helper method to get next milestone"""
    pending_milestones = [m for m in milestones if m['status'] == 'pending']
    
    if pending_milestones:
        next_milestone = min(pending_milestones, key=lambda x: x['date'])
        days_until = (next_milestone['date'].date() - current_date.date()).days if hasattr(next_milestone['date'], 'date') else (next_milestone['date'] - current_date.date()).days
        
        return {
            'milestone': next_milestone['milestone'],
            'date': next_milestone['date'],
            'days_until': days_until,
            'importance': next_milestone['importance']
        }
    
    return None

def _assess_milestone_health(self, milestones, current_date):
    """Helper method to assess milestone health"""
    overdue_milestones = []
    upcoming_critical = []
    
    for milestone in milestones:
        if milestone['status'] == 'pending':
            milestone_date = milestone['date'].date() if hasattr(milestone['date'], 'date') else milestone['date']
            days_until = (milestone_date - current_date.date()).days
            
            if days_until < 0:
                overdue_milestones.append(milestone)
            elif days_until <= 7 and milestone['importance'] == 'critical':
                upcoming_critical.append(milestone)
    
    if overdue_milestones:
        health_status = 'critical'
    elif upcoming_critical:
        health_status = 'warning'
    elif len([m for m in milestones if m['status'] == 'completed']) > len(milestones) * 0.5:
        health_status = 'good'
    else:
        health_status = 'normal'
    
    return {
        'health_status': health_status,
        'overdue_count': len(overdue_milestones),
        'upcoming_critical_count': len(upcoming_critical),
        'overdue_milestones': overdue_milestones,
        'upcoming_critical': upcoming_critical
    }

def _identify_upcoming_deadlines(self, current_date):
    """Helper method to identify upcoming deadlines"""
    deadlines = []
    
    # Session deadlines
    session_deadlines = [
        ('Registration End', self.registration_end),
        ('Budget Submission', self.budget_submission_deadline),
        ('Activities End', self.activity_end),
        ('Report Submission', self.report_submission_deadline)
    ]
    
    for deadline_name, deadline_date in session_deadlines:
        if deadline_date > current_date:
            days_until = (deadline_date.date() - current_date.date()).days if hasattr(deadline_date, 'date') else (deadline_date - current_date.date()).days
            
            if days_until <= 30:  # Next 30 days
                urgency = 'high' if days_until <= 7 else 'medium' if days_until <= 14 else 'low'
                deadlines.append({
                    'deadline': deadline_name,
                    'date': deadline_date,
                    'days_until': days_until,
                    'urgency': urgency
                })
    
    # Sort by urgency and date
    deadlines.sort(key=lambda x: (x['urgency'] == 'low', x['urgency'] == 'medium', x['days_until']))
    
    return deadlines

def _assess_timeline_health(self, timeline_phases, current_phase_info):
    """Helper method to assess overall timeline health"""
    health_factors = []
    health_score = 75  # Base score
    
    # Phase progression health
    completed_phases = sum(1 for phase in timeline_phases if phase['status'] == 'completed')
    total_phases = len(timeline_phases)
    
    if completed_phases / total_phases > 0.5:
        health_factors.append('good_phase_progression')
        health_score += 10
    elif completed_phases / total_phases < 0.25:
        health_factors.append('slow_phase_progression')
        health_score -= 15
    
    # Current phase progress
    current_progress = current_phase_info.get('progress_percentage', 0)
    if current_progress > 75:
        health_factors.append('strong_current_phase_progress')
        health_score += 10
    elif current_progress < 25:
        health_factors.append('early_current_phase')
        health_score -= 5
    
    # Performance factors
    if self.overall_session_rating > 80:
        health_factors.append('excellent_session_performance')
        health_score += 15
    elif self.overall_session_rating < 60:
        health_factors.append('poor_session_performance')
        health_score -= 20
    
    # Determine health level
    if health_score >= 85:
        health_level = 'excellent'
    elif health_score >= 70:
        health_level = 'good'
    elif health_score >= 55:
        health_level = 'moderate'
    else:
        health_level = 'poor'
    
    return {
        'health_score': health_score,
        'health_level': health_level,
        'health_factors': health_factors,
        'timeline_efficiency': self._calculate_timeline_efficiency(timeline_phases)
    }

def _calculate_timeline_efficiency(self, timeline_phases):
    """Helper method to calculate timeline efficiency"""
    total_planned_duration = sum(phase['duration_days'] for phase in timeline_phases)
    
    # Efficiency based on phase completion and duration optimization
    efficiency_factors = []
    
    # Check for appropriate phase durations
    for phase in timeline_phases:
        if phase['phase_name'] == 'Planning Phase' and phase['duration_days'] < 7:
            efficiency_factors.append('insufficient_planning_time')
        elif phase['phase_name'] == 'Registration Phase' and phase['duration_days'] < 14:
            efficiency_factors.append('short_registration_period')
        elif phase['phase_name'] == 'Active Phase' and phase['duration_days'] < 60:
            efficiency_factors.append('limited_activity_period')
    
    # Calculate efficiency score
    if not efficiency_factors:
        efficiency_score = 90
    elif len(efficiency_factors) == 1:
        efficiency_score = 75
    else:
        efficiency_score = 60
    
    return {
        'efficiency_score': efficiency_score,
        'efficiency_factors': efficiency_factors,
        'timeline_optimization_potential': 100 - efficiency_score
    }

def _generate_timeline_recommendations(self, current_phase_info, upcoming_deadlines):
    """Helper method to generate timeline recommendations"""
    recommendations = []
    
    # Phase-specific recommendations
    current_phase = current_phase_info.get('current_phase')
    
    if current_phase == 'Planning Phase':
        recommendations.extend([
            'finalize_session_budget_allocations',
            'coordinate_with_academic_calendar',
            'prepare_registration_materials'
        ])
    elif current_phase == 'Registration Phase':
        recommendations.extend([
            'promote_registration_actively',
            'monitor_registration_numbers',
            'prepare_club_orientation_materials'
        ])
    elif current_phase == 'Active Phase':
        recommendations.extend([
            'monitor_event_execution_progress',
            'track_budget_utilization',
            'maintain_member_engagement'
        ])
    elif current_phase == 'Evaluation Phase':
        recommendations.extend([
            'collect_session_feedback',
            'prepare_completion_reports',
            'plan_next_session_improvements'
        ])
    
    # Deadline-based recommendations
    high_urgency_deadlines = [d for d in upcoming_deadlines if d['urgency'] == 'high']
    if high_urgency_deadlines:
        recommendations.extend([
            'prioritize_upcoming_high_urgency_deadlines',
            'establish_deadline_monitoring_system'
        ])
    
    # Performance-based recommendations
    if self.overall_session_rating < 70:
        recommendations.extend([
            'implement_session_improvement_plan',
            'seek_coordinator_support'
        ])
    
    return recommendations

def get_session_analytics(self):
    """
    CORE LOGIC: Comprehensive session analytics and performance insights
    
    HOW IT WORKS:
    1. Analyzes session performance across multiple dimensions
    2. Provides comparative analysis with historical sessions
    3. Generates insights for session optimization
    4. Tracks key performance indicators and trends
    
    BUSINESS PURPOSE:
    - Session performance evaluation and optimization
    - Strategic planning for future sessions
    - Resource allocation and utilization analysis
    - Institutional planning and policy development
    """
    # Performance analytics
    performance_analytics = self._analyze_session_performance()
    
    # Participation analytics
    participation_analytics = self._analyze_participation_patterns()
    
    # Financial analytics
    financial_analytics = self._analyze_session_financials()
    
    # Comparative analytics
    comparative_analytics = self._perform_comparative_session_analysis()
    
    # Trend analytics
    trend_analytics = self._analyze_session_trends()
    
    return {
        'session_summary': {
            'session_id': self.session,
            'academic_year': self.academic_year,
            'session_status': self.session_status,
            'overall_rating': self.overall_session_rating,
            'analysis_date': datetime.datetime.now().strftime('%Y-%m-%d')
        },
        'performance_analytics': performance_analytics,
        'participation_analytics': participation_analytics,
        'financial_analytics': financial_analytics,
        'comparative_analytics': comparative_analytics,
        'trend_analytics': trend_analytics,
        'strategic_insights': self._generate_strategic_insights(
            performance_analytics, participation_analytics, financial_analytics
        ),
        'recommendations': self._generate_session_optimization_recommendations(
            performance_analytics, participation_analytics, financial_analytics
        )
    }

def _analyze_session_performance(self):
    """Helper method to analyze session performance"""
    # Multi-dimensional performance analysis
    performance_dimensions = {
        'engagement_performance': {
            'score': self.member_engagement_score,
            'level': self._categorize_performance_level(self.member_engagement_score),
            'factors': self._identify_engagement_performance_factors()
        },
        'financial_performance': {
            'score': self.financial_efficiency_score,
            'level': self._categorize_performance_level(self.financial_efficiency_score),
            'factors': self._identify_financial_performance_factors()
        },
        'activity_performance': {
            'score': self.activity_success_rate,
            'level': self._categorize_performance_level(self.activity_success_rate),
            'factors': self._identify_activity_performance_factors()
        },
        'overall_performance': {
            'score': self.overall_session_rating,
            'level': self._categorize_performance_level(self.overall_session_rating),
            'factors': self._identify_overall_performance_factors()
        }
    }
    
    # Performance summary
    avg_performance = sum(dim['score'] for dim in performance_dimensions.values()) / len(performance_dimensions)
    
    return {
        'performance_dimensions': performance_dimensions,
        'average_performance': round(avg_performance, 2),
        'performance_strengths': self._identify_performance_strengths(performance_dimensions),
        'performance_weaknesses': self._identify_performance_weaknesses(performance_dimensions),
        'improvement_priorities': self._prioritize_performance_improvements(performance_dimensions)
    }

def _categorize_performance_level(self, score):
    """Helper method to categorize performance level"""
    if score >= 85:
        return 'excellent'
    elif score >= 70:
        return 'good'
    elif score >= 55:
        return 'moderate'
    elif score >= 40:
        return 'poor'
    else:
        return 'very_poor'

def _identify_engagement_performance_factors(self):
    """Helper method to identify engagement performance factors"""
    factors = []
    
    if self.member_engagement_score >= 80:
        factors.extend(['high_member_participation', 'strong_leadership_engagement'])
    elif self.member_engagement_score >= 60:
        factors.extend(['moderate_engagement', 'adequate_participation'])
    else:
        factors.extend(['low_engagement', 'limited_participation'])
    
    # Additional factors based on metrics
    total_members = Club_member.objects.filter(session=self.session).count()
    if total_members > 100:
        factors.append('large_member_base')
    elif total_members < 30:
        factors.append('small_member_base')
    
    return factors

def _identify_financial_performance_factors(self):
    """Helper method to identify financial performance factors"""
    factors = []
    
    utilization_rate = (self.total_budget_utilized / self.total_budget_allocated) * 100 if self.total_budget_allocated > 0 else 0
    
    if 80 <= utilization_rate <= 95:
        factors.extend(['optimal_budget_utilization', 'effective_financial_planning'])
    elif utilization_rate > 95:
        factors.extend(['high_budget_utilization', 'potential_overspending_risk'])
    elif utilization_rate < 60:
        factors.extend(['low_budget_utilization', 'underutilized_resources'])
    
    if self.total_budget_allocated > 100000:
        factors.append('large_budget_management')
    elif self.total_budget_allocated < 20000:
        factors.append('limited_budget_constraints')
    
    return factors

def _identify_activity_performance_factors(self):
    """Helper method to identify activity performance factors"""
    factors = []
    
    if self.total_events_conducted > 20:
        factors.extend(['high_activity_level', 'diverse_event_portfolio'])
    elif self.total_events_conducted > 10:
        factors.extend(['moderate_activity_level', 'regular_event_execution'])
    else:
        factors.extend(['low_activity_level', 'limited_event_execution'])
    
    events_per_club = self.total_events_conducted / max(1, self.total_clubs_participated)
    if events_per_club > 2:
        factors.append('high_club_activity_intensity')
    elif events_per_club < 1:
        factors.append('low_club_activity_intensity')
    
    return factors

def _identify_overall_performance_factors(self):
    """Helper method to identify overall performance factors"""
    factors = []
    
    # Balanced performance factors
    performance_scores = [
        self.member_engagement_score,
        self.financial_efficiency_score,
        self.activity_success_rate
    ]
    
    score_variance = max(performance_scores) - min(performance_scores)
    
    if score_variance < 15:
        factors.append('balanced_performance_across_dimensions')
    elif score_variance > 30:
        factors.append('unbalanced_performance_dimensions')
    
    # Trend factors
    if self.overall_session_rating > 80:
        factors.append('excellent_overall_execution')
    elif self.overall_session_rating < 60:
        factors.append('poor_overall_execution')
    
    return factors

def _identify_performance_strengths(self, performance_dimensions):
    """Helper method to identify performance strengths"""
    strengths = []
    
    for dimension, data in performance_dimensions.items():
        if data['level'] in ['excellent', 'good']:
            strengths.append({
                'dimension': dimension,
                'score': data['score'],
                'level': data['level']
            })
    
    return strengths

def _identify_performance_weaknesses(self, performance_dimensions):
    """Helper method to identify performance weaknesses"""
    weaknesses = []
    
    for dimension, data in performance_dimensions.items():
        if data['level'] in ['poor', 'very_poor']:
            weaknesses.append({
                'dimension': dimension,
                'score': data['score'],
                'level': data['level']
            })
    
    return weaknesses

def _prioritize_performance_improvements(self, performance_dimensions):
    """Helper method to prioritize performance improvements"""
    # Sort dimensions by score (lowest first for priority)
    sorted_dimensions = sorted(
        performance_dimensions.items(),
        key=lambda x: x[1]['score']
    )
    
    priorities = []
    for dimension, data in sorted_dimensions[:3]:  # Top 3 priorities
        if data['score'] < 70:  # Only if needs improvement
            priorities.append({
                'dimension': dimension,
                'current_score': data['score'],
                'improvement_potential': 85 - data['score'],  # Target score of 85
                'priority_level': 'high' if data['score'] < 50 else 'medium'
            })
    
    return priorities

def _analyze_participation_patterns(self):
    """Helper method to analyze participation patterns"""
    # Club participation analysis
    club_participation = self._analyze_club_participation()
    
    # Member participation analysis
    member_participation = self._analyze_member_participation()
    
    # Leadership participation analysis
    leadership_participation = self._analyze_leadership_participation()
    
    # Participation trends
    participation_trends = self._analyze_participation_trends()
    
    return {
        'club_participation': club_participation,
        'member_participation': member_participation,
        'leadership_participation': leadership_participation,
        'participation_trends': participation_trends,
        'participation_health': self._assess_participation_health(
            club_participation, member_participation, leadership_participation
        )
    }

def _analyze_club_participation(self):
    """Helper method to analyze club participation"""
    # Club participation metrics
    active_clubs = Club_member.objects.filter(
        session=self.session,
        membership_status='active'
    ).values('club').distinct().count()
    
    total_clubs_with_budget = Club_budget.objects.filter(session=self.session).count()
    
    participation_rate = (active_clubs / max(1, total_clubs_with_budget)) * 100
    
    # Club activity distribution
    club_activity_levels = []
    for club_budget in Club_budget.objects.filter(session=self.session):
        club_members = Club_member.objects.filter(session=self.session, club=club_budget.club).count()
        club_events = Event_info.objects.filter(session=self.session, organizing_club=club_budget.club).count()
        
        activity_level = 'high' if club_events > 3 else 'medium' if club_events > 1 else 'low'
        club_activity_levels.append({
            'club': club_budget.club.club_name,
            'member_count': club_members,
            'event_count': club_events,
            'activity_level': activity_level
        })
    
    return {
        'total_participating_clubs': active_clubs,
        'participation_rate': round(participation_rate, 2),
        'club_activity_distribution': club_activity_levels,
        'average_club_size': self._calculate_average_club_size(),
        'club_engagement_diversity': self._assess_club_engagement_diversity(club_activity_levels)
    }

def _calculate_average_club_size(self):
    """Helper method to calculate average club size"""
    club_sizes = Club_member.objects.filter(session=self.session).values('club').annotate(
        member_count=Count('id')
    ).values_list('member_count', flat=True)
    
    return round(sum(club_sizes) / len(club_sizes), 1) if club_sizes else 0

def _assess_club_engagement_diversity(self, club_activity_levels):
    """Helper method to assess club engagement diversity"""
    if not club_activity_levels:
        return 'no_data'
    
    activity_distribution = {}
    for club in club_activity_levels:
        level = club['activity_level']
        activity_distribution[level] = activity_distribution.get(level, 0) + 1
    
    total_clubs = len(club_activity_levels)
    diversity_score = 0
    
    # Calculate diversity based on distribution
    for level, count in activity_distribution.items():
        percentage = (count / total_clubs) * 100
        if level == 'high':
            diversity_score += percentage * 2
        elif level == 'medium':
            diversity_score += percentage * 1.5
        else:
            diversity_score += percentage * 0.5
    
    if diversity_score >= 150:
        return 'high_diversity'
    elif diversity_score >= 100:
        return 'moderate_diversity'
    else:
        return 'low_diversity'
```

This completes Session_info model with comprehensive session management capabilities.