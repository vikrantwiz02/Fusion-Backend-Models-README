# Complete Theoretical Documentation: Hostel Management System - Part 1

## **COMPREHENSIVE HOSTEL MANAGEMENT SYSTEM DOCUMENTATION - PART 1**

---

## Core Infrastructure and Administration

### 1. **Hall Model - Hostel Infrastructure Management**

```python
class Hall(models.Model):
    hall_id = models.CharField(max_length=10)
    hall_name = models.CharField(max_length=50)
    max_accomodation = models.IntegerField(default=0)
    number_students = models.PositiveIntegerField(default=0)
    assigned_batch = models.CharField(max_length=50, null=True, blank=True)
    TYPE_OF_SEATER_CHOICES = [
        ('single', 'Single Seater'),
        ('double', 'Double Seater'),
        ('triple', 'Triple Seater'),
    ]
    type_of_seater = models.CharField(max_length=50, choices=TYPE_OF_SEATER_CHOICES, default='single')
```

#### **Field Analysis:**

#### **Identity and Naming:**
```python
hall_id = models.CharField(max_length=10)
hall_name = models.CharField(max_length=50)
```
**Hall Identification:**
- **hall_id**: Unique identifier for hostel hall (e.g., "H1", "BHAWAN-A")
- **hall_name**: Human-readable hall name (e.g., "Kalam Hall", "Vishwakarma Bhawan")
- Used for hall lookup, assignment, and identification across system
- Enables hierarchical hostel structure management

#### **Capacity Management:**
```python
max_accomodation = models.IntegerField(default=0)
number_students = models.PositiveIntegerField(default=0)
```
**Occupancy Control:**
- **max_accomodation**: Maximum student capacity for the hall
- **number_students**: Current number of students residing
- Enables occupancy tracking and room availability calculation
- Used for admission control and capacity planning

#### **Academic and Configuration:**
```python
assigned_batch = models.CharField(max_length=50, null=True, blank=True)
type_of_seater = models.CharField(max_length=50, choices=TYPE_OF_SEATER_CHOICES, default='single')
```
**Hall Configuration:**
- **assigned_batch**: Specific academic batch assigned to hall (e.g., "2023-2027")
- **type_of_seater**: Room occupancy type (single/double/triple)
- Enables batch-based allocation and room type management
- Supports segregation policies and accommodation preferences

#### **Hall Management Business Logic:**
```python
def create_new_hall(hall_id, hall_name, max_accommodation, type_of_seater, assigned_batch=None):
    """Create new hostel hall with validation"""
    
    # Check for duplicate hall ID
    if Hall.objects.filter(hall_id=hall_id).exists():
        return False, f"Hall with ID {hall_id} already exists"
    
    # Validate capacity
    if max_accommodation <= 0:
        return False, "Maximum accommodation must be greater than 0"
    
    # Validate seater type
    valid_seater_types = ['single', 'double', 'triple']
    if type_of_seater not in valid_seater_types:
        return False, f"Invalid seater type. Must be one of: {valid_seater_types}"
    
    # Create hall
    hall = Hall.objects.create(
        hall_id=hall_id,
        hall_name=hall_name,
        max_accomodation=max_accommodation,
        type_of_seater=type_of_seater,
        assigned_batch=assigned_batch,
        number_students=0
    )
    
    # Create default room structure
    create_default_room_structure(hall)
    
    # Initialize hall management
    initialize_hall_management(hall)
    
    return True, f"Hall {hall_name} created successfully"

def create_default_room_structure(hall):
    """Create default room structure for new hall"""
    
    # Calculate number of rooms based on capacity and seater type
    students_per_room = {'single': 1, 'double': 2, 'triple': 3}
    capacity_per_room = students_per_room.get(hall.type_of_seater, 3)
    
    total_rooms = hall.max_accomodation // capacity_per_room
    
    # Create rooms in blocks (assuming 4 blocks: A, B, C, D)
    blocks = ['A', 'B', 'C', 'D']
    rooms_per_block = total_rooms // len(blocks)
    
    room_counter = 1
    
    for block in blocks:
        for room_num in range(1, rooms_per_block + 1):
            room_number = f"{room_num:03d}"  # Format as 001, 002, etc.
            
            HallRoom.objects.create(
                hall=hall,
                room_no=room_number,
                block_no=block,
                room_cap=capacity_per_room,
                room_occupied=0
            )
            room_counter += 1

def calculate_hall_occupancy_rate(hall_id):
    """Calculate current occupancy rate for hall"""
    
    try:
        hall = Hall.objects.get(hall_id=hall_id)
        
        if hall.max_accomodation == 0:
            return 0
        
        occupancy_rate = (hall.number_students / hall.max_accomodation) * 100
        
        return round(occupancy_rate, 2)
        
    except Hall.DoesNotExist:
        return None

def update_hall_student_count(hall_id, operation='increment', count=1):
    """Update student count for hall"""
    
    try:
        hall = Hall.objects.get(hall_id=hall_id)
        
        if operation == 'increment':
            new_count = hall.number_students + count
            if new_count > hall.max_accomodation:
                return False, f"Cannot exceed maximum accommodation of {hall.max_accomodation}"
            hall.number_students = new_count
        
        elif operation == 'decrement':
            new_count = hall.number_students - count
            if new_count < 0:
                return False, "Student count cannot be negative"
            hall.number_students = new_count
        
        elif operation == 'set':
            if count > hall.max_accomodation:
                return False, f"Count exceeds maximum accommodation of {hall.max_accomodation}"
            hall.number_students = count
        
        hall.save()
        
        # Update room occupancy statistics
        update_room_occupancy_statistics(hall)
        
        return True, f"Student count updated to {hall.number_students}"
        
    except Hall.DoesNotExist:
        return False, "Hall not found"

def get_hall_availability_status(hall_id):
    """Get comprehensive availability status for hall"""
    
    try:
        hall = Hall.objects.get(hall_id=hall_id)
        
        # Calculate available spaces
        available_spaces = hall.max_accomodation - hall.number_students
        occupancy_rate = calculate_hall_occupancy_rate(hall_id)
        
        # Get room-wise availability
        total_rooms = HallRoom.objects.filter(hall=hall).count()
        occupied_rooms = HallRoom.objects.filter(hall=hall, room_occupied__gt=0).count()
        available_rooms = total_rooms - occupied_rooms
        
        # Determine availability status
        if available_spaces == 0:
            status = "Full"
        elif available_spaces <= 5:
            status = "Nearly Full"
        elif occupancy_rate < 50:
            status = "Available"
        else:
            status = "Moderately Occupied"
        
        return {
            'hall_id': hall.hall_id,
            'hall_name': hall.hall_name,
            'status': status,
            'available_spaces': available_spaces,
            'total_capacity': hall.max_accomodation,
            'current_students': hall.number_students,
            'occupancy_rate': occupancy_rate,
            'available_rooms': available_rooms,
            'total_rooms': total_rooms,
            'assigned_batch': hall.assigned_batch,
            'room_type': hall.type_of_seater
        }
        
    except Hall.DoesNotExist:
        return None

def batch_assign_hall(hall_id, batch_year, force_assign=False):
    """Assign hall to specific academic batch"""
    
    try:
        hall = Hall.objects.get(hall_id=hall_id)
        
        # Check if hall is already assigned to a different batch
        if hall.assigned_batch and hall.assigned_batch != batch_year and not force_assign:
            return False, f"Hall is already assigned to batch {hall.assigned_batch}"
        
        # Check if there are students from different batches
        if hall.number_students > 0 and not force_assign:
            current_students = get_hall_students(hall_id)
            different_batch_students = [
                student for student in current_students 
                if student.get('batch') != batch_year
            ]
            
            if different_batch_students:
                return False, f"Hall has {len(different_batch_students)} students from different batches"
        
        # Assign batch
        hall.assigned_batch = batch_year
        hall.save()
        
        # Log the assignment
        log_hall_assignment_change(hall, 'batch', hall.assigned_batch, batch_year)
        
        return True, f"Hall {hall.hall_name} assigned to batch {batch_year}"
        
    except Hall.DoesNotExist:
        return False, "Hall not found"

def generate_hall_utilization_report():
    """Generate comprehensive hall utilization report"""
    
    all_halls = Hall.objects.all()
    
    report = {
        'total_halls': all_halls.count(),
        'total_capacity': sum(hall.max_accomodation for hall in all_halls),
        'total_occupied': sum(hall.number_students for hall in all_halls),
        'overall_occupancy_rate': 0,
        'hall_details': [],
        'batch_distribution': {},
        'room_type_distribution': {},
        'utilization_categories': {
            'full': 0,
            'nearly_full': 0,
            'moderately_occupied': 0,
            'available': 0
        }
    }
    
    # Calculate overall occupancy rate
    if report['total_capacity'] > 0:
        report['overall_occupancy_rate'] = (report['total_occupied'] / report['total_capacity']) * 100
    
    # Process each hall
    for hall in all_halls:
        hall_status = get_hall_availability_status(hall.hall_id)
        
        if hall_status:
            report['hall_details'].append(hall_status)
            
            # Count by utilization category
            status_key = hall_status['status'].lower().replace(' ', '_')
            if status_key in report['utilization_categories']:
                report['utilization_categories'][status_key] += 1
            
            # Batch distribution
            batch = hall.assigned_batch or 'Unassigned'
            if batch not in report['batch_distribution']:
                report['batch_distribution'][batch] = {
                    'halls': 0,
                    'capacity': 0,
                    'occupied': 0
                }
            
            report['batch_distribution'][batch]['halls'] += 1
            report['batch_distribution'][batch]['capacity'] += hall.max_accomodation
            report['batch_distribution'][batch]['occupied'] += hall.number_students
            
            # Room type distribution
            room_type = hall.type_of_seater
            if room_type not in report['room_type_distribution']:
                report['room_type_distribution'][room_type] = {
                    'halls': 0,
                    'capacity': 0,
                    'occupied': 0
                }
            
            report['room_type_distribution'][room_type]['halls'] += 1
            report['room_type_distribution'][room_type]['capacity'] += hall.max_accomodation
            report['room_type_distribution'][room_type]['occupied'] += hall.number_students
    
    return report
```

### 2. **HallCaretaker Model - Support Staff Management**

```python
class HallCaretaker(models.Model):
    hall = models.ForeignKey(Hall, on_delete=models.CASCADE)
    staff = models.ForeignKey(Staff, on_delete=models.CASCADE)
```

#### **Field Analysis:**

#### **Hall Assignment:**
```python
hall = models.ForeignKey(Hall, on_delete=models.CASCADE)
```
**Location Responsibility:**
- Direct reference to specific hostel hall
- Enables hall-specific caretaker assignment
- Supports multiple caretakers per hall if needed
- Used for maintenance coordination and student support

#### **Staff Reference:**
```python
staff = models.ForeignKey(Staff, on_delete=models.CASCADE)
```
**Personnel Management:**
- Links to institution's staff management system
- Enables staff profile and credential access
- Supports payroll and HR integration
- Used for scheduling and performance tracking

#### **Caretaker Management Business Logic:**
```python
def assign_caretaker_to_hall(hall_id, staff_id):
    """Assign caretaker to specific hall"""
    
    try:
        hall = Hall.objects.get(hall_id=hall_id)
        staff = Staff.objects.get(id=staff_id)
        
        # Check if caretaker is already assigned to this hall
        existing_assignment = HallCaretaker.objects.filter(
            hall=hall,
            staff=staff
        ).first()
        
        if existing_assignment:
            return False, "Caretaker is already assigned to this hall"
        
        # Check caretaker's current workload
        current_assignments = HallCaretaker.objects.filter(staff=staff).count()
        
        if current_assignments >= 3:  # Maximum 3 halls per caretaker
            return False, f"Caretaker is already assigned to {current_assignments} halls (maximum 3)"
        
        # Create assignment
        assignment = HallCaretaker.objects.create(
            hall=hall,
            staff=staff
        )
        
        # Update hall allotment record
        update_hall_allotment_caretaker(hall, staff)
        
        # Log the assignment
        log_caretaker_assignment(hall, staff, 'assigned')
        
        # Notify caretaker of new assignment
        notify_caretaker_assignment(staff, hall)
        
        return True, f"Caretaker {staff.id.user.get_full_name()} assigned to {hall.hall_name}"
        
    except Hall.DoesNotExist:
        return False, "Hall not found"
    except Staff.DoesNotExist:
        return False, "Staff member not found"

def remove_caretaker_from_hall(hall_id, staff_id):
    """Remove caretaker assignment from hall"""
    
    try:
        assignment = HallCaretaker.objects.get(
            hall__hall_id=hall_id,
            staff__id=staff_id
        )
        
        hall = assignment.hall
        staff = assignment.staff
        
        # Check if this is the only caretaker for the hall
        caretaker_count = HallCaretaker.objects.filter(hall=hall).count()
        
        if caretaker_count == 1:
            return False, "Cannot remove the only caretaker. Assign another caretaker first."
        
        # Remove assignment
        assignment.delete()
        
        # Update hall allotment record
        update_hall_allotment_caretaker(hall, None)
        
        # Log the removal
        log_caretaker_assignment(hall, staff, 'removed')
        
        # Notify caretaker of assignment removal
        notify_caretaker_removal(staff, hall)
        
        return True, f"Caretaker {staff.id.user.get_full_name()} removed from {hall.hall_name}"
        
    except HallCaretaker.DoesNotExist:
        return False, "Caretaker assignment not found"

def get_caretaker_workload_report():
    """Generate caretaker workload analysis report"""
    
    all_caretakers = HallCaretaker.objects.select_related('staff', 'hall').all()
    
    caretaker_stats = {}
    
    for assignment in all_caretakers:
        staff_id = assignment.staff.id
        staff_name = assignment.staff.id.user.get_full_name()
        
        if staff_id not in caretaker_stats:
            caretaker_stats[staff_id] = {
                'name': staff_name,
                'halls_assigned': [],
                'total_students': 0,
                'workload_score': 0
            }
        
        caretaker_stats[staff_id]['halls_assigned'].append({
            'hall_id': assignment.hall.hall_id,
            'hall_name': assignment.hall.hall_name,
            'student_count': assignment.hall.number_students,
            'capacity': assignment.hall.max_accomodation
        })
        
        caretaker_stats[staff_id]['total_students'] += assignment.hall.number_students
    
    # Calculate workload scores
    for staff_id, stats in caretaker_stats.items():
        hall_count = len(stats['halls_assigned'])
        student_count = stats['total_students']
        
        # Workload score: weighted combination of halls and students
        stats['workload_score'] = (hall_count * 30) + (student_count * 0.5)
        stats['hall_count'] = hall_count
    
    # Sort by workload score
    sorted_caretakers = sorted(
        caretaker_stats.items(),
        key=lambda x: x[1]['workload_score'],
        reverse=True
    )
    
    report = {
        'total_caretakers': len(caretaker_stats),
        'caretaker_details': dict(sorted_caretakers),
        'workload_distribution': {
            'overloaded': 0,  # > 100 score
            'normal': 0,      # 50-100 score
            'underutilized': 0 # < 50 score
        },
        'recommendations': []
    }
    
    # Categorize workload distribution
    for staff_id, stats in caretaker_stats.items():
        score = stats['workload_score']
        
        if score > 100:
            report['workload_distribution']['overloaded'] += 1
        elif score >= 50:
            report['workload_distribution']['normal'] += 1
        else:
            report['workload_distribution']['underutilized'] += 1
    
    # Generate recommendations
    for staff_id, stats in sorted_caretakers:
        if stats['workload_score'] > 120:
            report['recommendations'].append({
                'type': 'reduce_workload',
                'caretaker': stats['name'],
                'current_score': stats['workload_score'],
                'suggestion': f"Consider redistributing some halls from {stats['name']} (score: {stats['workload_score']})"
            })
        elif stats['workload_score'] < 30:
            report['recommendations'].append({
                'type': 'increase_workload',
                'caretaker': stats['name'],
                'current_score': stats['workload_score'],
                'suggestion': f"Consider assigning more halls to {stats['name']} (score: {stats['workload_score']})"
            })
    
    return report

def find_optimal_caretaker_for_hall(hall_id):
    """Find the best caretaker for a hall based on current workload"""
    
    try:
        hall = Hall.objects.get(hall_id=hall_id)
        
        # Get all available staff who can be caretakers
        available_staff = Staff.objects.all()
        
        caretaker_options = []
        
        for staff in available_staff:
            current_assignments = HallCaretaker.objects.filter(staff=staff).count()
            
            if current_assignments < 3:  # Not at maximum capacity
                current_students = sum(
                    assignment.hall.number_students 
                    for assignment in HallCaretaker.objects.filter(staff=staff)
                )
                
                # Calculate projected workload
                projected_students = current_students + hall.number_students
                projected_score = ((current_assignments + 1) * 30) + (projected_students * 0.5)
                
                caretaker_options.append({
                    'staff': staff,
                    'current_assignments': current_assignments,
                    'current_students': current_students,
                    'projected_score': projected_score,
                    'name': staff.id.user.get_full_name()
                })
        
        # Sort by projected workload score (ascending - prefer lower workload)
        caretaker_options.sort(key=lambda x: x['projected_score'])
        
        return caretaker_options
        
    except Hall.DoesNotExist:
        return []

def generate_caretaker_performance_report(month, year):
    """Generate monthly performance report for caretakers"""
    
    caretakers = HallCaretaker.objects.select_related('staff', 'hall').all()
    
    performance_data = {}
    
    for assignment in caretakers:
        staff_id = assignment.staff.id
        staff_name = assignment.staff.id.user.get_full_name()
        hall = assignment.hall
        
        if staff_id not in performance_data:
            performance_data[staff_id] = {
                'name': staff_name,
                'halls': [],
                'total_complaints': 0,
                'resolved_complaints': 0,
                'maintenance_requests': 0,
                'student_satisfaction': 0,
                'attendance_score': 0
            }
        
        # Get hall-specific performance metrics
        hall_complaints = get_hall_complaints_for_month(hall.hall_id, month, year)
        resolved_complaints = get_resolved_complaints_for_month(hall.hall_id, month, year)
        maintenance_requests = get_maintenance_requests_for_month(hall.hall_id, month, year)
        
        performance_data[staff_id]['halls'].append(hall.hall_name)
        performance_data[staff_id]['total_complaints'] += hall_complaints
        performance_data[staff_id]['resolved_complaints'] += resolved_complaints
        performance_data[staff_id]['maintenance_requests'] += maintenance_requests
    
    # Calculate performance scores
    for staff_id, data in performance_data.items():
        # Resolution rate
        if data['total_complaints'] > 0:
            resolution_rate = (data['resolved_complaints'] / data['total_complaints']) * 100
        else:
            resolution_rate = 100  # No complaints is good
        
        # Overall performance score
        data['resolution_rate'] = resolution_rate
        data['performance_score'] = (
            (resolution_rate * 0.4) +  # 40% weight to complaint resolution
            (min(100, 100 - data['maintenance_requests'] * 2) * 0.3) +  # 30% weight to maintenance efficiency
            (data['attendance_score'] * 0.3)  # 30% weight to attendance
        )
    
    return performance_data
```

### 3. **HallWarden Model - Academic Staff Oversight**

```python
class HallWarden(models.Model):
    hall = models.ForeignKey(Hall, on_delete=models.CASCADE)
    faculty = models.ForeignKey(Faculty, on_delete=models.CASCADE)
```

#### **Field Analysis:**

#### **Hall Supervision:**
```python
hall = models.ForeignKey(Hall, on_delete=models.CASCADE)
```
**Administrative Oversight:**
- Direct link to specific hostel hall
- Enables academic supervision of residential life
- Supports student welfare and disciplinary management
- Used for academic-residential coordination

#### **Faculty Assignment:**
```python
faculty = models.ForeignKey(Faculty, on_delete=models.CASCADE)
```
**Academic Authority:**
- Links to faculty member serving as warden
- Enables academic oversight of student life
- Supports faculty-student mentoring relationships
- Used for disciplinary actions and academic guidance

#### **Warden Management Business Logic:**
```python
def assign_warden_to_hall(hall_id, faculty_id):
    """Assign faculty warden to hall"""
    
    try:
        hall = Hall.objects.get(hall_id=hall_id)
        faculty = Faculty.objects.get(id=faculty_id)
        
        # Check if warden is already assigned to this hall
        existing_warden = HallWarden.objects.filter(
            hall=hall,
            faculty=faculty
        ).first()
        
        if existing_warden:
            return False, "Faculty member is already warden of this hall"
        
        # Check if hall already has a warden
        current_warden = HallWarden.objects.filter(hall=hall).first()
        
        if current_warden:
            return False, f"Hall already has warden: {current_warden.faculty.id.user.get_full_name()}"
        
        # Check faculty's current warden responsibilities
        faculty_warden_count = HallWarden.objects.filter(faculty=faculty).count()
        
        if faculty_warden_count >= 2:  # Maximum 2 halls per faculty warden
            return False, f"Faculty member is already warden of {faculty_warden_count} halls (maximum 2)"
        
        # Create warden assignment
        warden_assignment = HallWarden.objects.create(
            hall=hall,
            faculty=faculty
        )
        
        # Update hall allotment record
        update_hall_allotment_warden(hall, faculty)
        
        # Log the assignment
        log_warden_assignment(hall, faculty, 'assigned')
        
        # Notify faculty of warden responsibilities
        notify_warden_assignment(faculty, hall)
        
        # Set up warden orientation if needed
        schedule_warden_orientation(faculty, hall)
        
        return True, f"Dr. {faculty.id.user.get_full_name()} assigned as warden of {hall.hall_name}"
        
    except Hall.DoesNotExist:
        return False, "Hall not found"
    except Faculty.DoesNotExist:
        return False, "Faculty member not found"

def remove_warden_from_hall(hall_id, faculty_id):
    """Remove warden assignment from hall"""
    
    try:
        warden_assignment = HallWarden.objects.get(
            hall__hall_id=hall_id,
            faculty__id=faculty_id
        )
        
        hall = warden_assignment.hall
        faculty = warden_assignment.faculty
        
        # Check for pending warden responsibilities
        pending_issues = check_pending_warden_responsibilities(hall)
        
        if pending_issues['critical_count'] > 0:
            return False, f"Cannot remove warden. {pending_issues['critical_count']} critical issues pending resolution"
        
        # Transfer responsibilities to temporary warden
        assign_temporary_warden(hall)
        
        # Remove warden assignment
        warden_assignment.delete()
        
        # Update hall allotment record
        update_hall_allotment_warden(hall, None)
        
        # Log the removal
        log_warden_assignment(hall, faculty, 'removed')
        
        # Notify faculty of assignment removal
        notify_warden_removal(faculty, hall)
        
        return True, f"Dr. {faculty.id.user.get_full_name()} removed as warden of {hall.hall_name}"
        
    except HallWarden.DoesNotExist:
        return False, "Warden assignment not found"

def get_warden_responsibility_report():
    """Generate comprehensive warden responsibility report"""
    
    all_wardens = HallWarden.objects.select_related('faculty', 'hall').all()
    
    warden_stats = {}
    
    for assignment in all_wardens:
        faculty_id = assignment.faculty.id
        faculty_name = assignment.faculty.id.user.get_full_name()
        department = assignment.faculty.id.department.name
        
        if faculty_id not in warden_stats:
            warden_stats[faculty_id] = {
                'name': faculty_name,
                'department': department,
                'halls_assigned': [],
                'total_students_supervised': 0,
                'responsibility_score': 0,
                'pending_disciplinary_cases': 0,
                'resolved_issues_this_month': 0
            }
        
        hall = assignment.hall
        
        # Get hall-specific metrics
        pending_cases = get_pending_disciplinary_cases(hall.hall_id)
        resolved_issues = get_resolved_warden_issues(hall.hall_id, current_month=True)
        
        warden_stats[faculty_id]['halls_assigned'].append({
            'hall_id': hall.hall_id,
            'hall_name': hall.hall_name,
            'student_count': hall.number_students,
            'batch': hall.assigned_batch,
            'pending_cases': pending_cases,
            'resolved_issues': resolved_issues
        })
        
        warden_stats[faculty_id]['total_students_supervised'] += hall.number_students
        warden_stats[faculty_id]['pending_disciplinary_cases'] += pending_cases
        warden_stats[faculty_id]['resolved_issues_this_month'] += resolved_issues
    
    # Calculate responsibility scores
    for faculty_id, stats in warden_stats.items():
        hall_count = len(stats['halls_assigned'])
        student_count = stats['total_students_supervised']
        pending_cases = stats['pending_disciplinary_cases']
        resolved_issues = stats['resolved_issues_this_month']
        
        # Responsibility score: considering students, cases, and performance
        base_score = (hall_count * 25) + (student_count * 0.3)
        
        # Adjust for performance
        if resolved_issues > 0:
            performance_bonus = min(20, resolved_issues * 2)
        else:
            performance_bonus = 0
        
        # Penalty for pending cases
        case_penalty = pending_cases * 5
        
        stats['responsibility_score'] = max(0, base_score + performance_bonus - case_penalty)
    
    report = {
        'total_wardens': len(warden_stats),
        'warden_details': warden_stats,
        'workload_distribution': {
            'high_responsibility': 0,     # > 100 score
            'moderate_responsibility': 0, # 50-100 score
            'light_responsibility': 0     # < 50 score
        },
        'department_distribution': {},
        'performance_metrics': {
            'avg_students_per_warden': 0,
            'total_pending_cases': 0,
            'total_resolved_issues': 0
        }
    }
    
    # Calculate aggregate metrics
    total_students = sum(stats['total_students_supervised'] for stats in warden_stats.values())
    total_pending = sum(stats['pending_disciplinary_cases'] for stats in warden_stats.values())
    total_resolved = sum(stats['resolved_issues_this_month'] for stats in warden_stats.values())
    
    if len(warden_stats) > 0:
        report['performance_metrics']['avg_students_per_warden'] = total_students / len(warden_stats)
    
    report['performance_metrics']['total_pending_cases'] = total_pending
    report['performance_metrics']['total_resolved_issues'] = total_resolved
    
    # Categorize by responsibility level
    for faculty_id, stats in warden_stats.items():
        score = stats['responsibility_score']
        department = stats['department']
        
        if score > 100:
            report['workload_distribution']['high_responsibility'] += 1
        elif score >= 50:
            report['workload_distribution']['moderate_responsibility'] += 1
        else:
            report['workload_distribution']['light_responsibility'] += 1
        
        # Department distribution
        if department not in report['department_distribution']:
            report['department_distribution'][department] = 0
        report['department_distribution'][department] += 1
    
    return report

def schedule_warden_meeting(hall_id, meeting_purpose, meeting_date, participants=None):
    """Schedule warden meeting for hall-related issues"""
    
    try:
        hall = Hall.objects.get(hall_id=hall_id)
        warden = HallWarden.objects.filter(hall=hall).first()
        
        if not warden:
            return False, "No warden assigned to this hall"
        
        # Validate meeting date
        if meeting_date <= datetime.date.today():
            return False, "Meeting date must be in the future"
        
        # Create meeting record
        meeting_data = {
            'hall': hall,
            'warden': warden.faculty,
            'purpose': meeting_purpose,
            'scheduled_date': meeting_date,
            'participants': participants or [],
            'status': 'scheduled'
        }
        
        # Store meeting (would use a WardenMeeting model if it existed)
        meeting_id = store_warden_meeting(meeting_data)
        
        # Notify participants
        notify_warden_meeting_scheduled(warden.faculty, hall, meeting_purpose, meeting_date)
        
        if participants:
            notify_meeting_participants(participants, hall, meeting_purpose, meeting_date)
        
        return True, f"Warden meeting scheduled for {meeting_date}"
        
    except Hall.DoesNotExist:
        return False, "Hall not found"

def evaluate_warden_performance(faculty_id, evaluation_period_months=6):
    """Evaluate warden performance over specified period"""
    
    try:
        faculty = Faculty.objects.get(id=faculty_id)
        warden_assignments = HallWarden.objects.filter(faculty=faculty)
        
        if not warden_assignments.exists():
            return False, "Faculty member is not assigned as warden"
        
        evaluation_data = {
            'faculty_name': faculty.id.user.get_full_name(),
            'evaluation_period': f"{evaluation_period_months} months",
            'halls_supervised': [],
            'overall_performance': {},
            'recommendations': []
        }
        
        total_students = 0
        total_complaints = 0
        resolved_complaints = 0
        disciplinary_actions = 0
        
        for assignment in warden_assignments:
            hall = assignment.hall
            
            # Get hall-specific performance data
            hall_performance = evaluate_hall_warden_performance(
                hall.hall_id, 
                faculty_id, 
                evaluation_period_months
            )
            
            evaluation_data['halls_supervised'].append(hall_performance)
            
            # Aggregate metrics
            total_students += hall.number_students
            total_complaints += hall_performance.get('complaints_received', 0)
            resolved_complaints += hall_performance.get('complaints_resolved', 0)
            disciplinary_actions += hall_performance.get('disciplinary_actions', 0)
        
        # Calculate overall performance metrics
        if total_complaints > 0:
            resolution_rate = (resolved_complaints / total_complaints) * 100
        else:
            resolution_rate = 100
        
        evaluation_data['overall_performance'] = {
            'total_students_supervised': total_students,
            'complaint_resolution_rate': resolution_rate,
            'total_complaints_handled': total_complaints,
            'disciplinary_actions_taken': disciplinary_actions,
            'student_satisfaction_score': calculate_student_satisfaction(faculty_id),
            'administrative_efficiency': calculate_administrative_efficiency(faculty_id)
        }
        
        # Generate recommendations
        if resolution_rate < 80:
            evaluation_data['recommendations'].append({
                'area': 'complaint_resolution',
                'suggestion': 'Improve complaint resolution process and response time'
            })
        
        if total_students > 200:
            evaluation_data['recommendations'].append({
                'area': 'workload',
                'suggestion': 'Consider reducing supervision load or additional support'
            })
        
        if disciplinary_actions > 50:
            evaluation_data['recommendations'].append({
                'area': 'discipline_management',
                'suggestion': 'Review disciplinary approaches and prevention strategies'
            })
        
        return True, evaluation_data
        
    except Faculty.DoesNotExist:
        return False, "Faculty member not found"
```

---

## Student Life and Services Management

### 4. **GuestRoomBooking Model - Visitor Accommodation**

```python
class GuestRoomBooking(models.Model):
    hall = models.ForeignKey(Hall, on_delete=models.CASCADE)
    intender = models.ForeignKey(User, on_delete=models.CASCADE)
    guest_name = models.CharField(max_length=255)
    guest_phone = models.CharField(max_length=255)
    guest_email = models.CharField(max_length=255, blank=True)
    guest_address = models.TextField(blank=True)
    rooms_required = models.IntegerField(default=1, null=True, blank=True)
    guest_room_id = models.CharField(max_length=255, blank=True)
    total_guest = models.IntegerField(default=1)
    purpose = models.TextField()
    arrival_date = models.DateField(auto_now_add=False, auto_now=False)
    arrival_time = models.TimeField(auto_now_add=False, auto_now=False)
    departure_date = models.DateField(auto_now_add=False, auto_now=False)
    departure_time = models.TimeField(auto_now_add=False, auto_now=False)
    status = models.CharField(max_length=255, choices=HostelManagementConstants.BOOKING_STATUS, default="Pending")
    booking_date = models.DateField(auto_now_add=False, auto_now=False, default=timezone.now)
    nationality = models.CharField(max_length=255, blank=True)
    ROOM_TYPES = [
        ('single', 'Single'),
        ('double', 'Double'),
        ('triple', 'Triple'),
    ]
    room_type = models.CharField(max_length=10, choices=ROOM_TYPES, default='single')
```

#### **Field Analysis:**

#### **Booking Authority:**
```python
hall = models.ForeignKey(Hall, on_delete=models.CASCADE)
intender = models.ForeignKey(User, on_delete=models.CASCADE)
```
**Request Management:**
- **hall**: Target hall for guest accommodation
- **intender**: User making the booking request (student/faculty/staff)
- Enables hall-specific guest room management
- Supports booking accountability and approval workflow

#### **Guest Information:**
```python
guest_name = models.CharField(max_length=255)
guest_phone = models.CharField(max_length=255)
guest_email = models.CharField(max_length=255, blank=True)
guest_address = models.TextField(blank=True)
nationality = models.CharField(max_length=255, blank=True)
```
**Visitor Details:**
- Comprehensive guest identification and contact information
- Enables guest verification and communication
- Supports security protocols and visitor tracking
- Used for emergency contact and legal compliance

#### **Accommodation Specifications:**
```python
rooms_required = models.IntegerField(default=1, null=True, blank=True)
guest_room_id = models.CharField(max_length=255, blank=True)
total_guest = models.IntegerField(default=1)
room_type = models.CharField(max_length=10, choices=ROOM_TYPES, default='single')
```
**Accommodation Requirements:**
- **rooms_required**: Number of rooms needed
- **guest_room_id**: Specific room assignment
- **total_guest**: Number of people in the group
- **room_type**: Type of accommodation (single/double/triple)
- Enables resource allocation and capacity planning

#### **Visit Schedule:**
```python
arrival_date = models.DateField(auto_now_add=False, auto_now=False)
arrival_time = models.TimeField(auto_now_add=False, auto_now=False)
departure_date = models.DateField(auto_now_add=False, auto_now=False)
departure_time = models.TimeField(auto_now_add=False, auto_now=False)
purpose = models.TextField()
```
**Visit Planning:**
- Complete arrival and departure schedule
- Visit purpose documentation
- Enables proper planning and resource allocation
- Used for security and administrative coordination

#### **Booking Management:**
```python
status = models.CharField(max_length=255, choices=HostelManagementConstants.BOOKING_STATUS, default="Pending")
booking_date = models.DateField(auto_now_add=False, auto_now=False, default=timezone.now)
```
**Status Tracking:**
- Comprehensive booking status workflow
- Booking date for audit and tracking
- Enables approval workflow and status management
- Used for booking confirmation and communication

#### **Guest Room Booking Business Logic:**
```python
def submit_guest_room_booking(hall_id, intender_id, guest_details, stay_details, room_requirements):
    """Submit new guest room booking request"""
    
    try:
        hall = Hall.objects.get(hall_id=hall_id)
        intender = User.objects.get(id=intender_id)
        
        # Validate booking request
        validation_result = validate_booking_request(guest_details, stay_details, room_requirements)
        if not validation_result['valid']:
            return False, validation_result['message']
        
        # Check room availability
        available_rooms = check_guest_room_availability(
            hall_id,
            stay_details['arrival_date'],
            stay_details['departure_date'],
            room_requirements['rooms_required'],
            room_requirements['room_type']
        )
        
        if not available_rooms['available']:
            return False, f"No {room_requirements['room_type']} rooms available for requested dates"
        
        # Create booking record
        booking = GuestRoomBooking.objects.create(
            hall=hall,
            intender=intender,
            guest_name=guest_details['name'],
            guest_phone=guest_details['phone'],
            guest_email=guest_details.get('email', ''),
            guest_address=guest_details.get('address', ''),
            nationality=guest_details.get('nationality', ''),
            rooms_required=room_requirements['rooms_required'],
            total_guest=room_requirements['total_guest'],
            room_type=room_requirements['room_type'],
            purpose=stay_details['purpose'],
            arrival_date=stay_details['arrival_date'],
            arrival_time=stay_details['arrival_time'],
            departure_date=stay_details['departure_date'],
            departure_time=stay_details['departure_time'],
            booking_date=datetime.date.today(),
            status='Pending'
        )
        
        # Send notification to hall authorities
        notify_booking_request(booking)
        
        # Generate booking reference
        booking_reference = f"GRB{booking.id:06d}"
        
        # Send confirmation to requester
        send_booking_confirmation(intender, booking, booking_reference)
        
        return True, f"Booking request submitted successfully. Reference: {booking_reference}"
        
    except Hall.DoesNotExist:
        return False, "Hall not found"
    except User.DoesNotExist:
        return False, "User not found"

def validate_booking_request(guest_details, stay_details, room_requirements):
    """Validate guest room booking request"""
    
    # Check required fields
    required_guest_fields = ['name', 'phone']
    for field in required_guest_fields:
        if not guest_details.get(field):
            return {'valid': False, 'message': f"Guest {field} is required"}
    
    # Validate phone number
    if len(guest_details['phone']) < 10:
        return {'valid': False, 'message': "Invalid phone number"}
    
    # Validate dates
    arrival_date = stay_details['arrival_date']
    departure_date = stay_details['departure_date']
    
    if arrival_date <= datetime.date.today():
        return {'valid': False, 'message': "Arrival date must be in the future"}
    
    if departure_date <= arrival_date:
        return {'valid': False, 'message': "Departure date must be after arrival date"}
    
    # Check maximum stay duration (30 days)
    stay_duration = (departure_date - arrival_date).days
    if stay_duration > 30:
        return {'valid': False, 'message': "Maximum stay duration is 30 days"}
    
    # Validate room requirements
    if room_requirements['rooms_required'] < 1 or room_requirements['rooms_required'] > 5:
        return {'valid': False, 'message': "Number of rooms must be between 1 and 5"}
    
    if room_requirements['total_guest'] < 1:
        return {'valid': False, 'message': "Total guests must be at least 1"}
    
    # Check guest-to-room ratio
    max_guests_per_room = {'single': 1, 'double': 2, 'triple': 3}
    max_capacity = max_guests_per_room[room_requirements['room_type']] * room_requirements['rooms_required']
    
    if room_requirements['total_guest'] > max_capacity:
        return {'valid': False, 'message': f"Too many guests for requested room type and count"}
    
    # Validate purpose
    if len(stay_details['purpose'].strip()) < 10:
        return {'valid': False, 'message': "Please provide detailed purpose for visit (minimum 10 characters)"}
    
    return {'valid': True, 'message': "Validation successful"}

def check_guest_room_availability(hall_id, arrival_date, departure_date, rooms_required, room_type):
    """Check availability of guest rooms for specified period"""
    
    try:
        hall = Hall.objects.get(hall_id=hall_id)
        
        # Get all guest rooms of specified type in the hall
        available_guest_rooms = GuestRoom.objects.filter(
            hall=hall,
            room_type=room_type
        )
        
        if available_guest_rooms.count() < rooms_required:
            return {
                'available': False,
                'message': f"Only {available_guest_rooms.count()} {room_type} rooms exist in this hall",
                'available_rooms': []
            }
        
        # Check which rooms are available during the requested period
        available_rooms = []
        
        for room in available_guest_rooms:
            if is_room_available_during_period(room, arrival_date, departure_date):
                available_rooms.append({
                    'room_id': room.room,
                    'room_type': room.room_type,
                    'currently_vacant': room.vacant,
                    'occupied_till': room.occupied_till
                })
        
        if len(available_rooms) >= rooms_required:
            return {
                'available': True,
                'message': f"{len(available_rooms)} {room_type} rooms available",
                'available_rooms': available_rooms[:rooms_required]
            }
        else:
            return {
                'available': False,
                'message': f"Only {len(available_rooms)} {room_type} rooms available during requested period",
                'available_rooms': available_rooms
            }
            
    except Hall.DoesNotExist:
        return {'available': False, 'message': "Hall not found", 'available_rooms': []}

def process_booking_approval(booking_id, action, approver_id, remarks=""):
    """Process guest room booking approval/rejection"""
    
    try:
        booking = GuestRoomBooking.objects.get(id=booking_id)
        approver = User.objects.get(id=approver_id)
        
        if booking.status != 'Pending':
            return False, f"Booking is already {booking.status.lower()}"
        
        if action == 'approve':
            # Check room availability again
            availability = check_guest_room_availability(
                booking.hall.hall_id,
                booking.arrival_date,
                booking.departure_date,
                booking.rooms_required,
                booking.room_type
            )
            
            if not availability['available']:
                return False, f"Rooms no longer available: {availability['message']}"
            
            # Assign specific rooms
            assigned_rooms = assign_guest_rooms(booking, availability['available_rooms'])
            
            if not assigned_rooms:
                return False, "Failed to assign specific rooms"
            
            # Update booking status
            booking.status = 'Confirmed'
            booking.guest_room_id = ','.join(assigned_rooms)
            
            # Block the rooms
            block_guest_rooms(assigned_rooms, booking.arrival_date, booking.departure_date)
            
            # Send confirmation
            send_booking_approval_notification(booking, assigned_rooms)
            
        elif action == 'reject':
            booking.status = 'Rejected'
            
            # Send rejection notification
            send_booking_rejection_notification(booking, remarks)
        
        booking.save()
        
        # Log the action
        log_booking_action(booking, action, approver, remarks)
        
        return True, f"Booking {action}ed successfully"
        
    except GuestRoomBooking.DoesNotExist:
        return False, "Booking not found"
    except User.DoesNotExist:
        return False, "Approver not found"

def assign_guest_rooms(booking, available_rooms):
    """Assign specific guest rooms to confirmed booking"""
    
    assigned_rooms = []
    rooms_needed = booking.rooms_required
    
    # Sort rooms by preference (e.g., lower room numbers first)
    sorted_rooms = sorted(available_rooms, key=lambda x: x['room_id'])
    
    for room in sorted_rooms[:rooms_needed]:
        assigned_rooms.append(room['room_id'])
    
    return assigned_rooms

def check_in_guest(booking_id, check_in_time=None):
    """Process guest check-in"""
    
    try:
        booking = GuestRoomBooking.objects.get(id=booking_id)
        
        if booking.status != 'Confirmed':
            return False, f"Cannot check in. Booking status is {booking.status}"
        
        # Validate check-in date
        today = datetime.date.today()
        if today < booking.arrival_date:
            return False, f"Cannot check in before arrival date ({booking.arrival_date})"
        
        if today > booking.arrival_date + datetime.timedelta(days=1):
            return False, "Check-in window has expired. Please contact administration."
        
        # Update booking status
        booking.status = 'CheckedIn'
        booking.save()
        
        # Update room occupancy
        if booking.guest_room_id:
            room_ids = booking.guest_room_id.split(',')
            update_guest_room_occupancy(room_ids, 'occupy', booking.departure_date)
        
        # Generate check-in confirmation
        check_in_data = {
            'booking_id': booking.id,
            'guest_name': booking.guest_name,
            'room_ids': booking.guest_room_id,
            'check_in_time': check_in_time or datetime.datetime.now(),
            'expected_checkout': booking.departure_date
        }
        
        # Send check-in notification
        send_check_in_notification(booking, check_in_data)
        
        # Provide guest information packet
        generate_guest_info_packet(booking)
        
        return True, f"Guest {booking.guest_name} checked in successfully"
        
    except GuestRoomBooking.DoesNotExist:
        return False, "Booking not found"

def check_out_guest(booking_id, check_out_time=None):
    """Process guest check-out"""
    
    try:
        booking = GuestRoomBooking.objects.get(id=booking_id)
        
        if booking.status != 'CheckedIn':
            return False, f"Cannot check out. Current status is {booking.status}"
        
        # Update booking status
        booking.status = 'Complete'
        booking.save()
        
        # Free up the rooms
        if booking.guest_room_id:
            room_ids = booking.guest_room_id.split(',')
            update_guest_room_occupancy(room_ids, 'vacate')
        
        # Generate check-out confirmation
        check_out_data = {
            'booking_id': booking.id,
            'guest_name': booking.guest_name,
            'check_out_time': check_out_time or datetime.datetime.now(),
            'duration_stayed': (datetime.date.today() - booking.arrival_date).days
        }
        
        # Send check-out notification
        send_check_out_notification(booking, check_out_data)
        
        # Generate billing summary if applicable
        generate_guest_billing_summary(booking)
        
        return True, f"Guest {booking.guest_name} checked out successfully"
        
    except GuestRoomBooking.DoesNotExist:
        return False, "Booking not found"

def generate_guest_room_occupancy_report(start_date, end_date):
    """Generate comprehensive guest room occupancy report"""
    
    bookings = GuestRoomBooking.objects.filter(
        arrival_date__range=[start_date, end_date]
    ).exclude(status='Rejected')
    
    report = {
        'reporting_period': f"{start_date} to {end_date}",
        'total_bookings': bookings.count(),
        'booking_status_breakdown': {},
        'occupancy_by_hall': {},
        'room_type_demand': {},
        'guest_statistics': {},
        'revenue_analysis': {},
        'utilization_metrics': {}
    }
    
    # Booking status breakdown
    status_counts = bookings.values('status').annotate(count=Count('id'))
    for status in status_counts:
        report['booking_status_breakdown'][status['status']] = status['count']
    
    # Occupancy by hall
    hall_occupancy = bookings.values('hall__hall_name').annotate(
        bookings_count=Count('id'),
        total_guests=Sum('total_guest'),
        total_nights=Sum(F('departure_date') - F('arrival_date'))
    )
    
    for hall in hall_occupancy:
        report['occupancy_by_hall'][hall['hall__hall_name']] = {
            'bookings': hall['bookings_count'],
            'total_guests': hall['total_guests'],
            'total_nights': hall['total_nights']
        }
    
    # Room type demand
    room_demand = bookings.values('room_type').annotate(
        demand_count=Count('id'),
        rooms_requested=Sum('rooms_required')
    )
    
    for demand in room_demand:
        report['room_type_demand'][demand['room_type']] = {
            'bookings': demand['demand_count'],
            'total_rooms_requested': demand['rooms_requested']
        }
    
    # Guest statistics
    guest_stats = bookings.aggregate(
        total_guests=Sum('total_guest'),
        avg_guests_per_booking=Avg('total_guest'),
        avg_stay_duration=Avg(F('departure_date') - F('arrival_date')),
        max_stay_duration=Max(F('departure_date') - F('arrival_date'))
    )
    
    report['guest_statistics'] = guest_stats
    
    # Calculate utilization metrics
    total_guest_room_nights = 0
    total_available_room_nights = 0
    
    for hall_name, hall_data in report['occupancy_by_hall'].items():
        # This would require knowledge of total guest rooms per hall
        # Simplified calculation for demonstration
        total_guest_room_nights += hall_data['total_nights']
    
    # Utilization rate (would need more data for accurate calculation)
    if total_available_room_nights > 0:
        report['utilization_metrics']['occupancy_rate'] = (total_guest_room_nights / total_available_room_nights) * 100
    else:
        report['utilization_metrics']['occupancy_rate'] = 0
    
    return report
```

---

## **HOSTEL MANAGEMENT SYSTEM SUMMARY - PART 1**

The Hostel Management system provides comprehensive residential life management through a multi-layered approach:

### **Infrastructure Management Models (3):**
1. **Hall** - Hostel hall infrastructure and capacity management
2. **HallCaretaker** - Support staff assignment and workload distribution
3. **HallWarden** - Faculty oversight and academic supervision

### **Guest Services Model (1):**
1. **GuestRoomBooking** - Visitor accommodation with complete booking workflow

### **Key System Features Covered:**
- **Hierarchical Management Structure** with wardens, caretakers, and administrative oversight
- **Capacity Management** with occupancy tracking and availability calculation
- **Guest Services** with comprehensive booking, approval, and check-in/check-out workflow
- **Performance Analytics** with workload distribution and utilization tracking
- **Automated Assignment** with optimal resource allocation algorithms
- **Quality Assurance** with performance evaluation and recommendation systems

---