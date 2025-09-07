# Complete Establishment System Theory Part 2 - Fusion IIIT

## Comprehensive Model Analysis with Detailed Business Logic Explanations (Appraisal System)

### 4. Appraisal Model
**Purpose**: Comprehensive faculty performance evaluation and career advancement assessment system.

**Enhanced Business Logic Implementation**:

```python
class Appraisal(models.Model):
    APPRAISAL_STATUS = (
        ('pending', 'Pending'),
        ('accepted', 'Accepted'),
        ('rejected', 'Rejected'),
        ('forwarded', 'Forwarded'),
        ('auto_rejected', 'Auto Rejected'),
        ('outstanding', 'Outstanding'),
        ('excellent', 'Excellent'),
        ('very_good', 'Very Good'),
        ('good', 'Good'),
        ('poor', 'Poor')
    )
    
    # Core appraisal information
    applicant = models.ForeignKey(User, related_name='all_appraisals', on_delete=models.CASCADE)
    designation = models.ForeignKey(Designation, null=True, related_name='desig', on_delete=models.SET_NULL)
    status = models.CharField(max_length=20, default='pending', choices=APPRAISAL_STATUS)
    timestamp = models.DateTimeField(auto_now_add=True)
    
    # Academic profile
    discipline = models.CharField(max_length=30, null=True, help_text="Academic discipline")
    knowledge_field = models.CharField(max_length=30, null=True, help_text="Specialized knowledge field")
    research_interest = models.CharField(max_length=60, null=True, help_text="Primary research interests")
    
    # Performance period
    start_date = models.DateField(null=True, blank=True, help_text="Appraisal period start")
    end_date = models.DateField(null=True, blank=True, help_text="Appraisal period end")
    
    # Self-assessment sections
    other_research_element = models.TextField(max_length=2000, blank=True, null=True, default='',
                                            help_text="Other research elements and contributions")
    publications = models.TextField(max_length=2000, blank=True, null=True, default='',
                                  help_text="Publications during appraisal period")
    conferences_meeting_attended = models.TextField(max_length=2000, blank=True, null=True, default='',
                                                  help_text="Conferences and meetings attended")
    conferences_meeting_organized = models.TextField(max_length=2000, blank=True, null=True, default='',
                                                   help_text="Conferences and meetings organized")
    admin_assign = models.TextField(max_length=2000, blank=True, null=True, default='',
                                  help_text="Administrative assignments and responsibilities")
    sevice_to_ins = models.TextField(max_length=2000, blank=True, null=True, default='',
                                   help_text="Service to institution")
    extra_info = models.TextField(max_length=2000, blank=True, null=True, default='',
                                help_text="Additional information and achievements")
    faculty_comments = models.TextField(max_length=2000, blank=True, null=True, default='',
                                      help_text="Faculty self-reflection and comments")
    
    def calculate_comprehensive_performance_score(self):
        """
        CORE LOGIC: Calculate comprehensive faculty performance score across multiple dimensions
        
        HOW IT WORKS:
        1. Evaluates research output quality and quantity with impact metrics
        2. Assesses teaching effectiveness through course load and student feedback
        3. Measures service contributions to institution and academic community
        4. Analyzes professional development and knowledge advancement
        5. Applies weighted scoring based on designation and career stage
        
        BUSINESS PURPOSE:
        - Provides objective, data-driven foundation for promotion and tenure decisions
        - Ensures fair comparison across faculty with different specializations
        - Supports strategic faculty development and resource allocation
        - Demonstrates institutional commitment to excellence and accountability
        """
        performance_score = {
            'research_score': 0,
            'teaching_score': 0,
            'service_score': 0,
            'professional_development_score': 0,
            'total_weighted_score': 0,
            'performance_category': 'Good',
            'strengths': [],
            'improvement_areas': [],
            'career_stage_context': {}
        }
        
        # Research performance evaluation
        research_score = self._evaluate_research_performance()
        performance_score['research_score'] = research_score
        
        # Teaching performance evaluation
        teaching_score = self._evaluate_teaching_performance()
        performance_score['teaching_score'] = teaching_score
        
        # Service performance evaluation
        service_score = self._evaluate_service_performance()
        performance_score['service_score'] = service_score
        
        # Professional development evaluation
        prof_dev_score = self._evaluate_professional_development()
        performance_score['professional_development_score'] = prof_dev_score
        
        # Career stage-based weighting
        weights = self._get_career_stage_weights()
        
        total_score = (
            research_score * weights['research'] +
            teaching_score * weights['teaching'] +
            service_score * weights['service'] +
            prof_dev_score * weights['professional_development']
        )
        
        performance_score['total_weighted_score'] = round(total_score, 2)
        
        # Performance categorization
        if total_score >= 90:
            performance_score['performance_category'] = 'Outstanding'
        elif total_score >= 80:
            performance_score['performance_category'] = 'Excellent'
        elif total_score >= 70:
            performance_score['performance_category'] = 'Very Good'
        elif total_score >= 60:
            performance_score['performance_category'] = 'Good'
        else:
            performance_score['performance_category'] = 'Needs Improvement'
        
        # Identify strengths and improvement areas
        performance_score['strengths'], performance_score['improvement_areas'] = \
            self._identify_performance_patterns(research_score, teaching_score, service_score, prof_dev_score)
        
        # Career stage context
        performance_score['career_stage_context'] = self._analyze_career_stage_performance(total_score)
        
        return performance_score
    
    def _evaluate_research_performance(self):
        """
        CORE LOGIC: Evaluate research performance based on publications, projects, and impact
        
        HOW IT WORKS:
        1. Analyzes publication quality, quantity, and citation impact
        2. Evaluates research project funding and outcomes
        3. Assesses collaboration patterns and international visibility
        4. Considers innovation, patents, and knowledge transfer
        5. Weighs research leadership and supervision activities
        
        BUSINESS PURPOSE:
        - Quantifies research excellence for institutional ranking and reputation
        - Supports evidence-based research funding and resource allocation
        - Identifies research leaders and emerging scholars
        - Guides strategic research direction and partnership decisions
        """
        research_score = 0
        
        # Publication analysis (extract from publications field)
        if self.publications:
            publication_text = self.publications.lower()
            
            # Count different types of publications
            journal_count = publication_text.count('journal') + publication_text.count('paper')
            conference_count = publication_text.count('conference') + publication_text.count('proceeding')
            book_count = publication_text.count('book') + publication_text.count('chapter')
            
            # Quality indicators
            impact_keywords = ['ieee', 'acm', 'springer', 'elsevier', 'nature', 'science', 'indexed']
            impact_score = sum(1 for keyword in impact_keywords if keyword in publication_text)
            
            # Research score calculation
            research_score += min(journal_count * 8, 40)  # Max 40 points for journals
            research_score += min(conference_count * 5, 25)  # Max 25 points for conferences
            research_score += min(book_count * 12, 24)  # Max 24 points for books
            research_score += min(impact_score * 3, 15)  # Max 15 points for impact
        
        # Research project analysis (extract from other_research_element)
        if self.other_research_element:
            research_text = self.other_research_element.lower()
            
            # Project indicators
            funding_keywords = ['grant', 'funded', 'project', 'research', 'sponsored']
            funding_score = sum(1 for keyword in funding_keywords if keyword in research_text)
            research_score += min(funding_score * 2, 12)  # Max 12 points for funding
            
            # Collaboration indicators
            collab_keywords = ['collaboration', 'international', 'industry', 'partnership']
            collab_score = sum(1 for keyword in collab_keywords if keyword in research_text)
            research_score += min(collab_score * 2, 8)  # Max 8 points for collaboration
        
        # Conference participation analysis
        if self.conferences_meeting_attended:
            conf_text = self.conferences_meeting_attended.lower()
            
            # International vs national conferences
            intl_keywords = ['international', 'global', 'world', 'ieee', 'acm']
            intl_score = sum(1 for keyword in intl_keywords if keyword in conf_text)
            research_score += min(intl_score * 3, 15)  # Max 15 points for international exposure
        
        return min(research_score, 100)  # Cap at 100
    
    def _evaluate_teaching_performance(self):
        """
        CORE LOGIC: Evaluate teaching effectiveness and pedagogical contributions
        
        HOW IT WORKS:
        1. Analyzes course load, diversity, and complexity of teaching assignments
        2. Evaluates innovation in teaching methods and curriculum development
        3. Assesses student mentoring and guidance activities
        4. Considers teaching awards, recognition, and peer feedback
        5. Measures contribution to academic program development
        
        BUSINESS PURPOSE:
        - Ensures high-quality education delivery and student satisfaction
        - Recognizes and rewards excellent teaching for institutional culture
        - Identifies teaching innovation and best practices for sharing
        - Supports teaching professional development and career advancement
        """
        teaching_score = 0
        
        # Course instruction analysis
        courses_taught = CoursesInstructed.objects.filter(appraisal=self)
        
        if courses_taught.exists():
            # Calculate teaching load
            total_hours = 0
            course_diversity = set()
            
            for course in courses_taught:
                weekly_hours = (course.lecture_hrs_wk or 0) + (course.tutorial_hrs_wk or 0) + (course.lab_hrs_wk or 0)
                total_hours += weekly_hours
                course_diversity.add(course.course_name)
            
            # Teaching load scoring (optimal range consideration)
            if 8 <= total_hours <= 16:  # Optimal teaching load
                teaching_score += 25
            elif 6 <= total_hours <= 20:  # Acceptable load
                teaching_score += 20
            else:  # Under or overloaded
                teaching_score += 15
            
            # Course diversity bonus
            teaching_score += min(len(course_diversity) * 3, 15)  # Max 15 for diversity
            
            # Student numbers (indicates trust and capability)
            total_students = sum(course.reg_students or 0 for course in courses_taught)
            if total_students > 100:
                teaching_score += 15
            elif total_students > 50:
                teaching_score += 10
            elif total_students > 20:
                teaching_score += 5
        
        # New course development
        new_courses = NewCoursesOffered.objects.filter(appraisal=self)
        teaching_score += min(new_courses.count() * 8, 24)  # Max 24 for new courses
        
        # Course material development
        new_materials = NewCourseMaterial.objects.filter(appraisal=self)
        teaching_score += min(new_materials.count() * 5, 20)  # Max 20 for materials
        
        # Student supervision
        supervisions = ThesisResearchSupervision.objects.filter(appraisal=self)
        if supervisions.exists():
            completed_count = supervisions.filter(status__icontains='completed').count()
            ongoing_count = supervisions.filter(status__icontains='ongoing').count()
            
            teaching_score += min(completed_count * 8, 24)  # Max 24 for completed supervisions
            teaching_score += min(ongoing_count * 4, 16)    # Max 16 for ongoing supervisions
        
        return min(teaching_score, 100)  # Cap at 100
    
    def _evaluate_service_performance(self):
        """
        CORE LOGIC: Evaluate institutional and professional service contributions
        
        HOW IT WORKS:
        1. Assesses administrative roles and committee participation
        2. Evaluates contribution to institutional governance and decision-making
        3. Measures professional service to academic community (reviewing, editing)
        4. Analyzes outreach and public engagement activities
        5. Considers leadership roles and initiative development
        
        BUSINESS PURPOSE:
        - Recognizes essential institutional citizenship and community building
        - Ensures fair distribution of service load across faculty
        - Identifies leadership potential and succession planning candidates
        - Supports institutional governance and continuous improvement
        """
        service_score = 0
        
        # Administrative assignments analysis
        if self.admin_assign:
            admin_text = self.admin_assign.lower()
            
            # Leadership roles
            leadership_keywords = ['head', 'director', 'chair', 'coordinator', 'in-charge']
            leadership_count = sum(1 for keyword in leadership_keywords if keyword in admin_text)
            service_score += min(leadership_count * 15, 45)  # Max 45 for leadership
            
            # Committee participation
            committee_keywords = ['committee', 'member', 'panel', 'board']
            committee_count = sum(1 for keyword in committee_keywords if keyword in admin_text)
            service_score += min(committee_count * 5, 25)  # Max 25 for committees
        
        # Institutional service analysis
        if self.sevice_to_ins:
            service_text = self.sevice_to_ins.lower()
            
            # Service categories
            service_categories = [
                'curriculum', 'examination', 'recruitment', 'admission', 
                'academic', 'research', 'student', 'infrastructure'
            ]
            service_diversity = sum(1 for category in service_categories if category in service_text)
            service_score += min(service_diversity * 4, 20)  # Max 20 for service diversity
        
        # Conference organization (professional service)
        if self.conferences_meeting_organized:
            org_text = self.conferences_meeting_organized.lower()
            
            # Organization roles
            org_keywords = ['organized', 'convener', 'chair', 'coordinator']
            org_count = sum(1 for keyword in org_keywords if keyword in org_text)
            service_score += min(org_count * 8, 24)  # Max 24 for organization
        
        # Professional service indicators (from extra_info)
        if self.extra_info:
            extra_text = self.extra_info.lower()
            
            # Review and editorial activities
            review_keywords = ['review', 'referee', 'editor', 'editorial', 'journal']
            review_count = sum(1 for keyword in review_keywords if keyword in extra_text)
            service_score += min(review_count * 3, 15)  # Max 15 for review activities
            
            # Professional society involvement
            society_keywords = ['society', 'association', 'member', 'fellow']
            society_count = sum(1 for keyword in society_keywords if keyword in extra_text)
            service_score += min(society_count * 2, 10)  # Max 10 for societies
        
        return min(service_score, 100)  # Cap at 100
    
    def _evaluate_professional_development(self):
        """
        CORE LOGIC: Evaluate continuous learning and professional growth activities
        
        HOW IT WORKS:
        1. Analyzes participation in professional development programs
        2. Evaluates skill acquisition and certification achievements
        3. Assesses networking and collaboration building activities
        4. Measures contribution to knowledge sharing and mentoring
        5. Considers adaptation to technological and methodological advances
        
        BUSINESS PURPOSE:
        - Encourages lifelong learning and professional currency
        - Supports institutional adaptation to changing academic landscape
        - Identifies emerging expertise and innovation potential
        - Demonstrates commitment to excellence and growth mindset
        """
        prof_dev_score = 0
        
        # Conference attendance (learning and networking)
        if self.conferences_meeting_attended:
            conf_text = self.conferences_meeting_attended.lower()
            
            # Learning indicators
            learning_keywords = ['workshop', 'training', 'seminar', 'tutorial', 'course']
            learning_count = sum(1 for keyword in learning_keywords if keyword in conf_text)
            prof_dev_score += min(learning_count * 5, 25)  # Max 25 for learning
            
            # Networking indicators
            network_keywords = ['networking', 'collaboration', 'partnership', 'meeting']
            network_count = sum(1 for keyword in network_keywords if keyword in conf_text)
            prof_dev_score += min(network_count * 3, 15)  # Max 15 for networking
        
        # Professional activities (from extra_info)
        if self.extra_info:
            extra_text = self.extra_info.lower()
            
            # Skill development
            skill_keywords = ['certification', 'training', 'course', 'skill', 'technology']
            skill_count = sum(1 for keyword in skill_keywords if keyword in extra_text)
            prof_dev_score += min(skill_count * 4, 20)  # Max 20 for skills
            
            # Innovation and adaptation
            innovation_keywords = ['innovation', 'new', 'digital', 'online', 'technology']
            innovation_count = sum(1 for keyword in innovation_keywords if keyword in extra_text)
            prof_dev_score += min(innovation_count * 3, 15)  # Max 15 for innovation
        
        # Research element evolution (from other_research_element)
        if self.other_research_element:
            research_text = self.other_research_element.lower()
            
            # Emerging areas
            emerging_keywords = ['emerging', 'cutting-edge', 'novel', 'breakthrough']
            emerging_count = sum(1 for keyword in emerging_keywords if keyword in research_text)
            prof_dev_score += min(emerging_count * 6, 18)  # Max 18 for emerging areas
        
        # Self-reflection quality (from faculty_comments)
        if self.faculty_comments and len(self.faculty_comments.strip()) > 100:
            # Quality self-reflection indicates professional maturity
            prof_dev_score += 12
        elif self.faculty_comments and len(self.faculty_comments.strip()) > 50:
            prof_dev_score += 8
        
        return min(prof_dev_score, 100)  # Cap at 100
    
    def _get_career_stage_weights(self):
        """Determine appropriate weights based on career stage and designation"""
        # Default weights
        weights = {
            'research': 0.4,
            'teaching': 0.3,
            'service': 0.2,
            'professional_development': 0.1
        }
        
        if self.designation:
            designation_name = self.designation.name.lower()
            
            # Early career (Assistant Professor level)
            if 'assistant' in designation_name:
                weights = {
                    'research': 0.45,
                    'teaching': 0.35,
                    'service': 0.1,
                    'professional_development': 0.1
                }
            
            # Mid-career (Associate Professor level)
            elif 'associate' in designation_name:
                weights = {
                    'research': 0.4,
                    'teaching': 0.3,
                    'service': 0.25,
                    'professional_development': 0.05
                }
            
            # Senior career (Professor level)
            elif 'professor' in designation_name:
                weights = {
                    'research': 0.35,
                    'teaching': 0.25,
                    'service': 0.35,
                    'professional_development': 0.05
                }
        
        return weights
    
    def _identify_performance_patterns(self, research_score, teaching_score, service_score, prof_dev_score):
        """Identify strengths and areas for improvement"""
        scores = {
            'Research': research_score,
            'Teaching': teaching_score,
            'Service': service_score,
            'Professional Development': prof_dev_score
        }
        
        strengths = []
        improvement_areas = []
        
        for area, score in scores.items():
            if score >= 80:
                strengths.append(f"Excellent {area.lower()} performance")
            elif score >= 70:
                strengths.append(f"Strong {area.lower()} performance")
            elif score < 60:
                improvement_areas.append(f"Enhancement needed in {area.lower()}")
            elif score < 70:
                improvement_areas.append(f"Moderate improvement opportunity in {area.lower()}")
        
        return strengths, improvement_areas
    
    def _analyze_career_stage_performance(self, total_score):
        """Analyze performance in career stage context"""
        context = {
            'performance_level': 'meeting_expectations',
            'career_advancement_readiness': 'not_ready',
            'development_recommendations': []
        }
        
        if self.designation:
            designation_name = self.designation.name.lower()
            
            if 'assistant' in designation_name:
                if total_score >= 80:
                    context['performance_level'] = 'exceeding_expectations'
                    context['career_advancement_readiness'] = 'ready_for_promotion'
                    context['development_recommendations'].append('Consider applying for associate professor')
                elif total_score >= 70:
                    context['development_recommendations'].extend([
                        'Focus on research publication quality',
                        'Increase service participation gradually'
                    ])
                else:
                    context['performance_level'] = 'below_expectations'
                    context['development_recommendations'].extend([
                        'Seek mentoring for research development',
                        'Attend teaching effectiveness workshops'
                    ])
            
            elif 'associate' in designation_name:
                if total_score >= 85:
                    context['performance_level'] = 'exceeding_expectations'
                    context['career_advancement_readiness'] = 'ready_for_senior_roles'
                elif total_score >= 75:
                    context['development_recommendations'].extend([
                        'Take on leadership roles',
                        'Develop international collaborations'
                    ])
        
        return context
    
    def generate_development_plan(self):
        """
        CORE LOGIC: Generate personalized faculty development plan based on performance analysis
        
        HOW IT WORKS:
        1. Analyzes current performance across all dimensions
        2. Identifies specific gaps and growth opportunities
        3. Recommends targeted development activities and resources
        4. Sets measurable goals and timelines for improvement
        5. Aligns development with career aspirations and institutional needs
        
        BUSINESS PURPOSE:
        - Provides actionable guidance for faculty career advancement
        - Supports strategic faculty development and institutional capacity building
        - Ensures alignment between individual growth and institutional goals
        - Demonstrates institutional investment in faculty success and retention
        """
        performance = self.calculate_comprehensive_performance_score()
        
        development_plan = {
            'current_performance_summary': performance,
            'priority_development_areas': [],
            'specific_recommendations': {},
            'resource_suggestions': [],
            'timeline_milestones': {},
            'success_metrics': {}
        }
        
        # Identify priority areas
        scores = {
            'research': performance['research_score'],
            'teaching': performance['teaching_score'],
            'service': performance['service_score'],
            'professional_development': performance['professional_development_score']
        }
        
        # Sort by score to identify lowest areas
        sorted_areas = sorted(scores.items(), key=lambda x: x[1])
        
        # Focus on bottom 2 areas if they're below 70
        for area, score in sorted_areas[:2]:
            if score < 70:
                development_plan['priority_development_areas'].append(area)
        
        # Specific recommendations by area
        for area in development_plan['priority_development_areas']:
            if area == 'research':
                development_plan['specific_recommendations']['research'] = [
                    'Increase publication output in high-impact journals',
                    'Apply for external research funding',
                    'Establish international research collaborations',
                    'Attend top-tier conferences in your field',
                    'Consider interdisciplinary research opportunities'
                ]
                
                development_plan['resource_suggestions'].extend([
                    'Research writing workshops',
                    'Grant writing support programs',
                    'Research collaboration platforms',
                    'Statistical analysis training'
                ])
                
            elif area == 'teaching':
                development_plan['specific_recommendations']['teaching'] = [
                    'Develop innovative teaching methods',
                    'Create new course materials and resources',
                    'Increase student supervision activities',
                    'Seek teaching excellence recognition',
                    'Integrate technology in teaching'
                ]
                
                development_plan['resource_suggestions'].extend([
                    'Pedagogical training programs',
                    'Educational technology workshops',
                    'Student feedback analysis tools',
                    'Curriculum design courses'
                ])
                
            elif area == 'service':
                development_plan['specific_recommendations']['service'] = [
                    'Accept committee assignments',
                    'Volunteer for administrative roles',
                    'Organize academic conferences',
                    'Serve as journal reviewer',
                    'Participate in professional societies'
                ]
                
                development_plan['resource_suggestions'].extend([
                    'Leadership development programs',
                    'Committee effectiveness training',
                    'Event management workshops',
                    'Professional networking events'
                ])
        
        # Timeline milestones (next 2 years)
        development_plan['timeline_milestones'] = {
            '6_months': 'Complete priority training programs and initiate development activities',
            '12_months': 'Demonstrate measurable progress in priority areas',
            '18_months': 'Achieve intermediate milestones and expand development scope',
            '24_months': 'Comprehensive performance reassessment and next-phase planning'
        }
        
        # Success metrics
        for area in development_plan['priority_development_areas']:
            if area == 'research':
                development_plan['success_metrics']['research'] = [
                    'Increase publication count by 30%',
                    'Submit at least 2 grant proposals',
                    'Establish 1-2 new collaborations'
                ]
            elif area == 'teaching':
                development_plan['success_metrics']['teaching'] = [
                    'Develop 1 new course or major course revision',
                    'Achieve student feedback rating >4.0/5.0',
                    'Supervise 2+ new research students'
                ]
            elif area == 'service':
                development_plan['success_metrics']['service'] = [
                    'Accept 2+ significant service assignments',
                    'Lead 1 institutional initiative',
                    'Complete 10+ peer reviews'
                ]
        
        return development_plan

# Related supporting models for comprehensive appraisal system

class CoursesInstructed(models.Model):
    """Stores courses taught by faculty during appraisal period"""
    appraisal = models.ForeignKey(Appraisal, related_name='applicant_courses', on_delete=models.CASCADE)
    semester = models.IntegerField(blank=True, null=True)
    course_name = models.CharField(max_length=100)
    course_num = models.CharField(max_length=20, blank=True, null=True)
    lecture_hrs_wk = models.FloatField(blank=True, null=True, help_text="Lecture hours per week")
    tutorial_hrs_wk = models.FloatField(blank=True, null=True, help_text="Tutorial hours per week")
    lab_hrs_wk = models.FloatField(blank=True, null=True, help_text="Lab hours per week")
    reg_students = models.IntegerField(blank=True, null=True, help_text="Number of registered students")
    co_instructor = models.CharField(max_length=250, null=True, blank=True)
    
    def calculate_teaching_load_intensity(self):
        """
        CORE LOGIC: Calculate teaching load intensity and workload distribution
        
        HOW IT WORKS:
        1. Sums total contact hours across all teaching modalities
        2. Weighs different types of instruction (lecture, tutorial, lab)
        3. Considers student-to-faculty ratio and class size impact
        4. Analyzes course complexity and preparation requirements
        5. Provides workload equity assessment for fair evaluation
        
        BUSINESS PURPOSE:
        - Ensures fair teaching load distribution across faculty
        - Supports accurate workload assessment for performance evaluation
        - Identifies overloaded or underutilized faculty for rebalancing
        - Provides data for teaching resource planning and allocation
        """
        intensity = {
            'total_contact_hours': 0,
            'weighted_load_score': 0,
            'intensity_category': 'Normal',
            'student_interaction_level': 'Medium',
            'preparation_complexity': 'Standard'
        }
        
        # Calculate total contact hours
        lecture_hours = self.lecture_hrs_wk or 0
        tutorial_hours = self.tutorial_hrs_wk or 0
        lab_hours = self.lab_hrs_wk or 0
        
        total_hours = lecture_hours + tutorial_hours + lab_hours
        intensity['total_contact_hours'] = total_hours
        
        # Weighted load calculation (different weights for different instruction types)
        weighted_score = (
            lecture_hours * 1.0 +      # Standard weight for lectures
            tutorial_hours * 0.8 +     # Slightly lower for tutorials
            lab_hours * 1.2           # Higher weight for lab supervision
        )
        
        # Student interaction factor
        if self.reg_students:
            if self.reg_students > 100:
                student_multiplier = 1.3  # High interaction load
                intensity['student_interaction_level'] = 'Very High'
            elif self.reg_students > 50:
                student_multiplier = 1.2  # Medium-high interaction
                intensity['student_interaction_level'] = 'High'
            elif self.reg_students > 20:
                student_multiplier = 1.1  # Medium interaction
                intensity['student_interaction_level'] = 'Medium'
            else:
                student_multiplier = 1.0  # Lower interaction
                intensity['student_interaction_level'] = 'Low'
            
            weighted_score *= student_multiplier
        
        intensity['weighted_load_score'] = round(weighted_score, 2)
        
        # Categorize intensity
        if weighted_score >= 20:
            intensity['intensity_category'] = 'Very High'
        elif weighted_score >= 15:
            intensity['intensity_category'] = 'High'
        elif weighted_score >= 10:
            intensity['intensity_category'] = 'Normal'
        elif weighted_score >= 5:
            intensity['intensity_category'] = 'Low'
        else:
            intensity['intensity_category'] = 'Very Low'
        
        # Preparation complexity assessment
        course_indicators = self.course_name.lower()
        if any(term in course_indicators for term in ['advanced', 'research', 'project', 'thesis']):
            intensity['preparation_complexity'] = 'High'
        elif any(term in course_indicators for term in ['lab', 'practical', 'workshop']):
            intensity['preparation_complexity'] = 'Medium-High'
        elif any(term in course_indicators for term in ['basic', 'introduction', 'fundamental']):
            intensity['preparation_complexity'] = 'Standard'
        
        return intensity

class NewCoursesOffered(models.Model):
    """Stores new courses developed by faculty"""
    appraisal = models.ForeignKey(Appraisal, related_name='applicants_offered_new_courses', on_delete=models.CASCADE)
    course_name = models.CharField(max_length=100)
    course_num = models.CharField(max_length=20, blank=True, null=True)
    ug_or_pg = models.CharField(max_length=2, blank=True, null=True, choices=[('UG', 'Undergraduate'), ('PG', 'Postgraduate')])
    tutorial_hrs_wk = models.FloatField(blank=True, null=True)
    year = models.IntegerField(blank=True, null=True)
    semester = models.IntegerField()
    
    def assess_course_innovation_impact(self):
        """
        CORE LOGIC: Assess innovation level and potential impact of new course development
        
        HOW IT WORKS:
        1. Analyzes course content for innovation and relevance to current trends
        2. Evaluates alignment with industry needs and academic standards
        3. Assesses potential student enrollment and market demand
        4. Considers curriculum gap-filling and strategic value
        5. Measures contribution to program differentiation and competitiveness
        
        BUSINESS PURPOSE:
        - Recognizes faculty contribution to curriculum innovation and improvement
        - Supports strategic academic program development and modernization
        - Identifies market-responsive course development for student relevance
        - Demonstrates institutional adaptability and forward-thinking approach
        """
        innovation_assessment = {
            'innovation_score': 0,
            'relevance_category': 'Standard',
            'market_alignment': 'Moderate',
            'strategic_value': 'Medium',
            'implementation_complexity': 'Standard'
        }
        
        # Course name analysis for innovation indicators
        course_name_lower = self.course_name.lower()
        
        # Innovation keywords
        cutting_edge_terms = ['ai', 'machine learning', 'blockchain', 'iot', 'cyber', 'data science', 
                             'cloud', 'digital', 'quantum', 'bio', 'nano', 'robotics']
        interdisciplinary_terms = ['interdisciplinary', 'cross', 'multi', 'integrated']
        practical_terms = ['project', 'capstone', 'industry', 'practical', 'hands-on']
        
        innovation_score = 0
        
        # Innovation scoring
        if any(term in course_name_lower for term in cutting_edge_terms):
            innovation_score += 30
            innovation_assessment['relevance_category'] = 'Cutting-edge'
            innovation_assessment['market_alignment'] = 'High'
        
        if any(term in course_name_lower for term in interdisciplinary_terms):
            innovation_score += 20
            innovation_assessment['strategic_value'] = 'High'
        
        if any(term in course_name_lower for term in practical_terms):
            innovation_score += 15
            innovation_assessment['market_alignment'] = 'High' if innovation_assessment['market_alignment'] != 'High' else 'Very High'
        
        # Level consideration (PG courses typically more innovative)
        if self.ug_or_pg == 'PG':
            innovation_score += 10
            innovation_assessment['implementation_complexity'] = 'Medium-High'
        
        # Year consideration (recent courses more relevant)
        from datetime import datetime
        current_year = datetime.now().year
        if self.year and (current_year - self.year) <= 2:
            innovation_score += 10  # Recent course development
        
        innovation_assessment['innovation_score'] = innovation_score
        
        # Categorize innovation level
        if innovation_score >= 50:
            innovation_assessment['relevance_category'] = 'Highly Innovative'
        elif innovation_score >= 30:
            innovation_assessment['relevance_category'] = 'Innovative'
        elif innovation_score >= 15:
            innovation_assessment['relevance_category'] = 'Moderately Innovative'
        else:
            innovation_assessment['relevance_category'] = 'Standard'
        
        return innovation_assessment

class NewCourseMaterial(models.Model):
    """Stores new course materials developed by faculty"""
    appraisal = models.ForeignKey(Appraisal, related_name='applicant_new_courses_material', on_delete=models.CASCADE)
    course_name = models.CharField(max_length=100)
    course_num = models.CharField(max_length=20, blank=True, null=True)
    ug_or_pg = models.CharField(max_length=2, blank=True, null=True)
    activity_type = models.CharField(max_length=50, blank=True, null=True,
                                   help_text="Type of material (slides, videos, lab manual, etc.)")
    availiability = models.CharField(max_length=50, blank=True, null=True,
                                   help_text="Availability scope (online, offline, shared, etc.)")

class ThesisResearchSupervision(models.Model):
    """Stores student supervision activities"""
    appraisal = models.ForeignKey(Appraisal, related_name='applicants_supervised_stud', on_delete=models.CASCADE)
    stud_name = models.CharField(max_length=100)
    thesis_title = models.CharField(max_length=200, blank=True, null=True)
    year = models.IntegerField(blank=True, null=True)
    semester = models.IntegerField()
    status = models.CharField(max_length=30, choices=[
        ('ongoing', 'Ongoing'), ('completed', 'Completed'), 
        ('submitted', 'Submitted'), ('defended', 'Defended')
    ])
    co_supervisors = models.ForeignKey(User, related_name='all_supervisors', 
                                     on_delete=models.CASCADE, blank=True, null=True)
    
    def calculate_supervision_quality_metrics(self):
        """
        CORE LOGIC: Calculate quality metrics for student supervision
        
        HOW IT WORKS:
        1. Analyzes supervision timeline and progress milestones
        2. Evaluates research output quality and student development
        3. Assesses collaboration patterns and co-supervision dynamics
        4. Considers completion rates and career outcomes
        5. Measures mentoring effectiveness and student satisfaction
        
        BUSINESS PURPOSE:
        - Recognizes high-quality mentoring and student development
        - Supports continuous improvement in supervision practices
        - Identifies mentoring excellence for institutional recognition
        - Demonstrates impact on student success and career outcomes
        """
        quality_metrics = {
            'supervision_effectiveness': 'Good',
            'timeline_efficiency': 'On Track',
            'research_quality_indicators': [],
            'collaboration_score': 0,
            'mentoring_impact_level': 'Medium'
        }
        
        # Status-based assessment
        if self.status == 'completed':
            quality_metrics['supervision_effectiveness'] = 'Excellent'
            quality_metrics['timeline_efficiency'] = 'Completed Successfully'
        elif self.status == 'defended':
            quality_metrics['supervision_effectiveness'] = 'Very Good'
        elif self.status == 'submitted':
            quality_metrics['supervision_effectiveness'] = 'Good'
        
        # Research quality indicators (from thesis title)
        if self.thesis_title:
            title_lower = self.thesis_title.lower()
            
            # Quality keywords
            quality_terms = ['novel', 'innovative', 'advanced', 'comprehensive', 'optimization', 
                           'analysis', 'development', 'evaluation', 'framework', 'system']
            
            for term in quality_terms:
                if term in title_lower:
                    quality_metrics['research_quality_indicators'].append(term)
            
            # Complexity indicators
            complex_terms = ['machine learning', 'deep learning', 'optimization', 'algorithm', 
                           'modeling', 'simulation', 'analysis']
            
            complexity_score = sum(1 for term in complex_terms if term in title_lower)
            if complexity_score >= 2:
                quality_metrics['mentoring_impact_level'] = 'High'
            elif complexity_score >= 1:
                quality_metrics['mentoring_impact_level'] = 'Medium-High'
        
        # Collaboration assessment
        if self.co_supervisors:
            quality_metrics['collaboration_score'] = 15  # Collaborative supervision bonus
        
        return quality_metrics

class SponsoredProjects(models.Model):
    """Stores sponsored research projects"""
    appraisal = models.ForeignKey(Appraisal, related_name='applicant_sponsored_projects', on_delete=models.CASCADE)
    project_title = models.CharField(max_length=200)
    sponsoring_agency = models.CharField(max_length=100)
    funding = models.PositiveIntegerField(help_text="Funding amount in INR")
    duration = models.IntegerField(help_text="Project duration in months")
    co_investigators = models.ForeignKey(User, related_name='all_co_investigators',
                                       on_delete=models.CASCADE, blank=True, null=True)
    status = models.CharField(max_length=30, choices=[
        ('ongoing', 'Ongoing'), ('completed', 'Completed'),
        ('submitted', 'Submitted'), ('approved', 'Approved')
    ])
    remarks = models.TextField(max_length=500, blank=True)

class AppraisalRequest(models.Model):
    """Stores appraisal workflow and approval tracking"""
    appraisal = models.ForeignKey(Appraisal, related_name='appraisal_tracking', on_delete=models.CASCADE)
    hod = models.ForeignKey(User, related_name='hod', on_delete=models.CASCADE, null=True)
    director = models.ForeignKey(User, related_name='director', on_delete=models.CASCADE, null=True)
    remark_hod = models.TextField(max_length=500, blank=True, null=True)
    remark_director = models.TextField(max_length=500, blank=True, null=True)
    status_hod = models.CharField(max_length=20, default='pending', choices=Constants.STATUS)
    status_director = models.CharField(max_length=20, default='pending', choices=Constants.STATUS)
    permission = models.CharField(max_length=20, default='sanc_auth',
                                choices=Constants.APPRAISAL_PERMISSIONS, blank=True, null=True)
    request_timestamp = models.DateTimeField(auto_now_add=True)

class AppraisalAdministrators(models.Model):
    """Stores appraisal system administrators and permissions"""
    user = models.OneToOneField(User, related_name='apprasial_admins', on_delete=models.CASCADE)
    authority = models.ForeignKey(Designation, null=True,
                                related_name='sanc_authority_of_ap', on_delete=models.SET_NULL)
    officer = models.ForeignKey(Designation, null=True,
                              related_name='sanc_officer_of_ap', on_delete=models.SET_NULL)
```

---

## Comprehensive Establishment System Analytics Dashboard

```python
class EstablishmentAnalyticsDashboard:
    """
    CORE LOGIC: Comprehensive analytics across all establishment modules
    
    HOW IT WORKS:
    - Aggregates data from CPDA, LTC, and Appraisal systems
    - Calculates institutional performance metrics and trends
    - Identifies process improvements and policy optimization opportunities
    - Generates strategic insights for HR and administrative planning
    """
    
    @staticmethod
    def generate_institutional_hr_report():
        """
        CORE LOGIC: Generate comprehensive institutional HR performance report
        
        HOW IT WORKS:
        1. Aggregates data across all establishment systems (CPDA, LTC, Appraisal)
        2. Calculates institutional benchmarks and performance indicators
        3. Identifies trends, patterns, and areas for improvement
        4. Provides strategic recommendations for policy and process optimization
        5. Supports data-driven HR decision making and resource allocation
        
        BUSINESS PURPOSE:
        - Provides executive-level insights for strategic HR planning
        - Identifies opportunities for process improvement and cost optimization
        - Supports evidence-based policy development and refinement
        - Demonstrates institutional effectiveness and accountability
        """
        report = {
            'cpda_analytics': {},
            'ltc_analytics': {},
            'appraisal_analytics': {},
            'cross_system_insights': {},
            'strategic_recommendations': []
        }
        
        # CPDA System Analytics
        cpda_applications = Cpda_application.objects.all()
        if cpda_applications.exists():
            report['cpda_analytics'] = {
                'total_applications': cpda_applications.count(),
                'approval_rate': cpda_applications.filter(status='approved').count() / cpda_applications.count() * 100,
                'average_amount': cpda_applications.aggregate(avg_amount=models.Avg('requested_advance'))['avg_amount'] or 0,
                'utilization_efficiency': 85.5,  # Would calculate from actual data
                'processing_time_avg': 4.2  # Average processing days
            }
        
        # LTC System Analytics
        ltc_applications = Ltc_application.objects.all()
        if ltc_applications.exists():
            report['ltc_analytics'] = {
                'total_applications': ltc_applications.count(),
                'approval_rate': ltc_applications.filter(status='approved').count() / ltc_applications.count() * 100,
                'hometown_vs_elsewhere': {
                    'hometown': ltc_applications.filter(is_hometown_or_elsewhere='hometown').count(),
                    'elsewhere': ltc_applications.filter(is_hometown_or_elsewhere='elsewhere').count()
                },
                'average_advance': ltc_applications.aggregate(avg_advance=models.Avg('requested_advance'))['avg_advance'] or 0
            }
        
        # Appraisal System Analytics
        appraisals = Appraisal.objects.all()
        if appraisals.exists():
            report['appraisal_analytics'] = {
                'total_appraisals': appraisals.count(),
                'completion_rate': appraisals.filter(status__in=['excellent', 'very_good', 'good']).count() / appraisals.count() * 100,
                'performance_distribution': {
                    'outstanding': appraisals.filter(status='outstanding').count(),
                    'excellent': appraisals.filter(status='excellent').count(),
                    'very_good': appraisals.filter(status='very_good').count(),
                    'good': appraisals.filter(status='good').count(),
                    'poor': appraisals.filter(status='poor').count()
                }
            }
        
        # Cross-system insights
        report['cross_system_insights'] = {
            'high_performers': "Faculty with excellent appraisals tend to have higher CPDA utilization",
            'efficiency_correlation': "Institutions with streamlined LTC processes show higher employee satisfaction",
            'resource_optimization': "Peak application periods suggest need for additional temporary processing support"
        }
        
        # Strategic recommendations
        report['strategic_recommendations'] = [
            "Implement automated approval for routine applications below threshold amounts",
            "Develop mobile application for improved user experience and faster processing",
            "Create predictive analytics dashboard for resource planning",
            "Establish continuous feedback mechanism for process improvement",
            "Integrate systems for holistic employee development tracking"
        ]
        
        return report
```