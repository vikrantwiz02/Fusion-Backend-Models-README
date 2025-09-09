# Complete Globals Module System Documentation - Fusion IIIT (Part 4)

## Database Models Analysis with Business Logic (Continued)

### 9. IssueImage Model
**Purpose**: Manages file attachments for issue reports, enabling visual documentation of bugs, interface problems, and feature requests.

**PostgreSQL Table**: `globals_issueimage`

**Fields Structure**:
```python
def Issue_image_directory(instance, filename):
    return 'issues/{0}/images/{1}'.format(instance.user.username, filename)

class IssueImage(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    image = models.ImageField(upload_to=Issue_image_directory)
```

**Enhanced Business Methods**:
```python
def get_image_metadata(self):
    """
    CORE LOGIC: Extract and analyze image metadata for issue documentation
    
    HOW IT WORKS:
    1. Analyzes uploaded image file properties and characteristics
    2. Extracts technical metadata useful for debugging
    3. Determines image quality and suitability for issue reporting
    4. Provides file management and storage information
    
    BUSINESS PURPOSE:
    - Issue documentation quality assurance
    - Technical debugging support through visual evidence
    - Storage optimization and file management
    - User experience improvement in issue reporting
    """
    if not self.image:
        return {'status': 'no_image', 'metadata': {}}
    
    try:
        from PIL import Image
        import os
        
        image_path = self.image.path
        pil_image = Image.open(image_path)
        
        # File system information
        file_stats = os.stat(image_path)
        
        metadata = {
            'file_info': {
                'filename': os.path.basename(self.image.name),
                'file_size_bytes': file_stats.st_size,
                'file_size_mb': round(file_stats.st_size / (1024 * 1024), 2),
                'upload_path': self.image.url,
                'full_path': image_path
            },
            'image_properties': {
                'format': pil_image.format,
                'mode': pil_image.mode,
                'width': pil_image.width,
                'height': pil_image.height,
                'aspect_ratio': round(pil_image.width / pil_image.height, 2),
                'total_pixels': pil_image.width * pil_image.height
            },
            'quality_assessment': {
                'resolution_category': self._categorize_resolution(pil_image.width, pil_image.height),
                'file_size_category': self._categorize_file_size(file_stats.st_size),
                'is_suitable_for_web': file_stats.st_size < 5 * 1024 * 1024,  # 5MB limit
                'is_high_quality': pil_image.width >= 1024 and pil_image.height >= 768
            },
            'technical_details': {
                'color_channels': len(pil_image.getbands()) if hasattr(pil_image, 'getbands') else 'unknown',
                'has_transparency': pil_image.mode in ('RGBA', 'LA') or 'transparency' in pil_image.info,
                'estimated_compression': self._estimate_compression_ratio(pil_image, file_stats.st_size)
            }
        }
        
        # Add EXIF data if available
        if hasattr(pil_image, '_getexif') and pil_image._getexif():
            metadata['exif_data'] = self._extract_relevant_exif(pil_image._getexif())
        
        return {
            'status': 'success',
            'metadata': metadata,
            'recommendations': self._generate_image_recommendations(metadata)
        }
        
    except Exception as e:
        return {
            'status': 'error',
            'error_message': str(e),
            'metadata': {}
        }

def _categorize_resolution(self, width, height):
    """Helper method to categorize image resolution"""
    total_pixels = width * height
    
    if total_pixels >= 2073600:  # 1920x1080 (Full HD)
        return 'high'
    elif total_pixels >= 921600:  # 1280x720 (HD)
        return 'medium'
    elif total_pixels >= 307200:  # 640x480 (VGA)
        return 'low'
    else:
        return 'very_low'

def _categorize_file_size(self, size_bytes):
    """Helper method to categorize file size"""
    size_mb = size_bytes / (1024 * 1024)
    
    if size_mb >= 10:
        return 'very_large'
    elif size_mb >= 5:
        return 'large'
    elif size_mb >= 1:
        return 'medium'
    else:
        return 'small'

def _estimate_compression_ratio(self, pil_image, file_size):
    """Helper method to estimate compression efficiency"""
    # Rough estimation of compression ratio
    uncompressed_size = pil_image.width * pil_image.height * 3  # RGB channels
    compression_ratio = file_size / uncompressed_size if uncompressed_size > 0 else 0
    
    return {
        'ratio': round(compression_ratio, 3),
        'efficiency': 'good' if compression_ratio < 0.1 else 'moderate' if compression_ratio < 0.3 else 'poor'
    }

def _extract_relevant_exif(self, exif_dict):
    """Helper method to extract relevant EXIF metadata"""
    relevant_tags = {
        'DateTime': 'capture_time',
        'Software': 'software_used',
        'ImageWidth': 'original_width',
        'ImageLength': 'original_height',
        'Orientation': 'orientation'
    }
    
    extracted_exif = {}
    for tag_id, tag_name in relevant_tags.items():
        if tag_id in exif_dict:
            extracted_exif[tag_name] = exif_dict[tag_id]
    
    return extracted_exif

def _generate_image_recommendations(self, metadata):
    """Helper method to generate recommendations for image optimization"""
    recommendations = []
    
    file_info = metadata['file_info']
    quality = metadata['quality_assessment']
    
    # File size recommendations
    if file_info['file_size_mb'] > 10:
        recommendations.append("Consider compressing the image to reduce file size (current: {:.1f}MB)".format(file_info['file_size_mb']))
    
    # Resolution recommendations
    if quality['resolution_category'] == 'very_low':
        recommendations.append("Image resolution is quite low - consider taking a higher quality screenshot")
    elif quality['resolution_category'] == 'high' and file_info['file_size_mb'] > 5:
        recommendations.append("High resolution image detected - consider resizing for web use")
    
    # Format recommendations
    image_props = metadata['image_properties']
    if image_props['format'] == 'PNG' and file_info['file_size_mb'] > 2:
        recommendations.append("Consider converting PNG to JPEG for smaller file size")
    
    # General recommendations
    if not quality['is_suitable_for_web']:
        recommendations.append("Image may be too large for web viewing - consider optimization")
    
    if len(recommendations) == 0:
        recommendations.append("Image quality and size are suitable for issue reporting")
    
    return recommendations

def get_usage_context(self):
    """
    CORE LOGIC: Determine how and where this image is being used in issue reporting
    
    HOW IT WORKS:
    1. Finds all issues that reference this image
    2. Analyzes the context and purpose of image usage
    3. Determines the effectiveness of the image for issue resolution
    4. Provides insights on image utility and relevance
    
    BUSINESS PURPOSE:
    - Issue resolution effectiveness tracking
    - Image utility assessment for better reporting
    - Context-aware issue management
    - User guidance for effective issue documentation
    """
    # Find issues that use this image
    related_issues = Issue.objects.filter(images=self)
    
    usage_context = {
        'usage_statistics': {
            'total_issues_using': related_issues.count(),
            'issue_types': list(related_issues.values_list('report_type', flat=True).distinct()),
            'modules_affected': list(related_issues.values_list('module', flat=True).distinct()),
            'reporting_users': list(related_issues.values_list('user__username', flat=True))
        },
        'issue_resolution': {
            'closed_issues': related_issues.filter(closed=True).count(),
            'open_issues': related_issues.filter(closed=False).count(),
            'resolution_rate': 0,
            'average_resolution_time': self._calculate_avg_resolution_time(related_issues)
        },
        'image_effectiveness': {
            'helps_bug_reports': related_issues.filter(report_type='bug_report').count(),
            'helps_ui_issues': related_issues.filter(report_type='ui_issue').count(),
            'supports_feature_requests': related_issues.filter(report_type='feature_request').count(),
            'clarifies_security_issues': related_issues.filter(report_type='security_issue').count()
        }
    }
    
    # Calculate resolution rate
    total_issues = usage_context['usage_statistics']['total_issues_using']
    closed_issues = usage_context['issue_resolution']['closed_issues']
    
    if total_issues > 0:
        usage_context['issue_resolution']['resolution_rate'] = round(
            (closed_issues / total_issues) * 100, 1
        )
    
    # Determine image utility
    usage_context['utility_assessment'] = self._assess_image_utility(usage_context)
    
    return usage_context

def _calculate_avg_resolution_time(self, issues):
    """Helper method to calculate average resolution time for issues with this image"""
    closed_issues = issues.filter(closed=True)
    
    if not closed_issues.exists():
        return None
    
    total_resolution_time = 0
    resolution_count = 0
    
    for issue in closed_issues:
        # Note: This assumes there's a resolution timestamp field
        # In actual implementation, you'd need to track when issues are closed
        resolution_time = (timezone.now() - issue.added_on).days
        total_resolution_time += resolution_time
        resolution_count += 1
    
    if resolution_count > 0:
        avg_days = total_resolution_time / resolution_count
        return {
            'average_days': round(avg_days, 1),
            'category': 'fast' if avg_days <= 3 else 'moderate' if avg_days <= 7 else 'slow'
        }
    
    return None

def _assess_image_utility(self, usage_context):
    """Helper method to assess the utility of this image for issue resolution"""
    utility_score = 0
    utility_factors = []
    
    stats = usage_context['usage_statistics']
    resolution = usage_context['issue_resolution']
    
    # Multiple usage indicates utility
    if stats['total_issues_using'] > 1:
        utility_score += 20
        utility_factors.append('reused_across_issues')
    
    # High resolution rate indicates effectiveness
    if resolution['resolution_rate'] > 70:
        utility_score += 30
        utility_factors.append('helps_issue_resolution')
    
    # Visual issue types benefit most from images
    effectiveness = usage_context['image_effectiveness']
    visual_issue_count = effectiveness['helps_ui_issues'] + effectiveness['helps_bug_reports']
    
    if visual_issue_count > 0:
        utility_score += 25
        utility_factors.append('suitable_for_visual_issues')
    
    # Quick resolution indicates clarity
    avg_resolution = resolution['average_resolution_time']
    if avg_resolution and avg_resolution['category'] == 'fast':
        utility_score += 25
        utility_factors.append('enables_quick_resolution')
    
    # Determine overall utility
    if utility_score >= 70:
        utility_level = 'highly_useful'
    elif utility_score >= 50:
        utility_level = 'moderately_useful'
    elif utility_score >= 30:
        utility_level = 'somewhat_useful'
    else:
        utility_level = 'limited_utility'
    
    return {
        'score': utility_score,
        'level': utility_level,
        'factors': utility_factors,
        'recommendations': self._generate_utility_recommendations(utility_score, utility_factors)
    }

def _generate_utility_recommendations(self, score, factors):
    """Helper method to generate recommendations for improving image utility"""
    recommendations = []
    
    if score < 30:
        recommendations.extend([
            "Consider adding more descriptive context to images",
            "Ensure images clearly show the issue being reported",
            "Include annotations or highlights to point out specific problems"
        ])
    
    if 'reused_across_issues' not in factors:
        recommendations.append("This type of image could be useful for similar issue reports")
    
    if 'helps_issue_resolution' not in factors:
        recommendations.append("Consider including additional visual information to aid resolution")
    
    if 'suitable_for_visual_issues' not in factors:
        recommendations.append("Images work best for UI/visual bugs - ensure the image clearly demonstrates the issue")
    
    return recommendations

def generate_thumbnail(self, size=(150, 150)):
    """
    CORE LOGIC: Generate optimized thumbnail for quick preview in issue lists
    
    HOW IT WORKS:
    1. Creates a smaller version of the original image
    2. Optimizes for web display and quick loading
    3. Maintains aspect ratio while fitting within specified dimensions
    4. Stores thumbnail for efficient repeated access
    
    BUSINESS PURPOSE:
    - Improve user interface performance in issue lists
    - Enable quick visual identification of issue types
    - Reduce bandwidth usage for issue browsing
    - Enhance user experience in issue management
    """
    if not self.image:
        return None
    
    try:
        from PIL import Image
        import os
        from io import BytesIO
        from django.core.files.base import ContentFile
        
        # Open original image
        original = Image.open(self.image.path)
        
        # Calculate thumbnail size maintaining aspect ratio
        original.thumbnail(size, Image.Resampling.LANCZOS)
        
        # Create thumbnail path
        thumbnail_name = f"thumb_{os.path.basename(self.image.name)}"
        thumbnail_dir = os.path.join(os.path.dirname(self.image.path), 'thumbnails')
        
        # Create thumbnails directory if it doesn't exist
        os.makedirs(thumbnail_dir, exist_ok=True)
        
        thumbnail_path = os.path.join(thumbnail_dir, thumbnail_name)
        
        # Save thumbnail
        original.save(thumbnail_path, optimize=True, quality=85)
        
        # Calculate thumbnail statistics
        thumbnail_stats = os.stat(thumbnail_path)
        original_stats = os.stat(self.image.path)
        
        return {
            'status': 'success',
            'thumbnail_path': thumbnail_path,
            'thumbnail_url': self.image.url.replace(os.path.basename(self.image.name), f'thumbnails/{thumbnail_name}'),
            'size_reduction': {
                'original_bytes': original_stats.st_size,
                'thumbnail_bytes': thumbnail_stats.st_size,
                'reduction_percentage': round(
                    (1 - thumbnail_stats.st_size / original_stats.st_size) * 100, 1
                ),
                'space_saved_kb': round((original_stats.st_size - thumbnail_stats.st_size) / 1024, 1)
            },
            'dimensions': {
                'original': (Image.open(self.image.path).width, Image.open(self.image.path).height),
                'thumbnail': original.size
            }
        }
        
    except Exception as e:
        return {
            'status': 'error',
            'error_message': str(e)
        }

def validate_image_for_issue_reporting(self):
    """
    CORE LOGIC: Validate image suitability for effective issue reporting
    
    HOW IT WORKS:
    1. Checks image technical properties against issue reporting requirements
    2. Analyzes content suitability for different issue types
    3. Provides validation results and improvement suggestions
    4. Ensures images contribute effectively to issue resolution
    
    BUSINESS PURPOSE:
    - Quality assurance for issue documentation
    - User guidance for effective issue reporting
    - Improve issue resolution efficiency through better documentation
    - Reduce back-and-forth communication in issue resolution
    """
    validation_results = {
        'is_valid': True,
        'validation_errors': [],
        'validation_warnings': [],
        'quality_score': 0,
        'recommendations': []
    }
    
    # Get image metadata
    metadata_result = self.get_image_metadata()
    
    if metadata_result['status'] != 'success':
        validation_results['is_valid'] = False
        validation_results['validation_errors'].append(f"Cannot analyze image: {metadata_result.get('error_message', 'Unknown error')}")
        return validation_results
    
    metadata = metadata_result['metadata']
    file_info = metadata['file_info']
    image_props = metadata['image_properties']
    quality = metadata['quality_assessment']
    
    # File size validation
    if file_info['file_size_mb'] > 25:
        validation_results['is_valid'] = False
        validation_results['validation_errors'].append(f"Image too large ({file_info['file_size_mb']:.1f}MB). Maximum allowed: 25MB")
    elif file_info['file_size_mb'] > 10:
        validation_results['validation_warnings'].append(f"Large image size ({file_info['file_size_mb']:.1f}MB). Consider compressing for faster upload")
    
    # Resolution validation
    if quality['resolution_category'] == 'very_low':
        validation_results['validation_warnings'].append("Very low resolution image. Consider taking a higher quality screenshot")
        validation_results['quality_score'] -= 20
    elif quality['resolution_category'] == 'high':
        validation_results['quality_score'] += 20
    
    # Format validation
    acceptable_formats = ['JPEG', 'PNG', 'GIF', 'WebP']
    if image_props['format'] not in acceptable_formats:
        validation_results['validation_warnings'].append(f"Uncommon image format ({image_props['format']}). Consider using JPEG or PNG")
    
    # Aspect ratio validation (for UI screenshots)
    aspect_ratio = image_props['aspect_ratio']
    if aspect_ratio < 0.5 or aspect_ratio > 3.0:
        validation_results['validation_warnings'].append("Unusual aspect ratio detected. Ensure the image shows the relevant interface area")
    
    # Quality score calculation
    base_score = 50
    
    if quality['is_high_quality']:
        base_score += 30
    if quality['is_suitable_for_web']:
        base_score += 20
    if image_props['format'] in ['JPEG', 'PNG']:
        base_score += 10
    
    validation_results['quality_score'] = min(100, base_score)
    
    # Generate recommendations
    if validation_results['quality_score'] < 60:
        validation_results['recommendations'].append("Consider retaking the image with better quality settings")
    
    if file_info['file_size_mb'] > 5:
        validation_results['recommendations'].append("Compress the image to improve upload speed")
    
    if quality['resolution_category'] in ['low', 'very_low']:
        validation_results['recommendations'].append("Take a higher resolution screenshot for better clarity")
    
    validation_results['recommendations'].extend(metadata_result.get('recommendations', []))
    
    return validation_results

def __str__(self):
    return f"Issue Image by {self.user.username}: {os.path.basename(self.image.name) if self.image else 'No image'}"
```

**Key Business Logic**:
- **Image Analysis**: Comprehensive metadata extraction and quality assessment
- **Usage Tracking**: Context analysis for image effectiveness in issue resolution
- **Thumbnail Generation**: Optimized preview creation for performance
- **Validation**: Quality assurance for effective issue documentation

---

### 10. Issue Model
**Purpose**: Comprehensive issue tracking system for bug reports, feature requests, and system improvement suggestions with community support mechanisms.

**PostgreSQL Table**: `globals_issue`

**Fields Structure**:
```python
class Issue(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name="reported_issues")
    report_type = models.CharField(max_length=63, choices=Constants.ISSUE_TYPES)
    module = models.CharField(max_length=63, choices=Constants.MODULES)
    closed = models.BooleanField(default=False)
    text = models.TextField()
    title = models.CharField(max_length=255)
    images = models.ManyToManyField(IssueImage, blank=True)
    support = models.ManyToManyField(User, blank=True)
    timestamp = models.DateTimeField(auto_now=True)
    added_on = models.DateTimeField(auto_now_add=True)
```

**Enhanced Business Methods**:
```python
def get_comprehensive_issue_analysis(self):
    """
    CORE LOGIC: Generate comprehensive analysis of issue for effective resolution
    
    HOW IT WORKS:
    1. Analyzes issue content, type, and context for resolution planning
    2. Assesses community support and priority indicators
    3. Evaluates technical details and reproduction information
    4. Provides structured data for development team action
    
    BUSINESS PURPOSE:
    - Efficient issue triage and prioritization
    - Development resource allocation optimization
    - Quality assurance and testing guidance
    - User satisfaction improvement through effective resolution
    """
    analysis = {
        'issue_classification': {
            'type': self.report_type,
            'module': self.module,
            'severity_level': self._assess_severity_level(),
            'complexity_estimate': self._estimate_complexity(),
            'priority_score': self._calculate_priority_score(),
            'category': self._categorize_issue_type()
        },
        'content_analysis': {
            'title_keywords': self._extract_title_keywords(),
            'description_length': len(self.text),
            'detail_level': self._assess_detail_level(),
            'technical_terms': self._extract_technical_terms(),
            'reproduction_info': self._extract_reproduction_info(),
            'environment_details': self._extract_environment_details()
        },
        'community_metrics': {
            'support_count': self.support.count(),
            'support_ratio': self._calculate_support_ratio(),
            'community_priority': self._assess_community_priority(),
            'user_engagement': self._analyze_user_engagement(),
            'social_proof': self._calculate_social_proof_score()
        },
        'visual_documentation': {
            'has_images': self.images.exists(),
            'image_count': self.images.count(),
            'image_quality': self._assess_image_quality(),
            'visual_clarity': self._assess_visual_clarity(),
            'documentation_completeness': self._assess_documentation_completeness()
        },
        'resolution_indicators': {
            'is_actionable': self._assess_actionability(),
            'required_expertise': self._identify_required_expertise(),
            'estimated_effort': self._estimate_resolution_effort(),
            'dependencies': self._identify_dependencies(),
            'quick_fix_potential': self._assess_quick_fix_potential()
        },
        'timeline_analysis': {
            'reported_date': self.added_on,
            'last_updated': self.timestamp,
            'age_in_days': (timezone.now() - self.added_on).days,
            'staleness_level': self._assess_staleness(),
            'urgency_factor': self._calculate_urgency_factor()
        }
    }
    
    return analysis

def _assess_severity_level(self):
    """Helper method to assess issue severity"""
    severity_keywords = {
        'critical': ['crash', 'data loss', 'security', 'cannot login', 'system down', 'emergency'],
        'high': ['error', 'broken', 'not working', 'fails', 'incorrect', 'missing'],
        'medium': ['slow', 'difficult', 'confusing', 'improvement', 'enhancement'],
        'low': ['minor', 'cosmetic', 'suggestion', 'nice to have', 'polish']
    }
    
    text_lower = (self.title + ' ' + self.text).lower()
    
    for severity, keywords in severity_keywords.items():
        if any(keyword in text_lower for keyword in keywords):
            return severity
    
    # Default based on issue type
    if self.report_type == 'security_issue':
        return 'critical'
    elif self.report_type == 'bug_report':
        return 'high'
    else:
        return 'medium'

def _estimate_complexity(self):
    """Helper method to estimate implementation complexity"""
    complexity_indicators = {
        'high': ['integration', 'database', 'architecture', 'system-wide', 'major change'],
        'medium': ['feature', 'functionality', 'workflow', 'multiple pages', 'backend'],
        'low': ['ui', 'text', 'color', 'button', 'link', 'cosmetic', 'typo']
    }
    
    text_lower = (self.title + ' ' + self.text).lower()
    
    for complexity, indicators in complexity_indicators.items():
        if any(indicator in text_lower for indicator in indicators):
            return complexity
    
    # Default based on report type
    if self.report_type == 'feature_request':
        return 'medium'
    elif self.report_type == 'ui_issue':
        return 'low'
    else:
        return 'medium'

def _calculate_priority_score(self):
    """Helper method to calculate numerical priority score"""
    score = 0
    
    # Severity weight
    severity_weights = {'critical': 40, 'high': 30, 'medium': 20, 'low': 10}
    score += severity_weights.get(self._assess_severity_level(), 20)
    
    # Community support weight
    support_count = self.support.count()
    score += min(support_count * 5, 25)  # Max 25 points for community support
    
    # Age factor (older issues get slight priority boost)
    age_days = (timezone.now() - self.added_on).days
    age_bonus = min(age_days * 0.5, 15)  # Max 15 points for age
    score += age_bonus
    
    # User type factor
    if hasattr(self.user, 'extrainfo'):
        if self.user.extrainfo.user_type == 'faculty':
            score += 10
        elif self.user.extrainfo.user_type == 'staff':
            score += 8
    
    # Issue type factor
    type_weights = {
        'security_issue': 20,
        'bug_report': 15,
        'feature_request': 10,
        'ui_issue': 8,
        'other': 5
    }
    score += type_weights.get(self.report_type, 5)
    
    return round(score, 1)

def _categorize_issue_type(self):
    """Helper method to categorize issue for routing"""
    if self.report_type == 'security_issue':
        return 'security_critical'
    elif self.report_type == 'bug_report':
        if self._assess_severity_level() in ['critical', 'high']:
            return 'production_bug'
        else:
            return 'minor_bug'
    elif self.report_type == 'feature_request':
        return 'enhancement'
    elif self.report_type == 'ui_issue':
        return 'user_experience'
    else:
        return 'general_issue'

def _extract_title_keywords(self):
    """Helper method to extract keywords from issue title"""
    import re
    
    # Remove common words and extract meaningful keywords
    common_words = {'the', 'a', 'an', 'and', 'or', 'but', 'in', 'on', 'at', 'to', 'for', 'of', 'with', 'by', 'is', 'are', 'was', 'were', 'be', 'been', 'have', 'has', 'had', 'do', 'does', 'did', 'will', 'would', 'could', 'should', 'may', 'might', 'must', 'can', 'cannot', 'not'}
    
    # Extract words from title
    words = re.findall(r'\b\w+\b', self.title.lower())
    keywords = [word for word in words if word not in common_words and len(word) > 2]
    
    return keywords[:10]  # Return top 10 keywords

def _assess_detail_level(self):
    """Helper method to assess level of detail in issue description"""
    text_length = len(self.text)
    
    if text_length < 50:
        return 'minimal'
    elif text_length < 200:
        return 'brief'
    elif text_length < 500:
        return 'adequate'
    elif text_length < 1000:
        return 'detailed'
    else:
        return 'comprehensive'

def _extract_technical_terms(self):
    """Helper method to extract technical terms from issue description"""
    technical_keywords = [
        'error', 'exception', 'bug', 'crash', 'timeout', 'loading', 'response',
        'database', 'server', 'client', 'browser', 'api', 'authentication',
        'permission', 'validation', 'form', 'field', 'button', 'link',
        'page', 'redirect', 'session', 'cookie', 'javascript', 'css',
        'html', 'sql', 'json', 'xml', 'http', 'https', 'ajax'
    ]
    
    text_lower = self.text.lower()
    found_terms = [term for term in technical_keywords if term in text_lower]
    
    return found_terms

def _extract_reproduction_info(self):
    """Helper method to extract reproduction steps from description"""
    reproduction_indicators = [
        'steps to reproduce', 'how to reproduce', 'reproduction steps',
        'to reproduce', 'reproduce the issue', 'steps:', 'procedure:',
        '1.', '2.', '3.', 'first', 'then', 'next', 'finally'
    ]
    
    text_lower = self.text.lower()
    has_reproduction_info = any(indicator in text_lower for indicator in reproduction_indicators)
    
    # Simple step counting
    step_count = len(re.findall(r'\d+\.', self.text))
    
    return {
        'has_steps': has_reproduction_info,
        'step_count': step_count,
        'quality': 'good' if step_count >= 3 else 'basic' if step_count > 0 else 'none'
    }

def _extract_environment_details(self):
    """Helper method to extract environment/system details"""
    environment_keywords = [
        'browser', 'chrome', 'firefox', 'safari', 'edge',
        'mobile', 'desktop', 'tablet', 'android', 'ios',
        'windows', 'mac', 'linux', 'version', 'operating system'
    ]
    
    text_lower = self.text.lower()
    found_environment = [keyword for keyword in environment_keywords if keyword in text_lower]
    
    return {
        'mentioned_environment': found_environment,
        'has_environment_info': len(found_environment) > 0,
        'detail_level': 'good' if len(found_environment) >= 3 else 'basic' if len(found_environment) > 0 else 'none'
    }

def _calculate_support_ratio(self):
    """Helper method to calculate community support ratio"""
    # This would ideally compare against similar issues or user base
    # For now, simple calculation based on support count
    support_count = self.support.count()
    
    if support_count >= 10:
        return 'high'
    elif support_count >= 5:
        return 'medium'
    elif support_count >= 1:
        return 'low'
    else:
        return 'none'

def _assess_community_priority(self):
    """Helper method to assess priority based on community engagement"""
    support_count = self.support.count()
    age_days = (timezone.now() - self.added_on).days
    
    # Calculate daily support rate
    daily_support_rate = support_count / max(age_days, 1)
    
    if daily_support_rate >= 1.0:
        return 'very_high'
    elif daily_support_rate >= 0.5:
        return 'high'
    elif daily_support_rate >= 0.2:
        return 'medium'
    else:
        return 'low'

def _analyze_user_engagement(self):
    """Helper method to analyze user engagement patterns"""
    support_users = self.support.all()
    
    engagement_analysis = {
        'total_supporters': support_users.count(),
        'unique_departments': len(set(
            user.extrainfo.department.name 
            for user in support_users 
            if hasattr(user, 'extrainfo') and user.extrainfo.department
        )),
        'user_types': {},
        'engagement_diversity': 'low'
    }
    
    # Analyze user types of supporters
    for user in support_users:
        if hasattr(user, 'extrainfo'):
            user_type = user.extrainfo.user_type
            engagement_analysis['user_types'][user_type] = engagement_analysis['user_types'].get(user_type, 0) + 1
    
    # Assess engagement diversity
    unique_user_types = len(engagement_analysis['user_types'])
    unique_departments = engagement_analysis['unique_departments']
    
    if unique_user_types >= 3 and unique_departments >= 2:
        engagement_analysis['engagement_diversity'] = 'high'
    elif unique_user_types >= 2 or unique_departments >= 2:
        engagement_analysis['engagement_diversity'] = 'medium'
    
    return engagement_analysis

def _calculate_social_proof_score(self):
    """Helper method to calculate social proof score"""
    score = 0
    
    # Base support count
    score += self.support.count() * 10
    
    # Diversity bonus
    engagement = self._analyze_user_engagement()
    score += engagement['unique_departments'] * 5
    score += len(engagement['user_types']) * 3
    
    # Age adjustment (newer issues with support are more significant)
    age_days = (timezone.now() - self.added_on).days
    if age_days <= 7:
        score *= 1.5
    elif age_days <= 30:
        score *= 1.2
    
    return round(score, 1)

def _assess_image_quality(self):
    """Helper method to assess quality of attached images"""
    images = self.images.all()
    
    if not images.exists():
        return {'has_images': False, 'quality': 'none'}
    
    total_quality_score = 0
    image_count = images.count()
    
    for image in images:
        validation = image.validate_image_for_issue_reporting()
        total_quality_score += validation['quality_score']
    
    average_quality = total_quality_score / image_count if image_count > 0 else 0
    
    quality_level = 'excellent' if average_quality >= 80 else 'good' if average_quality >= 60 else 'poor'
    
    return {
        'has_images': True,
        'image_count': image_count,
        'average_quality_score': round(average_quality, 1),
        'quality_level': quality_level
    }

def _assess_visual_clarity(self):
    """Helper method to assess visual clarity of issue documentation"""
    images = self.images.all()
    
    if not images.exists():
        return 'text_only'
    
    # Analyze image types and their utility
    image_analysis = []
    for image in images:
        usage_context = image.get_usage_context()
        image_analysis.append(usage_context['utility_assessment']['level'])
    
    if any(level in ['highly_useful', 'moderately_useful'] for level in image_analysis):
        return 'clear'
    else:
        return 'needs_improvement'

def _assess_documentation_completeness(self):
    """Helper method to assess overall documentation completeness"""
    completeness_score = 0
    
    # Text description quality
    detail_level = self._assess_detail_level()
    detail_scores = {'comprehensive': 25, 'detailed': 20, 'adequate': 15, 'brief': 10, 'minimal': 5}
    completeness_score += detail_scores.get(detail_level, 5)
    
    # Reproduction information
    reproduction_info = self._extract_reproduction_info()
    if reproduction_info['quality'] == 'good':
        completeness_score += 25
    elif reproduction_info['quality'] == 'basic':
        completeness_score += 15
    
    # Environment details
    environment_info = self._extract_environment_details()
    if environment_info['detail_level'] == 'good':
        completeness_score += 20
    elif environment_info['detail_level'] == 'basic':
        completeness_score += 10
    
    # Visual documentation
    if self.images.exists():
        image_quality = self._assess_image_quality()
        if image_quality['quality_level'] == 'excellent':
            completeness_score += 20
        elif image_quality['quality_level'] == 'good':
            completeness_score += 15
        else:
            completeness_score += 5
    
    # Technical details
    technical_terms = self._extract_technical_terms()
    if len(technical_terms) >= 3:
        completeness_score += 10
    
    return {
        'score': min(completeness_score, 100),
        'level': 'excellent' if completeness_score >= 80 else 'good' if completeness_score >= 60 else 'adequate' if completeness_score >= 40 else 'poor'
    }

def _assess_actionability(self):
    """Helper method to assess if issue is actionable by development team"""
    actionability_factors = {
        'has_clear_description': len(self.text) > 100,
        'has_reproduction_steps': self._extract_reproduction_info()['has_steps'],
        'has_environment_info': self._extract_environment_details()['has_environment_info'],
        'has_visual_evidence': self.images.exists(),
        'severity_identified': self._assess_severity_level() != 'low',
        'module_specified': self.module != 'other'
    }
    
    actionable_count = sum(actionability_factors.values())
    total_factors = len(actionability_factors)
    
    actionability_score = (actionable_count / total_factors) * 100
    
    return {
        'score': round(actionability_score, 1),
        'is_actionable': actionability_score >= 60,
        'factors': actionability_factors,
        'missing_factors': [factor for factor, present in actionability_factors.items() if not present]
    }

def _identify_required_expertise(self):
    """Helper method to identify required expertise for resolution"""
    expertise_mapping = {
        'frontend': ['ui', 'interface', 'design', 'css', 'javascript', 'responsive'],
        'backend': ['database', 'server', 'api', 'authentication', 'performance'],
        'security': ['security', 'permission', 'authentication', 'vulnerability'],
        'devops': ['deployment', 'server', 'configuration', 'environment'],
        'ux_design': ['user experience', 'usability', 'workflow', 'navigation'],
        'qa_testing': ['testing', 'validation', 'verification', 'quality']
    }
    
    text_lower = (self.title + ' ' + self.text).lower()
    required_expertise = []
    
    for expertise, keywords in expertise_mapping.items():
        if any(keyword in text_lower for keyword in keywords):
            required_expertise.append(expertise)
    
    # Default based on issue type and module
    if not required_expertise:
        if self.report_type == 'ui_issue':
            required_expertise.append('frontend')
        elif self.report_type == 'security_issue':
            required_expertise.append('security')
        else:
            required_expertise.append('general_development')
    
    return required_expertise

def _estimate_resolution_effort(self):
    """Helper method to estimate effort required for resolution"""
    base_effort = 1  # Base effort in story points
    
    # Complexity factor
    complexity = self._estimate_complexity()
    complexity_multipliers = {'low': 1, 'medium': 2, 'high': 5}
    base_effort *= complexity_multipliers.get(complexity, 2)
    
    # Module factor (some modules are more complex)
    complex_modules = ['academic_information', 'file_tracking', 'eis']
    if self.module in complex_modules:
        base_effort *= 1.5
    
    # Issue type factor
    type_multipliers = {
        'security_issue': 3,
        'feature_request': 2,
        'bug_report': 1.5,
        'ui_issue': 1,
        'other': 1.2
    }
    base_effort *= type_multipliers.get(self.report_type, 1.2)
    
    # Documentation quality factor
    documentation = self._assess_documentation_completeness()
    if documentation['level'] == 'poor':
        base_effort *= 1.3  # Poor documentation increases effort
    elif documentation['level'] == 'excellent':
        base_effort *= 0.8  # Good documentation reduces effort
    
    effort_hours = base_effort * 4  # Convert to hours (assuming 4 hours per story point)
    
    return {
        'story_points': round(base_effort, 1),
        'estimated_hours': round(effort_hours, 1),
        'effort_category': 'quick' if effort_hours <= 4 else 'moderate' if effort_hours <= 16 else 'significant'
    }

def _identify_dependencies(self):
    """Helper method to identify potential dependencies"""
    dependency_keywords = {
        'database_migration': ['database', 'schema', 'migration', 'table'],
        'api_changes': ['api', 'endpoint', 'integration', 'service'],
        'ui_framework': ['framework', 'library', 'component', 'template'],
        'authentication_system': ['login', 'authentication', 'permission', 'access'],
        'external_service': ['external', 'third-party', 'service', 'integration']
    }
    
    text_lower = (self.title + ' ' + self.text).lower()
    identified_dependencies = []
    
    for dependency, keywords in dependency_keywords.items():
        if any(keyword in text_lower for keyword in keywords):
            identified_dependencies.append(dependency)
    
    return identified_dependencies

def _assess_quick_fix_potential(self):
    """Helper method to assess if issue could be a quick fix"""
    quick_fix_indicators = [
        'typo', 'spelling', 'text', 'label', 'color', 'alignment',
        'button text', 'link', 'tooltip', 'message', 'wording'
    ]
    
    text_lower = (self.title + ' ' + self.text).lower()
    has_quick_fix_indicators = any(indicator in text_lower for indicator in quick_fix_indicators)
    
    # Additional factors
    is_ui_issue = self.report_type == 'ui_issue'
    is_low_complexity = self._estimate_complexity() == 'low'
    has_good_documentation = self._assess_documentation_completeness()['level'] in ['good', 'excellent']
    
    quick_fix_score = 0
    if has_quick_fix_indicators:
        quick_fix_score += 40
    if is_ui_issue:
        quick_fix_score += 30
    if is_low_complexity:
        quick_fix_score += 20
    if has_good_documentation:
        quick_fix_score += 10
    
    return {
        'potential': quick_fix_score >= 60,
        'score': quick_fix_score,
        'indicators': {
            'has_quick_fix_keywords': has_quick_fix_indicators,
            'is_ui_issue': is_ui_issue,
            'is_low_complexity': is_low_complexity,
            'well_documented': has_good_documentation
        }
    }

def _assess_staleness(self):
    """Helper method to assess issue staleness"""
    age_days = (timezone.now() - self.added_on).days
    
    if age_days <= 7:
        return 'fresh'
    elif age_days <= 30:
        return 'recent'
    elif age_days <= 90:
        return 'aging'
    else:
        return 'stale'

def _calculate_urgency_factor(self):
    """Helper method to calculate urgency factor"""
    urgency_score = 0
    
    # Severity urgency
    severity = self._assess_severity_level()
    severity_urgency = {'critical': 50, 'high': 30, 'medium': 15, 'low': 5}
    urgency_score += severity_urgency.get(severity, 15)
    
    # Community pressure
    support_count = self.support.count()
    urgency_score += min(support_count * 3, 20)
    
    # Age factor (old critical issues become more urgent)
    age_days = (timezone.now() - self.added_on).days
    if severity in ['critical', 'high'] and age_days > 7:
        urgency_score += min(age_days, 15)
    
    # Issue type urgency
    type_urgency = {
        'security_issue': 25,
        'bug_report': 15,
        'feature_request': 5,
        'ui_issue': 8,
        'other': 3
    }
    urgency_score += type_urgency.get(self.report_type, 3)
    
    return round(urgency_score, 1)

def get_resolution_recommendations(self):
    """
    CORE LOGIC: Generate actionable recommendations for issue resolution
    
    HOW IT WORKS:
    1. Analyzes comprehensive issue data to identify resolution path
    2. Provides specific recommendations for different stakeholders
    3. Suggests process improvements and resource allocation
    4. Creates actionable roadmap for issue closure
    
    BUSINESS PURPOSE:
    - Accelerate issue resolution through guided recommendations
    - Optimize resource allocation and team assignment
    - Improve process efficiency and user satisfaction
    - Enable data-driven decision making in issue management
    """
    analysis = self.get_comprehensive_issue_analysis()
    
    recommendations = {
        'immediate_actions': [],
        'development_team': [],
        'project_management': [],
        'quality_assurance': [],
        'user_communication': [],
        'process_improvements': []
    }
    
    # Immediate actions based on priority and urgency
    priority_score = analysis['issue_classification']['priority_score']
    urgency = analysis['timeline_analysis']['urgency_factor']
    
    if urgency >= 70:
        recommendations['immediate_actions'].extend([
            "Escalate to senior developer immediately",
            "Assign dedicated resource for resolution",
            "Schedule emergency fix deployment if needed"
        ])
    elif priority_score >= 60:
        recommendations['immediate_actions'].extend([
            "Add to current sprint backlog",
            "Assign experienced developer",
            "Set target resolution within 1 week"
        ])
    else:
        recommendations['immediate_actions'].append("Add to product backlog with appropriate priority")
    
    # Development team recommendations
    severity = analysis['issue_classification']['severity_level']
    complexity = analysis['issue_classification']['complexity_estimate']
    required_expertise = analysis['resolution_indicators']['required_expertise']
    
    if severity in ['critical', 'high']:
        recommendations['development_team'].append(f"Assign {', '.join(required_expertise)} expert(s)")
    
    if complexity == 'high':
        recommendations['development_team'].extend([
            "Conduct technical design review before implementation",
            "Break down into smaller, manageable tasks",
            "Plan for comprehensive testing"
        ])
    
    quick_fix = analysis['resolution_indicators']['quick_fix_potential']
    if quick_fix['potential']:
        recommendations['development_team'].append("Consider as quick win for immediate deployment")
    
    # Project management recommendations
    effort = analysis['resolution_indicators']['estimated_effort']
    dependencies = analysis['resolution_indicators']['dependencies']
    
    recommendations['project_management'].append(f"Allocate {effort['estimated_hours']} hours ({effort['story_points']} story points)")
    
    if dependencies:
        recommendations['project_management'].append(f"Address dependencies: {', '.join(dependencies)}")
    
    support_count = analysis['community_metrics']['support_count']
    if support_count >= 5:
        recommendations['project_management'].append("High community interest - consider for next release")
    
    # Quality assurance recommendations
    documentation_quality = analysis['visual_documentation']['documentation_completeness']
    
    if documentation_quality['level'] == 'poor':
        recommendations['quality_assurance'].extend([
            "Request additional details from reporter",
            "Create detailed test cases for reproduction",
            "Document expected vs actual behavior clearly"
        ])
    
    if analysis['visual_documentation']['has_images']:
        recommendations['quality_assurance'].append("Use provided screenshots for test case validation")
    
    reproduction_info = analysis['content_analysis']['reproduction_info']
    if reproduction_info['quality'] == 'none':
        recommendations['quality_assurance'].append("Work with reporter to establish reproduction steps")
    
    # User communication recommendations
    age_days = analysis['timeline_analysis']['age_in_days']
    
    if age_days > 7 and not self.closed:
        recommendations['user_communication'].append("Provide status update to reporter and supporters")
    
    if support_count > 0:
        recommendations['user_communication'].append("Acknowledge community support and provide timeline")
    
    if analysis['issue_classification']['category'] == 'security_critical':
        recommendations['user_communication'].append("Use security incident communication protocol")
    
    # Process improvement recommendations
    actionability = analysis['resolution_indicators']['is_actionable']
    if not actionability:
        recommendations['process_improvements'].append("Improve issue template to capture essential information")
    
    if documentation_quality['level'] == 'poor' and self.images.count() == 0:
        recommendations['process_improvements'].append("Encourage visual documentation for better issue reports")
    
    if analysis['issue_classification']['module'] == 'other':
        recommendations['process_improvements'].append("Review module categorization for better routing")
    
    return recommendations

def generate_status_update(self):
    """
    CORE LOGIC: Generate comprehensive status update for stakeholders
    
    HOW IT WORKS:
    1. Compiles current issue status and progress indicators
    2. Provides different update levels for different stakeholders
    3. Includes actionable next steps and timeline information
    4. Formats information for various communication channels
    
    BUSINESS PURPOSE:
    - Transparent communication with users and stakeholders
    - Progress tracking and accountability
    - Expectation management and user satisfaction
    - Process documentation and audit trail
    """
    analysis = self.get_comprehensive_issue_analysis()
    
    status_update = {
        'executive_summary': {
            'issue_id': self.id,
            'title': self.title,
            'status': 'closed' if self.closed else 'open',
            'priority': analysis['issue_classification']['priority_score'],
            'urgency': 'high' if analysis['timeline_analysis']['urgency_factor'] >= 50 else 'medium',
            'community_support': analysis['community_metrics']['support_count'],
            'age_days': analysis['timeline_analysis']['age_in_days']
        },
        'technical_status': {
            'module': self.module,
            'issue_type': self.report_type,
            'severity': analysis['issue_classification']['severity_level'],
            'complexity': analysis['issue_classification']['complexity_estimate'],
            'estimated_effort': analysis['resolution_indicators']['estimated_effort'],
            'required_expertise': analysis['resolution_indicators']['required_expertise']
        },
        'progress_indicators': {
            'actionability_score': analysis['resolution_indicators']['is_actionable'],
            'documentation_quality': analysis['visual_documentation']['documentation_completeness']['level'],
            'reproduction_status': analysis['content_analysis']['reproduction_info']['quality'],
            'visual_evidence': 'available' if analysis['visual_documentation']['has_images'] else 'none'
        },
        'stakeholder_updates': {
            'for_reporter': self._generate_reporter_update(analysis),
            'for_supporters': self._generate_supporter_update(analysis),
            'for_development_team': self._generate_dev_team_update(analysis),
            'for_management': self._generate_management_update(analysis)
        },
        'next_steps': self._generate_next_steps(analysis),
        'communication_templates': {
            'email_subject': f"Issue #{self.id} Update: {self.title[:50]}{'...' if len(self.title) > 50 else ''}",
            'slack_message': self._generate_slack_update(analysis),
            'dashboard_summary': self._generate_dashboard_summary(analysis)
        }
    }
    
    return status_update

def _generate_reporter_update(self, analysis):
    """Helper method to generate update for issue reporter"""
    if self.closed:
        return f"Your issue '{self.title}' has been resolved. Thank you for your detailed report!"
    
    priority = analysis['issue_classification']['priority_score']
    effort = analysis['resolution_indicators']['estimated_effort']
    
    update = f"Thank you for reporting '{self.title}'. "
    
    if priority >= 60:
        update += "This issue has been prioritized for immediate attention. "
    else:
        update += "This issue has been added to our development backlog. "
    
    update += f"Estimated resolution time: {effort['effort_category']} effort. "
    
    if analysis['community_metrics']['support_count'] > 0:
        update += f"Your issue has received support from {analysis['community_metrics']['support_count']} community members. "
    
    return update

def _generate_supporter_update(self, analysis):
    """Helper method to generate update for issue supporters"""
    support_count = analysis['community_metrics']['support_count']
    
    if support_count == 0:
        return ""
    
    update = f"Issue '{self.title}' supported by {support_count} community members. "
    
    if self.closed:
        update += "This issue has been resolved thanks to community support!"
    else:
        priority = analysis['issue_classification']['priority_score']
        if priority >= 60:
            update += "High community support has elevated this issue's priority."
        else:
            update += "Community support is being considered in prioritization."
    
    return update

def _generate_dev_team_update(self, analysis):
    """Helper method to generate update for development team"""
    if self.closed:
        return f"Issue #{self.id} completed. Please update any relevant documentation."
    
    complexity = analysis['issue_classification']['complexity_estimate']
    effort = analysis['resolution_indicators']['estimated_effort']
    expertise = analysis['resolution_indicators']['required_expertise']
    
    update = f"Issue #{self.id} requires {complexity} complexity resolution. "
    update += f"Estimated effort: {effort['estimated_hours']} hours. "
    update += f"Required expertise: {', '.join(expertise)}. "
    
    quick_fix = analysis['resolution_indicators']['quick_fix_potential']
    if quick_fix['potential']:
        update += "Identified as potential quick fix."
    
    return update

def _generate_management_update(self, analysis):
    """Helper method to generate update for management"""
    priority = analysis['issue_classification']['priority_score']
    urgency = analysis['timeline_analysis']['urgency_factor']
    age = analysis['timeline_analysis']['age_in_days']
    
    update = f"Issue #{self.id}: Priority {priority}/100, "
    
    if urgency >= 70:
        update += "HIGH URGENCY. "
    elif urgency >= 40:
        update += "Medium urgency. "
    
    if age > 30:
        update += f"Age: {age} days - consider escalation. "
    
    support = analysis['community_metrics']['support_count']
    if support >= 5:
        update += f"Strong community support ({support} users). "
    
    return update

def _generate_next_steps(self, analysis):
    """Helper method to generate next steps"""
    if self.closed:
        return ["Mark as resolved", "Update documentation if needed", "Monitor for related issues"]
    
    next_steps = []
    
    actionability = analysis['resolution_indicators']['is_actionable']
    if not actionability:
        next_steps.append("Gather additional requirements and reproduction steps")
    
    priority = analysis['issue_classification']['priority_score']
    if priority >= 60:
        next_steps.append("Assign to current sprint")
    else:
        next_steps.append("Add to product backlog")
    
    documentation = analysis['visual_documentation']['documentation_completeness']
    if documentation['level'] == 'poor':
        next_steps.append("Request additional documentation from reporter")
    
    effort = analysis['resolution_indicators']['estimated_effort']
    if effort['effort_category'] == 'significant':
        next_steps.append("Break down into smaller tasks")
    
    return next_steps

def _generate_slack_update(self, analysis):
    """Helper method to generate Slack-formatted update"""
    priority = analysis['issue_classification']['priority_score']
    status_emoji = "🔴" if priority >= 70 else "🟡" if priority >= 40 else "🟢"
    
    slack_msg = f"{status_emoji} Issue #{self.id}: {self.title[:100]}\n"
    slack_msg += f"Priority: {priority}/100 | "
    slack_msg += f"Type: {self.report_type} | "
    slack_msg += f"Module: {self.module}\n"
    
    if analysis['community_metrics']['support_count'] > 0:
        slack_msg += f"👥 {analysis['community_metrics']['support_count']} supporters\n"
    
    effort = analysis['resolution_indicators']['estimated_effort']
    slack_msg += f"⏱️ Estimated: {effort['estimated_hours']}h"
    
    return slack_msg

def _generate_dashboard_summary(self, analysis):
    """Helper method to generate dashboard summary"""
    return {
        'id': self.id,
        'title_short': self.title[:50] + ('...' if len(self.title) > 50 else ''),
        'priority_score': analysis['issue_classification']['priority_score'],
        'urgency_level': 'high' if analysis['timeline_analysis']['urgency_factor'] >= 50 else 'medium',
        'status': 'closed' if self.closed else 'open',
        'age_days': analysis['timeline_analysis']['age_in_days'],
        'support_count': analysis['community_metrics']['support_count'],
        'effort_hours': analysis['resolution_indicators']['estimated_effort']['estimated_hours'],
        'module': self.module,
        'type': self.report_type
    }

def __str__(self):
    return f"Issue #{self.id}: {self.title} [{self.report_type}]"
```

**Key Business Logic**:
- **Comprehensive Analysis**: Multi-dimensional assessment of issues for effective resolution
- **Priority Calculation**: Intelligent prioritization based on multiple factors
- **Resolution Planning**: Effort estimation and resource allocation guidance
- **Stakeholder Communication**: Targeted updates for different user groups

---

This concludes Part 4 of the Globals Module documentation. The next parts will cover:

**Part 5**: ModuleAccess and PasswordResetTracker models
**Part 6**: View functions and business logic implementation
**Part 7**: API architecture and integration patterns
