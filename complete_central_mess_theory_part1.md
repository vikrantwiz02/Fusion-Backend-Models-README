# Complete Theoretical Documentation: Central Mess Models

## **COMPREHENSIVE CENTRAL MESS SYSTEM DOCUMENTATION**

### **System Overview**
The Central Mess system manages institutional dining facility operations including student registration, meal planning, billing, payments, feedback, and administrative workflows. This system handles everything from menu management to financial transactions across multiple mess facilities.

---

## Constants and Utility Definitions

### **Meal Time Classifications**
```python
MEAL_TIME = (
    ('breakfast', 'Breakfast'),
    ('lunch', 'Lunch'),
    ('dinner','Dinner')
)

MEAL = (
    ('MB', 'Monday Breakfast'), ('ML', 'Monday Lunch'), ('MD', 'Monday Dinner'),
    ('TB', 'Tuesday Breakfast'), ('TL', 'Tuesday Lunch'), ('TD', 'Tuesday Dinner'),
    ('WB', 'Wednesday Breakfast'), ('WL', 'Wednesday Lunch'), ('WD', 'Wednesday Dinner'),
    ('THB', 'Thursday Breakfast'), ('THL', 'Thursday Lunch'), ('THD', 'Thursday Dinner'),
    ('FB', 'Friday Breakfast'), ('FL', 'Friday Lunch'), ('FD', 'Friday Dinner'),
    ('SB', 'Saturday Breakfast'), ('SL', 'Saturday Lunch'), ('SD', 'Saturday Dinner'),
    ('SUB', 'Sunday Breakfast'), ('SUL', 'Sunday Lunch'), ('SUD', 'Sunday Dinner')
)
```

### **Status and Request Management**
```python
STATUS = (
    ('0', 'rejected'),
    ('1', 'pending'),
    ('2', 'accepted')
)

LEAVE_TYPE = (
    ('casual', 'Casual'),
    ('vacation', 'Vacation')
)

FEEDBACK_TYPE = (
    ('maintenance', 'Maintenance'),
    ('food', 'Food'),
    ('cleanliness', 'Cleanliness & Hygiene'),
    ('others', 'Others')
)

MESS_OPTION = (
    ('mess1', 'Mess1'),
    ('mess2', 'Mess2')
)
```

---

## Core Mess Management System

### 1. **Messinfo Model - Student Mess Registration**

```python
class Messinfo(models.Model):
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    mess_option = models.CharField(max_length=20, choices=MESS_OPTION, default='mess2')
```

#### **Field Analysis:**

#### **student_id** - Student Reference
```python
student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
```
**Student Integration:**
- Primary link to student academic record
- Enables comprehensive student mess service tracking
- Provides access to student programme, batch, and department context
- Supports student-specific mess policies and billing

#### **mess_option** - Mess Facility Selection
```python
mess_option = models.CharField(max_length=20, choices=MESS_OPTION, default='mess2')
```
**Mess Facility Management:**
- **mess1**: Primary mess facility
- **mess2**: Secondary mess facility (default)
- Enables multi-facility mess management
- Supports facility-specific menu and pricing policies
- Used for capacity planning and resource allocation

#### **Business Logic:**
```python
def register_student_to_mess(student, mess_option='mess2'):
    """Register student to specific mess facility"""
    
    # Check if already registered
    existing_reg = Messinfo.objects.filter(student_id=student).first()
    if existing_reg:
        return False, f"Student already registered to {existing_reg.mess_option}"
    
    # Create mess registration
    mess_info = Messinfo.objects.create(
        student_id=student,
        mess_option=mess_option
    )
    
    # Create corresponding main registration record
    create_main_registration(student, mess_option)
    
    return True, f"Student registered to {mess_option} successfully"

def transfer_mess_facility(student, new_mess_option):
    """Transfer student between mess facilities"""
    
    try:
        mess_info = Messinfo.objects.get(student_id=student)
        old_mess = mess_info.mess_option
        
        # Update mess option
        mess_info.mess_option = new_mess_option
        mess_info.save()
        
        # Update main registration
        update_main_registration(student, new_mess_option)
        
        # Log transfer
        log_mess_transfer(student, old_mess, new_mess_option)
        
        return True, f"Transferred from {old_mess} to {new_mess_option}"
        
    except Messinfo.DoesNotExist:
        return False, "Student not registered to any mess"
```

### 2. **Reg_main Model - Primary Registration Management**

```python
class Reg_main(models.Model):
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    program = models.CharField(max_length=10)
    current_mess_status = models.CharField(max_length=20, default='Deregistered') 
    balance = models.IntegerField(default=0)
    mess_option = models.CharField(max_length=20, default='mess2')
```

#### **Field Analysis:**

#### **student_id** - Student Reference
```python
student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
```
**Primary Student Link:**
- Core student identification for mess services
- Enables comprehensive mess service management
- Links to student academic and financial records
- Supports cross-system integration and reporting

#### **program** - Academic Programme
```python
program = models.CharField(max_length=10)
```
**Programme Integration:**
- Student's academic programme (B.Tech, M.Tech, PhD)
- Enables programme-specific mess policies
- Supports differential pricing based on programme
- Used for programme-wise mess analytics and planning

#### **current_mess_status** - Registration Status
```python
current_mess_status = models.CharField(max_length=20, default='Deregistered')
```
**Status Management:**
- **Registered**: Active mess service
- **Deregistered**: No active mess service (default)
- **Suspended**: Temporarily suspended service
- **Pending**: Registration/deregistration under process
- Enables status-based service control and billing

#### **balance** - Financial Balance
```python
balance = models.IntegerField(default=0)
```
**Financial Management:**
- Current account balance (positive/negative)
- Tracks payments and outstanding dues
- Enables prepaid mess service model
- Supports financial planning and cash flow management

#### **mess_option** - Facility Assignment
```python
mess_option = models.CharField(max_length=20, default='mess2')
```
**Facility Management:**
- Current mess facility assignment
- Maintains consistency with Messinfo model
- Enables facility-specific service delivery
- Supports load balancing across facilities

### 3. **Reg_records Model - Registration History**

```python
class Reg_records(models.Model):
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    start_date = models.DateField(default=datetime.date.today)
    end_date = models.DateField(default=None, null=True)
```

#### **Field Analysis:**

#### **Temporal Tracking:**
```python
start_date = models.DateField(default=datetime.date.today)
end_date = models.DateField(default=None, null=True)
```
**Registration Timeline:**
- **start_date**: Registration commencement date
- **end_date**: Registration termination date (null for active)
- Enables complete registration history tracking
- Supports duration-based billing and analytics
- Used for historical reporting and trend analysis

#### **Business Logic:**
```python
def create_registration_record(student, start_date=None):
    """Create new registration record"""
    
    if not start_date:
        start_date = datetime.date.today()
    
    # Close any open registration records
    open_records = Reg_records.objects.filter(
        student_id=student,
        end_date__isnull=True
    )
    
    for record in open_records:
        record.end_date = start_date - datetime.timedelta(days=1)
        record.save()
    
    # Create new registration record
    new_record = Reg_records.objects.create(
        student_id=student,
        start_date=start_date
    )
    
    return new_record

def close_registration_record(student, end_date=None):
    """Close active registration record"""
    
    if not end_date:
        end_date = datetime.date.today()
    
    try:
        active_record = Reg_records.objects.get(
            student_id=student,
            end_date__isnull=True
        )
        
        active_record.end_date = end_date
        active_record.save()
        
        return True, "Registration record closed"
        
    except Reg_records.DoesNotExist:
        return False, "No active registration record found"

def get_registration_duration(student, start_date, end_date=None):
    """Calculate registration duration for billing"""
    
    if not end_date:
        end_date = datetime.date.today()
    
    duration = (end_date - start_date).days
    return max(duration, 1)  # Minimum 1 day
```

---

## Registration Request Management

### 1. **Registration_Request Model - Registration Applications**

```python
class Registration_Request(models.Model):
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    Txn_no = models.CharField(max_length=20)
    img = models.ImageField(upload_to='images/', default=None)
    amount = models.IntegerField(default=0)
    status = models.CharField(max_length=10, default='pending')
    registration_remark = models.CharField(max_length=50, default='NA')
    start_date = models.DateField(default=None, null=True)
    payment_date = models.DateField(default=None, null=True)
```

#### **Field Analysis:**

#### **student_id** - Applicant Student
```python
student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
```
**Request Origin:**
- Student submitting registration request
- Links request to student academic and financial records
- Enables student-specific request tracking
- Supports automated notification and communication

#### **Payment Information:**
```python
Txn_no = models.CharField(max_length=20)
img = models.ImageField(upload_to='images/', default=None)
amount = models.IntegerField(default=0)
payment_date = models.DateField(default=None, null=True)
```
**Financial Transaction:**
- **Txn_no**: Transaction reference number for payment verification
- **img**: Payment receipt/proof image upload
- **amount**: Registration fee amount paid
- **payment_date**: Date of payment submission
- Enables payment verification and financial reconciliation

#### **Request Management:**
```python
status = models.CharField(max_length=10, default='pending')
registration_remark = models.CharField(max_length=50, default='NA')
start_date = models.DateField(default=None, null=True)
```
**Workflow Management:**
- **status**: Request status (pending/approved/rejected)
- **registration_remark**: Administrative comments/reasons
- **start_date**: Proposed registration start date
- Enables approval workflow and audit trail

### 2. **Deregistration_Request Model - Deregistration Applications**

```python
class Deregistration_Request(models.Model):
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    status = models.CharField(max_length=10, default='pending')
    deregistration_remark = models.CharField(max_length=50, default='NA')
    end_date = models.DateField(default=None, null=True)
```

#### **Field Analysis:**

#### **Request Management:**
```python
status = models.CharField(max_length=10, default='pending')
deregistration_remark = models.CharField(max_length=50, default='NA')
end_date = models.DateField(default=None, null=True)
```
**Deregistration Workflow:**
- **status**: Request status (pending/approved/rejected)
- **deregistration_remark**: Administrative comments/reasons
- **end_date**: Proposed deregistration date
- Enables controlled service termination with proper billing

#### **Business Logic:**
```python
def process_registration_request(request_id, action, approver, remarks=""):
    """Process registration/deregistration request"""
    
    try:
        if 'registration' in str(type(request_id)):
            request = Registration_Request.objects.get(id=request_id)
            request_type = 'registration'
        else:
            request = Deregistration_Request.objects.get(id=request_id)
            request_type = 'deregistration'
        
        if action == 'approve':
            request.status = 'approved'
            
            if request_type == 'registration':
                # Activate mess registration
                activate_mess_registration(request.student_id, request.start_date)
                request.registration_remark = remarks or "Approved by admin"
            else:
                # Process deregistration
                process_mess_deregistration(request.student_id, request.end_date)
                request.deregistration_remark = remarks or "Approved by admin"
                
        elif action == 'reject':
            request.status = 'rejected'
            
            if request_type == 'registration':
                request.registration_remark = remarks or "Request rejected"
            else:
                request.deregistration_remark = remarks or "Request rejected"
        
        request.save()
        
        # Notify student
        notify_student_request_decision(request.student_id, request_type, action)
        
        return True, f"{request_type.capitalize()} request {action}d successfully"
        
    except Exception as e:
        return False, f"Error processing request: {str(e)}"

def activate_mess_registration(student, start_date):
    """Activate student mess registration"""
    
    # Update main registration status
    reg_main, created = Reg_main.objects.get_or_create(
        student_id=student,
        defaults={
            'program': student.programme,
            'current_mess_status': 'Registered',
            'balance': 0,
            'mess_option': 'mess2'
        }
    )
    
    if not created:
        reg_main.current_mess_status = 'Registered'
        reg_main.save()
    
    # Create registration record
    create_registration_record(student, start_date)
    
    # Create messinfo if not exists
    Messinfo.objects.get_or_create(
        student_id=student,
        defaults={'mess_option': reg_main.mess_option}
    )
    
    return True

def process_mess_deregistration(student, end_date):
    """Process student mess deregistration"""
    
    # Update main registration status
    try:
        reg_main = Reg_main.objects.get(student_id=student)
        reg_main.current_mess_status = 'Deregistered'
        reg_main.save()
        
        # Close registration record
        close_registration_record(student, end_date)
        
        # Process final billing
        process_final_billing(student, end_date)
        
        return True
        
    except Reg_main.DoesNotExist:
        return False
```

---

## Billing and Payment Management

### 1. **Monthly_bill Model - Monthly Bill Generation**

```python
class Monthly_bill(models.Model):
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    month = models.CharField(max_length=20, default=current_month)
    year = models.IntegerField(default=current_year)
    amount = models.IntegerField(default=0)
    rebate_count = models.IntegerField(default=0)
    rebate_amount = models.IntegerField(default=0)
    total_bill = models.IntegerField(default=0)
    paid = models.BooleanField(default=False)
```

#### **Field Analysis:**

#### **Billing Period:**
```python
student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
month = models.CharField(max_length=20, default=current_month)
year = models.IntegerField(default=current_year)
```
**Temporal Billing:**
- **student_id**: Student for whom bill is generated
- **month**: Billing month (January, February, etc.)
- **year**: Billing year
- Enables month-wise billing cycle management
- Supports historical billing analysis and reporting

#### **Amount Calculations:**
```python
amount = models.IntegerField(default=0)
rebate_count = models.IntegerField(default=0)
rebate_amount = models.IntegerField(default=0)
total_bill = models.IntegerField(default=0)
```
**Financial Calculations:**
- **amount**: Base mess charges for the month
- **rebate_count**: Number of rebate days approved
- **rebate_amount**: Total rebate amount deducted
- **total_bill**: Final bill amount (amount - rebate_amount)
- Enables accurate billing with rebate calculations

#### **Payment Status:**
```python
paid = models.BooleanField(default=False)
```
**Payment Tracking:**
- **paid**: Boolean flag indicating payment status
- Enables payment status tracking and outstanding analysis
- Supports automated reminder and follow-up systems
- Used for financial reporting and collection management

### 2. **Payments Model - Payment Records**

```python
class Payments(models.Model):
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    amount_paid = models.IntegerField(default=0)
    payment_month = models.CharField(max_length=20, default=current_month)
    payment_year = models.IntegerField(default=current_year)
    payment_date = models.DateField(default=datetime.date.today)
```

#### **Field Analysis:**

#### **Payment Details:**
```python
amount_paid = models.IntegerField(default=0)
payment_date = models.DateField(default=datetime.date.today)
```
**Transaction Tracking:**
- **amount_paid**: Actual amount paid by student
- **payment_date**: Date of payment transaction
- Enables precise payment tracking and reconciliation
- Supports payment history and audit trail

#### **Payment Allocation:**
```python
payment_month = models.CharField(max_length=20, default=current_month)
payment_year = models.IntegerField(default=current_year)
```
**Bill Allocation:**
- **payment_month**: Month for which payment is made
- **payment_year**: Year for which payment is made
- Enables payment allocation to specific billing periods
- Supports advance payment and arrears management

### 3. **Update_Payment Model - Payment Updates**

```python
class Update_Payment(models.Model):
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    Txn_no = models.CharField(max_length=20)
    img = models.ImageField(upload_to='images/', default=None)
    amount = models.IntegerField(default=0)
    status = models.CharField(max_length=10, default='pending')
    update_remark = models.CharField(max_length=50, default='NA')
    payment_date = models.DateField(default=None, null=True)
```

#### **Payment Verification System:**
```python
Txn_no = models.CharField(max_length=20)
img = models.ImageField(upload_to='images/', default=None)
status = models.CharField(max_length=10, default='pending')
update_remark = models.CharField(max_length=50, default='NA')
```
**Verification Workflow:**
- **Txn_no**: Transaction reference for verification
- **img**: Payment proof/receipt image
- **status**: Verification status (pending/approved/rejected)
- **update_remark**: Administrative comments/verification notes
- Enables manual payment verification and approval

#### **Billing Business Logic:**
```python
def generate_monthly_bill(student, month, year):
    """Generate monthly mess bill for student"""
    
    # Check if bill already exists
    existing_bill = Monthly_bill.objects.filter(
        student_id=student,
        month=month,
        year=year
    ).first()
    
    if existing_bill:
        return existing_bill
    
    # Calculate base amount
    daily_rate = get_daily_mess_rate(student.programme)
    days_in_month = get_days_in_month(month, year)
    base_amount = daily_rate * days_in_month
    
    # Calculate rebates
    rebate_data = calculate_monthly_rebates(student, month, year)
    
    # Create monthly bill
    monthly_bill = Monthly_bill.objects.create(
        student_id=student,
        month=month,
        year=year,
        amount=base_amount,
        rebate_count=rebate_data['count'],
        rebate_amount=rebate_data['amount'],
        total_bill=base_amount - rebate_data['amount']
    )
    
    return monthly_bill

def process_payment(student, amount, payment_month, payment_year):
    """Process student payment"""
    
    # Create payment record
    payment = Payments.objects.create(
        student_id=student,
        amount_paid=amount,
        payment_month=payment_month,
        payment_year=payment_year
    )
    
    # Update monthly bill if exists
    try:
        monthly_bill = Monthly_bill.objects.get(
            student_id=student,
            month=payment_month,
            year=payment_year
        )
        
        if amount >= monthly_bill.total_bill:
            monthly_bill.paid = True
            monthly_bill.save()
            
            # Update account balance
            update_student_balance(student, amount - monthly_bill.total_bill)
            
    except Monthly_bill.DoesNotExist:
        # Payment made in advance, add to balance
        update_student_balance(student, amount)
    
    return payment

def calculate_monthly_rebates(student, month, year):
    """Calculate total rebates for month"""
    
    # Get approved rebates for the month
    rebates = Rebate.objects.filter(
        student_id=student,
        status='2',  # accepted
        start_date__month__lte=get_month_number(month),
        end_date__month__gte=get_month_number(month),
        start_date__year__lte=year,
        end_date__year__gte=year
    )
    
    total_days = 0
    daily_rate = get_daily_mess_rate(student.programme)
    
    for rebate in rebates:
        # Calculate overlapping days with the month
        month_start = datetime.date(year, get_month_number(month), 1)
        month_end = get_month_end_date(month, year)
        
        rebate_start = max(rebate.start_date, month_start)
        rebate_end = min(rebate.end_date, month_end)
        
        if rebate_start <= rebate_end:
            days = (rebate_end - rebate_start).days + 1
            total_days += days
    
    rebate_amount = total_days * daily_rate
    
    return {
        'count': total_days,
        'amount': rebate_amount
    }

def update_student_balance(student, amount):
    """Update student account balance"""
    
    reg_main, created = Reg_main.objects.get_or_create(
        student_id=student,
        defaults={
            'program': student.programme,
            'current_mess_status': 'Registered',
            'balance': 0,
            'mess_option': 'mess2'
        }
    )
    
    reg_main.balance += amount
    reg_main.save()
    
    return reg_main.balance
```

---

## Menu and Meal Management

### 1. **Menu Model - Meal Menu Management**

```python
class Menu(models.Model):
    mess_option = models.CharField(max_length=20, choices=MESS_OPTION, default='mess2')
    meal_time = models.CharField(max_length=20, choices=MEAL)
    dish = models.CharField(max_length=200)
```

#### **Field Analysis:**

#### **Facility and Timing:**
```python
mess_option = models.CharField(max_length=20, choices=MESS_OPTION, default='mess2')
meal_time = models.CharField(max_length=20, choices=MEAL)
```
**Menu Organization:**
- **mess_option**: Mess facility (mess1/mess2)
- **meal_time**: Specific meal slot (MB=Monday Breakfast, TL=Tuesday Lunch, etc.)
- Enables facility-specific menu management
- Supports day-wise and meal-wise menu planning

#### **dish** - Menu Item
```python
dish = models.CharField(max_length=200)
```
**Meal Content:**
- Detailed dish description (up to 200 characters)
- Enables comprehensive menu item specification
- Supports multiple dishes per meal slot
- Used for menu display and student information

### 2. **Menu_change_request Model - Menu Modification Requests**

```python
class Menu_change_request(models.Model):
    dish = models.ForeignKey(Menu, on_delete=models.CASCADE)
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    reason = models.TextField()
    request = models.CharField(max_length=100)
    status = models.CharField(max_length=20, choices=STATUS, default='1')
    app_date = models.DateField(default=datetime.date.today)
```

#### **Field Analysis:**

#### **Request Details:**
```python
dish = models.ForeignKey(Menu, on_delete=models.CASCADE)
reason = models.TextField()
request = models.CharField(max_length=100)
```
**Change Request Specification:**
- **dish**: Specific menu item to be changed
- **reason**: Detailed justification for change
- **request**: Specific change requested
- Enables targeted menu improvement based on feedback

#### **Request Management:**
```python
status = models.CharField(max_length=20, choices=STATUS, default='1')
app_date = models.DateField(default=datetime.date.today)
```
**Workflow Tracking:**
- **status**: Request status (pending/accepted/rejected)
- **app_date**: Request submission date
- Enables systematic menu change management

#### **Menu Business Logic:**
```python
def create_weekly_menu(mess_option, week_start_date):
    """Create complete weekly menu for mess facility"""
    
    days = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday']
    meals = ['Breakfast', 'Lunch', 'Dinner']
    
    menu_items = []
    
    for day in days:
        for meal in meals:
            meal_code = f"{day[0]}{meal[0]}"  # MB, ML, MD, etc.
            if day == 'Thursday':
                meal_code = f"TH{meal[0]}"
            elif day == 'Sunday':
                meal_code = f"SU{meal[0]}"
            
            # Get default dishes for this meal
            default_dishes = get_default_dishes(meal_code, mess_option)
            
            for dish in default_dishes:
                menu_item = Menu.objects.create(
                    mess_option=mess_option,
                    meal_time=meal_code,
                    dish=dish
                )
                menu_items.append(menu_item)
    
    return menu_items

def process_menu_change_request(request_id, action, remarks=""):
    """Process menu change request"""
    
    try:
        request = Menu_change_request.objects.get(id=request_id)
        
        if action == 'approve':
            request.status = '2'  # accepted
            
            # Apply menu change
            apply_menu_change(request.dish, request.request)
            
            # Notify student
            notify_menu_change_approval(request.student_id, request.dish, request.request)
            
        elif action == 'reject':
            request.status = '0'  # rejected
            
            # Notify student with reason
            notify_menu_change_rejection(request.student_id, request.dish, remarks)
        
        request.save()
        
        return True, f"Menu change request {action}d successfully"
        
    except Menu_change_request.DoesNotExist:
        return False, "Menu change request not found"

def get_current_menu(mess_option, meal_time):
    """Get current menu for specific meal"""
    
    menu_items = Menu.objects.filter(
        mess_option=mess_option,
        meal_time=meal_time
    )
    
    return [item.dish for item in menu_items]

def get_weekly_menu_schedule(mess_option):
    """Get complete weekly menu schedule"""
    
    schedule = {}
    
    days = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday']
    
    for day in days:
        schedule[day] = {}
        
        # Get meal codes for the day
        day_prefix = 'TH' if day == 'Thursday' else ('SU' if day == 'Sunday' else day[0])
        
        for meal in ['Breakfast', 'Lunch', 'Dinner']:
            meal_code = f"{day_prefix}{meal[0]}"
            menu_items = get_current_menu(mess_option, meal_code)
            schedule[day][meal] = menu_items
    
    return schedule
```

---

## Request and Special Services Management

### 1. **Rebate Model - Mess Fee Rebate System**

```python
class Rebate(models.Model):
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    start_date = models.DateField(default=datetime.date.today)
    end_date = models.DateField(default=datetime.date.today)
    purpose = models.TextField()
    status = models.CharField(max_length=20, choices=STATUS, default='1')
    app_date = models.DateField(default=datetime.date.today)
    leave_type = models.CharField(choices=LEAVE_TYPE, max_length=20, default="casual")
    rebate_remark = models.CharField(max_length=50, default='NA')
```

#### **Field Analysis:**

#### **Rebate Period:**
```python
start_date = models.DateField(default=datetime.date.today)
end_date = models.DateField(default=datetime.date.today)
purpose = models.TextField()
```
**Rebate Specification:**
- **start_date**: Rebate period start date
- **end_date**: Rebate period end date
- **purpose**: Detailed reason for rebate request
- Enables precise rebate period management and justification

#### **Leave Classification:**
```python
leave_type = models.CharField(choices=LEAVE_TYPE, max_length=20, default="casual")
```
**Leave Type Management:**
- **casual**: Casual leave-based rebate
- **vacation**: Vacation leave-based rebate
- Enables leave-type specific rebate policies
- Supports institutional leave policy integration

#### **Request Tracking:**
```python
status = models.CharField(max_length=20, choices=STATUS, default='1')
app_date = models.DateField(default=datetime.date.today)
rebate_remark = models.CharField(max_length=50, default='NA')
```
**Approval Workflow:**
- **status**: Request status (pending/accepted/rejected)
- **app_date**: Application submission date
- **rebate_remark**: Administrative comments/decision notes
- Enables systematic rebate approval and audit trail

### 2. **Vacation_food Model - Vacation Food Service**

```python
class Vacation_food(models.Model):
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    start_date = models.DateField(default=datetime.date.today)
    end_date = models.DateField(default=datetime.date.today)
    purpose = models.TextField()
    status = models.CharField(max_length=20, choices=STATUS, default='1')
    app_date = models.DateField(default=datetime.date.today)
```

#### **Vacation Service Management:**
- **start_date/end_date**: Vacation food service period
- **purpose**: Reason for requiring vacation food service
- **status**: Request approval status
- Enables vacation period food service for students staying on campus

### 3. **Special_request Model - Special Food Requests**

```python
class Special_request(models.Model):
    student_id = models.ForeignKey(Student, on_delete=models.CASCADE)
    start_date = models.DateField(default=datetime.date.today)
    end_date = models.DateField(default=datetime.date.today)
    request = models.TextField()
    status = models.CharField(max_length=20, choices=STATUS, default='1')
    item1 = models.CharField(choices=SPECIAL_FOOD, max_length=50, default='dal_chawal')
    item2 = models.CharField(choices=MEAL_TIME, max_length=50, default='breakfast')
    app_date = models.DateField(default=datetime.date.today)
```

#### **Field Analysis:**

#### **Special Food Specification:**
```python
item1 = models.CharField(choices=SPECIAL_FOOD, max_length=50, default='dal_chawal')
item2 = models.CharField(choices=MEAL_TIME, max_length=50, default='breakfast')
request = models.TextField()
```
**Special Food Options:**
- **item1**: Special food type (dal_chawal, khicdi, tomato_soup)
- **item2**: Meal timing (breakfast, lunch, dinner)
- **request**: Detailed special request description
- Enables medical/dietary special food arrangements

#### **Request Business Logic:**
```python
def submit_rebate_request(student, start_date, end_date, purpose, leave_type='casual'):
    """Submit rebate request for student"""
    
    # Validate date range
    if start_date > end_date:
        return False, "Start date cannot be after end date"
    
    if start_date < datetime.date.today():
        return False, "Cannot request rebate for past dates"
    
    # Check for overlapping requests
    overlapping = Rebate.objects.filter(
        student_id=student,
        status__in=['1', '2'],  # pending or accepted
        start_date__lte=end_date,
        end_date__gte=start_date
    )
    
    if overlapping.exists():
        return False, "Overlapping rebate request already exists"
    
    # Calculate rebate days and amount
    rebate_days = (end_date - start_date).days + 1
    daily_rate = get_daily_mess_rate(student.programme)
    estimated_rebate = rebate_days * daily_rate
    
    # Create rebate request
    rebate = Rebate.objects.create(
        student_id=student,
        start_date=start_date,
        end_date=end_date,
        purpose=purpose,
        leave_type=leave_type
    )
    
    # Notify administration
    notify_rebate_request(rebate)
    
    return True, f"Rebate request submitted for {rebate_days} days (Est. ₹{estimated_rebate})"

def process_rebate_request(rebate_id, action, remarks=""):
    """Process rebate request approval/rejection"""
    
    try:
        rebate = Rebate.objects.get(id=rebate_id)
        
        if action == 'approve':
            rebate.status = '2'  # accepted
            rebate.rebate_remark = remarks or "Approved"
            
            # Apply rebate to billing
            apply_rebate_to_billing(rebate)
            
        elif action == 'reject':
            rebate.status = '0'  # rejected
            rebate.rebate_remark = remarks or "Rejected"
        
        rebate.save()
        
        # Notify student
        notify_rebate_decision(rebate, action)
        
        return True, f"Rebate request {action}d successfully"
        
    except Rebate.DoesNotExist:
        return False, "Rebate request not found"

def submit_special_food_request(student, start_date, end_date, food_item, meal_time, reason):
    """Submit special food request"""
    
    # Check for existing active requests
    active_requests = Special_request.objects.filter(
        student_id=student,
        status='1',  # pending
        end_date__gte=datetime.date.today()
    )
    
    if active_requests.exists():
        return False, "You have pending special food requests"
    
    # Create special request
    special_request = Special_request.objects.create(
        student_id=student,
        start_date=start_date,
        end_date=end_date,
        request=reason,
        item1=food_item,
        item2=meal_time
    )
    
    # Notify mess administration
    notify_special_food_request(special_request)
    
    return True, "Special food request submitted successfully"

def get_student_active_requests(student):
    """Get all active requests for student"""
    
    active_requests = {
        'rebates': Rebate.objects.filter(
            student_id=student,
            status='1',  # pending
            end_date__gte=datetime.date.today()
        ),
        'special_food': Special_request.objects.filter(
            student_id=student,
            status='1',  # pending
            end_date__gte=datetime.date.today()
        ),
        'vacation_food': Vacation_food.objects.filter(
            student_id=student,
            status='1',  # pending
            end_date__gte=datetime.date.today()
        )
    }
    
    return active_requests
```

---

This completes Part 1 of the Central Mess documentation. Would you like me to continue with Part 2 covering the remaining models (Feedback, Mess Meetings, Administrative models, etc.)?
