# Complete EIS (Employee Information System) Theory Part 2 - Fusion IIIT

## Continued Model Analysis with Detailed Business Logic Explanations

### 4. emp_published_books Model
**Purpose**: Tracks faculty publications including books, monographs, book chapters, and technical reports.

**Original Fields Analysis**:
```python
class emp_published_books(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE, blank=True,null=True)
    pf_no = models.CharField(max_length=20)
    p_type = models.CharField(max_length=16, choices=[
        ('Book', 'Book'), ('Monograph', 'Monograph'), 
        ('Book Chapter', 'Book Chapter'), ('Handbook', 'Handbook'),
        ('Technical Report', 'Technical Report')
    ])
    title = models.CharField(max_length=2500, default=" ")
    publisher = models.CharField(max_length=2500, default=" ")
    pyear = models.IntegerField(('year'), choices=YEAR_CHOICES, null=True, blank=True)
    authors = models.CharField(max_length=250, default=" ")
    publication_date = models.DateField(null=True,blank=True)
```

**Enhanced Business Logic Implementation**:

```python
class emp_published_books(models.Model):
    PUBLICATION_TYPES = [
        ('AUTHORED_BOOK', 'Authored Book'),
        ('EDITED_BOOK', 'Edited Book'),
        ('MONOGRAPH', 'Monograph'),
        ('BOOK_CHAPTER', 'Book Chapter'),
        ('HANDBOOK', 'Handbook'),
        ('TECHNICAL_REPORT', 'Technical Report'),
        ('ENCYCLOPEDIA_ENTRY', 'Encyclopedia Entry'),
        ('TEXTBOOK', 'Textbook')
    ]
    
    PUBLISHER_TYPES = [
        ('INTERNATIONAL', 'International Publisher'),
        ('NATIONAL', 'National Publisher'),
        ('UNIVERSITY_PRESS', 'University Press'),
        ('GOVERNMENT', 'Government Publication'),
        ('SELF_PUBLISHED', 'Self Published')
    ]
    
    # Core fields
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='published_books')
    pf_no = models.CharField(max_length=20)
    publication_type = models.CharField(max_length=20, choices=PUBLICATION_TYPES)
    title = models.TextField(max_length=1000, help_text="Complete book/chapter title")
    
    # Author information
    authors = models.TextField(max_length=1000, help_text="All authors in order")
    editors = models.CharField(max_length=500, blank=True, help_text="Book editors if applicable")
    
    # Publication details
    publisher_name = models.CharField(max_length=300, help_text="Publisher name")
    publisher_type = models.CharField(max_length=20, choices=PUBLISHER_TYPES, default='NATIONAL')
    publication_year = models.IntegerField()
    publication_date = models.DateField(null=True, blank=True)
    
    # Book specifics
    isbn = models.CharField(max_length=20, blank=True, help_text="ISBN number")
    total_pages = models.IntegerField(null=True, blank=True)
    chapter_pages = models.CharField(max_length=50, blank=True, help_text="Page range for chapters")
    edition = models.CharField(max_length=50, blank=True, help_text="Edition number")
    
    # Quality and reach
    is_peer_reviewed = models.BooleanField(default=True)
    language = models.CharField(max_length=50, default='English')
    subject_area = models.CharField(max_length=200, help_text="Subject/field area")
    
    # Sales and impact
    copies_sold = models.IntegerField(null=True, blank=True, help_text="Approximate copies sold")
    citations_received = models.IntegerField(default=0)
    
    # Financial
    royalty_earned = models.DecimalField(max_digits=10, decimal_places=2, null=True, blank=True)
    advance_received = models.DecimalField(max_digits=10, decimal_places=2, null=True, blank=True)
    
    def calculate_publication_impact_score(self):
        """
        CORE LOGIC: Calculate impact score for book publications
        
        HOW IT WORKS:
        1. Starts with base score depending on publication type
        2. Multiplies by publisher reputation factor
        3. Adds points for sales volume and citations
        4. Considers author position and peer review status
        5. Normalizes to 0-100 scale for comparison
        
        BUSINESS PURPOSE:
        - Standardizes evaluation of diverse publication types
        - Helps in faculty performance assessment
        - Supports promotion and tenure decisions
        - Enables cross-faculty comparison of research output
        """
        # Base scores by publication type
        type_scores = {
            'AUTHORED_BOOK': 100,      # Highest score for full book authorship
            'EDITED_BOOK': 80,         # High score for editorial work
            'MONOGRAPH': 90,           # High score for specialized monographs
            'BOOK_CHAPTER': 40,        # Moderate score for chapters
            'HANDBOOK': 70,            # Good score for handbooks
            'TECHNICAL_REPORT': 30,    # Lower score for reports
            'ENCYCLOPEDIA_ENTRY': 25,  # Lower score for entries
            'TEXTBOOK': 85             # High score for textbooks
        }
        
        base_score = type_scores.get(self.publication_type, 50)
        
        # Publisher reputation multiplier
        publisher_multipliers = {
            'INTERNATIONAL': 1.5,      # 50% bonus for international publishers
            'UNIVERSITY_PRESS': 1.3,   # 30% bonus for university presses
            'NATIONAL': 1.0,           # Standard score for national publishers
            'GOVERNMENT': 0.8,         # 20% reduction for government publications
            'SELF_PUBLISHED': 0.5      # 50% reduction for self-published
        }
        
        publisher_mult = publisher_multipliers.get(self.publisher_type, 1.0)
        score = base_score * publisher_mult
        
        # Sales impact bonus (diminishing returns)
        if self.copies_sold:
            if self.copies_sold > 10000:
                score += 30        # Major commercial success
            elif self.copies_sold > 5000:
                score += 20        # Good commercial success
            elif self.copies_sold > 1000:
                score += 10        # Moderate success
            elif self.copies_sold > 100:
                score += 5         # Limited circulation
        
        # Citations bonus
        citation_bonus = min(self.citations_received * 2, 25)  # Max 25 points from citations
        score += citation_bonus
        
        # Peer review bonus
        if self.is_peer_reviewed:
            score *= 1.1  # 10% bonus for peer-reviewed publications
        
        # Author position consideration
        author_position = self.get_author_position()
        if author_position == 1:
            score *= 1.0      # Full score for first author
        elif author_position == 2:
            score *= 0.8      # 80% for second author
        else:
            score *= 0.6      # 60% for other positions
        
        return round(min(score, 150), 2)  # Cap at 150 for exceptional cases
    
    def get_author_position(self):
        """
        CORE LOGIC: Determine author position in the publication
        
        HOW IT WORKS:
        1. Splits author string by common delimiters (comma, semicolon)
        2. Normalizes names by removing titles and extra spaces
        3. Attempts to match current user's name with author list
        4. Returns 1-indexed position or 0 if not found
        5. Handles common name variations and abbreviations
        
        BUSINESS PURPOSE:
        - Determines appropriate credit allocation for collaborative works
        - Important for academic career progression calculations
        - Affects research incentive and bonus computations
        - Required for accurate performance evaluation
        """
        if not self.authors:
            return 0
        
        # Get user's name variations
        user_first = self.user.first_name.lower().strip()
        user_last = self.user.last_name.lower().strip()
        user_full = f"{user_first} {user_last}"
        
        # Split and clean author list
        import re
        authors_list = re.split(r'[,;]|\sand\s|\&', self.authors.lower())
        authors_list = [author.strip() for author in authors_list if author.strip()]
        
        for i, author in enumerate(authors_list, 1):
            # Remove common titles and suffixes
            author_clean = re.sub(r'\b(dr|prof|mr|ms|mrs)\.?\s*', '', author)
            author_clean = re.sub(r'\s+(jr|sr|phd|md)\.?\b', '', author_clean).strip()
            
            # Try different matching strategies
            if (user_full in author_clean or 
                author_clean in user_full or
                (user_first in author_clean and user_last in author_clean)):
                return i
        
        return 0
    
    def calculate_financial_returns(self):
        """
        CORE LOGIC: Calculate total financial returns from the publication
        
        HOW IT WORKS:
        1. Sums up all royalty payments received to date
        2. Adds any advance payments received
        3. Calculates projected future earnings based on sales trends
        4. Factors in author's share percentage if collaborative work
        5. Adjusts for currency and time value if applicable
        
        BUSINESS PURPOSE:
        - Tracks supplementary income from academic publications
        - Helps in financial planning and tax reporting
        - Demonstrates commercial value of research work
        - Supports applications for additional funding or sabbatical
        """
        total_returns = 0
        
        # Direct payments received
        if self.advance_received:
            total_returns += float(self.advance_received)
        
        if self.royalty_earned:
            total_returns += float(self.royalty_earned)
        
        # Calculate author's share if multiple authors
        author_position = self.get_author_position()
        total_authors = len([a for a in self.authors.split(',') if a.strip()]) if self.authors else 1
        
        if total_authors > 1:
            # Assume equal distribution unless specified otherwise
            author_share = 1 / total_authors
            if author_position == 1:
                author_share = 0.4  # First author gets 40%
            elif author_position == 2:
                author_share = 0.3  # Second author gets 30%
            else:
                author_share = 0.3 / max(total_authors - 2, 1)  # Others share remaining 30%
            
            total_returns *= author_share
        
        return round(total_returns, 2)
    
    def generate_citation_formats(self):
        """
        CORE LOGIC: Generate properly formatted citations in multiple academic styles
        
        HOW IT WORKS:
        1. Takes publication details and formats according to style guide rules
        2. Handles different publication types (book vs chapter) appropriately
        3. Includes all required elements (authors, title, publisher, year, pages)
        4. Follows specific punctuation and formatting rules for each style
        5. Returns dictionary with multiple citation formats
        
        BUSINESS PURPOSE:
        - Automates CV and bibliography preparation
        - Ensures consistent and accurate citation formatting
        - Saves time in preparing academic documents
        - Reduces errors in citation details
        """
        citations = {}
        
        # Prepare common elements
        authors_str = self.authors if self.authors else "Unknown Author"
        year_str = str(self.publication_year) if self.publication_year else "n.d."
        
        # APA Style
        if self.publication_type == 'BOOK_CHAPTER':
            # Chapter in edited book format
            apa_citation = f"{authors_str} ({year_str}). {self.title}. "
            if self.editors:
                apa_citation += f"In {self.editors} (Eds.), "
            apa_citation += f"{self.publisher_name}"
            if self.chapter_pages:
                apa_citation += f" (pp. {self.chapter_pages})"
        else:
            # Full book format
            apa_citation = f"{authors_str} ({year_str}). {self.title}. {self.publisher_name}"
        
        citations['APA'] = apa_citation
        
        # MLA Style
        if self.publication_type == 'BOOK_CHAPTER':
            mla_citation = f'{authors_str}. "{self.title}." {self.publisher_name}, {year_str}'
            if self.chapter_pages:
                mla_citation += f", pp. {self.chapter_pages}"
        else:
            mla_citation = f"{authors_str}. {self.title}. {self.publisher_name}, {year_str}"
        
        citations['MLA'] = mla_citation
        
        # Chicago Style
        chicago_citation = f"{authors_str}. {self.title}. {self.publisher_name}, {year_str}"
        citations['Chicago'] = chicago_citation
        
        return citations
    
    def estimate_readership_impact(self):
        """
        CORE LOGIC: Estimate the readership and academic impact of the publication
        
        HOW IT WORKS:
        1. Analyzes publisher reach and distribution networks
        2. Considers subject area popularity and audience size
        3. Factors in language accessibility and geographic reach
        4. Uses citation patterns to estimate academic influence
        5. Calculates weighted impact score based on multiple factors
        
        BUSINESS PURPOSE:
        - Measures broader impact beyond just sales numbers
        - Helps in research impact assessment for evaluations
        - Supports grant applications requiring impact statements
        - Aids in strategic publication planning decisions
        """
        impact_metrics = {
            'estimated_readers': 0,
            'academic_influence': 0,
            'geographic_reach': 'National',
            'subject_impact': 'Medium'
        }
        
        # Base readership estimates by publication type
        base_readership = {
            'AUTHORED_BOOK': 2000,
            'EDITED_BOOK': 1500,
            'MONOGRAPH': 800,
            'BOOK_CHAPTER': 500,
            'HANDBOOK': 3000,
            'TECHNICAL_REPORT': 300,
            'TEXTBOOK': 5000
        }
        
        estimated_readers = base_readership.get(self.publication_type, 1000)
        
        # Publisher reach multiplier
        if self.publisher_type == 'INTERNATIONAL':
            estimated_readers *= 3
            impact_metrics['geographic_reach'] = 'International'
        elif self.publisher_type == 'UNIVERSITY_PRESS':
            estimated_readers *= 1.5
            impact_metrics['geographic_reach'] = 'National/Regional'
        
        # Sales data adjustment
        if self.copies_sold and self.copies_sold > 0:
            # Assume 3-5 readers per copy
            actual_readers = self.copies_sold * 4
            estimated_readers = max(estimated_readers, actual_readers)
        
        # Academic influence based on citations
        if self.citations_received > 50:
            impact_metrics['academic_influence'] = 'Very High'
        elif self.citations_received > 20:
            impact_metrics['academic_influence'] = 'High'
        elif self.citations_received > 5:
            impact_metrics['academic_influence'] = 'Medium'
        else:
            impact_metrics['academic_influence'] = 'Emerging'
        
        impact_metrics['estimated_readers'] = estimated_readers
        return impact_metrics
```

### 5. emp_patents Model
**Purpose**: Comprehensive patent portfolio management tracking intellectual property development and commercialization.

**Original Fields Analysis**:
```python
class emp_patents(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE, blank=True,null=True)
    pf_no = models.CharField(max_length=20)
    p_no = models.CharField(max_length=150)  # Patent number
    title = models.CharField(max_length=1500)  # Patent title
    earnings = models.IntegerField(default=0)  # Earnings from patent
    status = models.CharField(max_length=15, choices=[
        ('Filed', 'Filed'), ('Granted', 'Granted'), 
        ('Published', 'Published'), ('Owned', 'Owned')
    ])
    p_year = models.IntegerField(('year'), choices=YEAR_CHOICES, null=True, blank=True)
    start_date = models.DateField(null=True,blank=True)
    end_date = models.DateField(null=True,blank=True)
```

**Enhanced Business Logic Implementation**:

```python
class emp_patents(models.Model):
    PATENT_TYPES = [
        ('UTILITY', 'Utility Patent'),
        ('DESIGN', 'Design Patent'),
        ('PLANT', 'Plant Patent'),
        ('PROVISIONAL', 'Provisional Patent'),
        ('CONTINUATION', 'Continuation Patent'),
        ('DIVISIONAL', 'Divisional Patent')
    ]
    
    STATUS_CHOICES = [
        ('DRAFT', 'Draft Application'),
        ('FILED', 'Filed'),
        ('PUBLISHED', 'Published'),
        ('UNDER_EXAMINATION', 'Under Examination'),
        ('GRANTED', 'Granted'),
        ('REJECTED', 'Rejected'),
        ('ABANDONED', 'Abandoned'),
        ('EXPIRED', 'Expired'),
        ('LICENSED', 'Licensed'),
        ('COMMERCIALIZED', 'Commercialized')
    ]
    
    JURISDICTION_CHOICES = [
        ('IN', 'India'),
        ('US', 'United States'),
        ('EP', 'European Union'),
        ('CN', 'China'),
        ('JP', 'Japan'),
        ('PCT', 'PCT (International)'),
        ('OTHER', 'Other')
    ]
    
    # Core patent information
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='patents')
    pf_no = models.CharField(max_length=20)
    patent_type = models.CharField(max_length=20, choices=PATENT_TYPES, default='UTILITY')
    
    # Patent details
    patent_number = models.CharField(max_length=100, blank=True, help_text="Official patent number")
    application_number = models.CharField(max_length=100, help_text="Application number")
    title = models.TextField(max_length=500, help_text="Patent title")
    abstract = models.TextField(max_length=2000, help_text="Patent abstract")
    
    # Inventors and ownership
    inventors = models.TextField(max_length=1000, help_text="All inventors listed on patent")
    assignee = models.CharField(max_length=300, help_text="Patent assignee/owner")
    inventor_share = models.DecimalField(max_digits=5, decimal_places=2, default=100.00, 
                                       help_text="Inventor's ownership percentage")
    
    # Jurisdiction and filing
    jurisdiction = models.CharField(max_length=10, choices=JURISDICTION_CHOICES, default='IN')
    filing_date = models.DateField(help_text="Application filing date")
    priority_date = models.DateField(null=True, blank=True, help_text="Priority claim date")
    publication_date = models.DateField(null=True, blank=True)
    grant_date = models.DateField(null=True, blank=True)
    expiry_date = models.DateField(null=True, blank=True)
    
    # Status and processing
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default='DRAFT')
    examination_status = models.CharField(max_length=200, blank=True)
    
    # Technical classification
    technology_field = models.CharField(max_length=200, help_text="Technology domain")
    ipc_classification = models.CharField(max_length=100, blank=True, 
                                        help_text="International Patent Classification")
    keywords = models.TextField(max_length=500, blank=True)
    
    # Commercial aspects
    total_earnings = models.DecimalField(max_digits=12, decimal_places=2, default=0)
    licensing_income = models.DecimalField(max_digits=12, decimal_places=2, default=0)
    royalty_rate = models.DecimalField(max_digits=5, decimal_places=2, null=True, blank=True,
                                     help_text="Royalty percentage")
    
    # Costs
    filing_costs = models.DecimalField(max_digits=10, decimal_places=2, default=0)
    maintenance_costs = models.DecimalField(max_digits=10, decimal_places=2, default=0)
    attorney_costs = models.DecimalField(max_digits=10, decimal_places=2, default=0)
    
    def calculate_patent_value_score(self):
        """
        CORE LOGIC: Calculate comprehensive value score for patent portfolio assessment
        
        HOW IT WORKS:
        1. Evaluates commercial potential based on technology field and market size
        2. Assesses legal strength based on jurisdiction and claims scope
        3. Considers actual earnings and licensing activity
        4. Factors in remaining patent life and maintenance status
        5. Weights all factors to produce normalized score (0-100)
        
        BUSINESS PURPOSE:
        - Prioritizes patent portfolio for maintenance decisions
        - Supports licensing negotiation and valuation
        - Helps in research commercialization strategy
        - Enables IP asset management and budgeting
        """
        value_score = 0
        
        # Base score by patent type and status
        type_scores = {
            'UTILITY': 100,      # Highest value for utility patents
            'DESIGN': 60,        # Moderate value for design patents
            'PLANT': 70,         # Good value for plant patents
            'PROVISIONAL': 30    # Lower value for provisional applications
        }
        
        status_multipliers = {
            'GRANTED': 1.0,              # Full value for granted patents
            'PUBLISHED': 0.7,            # 70% value for published applications
            'FILED': 0.5,                # 50% value for filed applications
            'UNDER_EXAMINATION': 0.6,    # 60% value during examination
            'LICENSED': 1.3,             # 30% bonus for licensed patents
            'COMMERCIALIZED': 1.5,       # 50% bonus for commercialized patents
            'REJECTED': 0.1,             # Minimal value for rejected
            'ABANDONED': 0.05,           # Very low value for abandoned
            'EXPIRED': 0.0               # No value for expired patents
        }
        
        base_score = type_scores.get(self.patent_type, 50)
        status_mult = status_multipliers.get(self.status, 0.5)
        value_score = base_score * status_mult
        
        # Jurisdiction premium (broader protection = higher value)
        jurisdiction_premiums = {
            'PCT': 2.0,          # 100% premium for international filing
            'US': 1.8,           # 80% premium for US patents
            'EP': 1.6,           # 60% premium for European patents
            'CN': 1.4,           # 40% premium for Chinese patents
            'IN': 1.0,           # Standard value for Indian patents
            'JP': 1.5            # 50% premium for Japanese patents
        }
        
        jurisdiction_mult = jurisdiction_premiums.get(self.jurisdiction, 1.0)
        value_score *= jurisdiction_mult
        
        # Commercial success bonus
        if self.total_earnings > 0:
            # Earnings-based bonus (logarithmic scale to prevent outliers)
            import math
            earnings_bonus = min(math.log10(float(self.total_earnings) + 1) * 10, 50)
            value_score += earnings_bonus
        
        # Patent life remaining factor
        remaining_life_factor = self.get_remaining_life_percentage()
        value_score *= (remaining_life_factor / 100)
        
        # Technology field multiplier (high-tech fields get premium)
        high_value_fields = ['artificial intelligence', 'biotechnology', 'nanotechnology', 
                           'renewable energy', 'medical devices', 'semiconductors']
        
        field_lower = self.technology_field.lower()
        if any(hvf in field_lower for hvf in high_value_fields):
            value_score *= 1.2  # 20% premium for high-value technology fields
        
        return round(min(value_score, 200), 2)  # Cap at 200 for exceptional patents
    
    def get_remaining_life_percentage(self):
        """
        CORE LOGIC: Calculate remaining patent life as percentage of total term
        
        HOW IT WORKS:
        1. Determines patent term based on type and jurisdiction (usually 20 years)
        2. Calculates years elapsed since filing date
        3. Computes remaining years until expiry
        4. Returns percentage of original term remaining
        5. Handles special cases like expired or abandoned patents
        
        BUSINESS PURPOSE:
        - Critical for maintenance fee decisions
        - Important for licensing valuation and terms
        - Helps prioritize patent portfolio management
        - Supports strategic commercialization timing
        """
        if self.status in ['EXPIRED', 'ABANDONED', 'REJECTED']:
            return 0
        
        if not self.filing_date:
            return 100  # Unknown, assume full term
        
        # Standard patent terms by type and jurisdiction
        patent_terms = {
            'UTILITY': 20,      # 20 years from filing
            'DESIGN': 15,       # 15 years (varies by jurisdiction)
            'PLANT': 20,        # 20 years from filing
            'PROVISIONAL': 1    # 1 year for provisional
        }
        
        # Jurisdiction adjustments
        if self.jurisdiction == 'US' and self.patent_type == 'DESIGN':
            term_years = 15
        else:
            term_years = patent_terms.get(self.patent_type, 20)
        
        # Calculate years elapsed
        today = timezone.now().date()
        years_elapsed = (today - self.filing_date).days / 365.25
        
        # Calculate remaining percentage
        remaining_years = max(term_years - years_elapsed, 0)
        remaining_percentage = (remaining_years / term_years) * 100
        
        return round(remaining_percentage, 1)
    
    def calculate_maintenance_schedule(self):
        """
        CORE LOGIC: Generate maintenance fee schedule and calculate upcoming costs
        
        HOW IT WORKS:
        1. Determines maintenance fee schedule based on jurisdiction and patent type
        2. Calculates due dates for upcoming maintenance fees
        3. Estimates fee amounts based on current fee schedules
        4. Considers small entity or micro entity discounts if applicable
        5. Generates prioritized payment schedule with deadlines
        
        BUSINESS PURPOSE:
        - Prevents accidental patent abandonment due to missed fees
        - Enables budget planning for IP maintenance costs
        - Supports portfolio management decisions
        - Automates administrative tracking of patent obligations
        """
        if self.status not in ['GRANTED', 'PUBLISHED'] or not self.filing_date:
            return {'schedule': [], 'total_cost': 0}
        
        maintenance_schedule = []
        
        # US Patent maintenance fee schedule (example)
        if self.jurisdiction == 'US' and self.patent_type == 'UTILITY':
            fee_years = [3.5, 7.5, 11.5]  # Years from grant
            fee_amounts = [1600, 3600, 7700]  # USD amounts (subject to change)
            
            if self.grant_date:
                for i, years in enumerate(fee_years):
                    due_date = self.grant_date + timedelta(days=int(years * 365.25))
                    
                    # Only include future fees
                    if due_date > timezone.now().date():
                        maintenance_schedule.append({
                            'due_date': due_date,
                            'fee_amount': fee_amounts[i],
                            'currency': 'USD',
                            'description': f'{years} year maintenance fee',
                            'grace_period_end': due_date + timedelta(days=180)
                        })
        
        # Indian Patent maintenance fee schedule (example)
        elif self.jurisdiction == 'IN':
            # Annual fees from year 2 onwards
            base_fee = 8000  # INR (varies by entity size)
            
            for year in range(2, 21):  # Years 2-20
                fee_due_date = self.filing_date.replace(year=self.filing_date.year + year)
                
                if fee_due_date > timezone.now().date():
                    # Progressive fee structure
                    if year <= 5:
                        fee_amount = base_fee
                    elif year <= 10:
                        fee_amount = base_fee * 2
                    else:
                        fee_amount = base_fee * 4
                    
                    maintenance_schedule.append({
                        'due_date': fee_due_date,
                        'fee_amount': fee_amount,
                        'currency': 'INR',
                        'description': f'Year {year} annual fee',
                        'grace_period_end': fee_due_date + timedelta(days=365)
                    })
        
        # Calculate total upcoming costs
        total_cost = sum(fee['fee_amount'] for fee in maintenance_schedule)
        
        return {
            'schedule': maintenance_schedule,
            'total_cost': total_cost,
            'next_due_date': maintenance_schedule[0]['due_date'] if maintenance_schedule else None,
            'currency': maintenance_schedule[0]['currency'] if maintenance_schedule else 'USD'
        }
    
    def assess_commercialization_potential(self):
        """
        CORE LOGIC: Evaluate commercial viability and market potential of the patent
        
        HOW IT WORKS:
        1. Analyzes technology field market size and growth trends
        2. Evaluates competitive landscape and patent strength
        3. Assesses implementation barriers and costs
        4. Considers licensing opportunities and industry interest
        5. Scores various factors to determine commercialization readiness
        
        BUSINESS PURPOSE:
        - Guides technology transfer and licensing strategies
        - Supports startup formation and spin-off decisions
        - Helps prioritize commercialization investments
        - Informs patent prosecution and portfolio decisions
        """
        assessment = {
            'overall_score': 0,
            'market_potential': 'Medium',
            'technical_feasibility': 'Medium',
            'competitive_advantage': 'Medium',
            'commercialization_readiness': 'Medium',
            'recommendations': []
        }
        
        score = 0
        
        # Technology field market assessment
        high_demand_fields = [
            'artificial intelligence', 'machine learning', 'biotechnology',
            'renewable energy', 'medical devices', 'autonomous vehicles',
            'cybersecurity', 'blockchain', 'quantum computing'
        ]
        
        field_lower = self.technology_field.lower()
        if any(field in field_lower for field in high_demand_fields):
            score += 25
            assessment['market_potential'] = 'High'
        else:
            score += 10
            assessment['market_potential'] = 'Medium'
        
        # Patent strength indicators
        if self.status == 'GRANTED':
            score += 20  # Granted patents have stronger position
        elif self.status in ['PUBLISHED', 'UNDER_EXAMINATION']:
            score += 10
        
        # Jurisdiction coverage
        if self.jurisdiction in ['US', 'EP', 'PCT']:
            score += 15  # Broad market coverage
            assessment['competitive_advantage'] = 'High'
        elif self.jurisdiction in ['CN', 'JP']:
            score += 10
        else:
            score += 5
        
        # Commercial activity indicators
        if self.total_earnings > 0:
            score += 20  # Already generating revenue
            assessment['commercialization_readiness'] = 'High'
        elif self.status == 'LICENSED':
            score += 15  # Licensed but may not be generating revenue yet
        
        # Patent age factor (newer patents often have better commercial potential)
        if self.filing_date:
            years_old = (timezone.now().date() - self.filing_date).days / 365.25
            if years_old < 2:
                score += 10  # Very recent, good potential
            elif years_old < 5:
                score += 5   # Recent, moderate potential
            # Older patents get no bonus
        
        # Generate recommendations based on assessment
        if score >= 70:
            assessment['recommendations'].append("High commercialization potential - actively pursue licensing")
            assessment['recommendations'].append("Consider startup formation or spin-off opportunity")
        elif score >= 50:
            assessment['recommendations'].append("Moderate potential - explore industry partnerships")
            assessment['recommendations'].append("Conduct market research to validate demand")
        else:
            assessment['recommendations'].append("Limited commercial potential - consider maintenance costs vs value")
            assessment['recommendations'].append("May be better suited for defensive purposes")
        
        assessment['overall_score'] = min(score, 100)
        
        # Set categorical assessments based on score
        if score >= 70:
            assessment['commercialization_readiness'] = 'High'
        elif score >= 40:
            assessment['commercialization_readiness'] = 'Medium'
        else:
            assessment['commercialization_readiness'] = 'Low'
        
        return assessment
    
    def calculate_roi_projection(self, projection_years=5):
        """
        CORE LOGIC: Calculate projected return on investment for patent commercialization
        
        HOW IT WORKS:
        1. Analyzes historical earnings trends if available
        2. Estimates market size and penetration potential
        3. Models different commercialization scenarios (licensing vs direct use)
        4. Factors in ongoing maintenance costs and legal expenses
        5. Calculates NPV and ROI projections over specified timeframe
        
        BUSINESS PURPOSE:
        - Supports investment decisions for patent commercialization
        - Helps justify maintenance costs vs abandonment
        - Guides licensing negotiation strategies
        - Informs technology transfer office priorities
        """
        # Calculate total investment to date
        total_investment = (self.filing_costs + self.maintenance_costs + 
                          self.attorney_costs)
        
        if total_investment == 0:
            total_investment = 50000  # Estimate if no cost data available
        
        # Historical earnings analysis
        historical_annual_earnings = float(self.total_earnings) / max(
            (timezone.now().date() - self.filing_date).days / 365.25, 1
        ) if self.filing_date else 0
        
        # Projection scenarios
        scenarios = {
            'conservative': {
                'annual_growth_rate': 0.05,  # 5% growth
                'market_penetration': 0.01,  # 1% market penetration
                'royalty_rate': 0.02        # 2% royalty
            },
            'optimistic': {
                'annual_growth_rate': 0.15,  # 15% growth
                'market_penetration': 0.05,  # 5% market penetration
                'royalty_rate': 0.05        # 5% royalty
            },
            'realistic': {
                'annual_growth_rate': 0.08,  # 8% growth
                'market_penetration': 0.02,  # 2% market penetration
                'royalty_rate': 0.03        # 3% royalty
            }
        }
        
        projections = {}
        
        for scenario_name, params in scenarios.items():
            projected_earnings = []
            annual_earnings = max(historical_annual_earnings, 10000)  # Minimum baseline
            
            for year in range(1, projection_years + 1):
                # Apply growth rate
                annual_earnings *= (1 + params['annual_growth_rate'])
                
                # Estimate remaining patent life impact
                remaining_life = self.get_remaining_life_percentage()
                life_factor = max(remaining_life / 100, 0.1)  # Minimum 10% value
                
                adjusted_earnings = annual_earnings * life_factor
                projected_earnings.append(adjusted_earnings)
            
            # Calculate NPV (assuming 10% discount rate)
            discount_rate = 0.10
            npv = sum(earnings / ((1 + discount_rate) ** year) 
                     for year, earnings in enumerate(projected_earnings, 1))
            
            # Calculate ROI
            roi_percentage = ((npv - total_investment) / total_investment) * 100
            
            projections[scenario_name] = {
                'total_projected_earnings': sum(projected_earnings),
                'npv': round(npv, 2),
                'roi_percentage': round(roi_percentage, 2),
                'payback_period_years': self._calculate_payback_period(
                    projected_earnings, total_investment
                ),
                'annual_projections': [round(e, 2) for e in projected_earnings]
            }
        
        return {
            'total_investment': total_investment,
            'scenarios': projections,
            'recommendation': self._get_investment_recommendation(projections)
        }
    
    def _calculate_payback_period(self, projected_earnings, initial_investment):
        """Helper method to calculate payback period"""
        cumulative_earnings = 0
        for year, earnings in enumerate(projected_earnings, 1):
            cumulative_earnings += earnings
            if cumulative_earnings >= initial_investment:
                return year
        return None  # Investment not recovered within projection period
    
    def _get_investment_recommendation(self, projections):
        """Helper method to generate investment recommendation"""
        realistic_roi = projections['realistic']['roi_percentage']
        
        if realistic_roi > 50:
            return "Excellent investment opportunity - high ROI potential"
        elif realistic_roi > 20:
            return "Good investment opportunity - positive ROI expected"
        elif realistic_roi > 0:
            return "Moderate opportunity - break-even or small positive returns"
        else:
            return "Poor investment outlook - consider abandonment to minimize losses"
```