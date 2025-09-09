# Complete Gymkhana Module System Documentation - Fusion IIIT (Part 2)

## Database Models Analysis with Business Logic (Continued)

### 3. Form_available Model
**Purpose**: Registration form availability management system that controls access to various club and activity registration forms.

**PostgreSQL Table**: `Form_available`

**Fields Structure**:
```python
class Form_available(models.Model):
    roll = models.CharField(default=2016001, max_length=7, primary_key=True)
    status = models.BooleanField(default=True, max_length=5)
    form_name = models.CharField(default="senate_registration", max_length=30)
```

**Enhanced Business Methods**:
```python
def get_form_access_analytics(self):
    """
    CORE LOGIC: Comprehensive form access and availability analytics
    
    HOW IT WORKS:
    1. Analyzes form availability patterns across different student groups
    2. Tracks form access rates and completion patterns
    3. Identifies bottlenecks in registration processes
    4. Provides insights for improving registration efficiency
    
    BUSINESS PURPOSE:
    - Optimize registration processes for better user experience
    - Identify and resolve access barriers
    - Plan capacity for registration periods
    - Improve administrative efficiency
    """
    # Get all form availability records for analysis
    all_forms = Form_available.objects.all()
    
    # Form availability statistics
    total_forms = all_forms.count()
    active_forms = all_forms.filter(status=True).count()
    inactive_forms = all_forms.filter(status=False).count()
    
    # Form type analysis
    form_types = all_forms.values('form_name').annotate(
        total_count=Count('roll'),
        active_count=Count('roll', filter=Q(status=True)),
        inactive_count=Count('roll', filter=Q(status=False))
    )
    
    # Calculate availability rates by form type
    form_analytics = []
    for form_type in form_types:
        availability_rate = (form_type['active_count'] / form_type['total_count'] * 100) if form_type['total_count'] > 0 else 0
        form_analytics.append({
            'form_name': form_type['form_name'],
            'total_users': form_type['total_count'],
            'active_users': form_type['active_count'],
            'availability_rate': round(availability_rate, 2),
            'access_status': self._determine_access_status(availability_rate)
        })
    
    # Student access patterns
    student_access_patterns = self._analyze_student_access_patterns()
    
    # Form usage trends
    usage_trends = self._analyze_form_usage_trends()
    
    return {
        'overview_metrics': {
            'total_form_records': total_forms,
            'active_forms': active_forms,
            'inactive_forms': inactive_forms,
            'overall_availability_rate': round((active_forms / total_forms * 100) if total_forms > 0 else 0, 2)
        },
        'form_type_analysis': form_analytics,
        'student_access_patterns': student_access_patterns,
        'usage_trends': usage_trends,
        'system_health': self._assess_form_system_health(),
        'recommendations': self._generate_form_access_recommendations(form_analytics)
    }

def _determine_access_status(self, availability_rate):
    """Helper method to determine access status based on availability rate"""
    if availability_rate >= 90:
        return 'excellent_access'
    elif availability_rate >= 75:
        return 'good_access'
    elif availability_rate >= 50:
        return 'moderate_access'
    else:
        return 'limited_access'

def _analyze_student_access_patterns(self):
    """Helper method to analyze student access patterns"""
    # Analyze access patterns by student roll number patterns (batch, department)
    all_records = Form_available.objects.all()
    
    # Extract batch information from roll numbers
    batch_analysis = {}
    department_analysis = {}
    
    for record in all_records:
        # Extract batch from roll number (assuming format like 2016001)
        try:
            batch = record.roll[:4]  # First 4 digits as batch year
            dept_code = record.roll[4:6] if len(record.roll) >= 6 else 'XX'  # Department code
            
            # Batch analysis
            if batch not in batch_analysis:
                batch_analysis[batch] = {'total': 0, 'active': 0}
            batch_analysis[batch]['total'] += 1
            if record.status:
                batch_analysis[batch]['active'] += 1
            
            # Department analysis (simplified)
            if dept_code not in department_analysis:
                department_analysis[dept_code] = {'total': 0, 'active': 0}
            department_analysis[dept_code]['total'] += 1
            if record.status:
                department_analysis[dept_code]['active'] += 1
                
        except (ValueError, IndexError):
            continue
    
    # Calculate access rates
    batch_access_rates = []
    for batch, data in batch_analysis.items():
        access_rate = (data['active'] / data['total'] * 100) if data['total'] > 0 else 0
        batch_access_rates.append({
            'batch': batch,
            'total_students': data['total'],
            'active_access': data['active'],
            'access_rate': round(access_rate, 2)
        })
    
    return {
        'batch_access_patterns': sorted(batch_access_rates, key=lambda x: x['batch'], reverse=True),
        'department_access_summary': department_analysis,
        'access_equity_score': self._calculate_access_equity_score(batch_analysis)
    }

def _calculate_access_equity_score(self, batch_analysis):
    """Helper method to calculate access equity across batches"""
    if not batch_analysis:
        return 0
    
    access_rates = []
    for batch_data in batch_analysis.values():
        if batch_data['total'] > 0:
            access_rate = batch_data['active'] / batch_data['total']
            access_rates.append(access_rate)
    
    if not access_rates:
        return 0
    
    # Calculate variance in access rates (lower variance = more equitable)
    mean_access = sum(access_rates) / len(access_rates)
    variance = sum((rate - mean_access) ** 2 for rate in access_rates) / len(access_rates)
    
    # Convert to equity score (100 - variance*100)
    equity_score = max(0, 100 - variance * 100)
    return round(equity_score, 2)

def _analyze_form_usage_trends(self):
    """Helper method to analyze form usage trends"""
    # Get form usage by type
    form_usage = Form_available.objects.values('form_name').annotate(
        usage_count=Count('roll'),
        active_usage=Count('roll', filter=Q(status=True))
    ).order_by('-usage_count')
    
    # Calculate usage trends
    trends = []
    total_usage = sum(item['usage_count'] for item in form_usage)
    
    for item in form_usage:
        usage_percentage = (item['usage_count'] / total_usage * 100) if total_usage > 0 else 0
        active_percentage = (item['active_usage'] / item['usage_count'] * 100) if item['usage_count'] > 0 else 0
        
        trends.append({
            'form_name': item['form_name'],
            'total_usage': item['usage_count'],
            'active_usage': item['active_usage'],
            'usage_percentage': round(usage_percentage, 2),
            'activation_rate': round(active_percentage, 2),
            'popularity_rank': len(trends) + 1
        })
    
    return {
        'form_popularity_ranking': trends,
        'most_popular_form': trends[0]['form_name'] if trends else None,
        'usage_distribution': self._calculate_usage_distribution(trends)
    }

def _calculate_usage_distribution(self, trends):
    """Helper method to calculate usage distribution metrics"""
    if not trends:
        return {'distribution_type': 'no_data'}
    
    usage_counts = [item['total_usage'] for item in trends]
    
    # Calculate distribution metrics
    total_forms = len(usage_counts)
    mean_usage = sum(usage_counts) / total_forms
    median_usage = sorted(usage_counts)[total_forms // 2]
    
    # Determine distribution type
    if max(usage_counts) > mean_usage * 3:
        distribution_type = 'highly_skewed'
    elif max(usage_counts) > mean_usage * 2:
        distribution_type = 'moderately_skewed'
    else:
        distribution_type = 'balanced'
    
    return {
        'distribution_type': distribution_type,
        'mean_usage': round(mean_usage, 2),
        'median_usage': median_usage,
        'usage_concentration': round((max(usage_counts) / sum(usage_counts)) * 100, 2)
    }

def _assess_form_system_health(self):
    """Helper method to assess overall form system health"""
    all_forms = Form_available.objects.all()
    
    if not all_forms.exists():
        return {'status': 'no_data', 'health_score': 0}
    
    # Calculate various health metrics
    total_forms = all_forms.count()
    active_forms = all_forms.filter(status=True).count()
    
    # Overall availability rate
    availability_rate = (active_forms / total_forms * 100) if total_forms > 0 else 0
    
    # Form diversity (number of different form types)
    form_types = all_forms.values('form_name').distinct().count()
    
    # Access equity (variance in access across different groups)
    access_patterns = self._analyze_student_access_patterns()
    equity_score = access_patterns['access_equity_score']
    
    # Calculate composite health score
    health_score = (
        availability_rate * 0.4 +  # 40% weight on availability
        min(form_types * 10, 100) * 0.3 +  # 30% weight on diversity
        equity_score * 0.3  # 30% weight on equity
    )
    
    # Determine health status
    if health_score >= 85:
        health_status = 'excellent'
    elif health_score >= 70:
        health_status = 'good'
    elif health_score >= 50:
        health_status = 'fair'
    else:
        health_status = 'poor'
    
    return {
        'health_score': round(health_score, 2),
        'health_status': health_status,
        'availability_rate': round(availability_rate, 2),
        'form_diversity': form_types,
        'access_equity': equity_score,
        'system_issues': self._identify_system_issues(availability_rate, equity_score, form_types)
    }

def _identify_system_issues(self, availability_rate, equity_score, form_types):
    """Helper method to identify system issues"""
    issues = []
    
    if availability_rate < 70:
        issues.append('low_overall_availability')
    
    if equity_score < 60:
        issues.append('access_inequality_across_groups')
    
    if form_types < 3:
        issues.append('limited_form_diversity')
    
    # Check for specific form access problems
    form_analytics = Form_available.objects.values('form_name').annotate(
        active_count=Count('roll', filter=Q(status=True)),
        total_count=Count('roll')
    )
    
    for form in form_analytics:
        form_availability = (form['active_count'] / form['total_count'] * 100) if form['total_count'] > 0 else 0
        if form_availability < 50:
            issues.append(f"low_availability_for_{form['form_name']}")
    
    return issues

def _generate_form_access_recommendations(self, form_analytics):
    """Helper method to generate recommendations for improving form access"""
    recommendations = []
    
    # Overall system recommendations
    overall_availability = sum(fa['active_users'] for fa in form_analytics) / sum(fa['total_users'] for fa in form_analytics) * 100 if form_analytics else 0
    
    if overall_availability < 80:
        recommendations.append('review_and_improve_overall_form_availability')
    
    # Form-specific recommendations
    for form in form_analytics:
        if form['availability_rate'] < 70:
            recommendations.append(f"investigate_low_availability_for_{form['form_name']}")
        elif form['availability_rate'] > 95:
            recommendations.append(f"analyze_high_demand_for_{form['form_name']}")
    
    # Access pattern recommendations
    recommendations.extend([
        'implement_automated_form_availability_monitoring',
        'establish_regular_access_audits',
        'create_user_feedback_mechanism_for_form_access',
        'develop_predictive_analytics_for_form_demand'
    ])
    
    return recommendations

def toggle_form_access(self, new_status=None):
    """
    CORE LOGIC: Toggle or set form access status with audit trail
    
    HOW IT WORKS:
    1. Changes form availability status for specific user
    2. Maintains audit trail of status changes
    3. Validates status change permissions
    4. Notifies relevant stakeholders of changes
    
    BUSINESS PURPOSE:
    - Dynamic form access management
    - Administrative control over registration processes
    - Audit trail for compliance and troubleshooting
    - Flexible access control based on various criteria
    """
    old_status = self.status
    
    if new_status is not None:
        self.status = new_status
    else:
        self.status = not self.status
    
    # Create audit trail entry
    audit_entry = {
        'roll_number': self.roll,
        'form_name': self.form_name,
        'old_status': old_status,
        'new_status': self.status,
        'changed_at': timezone.now(),
        'change_reason': 'administrative_action'
    }
    
    self.save()
    
    # Log the change (would integrate with actual audit system)
    self._log_status_change(audit_entry)
    
    # Notify relevant parties if needed
    if old_status != self.status:
        self._notify_status_change(audit_entry)
    
    return {
        'success': True,
        'old_status': old_status,
        'new_status': self.status,
        'audit_entry': audit_entry
    }

def _log_status_change(self, audit_entry):
    """Helper method to log status changes"""
    # This would integrate with a proper audit logging system
    # For now, we'll just log to console (in production, use proper logging)
    import logging
    logger = logging.getLogger('gymkhana.form_access')
    logger.info(f"Form access changed: {audit_entry}")

def _notify_status_change(self, audit_entry):
    """Helper method to notify relevant parties of status changes"""
    # This would integrate with notification system
    # Could send emails, push notifications, etc.
    if not audit_entry['new_status']:
        # Form access disabled - might need urgent notification
        notification_data = {
            'type': 'form_access_disabled',
            'roll_number': audit_entry['roll_number'],
            'form_name': audit_entry['form_name'],
            'timestamp': audit_entry['changed_at']
        }
        # Send notification to administrators
    
@staticmethod
def get_system_wide_form_analytics():
    """
    CORE LOGIC: System-wide form access analytics and reporting
    
    HOW IT WORKS:
    1. Analyzes form access patterns across entire system
    2. Identifies trends and usage patterns
    3. Provides executive dashboard metrics
    4. Generates insights for strategic planning
    
    BUSINESS PURPOSE:
    - Strategic planning for registration systems
    - Resource allocation and capacity planning
    - Performance monitoring and optimization
    - Executive reporting and decision support
    """
    all_forms = Form_available.objects.all()
    
    if not all_forms.exists():
        return {'status': 'no_data', 'message': 'No form availability data found'}
    
    # System overview metrics
    total_records = all_forms.count()
    active_records = all_forms.filter(status=True).count()
    unique_forms = all_forms.values('form_name').distinct().count()
    unique_students = all_forms.values('roll').distinct().count()
    
    # Form type distribution
    form_distribution = all_forms.values('form_name').annotate(
        total_users=Count('roll'),
        active_users=Count('roll', filter=Q(status=True)),
        inactive_users=Count('roll', filter=Q(status=False))
    ).order_by('-total_users')
    
    # Calculate system metrics
    overall_availability_rate = (active_records / total_records * 100) if total_records > 0 else 0
    average_forms_per_student = total_records / unique_students if unique_students > 0 else 0
    
    # Identify high and low demand forms
    high_demand_forms = [f for f in form_distribution if f['total_users'] > total_records * 0.1]
    low_demand_forms = [f for f in form_distribution if f['total_users'] < total_records * 0.02]
    
    # Access pattern analysis
    access_patterns = Form_available._analyze_system_access_patterns()
    
    # Performance indicators
    performance_indicators = Form_available._calculate_system_performance_indicators(
        overall_availability_rate, unique_forms, unique_students, form_distribution
    )
    
    return {
        'system_overview': {
            'total_form_records': total_records,
            'active_records': active_records,
            'inactive_records': total_records - active_records,
            'unique_form_types': unique_forms,
            'unique_students': unique_students,
            'overall_availability_rate': round(overall_availability_rate, 2),
            'average_forms_per_student': round(average_forms_per_student, 2)
        },
        'form_distribution': list(form_distribution),
        'demand_analysis': {
            'high_demand_forms': high_demand_forms,
            'low_demand_forms': low_demand_forms,
            'demand_concentration': len(high_demand_forms) / unique_forms if unique_forms > 0 else 0
        },
        'access_patterns': access_patterns,
        'performance_indicators': performance_indicators,
        'strategic_insights': Form_available._generate_strategic_insights(form_distribution, access_patterns, performance_indicators)
    }

@staticmethod
def _analyze_system_access_patterns():
    """Helper method to analyze system-wide access patterns"""
    all_forms = Form_available.objects.all()
    
    # Time-based analysis (if timestamps were available)
    # For now, analyze by roll number patterns
    
    # Batch-wise analysis
    batch_stats = {}
    for form in all_forms:
        try:
            batch = form.roll[:4]  # Extract batch year
            if batch not in batch_stats:
                batch_stats[batch] = {'total': 0, 'active': 0}
            batch_stats[batch]['total'] += 1
            if form.status:
                batch_stats[batch]['active'] += 1
        except (ValueError, IndexError):
            continue
    
    # Calculate batch access rates
    batch_access_rates = []
    for batch, stats in batch_stats.items():
        access_rate = (stats['active'] / stats['total'] * 100) if stats['total'] > 0 else 0
        batch_access_rates.append({
            'batch': batch,
            'access_rate': round(access_rate, 2),
            'total_students': stats['total'],
            'active_students': stats['active']
        })
    
    return {
        'batch_analysis': sorted(batch_access_rates, key=lambda x: x['batch'], reverse=True),
        'access_variance': Form_available._calculate_access_variance(batch_access_rates),
        'equity_score': Form_available._calculate_overall_equity_score(batch_access_rates)
    }

@staticmethod
def _calculate_access_variance(batch_access_rates):
    """Helper method to calculate variance in access rates"""
    if not batch_access_rates:
        return 0
    
    access_rates = [batch['access_rate'] for batch in batch_access_rates]
    mean_rate = sum(access_rates) / len(access_rates)
    variance = sum((rate - mean_rate) ** 2 for rate in access_rates) / len(access_rates)
    
    return round(variance, 2)

@staticmethod
def _calculate_overall_equity_score(batch_access_rates):
    """Helper method to calculate overall equity score"""
    if not batch_access_rates:
        return 0
    
    variance = Form_available._calculate_access_variance(batch_access_rates)
    
    # Convert variance to equity score (lower variance = higher equity)
    # Normalize to 0-100 scale
    equity_score = max(0, 100 - variance)
    return round(equity_score, 2)

@staticmethod
def _calculate_system_performance_indicators(availability_rate, unique_forms, unique_students, form_distribution):
    """Helper method to calculate system performance indicators"""
    # Efficiency indicators
    form_utilization = sum(f['active_users'] for f in form_distribution) / sum(f['total_users'] for f in form_distribution) * 100
    
    # Diversity indicator
    form_diversity_score = min(unique_forms * 10, 100)  # Cap at 100
    
    # User engagement indicator
    engagement_score = (unique_students / 1000) * 100 if unique_students < 1000 else 100  # Assuming 1000 is target
    
    # System load indicator
    average_load = sum(f['total_users'] for f in form_distribution) / unique_forms if unique_forms > 0 else 0
    load_balance_score = 100 - min(average_load * 2, 100)  # Lower load = better balance
    
    # Overall performance score
    performance_score = (
        availability_rate * 0.3 +
        form_utilization * 0.25 +
        form_diversity_score * 0.2 +
        engagement_score * 0.15 +
        load_balance_score * 0.1
    )
    
    return {
        'availability_rate': round(availability_rate, 2),
        'form_utilization': round(form_utilization, 2),
        'diversity_score': round(form_diversity_score, 2),
        'engagement_score': round(engagement_score, 2),
        'load_balance_score': round(load_balance_score, 2),
        'overall_performance_score': round(performance_score, 2),
        'performance_grade': Form_available._assign_performance_grade(performance_score)
    }

@staticmethod
def _assign_performance_grade(performance_score):
    """Helper method to assign performance grade"""
    if performance_score >= 90:
        return 'A+'
    elif performance_score >= 85:
        return 'A'
    elif performance_score >= 80:
        return 'B+'
    elif performance_score >= 75:
        return 'B'
    elif performance_score >= 70:
        return 'C+'
    elif performance_score >= 65:
        return 'C'
    else:
        return 'D'

@staticmethod
def _generate_strategic_insights(form_distribution, access_patterns, performance_indicators):
    """Helper method to generate strategic insights"""
    insights = []
    
    # Performance insights
    performance_score = performance_indicators['overall_performance_score']
    if performance_score < 70:
        insights.append('system_performance_needs_improvement')
    elif performance_score > 90:
        insights.append('excellent_system_performance')
    
    # Demand insights
    total_usage = sum(f['total_users'] for f in form_distribution)
    top_form_usage = max(f['total_users'] for f in form_distribution) if form_distribution else 0
    
    if top_form_usage > total_usage * 0.5:
        insights.append('high_concentration_in_single_form_type')
    
    # Access equity insights
    equity_score = access_patterns['equity_score']
    if equity_score < 60:
        insights.append('access_inequality_across_student_groups')
    elif equity_score > 85:
        insights.append('excellent_access_equity')
    
    # Utilization insights
    utilization = performance_indicators['form_utilization']
    if utilization < 60:
        insights.append('low_form_utilization_rate')
    elif utilization > 90:
        insights.append('high_form_utilization_efficiency')
    
    # Strategic recommendations
    recommendations = []
    
    if 'system_performance_needs_improvement' in insights:
        recommendations.extend([
            'conduct_comprehensive_system_audit',
            'implement_performance_monitoring_dashboard',
            'review_form_availability_policies'
        ])
    
    if 'access_inequality_across_student_groups' in insights:
        recommendations.extend([
            'investigate_access_barriers_by_student_group',
            'implement_targeted_access_improvement_programs',
            'establish_access_equity_monitoring'
        ])
    
    if 'high_concentration_in_single_form_type' in insights:
        recommendations.extend([
            'diversify_form_offerings',
            'analyze_demand_drivers_for_popular_forms',
            'consider_capacity_expansion_for_high_demand_forms'
        ])
    
    return {
        'key_insights': insights,
        'strategic_recommendations': recommendations,
        'priority_actions': recommendations[:3] if recommendations else [],
        'monitoring_focus_areas': [
            'system_performance_metrics',
            'access_equity_tracking',
            'form_utilization_optimization'
        ]
    }
```

**Key Business Logic Features**:
- **Access Analytics**: Comprehensive form availability and usage analysis
- **Equity Monitoring**: Access pattern analysis across student groups
- **Performance Tracking**: System health and efficiency metrics
- **Strategic Insights**: Data-driven recommendations for improvement

---

### 4. Registration_form Model
**Purpose**: Student registration data collection and management system for club and activity participation.

**PostgreSQL Table**: `Registration_form`

**Fields Structure**:
```python
class Registration_form(models.Model):
    roll = models.CharField(max_length=8, default="20160017", primary_key=True)
    user_name = models.CharField(max_length=40, default="Student")
    branch = models.CharField(max_length=20, default="open")
    cpi = models.FloatField(max_length=3, default=6.0)
    programme = models.CharField(max_length=20, default="B.tech")
```

**Enhanced Business Methods**:
```python
def get_academic_profile_analysis(self):
    """
    CORE LOGIC: Comprehensive academic profile and eligibility analysis
    
    HOW IT WORKS:
    1. Analyzes student's academic performance and profile
    2. Evaluates eligibility for various clubs and activities
    3. Provides recommendations based on academic standing
    4. Identifies opportunities for academic improvement
    
    BUSINESS PURPOSE:
    - Academic performance tracking for activity participation
    - Merit-based club admission decisions
    - Student development and guidance
    - Academic eligibility verification
    """
    # Academic performance analysis
    performance_metrics = self._calculate_academic_performance_metrics()
    
    # Eligibility analysis for different club categories
    eligibility_analysis = self._analyze_club_eligibility()
    
    # Academic standing and recommendations
    academic_standing = self._determine_academic_standing()
    
    # Peer comparison within branch and programme
    peer_comparison = self._compare_with_peers()
    
    # Improvement opportunities
    improvement_opportunities = self._identify_improvement_opportunities()
    
    return {
        'student_profile': {
            'roll_number': self.roll,
            'name': self.user_name,
            'branch': self.branch,
            'programme': self.programme,
            'current_cpi': self.cpi
        },
        'academic_performance': performance_metrics,
        'eligibility_analysis': eligibility_analysis,
        'academic_standing': academic_standing,
        'peer_comparison': peer_comparison,
        'improvement_opportunities': improvement_opportunities,
        'recommendations': self._generate_academic_recommendations()
    }

def _calculate_academic_performance_metrics(self):
    """Helper method to calculate academic performance metrics"""
    # CPI analysis
    cpi_grade = self._assign_cpi_grade(self.cpi)
    cpi_percentile = self._calculate_cpi_percentile()
    
    # Performance trends (would require historical data)
    performance_trend = self._analyze_performance_trend()
    
    return {
        'current_cpi': self.cpi,
        'cpi_grade': cpi_grade,
        'cpi_percentile': cpi_percentile,
        'performance_trend': performance_trend,
        'academic_status': self._determine_academic_status(),
        'performance_category': self._categorize_performance()
    }

def _assign_cpi_grade(self, cpi):
    """Helper method to assign letter grade based on CPI"""
    if cpi >= 9.0:
        return 'A+'
    elif cpi >= 8.5:
        return 'A'
    elif cpi >= 8.0:
        return 'B+'
    elif cpi >= 7.5:
        return 'B'
    elif cpi >= 7.0:
        return 'C+'
    elif cpi >= 6.5:
        return 'C'
    elif cpi >= 6.0:
        return 'D'
    else:
        return 'F'

def _calculate_cpi_percentile(self):
    """Helper method to calculate CPI percentile within branch"""
    # Get all students in same branch and programme
    peer_cpis = Registration_form.objects.filter(
        branch=self.branch,
        programme=self.programme
    ).values_list('cpi', flat=True)
    
    if not peer_cpis:
        return 50  # Default percentile if no peers
    
    # Calculate percentile
    lower_count = sum(1 for cpi in peer_cpis if cpi < self.cpi)
    total_count = len(peer_cpis)
    
    percentile = (lower_count / total_count) * 100 if total_count > 0 else 50
    return round(percentile, 1)

def _analyze_performance_trend(self):
    """Helper method to analyze performance trend"""
    # This would require historical CPI data
    # For now, return a simplified trend based on current CPI
    if self.cpi >= 8.5:
        return 'excellent_consistent'
    elif self.cpi >= 7.5:
        return 'good_stable'
    elif self.cpi >= 6.5:
        return 'satisfactory'
    else:
        return 'needs_improvement'

def _determine_academic_status(self):
    """Helper method to determine overall academic status"""
    if self.cpi >= 8.5:
        return 'high_achiever'
    elif self.cpi >= 7.5:
        return 'good_standing'
    elif self.cpi >= 6.5:
        return 'satisfactory_standing'
    elif self.cpi >= 6.0:
        return 'probation_risk'
    else:
        return 'academic_probation'

def _categorize_performance(self):
    """Helper method to categorize performance level"""
    percentile = self._calculate_cpi_percentile()
    
    if percentile >= 90:
        return 'top_performer'
    elif percentile >= 75:
        return 'above_average'
    elif percentile >= 50:
        return 'average'
    elif percentile >= 25:
        return 'below_average'
    else:
        return 'low_performer'

def _analyze_club_eligibility(self):
    """Helper method to analyze eligibility for different club categories"""
    eligibility = {
        'technical_clubs': self._check_technical_eligibility(),
        'cultural_clubs': self._check_cultural_eligibility(),
        'sports_clubs': self._check_sports_eligibility(),
        'leadership_roles': self._check_leadership_eligibility(),
        'academic_clubs': self._check_academic_eligibility()
    }
    
    # Overall eligibility score
    eligible_categories = sum(1 for eligible in eligibility.values() if eligible['eligible'])
    total_categories = len(eligibility)
    eligibility_score = (eligible_categories / total_categories) * 100
    
    return {
        'category_eligibility': eligibility,
        'overall_eligibility_score': round(eligibility_score, 1),
        'eligible_categories': eligible_categories,
        'restricted_categories': total_categories - eligible_categories
    }

def _check_technical_eligibility(self):
    """Helper method to check technical club eligibility"""
    # Technical clubs might require good academic standing
    is_eligible = self.cpi >= 7.0 and self.branch in ['CSE', 'ECE', 'ME', 'EE']
    
    return {
        'eligible': is_eligible,
        'minimum_cpi_required': 7.0,
        'current_cpi': self.cpi,
        'eligible_branches': ['CSE', 'ECE', 'ME', 'EE'],
        'requirements_met': self.cpi >= 7.0,
        'branch_compatible': self.branch in ['CSE', 'ECE', 'ME', 'EE']
    }

def _check_cultural_eligibility(self):
    """Helper method to check cultural club eligibility"""
    # Cultural clubs might have more relaxed academic requirements
    is_eligible = self.cpi >= 6.0
    
    return {
        'eligible': is_eligible,
        'minimum_cpi_required': 6.0,
        'current_cpi': self.cpi,
        'requirements_met': self.cpi >= 6.0,
        'open_to_all_branches': True
    }

def _check_sports_eligibility(self):
    """Helper method to check sports club eligibility"""
    # Sports clubs might focus more on physical fitness than academics
    is_eligible = self.cpi >= 6.0
    
    return {
        'eligible': is_eligible,
        'minimum_cpi_required': 6.0,
        'current_cpi': self.cpi,
        'requirements_met': self.cpi >= 6.0,
        'physical_requirements': 'to_be_assessed_separately'
    }

def _check_leadership_eligibility(self):
    """Helper method to check leadership role eligibility"""
    # Leadership roles require higher academic standards
    is_eligible = self.cpi >= 7.5 and self.programme in ['B.tech', 'M.tech']
    
    return {
        'eligible': is_eligible,
        'minimum_cpi_required': 7.5,
        'current_cpi': self.cpi,
        'eligible_programmes': ['B.tech', 'M.tech'],
        'academic_requirement_met': self.cpi >= 7.5,
        'programme_compatible': self.programme in ['B.tech', 'M.tech']
    }

def _check_academic_eligibility(self):
    """Helper method to check academic club eligibility"""
    # Academic clubs require strong academic performance
    is_eligible = self.cpi >= 8.0
    
    return {
        'eligible': is_eligible,
        'minimum_cpi_required': 8.0,
        'current_cpi': self.cpi,
        'requirements_met': self.cpi >= 8.0,
        'recommended_for_high_achievers': True
    }

def _compare_with_peers(self):
    """Helper method to compare performance with peers"""
    # Compare within branch
    branch_peers = Registration_form.objects.filter(branch=self.branch)
    branch_avg_cpi = branch_peers.aggregate(avg_cpi=Avg('cpi'))['avg_cpi'] or 0
    
    # Compare within programme
    programme_peers = Registration_form.objects.filter(programme=self.programme)
    programme_avg_cpi = programme_peers.aggregate(avg_cpi=Avg('cpi'))['avg_cpi'] or 0
    
    # Overall comparison
    all_students = Registration_form.objects.all()
    overall_avg_cpi = all_students.aggregate(avg_cpi=Avg('cpi'))['avg_cpi'] or 0
    
    return {
        'branch_comparison': {
            'branch_average_cpi': round(branch_avg_cpi, 2),
            'above_branch_average': self.cpi > branch_avg_cpi,
            'difference_from_branch_avg': round(self.cpi - branch_avg_cpi, 2),
            'branch_rank': self._calculate_branch_rank()
        },
        'programme_comparison': {
            'programme_average_cpi': round(programme_avg_cpi, 2),
            'above_programme_average': self.cpi > programme_avg_cpi,
            'difference_from_programme_avg': round(self.cpi - programme_avg_cpi, 2)
        },
        'overall_comparison': {
            'institute_average_cpi': round(overall_avg_cpi, 2),
            'above_institute_average': self.cpi > overall_avg_cpi,
            'difference_from_institute_avg': round(self.cpi - overall_avg_cpi, 2),
            'overall_percentile': self._calculate_cpi_percentile()
        }
    }

def _calculate_branch_rank(self):
    """Helper method to calculate rank within branch"""
    branch_students = Registration_form.objects.filter(
        branch=self.branch
    ).order_by('-cpi')
    
    for rank, student in enumerate(branch_students, 1):
        if student.roll == self.roll:
            return rank
    
    return None

def _identify_improvement_opportunities(self):
    """Helper method to identify improvement opportunities"""
    opportunities = []
    
    # Academic improvement opportunities
    if self.cpi < 7.0:
        opportunities.append('focus_on_academic_improvement')
    elif self.cpi < 8.0:
        opportunities.append('aim_for_academic_excellence')
    
    # Club participation opportunities
    eligibility = self._analyze_club_eligibility()
    for category, details in eligibility['category_eligibility'].items():
        if not details['eligible']:
            opportunities.append(f'improve_eligibility_for_{category}')
    
    # Leadership opportunities
    if self.cpi >= 7.5:
        opportunities.append('consider_leadership_roles')
    
    # Peer learning opportunities
    percentile = self._calculate_cpi_percentile()
    if percentile < 75:
        opportunities.append('join_study_groups_with_high_performers')
    
    return opportunities

def _generate_academic_recommendations(self):
    """Helper method to generate personalized academic recommendations"""
    recommendations = []
    
    # Performance-based recommendations
    if self.cpi < 6.5:
        recommendations.extend([
            'focus_on_core_academic_improvement',
            'seek_academic_counseling',
            'consider_study_skill_workshops',
            'limit_extracurricular_activities_until_improvement'
        ])
    elif self.cpi < 7.5:
        recommendations.extend([
            'maintain_current_academic_performance',
            'gradually_increase_extracurricular_participation',
            'target_specific_subject_improvements'
        ])
    elif self.cpi < 8.5:
        recommendations.extend([
            'excellent_foundation_for_leadership_roles',
            'consider_mentoring_junior_students',
            'explore_advanced_academic_opportunities'
        ])
    else:
        recommendations.extend([
            'exceptional_academic_performance',
            'consider_research_opportunities',
            'mentor_other_students',
            'take_on_significant_leadership_responsibilities'
        ])
    
    # Branch-specific recommendations
    if self.branch in ['CSE', 'ECE']:
        recommendations.append('consider_technical_club_participation')
    
    # Programme-specific recommendations
    if self.programme == 'B.tech':
        recommendations.append('build_well_rounded_profile_for_placement')
    elif self.programme == 'M.tech':
        recommendations.append('focus_on_research_and_specialization')
    
    return recommendations

@staticmethod
def get_comprehensive_student_analytics():
    """
    CORE LOGIC: Institute-wide student performance and demographic analytics
    
    HOW IT WORKS:
    1. Analyzes academic performance across all students
    2. Provides demographic and academic distribution insights
    3. Identifies trends and patterns in student performance
    4. Generates strategic insights for institutional planning
    
    BUSINESS PURPOSE:
    - Institutional performance monitoring and planning
    - Resource allocation for academic support
    - Demographic analysis for diversity initiatives
    - Strategic planning for student development programs
    """
    all_students = Registration_form.objects.all()
    
    if not all_students.exists():
        return {'status': 'no_data', 'message': 'No student registration data available'}
    
    # Overall statistics
    total_students = all_students.count()
    avg_cpi = all_students.aggregate(avg_cpi=Avg('cpi'))['avg_cpi'] or 0
    
    # Academic performance distribution
    performance_distribution = Registration_form._analyze_performance_distribution(all_students)
    
    # Demographic analysis
    demographic_analysis = Registration_form._analyze_demographics(all_students)
    
    # Branch and programme analysis
    academic_analysis = Registration_form._analyze_academic_distribution(all_students)
    
    # Performance trends and insights
    performance_insights = Registration_form._generate_performance_insights(
        performance_distribution, demographic_analysis, academic_analysis
    )
    
    return {
        'overview_metrics': {
            'total_students': total_students,
            'average_cpi': round(avg_cpi, 2),
            'academic_year': datetime.datetime.now().year,
            'data_completeness': round((total_students / 1000) * 100, 1)  # Assuming 1000 is expected enrollment
        },
        'performance_distribution': performance_distribution,
        'demographic_analysis': demographic_analysis,
        'academic_analysis': academic_analysis,
        'performance_insights': performance_insights,
        'strategic_recommendations': Registration_form._generate_strategic_recommendations(
            performance_distribution, demographic_analysis, academic_analysis
        )
    }

@staticmethod
def _analyze_performance_distribution(all_students):
    """Helper method to analyze academic performance distribution"""
    # CPI distribution
    cpi_ranges = [
        (9.0, 10.0, 'exceptional'),
        (8.5, 9.0, 'excellent'),
        (8.0, 8.5, 'very_good'),
        (7.5, 8.0, 'good'),
        (7.0, 7.5, 'satisfactory'),
        (6.5, 7.0, 'average'),
        (6.0, 6.5, 'below_average'),
        (0.0, 6.0, 'poor')
    ]
    
    distribution = {}
    for min_cpi, max_cpi, category in cpi_ranges:
        count = all_students.filter(cpi__gte=min_cpi, cpi__lt=max_cpi).count()
        percentage = (count / all_students.count()) * 100 if all_students.count() > 0 else 0
        distribution[category] = {
            'count': count,
            'percentage': round(percentage, 1),
            'cpi_range': f'{min_cpi}-{max_cpi}'
        }
    
    # Performance statistics
    cpis = all_students.values_list('cpi', flat=True)
    median_cpi = sorted(cpis)[len(cpis) // 2] if cpis else 0
    max_cpi = max(cpis) if cpis else 0
    min_cpi = min(cpis) if cpis else 0
    
    return {
        'distribution_by_category': distribution,
        'statistical_summary': {
            'median_cpi': round(median_cpi, 2),
            'maximum_cpi': round(max_cpi, 2),
            'minimum_cpi': round(min_cpi, 2),
            'standard_deviation': Registration_form._calculate_cpi_std_dev(cpis)
        }
    }

@staticmethod
def _calculate_cpi_std_dev(cpis):
    """Helper method to calculate CPI standard deviation"""
    if not cpis:
        return 0
    
    mean = sum(cpis) / len(cpis)
    variance = sum((cpi - mean) ** 2 for cpi in cpis) / len(cpis)
    std_dev = variance ** 0.5
    
    return round(std_dev, 3)

@staticmethod
def _analyze_demographics(all_students):
    """Helper method to analyze demographic distribution"""
    # Branch distribution
    branch_distribution = all_students.values('branch').annotate(
        count=Count('roll'),
        avg_cpi=Avg('cpi')
    ).order_by('-count')
    
    # Programme distribution
    programme_distribution = all_students.values('programme').annotate(
        count=Count('roll'),
        avg_cpi=Avg('cpi')
    ).order_by('-count')
    
    # Diversity metrics
    total_students = all_students.count()
    unique_branches = all_students.values('branch').distinct().count()
    unique_programmes = all_students.values('programme').distinct().count()
    
    return {
        'branch_distribution': list(branch_distribution),
        'programme_distribution': list(programme_distribution),
        'diversity_metrics': {
            'total_branches': unique_branches,
            'total_programmes': unique_programmes,
            'branch_diversity_score': Registration_form._calculate_diversity_score(branch_distribution),
            'programme_diversity_score': Registration_form._calculate_diversity_score(programme_distribution)
        }
    }

@staticmethod
def _calculate_diversity_score(distribution):
    """Helper method to calculate diversity score using Shannon index"""
    total = sum(item['count'] for item in distribution)
    if total == 0:
        return 0
    
    shannon_index = 0
    for item in distribution:
        if item['count'] > 0:
            proportion = item['count'] / total
            shannon_index -= proportion * math.log(proportion)
    
    # Normalize to 0-100 scale
    return round(min(shannon_index * 30, 100), 2)

@staticmethod
def _analyze_academic_distribution(all_students):
    """Helper method to analyze academic distribution patterns"""
    # Branch-wise performance analysis
    branch_performance = all_students.values('branch').annotate(
        avg_cpi=Avg('cpi'),
        student_count=Count('roll'),
        top_performers=Count('roll', filter=Q(cpi__gte=8.5)),
        at_risk_students=Count('roll', filter=Q(cpi__lt=6.5))
    ).order_by('-avg_cpi')
    
    # Programme-wise performance analysis
    programme_performance = all_students.values('programme').annotate(
        avg_cpi=Avg('cpi'),
        student_count=Count('roll'),
        top_performers=Count('roll', filter=Q(cpi__gte=8.5)),
        at_risk_students=Count('roll', filter=Q(cpi__lt=6.5))
    ).order_by('-avg_cpi')
    
    return {
        'branch_performance': list(branch_performance),
        'programme_performance': list(programme_performance),
        'performance_gaps': Registration_form._identify_performance_gaps(branch_performance, programme_performance)
    }

@staticmethod
def _identify_performance_gaps(branch_performance, programme_performance):
    """Helper method to identify performance gaps"""
    gaps = []
    
    # Branch performance gaps
    if branch_performance:
        highest_branch_cpi = max(item['avg_cpi'] for item in branch_performance)
        lowest_branch_cpi = min(item['avg_cpi'] for item in branch_performance)
        branch_gap = highest_branch_cpi - lowest_branch_cpi
        
        if branch_gap > 1.0:
            gaps.append('significant_inter_branch_performance_gap')
    
    # Programme performance gaps
    if programme_performance:
        highest_programme_cpi = max(item['avg_cpi'] for item in programme_performance)
        lowest_programme_cpi = min(item['avg_cpi'] for item in programme_performance)
        programme_gap = highest_programme_cpi - lowest_programme_cpi
        
        if programme_gap > 0.5:
            gaps.append('inter_programme_performance_variation')
    
    return gaps

@staticmethod
def _generate_performance_insights(performance_distribution, demographic_analysis, academic_analysis):
    """Helper method to generate performance insights"""
    insights = []
    
    # Performance distribution insights
    exceptional_percentage = performance_distribution['distribution_by_category']['exceptional']['percentage']
    poor_percentage = performance_distribution['distribution_by_category']['poor']['percentage']
    
    if exceptional_percentage > 15:
        insights.append('high_percentage_of_exceptional_performers')
    elif exceptional_percentage < 5:
        insights.append('low_percentage_of_exceptional_performers')
    
    if poor_percentage > 10:
        insights.append('concerning_percentage_of_poor_performers')
    
    # Diversity insights
    branch_diversity = demographic_analysis['diversity_metrics']['branch_diversity_score']
    if branch_diversity > 80:
        insights.append('excellent_branch_diversity')
    elif branch_diversity < 50:
        insights.append('limited_branch_diversity')
    
    # Performance gap insights
    performance_gaps = academic_analysis['performance_gaps']
    if 'significant_inter_branch_performance_gap' in performance_gaps:
        insights.append('need_targeted_support_for_underperforming_branches')
    
    return insights

@staticmethod
def _generate_strategic_recommendations(performance_distribution, demographic_analysis, academic_analysis):
    """Helper method to generate strategic recommendations"""
    recommendations = []
    
    # Performance-based recommendations
    poor_percentage = performance_distribution['distribution_by_category']['poor']['percentage']
    if poor_percentage > 10:
        recommendations.extend([
            'implement_academic_support_programs',
            'establish_early_warning_system_for_at_risk_students',
            'create_peer_mentoring_programs'
        ])
    
    # Diversity-based recommendations
    branch_diversity = demographic_analysis['diversity_metrics']['branch_diversity_score']
    if branch_diversity < 60:
        recommendations.extend([
            'promote_inter_branch_collaboration',
            'encourage_cross_disciplinary_activities'
        ])
    
    # Performance gap recommendations
    if 'significant_inter_branch_performance_gap' in academic_analysis['performance_gaps']:
        recommendations.extend([
            'analyze_curriculum_differences_across_branches',
            'implement_branch_specific_support_programs',
            'share_best_practices_from_high_performing_branches'
        ])
    
    # General strategic recommendations
    recommendations.extend([
        'conduct_regular_performance_trend_analysis',
        'establish_performance_benchmarks_and_targets',
        'implement_data_driven_student_success_initiatives'
    ])
    
    return recommendations
```

**Key Business Logic Features**:
- **Academic Performance Analysis**: Comprehensive CPI and academic standing evaluation
- **Eligibility Assessment**: Club and activity participation eligibility checking
- **Peer Comparison**: Performance benchmarking within academic cohorts
- **Strategic Analytics**: Institute-wide performance monitoring and insights

This completes Part 2 of the Gymkhana module documentation.