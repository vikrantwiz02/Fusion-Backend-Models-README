# Complete Theoretical Documentation: Counselling Cell Models - Part 2

## **COMPREHENSIVE COUNSELLING CELL SYSTEM DOCUMENTATION - PART 2**

---

## Meeting and Communication Management

### 1. **CounsellingMeeting Model - Meeting Organization**

```python
class CounsellingMeeting(models.Model):
    meeting_host = models.ForeignKey(ExtraInfo,on_delete=models.CASCADE,null=True, blank=True)
    meeting_date = models.DateField(default=date.today)
    meeting_time = models.CharField(max_length=20, choices=CounsellingCellConstants.TIME)
    agenda = models.TextField()
    venue = models.CharField(max_length=20)
    student_invities = models.TextField(max_length=500,default=None)
```

#### **Field Analysis:**

#### **meeting_host** - Meeting Leadership
```python
meeting_host = models.ForeignKey(ExtraInfo,on_delete=models.CASCADE,null=True, blank=True)
```
**Meeting Authority:**
- Links to staff/faculty member hosting the meeting
- Enables accountability and meeting management
- Supports authorization and decision-making hierarchy
- Used for meeting coordination and follow-up responsibilities

#### **Temporal Planning:**
```python
meeting_date = models.DateField(default=date.today)
meeting_time = models.CharField(max_length=20, choices=CounsellingCellConstants.TIME)
```
**Schedule Management:**
- **meeting_date**: Specific date for the meeting
- **meeting_time**: Predefined time slots (10 AM - 9 PM)
- Enables systematic scheduling and calendar integration
- Supports conflict resolution and resource planning

#### **Meeting Content and Location:**
```python
agenda = models.TextField()
venue = models.CharField(max_length=20)
student_invities = models.TextField(max_length=500,default=None)
```
**Meeting Specification:**
- **agenda**: Detailed meeting agenda and objectives
- **venue**: Meeting location specification
- **student_invities**: Student participants and invitees
- Enables structured meeting planning and participant management

#### **Meeting Management Business Logic:**
```python
def schedule_counselling_meeting(host_id, meeting_date, meeting_time, agenda, venue, student_invites=None):
    """Schedule new counselling meeting"""
    
    # Validate meeting date
    if meeting_date < datetime.date.today():
        return False, "Cannot schedule meeting in the past"
    
    # Check venue availability
    if not is_venue_available(venue, meeting_date, meeting_time):
        return False, f"Venue {venue} is not available at {meeting_time} on {meeting_date}"
    
    # Check host availability
    if not is_host_available(host_id, meeting_date, meeting_time):
        return False, "Host is not available at the requested time"
    
    # Validate agenda
    if len(agenda.strip()) < 20:
        return False, "Please provide detailed agenda (minimum 20 characters)"
    
    # Create meeting
    meeting = CounsellingMeeting.objects.create(
        meeting_host_id=host_id,
        meeting_date=meeting_date,
        meeting_time=meeting_time,
        agenda=agenda,
        venue=venue,
        student_invities=student_invites or ""
    )
    
    # Send invitations
    send_meeting_invitations(meeting)
    
    # Create calendar entries
    create_calendar_entry(meeting)
    
    # Book venue
    book_venue(venue, meeting_date, meeting_time)
    
    return True, f"Meeting scheduled for {meeting_date} at {meeting_time}"

def is_venue_available(venue, meeting_date, meeting_time):
    """Check venue availability for requested time"""
    
    # Check for conflicting meetings
    conflicting_meetings = CounsellingMeeting.objects.filter(
        venue=venue,
        meeting_date=meeting_date,
        meeting_time=meeting_time
    )
    
    return not conflicting_meetings.exists()

def is_host_available(host_id, meeting_date, meeting_time):
    """Check host availability for requested time"""
    
    # Check for conflicting meetings hosted by same person
    conflicting_meetings = CounsellingMeeting.objects.filter(
        meeting_host=host_id,
        meeting_date=meeting_date,
        meeting_time=meeting_time
    )
    
    return not conflicting_meetings.exists()

def send_meeting_invitations(meeting):
    """Send meeting invitations to participants"""
    
    invitation_data = {
        'meeting_id': meeting.id,
        'host': meeting.meeting_host.user.get_full_name() if meeting.meeting_host else "Counselling Cell",
        'date': meeting.meeting_date,
        'time': dict(CounsellingCellConstants.TIME)[meeting.meeting_time],
        'venue': meeting.venue,
        'agenda': meeting.agenda
    }
    
    # Send to student invitees
    if meeting.student_invities:
        student_list = parse_student_invitees(meeting.student_invities)
        
        for student_id in student_list:
            send_student_meeting_invitation(student_id, invitation_data)
    
    # Send to counselling team members
    counselling_team = get_all_counselling_team_members()
    
    for member in counselling_team:
        send_team_meeting_invitation(member, invitation_data)

def parse_student_invitees(student_invities_text):
    """Parse student invitees from text format"""
    
    # Expected format: "student_id1, student_id2, student_id3" or "all_students"
    if student_invities_text.strip().lower() == 'all_students':
        # Get all students with counselling profiles
        all_students = StudentCounsellingInfo.objects.values_list('student_id', flat=True)
        return list(all_students)
    
    # Parse comma-separated student IDs
    student_ids = []
    for item in student_invities_text.split(','):
        item = item.strip()
        if item.isdigit():
            student_ids.append(int(item))
    
    return student_ids

def update_meeting_details(meeting_id, **updates):
    """Update meeting details"""
    
    try:
        meeting = CounsellingMeeting.objects.get(id=meeting_id)
        
        # Track changes for notifications
        changes = {}
        
        for field, new_value in updates.items():
            if hasattr(meeting, field):
                old_value = getattr(meeting, field)
                if old_value != new_value:
                    changes[field] = {'old': old_value, 'new': new_value}
                    setattr(meeting, field, new_value)
        
        meeting.save()
        
        # Notify participants of changes
        if changes:
            notify_meeting_changes(meeting, changes)
        
        return True, f"Meeting updated successfully"
        
    except CounsellingMeeting.DoesNotExist:
        return False, "Meeting not found"

def cancel_counselling_meeting(meeting_id, cancellation_reason=""):
    """Cancel scheduled meeting"""
    
    try:
        meeting = CounsellingMeeting.objects.get(id=meeting_id)
        
        # Check if meeting is in the future
        if meeting.meeting_date < datetime.date.today():
            return False, "Cannot cancel past meetings"
        
        if meeting.meeting_date == datetime.date.today():
            # Same day cancellation - check time
            current_time = datetime.datetime.now().hour
            meeting_hour = int(meeting.meeting_time)
            
            if current_time >= meeting_hour:
                return False, "Cannot cancel meeting on the same day after scheduled time"
        
        # Send cancellation notifications
        send_meeting_cancellation_notice(meeting, cancellation_reason)
        
        # Release venue booking
        release_venue_booking(meeting.venue, meeting.meeting_date, meeting.meeting_time)
        
        # Delete meeting
        meeting.delete()
        
        return True, "Meeting cancelled successfully"
        
    except CounsellingMeeting.DoesNotExist:
        return False, "Meeting not found"

def get_upcoming_meetings(days_ahead=30):
    """Get upcoming counselling meetings"""
    
    end_date = datetime.date.today() + datetime.timedelta(days=days_ahead)
    
    upcoming_meetings = CounsellingMeeting.objects.filter(
        meeting_date__gte=datetime.date.today(),
        meeting_date__lte=end_date
    ).order_by('meeting_date', 'meeting_time')
    
    return upcoming_meetings

def generate_meeting_schedule_report(start_date, end_date):
    """Generate comprehensive meeting schedule report"""
    
    meetings = CounsellingMeeting.objects.filter(
        meeting_date__range=[start_date, end_date]
    ).order_by('meeting_date', 'meeting_time')
    
    report = {
        'total_meetings': meetings.count(),
        'meetings_by_date': {},
        'venue_utilization': {},
        'host_activity': {},
        'student_participation': {}
    }
    
    # Group meetings by date
    for meeting in meetings:
        date_str = meeting.meeting_date.strftime('%Y-%m-%d')
        
        if date_str not in report['meetings_by_date']:
            report['meetings_by_date'][date_str] = []
        
        report['meetings_by_date'][date_str].append({
            'time': meeting.meeting_time,
            'venue': meeting.venue,
            'host': meeting.meeting_host.user.get_full_name() if meeting.meeting_host else "Not specified",
            'agenda': meeting.agenda[:100] + "..." if len(meeting.agenda) > 100 else meeting.agenda
        })
    
    # Venue utilization
    venue_usage = meetings.values('venue').annotate(
        usage_count=Count('id')
    ).order_by('-usage_count')
    
    for usage in venue_usage:
        report['venue_utilization'][usage['venue']] = usage['usage_count']
    
    # Host activity
    host_activity = meetings.filter(
        meeting_host__isnull=False
    ).values(
        'meeting_host__user__first_name',
        'meeting_host__user__last_name'
    ).annotate(
        meetings_hosted=Count('id')
    ).order_by('-meetings_hosted')
    
    for activity in host_activity:
        host_name = f"{activity['meeting_host__user__first_name']} {activity['meeting_host__user__last_name']}"
        report['host_activity'][host_name] = activity['meetings_hosted']
    
    return report
```

### 2. **CounsellingMinutes Model - Meeting Documentation**

```python
class CounsellingMinutes(models.Model):
    counselling_meeting = models.ForeignKey(CounsellingMeeting, on_delete=models.CASCADE)
    counselling_minutes = models.FileField(upload_to='counselling_cell/')
```

#### **Field Analysis:**

#### **counselling_meeting** - Meeting Reference
```python
counselling_meeting = models.ForeignKey(CounsellingMeeting, on_delete=models.CASCADE)
```
**Meeting Linkage:**
- Direct reference to specific counselling meeting
- Enables one-to-one meeting-minutes relationship
- Supports meeting documentation and historical tracking
- Used for follow-up and action item management

#### **counselling_minutes** - Document Storage
```python
counselling_minutes = models.FileField(upload_to='counselling_cell/')
```
**Documentation Management:**
- File upload for meeting minutes documents
- Supports various document formats (PDF, DOC, etc.)
- Enables comprehensive meeting record keeping
- Used for audit trail and reference documentation

#### **Minutes Management Business Logic:**
```python
def upload_meeting_minutes(meeting_id, minutes_file, uploader_id):
    """Upload meeting minutes document"""
    
    try:
        meeting = CounsellingMeeting.objects.get(id=meeting_id)
        
        # Check if meeting has already occurred
        if meeting.meeting_date > datetime.date.today():
            return False, "Cannot upload minutes for future meetings"
        
        # Check if minutes already exist
        existing_minutes = CounsellingMinutes.objects.filter(
            counselling_meeting=meeting
        ).first()
        
        if existing_minutes:
            return False, "Minutes already uploaded for this meeting"
        
        # Validate file
        if not is_valid_minutes_file(minutes_file):
            return False, "Invalid file format. Please upload PDF, DOC, or DOCX"
        
        # Create minutes record
        minutes = CounsellingMinutes.objects.create(
            counselling_meeting=meeting,
            counselling_minutes=minutes_file
        )
        
        # Extract and process action items
        action_items = extract_action_items_from_minutes(minutes_file)
        
        if action_items:
            create_action_item_tracking(meeting, action_items)
        
        # Notify meeting participants
        notify_minutes_uploaded(meeting, minutes)
        
        return True, "Meeting minutes uploaded successfully"
        
    except CounsellingMeeting.DoesNotExist:
        return False, "Meeting not found"

def is_valid_minutes_file(file):
    """Validate uploaded minutes file"""
    
    allowed_extensions = ['.pdf', '.doc', '.docx', '.txt']
    file_extension = file.name.lower().split('.')[-1]
    
    if f'.{file_extension}' not in allowed_extensions:
        return False
    
    # Check file size (max 10MB)
    if file.size > 10 * 1024 * 1024:
        return False
    
    return True

def extract_action_items_from_minutes(minutes_file):
    """Extract action items from uploaded minutes document"""
    
    # This would typically use text processing libraries
    # Simplified implementation for demonstration
    
    action_items = []
    
    try:
        # Read file content (implementation depends on file type)
        content = read_file_content(minutes_file)
        
        # Simple keyword-based extraction
        lines = content.split('\n')
        
        for line in lines:
            line_lower = line.lower().strip()
            
            # Look for action item indicators
            if any(keyword in line_lower for keyword in ['action:', 'todo:', 'follow-up:', 'action item:']):
                action_items.append({
                    'description': line.strip(),
                    'extracted_date': datetime.datetime.now(),
                    'status': 'pending'
                })
    
    except Exception as e:
        # Log error but don't fail the upload
        print(f"Error extracting action items: {e}")
    
    return action_items

def create_action_item_tracking(meeting, action_items):
    """Create tracking for action items from meeting"""
    
    for item in action_items:
        # This would create records in an ActionItem model (if it existed)
        # For now, we'll create a simple tracking mechanism
        
        action_tracking = {
            'meeting_id': meeting.id,
            'description': item['description'],
            'assigned_date': datetime.date.today(),
            'status': 'pending',
            'due_date': datetime.date.today() + datetime.timedelta(days=14)  # 2 weeks default
        }
        
        # Store action item (implementation would depend on specific model)
        store_action_item(action_tracking)

def get_meeting_minutes(meeting_id):
    """Retrieve meeting minutes for specific meeting"""
    
    try:
        meeting = CounsellingMeeting.objects.get(id=meeting_id)
        minutes = CounsellingMinutes.objects.filter(
            counselling_meeting=meeting
        ).first()
        
        if not minutes:
            return None, "No minutes found for this meeting"
        
        return minutes, "Minutes retrieved successfully"
        
    except CounsellingMeeting.DoesNotExist:
        return None, "Meeting not found"

def update_meeting_minutes(minutes_id, new_file):
    """Update existing meeting minutes"""
    
    try:
        minutes = CounsellingMinutes.objects.get(id=minutes_id)
        
        # Check if meeting is recent enough to allow updates
        days_since_meeting = (datetime.date.today() - minutes.counselling_meeting.meeting_date).days
        
        if days_since_meeting > 7:
            return False, "Cannot update minutes for meetings older than 7 days"
        
        # Validate new file
        if not is_valid_minutes_file(new_file):
            return False, "Invalid file format"
        
        # Backup old file (optional)
        backup_old_minutes(minutes.counselling_minutes)
        
        # Update minutes
        minutes.counselling_minutes = new_file
        minutes.save()
        
        # Notify participants of update
        notify_minutes_updated(minutes.counselling_meeting)
        
        return True, "Meeting minutes updated successfully"
        
    except CounsellingMinutes.DoesNotExist:
        return False, "Minutes record not found"

def generate_minutes_archive_report():
    """Generate report of all meeting minutes"""
    
    all_minutes = CounsellingMinutes.objects.select_related(
        'counselling_meeting'
    ).order_by('-counselling_meeting__meeting_date')
    
    archive_report = {
        'total_minutes': all_minutes.count(),
        'recent_minutes': [],
        'missing_minutes': [],
        'archive_summary': {}
    }
    
    # Recent minutes (last 30 days)
    recent_cutoff = datetime.date.today() - datetime.timedelta(days=30)
    
    for minutes in all_minutes:
        meeting = minutes.counselling_meeting
        
        if meeting.meeting_date >= recent_cutoff:
            archive_report['recent_minutes'].append({
                'meeting_date': meeting.meeting_date,
                'meeting_time': meeting.meeting_time,
                'venue': meeting.venue,
                'host': meeting.meeting_host.user.get_full_name() if meeting.meeting_host else "Not specified",
                'minutes_file': minutes.counselling_minutes.name
            })
    
    # Missing minutes (meetings without uploaded minutes)
    all_past_meetings = CounsellingMeeting.objects.filter(
        meeting_date__lt=datetime.date.today()
    )
    
    for meeting in all_past_meetings:
        has_minutes = CounsellingMinutes.objects.filter(
            counselling_meeting=meeting
        ).exists()
        
        if not has_minutes:
            archive_report['missing_minutes'].append({
                'meeting_date': meeting.meeting_date,
                'meeting_time': meeting.meeting_time,
                'venue': meeting.venue,
                'host': meeting.meeting_host.user.get_full_name() if meeting.meeting_host else "Not specified",
                'days_overdue': (datetime.date.today() - meeting.meeting_date).days
            })
    
    # Archive summary by month
    monthly_summary = all_minutes.extra(
        select={'month': "DATE_TRUNC('month', counselling_meeting__meeting_date)"}
    ).values('month').annotate(
        minutes_count=Count('id')
    ).order_by('-month')
    
    for summary in monthly_summary:
        month_str = summary['month'].strftime('%Y-%m') if summary['month'] else 'Unknown'
        archive_report['archive_summary'][month_str] = summary['minutes_count']
    
    return archive_report
```

### 3. **StudentMeetingRequest Model - Student-Initiated Meeting Requests**

```python
class StudentMeetingRequest(models.Model):
    requested_time = models.DateTimeField()
    student = models.ForeignKey(Student, on_delete=models.CASCADE)
    description = models.TextField(max_length=1000)
    requested_student_invitee = models.ForeignKey(StudentCounsellingTeam,on_delete=models.CASCADE,null=True, blank=True)
    requested_faculty_invitee = models.ForeignKey(FacultyCounsellingTeam,on_delete=models.CASCADE,null=True, blank=True)
    requested_meeting_status = models.CharField(max_length=20,choices=CounsellingCellConstants.MEETING_STATUS,default="status_pending")
    recipient_reply = models.TextField(max_length=1000)
```

#### **Field Analysis:**

#### **Request Details:**
```python
requested_time = models.DateTimeField()
student = models.ForeignKey(Student, on_delete=models.CASCADE)
description = models.TextField(max_length=1000)
```
**Meeting Request Specification:**
- **requested_time**: Preferred meeting date and time
- **student**: Student initiating the meeting request
- **description**: Detailed purpose and context for the meeting
- Enables student-driven meeting scheduling and support access

#### **Invitee Selection:**
```python
requested_student_invitee = models.ForeignKey(StudentCounsellingTeam,on_delete=models.CASCADE,null=True, blank=True)
requested_faculty_invitee = models.ForeignKey(FacultyCounsellingTeam,on_delete=models.CASCADE,null=True, blank=True)
```
**Target Participant:**
- **requested_student_invitee**: Specific student counsellor/guide requested
- **requested_faculty_invitee**: Specific faculty counsellor requested
- Enables targeted meeting requests and preference specification
- Supports both peer and professional counselling options

#### **Request Management:**
```python
requested_meeting_status = models.CharField(max_length=20,choices=CounsellingCellConstants.MEETING_STATUS,default="status_pending")
recipient_reply = models.TextField(max_length=1000)
```
**Approval Workflow:**
- **requested_meeting_status**: Request status (pending/accepted)
- **recipient_reply**: Response from requested counsellor
- Enables systematic request approval and communication
- Supports feedback and alternative suggestions

#### **Student Meeting Request Business Logic:**
```python
def submit_meeting_request(student_id, requested_time, description, student_invitee=None, faculty_invitee=None):
    """Submit student meeting request"""
    
    # Validate request time
    if requested_time <= datetime.datetime.now():
        return False, "Cannot request meeting in the past"
    
    # Check if too far in advance (max 30 days)
    max_advance_days = 30
    max_date = datetime.datetime.now() + datetime.timedelta(days=max_advance_days)
    
    if requested_time > max_date:
        return False, f"Cannot request meeting more than {max_advance_days} days in advance"
    
    # Validate description
    if len(description.strip()) < 20:
        return False, "Please provide detailed description (minimum 20 characters)"
    
    # Check for duplicate requests
    existing_request = StudentMeetingRequest.objects.filter(
        student_id=student_id,
        requested_time__date=requested_time.date(),
        requested_meeting_status='status_pending'
    ).first()
    
    if existing_request:
        return False, "You already have a pending meeting request for this date"
    
    # Validate invitee selection
    if not student_invitee and not faculty_invitee:
        # Auto-assign based on student's current guide
        try:
            student_info = StudentCounsellingInfo.objects.get(student_id=student_id)
            student_invitee = student_info.student_guide.id
        except StudentCounsellingInfo.DoesNotExist:
            # Assign to available counsellor
            available_counsellors = get_available_faculty_counsellors()
            if available_counsellors:
                faculty_invitee = available_counsellors[0]['counsellor'].id
            else:
                return False, "No counsellors available for meeting"
    
    # Create meeting request
    meeting_request = StudentMeetingRequest.objects.create(
        requested_time=requested_time,
        student_id=student_id,
        description=description,
        requested_student_invitee_id=student_invitee,
        requested_faculty_invitee_id=faculty_invitee
    )
    
    # Notify requested counsellor(s)
    notify_meeting_request(meeting_request)
    
    # Check for urgent requests
    if is_urgent_meeting_request(description):
        escalate_urgent_meeting_request(meeting_request)
    
    return True, f"Meeting request submitted successfully. Reference: REQ{meeting_request.id:06d}"

def process_meeting_request(request_id, action, reply_message="", alternative_time=None):
    """Process student meeting request"""
    
    try:
        meeting_request = StudentMeetingRequest.objects.get(id=request_id)
        
        if meeting_request.requested_meeting_status != 'status_pending':
            return False, "Request has already been processed"
        
        if action == 'accept':
            # Accept meeting request
            meeting_request.requested_meeting_status = 'status_accepted'
            meeting_request.recipient_reply = reply_message or "Meeting request accepted"
            
            # Create official meeting
            create_official_meeting_from_request(meeting_request)
            
        elif action == 'suggest_alternative':
            # Suggest alternative time
            if not alternative_time:
                return False, "Alternative time must be provided"
            
            meeting_request.recipient_reply = f"Alternative time suggested: {alternative_time}. {reply_message}"
            
            # Create alternative request
            create_alternative_meeting_request(meeting_request, alternative_time)
            
        elif action == 'decline':
            # Decline meeting request
            meeting_request.recipient_reply = reply_message or "Meeting request declined"
            
            # Suggest alternative support options
            suggest_alternative_support(meeting_request)
        
        meeting_request.save()
        
        # Notify student of decision
        notify_student_meeting_decision(meeting_request, action)
        
        return True, f"Meeting request {action}ed successfully"
        
    except StudentMeetingRequest.DoesNotExist:
        return False, "Meeting request not found"

def create_official_meeting_from_request(meeting_request):
    """Create official meeting from accepted request"""
    
    # Determine host
    if meeting_request.requested_faculty_invitee:
        host = meeting_request.requested_faculty_invitee.faculty.id
    elif meeting_request.requested_student_invitee:
        # For student guide meetings, assign to head counsellor as host
        head_counsellor = get_head_counsellor()
        host = head_counsellor.faculty.id if head_counsellor else None
    else:
        host = None
    
    # Find available venue
    venue = find_available_venue(meeting_request.requested_time)
    
    # Create meeting
    meeting = CounsellingMeeting.objects.create(
        meeting_host_id=host,
        meeting_date=meeting_request.requested_time.date(),
        meeting_time=str(meeting_request.requested_time.hour),
        agenda=f"Individual counselling session: {meeting_request.description}",
        venue=venue or "TBD",
        student_invities=str(meeting_request.student.id)
    )
    
    return meeting

def find_available_venue(requested_time):
    """Find available venue for meeting"""
    
    # List of available venues
    counselling_venues = ['Counselling Room 1', 'Counselling Room 2', 'Conference Room A']
    
    meeting_date = requested_time.date()
    meeting_hour = str(requested_time.hour)
    
    for venue in counselling_venues:
        # Check availability
        if is_venue_available(venue, meeting_date, meeting_hour):
            return venue
    
    return None

def is_urgent_meeting_request(description):
    """Determine if meeting request is urgent"""
    
    urgent_keywords = [
        'emergency', 'urgent', 'crisis', 'immediate',
        'depression', 'anxiety', 'stress', 'help'
    ]
    
    description_lower = description.lower()
    
    for keyword in urgent_keywords:
        if keyword in description_lower:
            return True
    
    return False

def escalate_urgent_meeting_request(meeting_request):
    """Escalate urgent meeting request to head counsellor"""
    
    head_counsellor = get_head_counsellor()
    if head_counsellor:
        # Update request to include head counsellor
        meeting_request.requested_faculty_invitee = head_counsellor
        meeting_request.save()
        
        # Send urgent notification
        send_urgent_meeting_notification(head_counsellor, meeting_request)
        
        # Try to schedule within 24 hours
        schedule_emergency_meeting(meeting_request)

def schedule_emergency_meeting(meeting_request):
    """Schedule emergency meeting within 24 hours"""
    
    # Find next available slot within 24 hours
    now = datetime.datetime.now()
    end_time = now + datetime.timedelta(hours=24)
    
    # Check hourly slots
    current_time = now.replace(minute=0, second=0, microsecond=0)
    
    while current_time < end_time:
        # Skip non-working hours (before 9 AM or after 6 PM)
        if 9 <= current_time.hour <= 18:
            meeting_time = str(current_time.hour)
            
            # Check if any counsellor is available
            for venue in ['Counselling Room 1', 'Emergency Room']:
                if is_venue_available(venue, current_time.date(), meeting_time):
                    # Create emergency meeting
                    emergency_meeting = CounsellingMeeting.objects.create(
                        meeting_host=meeting_request.requested_faculty_invitee.faculty if meeting_request.requested_faculty_invitee else None,
                        meeting_date=current_time.date(),
                        meeting_time=meeting_time,
                        agenda=f"URGENT: {meeting_request.description}",
                        venue=venue,
                        student_invities=str(meeting_request.student.id)
                    )
                    
                    # Update request status
                    meeting_request.requested_meeting_status = 'status_accepted'
                    meeting_request.recipient_reply = f"Emergency meeting scheduled for {current_time}"
                    meeting_request.save()
                    
                    # Notify student immediately
                    send_emergency_meeting_confirmation(meeting_request, emergency_meeting)
                    
                    return emergency_meeting
        
        current_time += datetime.timedelta(hours=1)
    
    return None

def get_student_meeting_history(student_id):
    """Get complete meeting request history for student"""
    
    requests = StudentMeetingRequest.objects.filter(
        student_id=student_id
    ).order_by('-requested_time')
    
    history = {
        'total_requests': requests.count(),
        'accepted_requests': requests.filter(requested_meeting_status='status_accepted').count(),
        'pending_requests': requests.filter(requested_meeting_status='status_pending').count(),
        'recent_requests': requests[:10],
        'meeting_frequency': calculate_meeting_frequency(requests)
    }
    
    return history

def calculate_meeting_frequency(requests):
    """Calculate meeting request frequency"""
    
    if not requests:
        return 0
    
    # Get first and last request dates
    first_request = requests.last().requested_time.date()
    last_request = requests.first().requested_time.date()
    
    # Calculate days between first and last request
    days_span = (last_request - first_request).days
    
    if days_span == 0:
        return requests.count()  # All requests on same day
    
    # Calculate requests per month
    requests_per_month = (requests.count() / max(days_span, 1)) * 30
    
    return round(requests_per_month, 2)

def generate_meeting_request_analytics(start_date, end_date):
    """Generate comprehensive meeting request analytics"""
    
    requests = StudentMeetingRequest.objects.filter(
        requested_time__range=[start_date, end_date]
    )
    
    analytics = {
        'total_requests': requests.count(),
        'accepted_requests': requests.filter(requested_meeting_status='status_accepted').count(),
        'pending_requests': requests.filter(requested_meeting_status='status_pending').count(),
        'request_trends': {},
        'popular_times': {},
        'counsellor_demand': {},
        'urgent_requests': 0
    }
    
    # Calculate acceptance rate
    if analytics['total_requests'] > 0:
        analytics['acceptance_rate'] = (analytics['accepted_requests'] / analytics['total_requests']) * 100
    else:
        analytics['acceptance_rate'] = 0
    
    # Request trends by day
    daily_requests = requests.extra(
        select={'day': "DATE_TRUNC('day', requested_time)"}
    ).values('day').annotate(
        request_count=Count('id')
    ).order_by('day')
    
    for daily in daily_requests:
        day_str = daily['day'].strftime('%Y-%m-%d') if daily['day'] else 'Unknown'
        analytics['request_trends'][day_str] = daily['request_count']
    
    # Popular meeting times
    time_preferences = requests.extra(
        select={'hour': "EXTRACT(hour FROM requested_time)"}
    ).values('hour').annotate(
        request_count=Count('id')
    ).order_by('-request_count')
    
    for time_pref in time_preferences:
        hour = int(time_pref['hour']) if time_pref['hour'] else 0
        time_str = f"{hour}:00"
        analytics['popular_times'][time_str] = time_pref['request_count']
    
    # Counsellor demand
    faculty_requests = requests.filter(
        requested_faculty_invitee__isnull=False
    ).values(
        'requested_faculty_invitee__faculty__id',
        'requested_faculty_invitee__faculty__first_name',
        'requested_faculty_invitee__faculty__last_name'
    ).annotate(
        request_count=Count('id')
    ).order_by('-request_count')
    
    for faculty_req in faculty_requests:
        faculty_name = f"{faculty_req['requested_faculty_invitee__faculty__first_name']} {faculty_req['requested_faculty_invitee__faculty__last_name']}"
        analytics['counsellor_demand'][faculty_name] = faculty_req['request_count']
    
    # Count urgent requests
    for request in requests:
        if is_urgent_meeting_request(request.description):
            analytics['urgent_requests'] += 1
    
    return analytics
```

---

## System Integration and Complete Workflow

### **Complete Counselling Cell Workflow Integration**

```python
def complete_student_support_workflow(student_id, support_type, details):
    """Complete end-to-end student support workflow"""
    
    workflow_log = []
    
    # Step 1: Determine support pathway
    if support_type == 'issue':
        # Issue submission workflow
        success, message = submit_counselling_issue(student_id, 'general', details)
        workflow_log.append(f"Issue submitted: {success} - {message}")
        
    elif support_type == 'meeting':
        # Meeting request workflow
        requested_time = datetime.datetime.now() + datetime.timedelta(days=2)
        success, message = submit_meeting_request(student_id, requested_time, details)
        workflow_log.append(f"Meeting requested: {success} - {message}")
        
    elif support_type == 'urgent':
        # Urgent support workflow
        success, message = submit_counselling_issue(student_id, 'emergency', details)
        if success:
            # Also submit urgent meeting request
            urgent_time = datetime.datetime.now() + datetime.timedelta(hours=2)
            submit_meeting_request(student_id, urgent_time, f"URGENT: {details}")
        workflow_log.append(f"Urgent support initiated: {success} - {message}")
    
    # Step 2: Notify support network
    try:
        student_info = StudentCounsellingInfo.objects.get(student_id=student_id)
        notify_student_guide(student_info.student_guide, support_type, details)
        workflow_log.append("Student guide notified")
    except StudentCounsellingInfo.DoesNotExist:
        # Create counselling profile
        create_student_counselling_profile(student_id)
        workflow_log.append("Counselling profile created")
    
    # Step 3: Provide immediate resources
    relevant_faqs = search_relevant_faqs(details)
    if relevant_faqs:
        workflow_log.append(f"Provided {len(relevant_faqs)} relevant FAQ resources")
    
    return True, workflow_log

def generate_comprehensive_counselling_report():
    """Generate comprehensive counselling system report"""
    
    today = datetime.date.today()
    last_month = today - datetime.timedelta(days=30)
    
    report = {
        'system_overview': {
            'total_students_enrolled': StudentCounsellingInfo.objects.count(),
            'active_student_guides': StudentCounsellingTeam.objects.filter(
                student_position='student_guide'
            ).count(),
            'faculty_counsellors': FacultyCounsellingTeam.objects.filter(
                faculty_position='faculty_counsellor'
            ).count(),
            'head_counsellor': FacultyCounsellingTeam.objects.filter(
                faculty_position='head_counsellor'
            ).exists()
        },
        'issue_analytics': generate_issue_analytics(last_month, today),
        'meeting_analytics': generate_meeting_request_analytics(last_month, today),
        'faq_analytics': get_faq_analytics(),
        'performance_metrics': calculate_system_performance_metrics(),
        'recommendations': generate_system_recommendations()
    }
    
    return report

def calculate_system_performance_metrics():
    """Calculate key performance indicators for counselling system"""
    
    last_30_days = datetime.datetime.now() - datetime.timedelta(days=30)
    
    metrics = {
        'issue_resolution_rate': 0,
        'average_response_time': 0,
        'student_satisfaction': 0,
        'counsellor_utilization': 0,
        'meeting_completion_rate': 0
    }
    
    # Issue resolution rate
    recent_issues = CounsellingIssue.objects.filter(
        issue_raised_date__gte=last_30_days
    )
    
    if recent_issues.count() > 0:
        resolved_issues = recent_issues.filter(issue_status='status_resolved')
        metrics['issue_resolution_rate'] = (resolved_issues.count() / recent_issues.count()) * 100
    
    # Average response time (simplified calculation)
    in_progress_issues = recent_issues.filter(issue_status='status_inprogress')
    if in_progress_issues.count() > 0:
        total_response_time = sum(
            (datetime.datetime.now() - issue.issue_raised_date).days
            for issue in in_progress_issues
        )
        metrics['average_response_time'] = total_response_time / in_progress_issues.count()
    
    # Meeting completion rate
    recent_meetings = CounsellingMeeting.objects.filter(
        meeting_date__gte=last_30_days.date()
    )
    
    if recent_meetings.count() > 0:
        completed_meetings = CounsellingMinutes.objects.filter(
            counselling_meeting__in=recent_meetings
        ).count()
        metrics['meeting_completion_rate'] = (completed_meetings / recent_meetings.count()) * 100
    
    # Counsellor utilization
    faculty_counsellors = FacultyCounsellingTeam.objects.filter(
        faculty_position='faculty_counsellor'
    )
    
    if faculty_counsellors.count() > 0:
        total_active_cases = sum(
            CounsellingIssue.objects.filter(
                resolved_by=counsellor.faculty.id,
                issue_status='status_inprogress'
            ).count()
            for counsellor in faculty_counsellors
        )
        
        # Assume ideal caseload is 10 per counsellor
        ideal_total_cases = faculty_counsellors.count() * 10
        metrics['counsellor_utilization'] = min((total_active_cases / ideal_total_cases) * 100, 100)
    
    return metrics

def generate_system_recommendations():
    """Generate system improvement recommendations"""
    
    recommendations = []
    
    # Check guide-to-student ratio
    total_students = StudentCounsellingInfo.objects.count()
    total_guides = StudentCounsellingTeam.objects.filter(
        student_position='student_guide'
    ).count()
    
    if total_guides > 0:
        ratio = total_students / total_guides
        if ratio > 20:  # More than 20 students per guide
            recommendations.append({
                'priority': 'high',
                'area': 'staffing',
                'recommendation': f'Consider recruiting more student guides. Current ratio: {ratio:.1f} students per guide'
            })
    
    # Check FAQ coverage
    faq_analytics = get_faq_analytics()
    for gap in faq_analytics.get('gaps_analysis', []):
        if gap['gap_score'] > 5:
            recommendations.append({
                'priority': 'medium',
                'area': 'knowledge_base',
                'recommendation': f'Create more FAQs for {gap["category"]} category. {gap["issues"]} issues but only {gap["faqs"]} FAQs'
            })
    
    # Check meeting venue utilization
    upcoming_meetings = get_upcoming_meetings(7)  # Next 7 days
    if upcoming_meetings.count() > 50:  # Assuming capacity issues
        recommendations.append({
            'priority': 'medium',
            'area': 'infrastructure',
            'recommendation': 'Consider adding more meeting venues. High meeting demand detected'
        })
    
    # Check urgent issue handling
    recent_urgent_issues = CounsellingIssue.objects.filter(
        issue_raised_date__gte=datetime.datetime.now() - datetime.timedelta(days=7)
    )
    
    urgent_count = sum(
        1 for issue in recent_urgent_issues
        if is_urgent_issue(issue.issue)
    )
    
    if urgent_count > 5:  # More than 5 urgent issues per week
        recommendations.append({
            'priority': 'high',
            'area': 'crisis_management',
            'recommendation': 'High number of urgent issues detected. Consider reviewing crisis intervention protocols'
        })
    
    return recommendations

def optimize_counsellor_assignments():
    """Optimize counsellor workload distribution"""
    
    # Get all active issues
    active_issues = CounsellingIssue.objects.filter(
        issue_status='status_inprogress'
    )
    
    # Get counsellor workloads
    counsellor_workloads = {}
    
    for issue in active_issues:
        counsellor_id = issue.resolved_by_id
        if counsellor_id:
            if counsellor_id not in counsellor_workloads:
                counsellor_workloads[counsellor_id] = []
            counsellor_workloads[counsellor_id].append(issue)
    
    # Identify overloaded and underutilized counsellors
    optimization_suggestions = []
    
    average_caseload = len(active_issues) / max(len(counsellor_workloads), 1)
    
    for counsellor_id, cases in counsellor_workloads.items():
        caseload = len(cases)
        
        if caseload > average_caseload * 1.5:  # 50% above average
            optimization_suggestions.append({
                'counsellor_id': counsellor_id,
                'current_caseload': caseload,
                'suggestion': 'redistribute_cases',
                'priority': 'high'
            })
        elif caseload < average_caseload * 0.5:  # 50% below average
            optimization_suggestions.append({
                'counsellor_id': counsellor_id,
                'current_caseload': caseload,
                'suggestion': 'assign_more_cases',
                'priority': 'medium'
            })
    
    return optimization_suggestions
```

---

## **COUNSELLING CELL SYSTEM SUMMARY**

The Counselling Cell system provides comprehensive student mental health and academic support through a multi-tiered approach:

### **Team Management Models (3):**
1. **FacultyCounsellingTeam** - Professional counsellor management with role hierarchy
2. **StudentCounsellingTeam** - Peer support team with student guides and coordinators
3. **StudentCounsellingInfo** - Student-guide assignment and support tracking

### **Issue and Knowledge Management Models (3):**
1. **CounsellingIssueCategory** - Issue classification and categorization system
2. **CounsellingIssue** - Core issue tracking with resolution workflow
3. **CounsellingFAQ** - Knowledge base for self-service support

### **Meeting and Communication Models (3):**
1. **CounsellingMeeting** - Meeting organization and scheduling
2. **CounsellingMinutes** - Meeting documentation and record keeping
3. **StudentMeetingRequest** - Student-initiated meeting request workflow

### **Key System Features:**
- **Hierarchical Support Structure** with peer and professional counselling tiers
- **Comprehensive Issue Management** with categorization, routing, and resolution tracking
- **Knowledge Base System** with FAQ management and intelligent suggestions
- **Meeting Coordination** with scheduling, documentation, and follow-up
- **Student-Driven Services** with meeting requests and preference-based routing
- **Crisis Management** with urgent issue detection and emergency scheduling
- **Performance Analytics** with utilization tracking and optimization recommendations
- **Workflow Automation** with auto-assignment, escalation, and notification systems

This system ensures comprehensive student welfare support from peer guidance through professional counselling, with built-in quality assurance, performance monitoring, and continuous improvement mechanisms.