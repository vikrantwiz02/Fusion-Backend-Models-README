# Complete Feeds Module System Documentation - Fusion IIIT

## System Overview
The Feeds Module is a comprehensive social networking and knowledge sharing platform within Fusion IIIT. It facilitates community interaction through a Q&A system, discussion forums, and content curation based on academic and campus interests. The system provides tag-based content organization, user profiles, role-based permissions, and administrative controls for community management.

---

## Database Models Analysis with Business Logic

### 1. AllTags Model
**Purpose**: Master registry for categorizing content with hierarchical tag and subtag system for organized content discovery.

**PostgreSQL Table**: `feeds_alltags`

**Fields Structure**:
```python
class AllTags(models.Model):
    id = models.AutoField(primary_key=True)                    # Primary key
    tag = models.CharField(max_length=100, default='CSE')      # Main category (CSE, ECE, Mechanical, etc.)
    subtag = models.CharField(max_length=100, unique=True)     # Specific subcategory (Web-Development, AI, etc.)
    
    # Business logic
    def __str__(self):
        return '{} - {}'.format(self.tag, self.subtag)
```

**Key Business Methods**:
- **Content Categorization**: Hierarchical organization from broad disciplines to specific topics
- **Unique Subtag Constraint**: Prevents duplicate specific topics across different main categories
- **Choice-Based Validation**: Enforces consistent terminology through predefined choices

**Core Business Logic**:
- **18 Main Tags**: Academic departments (CSE, ECE, Mechanical), club categories (Technical, Cultural, Sports), campus areas (IIITDMJ-Campus, Academics, Entertainment)
- **100+ Subtags**: Granular specializations within each main category
- **Cross-Reference System**: Enables filtering and recommendation algorithms

---

### 2. AskaQuestion Model
**Purpose**: Central model for user-generated questions with comprehensive interaction tracking and content management.

**PostgreSQL Table**: `feeds_askaquestion`

**Fields Structure**:
```python
class AskaQuestion(models.Model):
    id = models.AutoField(primary_key=True)
    user = models.ForeignKey(User, on_delete=models.CASCADE)           # Question author
    subject = models.CharField(max_length=100)                         # Question title
    description = models.CharField(max_length=500, default="")         # Question details
    file = models.FileField(upload_to='feeds/files', blank=True)       # Attachment support
    uploaded_at = models.DateTimeField(default=timezone.now)           # Timestamp
    
    # Many-to-Many relationships for interactions
    select_tag = models.ManyToManyField(AllTags)                       # Categorization
    likes = models.ManyToManyField(User, related_name='likes')         # User likes
    dislikes = models.ManyToManyField(User, related_name='dislikes')   # User dislikes
    requests = models.ManyToManyField(User, related_name='requests')   # Answer requests
    
    # Status and permission fields
    anonymous_ask = models.BooleanField(default=False)                 # Anonymous posting
    can_delete = models.BooleanField(default=False)                    # Deletion permission
    can_update = models.BooleanField(default=False)                    # Edit permission
    
    # Computed metrics
    total_likes = models.IntegerField(default=0)
    total_dislikes = models.IntegerField(default=0)
    is_liked = models.BooleanField(default=False)
    is_requested = models.BooleanField(default=False)
    request = models.IntegerField(default=0)
```

**Core Business Methods with Enhanced Logic**:
```python
def total_likes(self):
    """
    CORE LOGIC: Calculate total likes from many-to-many relationship
    
    HOW IT WORKS:
    1. Accesses the many-to-many 'likes' relationship
    2. Counts all User objects linked to this question
    3. Returns integer count for display and ranking algorithms
    
    BUSINESS PURPOSE:
    - Enables popularity ranking of questions
    - Supports recommendation algorithms
    - Provides engagement metrics for users and admins
    """
    return self.likes.count()

def total_dislikes(self):
    """
    CORE LOGIC: Calculate total dislikes for content quality assessment
    
    HOW IT WORKS:
    1. Counts users who disliked the question
    2. Used in conjunction with likes for net score calculation
    3. Helps identify controversial or low-quality content
    
    BUSINESS PURPOSE:
    - Content quality control and filtering
    - Identifies questions needing moderation
    - Balances like counts for accurate popularity metrics
    """
    return self.dislikes.count()

def total_requests(self):
    """
    CORE LOGIC: Track how many users want this question answered
    
    HOW IT WORKS:
    1. Counts users who clicked "request answer" button
    2. Creates urgency indicator for unanswered questions
    3. Helps prioritize which questions need expert attention
    
    BUSINESS PURPOSE:
    - Identifies high-demand unanswered questions
    - Drives expert engagement and response
    - Creates community-driven priority system
    """
    return self.requests.count()

def get_markdown(self):
    """
    CORE LOGIC: Convert plain text to rich HTML for enhanced readability
    
    HOW IT WORKS:
    1. Takes the question description text
    2. Processes markdown syntax (bold, italic, links, code blocks)
    3. Converts to safe HTML using mark_safe() to prevent XSS
    4. Returns formatted HTML for template rendering
    
    BUSINESS PURPOSE:
    - Enables rich text formatting in questions
    - Supports technical content with code blocks
    - Improves readability and user experience
    - Maintains security through safe HTML rendering
    """
    content = self.description
    markdown_text = markdown(content)
    return mark_safe(markdown_text)

def calculate_engagement_score(self):
    """
    CORE LOGIC: Comprehensive engagement scoring algorithm
    
    HOW IT WORKS:
    1. Calculates net votes (likes - dislikes)
    2. Adds weighted scores for answers, comments, and requests
    3. Factors in time decay for recent vs old content
    4. Normalizes score for ranking and recommendation
    
    BUSINESS PURPOSE:
    - Ranks questions for feed display order
    - Identifies trending and popular content
    - Supports recommendation engine
    - Helps surface high-quality discussions
    """
    net_votes = self.total_likes() - self.total_dislikes()
    answer_count = self.answeraquestion_set.count()
    comment_count = self.comments_set.count()
    request_weight = self.total_requests() * 0.5
    
    # Time decay factor (newer content gets boost)
    days_old = (timezone.now() - self.uploaded_at).days
    time_factor = max(0.1, 1 - (days_old * 0.05))  # 5% decay per day, min 10%
    
    base_score = (net_votes * 2) + (answer_count * 3) + comment_count + request_weight
    final_score = base_score * time_factor
    
    return round(final_score, 2)

def get_answer_quality_metrics(self):
    """
    CORE LOGIC: Analyze answer quality and response satisfaction
    
    HOW IT WORKS:
    1. Examines all answers to this question
    2. Calculates average answer vote ratios
    3. Identifies best answers and response patterns
    4. Determines if question was adequately answered
    
    BUSINESS PURPOSE:
    - Measures question resolution success
    - Identifies knowledge gaps requiring expert input
    - Supports answer recommendation algorithms
    - Helps improve community response quality
    """
    answers = self.answeraquestion_set.all()
    if not answers:
        return {
            'has_answers': False,
            'best_answer_score': 0,
            'average_answer_quality': 0,
            'needs_expert_attention': True
        }
    
    answer_scores = []
    for answer in answers:
        net_score = answer.total_votes()
        answer_scores.append(net_score)
    
    return {
        'has_answers': True,
        'answer_count': len(answers),
        'best_answer_score': max(answer_scores) if answer_scores else 0,
        'average_answer_quality': sum(answer_scores) / len(answer_scores),
        'needs_expert_attention': max(answer_scores) < 2 if answer_scores else True
    }

def check_content_violations(self):
    """
    CORE LOGIC: Content moderation and policy violation detection
    
    HOW IT WORKS:
    1. Checks report count and severity
    2. Analyzes content patterns for policy violations
    3. Determines if content needs administrative review
    4. Flags potential spam or inappropriate content
    
    BUSINESS PURPOSE:
    - Automated content moderation pipeline
    - Protects community from harmful content
    - Reduces administrative overhead
    - Maintains platform quality standards
    """
    report_count = self.report_set.count()
    dislike_ratio = self.total_dislikes() / max(1, self.total_likes() + self.total_dislikes())
    
    violations = {
        'high_reports': report_count >= 3,
        'poor_reception': dislike_ratio > 0.7 and (self.total_likes() + self.total_dislikes()) > 5,
        'needs_review': report_count > 0,
        'auto_hide_threshold': report_count >= 5
    }
    
    return violations
```

**Key Business Logic**:
- **Multi-Tag Support**: Questions can belong to multiple categories simultaneously
- **Interaction Tracking**: Real-time tracking of likes, dislikes, and answer requests
- **File Attachment**: Support for multimedia content in questions
- **Anonymous Posting**: Privacy option for sensitive questions
- **Markdown Processing**: Rich text formatting for enhanced readability

---

### 3. AnsweraQuestion Model
**Purpose**: Response system for questions with voting mechanisms and content quality assessment.

**PostgreSQL Table**: `feeds_answeraquestion`

**Fields Structure**:
```python
class AnsweraQuestion(models.Model):
    id = models.AutoField(primary_key=True)
    user = models.ForeignKey(User, on_delete=models.CASCADE)           # Answer author
    content = models.TextField(max_length=1000)                        # Answer content
    question = models.ForeignKey(AskaQuestion, on_delete=models.CASCADE) # Parent question
    uploaded_at = models.DateTimeField(default=timezone.now)           # Timestamp
    
    # Answer interaction tracking
    answers = models.ManyToManyField(User, related_name='answers')     # Answer acknowledgments
    likes = models.ManyToManyField(User, related_name='answer_likes')  # Answer likes
    dislikes = models.ManyToManyField(User, related_name='answer_dislikes') # Answer dislikes
    
    # Status fields
    total_answers = models.IntegerField(default=0)
    is_liked = models.BooleanField(default=False)
```

**Core Business Methods with Enhanced Logic**:
```python
def total_likes(self):
    """
    CORE LOGIC: Calculate answer engagement through like tracking
    
    HOW IT WORKS:
    1. Counts users who liked this specific answer
    2. Separate from question likes - tracks answer quality
    3. Used for ranking answers within a question thread
    
    BUSINESS PURPOSE:
    - Identifies most helpful answers
    - Enables answer sorting by community approval
    - Supports expert recognition system
    """
    return self.likes.count()

def total_votes(self):
    """
    CORE LOGIC: Net voting score for answer quality ranking
    
    HOW IT WORKS:
    1. Calculates likes minus dislikes for net approval
    2. Positive scores indicate helpful answers
    3. Negative scores flag potentially incorrect answers
    4. Zero scores indicate neutral or controversial responses
    
    BUSINESS PURPOSE:
    - Primary ranking metric for answer ordering
    - Quality control for community-driven validation
    - Helps users quickly identify best responses
    - Supports reputation systems for answer authors
    """
    return self.likes.count() - self.dislikes.count()

def total_dislikes(self):
    """
    CORE LOGIC: Track negative feedback for answer quality control
    
    HOW IT WORKS:
    1. Counts users who found answer unhelpful or incorrect
    2. Balances positive feedback for accurate quality assessment
    3. Triggers review for heavily disliked answers
    
    BUSINESS PURPOSE:
    - Content quality control mechanism
    - Identifies potentially misleading information
    - Community-driven fact-checking system
    """
    return self.dislikes.count()

def get_markdown(self):
    """
    CORE LOGIC: Rich text formatting for technical answer content
    
    HOW IT WORKS:
    1. Processes markdown syntax in answer content
    2. Supports code blocks, links, formatting for technical answers
    3. Converts to safe HTML for display
    4. Maintains formatting while preventing XSS attacks
    
    BUSINESS PURPOSE:
    - Enables detailed technical explanations
    - Supports code examples and formatted content
    - Improves answer readability and comprehension
    """
    content = self.content
    markdown_text = markdown(content)
    return mark_safe(markdown_text)

def calculate_answer_authority_score(self):
    """
    CORE LOGIC: Comprehensive scoring for answer ranking and credibility
    
    HOW IT WORKS:
    1. Combines vote ratio, response time, and author reputation
    2. Factors in answer length and technical depth
    3. Considers community engagement and follow-up questions
    4. Normalizes score for cross-question comparison
    
    BUSINESS PURPOSE:
    - Ranks answers for optimal display order
    - Identifies subject matter experts
    - Supports reputation building for quality contributors
    - Improves knowledge discovery efficiency
    """
    net_votes = self.total_votes()
    vote_ratio = net_votes / max(1, abs(net_votes) + 1)  # Normalize between -1 and 1
    
    # Response time factor (faster responses get slight boost)
    response_time_hours = (self.uploaded_at - self.question.uploaded_at).total_seconds() / 3600
    time_factor = max(0.8, 2 - (response_time_hours * 0.1))  # Diminishing returns
    
    # Content quality indicators
    content_length_score = min(1.0, len(self.content) / 500)  # Optimal around 500 chars
    
    # Author reputation (based on other answers' performance)
    author_reputation = self._calculate_author_reputation()
    
    authority_score = (
        (vote_ratio * 40) +           # Primary factor: community approval
        (time_factor * 20) +          # Secondary: responsiveness  
        (content_length_score * 20) + # Content depth
        (author_reputation * 20)      # Historical performance
    )
    
    return round(authority_score, 2)

def _calculate_author_reputation(self):
    """Helper method to calculate author's overall reputation"""
    user_answers = AnsweraQuestion.objects.filter(user=self.user)
    if not user_answers.exists():
        return 0.5  # Neutral starting point
    
    total_score = sum(answer.total_votes() for answer in user_answers)
    answer_count = user_answers.count()
    
    average_score = total_score / answer_count
    # Normalize to 0-1 scale
    reputation = max(0, min(1, (average_score + 5) / 10))
    
    return reputation

def assess_answer_completeness(self):
    """
    CORE LOGIC: Evaluate if answer adequately addresses the question
    
    HOW IT WORKS:
    1. Analyzes content length and structure
    2. Checks for code examples, explanations, and references
    3. Measures community satisfaction through voting patterns
    4. Identifies areas where answer could be improved
    
    BUSINESS PURPOSE:
    - Quality assurance for knowledge base
    - Identifies questions needing additional responses
    - Guides answer improvement suggestions
    """
    assessment = {
        'content_length': len(self.content),
        'has_code_examples': '```' in self.content or '`' in self.content,
        'has_external_links': 'http' in self.content,
        'community_satisfaction': self.total_votes() > 0,
        'needs_improvement': self.total_votes() < 0 or self.total_dislikes() > self.total_likes()
    }
    
    # Completeness score based on multiple factors
    score = 0
    if assessment['content_length'] > 100: score += 25
    if assessment['has_code_examples']: score += 25  
    if assessment['has_external_links']: score += 15
    if assessment['community_satisfaction']: score += 35
    
    assessment['completeness_score'] = score
    assessment['is_comprehensive'] = score >= 70
    
    return assessment
```

**Key Business Logic**:
- **Quality Scoring**: Net voting system helps identify helpful answers
- **Hierarchical Response**: Answers linked to specific questions
- **Rich Text Support**: Markdown formatting for technical explanations
- **Community Validation**: User voting determines answer quality

---

### 4. Comments Model
**Purpose**: Threaded discussion system for questions with engagement tracking.

**PostgreSQL Table**: `feeds_comments`

**Fields Structure**:
```python
class Comments(models.Model):
    id = models.AutoField(primary_key=True)
    user = models.ForeignKey(User, on_delete=models.CASCADE)           # Comment author
    question = models.ForeignKey(AskaQuestion, on_delete=models.CASCADE) # Parent question
    comment_text = models.CharField(max_length=5000)                   # Comment content
    commented_at = models.DateTimeField(default=timezone.now)          # Timestamp
    
    # Comment interaction tracking
    likes_comment = models.ManyToManyField(User, related_name='likes_comment')
    total_likes_comment = models.IntegerField(default=0)
    is_liked = models.BooleanField(default=False)
```

**Core Business Methods with Enhanced Logic**:
```python
def total_likes_comment(self):
    """
    CORE LOGIC: Track comment engagement and approval
    
    HOW IT WORKS:
    1. Counts users who liked this specific comment
    2. Provides feedback on comment helpfulness
    3. Enables ranking of comments within discussion threads
    
    BUSINESS PURPOSE:
    - Identifies most valuable comments for prioritized display
    - Encourages quality participation in discussions
    - Supports threaded conversation management
    """
    return self.likes_comment.count()

def calculate_comment_relevance_score(self):
    """
    CORE LOGIC: Determine comment importance and display priority
    
    HOW IT WORKS:
    1. Combines like count with timing and content analysis
    2. Factors in response patterns and user engagement
    3. Considers comment position in conversation flow
    4. Normalizes for fair comparison across different questions
    
    BUSINESS PURPOSE:
    - Optimizes comment display order for better UX
    - Surfaces most helpful clarifications and follow-ups
    - Reduces noise in discussion threads
    - Supports intelligent conversation threading
    """
    likes = self.total_likes_comment()
    
    # Time factor - newer comments get slight boost
    time_since_post = timezone.now() - self.commented_at
    hours_old = time_since_post.total_seconds() / 3600
    time_factor = max(0.5, 1 - (hours_old * 0.01))  # 1% decay per hour
    
    # Content length factor - balanced scoring
    content_score = min(1.0, len(self.comment_text) / 200)  # Optimal around 200 chars
    
    # Position in thread (earlier comments get slight boost)
    question_comments = Comments.objects.filter(question=self.question).order_by('commented_at')
    position = list(question_comments).index(self) + 1
    position_factor = max(0.7, 1 - (position * 0.05))  # Slight preference for early comments
    
    relevance_score = (likes * 3) + (time_factor * 2) + content_score + position_factor
    
    return round(relevance_score, 2)

def analyze_comment_engagement_pattern(self):
    """
    CORE LOGIC: Understand how comment contributes to overall discussion
    
    HOW IT WORKS:
    1. Analyzes reply patterns and user responses
    2. Measures discussion continuation after this comment
    3. Identifies conversation starters vs conversation enders
    4. Tracks cross-referencing and mention patterns
    
    BUSINESS PURPOSE:
    - Identifies valuable discussion catalysts
    - Optimizes conversation flow and threading
    - Supports community engagement analytics
    - Guides discussion moderation strategies
    """
    replies = Reply.objects.filter(comment=self)
    
    engagement_metrics = {
        'reply_count': replies.count(),
        'sparked_discussion': replies.count() > 2,
        'avg_reply_length': sum(len(r.content) for r in replies) / max(1, replies.count()),
        'engagement_span_hours': 0,
        'discussion_depth': self._calculate_discussion_depth()
    }
    
    if replies.exists():
        first_reply = replies.earliest('replied_at')
        last_reply = replies.latest('replied_at')
        span = last_reply.replied_at - first_reply.replied_at
        engagement_metrics['engagement_span_hours'] = span.total_seconds() / 3600
    
    # Overall engagement classification
    if engagement_metrics['reply_count'] > 3 and engagement_metrics['engagement_span_hours'] > 2:
        engagement_metrics['type'] = 'discussion_catalyst'
    elif engagement_metrics['reply_count'] == 0:
        engagement_metrics['type'] = 'standalone_insight'
    else:
        engagement_metrics['type'] = 'clarification_exchange'
    
    return engagement_metrics

def _calculate_discussion_depth(self):
    """Helper method to measure conversation depth triggered by comment"""
    replies = Reply.objects.filter(comment=self)
    if not replies.exists():
        return 0
    
    # Simple depth calculation based on reply patterns
    total_chars = sum(len(reply.content) for reply in replies)
    avg_depth = total_chars / replies.count() if replies.count() > 0 else 0
    
    return min(5, avg_depth / 100)  # Normalize to 0-5 scale
```

**Key Business Logic**:
- **Extended Discussion**: Comments provide clarification and follow-up discussion
- **Engagement Metrics**: Like tracking for comment quality assessment
- **Large Content Support**: 5000 character limit for detailed explanations

---

### 5. Reply Model
**Purpose**: Nested conversation system enabling replies to comments for detailed discussions.

**PostgreSQL Table**: `feeds_reply`

**Fields Structure**:
```python
class Reply(models.Model):
    id = models.AutoField(primary_key=True)
    user = models.ForeignKey(User, on_delete=models.CASCADE)           # Reply author
    comment = models.ForeignKey(Comments, on_delete=models.CASCADE)    # Parent comment
    msg = models.CharField(max_length=1000)                            # Reply message
    content = models.CharField(max_length=5000, default="")            # Extended content
    replied_at = models.DateTimeField(default=timezone.now)            # Timestamp
    
    # Reply interaction tracking
    replies = models.ManyToManyField(User, related_name='replies')
    total_replies = models.IntegerField(default=0)
```

**Core Business Methods**:
```python
def total_replies(self):
    """Count total replies"""
    return self.replies.count()
```

**Key Business Logic**:
- **Three-Level Hierarchy**: Question → Comment → Reply structure
- **Dual Content Fields**: Both msg and content fields for flexible use
- **Conversation Threading**: Maintains discussion context and flow

---

### 6. report Model
**Purpose**: Content moderation system for community-driven quality control.

**PostgreSQL Table**: `feeds_report`

**Fields Structure**:
```python
class report(models.Model):
    id = models.AutoField(primary_key=True)
    user = models.ForeignKey(User, on_delete=models.CASCADE)           # Reporter
    question = models.ForeignKey(AskaQuestion, on_delete=models.CASCADE) # Reported question
    report_msg = models.CharField(max_length=1000, default="")         # Report reason
```

**Key Business Logic**:
- **Community Moderation**: Users can flag inappropriate content
- **Report Tracking**: Links reporter to specific content and reason
- **Administrative Review**: Enables moderator intervention system

---

### 7. hidden Model
**Purpose**: User-level content filtering for personalized feed curation.

**PostgreSQL Table**: `feeds_hidden`

**Fields Structure**:
```python
class hidden(models.Model):
    id = models.AutoField(primary_key=True)
    user = models.ForeignKey(User, on_delete=models.CASCADE)           # User hiding content
    question = models.ForeignKey(AskaQuestion, on_delete=models.CASCADE) # Hidden question
    
    class Meta:
        unique_together = ('user', 'question')                         # Prevent duplicate hides
```

**Key Business Logic**:
- **Personal Curation**: Users can hide unwanted content from their feeds
- **Unique Constraint**: Prevents multiple hide records for same user-question pair
- **Non-Destructive**: Content remains available to other users

---

### 8. tags Model
**Purpose**: User interest tracking for personalized content recommendation and feed filtering.

**PostgreSQL Table**: `feeds_tags`

**Fields Structure**:
```python
class tags(models.Model):
    id = models.AutoField(primary_key=True)
    user = models.ForeignKey(User, on_delete=models.CASCADE)           # User
    my_tag = models.CharField(max_length=100, choices=TAG_LIST)        # Main interest category
    my_subtag = models.ForeignKey(AllTags, on_delete=models.CASCADE)   # Specific interest
    
    class Meta:
        unique_together = ('user', 'my_subtag')                        # Prevent duplicate interests
```

**Core Business Methods**:
```python
def __str__(self):
    return '%s is interested in ----> %s - %s' % (self.user, self.my_tag, self.my_subtag.subtag)
```

**Key Business Logic**:
- **Interest Profiling**: Tracks user preferences for content recommendation
- **Feed Personalization**: Enables customized content delivery
- **Duplicate Prevention**: Ensures clean user interest profiles

---

### 9. Profile Model
**Purpose**: User profile management with bio, images, and engagement metrics.

**PostgreSQL Table**: `feeds_profile`

**Fields Structure**:
```python
class Profile(models.Model):
    id = models.AutoField(primary_key=True)
    user = models.ForeignKey(User, on_delete=models.CASCADE)           # Profile owner
    bio = models.CharField(max_length=250, blank=True)                 # User biography
    profile_picture = models.ImageField(upload_to='feeds/profile_pictures', blank=True) # Avatar
    profile_view = models.IntegerField(default=0)                      # View counter
```

**Core Business Methods**:
```python
def __str__(self):
    return '%s\'s Bio is  ----> %s' % (self.user, self.bio)
```

**Key Business Logic**:
- **Identity Management**: Visual and textual user representation
- **Engagement Tracking**: Profile view metrics for popularity assessment
- **Media Support**: Image handling for enhanced user experience

---

### 10. Roles Model
**Purpose**: Role-based access control for administrative functions and special permissions.

**PostgreSQL Table**: `feeds_roles`

**Fields Structure**:
```python
class Roles(models.Model):
    id = models.AutoField(primary_key=True)
    user = models.ForeignKey(User, on_delete=models.CASCADE)           # Role holder
    role = models.CharField(max_length=100)                            # Role description
    active = models.BooleanField(default=True)                         # Role status
```

**Core Business Methods**:
```python
def __str__(self):
    return '%s is assigned %s role' % (self.user, self.role)
```

**Key Business Logic**:
- **Permission System**: Defines user capabilities within the system
- **Administrative Control**: Enables special privileges for moderators
- **Status Management**: Active/inactive role switching

---

### 11. QuestionAccessControl Model
**Purpose**: Granular permission system for individual questions with role-based restrictions.

**PostgreSQL Table**: `feeds_questionaccesscontrol`

**Fields Structure**:
```python
class QuestionAccessControl(models.Model):
    id = models.AutoField(primary_key=True)
    question = models.ForeignKey(AskaQuestion, related_name='question_list', on_delete=models.CASCADE)
    canVote = models.BooleanField()                                     # Voting permission
    canAnswer = models.BooleanField()                                   # Answer permission
    canComment = models.BooleanField()                                  # Comment permission
    posted_by = models.ForeignKey(Roles, on_delete=models.CASCADE)     # Role that posted
    created_at = models.DateTimeField(default=timezone.now)            # Creation timestamp
```

**Core Business Methods**:
```python
def __str__(self):
    return "question number " + str(self.question.id)
```

**Key Business Logic**:
- **Granular Permissions**: Individual control over voting, answering, and commenting
- **Role-Based Posting**: Links posting permissions to user roles
- **Special Content Management**: Enables restricted access for sensitive topics

---

## System Integration and Workflow

### Core View Functions

#### 1. feeds() View - Main Feed Engine
**Purpose**: Central feed rendering with intelligent content curation and user interaction management

**Comprehensive Business Logic**:
```python
@login_required
def feeds(request):
    """
    CORE LOGIC: Advanced feed curation and content delivery system
    
    HOW IT WORKS:
    1. CONTENT LOADING: Fetches questions with prefetch optimization for related data
    2. PERSONALIZATION: Filters content based on user interests and hidden items
    3. PAGINATION: Implements efficient pagination with 4 questions per page
    4. SEARCH ENGINE: Full-text search across subject and description fields
    5. TAG FILTERING: Dynamic filtering based on user-selected tags
    6. INTERACTION PROCESSING: Handles likes, dislikes, requests in real-time
    7. CONTENT CREATION: Complete question posting with file upload support
    8. PERMISSION CONTROL: Role-based content restrictions and special access
    
    BUSINESS PURPOSE:
    - Delivers personalized content experience
    - Manages community engagement and interactions
    - Enables knowledge discovery and sharing
    - Maintains content quality through moderation
    """
    
    # 1. INITIAL QUERY OPTIMIZATION
    query = AskaQuestion.objects.prefetch_related(
        'select_tag', 'likes', 'dislikes', 'requests'
    ).order_by('-uploaded_at')
    
    # 2. PAGINATION SETUP
    paginator = Paginator(query, PAGE_SIZE)  # 4 questions per page
    total_page = math.ceil(query.count() / PAGE_SIZE)
    current_page = int(request.GET.get("page_number", 1))
    
    # 3. SEARCH FUNCTIONALITY
    if request.GET.get("search") and request.GET.get('keyword'):
        search_term = request.GET.get('keyword')
        questions = AskaQuestion.objects.prefetch_related(
            'select_tag', 'likes', 'dislikes', 'requests'
        ).all()
        
        # Advanced search across multiple fields
        search_results = questions.filter(
            Q(subject__icontains=search_term) | 
            Q(description__icontains=search_term)
        ).order_by('-uploaded_at')
        
        query = search_results
        paginator = Paginator(query, PAGE_SIZE)
        total_page = math.ceil(query.count() / PAGE_SIZE)
    
    # 4. CONTENT CREATION PROCESSING
    if request.method == 'POST':
        if request.POST.get('add_qus'):
            # Create new question with comprehensive data processing
            question = AskaQuestion.objects.create(user=request.user)
            question.subject = request.POST.get('subject')
            question.description = request.POST.get('content')
            
            # File upload handling
            if request.FILES:
                question.file = request.FILES['file']
            
            # Tag assignment processing
            tag_string = request.POST.get('Add_Tag')[8:]  # Remove prefix
            tag_ids = [int(c) for c in tag_string.split(",")]
            
            for tag_id in tag_ids:
                tag_obj = AllTags.objects.get(id=tag_id)
                question.select_tag.add(tag_obj)
            
            # Anonymous posting option
            question.anonymous_ask = bool(request.POST.get('anonymous'))
            question.save()
            
            # Role-based access control setup
            role_check = Roles.objects.filter(user=request.user)
            if role_check.exists():
                access_control = QuestionAccessControl.objects.create(
                    question=question,
                    canVote=not request.POST.get("RestrictVote"),
                    canAnswer=not request.POST.get("RestrictAnswer"), 
                    canComment=not request.POST.get("RestrictComment"),
                    posted_by=role_check[0]
                )
                return redirect("/feeds/admin")
    
    # 5. USER PERSONALIZATION DATA
    user_tags = tags.objects.filter(user=request.user)
    all_available_tags = AllTags.objects.values('tag').distinct()
    
    # 6. CONTENT PROCESSING FOR DISPLAY
    hidden_questions = hidden.objects.filter(user=request.user)
    processed_questions = []
    
    try:
        current_page_questions = paginator.page(current_page)
    except:
        current_page_questions = []
    
    for question in current_page_questions:
        # Calculate user-specific interaction states
        user_liked = question.likes.filter(id=request.user.id).exists()
        user_disliked = question.dislikes.filter(id=request.user.id).exists()
        user_hidden = hidden_questions.filter(question=question).exists()
        
        # Check for special access controls
        access_controls = QuestionAccessControl.objects.filter(question=question)
        is_special = access_controls.exists()
        
        # Get user profile for display
        user_profile = Profile.objects.filter(user=question.user)
        
        question_data = {
            'question': question,
            'user_liked': user_liked,
            'user_disliked': user_disliked,
            'user_hidden': user_hidden,
            'is_special': is_special,
            'access_controls': access_controls,
            'user_profile': user_profile,
            'engagement_score': question.calculate_engagement_score(),
            'answer_metrics': question.get_answer_quality_metrics()
        }
        processed_questions.append(question_data)
    
    # 7. CONTEXT PREPARATION
    context = {
        'questions': processed_questions,
        'user_tags': user_tags,
        'available_tags': all_available_tags,
        'current_page': current_page,
        'total_pages': total_page,
        'has_previous': current_page > 1,
        'has_next': current_page < total_page,
        'search_keyword': request.GET.get('keyword', ''),
    }
    
    return render(request, 'feeds/feeds_main.html', context)
```

#### 2. Question Interaction System - Voting Engine
**Purpose**: Real-time voting system with toggle functionality and engagement tracking

**Enhanced Business Logic**:
```python
def upvoteQuestion(request, id):
    """
    CORE LOGIC: Intelligent voting system with toggle and validation
    
    HOW IT WORKS:
    1. VALIDATION: Checks user authentication and question existence
    2. TOGGLE LOGIC: Handles like/unlike and like/dislike switching
    3. STATE MANAGEMENT: Updates many-to-many relationships atomically
    4. CONFLICT RESOLUTION: Automatically handles dislike removal when liking
    5. REAL-TIME UPDATE: Returns JSON for immediate UI updates
    
    BUSINESS PURPOSE:
    - Provides instant feedback on content quality
    - Enables community-driven content curation
    - Supports recommendation algorithms through engagement data
    - Maintains data consistency across user interactions
    """
    question = get_object_or_404(AskaQuestion, id=id)
    user = request.user
    
    # Check current user state
    currently_liked = question.likes.filter(id=user.id).exists()
    currently_disliked = question.dislikes.filter(id=user.id).exists()
    
    if currently_liked:
        # User already liked - toggle off
        question.likes.remove(user)
        question.is_liked = False
        action = 'unliked'
    else:
        # User wants to like
        question.likes.add(user)
        question.is_liked = True
        action = 'liked'
        
        # Remove dislike if it exists (mutual exclusivity)
        if currently_disliked:
            question.dislikes.remove(user)
    
    question.save()
    
    # Calculate updated metrics
    response_data = {
        'action': action,
        'total_likes': question.total_likes(),
        'total_dislikes': question.total_dislikes(),
        'net_score': question.total_likes() - question.total_dislikes(),
        'user_liked': question.likes.filter(id=user.id).exists(),
        'user_disliked': question.dislikes.filter(id=user.id).exists()
    }
    
    if request.is_ajax():
        return JsonResponse(response_data)
    return redirect('feeds:feeds')

def Request(request):
    """
    CORE LOGIC: Answer request system for expert engagement
    
    HOW IT WORKS:
    1. Identifies high-value unanswered questions
    2. Creates urgency signals for expert users
    3. Tracks community demand for specific answers
    4. Enables notification systems for subject experts
    
    BUSINESS PURPOSE:
    - Drives expert participation in knowledge sharing
    - Prioritizes questions needing attention
    - Creates feedback loop for content gaps
    """
    question = get_object_or_404(AskaQuestion, id=request.POST.get('id'))
    user = request.user
    
    if question.requests.filter(id=user.id).exists():
        # Remove request (toggle off)
        question.requests.remove(user)
        question.is_requested = False
        action = 'unrequested'
    else:
        # Add request (toggle on)
        question.requests.add(user)
        question.is_requested = True
        action = 'requested'
        
        # Trigger notification system for experts
        _notify_subject_experts(question)
    
    question.save()
    
    context = {
        'question': question,
        'action': action,
        'total_requests': question.total_requests(),
        'user_requested': question.requests.filter(id=user.id).exists()
    }
    
    if request.is_ajax():
        html = render_to_string('feeds/question_request_count.html', context, request=request)
        return JsonResponse({'form': html})

def _notify_subject_experts(question):
    """Helper function to notify relevant experts about answer requests"""
    # Get question tags
    question_tags = question.select_tag.all()
    
    # Find users with matching interests and high reputation
    expert_candidates = User.objects.filter(
        tags__my_subtag__in=question_tags
    ).distinct()
    
    # Filter for users with good answer history
    qualified_experts = []
    for user in expert_candidates:
        user_answers = AnsweraQuestion.objects.filter(user=user)
        if user_answers.exists():
            avg_score = sum(a.total_votes() for a in user_answers) / user_answers.count()
            if avg_score > 2:  # Threshold for expert status
                qualified_experts.append(user)
    
    # TODO: Send notifications to qualified experts
    return qualified_experts
```

#### 3. Content Management System - Administrative Controls
**Purpose**: Comprehensive content lifecycle management with moderation capabilities

**Advanced Business Logic**:
```python
def delete_post(request, id):
    """
    CORE LOGIC: Secure content deletion with ownership validation
    
    HOW IT WORKS:
    1. AUTHORIZATION: Verifies user ownership or admin privileges
    2. CASCADE HANDLING: Manages related data (answers, comments, files)
    3. AUDIT TRAIL: Logs deletion for administrative review
    4. CLEANUP: Removes associated files and relationships
    
    BUSINESS PURPOSE:
    - Enables content lifecycle management
    - Protects against unauthorized deletions
    - Maintains data integrity across relationships
    - Supports content moderation workflows
    """
    question = get_object_or_404(AskaQuestion, id=id)
    user = request.user
    
    # Authorization check
    can_delete = (
        question.user == user or  # Owner
        user.is_superuser or     # Admin
        Roles.objects.filter(user=user, role__in=['moderator', 'admin']).exists()
    )
    
    if not can_delete:
        messages.error(request, "Unauthorized deletion attempt")
        return redirect('feeds:feeds')
    
    # Log deletion for audit trail
    _log_content_action(user, question, 'deleted')
    
    # Handle file cleanup
    if question.file:
        try:
            default_storage.delete(question.file.path)
        except:
            pass  # File might already be deleted
    
    # Soft delete vs hard delete based on content importance
    if _has_valuable_content(question):
        # Soft delete - mark as hidden but preserve data
        hidden.objects.get_or_create(user=user, question=question)
        messages.success(request, "Question hidden from feeds")
    else:
        # Hard delete - remove entirely
        question.delete()
        messages.success(request, "Question deleted permanently")
    
    return redirect('feeds:feeds')

def hide_post(request, id):
    """
    CORE LOGIC: Personal content filtering without affecting others
    
    HOW IT WORKS:
    1. Creates user-specific hide record
    2. Filters content from user's feed
    3. Preserves content availability for other users
    4. Enables unhide functionality
    """
    question = get_object_or_404(AskaQuestion, id=id)
    user = request.user
    
    hidden_obj, created = hidden.objects.get_or_create(
        user=user, 
        question=question
    )
    
    if created:
        messages.success(request, "Question hidden from your feed")
    else:
        messages.info(request, "Question was already hidden")
    
    return redirect('feeds:feeds')

def _has_valuable_content(question):
    """Helper to determine if content should be preserved"""
    return (
        question.total_likes() > 5 or
        question.answeraquestion_set.count() > 2 or
        question.comments_set.count() > 5
    )

def _log_content_action(user, question, action):
    """Helper to log content management actions"""
    # TODO: Implement comprehensive audit logging
    pass
```

#### 4. Profile and Administrative Views
**Functions**: `profile()`, `admin()`, `administrativeView()`

**Capabilities**:
- **User Profiles**: Bio management, interest tracking, activity metrics
- **Administrative Dashboard**: Content moderation, user management, system statistics
- **Role Management**: Permission assignment and role-based content control

### URL Pattern Architecture

```python
urlpatterns = [
    # Core functionality
    url(r'^$', views.feeds, name='feeds'),                             # Main feed
    url(r'^profile/(?P<string>[-\w]+)/$', views.profile),              # User profiles
    url(r'^admin', views.admin, name='feeds_admin'),                   # Admin dashboard
    
    # Content interaction
    url(r'^upvote_ques/(?P<id>[0-9]+)$', views.upvoteQuestion),       # Question voting
    url(r'^question_id_/(?P<id>[0-9]+)/$', views.ParticularQuestion), # Single question view
    
    # Content management
    url(r'^(?P<id>[0-9]+)/delete_post/$', views.delete_post),         # Post deletion
    url(r'^(?P<id>[0-9]+)/hide_post/$', views.hide_post),             # Content hiding
    
    # Administrative functions
    url(r'^administrative/(?P<string>[-\w]+)/$', views.administrativeView), # Admin views
]
```

### Administrative Integration

**Registered Models in Admin**:
- `AskaQuestion`: Question management and moderation
- `Comments`: Comment oversight and moderation
- `AllTags`: Tag system administration
- `report`: Content report review
- `AnsweraQuestion`: Answer quality control
- `Profile`: User profile management
- `Roles`: Role and permission administration
- `QuestionAccessControl`: Granular permission management

---

## Business Logic Implementation

### Content Discovery Algorithm
1. **Tag-Based Filtering**: Users see content matching their interest tags
2. **Chronological Ordering**: Latest content appears first
3. **Engagement Weighting**: Popular content (high likes/answers) gets visibility boost
4. **Quality Control**: Reported content gets reduced visibility

### Community Moderation System
1. **User Reporting**: Community flags inappropriate content
2. **Administrative Review**: Moderators review reported content
3. **Role-Based Actions**: Special permissions for content management
4. **Granular Controls**: Per-question permission settings

### Engagement Tracking
1. **Interaction Metrics**: Likes, dislikes, requests, views
2. **User Activity**: Profile views, question posting frequency
3. **Content Quality**: Answer vote ratios, comment engagement
4. **System Health**: Report frequency, active user counts

---

## System Dependencies

### Internal Dependencies
- **globals.models**: `ExtraInfo` for extended user information
- **Django Auth**: `User` model for authentication and authorization
- **Django Core**: File handling, pagination, messaging systems

### External Dependencies
- **markdown_deux**: Rich text processing for content formatting
- **Django Utils**: Timezone handling, safe string processing
- **File Storage**: Media file handling for attachments and profile images

---

## Key Features Summary

1. **Comprehensive Q&A System**: Full-featured question and answer platform
2. **Tag-Based Organization**: 18 main categories with 100+ subcategories
3. **Multi-Level Discussions**: Question → Answer → Comment → Reply hierarchy
4. **User Engagement**: Voting, requesting, and interaction tracking
5. **Content Moderation**: Reporting, hiding, and administrative controls
6. **Personalization**: Interest tracking and customized feeds
7. **Role-Based Permissions**: Granular access control system
8. **Rich Media Support**: File attachments and profile images
9. **Real-Time Interactions**: AJAX-based voting and commenting
10. **Administrative Dashboard**: Complete moderation and management tools

The Feeds Module serves as the central social networking hub for Fusion IIIT, enabling knowledge sharing, community building, and academic collaboration through a sophisticated content management and interaction system.
