# Complete Gymkhana Module System Documentation - Fusion IIIT (Part 6)

## Database Models Analysis with Business Logic (Continued)

### 7. Club_budget Model
**Purpose**: Financial management system that tracks budget allocation, expenditure, and financial health of clubs across different sessions.

**PostgreSQL Table**: `Club_budget`

**Fields Structure**:
```python
class Club_budget(models.Model):
    club = models.ForeignKey(Club_info, on_delete=models.CASCADE)
    budget_amount = models.FloatField(default=1000.0)
    expenditure = models.FloatField(default=0.0)
    session = models.CharField(max_length=15, default="2020-21")
```

**Enhanced Business Methods**:
```python
def get_comprehensive_financial_analytics(self):
    """
    CORE LOGIC: Complete financial management and budget analytics
    
    HOW IT WORKS:
    1. Analyzes budget utilization and expenditure patterns
    2. Tracks financial performance and efficiency metrics
    3. Evaluates fiscal responsibility and planning effectiveness
    4. Provides insights for budget optimization and planning
    
    BUSINESS PURPOSE:
    - Financial accountability and transparency
    - Budget optimization and resource allocation
    - Fiscal performance monitoring and evaluation
    - Strategic financial planning and forecasting
    """
    # Budget utilization and efficiency analysis
    utilization_analysis = self._analyze_budget_utilization()
    
    # Financial performance metrics
    performance_metrics = self._calculate_financial_performance()
    
    # Expenditure pattern analysis
    expenditure_analysis = self._analyze_expenditure_patterns()
    
    # Budget planning and forecasting
    planning_analysis = self._assess_budget_planning_effectiveness()
    
    # Risk assessment and financial health
    risk_assessment = self._evaluate_financial_risk()
    
    # Comparative analysis with other clubs
    comparative_analysis = self._perform_comparative_financial_analysis()
    
    return {
        'budget_overview': {
            'club_name': self.club.club_name,
            'session': self.session,
            'total_budget': self.budget_amount,
            'total_expenditure': self.expenditure,
            'remaining_budget': self.budget_amount - self.expenditure,
            'utilization_percentage': round((self.expenditure / self.budget_amount) * 100, 2) if self.budget_amount > 0 else 0
        },
        'utilization_analysis': utilization_analysis,
        'performance_metrics': performance_metrics,
        'expenditure_analysis': expenditure_analysis,
        'planning_analysis': planning_analysis,
        'risk_assessment': risk_assessment,
        'comparative_analysis': comparative_analysis,
        'financial_recommendations': self._generate_financial_recommendations()
    }

def _analyze_budget_utilization(self):
    """Helper method to analyze budget utilization patterns"""
    # Basic utilization metrics
    utilized_amount = self.expenditure
    total_budget = self.budget_amount
    remaining_budget = total_budget - utilized_amount
    utilization_rate = (utilized_amount / total_budget) * 100 if total_budget > 0 else 0
    
    # Utilization efficiency assessment
    efficiency_score = self._calculate_utilization_efficiency(utilization_rate)
    
    # Budget allocation effectiveness
    allocation_effectiveness = self._assess_allocation_effectiveness()
    
    # Seasonal utilization patterns (if historical data available)
    seasonal_patterns = self._analyze_seasonal_utilization_patterns()
    
    return {
        'utilization_metrics': {
            'utilized_amount': utilized_amount,
            'remaining_budget': remaining_budget,
            'utilization_rate': round(utilization_rate, 2),
            'utilization_category': self._categorize_utilization_rate(utilization_rate)
        },
        'efficiency_assessment': {
            'efficiency_score': efficiency_score,
            'efficiency_level': self._determine_efficiency_level(efficiency_score),
            'efficiency_factors': self._identify_efficiency_factors()
        },
        'allocation_effectiveness': allocation_effectiveness,
        'seasonal_patterns': seasonal_patterns
    }

def _calculate_utilization_efficiency(self, utilization_rate):
    """Helper method to calculate utilization efficiency score"""
    # Optimal utilization range is 75-95%
    if 75 <= utilization_rate <= 95:
        efficiency_score = 100
    elif 60 <= utilization_rate < 75:
        efficiency_score = 85 - (75 - utilization_rate) * 2
    elif 95 < utilization_rate <= 100:
        efficiency_score = 95
    elif utilization_rate > 100:
        efficiency_score = max(50, 95 - (utilization_rate - 100) * 5)  # Heavy penalty for over-expenditure
    else:
        efficiency_score = max(30, utilization_rate * 1.2)  # Low utilization penalty
    
    return round(efficiency_score, 1)

def _categorize_utilization_rate(self, utilization_rate):
    """Helper method to categorize utilization rate"""
    if utilization_rate > 100:
        return 'over_budget'
    elif utilization_rate >= 90:
        return 'high_utilization'
    elif utilization_rate >= 75:
        return 'optimal_utilization'
    elif utilization_rate >= 50:
        return 'moderate_utilization'
    elif utilization_rate >= 25:
        return 'low_utilization'
    else:
        return 'minimal_utilization'

def _determine_efficiency_level(self, efficiency_score):
    """Helper method to determine efficiency level"""
    if efficiency_score >= 90:
        return 'highly_efficient'
    elif efficiency_score >= 75:
        return 'efficient'
    elif efficiency_score >= 60:
        return 'moderately_efficient'
    else:
        return 'inefficient'

def _identify_efficiency_factors(self):
    """Helper method to identify factors affecting budget efficiency"""
    factors = []
    
    utilization_rate = (self.expenditure / self.budget_amount) * 100 if self.budget_amount > 0 else 0
    
    if utilization_rate > 100:
        factors.extend(['budget_planning_inadequacy', 'expenditure_control_issues'])
    elif utilization_rate < 50:
        factors.extend(['underutilization_of_resources', 'conservative_spending'])
    elif 75 <= utilization_rate <= 95:
        factors.append('optimal_budget_management')
    
    # Club-specific factors
    if self.budget_amount < 5000:
        factors.append('limited_budget_constraints')
    elif self.budget_amount > 20000:
        factors.append('substantial_budget_responsibility')
    
    return factors

def _assess_allocation_effectiveness(self):
    """Helper method to assess budget allocation effectiveness"""
    # Simplified allocation assessment based on utilization patterns
    utilization_rate = (self.expenditure / self.budget_amount) * 100 if self.budget_amount > 0 else 0
    
    # Allocation scoring
    if 80 <= utilization_rate <= 95:
        allocation_score = 95
        effectiveness_level = 'highly_effective'
    elif 65 <= utilization_rate < 80:
        allocation_score = 80
        effectiveness_level = 'effective'
    elif 95 < utilization_rate <= 100:
        allocation_score = 75
        effectiveness_level = 'adequate_but_tight'
    elif 50 <= utilization_rate < 65:
        allocation_score = 60
        effectiveness_level = 'conservative_allocation'
    else:
        allocation_score = 40
        effectiveness_level = 'suboptimal_allocation'
    
    return {
        'allocation_score': allocation_score,
        'effectiveness_level': effectiveness_level,
        'allocation_recommendations': self._generate_allocation_recommendations(effectiveness_level)
    }

def _generate_allocation_recommendations(self, effectiveness_level):
    """Helper method to generate allocation recommendations"""
    recommendations_map = {
        'highly_effective': ['maintain_current_allocation_strategy', 'share_best_practices'],
        'effective': ['minor_allocation_adjustments', 'monitor_utilization_trends'],
        'adequate_but_tight': ['review_budget_categories', 'improve_expenditure_planning'],
        'conservative_allocation': ['increase_program_investments', 'expand_activity_scope'],
        'suboptimal_allocation': ['comprehensive_budget_review', 'reallocate_resources_strategically']
    }
    
    return recommendations_map.get(effectiveness_level, ['review_allocation_strategy'])

def _analyze_seasonal_utilization_patterns(self):
    """Helper method to analyze seasonal utilization patterns"""
    # Get historical budget data for the club
    historical_budgets = Club_budget.objects.filter(club=self.club).order_by('session')
    
    if historical_budgets.count() < 2:
        return {'status': 'insufficient_historical_data'}
    
    # Analyze utilization trends
    utilization_trends = []
    for budget in historical_budgets:
        utilization_rate = (budget.expenditure / budget.budget_amount) * 100 if budget.budget_amount > 0 else 0
        utilization_trends.append({
            'session': budget.session,
            'utilization_rate': round(utilization_rate, 2),
            'budget_amount': budget.budget_amount,
            'expenditure': budget.expenditure
        })
    
    # Calculate trend metrics
    trend_analysis = self._calculate_utilization_trends(utilization_trends)
    
    return {
        'historical_utilization': utilization_trends,
        'trend_analysis': trend_analysis,
        'seasonal_insights': self._generate_seasonal_insights(utilization_trends)
    }

def _calculate_utilization_trends(self, utilization_trends):
    """Helper method to calculate utilization trend metrics"""
    if len(utilization_trends) < 2:
        return {'status': 'insufficient_data'}
    
    # Calculate trend direction
    recent_rates = [trend['utilization_rate'] for trend in utilization_trends[-3:]]
    
    if len(recent_rates) >= 2:
        if recent_rates[-1] > recent_rates[0]:
            trend_direction = 'increasing'
        elif recent_rates[-1] < recent_rates[0]:
            trend_direction = 'decreasing'
        else:
            trend_direction = 'stable'
    else:
        trend_direction = 'insufficient_data'
    
    # Calculate variance
    rates = [trend['utilization_rate'] for trend in utilization_trends]
    mean_rate = sum(rates) / len(rates)
    variance = sum((rate - mean_rate) ** 2 for rate in rates) / len(rates)
    consistency = 'high' if variance < 100 else 'moderate' if variance < 400 else 'low'
    
    return {
        'trend_direction': trend_direction,
        'average_utilization': round(mean_rate, 2),
        'utilization_variance': round(variance, 2),
        'consistency_level': consistency,
        'improvement_trajectory': self._assess_improvement_trajectory(utilization_trends)
    }

def _assess_improvement_trajectory(self, utilization_trends):
    """Helper method to assess improvement trajectory"""
    if len(utilization_trends) < 3:
        return 'insufficient_data'
    
    # Compare recent performance with earlier performance
    recent_avg = sum(trend['utilization_rate'] for trend in utilization_trends[-2:]) / 2
    earlier_avg = sum(trend['utilization_rate'] for trend in utilization_trends[:2]) / 2
    
    improvement = recent_avg - earlier_avg
    
    if improvement > 10:
        return 'significant_improvement'
    elif improvement > 5:
        return 'moderate_improvement'
    elif improvement > -5:
        return 'stable_performance'
    elif improvement > -10:
        return 'moderate_decline'
    else:
        return 'significant_decline'

def _generate_seasonal_insights(self, utilization_trends):
    """Helper method to generate seasonal insights"""
    insights = []
    
    if not utilization_trends:
        return ['no_historical_data_available']
    
    # Analyze latest trends
    if len(utilization_trends) >= 2:
        latest_utilization = utilization_trends[-1]['utilization_rate']
        previous_utilization = utilization_trends[-2]['utilization_rate']
        
        if latest_utilization > previous_utilization + 10:
            insights.append('increased_spending_efficiency')
        elif latest_utilization < previous_utilization - 10:
            insights.append('decreased_utilization_trend')
    
    # Budget growth analysis
    if len(utilization_trends) >= 2:
        latest_budget = utilization_trends[-1]['budget_amount']
        earlier_budget = utilization_trends[0]['budget_amount']
        
        growth_rate = ((latest_budget - earlier_budget) / earlier_budget) * 100 if earlier_budget > 0 else 0
        
        if growth_rate > 20:
            insights.append('significant_budget_growth')
        elif growth_rate < -10:
            insights.append('budget_reduction_trend')
    
    return insights

def _calculate_financial_performance(self):
    """Helper method to calculate comprehensive financial performance metrics"""
    # Performance indicators
    performance_indicators = {
        'budget_adherence': self._calculate_budget_adherence_score(),
        'expenditure_efficiency': self._calculate_expenditure_efficiency(),
        'financial_planning': self._assess_financial_planning_quality(),
        'resource_optimization': self._evaluate_resource_optimization()
    }
    
    # Overall performance score
    overall_performance = sum(performance_indicators.values()) / len(performance_indicators)
    
    # Performance benchmarking
    performance_benchmark = self._benchmark_performance()
    
    return {
        'performance_indicators': performance_indicators,
        'overall_performance_score': round(overall_performance, 1),
        'performance_grade': self._assign_performance_grade(overall_performance),
        'performance_benchmark': performance_benchmark,
        'performance_strengths': self._identify_performance_strengths(performance_indicators),
        'performance_improvement_areas': self._identify_performance_improvement_areas(performance_indicators)
    }

def _calculate_budget_adherence_score(self):
    """Helper method to calculate budget adherence score"""
    if self.budget_amount == 0:
        return 0
    
    variance = abs(self.expenditure - self.budget_amount)
    variance_percentage = (variance / self.budget_amount) * 100
    
    # Score calculation: higher penalty for over-budget
    if self.expenditure <= self.budget_amount:
        # Under or on budget
        adherence_score = max(70, 100 - variance_percentage * 2)
    else:
        # Over budget - heavier penalty
        over_budget_penalty = (self.expenditure - self.budget_amount) / self.budget_amount * 100
        adherence_score = max(30, 90 - over_budget_penalty * 3)
    
    return round(adherence_score, 1)

def _calculate_expenditure_efficiency(self):
    """Helper method to calculate expenditure efficiency"""
    utilization_rate = (self.expenditure / self.budget_amount) * 100 if self.budget_amount > 0 else 0
    
    # Efficiency curve: optimal around 85%
    if 80 <= utilization_rate <= 90:
        efficiency = 100
    elif 70 <= utilization_rate < 80:
        efficiency = 90 + (utilization_rate - 70) * 1
    elif 90 < utilization_rate <= 100:
        efficiency = 100 - (utilization_rate - 90) * 2
    elif utilization_rate > 100:
        efficiency = max(40, 80 - (utilization_rate - 100) * 3)
    else:
        efficiency = max(50, utilization_rate * 0.8)
    
    return round(efficiency, 1)

def _assess_financial_planning_quality(self):
    """Helper method to assess quality of financial planning"""
    # Planning quality based on budget utilization and variance
    utilization_rate = (self.expenditure / self.budget_amount) * 100 if self.budget_amount > 0 else 0
    
    # Planning quality indicators
    if 75 <= utilization_rate <= 95:
        planning_score = 95  # Excellent planning
    elif 60 <= utilization_rate < 75:
        planning_score = 80  # Good planning, conservative
    elif 95 < utilization_rate <= 100:
        planning_score = 85  # Good planning, close to limit
    elif 100 < utilization_rate <= 110:
        planning_score = 65  # Moderate planning, slight overrun
    elif utilization_rate > 110:
        planning_score = 40  # Poor planning, significant overrun
    else:
        planning_score = 60  # Conservative planning, underutilization
    
    return round(planning_score, 1)

def _evaluate_resource_optimization(self):
    """Helper method to evaluate resource optimization"""
    # Resource optimization based on budget size and utilization
    budget_category = self._categorize_budget_size()
    utilization_rate = (self.expenditure / self.budget_amount) * 100 if self.budget_amount > 0 else 0
    
    # Optimization scoring based on budget category
    optimization_scores = {
        'small_budget': {
            'high_utilization_bonus': 10 if utilization_rate > 80 else 0,
            'base_score': 70
        },
        'medium_budget': {
            'balanced_utilization_bonus': 15 if 75 <= utilization_rate <= 90 else 0,
            'base_score': 75
        },
        'large_budget': {
            'strategic_utilization_bonus': 20 if 80 <= utilization_rate <= 95 else 0,
            'base_score': 80
        }
    }
    
    category_scores = optimization_scores.get(budget_category, {'base_score': 60, 'balanced_utilization_bonus': 0})
    optimization_score = category_scores['base_score'] + sum(category_scores.values()) - category_scores['base_score']
    
    return round(min(optimization_score, 100), 1)

def _categorize_budget_size(self):
    """Helper method to categorize budget size"""
    if self.budget_amount < 5000:
        return 'small_budget'
    elif self.budget_amount < 15000:
        return 'medium_budget'
    else:
        return 'large_budget'

def _assign_performance_grade(self, performance_score):
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

def _benchmark_performance(self):
    """Helper method to benchmark performance against peers"""
    # Get similar budget range clubs for comparison
    budget_range_min = self.budget_amount * 0.8
    budget_range_max = self.budget_amount * 1.2
    
    peer_budgets = Club_budget.objects.filter(
        budget_amount__gte=budget_range_min,
        budget_amount__lte=budget_range_max,
        session=self.session
    ).exclude(id=self.id)
    
    if not peer_budgets.exists():
        return {'status': 'no_peer_data'}
    
    # Calculate peer metrics
    peer_utilization_rates = []
    for budget in peer_budgets:
        utilization = (budget.expenditure / budget.budget_amount) * 100 if budget.budget_amount > 0 else 0
        peer_utilization_rates.append(utilization)
    
    current_utilization = (self.expenditure / self.budget_amount) * 100 if self.budget_amount > 0 else 0
    
    # Benchmark analysis
    avg_peer_utilization = sum(peer_utilization_rates) / len(peer_utilization_rates)
    better_than_peers = sum(1 for rate in peer_utilization_rates if current_utilization > rate)
    percentile = (better_than_peers / len(peer_utilization_rates)) * 100
    
    return {
        'peer_count': len(peer_utilization_rates),
        'average_peer_utilization': round(avg_peer_utilization, 2),
        'current_utilization': round(current_utilization, 2),
        'performance_percentile': round(percentile, 1),
        'benchmark_status': self._determine_benchmark_status(percentile)
    }

def _determine_benchmark_status(self, percentile):
    """Helper method to determine benchmark status"""
    if percentile >= 80:
        return 'top_performer'
    elif percentile >= 60:
        return 'above_average'
    elif percentile >= 40:
        return 'average_performer'
    elif percentile >= 20:
        return 'below_average'
    else:
        return 'bottom_quartile'

def _identify_performance_strengths(self, performance_indicators):
    """Helper method to identify performance strengths"""
    strengths = []
    
    for indicator, score in performance_indicators.items():
        if score >= 85:
            strengths.append(f'excellent_{indicator}')
        elif score >= 75:
            strengths.append(f'strong_{indicator}')
    
    return strengths if strengths else ['developing_financial_capabilities']

def _identify_performance_improvement_areas(self, performance_indicators):
    """Helper method to identify performance improvement areas"""
    improvement_areas = []
    
    for indicator, score in performance_indicators.items():
        if score < 70:
            improvement_areas.append(f'improve_{indicator}')
        elif score < 80:
            improvement_areas.append(f'enhance_{indicator}')
    
    return improvement_areas if improvement_areas else ['maintain_current_performance_levels']

def _analyze_expenditure_patterns(self):
    """Helper method to analyze expenditure patterns and trends"""
    # Current expenditure analysis
    expenditure_breakdown = self._breakdown_expenditure_categories()
    
    # Spending velocity and timing
    spending_pattern = self._analyze_spending_pattern()
    
    # Cost efficiency analysis
    cost_efficiency = self._evaluate_cost_efficiency()
    
    # Expenditure risk assessment
    expenditure_risks = self._assess_expenditure_risks()
    
    return {
        'expenditure_breakdown': expenditure_breakdown,
        'spending_pattern': spending_pattern,
        'cost_efficiency': cost_efficiency,
        'expenditure_risks': expenditure_risks,
        'expenditure_insights': self._generate_expenditure_insights()
    }

def _breakdown_expenditure_categories(self):
    """Helper method to breakdown expenditure by categories"""
    # Simplified categorization based on expenditure amount
    total_expenditure = self.expenditure
    
    # Estimated category breakdown (in actual implementation, this would come from detailed records)
    if total_expenditure == 0:
        return {'status': 'no_expenditure_recorded'}
    
    # Simplified category estimation
    estimated_breakdown = {
        'events_and_activities': round(total_expenditure * 0.6, 2),
        'equipment_and_supplies': round(total_expenditure * 0.2, 2),
        'administrative_costs': round(total_expenditure * 0.1, 2),
        'miscellaneous': round(total_expenditure * 0.1, 2)
    }
    
    return {
        'category_breakdown': estimated_breakdown,
        'largest_category': max(estimated_breakdown.items(), key=lambda x: x[1]),
        'category_diversity': len([v for v in estimated_breakdown.values() if v > 0])
    }

def _analyze_spending_pattern(self):
    """Helper method to analyze spending patterns"""
    utilization_rate = (self.expenditure / self.budget_amount) * 100 if self.budget_amount > 0 else 0
    
    # Spending pattern classification
    if utilization_rate == 0:
        pattern_type = 'no_spending'
    elif utilization_rate < 25:
        pattern_type = 'conservative_spending'
    elif utilization_rate < 75:
        pattern_type = 'moderate_spending'
    elif utilization_rate < 95:
        pattern_type = 'active_spending'
    elif utilization_rate <= 100:
        pattern_type = 'full_utilization'
    else:
        pattern_type = 'over_spending'
    
    # Spending efficiency assessment
    efficiency_metrics = self._calculate_spending_efficiency(utilization_rate)
    
    return {
        'pattern_type': pattern_type,
        'utilization_rate': round(utilization_rate, 2),
        'efficiency_metrics': efficiency_metrics,
        'spending_recommendations': self._generate_spending_recommendations(pattern_type)
    }

def _calculate_spending_efficiency(self, utilization_rate):
    """Helper method to calculate spending efficiency metrics"""
    # Efficiency based on optimal utilization (80-90%)
    if 80 <= utilization_rate <= 90:
        efficiency_score = 100
        efficiency_level = 'optimal'
    elif 70 <= utilization_rate < 80:
        efficiency_score = 85
        efficiency_level = 'conservative_but_efficient'
    elif 90 < utilization_rate <= 100:
        efficiency_score = 90
        efficiency_level = 'aggressive_but_controlled'
    elif utilization_rate > 100:
        efficiency_score = max(40, 80 - (utilization_rate - 100) * 2)
        efficiency_level = 'over_budget_risk'
    else:
        efficiency_score = max(50, utilization_rate * 0.8)
        efficiency_level = 'underutilized'
    
    return {
        'efficiency_score': round(efficiency_score, 1),
        'efficiency_level': efficiency_level,
        'optimization_potential': 100 - efficiency_score
    }

def _generate_spending_recommendations(self, pattern_type):
    """Helper method to generate spending recommendations"""
    recommendations_map = {
        'no_spending': ['initiate_planned_activities', 'review_budget_allocation'],
        'conservative_spending': ['increase_program_investments', 'expand_activity_scope'],
        'moderate_spending': ['maintain_current_pace', 'monitor_remaining_budget'],
        'active_spending': ['monitor_expenditure_closely', 'ensure_budget_adherence'],
        'full_utilization': ['excellent_budget_utilization', 'plan_for_next_session'],
        'over_spending': ['immediate_expenditure_control', 'review_spending_authorization']
    }
    
    return recommendations_map.get(pattern_type, ['review_spending_strategy'])

def _evaluate_cost_efficiency(self):
    """Helper method to evaluate cost efficiency"""
    # Cost per activity/event estimation
    estimated_activities = max(1, int(self.expenditure / 1000)) if self.expenditure > 0 else 0
    cost_per_activity = self.expenditure / estimated_activities if estimated_activities > 0 else 0
    
    # Efficiency benchmarks
    if cost_per_activity == 0:
        efficiency_rating = 'no_activity_data'
    elif cost_per_activity < 500:
        efficiency_rating = 'highly_cost_efficient'
    elif cost_per_activity < 1500:
        efficiency_rating = 'cost_efficient'
    elif cost_per_activity < 3000:
        efficiency_rating = 'moderate_cost_efficiency'
    else:
        efficiency_rating = 'high_cost_per_activity'
    
    return {
        'estimated_activities': estimated_activities,
        'cost_per_activity': round(cost_per_activity, 2),
        'efficiency_rating': efficiency_rating,
        'cost_optimization_suggestions': self._suggest_cost_optimizations(efficiency_rating)
    }

def _suggest_cost_optimizations(self, efficiency_rating):
    """Helper method to suggest cost optimization strategies"""
    optimization_map = {
        'highly_cost_efficient': ['maintain_current_cost_structure', 'share_best_practices'],
        'cost_efficient': ['minor_optimizations_possible', 'monitor_cost_trends'],
        'moderate_cost_efficiency': ['review_vendor_negotiations', 'optimize_resource_allocation'],
        'high_cost_per_activity': ['comprehensive_cost_review', 'seek_alternative_suppliers'],
        'no_activity_data': ['establish_activity_tracking', 'implement_cost_monitoring']
    }
    
    return optimization_map.get(efficiency_rating, ['review_cost_efficiency_measures'])

def _assess_expenditure_risks(self):
    """Helper method to assess expenditure-related risks"""
    risks = []
    risk_scores = {}
    
    utilization_rate = (self.expenditure / self.budget_amount) * 100 if self.budget_amount > 0 else 0
    
    # Over-expenditure risk
    if utilization_rate > 95:
        risks.append('high_over_expenditure_risk')
        risk_scores['over_expenditure'] = min(100, (utilization_rate - 95) * 10)
    else:
        risk_scores['over_expenditure'] = 0
    
    # Under-utilization risk
    if utilization_rate < 50:
        risks.append('significant_under_utilization')
        risk_scores['under_utilization'] = (50 - utilization_rate) * 2
    else:
        risk_scores['under_utilization'] = 0
    
    # Budget planning risk
    historical_variance = self._calculate_historical_variance()
    if historical_variance > 20:
        risks.append('inconsistent_budget_planning')
        risk_scores['planning_inconsistency'] = min(historical_variance, 100)
    else:
        risk_scores['planning_inconsistency'] = historical_variance
    
    # Overall risk assessment
    overall_risk_score = sum(risk_scores.values()) / len(risk_scores) if risk_scores else 0
    
    return {
        'identified_risks': risks,
        'risk_scores': risk_scores,
        'overall_risk_score': round(overall_risk_score, 1),
        'risk_level': self._determine_risk_level(overall_risk_score),
        'risk_mitigation_strategies': self._suggest_risk_mitigation(risks)
    }

def _calculate_historical_variance(self):
    """Helper method to calculate historical budget variance"""
    historical_budgets = Club_budget.objects.filter(club=self.club).order_by('-session')[:3]
    
    if historical_budgets.count() < 2:
        return 0  # No variance calculation possible
    
    variances = []
    for budget in historical_budgets:
        if budget.budget_amount > 0:
            variance = abs(budget.expenditure - budget.budget_amount) / budget.budget_amount * 100
            variances.append(variance)
    
    return sum(variances) / len(variances) if variances else 0

def _determine_risk_level(self, risk_score):
    """Helper method to determine overall risk level"""
    if risk_score >= 70:
        return 'high_risk'
    elif risk_score >= 40:
        return 'medium_risk'
    elif risk_score >= 20:
        return 'low_risk'
    else:
        return 'minimal_risk'

def _suggest_risk_mitigation(self, risks):
    """Helper method to suggest risk mitigation strategies"""
    mitigation_strategies = []
    
    if 'high_over_expenditure_risk' in risks:
        mitigation_strategies.extend([
            'implement_spending_controls',
            'require_approval_for_large_expenditures',
            'establish_emergency_budget_reserves'
        ])
    
    if 'significant_under_utilization' in risks:
        mitigation_strategies.extend([
            'accelerate_planned_activities',
            'identify_additional_beneficial_expenditures',
            'plan_budget_reallocation'
        ])
    
    if 'inconsistent_budget_planning' in risks:
        mitigation_strategies.extend([
            'improve_budget_forecasting_methods',
            'implement_regular_budget_reviews',
            'establish_budget_planning_guidelines'
        ])
    
    return mitigation_strategies

def _generate_expenditure_insights(self):
    """Helper method to generate expenditure insights"""
    insights = []
    
    utilization_rate = (self.expenditure / self.budget_amount) * 100 if self.budget_amount > 0 else 0
    
    # Utilization insights
    if utilization_rate > 95:
        insights.append('approaching_budget_limit')
    elif utilization_rate < 30:
        insights.append('significant_budget_remaining')
    elif 80 <= utilization_rate <= 95:
        insights.append('optimal_budget_utilization')
    
    # Budget size insights
    budget_category = self._categorize_budget_size()
    if budget_category == 'large_budget':
        insights.append('substantial_financial_responsibility')
    elif budget_category == 'small_budget':
        insights.append('efficient_resource_management_required')
    
    return insights
```

This completes the Club_budget model with comprehensive financial analytics.