# Complete Gymkhana Module System Documentation - Fusion IIIT (Part 9)

## Database Models Analysis with Business Logic (Continued)

### Event_info Model

```python
class Event_info(models.Model):
    """
    CORE PURPOSE: Comprehensive event management lifecycle with planning, execution, and evaluation
    
    BUSINESS LOGIC:
    - Manages complete event lifecycle from planning to post-event analysis
    - Provides event analytics and performance metrics
    - Handles event resource allocation and budget management
    - Supports event registration and participant management
    - Generates event insights and recommendations
    
    INTEGRATION POINTS:
    - Links with Club_budget for event financial management
    - Coordinates with Session_info for academic calendar alignment
    - Integrates with Club_member for participant tracking
    - Connects with Core_team for event leadership roles
    """
    
    event_id = models.CharField(max_length=20, unique=True)
    event_name = models.CharField(max_length=100)
    event_description = models.TextField()
    organizing_club = models.ForeignKey(Club_info, on_delete=models.CASCADE)
    session = models.CharField(max_length=20)
    
    # Event timing and scheduling
    event_date = models.DateTimeField()
    event_end_date = models.DateTimeField(null=True, blank=True)
    registration_start = models.DateTimeField()
    registration_end = models.DateTimeField()
    venue = models.CharField(max_length=200)
    venue_capacity = models.IntegerField(default=0)
    
    # Event categorization
    event_type = models.CharField(max_length=30, choices=[
        ('technical', 'Technical Event'),
        ('cultural', 'Cultural Event'),
        ('sports', 'Sports Event'),
        ('workshop', 'Workshop'),
        ('seminar', 'Seminar'),
        ('competition', 'Competition'),
        ('social', 'Social Event'),
        ('fundraising', 'Fundraising'),
        ('community_service', 'Community Service'),
        ('fest', 'Festival Event')
    ])
    
    event_category = models.CharField(max_length=20, choices=[
        ('intra_college', 'Intra College'),
        ('inter_college', 'Inter College'),
        ('national', 'National Level'),
        ('international', 'International Level'),
        ('online', 'Online Event'),
        ('hybrid', 'Hybrid Event')
    ])
    
    # Event status and lifecycle
    event_status = models.CharField(max_length=20, choices=[
        ('planning', 'Planning'),
        ('approved', 'Approved'),
        ('registration_open', 'Registration Open'),
        ('registration_closed', 'Registration Closed'),
        ('ongoing', 'Ongoing'),
        ('completed', 'Completed'),
        ('cancelled', 'Cancelled'),
        ('postponed', 'Postponed')
    ], default='planning')
    
    # Registration and participation
    max_participants = models.IntegerField(default=0)
    current_registrations = models.IntegerField(default=0)
    actual_participants = models.IntegerField(default=0)
    registration_fee = models.DecimalField(max_digits=8, decimal_places=2, default=0)
    
    # Event budget and finances
    estimated_budget = models.DecimalField(max_digits=10, decimal_places=2, default=0)
    actual_expenditure = models.DecimalField(max_digits=10, decimal_places=2, default=0)
    revenue_generated = models.DecimalField(max_digits=10, decimal_places=2, default=0)
    sponsorship_amount = models.DecimalField(max_digits=10, decimal_places=2, default=0)
    
    # Event coordination and management
    event_coordinator = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True)
    co_coordinators = models.ManyToManyField(User, related_name='co_coordinated_events', blank=True)
    organizing_committee = models.JSONField(default=list, blank=True)
    
    # Event logistics
    equipment_requirements = models.JSONField(default=list, blank=True)
    technical_requirements = models.JSONField(default=list, blank=True)
    catering_requirements = models.JSONField(default=dict, blank=True)
    transportation_requirements = models.JSONField(default=dict, blank=True)
    
    # Event promotion and marketing
    promotion_channels = models.JSONField(default=list, blank=True)
    marketing_budget = models.DecimalField(max_digits=8, decimal_places=2, default=0)
    social_media_reach = models.IntegerField(default=0)
    promotional_materials = models.JSONField(default=list, blank=True)
    
    # Event evaluation and feedback
    participant_feedback_score = models.FloatField(default=0.0)
    organizer_feedback_score = models.FloatField(default=0.0)
    overall_event_rating = models.FloatField(default=0.0)
    feedback_count = models.IntegerField(default=0)
    
    # Event outcomes and impact
    learning_outcomes = models.JSONField(default=list, blank=True)
    skill_development_areas = models.JSONField(default=list, blank=True)
    networking_opportunities = models.IntegerField(default=0)
    industry_connections = models.IntegerField(default=0)
    
    # Event analytics and metrics
    attendance_rate = models.FloatField(default=0.0)
    engagement_score = models.FloatField(default=0.0)
    success_metrics = models.JSONField(default=dict, blank=True)
    roi_percentage = models.FloatField(default=0.0)
    
    # Quality and compliance
    safety_compliance_score = models.FloatField(default=0.0)
    quality_standards_met = models.BooleanField(default=False)
    institutional_guidelines_followed = models.BooleanField(default=True)
    
    # External collaborations
    external_collaborators = models.JSONField(default=list, blank=True)
    industry_partners = models.JSONField(default=list, blank=True)
    guest_speakers = models.JSONField(default=list, blank=True)
    
    # Documentation and media
    event_documentation = models.JSONField(default=dict, blank=True)
    media_coverage = models.JSONField(default=list, blank=True)
    photo_gallery_links = models.JSONField(default=list, blank=True)
    video_documentation = models.JSONField(default=list, blank=True)
    
    # Follow-up and continuity
    follow_up_activities = models.JSONField(default=list, blank=True)
    future_event_recommendations = models.TextField(blank=True)
    legacy_planning = models.JSONField(default=dict, blank=True)
    
    # Metadata
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    approved_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True, related_name='approved_events')
    approval_date = models.DateTimeField(null=True, blank=True)
    
    class Meta:
        db_table = 'gymkhana_event_info'
        ordering = ['-event_date']
        verbose_name = 'Event Information'
        verbose_name_plural = 'Event Information'
        unique_together = [['event_name', 'organizing_club', 'session']]
    
    def __str__(self):
        return f"{self.event_name} - {self.organizing_club.club_name}"

def save(self, *args, **kwargs):
    """Enhanced save method with event management logic"""
    # Generate event ID if not provided
    if not self.event_id:
        self.event_id = self._generate_event_id()
    
    # Validate event dates
    self._validate_event_dates()
    
    # Update event status based on current date
    self._update_event_status()
    
    # Calculate attendance rate
    self._calculate_attendance_rate()
    
    # Calculate financial metrics
    self._calculate_financial_metrics()
    
    # Update engagement score
    self._calculate_engagement_score()
    
    # Calculate overall event rating
    self._calculate_overall_rating()
    
    super().save(*args, **kwargs)
    
    # Post-save actions
    self._handle_status_transitions()

def _generate_event_id(self):
    """Helper method to generate unique event ID"""
    import uuid
    from datetime import datetime
    
    # Format: CLUB_YEAR_SEQUENCE
    club_prefix = self.organizing_club.club_name[:3].upper()
    year_suffix = datetime.now().year
    sequence = Event_info.objects.filter(
        organizing_club=self.organizing_club,
        event_date__year=year_suffix
    ).count() + 1
    
    return f"{club_prefix}_{year_suffix}_{sequence:03d}"

def _validate_event_dates(self):
    """Helper method to validate event dates"""
    from django.core.exceptions import ValidationError
    from django.utils import timezone
    
    if self.event_end_date and self.event_date >= self.event_end_date:
        raise ValidationError("Event start date must be before end date")
    
    if self.registration_start >= self.registration_end:
        raise ValidationError("Registration start must be before registration end")
    
    if self.registration_end > self.event_date:
        raise ValidationError("Registration must end before event starts")
    
    # Check for reasonable planning time
    if self.event_date <= timezone.now() + timezone.timedelta(days=7) and self.event_status == 'planning':
        # Warning for short planning time, but don't block
        pass

def _update_event_status(self):
    """Helper method to update event status based on current date"""
    from django.utils import timezone
    current_date = timezone.now()
    
    if self.event_status not in ['cancelled', 'postponed']:
        if current_date < self.registration_start:
            self.event_status = 'approved' if self.event_status != 'planning' else 'planning'
        elif self.registration_start <= current_date < self.registration_end:
            self.event_status = 'registration_open'
        elif self.registration_end <= current_date < self.event_date:
            self.event_status = 'registration_closed'
        elif self.event_date <= current_date:
            if self.event_end_date and current_date < self.event_end_date:
                self.event_status = 'ongoing'
            else:
                end_time = self.event_end_date or self.event_date + timezone.timedelta(hours=3)
                if current_date >= end_time:
                    self.event_status = 'completed'

def _calculate_attendance_rate(self):
    """Helper method to calculate attendance rate"""
    if self.current_registrations > 0:
        self.attendance_rate = (self.actual_participants / self.current_registrations) * 100
    else:
        self.attendance_rate = 0.0

def _calculate_financial_metrics(self):
    """Helper method to calculate financial metrics"""
    # Calculate ROI
    total_investment = self.actual_expenditure + self.marketing_budget
    total_return = self.revenue_generated + self.sponsorship_amount
    
    if total_investment > 0:
        self.roi_percentage = ((total_return - total_investment) / total_investment) * 100
    else:
        self.roi_percentage = 0.0

def _calculate_engagement_score(self):
    """Helper method to calculate engagement score"""
    # Multi-factor engagement calculation
    factors = []
    
    # Registration factor (40%)
    if self.max_participants > 0:
        registration_factor = min(100, (self.current_registrations / self.max_participants) * 100)
    else:
        registration_factor = 50  # Default if no limit
    factors.append(('registration', registration_factor, 0.4))
    
    # Attendance factor (30%)
    attendance_factor = self.attendance_rate
    factors.append(('attendance', attendance_factor, 0.3))
    
    # Feedback factor (20%)
    feedback_factor = self.participant_feedback_score * 20  # Convert to 0-100 scale
    factors.append(('feedback', feedback_factor, 0.2))
    
    # Social media factor (10%)
    social_media_factor = min(100, self.social_media_reach / 100)  # 1 point per 100 reach
    factors.append(('social_media', social_media_factor, 0.1))
    
    # Calculate weighted engagement score
    self.engagement_score = sum(score * weight for _, score, weight in factors)

def _calculate_overall_rating(self):
    """Helper method to calculate overall event rating"""
    # Comprehensive rating based on multiple criteria
    rating_components = []
    
    # Participant satisfaction (30%)
    if self.participant_feedback_score > 0:
        rating_components.append(('participant_satisfaction', self.participant_feedback_score, 0.3))
    
    # Organizer satisfaction (20%)
    if self.organizer_feedback_score > 0:
        rating_components.append(('organizer_satisfaction', self.organizer_feedback_score, 0.2))
    
    # Engagement score (25%)
    rating_components.append(('engagement', self.engagement_score / 20, 0.25))  # Convert to 0-5 scale
    
    # Financial performance (15%)
    financial_score = self._calculate_financial_performance_score()
    rating_components.append(('financial', financial_score, 0.15))
    
    # Attendance performance (10%)
    attendance_score = min(5.0, self.attendance_rate / 20)  # Convert to 0-5 scale
    rating_components.append(('attendance', attendance_score, 0.1))
    
    # Calculate weighted overall rating
    if rating_components:
        self.overall_event_rating = sum(score * weight for _, score, weight in rating_components)
    else:
        self.overall_event_rating = 0.0

def _calculate_financial_performance_score(self):
    """Helper method to calculate financial performance score"""
    if self.estimated_budget == 0:
        return 3.0  # Default neutral score
    
    # Budget adherence score
    budget_variance = abs(self.actual_expenditure - self.estimated_budget) / self.estimated_budget * 100
    
    if budget_variance <= 10:
        budget_score = 5.0
    elif budget_variance <= 20:
        budget_score = 4.0
    elif budget_variance <= 30:
        budget_score = 3.0
    else:
        budget_score = 2.0
    
    # ROI consideration
    if self.roi_percentage > 50:
        roi_bonus = 0.5
    elif self.roi_percentage > 0:
        roi_bonus = 0.3
    else:
        roi_bonus = 0.0
    
    return min(5.0, budget_score + roi_bonus)

def _handle_status_transitions(self):
    """Helper method to handle event status transitions"""
    if self.event_status == 'completed' and self._just_completed():
        self._initiate_post_event_process()
    elif self.event_status == 'registration_open' and self._just_opened_registration():
        self._initiate_registration_process()

def _just_completed(self):
    """Helper method to check if event just completed"""
    if self.pk:
        try:
            old_instance = Event_info.objects.get(pk=self.pk)
            return old_instance.event_status != 'completed'
        except Event_info.DoesNotExist:
            return False
    return False

def _just_opened_registration(self):
    """Helper method to check if registration just opened"""
    if self.pk:
        try:
            old_instance = Event_info.objects.get(pk=self.pk)
            return old_instance.event_status != 'registration_open'
        except Event_info.DoesNotExist:
            return False
    return False

def _initiate_post_event_process(self):
    """Helper method to initiate post-event process"""
    # Generate event completion reports
    self._generate_event_report()
    
    # Collect feedback
    self._initiate_feedback_collection()
    
    # Update club metrics
    self._update_club_event_metrics()

def _initiate_registration_process(self):
    """Helper method to initiate registration process"""
    # Send registration notifications
    self._send_registration_notifications()
    
    # Initialize registration tracking
    self._initialize_registration_tracking()

def _generate_event_report(self):
    """Helper method to generate event report"""
    # This would generate comprehensive event reports
    # Implementation depends on reporting requirements
    pass

def _initiate_feedback_collection(self):
    """Helper method to initiate feedback collection"""
    # Start feedback collection process
    # Implementation depends on feedback system
    pass

def _update_club_event_metrics(self):
    """Helper method to update club event metrics"""
    # Update organizing club's event statistics
    club_budget = Club_budget.objects.filter(
        club=self.organizing_club,
        session=self.session
    ).first()
    
    if club_budget:
        # Update event count and metrics
        club_budget.save()  # This will trigger recalculation

def _send_registration_notifications(self):
    """Helper method to send registration notifications"""
    # Send notifications about registration opening
    # Implementation depends on notification system
    pass

def _initialize_registration_tracking(self):
    """Helper method to initialize registration tracking"""
    # Initialize registration tracking systems
    # Implementation depends on tracking requirements
    pass

def get_event_analytics(self):
    """
    CORE LOGIC: Comprehensive event analytics and performance insights
    
    HOW IT WORKS:
    1. Analyzes event performance across multiple dimensions
    2. Provides comparative analysis with similar events
    3. Generates insights for event optimization
    4. Tracks key performance indicators and trends
    
    BUSINESS PURPOSE:
    - Event performance evaluation and optimization
    - Resource allocation and planning insights
    - Success factor identification and replication
    - Future event planning and improvement
    """
    # Performance analytics
    performance_analytics = self._analyze_event_performance()
    
    # Financial analytics
    financial_analytics = self._analyze_event_financials()
    
    # Participation analytics
    participation_analytics = self._analyze_participation_patterns()
    
    # Engagement analytics
    engagement_analytics = self._analyze_engagement_metrics()
    
    # Comparative analytics
    comparative_analytics = self._perform_comparative_event_analysis()
    
    # Success factor analysis
    success_factors = self._analyze_success_factors()
    
    return {
        'event_summary': {
            'event_id': self.event_id,
            'event_name': self.event_name,
            'organizing_club': self.organizing_club.club_name,
            'event_type': self.event_type,
            'event_status': self.event_status,
            'overall_rating': self.overall_event_rating,
            'analysis_date': datetime.datetime.now().strftime('%Y-%m-%d')
        },
        'performance_analytics': performance_analytics,
        'financial_analytics': financial_analytics,
        'participation_analytics': participation_analytics,
        'engagement_analytics': engagement_analytics,
        'comparative_analytics': comparative_analytics,
        'success_factors': success_factors,
        'improvement_recommendations': self._generate_event_improvement_recommendations(),
        'future_planning_insights': self._generate_future_planning_insights()
    }

def _analyze_event_performance(self):
    """Helper method to analyze event performance"""
    # Multi-dimensional performance analysis
    performance_dimensions = {
        'attendance_performance': {
            'metric': self.attendance_rate,
            'level': self._categorize_attendance_level(self.attendance_rate),
            'target_achievement': self._calculate_attendance_target_achievement()
        },
        'engagement_performance': {
            'metric': self.engagement_score,
            'level': self._categorize_engagement_level(self.engagement_score),
            'engagement_factors': self._identify_engagement_drivers()
        },
        'satisfaction_performance': {
            'metric': self.participant_feedback_score,
            'level': self._categorize_satisfaction_level(self.participant_feedback_score),
            'satisfaction_drivers': self._identify_satisfaction_drivers()
        },
        'execution_performance': {
            'metric': self.organizer_feedback_score,
            'level': self._categorize_execution_level(self.organizer_feedback_score),
            'execution_factors': self._identify_execution_factors()
        }
    }
    
    # Overall performance assessment
    avg_performance = sum(dim['metric'] for dim in performance_dimensions.values() if dim['metric'] > 0) / len([dim for dim in performance_dimensions.values() if dim['metric'] > 0])
    
    return {
        'performance_dimensions': performance_dimensions,
        'average_performance': round(avg_performance, 2),
        'performance_strengths': self._identify_performance_strengths(performance_dimensions),
        'performance_areas_for_improvement': self._identify_performance_improvement_areas(performance_dimensions),
        'performance_trends': self._analyze_performance_trends()
    }

def _categorize_attendance_level(self, attendance_rate):
    """Helper method to categorize attendance level"""
    if attendance_rate >= 90:
        return 'excellent'
    elif attendance_rate >= 75:
        return 'good'
    elif attendance_rate >= 60:
        return 'moderate'
    elif attendance_rate >= 40:
        return 'poor'
    else:
        return 'very_poor'

def _categorize_engagement_level(self, engagement_score):
    """Helper method to categorize engagement level"""
    if engagement_score >= 80:
        return 'highly_engaged'
    elif engagement_score >= 65:
        return 'well_engaged'
    elif engagement_score >= 50:
        return 'moderately_engaged'
    elif engagement_score >= 35:
        return 'poorly_engaged'
    else:
        return 'very_poorly_engaged'

def _categorize_satisfaction_level(self, satisfaction_score):
    """Helper method to categorize satisfaction level"""
    if satisfaction_score >= 4.5:
        return 'extremely_satisfied'
    elif satisfaction_score >= 3.5:
        return 'satisfied'
    elif satisfaction_score >= 2.5:
        return 'neutral'
    elif satisfaction_score >= 1.5:
        return 'dissatisfied'
    else:
        return 'extremely_dissatisfied'

def _categorize_execution_level(self, execution_score):
    """Helper method to categorize execution level"""
    if execution_score >= 4.5:
        return 'excellent_execution'
    elif execution_score >= 3.5:
        return 'good_execution'
    elif execution_score >= 2.5:
        return 'adequate_execution'
    elif execution_score >= 1.5:
        return 'poor_execution'
    else:
        return 'very_poor_execution'

def _calculate_attendance_target_achievement(self):
    """Helper method to calculate attendance target achievement"""
    if self.max_participants > 0:
        target_achievement = (self.actual_participants / self.max_participants) * 100
        return round(target_achievement, 2)
    return 0.0

def _identify_engagement_drivers(self):
    """Helper method to identify engagement drivers"""
    drivers = []
    
    if self.current_registrations > self.max_participants * 0.9:
        drivers.append('high_registration_demand')
    
    if self.social_media_reach > 1000:
        drivers.append('strong_social_media_presence')
    
    if self.guest_speakers:
        drivers.append('quality_guest_speakers')
    
    if self.networking_opportunities > 50:
        drivers.append('excellent_networking_opportunities')
    
    if self.registration_fee == 0:
        drivers.append('free_participation')
    
    return drivers

def _identify_satisfaction_drivers(self):
    """Helper method to identify satisfaction drivers"""
    drivers = []
    
    if self.venue_capacity >= self.actual_participants * 1.2:
        drivers.append('adequate_venue_capacity')
    
    if self.learning_outcomes:
        drivers.append('clear_learning_outcomes')
    
    if self.catering_requirements:
        drivers.append('catering_provided')
    
    if self.industry_connections > 0:
        drivers.append('industry_connections')
    
    if self.quality_standards_met:
        drivers.append('quality_standards_maintained')
    
    return drivers

def _identify_execution_factors(self):
    """Helper method to identify execution factors"""
    factors = []
    
    if self.organizing_committee:
        factors.append('well_organized_committee')
    
    if self.safety_compliance_score > 4.0:
        factors.append('high_safety_compliance')
    
    if self.equipment_requirements:
        factors.append('comprehensive_equipment_planning')
    
    if self.technical_requirements:
        factors.append('technical_requirements_addressed')
    
    if self.external_collaborators:
        factors.append('external_collaboration')
    
    return factors

def _identify_performance_strengths(self, performance_dimensions):
    """Helper method to identify performance strengths"""
    strengths = []
    
    for dimension, data in performance_dimensions.items():
        if data['level'] in ['excellent', 'good', 'highly_engaged', 'well_engaged', 'extremely_satisfied', 'satisfied', 'excellent_execution', 'good_execution']:
            strengths.append({
                'dimension': dimension,
                'level': data['level'],
                'metric': data['metric']
            })
    
    return strengths

def _identify_performance_improvement_areas(self, performance_dimensions):
    """Helper method to identify performance improvement areas"""
    improvement_areas = []
    
    for dimension, data in performance_dimensions.items():
        if data['level'] in ['poor', 'very_poor', 'poorly_engaged', 'very_poorly_engaged', 'dissatisfied', 'extremely_dissatisfied', 'poor_execution', 'very_poor_execution']:
            improvement_areas.append({
                'dimension': dimension,
                'level': data['level'],
                'metric': data['metric'],
                'improvement_potential': self._calculate_improvement_potential(dimension, data['metric'])
            })
    
    return improvement_areas

def _calculate_improvement_potential(self, dimension, current_metric):
    """Helper method to calculate improvement potential"""
    if dimension == 'attendance_performance':
        return max(0, 95 - current_metric)  # Target 95% attendance
    elif dimension == 'engagement_performance':
        return max(0, 85 - current_metric)  # Target 85% engagement
    elif dimension in ['satisfaction_performance', 'execution_performance']:
        return max(0, 4.5 - current_metric)  # Target 4.5/5 rating
    else:
        return 0

def _analyze_performance_trends(self):
    """Helper method to analyze performance trends"""
    # Compare with club's previous events
    previous_events = Event_info.objects.filter(
        organizing_club=self.organizing_club,
        event_status='completed'
    ).exclude(id=self.id).order_by('-event_date')[:3]
    
    if not previous_events.exists():
        return {'status': 'no_historical_data'}
    
    # Calculate trend metrics
    trend_metrics = {
        'attendance_trend': self._calculate_metric_trend(
            [event.attendance_rate for event in previous_events], self.attendance_rate
        ),
        'engagement_trend': self._calculate_metric_trend(
            [event.engagement_score for event in previous_events], self.engagement_score
        ),
        'satisfaction_trend': self._calculate_metric_trend(
            [event.participant_feedback_score for event in previous_events], self.participant_feedback_score
        ),
        'rating_trend': self._calculate_metric_trend(
            [event.overall_event_rating for event in previous_events], self.overall_event_rating
        )
    }
    
    return {
        'historical_comparison': list(previous_events.values(
            'event_name', 'attendance_rate', 'engagement_score', 
            'participant_feedback_score', 'overall_event_rating'
        )),
        'trend_metrics': trend_metrics,
        'overall_trend': self._determine_overall_trend(trend_metrics)
    }

def _calculate_metric_trend(self, historical_values, current_value):
    """Helper method to calculate metric trend"""
    if not historical_values:
        return 'no_data'
    
    avg_historical = sum(historical_values) / len(historical_values)
    improvement = current_value - avg_historical
    
    if improvement > 10:
        return 'significant_improvement'
    elif improvement > 5:
        return 'moderate_improvement'
    elif improvement > -5:
        return 'stable'
    elif improvement > -10:
        return 'moderate_decline'
    else:
        return 'significant_decline'

def _determine_overall_trend(self, trend_metrics):
    """Helper method to determine overall trend"""
    positive_trends = sum(1 for trend in trend_metrics.values() if 'improvement' in trend)
    stable_trends = sum(1 for trend in trend_metrics.values() if trend == 'stable')
    total_trends = len([trend for trend in trend_metrics.values() if trend != 'no_data'])
    
    if total_trends == 0:
        return 'insufficient_data'
    
    positive_ratio = positive_trends / total_trends
    
    if positive_ratio >= 0.75:
        return 'strong_positive_trend'
    elif positive_ratio >= 0.5:
        return 'moderate_positive_trend'
    elif stable_trends / total_trends >= 0.5:
        return 'stable_trend'
    else:
        return 'negative_trend'

def _analyze_event_financials(self):
    """Helper method to analyze event financials"""
    # Financial performance metrics
    financial_metrics = {
        'budget_performance': self._analyze_budget_performance(),
        'revenue_analysis': self._analyze_revenue_generation(),
        'cost_efficiency': self._analyze_cost_efficiency(),
        'roi_analysis': self._analyze_roi_performance(),
        'financial_sustainability': self._assess_financial_sustainability()
    }
    
    # Overall financial health
    financial_health = self._assess_overall_financial_health(financial_metrics)
    
    return {
        'financial_metrics': financial_metrics,
        'financial_health': financial_health,
        'cost_breakdown': self._estimate_cost_breakdown(),
        'revenue_streams': self._analyze_revenue_streams(),
        'financial_recommendations': self._generate_financial_recommendations()
    }

def _analyze_budget_performance(self):
    """Helper method to analyze budget performance"""
    if self.estimated_budget == 0:
        return {'status': 'no_budget_data'}
    
    budget_variance = abs(self.actual_expenditure - self.estimated_budget)
    variance_percentage = (budget_variance / self.estimated_budget) * 100
    
    if variance_percentage <= 10:
        performance_level = 'excellent'
    elif variance_percentage <= 20:
        performance_level = 'good'
    elif variance_percentage <= 30:
        performance_level = 'moderate'
    else:
        performance_level = 'poor'
    
    return {
        'estimated_budget': self.estimated_budget,
        'actual_expenditure': self.actual_expenditure,
        'variance_amount': budget_variance,
        'variance_percentage': round(variance_percentage, 2),
        'performance_level': performance_level,
        'budget_adherence': self._calculate_budget_adherence()
    }

def _calculate_budget_adherence(self):
    """Helper method to calculate budget adherence"""
    if self.estimated_budget == 0:
        return 0
    
    if self.actual_expenditure <= self.estimated_budget:
        return 100  # Perfect adherence if under/on budget
    else:
        overspend_percentage = ((self.actual_expenditure - self.estimated_budget) / self.estimated_budget) * 100
        return max(0, 100 - overspend_percentage * 2)  # Penalty for overspending

def _analyze_revenue_generation(self):
    """Helper method to analyze revenue generation"""
    total_revenue = self.revenue_generated + self.sponsorship_amount
    
    # Revenue composition
    revenue_composition = {
        'registration_revenue': self.revenue_generated,
        'sponsorship_revenue': self.sponsorship_amount,
        'total_revenue': total_revenue
    }
    
    # Revenue efficiency
    if self.current_registrations > 0:
        revenue_per_participant = total_revenue / self.current_registrations
    else:
        revenue_per_participant = 0
    
    return {
        'revenue_composition': revenue_composition,
        'revenue_per_participant': round(revenue_per_participant, 2),
        'revenue_target_achievement': self._calculate_revenue_target_achievement(),
        'revenue_growth_potential': self._assess_revenue_growth_potential()
    }

def _calculate_revenue_target_achievement(self):
    """Helper method to calculate revenue target achievement"""
    # Estimate target revenue based on max participants and registration fee
    if self.max_participants > 0 and self.registration_fee > 0:
        target_revenue = self.max_participants * self.registration_fee
        total_revenue = self.revenue_generated + self.sponsorship_amount
        
        if target_revenue > 0:
            achievement_percentage = (total_revenue / target_revenue) * 100
            return round(achievement_percentage, 2)
    
    return 0

def _assess_revenue_growth_potential(self):
    """Helper method to assess revenue growth potential"""
    growth_factors = []
    
    if self.current_registrations < self.max_participants:
        growth_factors.append('increase_participation')
    
    if self.registration_fee == 0:
        growth_factors.append('introduce_nominal_fee')
    
    if not self.sponsorship_amount:
        growth_factors.append('seek_sponsorship_opportunities')
    
    if not self.external_collaborators:
        growth_factors.append('explore_external_partnerships')
    
    return {
        'growth_factors': growth_factors,
        'potential_revenue_increase': self._estimate_potential_revenue_increase()
    }

def _estimate_potential_revenue_increase(self):
    """Helper method to estimate potential revenue increase"""
    current_revenue = self.revenue_generated + self.sponsorship_amount
    
    # Estimate potential from full participation
    participation_potential = 0
    if self.max_participants > self.current_registrations:
        additional_participants = self.max_participants - self.current_registrations
        participation_potential = additional_participants * self.registration_fee
    
    # Estimate sponsorship potential
    sponsorship_potential = max(0, self.actual_expenditure * 0.3 - self.sponsorship_amount)
    
    total_potential = participation_potential + sponsorship_potential
    
    return {
        'participation_potential': round(participation_potential, 2),
        'sponsorship_potential': round(sponsorship_potential, 2),
        'total_potential': round(total_potential, 2),
        'percentage_increase': round((total_potential / max(1, current_revenue)) * 100, 2)
    }

def _analyze_cost_efficiency(self):
    """Helper method to analyze cost efficiency"""
    if self.actual_participants == 0:
        return {'status': 'no_participants_for_analysis'}
    
    cost_per_participant = self.actual_expenditure / self.actual_participants
    
    # Cost efficiency benchmarks
    efficiency_level = self._determine_cost_efficiency_level(cost_per_participant)
    
    # Cost optimization potential
    optimization_potential = self._identify_cost_optimization_opportunities()
    
    return {
        'cost_per_participant': round(cost_per_participant, 2),
        'efficiency_level': efficiency_level,
        'optimization_potential': optimization_potential,
        'cost_effectiveness_score': self._calculate_cost_effectiveness_score()
    }

def _determine_cost_efficiency_level(self, cost_per_participant):
    """Helper method to determine cost efficiency level"""
    # Event type-based efficiency benchmarks
    if self.event_type in ['workshop', 'seminar']:
        if cost_per_participant <= 200:
            return 'highly_efficient'
        elif cost_per_participant <= 400:
            return 'efficient'
        elif cost_per_participant <= 600:
            return 'moderate'
        else:
            return 'inefficient'
    elif self.event_type in ['cultural', 'fest']:
        if cost_per_participant <= 500:
            return 'highly_efficient'
        elif cost_per_participant <= 800:
            return 'efficient'
        elif cost_per_participant <= 1200:
            return 'moderate'
        else:
            return 'inefficient'
    else:  # Other events
        if cost_per_participant <= 300:
            return 'highly_efficient'
        elif cost_per_participant <= 500:
            return 'efficient'
        elif cost_per_participant <= 700:
            return 'moderate'
        else:
            return 'inefficient'

def _identify_cost_optimization_opportunities(self):
    """Helper method to identify cost optimization opportunities"""
    opportunities = []
    
    # Marketing budget optimization
    if self.marketing_budget > self.actual_expenditure * 0.2:
        opportunities.append('optimize_marketing_spend')
    
    # Venue optimization
    if self.venue_capacity > self.actual_participants * 2:
        opportunities.append('right_size_venue')
    
    # Equipment optimization
    if self.equipment_requirements:
        opportunities.append('optimize_equipment_rental')
    
    # Sponsorship opportunities
    if self.sponsorship_amount < self.actual_expenditure * 0.2:
        opportunities.append('increase_sponsorship')
    
    return opportunities

def _calculate_cost_effectiveness_score(self):
    """Helper method to calculate cost effectiveness score"""
    # Multi-factor cost effectiveness
    factors = []
    
    # Budget adherence factor
    budget_adherence = self._calculate_budget_adherence()
    factors.append(('budget_adherence', budget_adherence * 0.3))
    
    # Participant satisfaction vs cost factor
    if self.participant_feedback_score > 0:
        satisfaction_cost_ratio = self.participant_feedback_score / max(1, self.actual_expenditure / 1000)
        satisfaction_factor = min(100, satisfaction_cost_ratio * 20)
        factors.append(('satisfaction_cost_ratio', satisfaction_factor * 0.4))
    
    # Revenue offset factor
    total_revenue = self.revenue_generated + self.sponsorship_amount
    revenue_offset = (total_revenue / max(1, self.actual_expenditure)) * 100
    factors.append(('revenue_offset', min(100, revenue_offset) * 0.3))
    
    # Calculate weighted score
    total_score = sum(score for _, score in factors)
    return round(total_score, 2)

def _analyze_roi_performance(self):
    """Helper method to analyze ROI performance"""
    roi_analysis = {
        'roi_percentage': self.roi_percentage,
        'roi_level': self._categorize_roi_level(self.roi_percentage),
        'roi_components': self._break_down_roi_components(),
        'roi_improvement_potential': self._assess_roi_improvement_potential()
    }
    
    return roi_analysis

def _categorize_roi_level(self, roi_percentage):
    """Helper method to categorize ROI level"""
    if roi_percentage >= 50:
        return 'excellent'
    elif roi_percentage >= 20:
        return 'good'
    elif roi_percentage >= 0:
        return 'break_even'
    elif roi_percentage >= -20:
        return 'moderate_loss'
    else:
        return 'significant_loss'

def _break_down_roi_components(self):
    """Helper method to break down ROI components"""
    total_investment = self.actual_expenditure + self.marketing_budget
    total_return = self.revenue_generated + self.sponsorship_amount
    
    return {
        'total_investment': total_investment,
        'total_return': total_return,
        'investment_breakdown': {
            'event_expenditure': self.actual_expenditure,
            'marketing_investment': self.marketing_budget
        },
        'return_breakdown': {
            'direct_revenue': self.revenue_generated,
            'sponsorship_revenue': self.sponsorship_amount
        },
        'net_result': total_return - total_investment
    }

def _assess_roi_improvement_potential(self):
    """Helper method to assess ROI improvement potential"""
    improvement_strategies = []
    
    # Revenue enhancement
    if self.current_registrations < self.max_participants:
        improvement_strategies.append('increase_participation')
    
    if self.sponsorship_amount < self.actual_expenditure * 0.3:
        improvement_strategies.append('enhance_sponsorship')
    
    # Cost reduction
    if self.actual_expenditure > self.estimated_budget * 1.1:
        improvement_strategies.append('improve_cost_control')
    
    # Value addition
    if self.participant_feedback_score < 4.0:
        improvement_strategies.append('enhance_value_proposition')
    
    return {
        'improvement_strategies': improvement_strategies,
        'potential_roi_gain': self._calculate_potential_roi_gain()
    }

def _calculate_potential_roi_gain(self):
    """Helper method to calculate potential ROI gain"""
    current_investment = self.actual_expenditure + self.marketing_budget
    current_return = self.revenue_generated + self.sponsorship_amount
    
    # Estimate potential improvements
    potential_revenue_increase = (self.max_participants - self.current_registrations) * self.registration_fee
    potential_sponsorship_increase = max(0, current_investment * 0.3 - self.sponsorship_amount)
    potential_cost_reduction = max(0, self.actual_expenditure - self.estimated_budget)
    
    potential_return = current_return + potential_revenue_increase + potential_sponsorship_increase
    potential_investment = current_investment - potential_cost_reduction
    
    if potential_investment > 0:
        potential_roi = ((potential_return - potential_investment) / potential_investment) * 100
        roi_improvement = potential_roi - self.roi_percentage
        return round(roi_improvement, 2)
    
    return 0

def _assess_financial_sustainability(self):
    """Helper method to assess financial sustainability"""
    sustainability_factors = []
    
    # Revenue sustainability
    if self.revenue_generated > 0:
        sustainability_factors.append('revenue_generating')
    
    # Sponsorship sustainability
    if self.sponsorship_amount > 0:
        sustainability_factors.append('sponsorship_secured')
    
    # Cost management
    if self.actual_expenditure <= self.estimated_budget:
        sustainability_factors.append('budget_controlled')
    
    # Break-even achievement
    total_revenue = self.revenue_generated + self.sponsorship_amount
    if total_revenue >= self.actual_expenditure:
        sustainability_factors.append('break_even_achieved')
    
    # Sustainability score
    sustainability_score = len(sustainability_factors) * 25  # 25 points per factor
    
    return {
        'sustainability_factors': sustainability_factors,
        'sustainability_score': sustainability_score,
        'sustainability_level': self._determine_sustainability_level(sustainability_score),
        'sustainability_recommendations': self._generate_sustainability_recommendations(sustainability_factors)
    }

def _determine_sustainability_level(self, sustainability_score):
    """Helper method to determine sustainability level"""
    if sustainability_score >= 75:
        return 'highly_sustainable'
    elif sustainability_score >= 50:
        return 'moderately_sustainable'
    elif sustainability_score >= 25:
        return 'marginally_sustainable'
    else:
        return 'unsustainable'

def _generate_sustainability_recommendations(self, sustainability_factors):
    """Helper method to generate sustainability recommendations"""
    recommendations = []
    
    if 'revenue_generating' not in sustainability_factors:
        recommendations.append('develop_revenue_streams')
    
    if 'sponsorship_secured' not in sustainability_factors:
        recommendations.append('pursue_sponsorship_opportunities')
    
    if 'budget_controlled' not in sustainability_factors:
        recommendations.append('implement_budget_controls')
    
    if 'break_even_achieved' not in sustainability_factors:
        recommendations.append('achieve_financial_break_even')
    
    # General recommendations
    recommendations.extend([
        'diversify_funding_sources',
        'optimize_cost_structure',
        'build_sponsor_relationships'
    ])
    
    return recommendations
```

This comprehensive Event_info model provides complete event lifecycle management.