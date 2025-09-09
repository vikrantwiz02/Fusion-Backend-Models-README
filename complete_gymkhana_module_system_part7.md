# Complete Gymkhana Module System Documentation - Fusion IIIT (Part 7)

## Database Models Analysis with Business Logic (Continued)

### Club_budget Model (Continued)

```python
def _assess_budget_planning_effectiveness(self):
    """Helper method to assess budget planning effectiveness"""
    # Planning effectiveness based on budget accuracy and utilization
    planning_metrics = self._calculate_planning_accuracy()
    
    # Budget forecasting quality
    forecasting_quality = self._evaluate_forecasting_quality()
    
    # Resource allocation optimization
    allocation_optimization = self._assess_resource_allocation_optimization()
    
    # Planning process maturity
    process_maturity = self._evaluate_planning_process_maturity()
    
    return {
        'planning_accuracy': planning_metrics,
        'forecasting_quality': forecasting_quality,
        'allocation_optimization': allocation_optimization,
        'process_maturity': process_maturity,
        'planning_effectiveness_score': self._calculate_planning_effectiveness_score(
            planning_metrics, forecasting_quality, allocation_optimization
        )
    }

def _calculate_planning_accuracy(self):
    """Helper method to calculate budget planning accuracy"""
    if self.budget_amount == 0:
        return {'status': 'no_budget_allocated'}
    
    # Accuracy based on variance from budget
    variance = abs(self.expenditure - self.budget_amount)
    variance_percentage = (variance / self.budget_amount) * 100
    
    # Accuracy scoring
    if variance_percentage <= 5:
        accuracy_score = 95
        accuracy_level = 'highly_accurate'
    elif variance_percentage <= 10:
        accuracy_score = 85
        accuracy_level = 'accurate'
    elif variance_percentage <= 20:
        accuracy_score = 70
        accuracy_level = 'moderately_accurate'
    elif variance_percentage <= 30:
        accuracy_score = 55
        accuracy_level = 'somewhat_inaccurate'
    else:
        accuracy_score = 40
        accuracy_level = 'inaccurate'
    
    return {
        'variance_amount': variance,
        'variance_percentage': round(variance_percentage, 2),
        'accuracy_score': accuracy_score,
        'accuracy_level': accuracy_level,
        'planning_quality': self._assess_planning_quality(variance_percentage)
    }

def _assess_planning_quality(self, variance_percentage):
    """Helper method to assess overall planning quality"""
    if variance_percentage <= 10:
        return 'excellent_planning'
    elif variance_percentage <= 20:
        return 'good_planning'
    elif variance_percentage <= 30:
        return 'adequate_planning'
    else:
        return 'needs_improvement'

def _evaluate_forecasting_quality(self):
    """Helper method to evaluate budget forecasting quality"""
    # Compare with historical forecasting accuracy
    historical_accuracy = self._calculate_historical_forecasting_accuracy()
    
    # Current session forecasting assessment
    current_accuracy = self._assess_current_forecasting()
    
    # Forecasting improvement trends
    improvement_trends = self._analyze_forecasting_trends()
    
    return {
        'historical_accuracy': historical_accuracy,
        'current_accuracy': current_accuracy,
        'improvement_trends': improvement_trends,
        'forecasting_maturity': self._determine_forecasting_maturity(historical_accuracy, current_accuracy)
    }

def _calculate_historical_forecasting_accuracy(self):
    """Helper method to calculate historical forecasting accuracy"""
    historical_budgets = Club_budget.objects.filter(club=self.club).order_by('-session')[:3]
    
    if historical_budgets.count() < 2:
        return {'status': 'insufficient_historical_data'}
    
    accuracy_scores = []
    for budget in historical_budgets:
        if budget.budget_amount > 0:
            variance = abs(budget.expenditure - budget.budget_amount) / budget.budget_amount * 100
            accuracy = max(0, 100 - variance * 2)  # 2% penalty per 1% variance
            accuracy_scores.append(accuracy)
    
    avg_accuracy = sum(accuracy_scores) / len(accuracy_scores) if accuracy_scores else 0
    
    return {
        'average_accuracy': round(avg_accuracy, 1),
        'accuracy_trend': self._determine_accuracy_trend(accuracy_scores),
        'consistency': self._calculate_forecasting_consistency(accuracy_scores)
    }

def _determine_accuracy_trend(self, accuracy_scores):
    """Helper method to determine accuracy trend"""
    if len(accuracy_scores) < 2:
        return 'insufficient_data'
    
    recent_avg = sum(accuracy_scores[:2]) / 2 if len(accuracy_scores) >= 2 else accuracy_scores[0]
    older_avg = sum(accuracy_scores[-2:]) / 2 if len(accuracy_scores) >= 2 else accuracy_scores[-1]
    
    if recent_avg > older_avg + 10:
        return 'improving'
    elif recent_avg < older_avg - 10:
        return 'declining'
    else:
        return 'stable'

def _calculate_forecasting_consistency(self, accuracy_scores):
    """Helper method to calculate forecasting consistency"""
    if len(accuracy_scores) < 2:
        return 'insufficient_data'
    
    variance = sum((score - sum(accuracy_scores)/len(accuracy_scores))**2 for score in accuracy_scores) / len(accuracy_scores)
    std_dev = variance ** 0.5
    
    if std_dev < 10:
        return 'highly_consistent'
    elif std_dev < 20:
        return 'moderately_consistent'
    else:
        return 'inconsistent'

def _assess_current_forecasting(self):
    """Helper method to assess current session forecasting"""
    utilization_rate = (self.expenditure / self.budget_amount) * 100 if self.budget_amount > 0 else 0
    
    # Current forecasting quality based on utilization
    if 75 <= utilization_rate <= 95:
        forecasting_quality = 'excellent'
        quality_score = 95
    elif 60 <= utilization_rate < 75 or 95 < utilization_rate <= 100:
        forecasting_quality = 'good'
        quality_score = 80
    elif 50 <= utilization_rate < 60 or 100 < utilization_rate <= 110:
        forecasting_quality = 'moderate'
        quality_score = 65
    else:
        forecasting_quality = 'poor'
        quality_score = 45
    
    return {
        'quality_level': forecasting_quality,
        'quality_score': quality_score,
        'forecasting_factors': self._identify_forecasting_factors()
    }

def _identify_forecasting_factors(self):
    """Helper method to identify factors affecting forecasting quality"""
    factors = []
    
    utilization_rate = (self.expenditure / self.budget_amount) * 100 if self.budget_amount > 0 else 0
    
    if utilization_rate > 100:
        factors.extend(['underestimated_costs', 'inadequate_contingency_planning'])
    elif utilization_rate < 60:
        factors.extend(['overestimated_needs', 'conservative_planning'])
    else:
        factors.append('balanced_forecasting')
    
    # Budget size factors
    if self.budget_amount > 20000:
        factors.append('complex_budget_management')
    elif self.budget_amount < 2000:
        factors.append('limited_budget_constraints')
    
    return factors

def _analyze_forecasting_trends(self):
    """Helper method to analyze forecasting improvement trends"""
    historical_budgets = Club_budget.objects.filter(club=self.club).order_by('-session')[:4]
    
    if historical_budgets.count() < 3:
        return {'status': 'insufficient_trend_data'}
    
    # Calculate trend in forecasting accuracy
    accuracy_improvements = []
    for i in range(len(historical_budgets) - 1):
        current = historical_budgets[i]
        previous = historical_budgets[i + 1]
        
        current_accuracy = 100 - (abs(current.expenditure - current.budget_amount) / current.budget_amount * 100) if current.budget_amount > 0 else 0
        previous_accuracy = 100 - (abs(previous.expenditure - previous.budget_amount) / previous.budget_amount * 100) if previous.budget_amount > 0 else 0
        
        improvement = current_accuracy - previous_accuracy
        accuracy_improvements.append(improvement)
    
    avg_improvement = sum(accuracy_improvements) / len(accuracy_improvements) if accuracy_improvements else 0
    
    return {
        'average_improvement': round(avg_improvement, 2),
        'trend_direction': 'improving' if avg_improvement > 5 else 'declining' if avg_improvement < -5 else 'stable',
        'improvement_velocity': abs(avg_improvement)
    }

def _determine_forecasting_maturity(self, historical_accuracy, current_accuracy):
    """Helper method to determine forecasting maturity level"""
    if historical_accuracy.get('status') == 'insufficient_historical_data':
        return 'developing_maturity'
    
    avg_historical = historical_accuracy.get('average_accuracy', 0)
    current_score = current_accuracy.get('quality_score', 0)
    
    combined_score = (avg_historical + current_score) / 2
    
    if combined_score >= 85:
        return 'mature_forecasting'
    elif combined_score >= 70:
        return 'developing_maturity'
    else:
        return 'immature_forecasting'

def _assess_resource_allocation_optimization(self):
    """Helper method to assess resource allocation optimization"""
    # Allocation efficiency metrics
    allocation_efficiency = self._calculate_allocation_efficiency()
    
    # Resource distribution analysis
    resource_distribution = self._analyze_resource_distribution()
    
    # Allocation flexibility assessment
    flexibility_assessment = self._assess_allocation_flexibility()
    
    return {
        'allocation_efficiency': allocation_efficiency,
        'resource_distribution': resource_distribution,
        'flexibility_assessment': flexibility_assessment,
        'optimization_recommendations': self._generate_allocation_optimization_recommendations()
    }

def _calculate_allocation_efficiency(self):
    """Helper method to calculate allocation efficiency"""
    utilization_rate = (self.expenditure / self.budget_amount) * 100 if self.budget_amount > 0 else 0
    
    # Efficiency based on optimal allocation (80-90% utilization)
    if 80 <= utilization_rate <= 90:
        efficiency_score = 100
    elif 70 <= utilization_rate < 80:
        efficiency_score = 90 - (80 - utilization_rate) * 2
    elif 90 < utilization_rate <= 100:
        efficiency_score = 100 - (utilization_rate - 90) * 1
    elif utilization_rate > 100:
        efficiency_score = max(60, 90 - (utilization_rate - 100) * 2)
    else:
        efficiency_score = max(50, utilization_rate * 1.2)
    
    return {
        'efficiency_score': round(efficiency_score, 1),
        'efficiency_level': self._categorize_efficiency_level(efficiency_score),
        'optimization_potential': 100 - efficiency_score
    }

def _categorize_efficiency_level(self, efficiency_score):
    """Helper method to categorize efficiency level"""
    if efficiency_score >= 90:
        return 'highly_efficient'
    elif efficiency_score >= 75:
        return 'efficient'
    elif efficiency_score >= 60:
        return 'moderately_efficient'
    else:
        return 'inefficient'

def _analyze_resource_distribution(self):
    """Helper method to analyze resource distribution patterns"""
    # Simplified distribution analysis based on expenditure patterns
    total_expenditure = self.expenditure
    
    if total_expenditure == 0:
        return {'status': 'no_expenditure_for_analysis'}
    
    # Estimated distribution categories
    distribution_analysis = {
        'core_activities': self._estimate_core_activity_allocation(),
        'support_functions': self._estimate_support_allocation(),
        'development_initiatives': self._estimate_development_allocation(),
        'contingency_reserves': self._estimate_contingency_allocation()
    }
    
    return {
        'distribution_breakdown': distribution_analysis,
        'distribution_balance': self._assess_distribution_balance(distribution_analysis),
        'allocation_priorities': self._identify_allocation_priorities(distribution_analysis)
    }

def _estimate_core_activity_allocation(self):
    """Helper method to estimate core activity allocation"""
    # Assume 60-70% goes to core activities
    return round(self.expenditure * 0.65, 2)

def _estimate_support_allocation(self):
    """Helper method to estimate support function allocation"""
    # Assume 15-20% goes to support functions
    return round(self.expenditure * 0.18, 2)

def _estimate_development_allocation(self):
    """Helper method to estimate development initiative allocation"""
    # Assume 10-15% goes to development
    return round(self.expenditure * 0.12, 2)

def _estimate_contingency_allocation(self):
    """Helper method to estimate contingency allocation"""
    # Remaining amount as contingency
    used_allocation = self._estimate_core_activity_allocation() + self._estimate_support_allocation() + self._estimate_development_allocation()
    return round(self.expenditure - used_allocation, 2)

def _assess_distribution_balance(self, distribution_analysis):
    """Helper method to assess distribution balance"""
    total_expenditure = sum(distribution_analysis.values())
    
    if total_expenditure == 0:
        return 'no_expenditure'
    
    # Calculate distribution percentages
    percentages = {k: (v / total_expenditure) * 100 for k, v in distribution_analysis.items()}
    
    # Assess balance based on ideal distributions
    ideal_distributions = {
        'core_activities': 65,
        'support_functions': 18,
        'development_initiatives': 12,
        'contingency_reserves': 5
    }
    
    balance_score = 0
    for category, actual_pct in percentages.items():
        ideal_pct = ideal_distributions.get(category, 25)
        deviation = abs(actual_pct - ideal_pct)
        category_score = max(0, 100 - deviation * 3)
        balance_score += category_score * 0.25
    
    return {
        'balance_score': round(balance_score, 1),
        'balance_level': 'balanced' if balance_score >= 80 else 'moderately_balanced' if balance_score >= 60 else 'imbalanced',
        'distribution_percentages': {k: round(v, 1) for k, v in percentages.items()}
    }

def _identify_allocation_priorities(self, distribution_analysis):
    """Helper method to identify allocation priorities"""
    priorities = []
    
    total_expenditure = sum(distribution_analysis.values())
    if total_expenditure == 0:
        return ['establish_expenditure_tracking']
    
    # Identify priorities based on allocation patterns
    core_percentage = (distribution_analysis['core_activities'] / total_expenditure) * 100
    
    if core_percentage < 50:
        priorities.append('increase_core_activity_focus')
    elif core_percentage > 80:
        priorities.append('balance_activity_portfolio')
    
    development_percentage = (distribution_analysis['development_initiatives'] / total_expenditure) * 100
    if development_percentage < 5:
        priorities.append('invest_in_development_initiatives')
    
    priorities.extend(['optimize_resource_allocation', 'monitor_allocation_effectiveness'])
    
    return priorities

def _assess_allocation_flexibility(self):
    """Helper method to assess allocation flexibility"""
    # Flexibility based on budget utilization and planning
    utilization_rate = (self.expenditure / self.budget_amount) * 100 if self.budget_amount > 0 else 0
    
    # Calculate flexibility score
    if utilization_rate <= 80:
        flexibility_score = 90  # High flexibility with remaining budget
    elif utilization_rate <= 95:
        flexibility_score = 70  # Moderate flexibility
    elif utilization_rate <= 100:
        flexibility_score = 50  # Limited flexibility
    else:
        flexibility_score = 20  # Very limited flexibility
    
    return {
        'flexibility_score': flexibility_score,
        'flexibility_level': self._categorize_flexibility_level(flexibility_score),
        'available_budget': max(0, self.budget_amount - self.expenditure),
        'reallocation_potential': self._assess_reallocation_potential(flexibility_score)
    }

def _categorize_flexibility_level(self, flexibility_score):
    """Helper method to categorize flexibility level"""
    if flexibility_score >= 80:
        return 'high_flexibility'
    elif flexibility_score >= 60:
        return 'moderate_flexibility'
    elif flexibility_score >= 40:
        return 'limited_flexibility'
    else:
        return 'constrained_flexibility'

def _assess_reallocation_potential(self, flexibility_score):
    """Helper method to assess reallocation potential"""
    if flexibility_score >= 80:
        return 'significant_reallocation_possible'
    elif flexibility_score >= 60:
        return 'moderate_reallocation_possible'
    elif flexibility_score >= 40:
        return 'limited_reallocation_possible'
    else:
        return 'minimal_reallocation_possible'

def _generate_allocation_optimization_recommendations(self):
    """Helper method to generate allocation optimization recommendations"""
    recommendations = []
    
    utilization_rate = (self.expenditure / self.budget_amount) * 100 if self.budget_amount > 0 else 0
    
    if utilization_rate < 70:
        recommendations.extend([
            'increase_strategic_investments',
            'expand_program_offerings',
            'accelerate_planned_initiatives'
        ])
    elif utilization_rate > 95:
        recommendations.extend([
            'implement_strict_expenditure_controls',
            'review_and_prioritize_remaining_expenses',
            'establish_contingency_protocols'
        ])
    
    recommendations.extend([
        'implement_dynamic_budget_allocation',
        'establish_regular_allocation_reviews',
        'optimize_resource_distribution_across_categories'
    ])
    
    return recommendations

def _evaluate_planning_process_maturity(self):
    """Helper method to evaluate budget planning process maturity"""
    # Maturity assessment based on various factors
    maturity_factors = {
        'planning_accuracy': self._assess_planning_accuracy_maturity(),
        'forecasting_capability': self._assess_forecasting_maturity(),
        'allocation_optimization': self._assess_allocation_maturity(),
        'risk_management': self._assess_risk_management_maturity()
    }
    
    overall_maturity = sum(maturity_factors.values()) / len(maturity_factors)
    
    return {
        'maturity_factors': maturity_factors,
        'overall_maturity_score': round(overall_maturity, 1),
        'maturity_level': self._determine_process_maturity_level(overall_maturity),
        'maturity_development_areas': self._identify_maturity_development_areas(maturity_factors)
    }

def _assess_planning_accuracy_maturity(self):
    """Helper method to assess planning accuracy maturity"""
    variance_percentage = abs(self.expenditure - self.budget_amount) / self.budget_amount * 100 if self.budget_amount > 0 else 0
    
    if variance_percentage <= 10:
        return 85
    elif variance_percentage <= 20:
        return 70
    elif variance_percentage <= 30:
        return 55
    else:
        return 40

def _assess_forecasting_maturity(self):
    """Helper method to assess forecasting maturity"""
    # Simplified forecasting maturity based on budget size and utilization
    utilization_rate = (self.expenditure / self.budget_amount) * 100 if self.budget_amount > 0 else 0
    
    if 75 <= utilization_rate <= 95:
        return 85
    elif 60 <= utilization_rate < 75 or 95 < utilization_rate <= 100:
        return 70
    else:
        return 55

def _assess_allocation_maturity(self):
    """Helper method to assess allocation maturity"""
    allocation_efficiency = self._calculate_allocation_efficiency()
    return allocation_efficiency['efficiency_score'] * 0.8  # Convert to maturity score

def _assess_risk_management_maturity(self):
    """Helper method to assess risk management maturity"""
    risk_assessment = self._evaluate_financial_risk()
    risk_score = risk_assessment['overall_risk_score']
    
    # Convert risk to maturity (lower risk = higher maturity)
    maturity_score = max(40, 100 - risk_score)
    return maturity_score

def _determine_process_maturity_level(self, maturity_score):
    """Helper method to determine process maturity level"""
    if maturity_score >= 85:
        return 'optimized'
    elif maturity_score >= 70:
        return 'managed'
    elif maturity_score >= 55:
        return 'defined'
    elif maturity_score >= 40:
        return 'developing'
    else:
        return 'initial'

def _identify_maturity_development_areas(self, maturity_factors):
    """Helper method to identify maturity development areas"""
    development_areas = []
    
    for factor, score in maturity_factors.items():
        if score < 60:
            development_areas.append(f'improve_{factor}')
        elif score < 75:
            development_areas.append(f'enhance_{factor}')
    
    return development_areas if development_areas else ['maintain_current_maturity_levels']

def _calculate_planning_effectiveness_score(self, planning_metrics, forecasting_quality, allocation_optimization):
    """Helper method to calculate overall planning effectiveness score"""
    # Weight different components
    accuracy_score = planning_metrics.get('accuracy_score', 0) * 0.3
    forecasting_score = forecasting_quality.get('current_accuracy', {}).get('quality_score', 0) * 0.3
    allocation_score = allocation_optimization.get('allocation_efficiency', {}).get('efficiency_score', 0) * 0.4
    
    effectiveness_score = accuracy_score + forecasting_score + allocation_score
    
    return {
        'effectiveness_score': round(effectiveness_score, 1),
        'effectiveness_level': self._determine_effectiveness_level(effectiveness_score),
        'improvement_potential': 100 - effectiveness_score
    }

def _determine_effectiveness_level(self, effectiveness_score):
    """Helper method to determine effectiveness level"""
    if effectiveness_score >= 85:
        return 'highly_effective'
    elif effectiveness_score >= 70:
        return 'effective'
    elif effectiveness_score >= 55:
        return 'moderately_effective'
    else:
        return 'needs_improvement'

def _evaluate_financial_risk(self):
    """Helper method to evaluate comprehensive financial risk"""
    # Multiple risk categories
    risk_categories = {
        'over_expenditure_risk': self._assess_over_expenditure_risk(),
        'under_utilization_risk': self._assess_under_utilization_risk(),
        'planning_risk': self._assess_planning_risk(),
        'cash_flow_risk': self._assess_cash_flow_risk(),
        'compliance_risk': self._assess_compliance_risk()
    }
    
    # Overall risk calculation
    overall_risk = sum(risk_categories.values()) / len(risk_categories)
    
    return {
        'risk_categories': risk_categories,
        'overall_risk_score': round(overall_risk, 1),
        'risk_level': self._determine_overall_risk_level(overall_risk),
        'priority_risks': self._identify_priority_risks(risk_categories),
        'risk_mitigation_plan': self._create_risk_mitigation_plan(risk_categories)
    }

def _assess_over_expenditure_risk(self):
    """Helper method to assess over-expenditure risk"""
    utilization_rate = (self.expenditure / self.budget_amount) * 100 if self.budget_amount > 0 else 0
    
    if utilization_rate > 100:
        return min(100, 70 + (utilization_rate - 100) * 3)
    elif utilization_rate > 95:
        return 60 + (utilization_rate - 95) * 2
    elif utilization_rate > 90:
        return 40 + (utilization_rate - 90) * 4
    else:
        return max(10, 30 - (90 - utilization_rate) * 0.5)

def _assess_under_utilization_risk(self):
    """Helper method to assess under-utilization risk"""
    utilization_rate = (self.expenditure / self.budget_amount) * 100 if self.budget_amount > 0 else 0
    
    if utilization_rate < 30:
        return 80 - utilization_rate
    elif utilization_rate < 50:
        return 60 - utilization_rate
    elif utilization_rate < 70:
        return 40 - (utilization_rate - 50) * 0.5
    else:
        return max(5, 20 - (utilization_rate - 70))

def _assess_planning_risk(self):
    """Helper method to assess planning risk"""
    variance_percentage = abs(self.expenditure - self.budget_amount) / self.budget_amount * 100 if self.budget_amount > 0 else 0
    
    if variance_percentage > 30:
        return min(90, 50 + variance_percentage)
    elif variance_percentage > 20:
        return 40 + variance_percentage
    elif variance_percentage > 10:
        return 20 + variance_percentage * 1.5
    else:
        return max(5, 15 - variance_percentage)

def _assess_cash_flow_risk(self):
    """Helper method to assess cash flow risk"""
    remaining_budget = self.budget_amount - self.expenditure
    
    if remaining_budget < 0:
        return 85  # High risk if over budget
    elif remaining_budget < self.budget_amount * 0.1:
        return 60  # Moderate risk if less than 10% remaining
    elif remaining_budget > self.budget_amount * 0.5:
        return 40  # Some risk if more than 50% remaining
    else:
        return 20  # Low risk for balanced spending

def _assess_compliance_risk(self):
    """Helper method to assess compliance risk"""
    # Simplified compliance risk based on expenditure patterns
    utilization_rate = (self.expenditure / self.budget_amount) * 100 if self.budget_amount > 0 else 0
    
    if utilization_rate > 100:
        return 70  # High compliance risk if over budget
    elif utilization_rate < 25:
        return 50  # Moderate risk if severely under-utilized
    else:
        return 25  # Low compliance risk for normal operations

def _determine_overall_risk_level(self, overall_risk):
    """Helper method to determine overall risk level"""
    if overall_risk >= 70:
        return 'high_risk'
    elif overall_risk >= 50:
        return 'moderate_risk'
    elif overall_risk >= 30:
        return 'low_risk'
    else:
        return 'minimal_risk'

def _identify_priority_risks(self, risk_categories):
    """Helper method to identify priority risks"""
    # Sort risks by severity
    sorted_risks = sorted(risk_categories.items(), key=lambda x: x[1], reverse=True)
    
    priority_risks = []
    for risk_type, risk_score in sorted_risks:
        if risk_score >= 60:
            priority_risks.append({'risk_type': risk_type, 'severity': 'high', 'score': risk_score})
        elif risk_score >= 40:
            priority_risks.append({'risk_type': risk_type, 'severity': 'moderate', 'score': risk_score})
    
    return priority_risks[:3]  # Top 3 priority risks

def _create_risk_mitigation_plan(self, risk_categories):
    """Helper method to create risk mitigation plan"""
    mitigation_plan = []
    
    for risk_type, risk_score in risk_categories.items():
        if risk_score >= 50:
            if risk_type == 'over_expenditure_risk':
                mitigation_plan.extend([
                    'implement_spending_controls',
                    'require_approval_for_major_expenses',
                    'establish_emergency_fund'
                ])
            elif risk_type == 'under_utilization_risk':
                mitigation_plan.extend([
                    'accelerate_planned_activities',
                    'identify_value_adding_investments',
                    'reallocate_unused_funds'
                ])
            elif risk_type == 'planning_risk':
                mitigation_plan.extend([
                    'improve_budget_forecasting',
                    'implement_regular_budget_reviews',
                    'enhance_planning_processes'
                ])
    
    return list(set(mitigation_plan))  # Remove duplicates

def _perform_comparative_financial_analysis(self):
    """Helper method to perform comparative financial analysis"""
    # Compare with peer clubs
    peer_analysis = self._compare_with_peer_clubs()
    
    # Historical comparison
    historical_comparison = self._compare_with_historical_performance()
    
    # Benchmark analysis
    benchmark_analysis = self._perform_benchmark_analysis()
    
    return {
        'peer_comparison': peer_analysis,
        'historical_comparison': historical_comparison,
        'benchmark_analysis': benchmark_analysis,
        'competitive_position': self._determine_competitive_position(peer_analysis, benchmark_analysis)
    }

def _compare_with_peer_clubs(self):
    """Helper method to compare with peer clubs"""
    # Get clubs with similar budget ranges
    budget_range_min = self.budget_amount * 0.7
    budget_range_max = self.budget_amount * 1.3
    
    peer_budgets = Club_budget.objects.filter(
        budget_amount__gte=budget_range_min,
        budget_amount__lte=budget_range_max,
        session=self.session
    ).exclude(id=self.id)
    
    if not peer_budgets.exists():
        return {'status': 'no_peer_data_available'}
    
    # Calculate peer metrics
    peer_metrics = []
    for budget in peer_budgets:
        utilization = (budget.expenditure / budget.budget_amount) * 100 if budget.budget_amount > 0 else 0
        peer_metrics.append({
            'club_name': budget.club.club_name,
            'budget_amount': budget.budget_amount,
            'utilization_rate': round(utilization, 2)
        })
    
    # Current club metrics
    current_utilization = (self.expenditure / self.budget_amount) * 100 if self.budget_amount > 0 else 0
    
    # Comparison analysis
    peer_utilizations = [p['utilization_rate'] for p in peer_metrics]
    avg_peer_utilization = sum(peer_utilizations) / len(peer_utilizations) if peer_utilizations else 0
    
    return {
        'peer_count': len(peer_metrics),
        'peer_metrics': peer_metrics,
        'average_peer_utilization': round(avg_peer_utilization, 2),
        'current_utilization': round(current_utilization, 2),
        'relative_performance': self._assess_relative_performance(current_utilization, avg_peer_utilization),
        'peer_ranking': self._calculate_peer_ranking(current_utilization, peer_utilizations)
    }

def _assess_relative_performance(self, current_utilization, avg_peer_utilization):
    """Helper method to assess relative performance"""
    difference = current_utilization - avg_peer_utilization
    
    if difference > 10:
        return 'significantly_above_peers'
    elif difference > 5:
        return 'above_peers'
    elif difference > -5:
        return 'similar_to_peers'
    elif difference > -10:
        return 'below_peers'
    else:
        return 'significantly_below_peers'

def _calculate_peer_ranking(self, current_utilization, peer_utilizations):
    """Helper method to calculate ranking among peers"""
    if not peer_utilizations:
        return {'status': 'no_peers_for_ranking'}
    
    all_utilizations = peer_utilizations + [current_utilization]
    sorted_utilizations = sorted(all_utilizations, reverse=True)
    
    rank = sorted_utilizations.index(current_utilization) + 1
    total_clubs = len(all_utilizations)
    percentile = ((total_clubs - rank) / total_clubs) * 100
    
    return {
        'rank': rank,
        'total_clubs': total_clubs,
        'percentile': round(percentile, 1)
    }

def _compare_with_historical_performance(self):
    """Helper method to compare with historical performance"""
    historical_budgets = Club_budget.objects.filter(
        club=self.club
    ).exclude(id=self.id).order_by('-session')[:3]
    
    if not historical_budgets.exists():
        return {'status': 'no_historical_data'}
    
    # Historical performance analysis
    historical_metrics = []
    for budget in historical_budgets:
        utilization = (budget.expenditure / budget.budget_amount) * 100 if budget.budget_amount > 0 else 0
        historical_metrics.append({
            'session': budget.session,
            'budget_amount': budget.budget_amount,
            'utilization_rate': round(utilization, 2)
        })
    
    # Current vs historical comparison
    current_utilization = (self.expenditure / self.budget_amount) * 100 if self.budget_amount > 0 else 0
    historical_utilizations = [h['utilization_rate'] for h in historical_metrics]
    avg_historical_utilization = sum(historical_utilizations) / len(historical_utilizations) if historical_utilizations else 0
    
    return {
        'historical_metrics': historical_metrics,
        'average_historical_utilization': round(avg_historical_utilization, 2),
        'current_utilization': round(current_utilization, 2),
        'performance_trend': self._assess_performance_trend(current_utilization, historical_utilizations),
        'improvement_analysis': self._analyze_improvement_trends(historical_metrics)
    }

def _assess_performance_trend(self, current_utilization, historical_utilizations):
    """Helper method to assess performance trend"""
    if not historical_utilizations:
        return 'no_trend_data'
    
    avg_historical = sum(historical_utilizations) / len(historical_utilizations)
    improvement = current_utilization - avg_historical
    
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

def _analyze_improvement_trends(self, historical_metrics):
    """Helper method to analyze improvement trends"""
    if len(historical_metrics) < 2:
        return {'status': 'insufficient_data_for_trend_analysis'}
    
    # Calculate year-over-year improvements
    improvements = []
    for i in range(len(historical_metrics) - 1):
        current_metric = historical_metrics[i]
        previous_metric = historical_metrics[i + 1]
        
        improvement = current_metric['utilization_rate'] - previous_metric['utilization_rate']
        improvements.append(improvement)
    
    avg_improvement = sum(improvements) / len(improvements) if improvements else 0
    
    return {
        'year_over_year_improvements': improvements,
        'average_improvement_rate': round(avg_improvement, 2),
        'trend_consistency': self._assess_trend_consistency(improvements)
    }

def _assess_trend_consistency(self, improvements):
    """Helper method to assess trend consistency"""
    if not improvements:
        return 'no_data'
    
    positive_trends = sum(1 for imp in improvements if imp > 0)
    total_trends = len(improvements)
    consistency_ratio = positive_trends / total_trends if total_trends > 0 else 0
    
    if consistency_ratio >= 0.8:
        return 'consistently_improving'
    elif consistency_ratio >= 0.6:
        return 'mostly_improving'
    elif consistency_ratio >= 0.4:
        return 'mixed_trends'
    else:
        return 'declining_trends'

def _perform_benchmark_analysis(self):
    """Helper method to perform benchmark analysis"""
    # Industry/institutional benchmarks
    institutional_benchmarks = self._get_institutional_benchmarks()
    
    # Performance against benchmarks
    benchmark_comparison = self._compare_against_benchmarks(institutional_benchmarks)
    
    return {
        'institutional_benchmarks': institutional_benchmarks,
        'benchmark_comparison': benchmark_comparison,
        'benchmark_achievement': self._assess_benchmark_achievement(benchmark_comparison)
    }

def _get_institutional_benchmarks(self):
    """Helper method to get institutional benchmarks"""
    # Standard institutional benchmarks for club financial management
    return {
        'optimal_utilization_range': {'min': 75, 'max': 95},
        'planning_accuracy_target': 90,
        'budget_variance_limit': 10,
        'minimum_activity_investment': 60
    }

def _compare_against_benchmarks(self, benchmarks):
    """Helper method to compare against benchmarks"""
    current_utilization = (self.expenditure / self.budget_amount) * 100 if self.budget_amount > 0 else 0
    variance_percentage = abs(self.expenditure - self.budget_amount) / self.budget_amount * 100 if self.budget_amount > 0 else 0
    
    comparison_results = {}
    
    # Utilization benchmark
    optimal_range = benchmarks['optimal_utilization_range']
    if optimal_range['min'] <= current_utilization <= optimal_range['max']:
        comparison_results['utilization'] = 'meets_benchmark'
    elif current_utilization < optimal_range['min']:
        comparison_results['utilization'] = 'below_benchmark'
    else:
        comparison_results['utilization'] = 'above_benchmark'
    
    # Variance benchmark
    variance_limit = benchmarks['budget_variance_limit']
    comparison_results['variance'] = 'meets_benchmark' if variance_percentage <= variance_limit else 'exceeds_benchmark'
    
    return comparison_results

def _assess_benchmark_achievement(self, benchmark_comparison):
    """Helper method to assess benchmark achievement"""
    met_benchmarks = sum(1 for result in benchmark_comparison.values() if result == 'meets_benchmark')
    total_benchmarks = len(benchmark_comparison)
    
    achievement_ratio = met_benchmarks / total_benchmarks if total_benchmarks > 0 else 0
    
    if achievement_ratio >= 0.8:
        return 'excellent_benchmark_achievement'
    elif achievement_ratio >= 0.6:
        return 'good_benchmark_achievement'
    elif achievement_ratio >= 0.4:
        return 'moderate_benchmark_achievement'
    else:
        return 'poor_benchmark_achievement'

def _determine_competitive_position(self, peer_analysis, benchmark_analysis):
    """Helper method to determine competitive position"""
    position_factors = []
    
    # Peer comparison factor
    if peer_analysis.get('status') != 'no_peer_data_available':
        relative_performance = peer_analysis.get('relative_performance', '')
        if 'above_peers' in relative_performance:
            position_factors.append('strong_peer_performance')
        elif 'below_peers' in relative_performance:
            position_factors.append('weak_peer_performance')
        else:
            position_factors.append('average_peer_performance')
    
    # Benchmark factor
    benchmark_achievement = benchmark_analysis.get('benchmark_achievement', '')
    if 'excellent' in benchmark_achievement or 'good' in benchmark_achievement:
        position_factors.append('strong_benchmark_performance')
    else:
        position_factors.append('benchmark_improvement_needed')
    
    # Overall competitive position
    if 'strong_peer_performance' in position_factors and 'strong_benchmark_performance' in position_factors:
        return 'leading_position'
    elif 'strong_peer_performance' in position_factors or 'strong_benchmark_performance' in position_factors:
        return 'competitive_position'
    else:
        return 'improvement_needed_position'

def _generate_financial_recommendations(self):
    """Helper method to generate comprehensive financial recommendations"""
    recommendations = []
    
    # Utilization-based recommendations
    utilization_rate = (self.expenditure / self.budget_amount) * 100 if self.budget_amount > 0 else 0
    
    if utilization_rate > 100:
        recommendations.extend([
            'implement_immediate_spending_controls',
            'review_budget_planning_process',
            'establish_approval_hierarchy_for_expenses'
        ])
    elif utilization_rate < 50:
        recommendations.extend([
            'accelerate_planned_activities',
            'explore_additional_value_adding_investments',
            'consider_budget_reallocation'
        ])
    elif 75 <= utilization_rate <= 95:
        recommendations.extend([
            'maintain_excellent_budget_management',
            'document_best_practices',
            'mentor_other_clubs'
        ])
    
    # Performance-based recommendations
    performance_metrics = self._calculate_financial_performance()
    overall_performance = performance_metrics['overall_performance_score']
    
    if overall_performance < 70:
        recommendations.extend([
            'comprehensive_financial_management_review',
            'implement_performance_monitoring_systems',
            'seek_financial_management_training'
        ])
    
    # Risk-based recommendations
    risk_assessment = self._evaluate_financial_risk()
    if risk_assessment['risk_level'] in ['high_risk', 'moderate_risk']:
        recommendations.extend([
            'develop_risk_mitigation_strategies',
            'implement_regular_risk_monitoring',
            'establish_contingency_planning'
        ])
    
    # Strategic recommendations
    recommendations.extend([
        'implement_regular_financial_reviews',
        'establish_financial_performance_metrics',
        'create_multi_year_financial_planning'
    ])
    
    return recommendations

@staticmethod
def get_institutional_budget_analytics(session=None):
    """
    CORE LOGIC: Institution-wide budget analytics and financial insights
    
    HOW IT WORKS:
    1. Analyzes budget allocation and utilization across all clubs
    2. Provides institutional financial performance metrics
    3. Identifies trends and patterns in financial management
    4. Generates strategic insights for institutional planning
    
    BUSINESS PURPOSE:
    - Institutional financial oversight and planning
    - Resource allocation optimization across clubs
    - Financial performance monitoring and evaluation
    - Strategic financial planning and policy development
    """
    # Filter by session if provided
    queryset = Club_budget.objects.all()
    if session:
        queryset = queryset.filter(session=session)
    
    if not queryset.exists():
        return {'status': 'no_budget_data', 'message': 'No budget data available for analysis'}
    
    # Comprehensive institutional analytics
    institutional_overview = Club_budget._analyze_institutional_overview(queryset)
    budget_distribution = Club_budget._analyze_budget_distribution(queryset)
    utilization_patterns = Club_budget._analyze_utilization_patterns(queryset)
    financial_performance = Club_budget._assess_institutional_financial_performance(queryset)
    
    return {
        'analysis_scope': {
            'total_clubs': queryset.values('club').distinct().count(),
            'total_budget_allocated': queryset.aggregate(total=Sum('budget_amount'))['total'] or 0,
            'total_expenditure': queryset.aggregate(total=Sum('expenditure'))['total'] or 0,
            'sessions_covered': queryset.values('session').distinct().count(),
            'analysis_date': datetime.datetime.now().strftime('%Y-%m-%d')
        },
        'institutional_overview': institutional_overview,
        'budget_distribution': budget_distribution,
        'utilization_patterns': utilization_patterns,
        'financial_performance': financial_performance,
        'strategic_recommendations': Club_budget._generate_institutional_recommendations(
            institutional_overview, budget_distribution, utilization_patterns, financial_performance
        )
    }

@staticmethod
def _analyze_institutional_overview(queryset):
    """Helper method to analyze institutional overview"""
    total_budget = queryset.aggregate(total=Sum('budget_amount'))['total'] or 0
    total_expenditure = queryset.aggregate(total=Sum('expenditure'))['total'] or 0
    overall_utilization = (total_expenditure / total_budget) * 100 if total_budget > 0 else 0
    
    # Club-wise analysis
    club_analysis = queryset.values('club__club_name').annotate(
        budget=Sum('budget_amount'),
        expenditure=Sum('expenditure'),
        utilization=Case(
            When(budget__gt=0, then=F('expenditure') * 100.0 / F('budget')),
            default=Value(0),
            output_field=FloatField()
        )
    ).order_by('-budget')
    
    return {
        'financial_summary': {
            'total_institutional_budget': total_budget,
            'total_institutional_expenditure': total_expenditure,
            'overall_utilization_rate': round(overall_utilization, 2),
            'remaining_budget': total_budget - total_expenditure
        },
        'club_analysis': list(club_analysis),
        'performance_distribution': Club_budget._analyze_performance_distribution(queryset)
    }

@staticmethod
def _analyze_performance_distribution(queryset):
    """Helper method to analyze performance distribution"""
    # Categorize clubs by utilization rates
    utilization_categories = {
        'over_budget': 0,
        'high_utilization': 0,
        'optimal_utilization': 0,
        'moderate_utilization': 0,
        'low_utilization': 0
    }
    
    for budget in queryset:
        utilization = (budget.expenditure / budget.budget_amount) * 100 if budget.budget_amount > 0 else 0
        
        if utilization > 100:
            utilization_categories['over_budget'] += 1
        elif utilization >= 90:
            utilization_categories['high_utilization'] += 1
        elif utilization >= 75:
            utilization_categories['optimal_utilization'] += 1
        elif utilization >= 50:
            utilization_categories['moderate_utilization'] += 1
        else:
            utilization_categories['low_utilization'] += 1
    
    total_clubs = queryset.count()
    distribution_percentages = {
        category: round((count / total_clubs) * 100, 1) if total_clubs > 0 else 0
        for category, count in utilization_categories.items()
    }
    
    return {
        'utilization_categories': utilization_categories,
        'distribution_percentages': distribution_percentages,
        'institutional_health': Club_budget._assess_institutional_health(distribution_percentages)
    }

@staticmethod
def _assess_institutional_health(distribution_percentages):
    """Helper method to assess institutional financial health"""
    optimal_percentage = distribution_percentages.get('optimal_utilization', 0)
    over_budget_percentage = distribution_percentages.get('over_budget', 0)
    low_utilization_percentage = distribution_percentages.get('low_utilization', 0)
    
    # Health scoring
    health_score = 0
    
    # Optimal utilization bonus
    health_score += optimal_percentage * 2
    
    # High utilization partial bonus
    health_score += distribution_percentages.get('high_utilization', 0) * 1.5
    
    # Moderate utilization partial bonus
    health_score += distribution_percentages.get('moderate_utilization', 0) * 1
    
    # Penalties
    health_score -= over_budget_percentage * 2
    health_score -= low_utilization_percentage * 1
    
    if health_score >= 150:
        return 'excellent_institutional_health'
    elif health_score >= 120:
        return 'good_institutional_health'
    elif health_score >= 90:
        return 'moderate_institutional_health'
    else:
        return 'poor_institutional_health'
```

This completes the comprehensive Club_budget model with extensive financial analytics, planning effectiveness assessment, risk evaluation, and institutional-level analysis. 
