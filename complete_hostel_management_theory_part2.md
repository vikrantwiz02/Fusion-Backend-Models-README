# Complete Theoretical Documentation: Hostel Management System - Part 2

## **COMPREHENSIVE HOSTEL MANAGEMENT SYSTEM DOCUMENTATION - PART 2**

---

## Operations and Service Management

### 5. **StaffSchedule Model - Staff Work Management**

```python
class StaffSchedule(models.Model):
    hall = models.ForeignKey(Hall, on_delete=models.CASCADE)   
    staff_id = models.ForeignKey(Staff, on_delete=models.CASCADE)
    staff_type = models.CharField(max_length=100, default='Caretaker')
    day = models.CharField(max_length=15, choices=HostelManagementConstants.DAYS_OF_WEEK)
    start_time = models.TimeField(null=True,blank=True)
    end_time = models.TimeField(null=True,blank=True)
```

#### **Field Analysis:**

#### **Location and Personnel:**
```python
hall = models.ForeignKey(Hall, on_delete=models.CASCADE)   
staff_id = models.ForeignKey(Staff, on_delete=models.CASCADE)
staff_type = models.CharField(max_length=100, default='Caretaker')
```
**Staff Assignment:**
- **hall**: Specific hall assignment for staff member
- **staff_id**: Reference to staff member in schedule
- **staff_type**: Type of staff role (Caretaker, Security, Maintenance, etc.)
- Enables hall-specific workforce management and role-based scheduling

#### **Schedule Configuration:**
```python
day = models.CharField(max_length=15, choices=HostelManagementConstants.DAYS_OF_WEEK)
start_time = models.TimeField(null=True,blank=True)
end_time = models.TimeField(null=True,blank=True)
```
**Work Schedule:**
- **day**: Day of week for the schedule
- **start_time/end_time**: Work shift hours
- Enables flexible scheduling and shift management
- Supports coverage planning and staff rotation

#### **Staff Schedule Business Logic:**
```python
def create_staff_schedule(hall_id, staff_id, staff_type, day, start_time, end_time):
    """Create new staff schedule entry"""
    
    try:
        hall = Hall.objects.get(hall_id=hall_id)
        staff = Staff.objects.get(id=staff_id)
        
        # Validate time inputs
        if start_time >= end_time:
            return False, "Start time must be before end time"
        
        # Check for schedule conflicts
        existing_schedule = StaffSchedule.objects.filter(
            staff_id=staff,
            day=day
        ).first()
        
        if existing_schedule:
            # Check for time overlap
            if (start_time < existing_schedule.end_time and 
                end_time > existing_schedule.start_time):
                return False, f"Schedule conflicts with existing schedule for {day}"
        
        # Validate maximum working hours per day (8 hours)
        work_duration = datetime.datetime.combine(datetime.date.today(), end_time) - \
                       datetime.datetime.combine(datetime.date.today(), start_time)
        
        if work_duration.seconds > 8 * 3600:  # 8 hours in seconds
            return False, "Maximum working hours per day is 8 hours"
        
        # Check total weekly hours for staff member
        weekly_hours = calculate_weekly_hours(staff_id, day, work_duration.seconds / 3600)
        
        if weekly_hours > 48:  # Maximum 48 hours per week
            return False, f"Schedule would exceed maximum weekly hours (48). Current: {weekly_hours - work_duration.seconds / 3600:.1f}"
        
        # Create schedule
        schedule = StaffSchedule.objects.create(
            hall=hall,
            staff_id=staff,
            staff_type=staff_type,
            day=day,
            start_time=start_time,
            end_time=end_time
        )
        
        # Update hall coverage analysis
        update_hall_coverage_analysis(hall_id)
        
        # Notify staff of schedule assignment
        notify_staff_schedule_assignment(staff, hall, day, start_time, end_time)
        
        return True, f"Schedule created for {staff.id.user.get_full_name()} on {day}"
        
    except Hall.DoesNotExist:
        return False, "Hall not found"
    except Staff.DoesNotExist:
        return False, "Staff member not found"

def calculate_weekly_hours(staff_id, exclude_day=None, additional_hours=0):
    """Calculate total weekly working hours for staff member"""
    
    schedules = StaffSchedule.objects.filter(staff_id=staff_id)
    
    if exclude_day:
        schedules = schedules.exclude(day=exclude_day)
    
    total_hours = additional_hours
    
    for schedule in schedules:
        if schedule.start_time and schedule.end_time:
            duration = datetime.datetime.combine(datetime.date.today(), schedule.end_time) - \
                      datetime.datetime.combine(datetime.date.today(), schedule.start_time)
            total_hours += duration.seconds / 3600
    
    return total_hours

def generate_weekly_staff_schedule(hall_id=None):
    """Generate comprehensive weekly staff schedule"""
    
    if hall_id:
        schedules = StaffSchedule.objects.filter(hall__hall_id=hall_id).select_related('hall', 'staff_id')
        halls = [Hall.objects.get(hall_id=hall_id)]
    else:
        schedules = StaffSchedule.objects.all().select_related('hall', 'staff_id')
        halls = Hall.objects.all()
    
    weekly_schedule = {}
    
    # Initialize schedule structure
    days = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday']
    
    for hall in halls:
        weekly_schedule[hall.hall_id] = {
            'hall_name': hall.hall_name,
            'daily_schedule': {}
        }
        
        for day in days:
            weekly_schedule[hall.hall_id]['daily_schedule'][day] = {
                'staff_assignments': [],
                'coverage_hours': 0,
                'staff_types': set()
            }
    
    # Populate schedule data
    for schedule in schedules:
        hall_id = schedule.hall.hall_id
        day = schedule.day
        
        if schedule.start_time and schedule.end_time:
            duration = datetime.datetime.combine(datetime.date.today(), schedule.end_time) - \
                      datetime.datetime.combine(datetime.date.today(), schedule.start_time)
            hours = duration.seconds / 3600
        else:
            hours = 0
        
        staff_assignment = {
            'staff_name': schedule.staff_id.id.user.get_full_name(),
            'staff_type': schedule.staff_type,
            'start_time': schedule.start_time.strftime('%H:%M') if schedule.start_time else 'TBD',
            'end_time': schedule.end_time.strftime('%H:%M') if schedule.end_time else 'TBD',
            'duration_hours': hours
        }
        
        weekly_schedule[hall_id]['daily_schedule'][day]['staff_assignments'].append(staff_assignment)
        weekly_schedule[hall_id]['daily_schedule'][day]['coverage_hours'] += hours
        weekly_schedule[hall_id]['daily_schedule'][day]['staff_types'].add(schedule.staff_type)
    
    # Convert sets to lists for JSON serialization
    for hall_id in weekly_schedule:
        for day in weekly_schedule[hall_id]['daily_schedule']:
            weekly_schedule[hall_id]['daily_schedule'][day]['staff_types'] = \
                list(weekly_schedule[hall_id]['daily_schedule'][day]['staff_types'])
    
    return weekly_schedule

def optimize_staff_schedules():
    """Optimize staff schedules for better coverage and efficiency"""
    
    all_halls = Hall.objects.all()
    optimization_suggestions = []
    
    for hall in all_halls:
        hall_schedules = StaffSchedule.objects.filter(hall=hall)
        coverage_analysis = analyze_hall_coverage(hall.hall_id)
        
        # Check for coverage gaps
        for day, coverage in coverage_analysis.items():
            if coverage['total_hours'] < 16:  # Less than 16 hours coverage
                optimization_suggestions.append({
                    'hall': hall.hall_name,
                    'issue': 'insufficient_coverage',
                    'day': day,
                    'current_hours': coverage['total_hours'],
                    'recommendation': f"Add {16 - coverage['total_hours']:.1f} hours of coverage for {day}"
                })
            
            if len(coverage['staff_types']) < 2:  # Need diverse staff types
                optimization_suggestions.append({
                    'hall': hall.hall_name,
                    'issue': 'limited_staff_diversity',
                    'day': day,
                    'current_types': coverage['staff_types'],
                    'recommendation': f"Add different staff types for {day}"
                })
        
        # Check for overtime risks
        staff_workloads = calculate_hall_staff_workloads(hall.hall_id)
        
        for staff_id, workload in staff_workloads.items():
            if workload['weekly_hours'] > 45:
                optimization_suggestions.append({
                    'hall': hall.hall_name,
                    'issue': 'potential_overtime',
                    'staff': workload['staff_name'],
                    'current_hours': workload['weekly_hours'],
                    'recommendation': f"Reduce hours for {workload['staff_name']} to prevent overtime"
                })
    
    return optimization_suggestions

def generate_staff_performance_schedule_report(staff_id):
    """Generate performance report based on schedule adherence"""
    
    try:
        staff = Staff.objects.get(id=staff_id)
        schedules = StaffSchedule.objects.filter(staff_id=staff)
        
        performance_data = {
            'staff_name': staff.id.user.get_full_name(),
            'total_scheduled_hours': 0,
            'halls_assigned': [],
            'weekly_distribution': {},
            'workload_analysis': {},
            'schedule_efficiency': {}
        }
        
        # Calculate total scheduled hours
        total_hours = 0
        hall_assignments = {}
        
        for schedule in schedules:
            if schedule.start_time and schedule.end_time:
                duration = datetime.datetime.combine(datetime.date.today(), schedule.end_time) - \
                          datetime.datetime.combine(datetime.date.today(), schedule.start_time)
                hours = duration.seconds / 3600
                total_hours += hours
                
                hall_name = schedule.hall.hall_name
                if hall_name not in hall_assignments:
                    hall_assignments[hall_name] = []
                
                hall_assignments[hall_name].append({
                    'day': schedule.day,
                    'hours': hours,
                    'staff_type': schedule.staff_type
                })
        
        performance_data['total_scheduled_hours'] = total_hours
        performance_data['halls_assigned'] = list(hall_assignments.keys())
        
        # Weekly distribution
        days = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday']
        for day in days:
            day_schedules = schedules.filter(day=day)
            day_hours = sum(
                (datetime.datetime.combine(datetime.date.today(), s.end_time) - 
                 datetime.datetime.combine(datetime.date.today(), s.start_time)).seconds / 3600
                for s in day_schedules if s.start_time and s.end_time
            )
            performance_data['weekly_distribution'][day] = day_hours
        
        # Workload analysis
        performance_data['workload_analysis'] = {
            'is_overloaded': total_hours > 45,
            'is_underutilized': total_hours < 20,
            'optimal_range': 20 <= total_hours <= 45,
            'efficiency_score': min(100, (total_hours / 40) * 100)  # Based on 40-hour optimal week
        }
        
        return performance_data
        
    except Staff.DoesNotExist:
        return None
```

### 6. **HostelNoticeBoard Model - Communication Management**

```python
class HostelNoticeBoard(models.Model):
    hall = models.ForeignKey(Hall, on_delete=models.CASCADE)
    posted_by = models.ForeignKey(ExtraInfo, on_delete=models.ForeignKey)
    head_line = models.CharField(max_length=100)
    content = models.FileField(upload_to='hostel_management/', blank=True, null=True)
    description = models.TextField(blank=True)
```

#### **Field Analysis:**

#### **Location and Authority:**
```python
hall = models.ForeignKey(Hall, on_delete=models.CASCADE)
posted_by = models.ForeignKey(ExtraInfo, on_delete=models.ForeignKey)
```
**Notice Management:**
- **hall**: Target hall for the notice
- **posted_by**: Authority posting the notice (warden, caretaker, admin)
- Enables hall-specific communication
- Supports hierarchical notice authority and accountability

#### **Notice Content:**
```python
head_line = models.CharField(max_length=100)
content = models.FileField(upload_to='hostel_management/', blank=True, null=True)
description = models.TextField(blank=True)
```
**Content Management:**
- **head_line**: Notice title and summary
- **content**: File attachments (documents, images)
- **description**: Detailed notice description
- Enables comprehensive communication with multimedia support

#### **Notice Board Business Logic:**
```python
def post_hostel_notice(hall_id, posted_by_id, headline, description, content_file=None, notice_type='general'):
    """Post new notice on hostel notice board"""
    
    try:
        hall = Hall.objects.get(hall_id=hall_id)
        posted_by = ExtraInfo.objects.get(id=posted_by_id)
        
        # Validate posting authority
        if not validate_notice_posting_authority(posted_by, hall):
            return False, "You don't have permission to post notices for this hall"
        
        # Validate content
        if len(headline.strip()) < 5:
            return False, "Notice headline must be at least 5 characters"
        
        if len(description.strip()) < 20:
            return False, "Notice description must be at least 20 characters"
        
        # Validate file if provided
        if content_file and not validate_notice_file(content_file):
            return False, "Invalid file format. Allowed: PDF, DOC, DOCX, JPG, PNG"
        
        # Create notice
        notice = HostelNoticeBoard.objects.create(
            hall=hall,
            posted_by=posted_by,
            head_line=headline,
            description=description,
            content=content_file
        )
        
        # Categorize notice
        categorize_notice(notice, notice_type)
        
        # Send notifications to hall residents
        notify_hall_residents(hall, notice)
        
        # Log notice posting
        log_notice_activity(notice, 'posted')
        
        return True, f"Notice posted successfully for {hall.hall_name}"
        
    except Hall.DoesNotExist:
        return False, "Hall not found"
    except ExtraInfo.DoesNotExist:
        return False, "User not found"

def validate_notice_posting_authority(posted_by, hall):
    """Validate if user has authority to post notices for hall"""
    
    # Check if user is hall warden
    is_warden = HallWarden.objects.filter(hall=hall, faculty=posted_by).exists()
    
    # Check if user is hall caretaker
    is_caretaker = HallCaretaker.objects.filter(hall=hall, staff=posted_by).exists()
    
    # Check if user is admin (you would need to define admin roles)
    is_admin = check_admin_privileges(posted_by)
    
    return is_warden or is_caretaker or is_admin

def get_hall_notices(hall_id, limit=10, notice_type=None, active_only=True):
    """Retrieve notices for specific hall"""
    
    try:
        hall = Hall.objects.get(hall_id=hall_id)
        
        notices = HostelNoticeBoard.objects.filter(hall=hall)
        
        # Filter by type if specified
        if notice_type:
            notices = notices.filter(notice_type=notice_type)
        
        # Filter active notices only
        if active_only:
            # Assuming notices are active for 30 days by default
            cutoff_date = datetime.datetime.now() - datetime.timedelta(days=30)
            notices = notices.filter(posted_date__gte=cutoff_date)
        
        # Order by most recent first
        notices = notices.order_by('-id')[:limit]
        
        notice_list = []
        
        for notice in notices:
            notice_data = {
                'id': notice.id,
                'headline': notice.head_line,
                'description': notice.description,
                'posted_by': notice.posted_by.user.get_full_name(),
                'posted_by_role': get_user_role_in_hall(notice.posted_by, hall),
                'has_attachment': bool(notice.content),
                'attachment_url': notice.content.url if notice.content else None,
                'posted_date': notice.id  # Assuming auto-increment ID represents chronological order
            }
            
            notice_list.append(notice_data)
        
        return notice_list
        
    except Hall.DoesNotExist:
        return []

def update_notice(notice_id, updated_by_id, **updates):
    """Update existing notice"""
    
    try:
        notice = HostelNoticeBoard.objects.get(id=notice_id)
        updated_by = ExtraInfo.objects.get(id=updated_by_id)
        
        # Check permission to update
        if notice.posted_by != updated_by:
            # Check if updater has authority over the hall
            if not validate_notice_posting_authority(updated_by, notice.hall):
                return False, "You don't have permission to update this notice"
        
        # Track changes
        changes = {}
        
        for field, new_value in updates.items():
            if hasattr(notice, field):
                old_value = getattr(notice, field)
                if old_value != new_value:
                    changes[field] = {'old': old_value, 'new': new_value}
                    setattr(notice, field, new_value)
        
        if changes:
            notice.save()
            
            # Log the update
            log_notice_update(notice, updated_by, changes)
            
            # Notify if significant changes
            if 'head_line' in changes or 'description' in changes:
                notify_notice_update(notice)
        
        return True, "Notice updated successfully"
        
    except HostelNoticeBoard.DoesNotExist:
        return False, "Notice not found"
    except ExtraInfo.DoesNotExist:
        return False, "User not found"

def remove_notice(notice_id, removed_by_id, removal_reason=""):
    """Remove notice from notice board"""
    
    try:
        notice = HostelNoticeBoard.objects.get(id=notice_id)
        removed_by = ExtraInfo.objects.get(id=removed_by_id)
        
        # Check permission to remove
        if notice.posted_by != removed_by:
            if not validate_notice_posting_authority(removed_by, notice.hall):
                return False, "You don't have permission to remove this notice"
        
        # Archive notice before deletion
        archive_notice(notice, removed_by, removal_reason)
        
        # Log the removal
        log_notice_activity(notice, 'removed', removed_by, removal_reason)
        
        # Delete notice
        notice.delete()
        
        return True, "Notice removed successfully"
        
    except HostelNoticeBoard.DoesNotExist:
        return False, "Notice not found"
    except ExtraInfo.DoesNotExist:
        return False, "User not found"

def generate_notice_analytics(hall_id=None, start_date=None, end_date=None):
    """Generate analytics on notice board usage"""
    
    notices = HostelNoticeBoard.objects.all()
    
    if hall_id:
        notices = notices.filter(hall__hall_id=hall_id)
    
    if start_date and end_date:
        # This would require a posted_date field which doesn't exist in the model
        # Using ID as a proxy for chronological order
        start_id = estimate_id_from_date(start_date)
        end_id = estimate_id_from_date(end_date)
        notices = notices.filter(id__range=[start_id, end_id])
    
    analytics = {
        'total_notices': notices.count(),
        'notices_by_hall': {},
        'notices_by_poster': {},
        'notices_with_attachments': notices.exclude(content='').count(),
        'average_headline_length': 0,
        'most_active_posters': [],
        'content_analysis': {}
    }
    
    # Notices by hall
    hall_counts = notices.values('hall__hall_name').annotate(
        notice_count=Count('id')
    ).order_by('-notice_count')
    
    for hall in hall_counts:
        analytics['notices_by_hall'][hall['hall__hall_name']] = hall['notice_count']
    
    # Notices by poster
    poster_counts = notices.values(
        'posted_by__user__first_name',
        'posted_by__user__last_name'
    ).annotate(
        notice_count=Count('id')
    ).order_by('-notice_count')
    
    for poster in poster_counts:
        poster_name = f"{poster['posted_by__user__first_name']} {poster['posted_by__user__last_name']}"
        analytics['notices_by_poster'][poster_name] = poster['notice_count']
    
    # Average headline length
    if notices.count() > 0:
        headlines = notices.values_list('head_line', flat=True)
        total_length = sum(len(headline) for headline in headlines)
        analytics['average_headline_length'] = total_length / len(headlines)
    
    # Most active posters (top 5)
    analytics['most_active_posters'] = list(poster_counts[:5])
    
    return analytics
```

### 7. **HostelStudentAttendence Model - Attendance Tracking**

```python
class HostelStudentAttendence(models.Model):
    hall = models.ForeignKey(Hall, on_delete=models.CASCADE)
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    date = models.DateField()
    present = models.BooleanField()
```

#### **Field Analysis:**

#### **Location and Identity:**
```python
hall = models.ForeignKey(Hall, on_delete=models.CASCADE)
student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
```
**Attendance Tracking:**
- **hall**: Specific hall for attendance record
- **student_id**: Student being tracked
- Enables hall-specific attendance monitoring
- Supports student welfare and safety tracking

#### **Temporal and Status:**
```python
date = models.DateField()
present = models.BooleanField()
```
**Attendance Record:**
- **date**: Specific date for attendance
- **present**: Boolean indicator of presence
- Enables daily attendance tracking and historical analysis
- Supports safety protocols and welfare monitoring

#### **Student Attendance Business Logic:**
```python
def record_student_attendance(hall_id, student_id, attendance_date, is_present, recorded_by_id):
    """Record daily attendance for student"""
    
    try:
        hall = Hall.objects.get(hall_id=hall_id)
        student = Student.objects.get(id=student_id)
        recorded_by = ExtraInfo.objects.get(id=recorded_by_id)
        
        # Validate recording authority
        if not validate_attendance_recording_authority(recorded_by, hall):
            return False, "You don't have permission to record attendance for this hall"
        
        # Check if student belongs to this hall
        if not verify_student_hall_assignment(student, hall):
            return False, f"Student {student.id.user.get_full_name()} is not assigned to {hall.hall_name}"
        
        # Check if attendance already recorded for this date
        existing_record = HostelStudentAttendence.objects.filter(
            hall=hall,
            student_id=student,
            date=attendance_date
        ).first()
        
        if existing_record:
            # Update existing record
            old_status = existing_record.present
            existing_record.present = is_present
            existing_record.save()
            
            # Log the change
            log_attendance_change(existing_record, old_status, is_present, recorded_by)
            
            return True, f"Attendance updated for {student.id.user.get_full_name()}"
        else:
            # Create new attendance record
            attendance = HostelStudentAttendence.objects.create(
                hall=hall,
                student_id=student,
                date=attendance_date,
                present=is_present
            )
            
            # Check for consecutive absences
            check_consecutive_absences(student, hall, attendance_date)
            
            # Log the record
            log_attendance_record(attendance, recorded_by)
            
            return True, f"Attendance recorded for {student.id.user.get_full_name()}"
        
    except Hall.DoesNotExist:
        return False, "Hall not found"
    except Student.DoesNotExist:
        return False, "Student not found"
    except ExtraInfo.DoesNotExist:
        return False, "Recorder not found"

def check_consecutive_absences(student, hall, current_date):
    """Check for consecutive absences and trigger alerts"""
    
    # Check last 7 days for consecutive absences
    date_range = [current_date - datetime.timedelta(days=i) for i in range(7)]
    
    recent_attendance = HostelStudentAttendence.objects.filter(
        hall=hall,
        student_id=student,
        date__in=date_range
    ).order_by('-date')
    
    consecutive_absences = 0
    
    for attendance in recent_attendance:
        if not attendance.present:
            consecutive_absences += 1
        else:
            break
    
    # Trigger alerts based on absence count
    if consecutive_absences >= 3:
        trigger_absence_alert(student, hall, consecutive_absences, 'moderate')
    
    if consecutive_absences >= 5:
        trigger_absence_alert(student, hall, consecutive_absences, 'critical')
        
        # Notify warden and parents
        notify_extended_absence(student, hall, consecutive_absences)

def generate_attendance_report(hall_id, start_date, end_date):
    """Generate comprehensive attendance report for hall"""
    
    try:
        hall = Hall.objects.get(hall_id=hall_id)
        
        attendance_records = HostelStudentAttendence.objects.filter(
            hall=hall,
            date__range=[start_date, end_date]
        ).select_related('student_id')
        
        # Get all students in the hall
        hall_students = get_hall_students(hall_id)
        
        report = {
            'hall_name': hall.hall_name,
            'reporting_period': f"{start_date} to {end_date}",
            'total_students': len(hall_students),
            'total_days': (end_date - start_date).days + 1,
            'overall_statistics': {},
            'student_attendance': {},
            'daily_attendance': {},
            'alert_summary': {}
        }
        
        # Calculate overall statistics
        total_possible_attendance = len(hall_students) * report['total_days']
        total_present = attendance_records.filter(present=True).count()
        total_absent = attendance_records.filter(present=False).count()
        total_recorded = attendance_records.count()
        
        report['overall_statistics'] = {
            'overall_attendance_rate': (total_present / max(total_recorded, 1)) * 100,
            'total_present_days': total_present,
            'total_absent_days': total_absent,
            'total_recorded_days': total_recorded,
            'missing_records': total_possible_attendance - total_recorded
        }
        
        # Student-wise attendance
        for student in hall_students:
            student_records = attendance_records.filter(student_id=student.id)
            
            present_days = student_records.filter(present=True).count()
            total_days_recorded = student_records.count()
            
            if total_days_recorded > 0:
                attendance_rate = (present_days / total_days_recorded) * 100
            else:
                attendance_rate = 0
            
            report['student_attendance'][student.id.user.get_full_name()] = {
                'student_id': student.id.user.username,
                'present_days': present_days,
                'absent_days': total_days_recorded - present_days,
                'attendance_rate': attendance_rate,
                'total_recorded': total_days_recorded
            }
        
        # Daily attendance summary
        current_date = start_date
        while current_date <= end_date:
            daily_records = attendance_records.filter(date=current_date)
            
            present_count = daily_records.filter(present=True).count()
            absent_count = daily_records.filter(present=False).count()
            total_recorded = daily_records.count()
            
            report['daily_attendance'][current_date.strftime('%Y-%m-%d')] = {
                'present': present_count,
                'absent': absent_count,
                'total_recorded': total_recorded,
                'attendance_rate': (present_count / max(total_recorded, 1)) * 100
            }
            
            current_date += datetime.timedelta(days=1)
        
        # Alert summary
        alert_data = generate_attendance_alerts(hall_id, start_date, end_date)
        report['alert_summary'] = alert_data
        
        return report
        
    except Hall.DoesNotExist:
        return None

def bulk_record_attendance(hall_id, attendance_date, student_attendance_data, recorded_by_id):
    """Record attendance for multiple students at once"""
    
    try:
        hall = Hall.objects.get(hall_id=hall_id)
        recorded_by = ExtraInfo.objects.get(id=recorded_by_id)
        
        if not validate_attendance_recording_authority(recorded_by, hall):
            return False, "You don't have permission to record attendance for this hall"
        
        successful_records = 0
        failed_records = []
        
        for student_data in student_attendance_data:
            student_id = student_data['student_id']
            is_present = student_data['is_present']
            
            try:
                student = Student.objects.get(id=student_id)
                
                # Record or update attendance
                attendance, created = HostelStudentAttendence.objects.update_or_create(
                    hall=hall,
                    student_id=student,
                    date=attendance_date,
                    defaults={'present': is_present}
                )
                
                successful_records += 1
                
                # Check for consecutive absences if absent
                if not is_present:
                    check_consecutive_absences(student, hall, attendance_date)
                
            except Student.DoesNotExist:
                failed_records.append({
                    'student_id': student_id,
                    'error': 'Student not found'
                })
        
        # Log bulk attendance recording
        log_bulk_attendance_recording(hall, attendance_date, successful_records, len(failed_records), recorded_by)
        
        return True, {
            'successful_records': successful_records,
            'failed_records': failed_records,
            'message': f"Recorded attendance for {successful_records} students"
        }
        
    except Hall.DoesNotExist:
        return False, "Hall not found"
    except ExtraInfo.DoesNotExist:
        return False, "Recorder not found"

def generate_absentee_alert_system():
    """Generate system-wide absentee alerts"""
    
    yesterday = datetime.date.today() - datetime.timedelta(days=1)
    
    # Find all students who were absent yesterday
    absent_students = HostelStudentAttendence.objects.filter(
        date=yesterday,
        present=False
    ).select_related('student_id', 'hall')
    
    alerts = {
        'critical_alerts': [],    # 5+ consecutive absences
        'moderate_alerts': [],    # 3-4 consecutive absences
        'new_absences': [],       # First-time absences
        'summary': {
            'total_absent': absent_students.count(),
            'halls_affected': set(),
            'action_required': 0
        }
    }
    
    for attendance in absent_students:
        student = attendance.student_id
        hall = attendance.hall
        
        alerts['summary']['halls_affected'].add(hall.hall_name)
        
        # Check consecutive absence pattern
        consecutive_count = get_consecutive_absence_count(student, hall, yesterday)
        
        alert_data = {
            'student_name': student.id.user.get_full_name(),
            'student_id': student.id.user.username,
            'hall': hall.hall_name,
            'consecutive_absences': consecutive_count,
            'last_present_date': get_last_present_date(student, hall, yesterday)
        }
        
        if consecutive_count >= 5:
            alerts['critical_alerts'].append(alert_data)
            alerts['summary']['action_required'] += 1
        elif consecutive_count >= 3:
            alerts['moderate_alerts'].append(alert_data)
            alerts['summary']['action_required'] += 1
        else:
            alerts['new_absences'].append(alert_data)
    
    alerts['summary']['halls_affected'] = list(alerts['summary']['halls_affected'])
    
    return alerts
```

### 8. **HallRoom Model - Room Management**

```python
class HallRoom(models.Model):
    hall = models.ForeignKey(Hall, on_delete=models.CASCADE)
    room_no = models.CharField(max_length=4) 
    block_no = models.CharField(max_length=1)
    room_cap = models.IntegerField(default=3)
    room_occupied = models.IntegerField(default=0)
```

#### **Field Analysis:**

#### **Location Hierarchy:**
```python
hall = models.ForeignKey(Hall, on_delete=models.CASCADE)
room_no = models.CharField(max_length=4) 
block_no = models.CharField(max_length=1)
```
**Room Identification:**
- **hall**: Parent hall containing the room
- **room_no**: Specific room number (e.g., "101", "205")
- **block_no**: Block or wing identifier (A, B, C, D)
- Enables hierarchical room addressing and management

#### **Capacity Management:**
```python
room_cap = models.IntegerField(default=3)
room_occupied = models.IntegerField(default=0)
```
**Occupancy Control:**
- **room_cap**: Maximum students per room
- **room_occupied**: Current number of occupants
- Enables occupancy tracking and availability management
- Supports room allocation and capacity planning

#### **Hall Room Business Logic:**
```python
def create_hall_room(hall_id, room_no, block_no, room_capacity):
    """Create new room in hall"""
    
    try:
        hall = Hall.objects.get(hall_id=hall_id)
        
        # Validate room number uniqueness within hall and block
        existing_room = HallRoom.objects.filter(
            hall=hall,
            room_no=room_no,
            block_no=block_no
        ).first()
        
        if existing_room:
            return False, f"Room {block_no}-{room_no} already exists in {hall.hall_name}"
        
        # Validate capacity
        if room_capacity < 1 or room_capacity > 4:
            return False, "Room capacity must be between 1 and 4"
        
        # Create room
        room = HallRoom.objects.create(
            hall=hall,
            room_no=room_no,
            block_no=block_no,
            room_cap=room_capacity,
            room_occupied=0
        )
        
        # Update hall total capacity if needed
        update_hall_capacity_calculation(hall)
        
        # Create room amenities record
        initialize_room_amenities(room)
        
        return True, f"Room {block_no}-{room_no} created successfully"
        
    except Hall.DoesNotExist:
        return False, "Hall not found"

def allocate_student_to_room(hall_id, block_no, room_no, student_id):
    """Allocate student to specific room"""
    
    try:
        room = HallRoom.objects.get(
            hall__hall_id=hall_id,
            block_no=block_no,
            room_no=room_no
        )
        student = Student.objects.get(id=student_id)
        
        # Check room availability
        if room.room_occupied >= room.room_cap:
            return False, f"Room {block_no}-{room_no} is at full capacity"
        
        # Check if student is already allocated to a room in this hall
        existing_allocation = check_student_room_allocation(student, room.hall)
        
        if existing_allocation:
            return False, f"Student is already allocated to room {existing_allocation}"
        
        # Check if student belongs to correct batch for this hall
        if room.hall.assigned_batch:
            student_batch = get_student_batch(student)
            if student_batch != room.hall.assigned_batch:
                return False, f"Student batch {student_batch} doesn't match hall batch {room.hall.assigned_batch}"
        
        # Allocate student
        room.room_occupied += 1
        room.save()
        
        # Create allocation record
        create_student_room_allocation(student, room)
        
        # Update hall student count
        update_hall_student_count(room.hall.hall_id, 'increment', 1)
        
        # Notify student of room allocation
        notify_room_allocation(student, room)
        
        # Create welcome packet
        generate_student_welcome_packet(student, room)
        
        return True, f"Student {student.id.user.get_full_name()} allocated to room {block_no}-{room_no}"
        
    except HallRoom.DoesNotExist:
        return False, "Room not found"
    except Student.DoesNotExist:
        return False, "Student not found"

def deallocate_student_from_room(hall_id, block_no, room_no, student_id, reason=""):
    """Remove student from room allocation"""
    
    try:
        room = HallRoom.objects.get(
            hall__hall_id=hall_id,
            block_no=block_no,
            room_no=room_no
        )
        student = Student.objects.get(id=student_id)
        
        # Verify student is actually in this room
        allocation = verify_student_in_room(student, room)
        
        if not allocation:
            return False, f"Student is not allocated to room {block_no}-{room_no}"
        
        # Remove allocation
        room.room_occupied = max(0, room.room_occupied - 1)
        room.save()
        
        # Remove allocation record
        remove_student_room_allocation(student, room)
        
        # Update hall student count
        update_hall_student_count(room.hall.hall_id, 'decrement', 1)
        
        # Log deallocation
        log_room_deallocation(student, room, reason)
        
        # Notify student of deallocation
        notify_room_deallocation(student, room, reason)
        
        # Check if room maintenance is needed
        schedule_room_inspection(room)
        
        return True, f"Student {student.id.user.get_full_name()} deallocated from room {block_no}-{room_no}"
        
    except HallRoom.DoesNotExist:
        return False, "Room not found"
    except Student.DoesNotExist:
        return False, "Student not found"

def get_room_availability_status(hall_id=None):
    """Get comprehensive room availability status"""
    
    if hall_id:
        rooms = HallRoom.objects.filter(hall__hall_id=hall_id)
        halls = [Hall.objects.get(hall_id=hall_id)]
    else:
        rooms = HallRoom.objects.all()
        halls = Hall.objects.all()
    
    availability_report = {
        'summary': {
            'total_rooms': rooms.count(),
            'available_rooms': 0,
            'fully_occupied_rooms': 0,
            'partially_occupied_rooms': 0,
            'total_capacity': 0,
            'total_occupied': 0
        },
        'hall_breakdown': {},
        'block_breakdown': {},
        'room_details': []
    }
    
    # Calculate summary statistics
    for room in rooms:
        availability_report['summary']['total_capacity'] += room.room_cap
        availability_report['summary']['total_occupied'] += room.room_occupied
        
        if room.room_occupied == 0:
            availability_report['summary']['available_rooms'] += 1
        elif room.room_occupied == room.room_cap:
            availability_report['summary']['fully_occupied_rooms'] += 1
        else:
            availability_report['summary']['partially_occupied_rooms'] += 1
    
    # Hall breakdown
    for hall in halls:
        hall_rooms = rooms.filter(hall=hall)
        
        hall_stats = {
            'hall_name': hall.hall_name,
            'total_rooms': hall_rooms.count(),
            'available_spaces': 0,
            'total_capacity': 0,
            'occupancy_rate': 0,
            'block_distribution': {}
        }
        
        for room in hall_rooms:
            available_spaces = room.room_cap - room.room_occupied
            hall_stats['available_spaces'] += available_spaces
            hall_stats['total_capacity'] += room.room_cap
            
            # Block distribution
            if room.block_no not in hall_stats['block_distribution']:
                hall_stats['block_distribution'][room.block_no] = {
                    'rooms': 0,
                    'capacity': 0,
                    'occupied': 0,
                    'available': 0
                }
            
            hall_stats['block_distribution'][room.block_no]['rooms'] += 1
            hall_stats['block_distribution'][room.block_no]['capacity'] += room.room_cap
            hall_stats['block_distribution'][room.block_no]['occupied'] += room.room_occupied
            hall_stats['block_distribution'][room.block_no]['available'] += available_spaces
        
        if hall_stats['total_capacity'] > 0:
            hall_stats['occupancy_rate'] = ((hall_stats['total_capacity'] - hall_stats['available_spaces']) / hall_stats['total_capacity']) * 100
        
        availability_report['hall_breakdown'][hall.hall_id] = hall_stats
    
    # Detailed room information
    for room in rooms.select_related('hall'):
        room_detail = {
            'hall_id': room.hall.hall_id,
            'hall_name': room.hall.hall_name,
            'block': room.block_no,
            'room_number': room.room_no,
            'capacity': room.room_cap,
            'occupied': room.room_occupied,
            'available_spaces': room.room_cap - room.room_occupied,
            'occupancy_rate': (room.room_occupied / room.room_cap) * 100,
            'status': get_room_status(room)
        }
        
        availability_report['room_details'].append(room_detail)
    
    return availability_report

def get_room_status(room):
    """Determine room status based on occupancy"""
    
    if room.room_occupied == 0:
        return "Available"
    elif room.room_occupied == room.room_cap:
        return "Full"
    else:
        return "Partially Occupied"

def optimize_room_allocation(hall_id):
    """Suggest optimal room allocation strategies"""
    
    try:
        hall = Hall.objects.get(hall_id=hall_id)
        rooms = HallRoom.objects.filter(hall=hall)
        
        optimization_report = {
            'current_utilization': 0,
            'optimal_utilization': 0,
            'recommendations': [],
            'potential_improvements': {}
        }
        
        # Calculate current utilization
        total_capacity = sum(room.room_cap for room in rooms)
        total_occupied = sum(room.room_occupied for room in rooms)
        
        if total_capacity > 0:
            optimization_report['current_utilization'] = (total_occupied / total_capacity) * 100
        
        # Analyze room distribution
        underutilized_rooms = rooms.filter(room_occupied__lt=F('room_cap') - 1)
        single_occupancy_rooms = rooms.filter(room_occupied=1, room_cap__gt=1)
        
        # Generate recommendations
        if underutilized_rooms.count() > 0:
            optimization_report['recommendations'].append({
                'type': 'consolidation',
                'description': f"Consider consolidating students from {underutilized_rooms.count()} underutilized rooms",
                'potential_freed_rooms': underutilized_rooms.count() // 2
            })
        
        if single_occupancy_rooms.count() > 2:
            optimization_report['recommendations'].append({
                'type': 'pair_matching',
                'description': f"Consider pairing students in {single_occupancy_rooms.count()} single-occupancy rooms",
                'potential_space_gain': single_occupancy_rooms.count() // 2
            })
        
        # Calculate potential optimal utilization
        if optimization_report['recommendations']:
            potential_freed_capacity = sum(
                rec.get('potential_freed_rooms', 0) + rec.get('potential_space_gain', 0)
                for rec in optimization_report['recommendations']
            )
            
            potential_capacity = total_capacity + (potential_freed_capacity * 3)  # Assuming 3-person rooms
            optimization_report['optimal_utilization'] = min(100, (total_occupied / potential_capacity) * 100)
        
        return optimization_report
        
    except Hall.DoesNotExist:
        return None

def generate_room_maintenance_schedule(hall_id=None):
    """Generate room maintenance schedule based on occupancy"""
    
    if hall_id:
        rooms = HallRoom.objects.filter(hall__hall_id=hall_id)
    else:
        rooms = HallRoom.objects.all()
    
    maintenance_schedule = {
        'immediate_maintenance': [],    # Empty rooms
        'scheduled_maintenance': [],    # Partially occupied
        'deferred_maintenance': [],     # Full rooms
        'maintenance_statistics': {}
    }
    
    for room in rooms.select_related('hall'):
        room_info = {
            'hall': room.hall.hall_name,
            'room_id': f"{room.block_no}-{room.room_no}",
            'capacity': room.room_cap,
            'occupied': room.room_occupied,
            'last_maintenance': get_last_maintenance_date(room),
            'maintenance_priority': calculate_maintenance_priority(room)
        }
        
        if room.room_occupied == 0:
            maintenance_schedule['immediate_maintenance'].append(room_info)
        elif room.room_occupied < room.room_cap:
            maintenance_schedule['scheduled_maintenance'].append(room_info)
        else:
            maintenance_schedule['deferred_maintenance'].append(room_info)
    
    # Sort by maintenance priority
    for category in ['immediate_maintenance', 'scheduled_maintenance', 'deferred_maintenance']:
        maintenance_schedule[category].sort(key=lambda x: x['maintenance_priority'], reverse=True)
    
    # Generate statistics
    maintenance_schedule['maintenance_statistics'] = {
        'total_rooms': rooms.count(),
        'immediate_available': len(maintenance_schedule['immediate_maintenance']),
        'scheduled_possible': len(maintenance_schedule['scheduled_maintenance']),
        'deferred_required': len(maintenance_schedule['deferred_maintenance']),
        'recommended_maintenance_days': calculate_recommended_maintenance_duration(rooms)
    }
    
    return maintenance_schedule
```

---

## **HOSTEL MANAGEMENT SYSTEM SUMMARY - PART 2**

### **Operations and Service Models (4):**
1. **StaffSchedule** - Workforce scheduling and shift management
2. **HostelNoticeBoard** - Communication and announcement system
3. **HostelStudentAttendence** - Student presence monitoring and safety tracking
4. **HallRoom** - Room allocation and occupancy management

### **Advanced Features Implemented:**
- **Intelligent Scheduling** with workload optimization and coverage analysis
- **Comprehensive Communication** with role-based posting authority and notification systems
- **Safety Monitoring** with consecutive absence detection and alert systems
- **Optimal Resource Allocation** with room consolidation and utilization optimization
- **Performance Analytics** with attendance patterns and utilization metrics
- **Automated Alerts** with escalation protocols and notification systems