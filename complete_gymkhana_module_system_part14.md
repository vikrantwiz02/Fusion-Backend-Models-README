# Complete Gymkhana Module System Documentation - Fusion IIIT (Part 14)

## View Functions and Business Logic Implementation

### Core View Functions Analysis

```python
# applications/gymkhana/views.py

from django.shortcuts import render, redirect, get_object_or_404
from django.contrib.auth.decorators import login_required
from django.contrib import messages
from django.http import JsonResponse, HttpResponse
from django.core.paginator import Paginator
from django.db.models import Q, Count, Sum, Avg
from django.utils import timezone
from django.views.decorators.csrf import csrf_exempt
from django.views.decorators.http import require_http_methods
import json
import datetime
from decimal import Decimal

from applications.globals.models import ExtraInfo, HoldsDesignation, Designation
from applications.academic_information.models import Student
from .models import (
    Club_info, Core_team, Club_member, Registration_form, Form_available,
    Session_info, Event_info, Club_budget, Club_report, Fest_budget,
    Other_report, Change_office, Voting_poll, Inventory
)
from .forms import (
    ClubRegistrationForm, EventCreationForm, BudgetAllocationForm,
    ReportSubmissionForm, VotingPollForm, InventoryForm
)
```

### 1. Dashboard and Overview Views

```python
@login_required
def gymkhana_dashboard(request):
    """
    CORE PURPOSE: Central dashboard for gymkhana activities and management
    
    BUSINESS LOGIC:
    - Provides comprehensive overview of all club activities
    - Shows personalized content based on user roles and memberships
    - Displays key metrics, upcoming events, and notifications
    - Supports role-based access control for different user types
    
    ANALYTICS INTEGRATION:
    - Real-time activity monitoring
    - Performance metrics dashboard
    - Trend analysis and insights
    """
    user = request.user
    current_session = Session_info.get_current_session()
    
    # Get user's club affiliations
    user_clubs = Club_member.objects.filter(
        member=user,
        session=current_session.session_id if current_session else None
    ).select_related('club')
    
    # Get user's leadership positions
    leadership_positions = Core_team.objects.filter(
        member=user,
        session=current_session.session_id if current_session else None
    ).select_related('club')
    
    # Dashboard metrics calculation
    dashboard_metrics = _calculate_dashboard_metrics(user, current_session)
    
    # Recent activities
    recent_activities = _get_recent_activities(user, limit=10)
    
    # Upcoming events
    upcoming_events = Event_info.objects.filter(
        event_date__gte=timezone.now(),
        event_status__in=['approved', 'ongoing']
    ).order_by('event_date')[:5]
    
    # Pending approvals (for administrators)
    pending_approvals = _get_pending_approvals(user)
    
    # Notifications
    notifications = _get_user_notifications(user, limit=5)
    
    # Performance analytics
    performance_analytics = _generate_performance_analytics(user, current_session)
    
    context = {
        'user_clubs': user_clubs,
        'leadership_positions': leadership_positions,
        'dashboard_metrics': dashboard_metrics,
        'recent_activities': recent_activities,
        'upcoming_events': upcoming_events,
        'pending_approvals': pending_approvals,
        'notifications': notifications,
        'performance_analytics': performance_analytics,
        'current_session': current_session,
        'is_club_admin': _is_club_admin(user),
        'is_gymkhana_admin': _is_gymkhana_admin(user)
    }
    
    return render(request, 'gymkhana/dashboard.html', context)

def _calculate_dashboard_metrics(user, current_session):
    """Helper function to calculate dashboard metrics"""
    metrics = {}
    
    # User-specific metrics
    metrics['user_clubs_count'] = Club_member.objects.filter(
        member=user,
        session=current_session.session_id if current_session else None
    ).count()
    
    metrics['user_events_participated'] = Event_info.objects.filter(
        participants__contains=[{'user_id': user.id}],
        session=current_session.session_id if current_session else None
    ).count()
    
    # System-wide metrics
    metrics['total_clubs'] = Club_info.objects.filter(status='active').count()
    metrics['total_members'] = Club_member.objects.filter(
        session=current_session.session_id if current_session else None
    ).count()
    metrics['total_events_this_month'] = Event_info.objects.filter(
        event_date__month=timezone.now().month,
        event_date__year=timezone.now().year
    ).count()
    metrics['total_budget_allocated'] = Club_budget.objects.filter(
        session=current_session.session_id if current_session else None
    ).aggregate(total=Sum('total_budget'))['total'] or 0
    
    return metrics

def _get_recent_activities(user, limit=10):
    """Helper function to get recent activities"""
    activities = []
    
    # Recent club memberships
    recent_memberships = Club_member.objects.filter(
        member=user
    ).order_by('-created_at')[:limit//2]
    
    for membership in recent_memberships:
        activities.append({
            'type': 'membership',
            'description': f"Joined {membership.club.club_name}",
            'timestamp': membership.created_at,
            'related_object': membership
        })
    
    # Recent event participations
    recent_events = Event_info.objects.filter(
        participants__contains=[{'user_id': user.id}]
    ).order_by('-event_date')[:limit//2]
    
    for event in recent_events:
        activities.append({
            'type': 'event',
            'description': f"Participated in {event.event_name}",
            'timestamp': event.event_date,
            'related_object': event
        })
    
    # Sort by timestamp and limit
    activities.sort(key=lambda x: x['timestamp'], reverse=True)
    return activities[:limit]

def _get_pending_approvals(user):
    """Helper function to get pending approvals for user"""
    pending = []
    
    if _is_club_admin(user) or _is_gymkhana_admin(user):
        # Pending club registrations
        pending_clubs = Club_info.objects.filter(status='pending_approval').count()
        if pending_clubs > 0:
            pending.append({
                'type': 'club_approval',
                'count': pending_clubs,
                'description': f"{pending_clubs} clubs pending approval"
            })
        
        # Pending event approvals
        pending_events = Event_info.objects.filter(approval_status='pending').count()
        if pending_events > 0:
            pending.append({
                'type': 'event_approval',
                'count': pending_events,
                'description': f"{pending_events} events pending approval"
            })
        
        # Pending budget requests
        pending_budgets = Club_budget.objects.filter(approval_status='pending').count()
        if pending_budgets > 0:
            pending.append({
                'type': 'budget_approval',
                'count': pending_budgets,
                'description': f"{pending_budgets} budget requests pending"
            })
    
    return pending

def _get_user_notifications(user, limit=5):
    """Helper function to get user notifications"""
    # This would integrate with the notification system
    # For now, return mock notifications
    notifications = [
        {
            'type': 'info',
            'message': 'New event registration open',
            'timestamp': timezone.now() - timezone.timedelta(hours=2)
        },
        {
            'type': 'warning',
            'message': 'Budget submission deadline approaching',
            'timestamp': timezone.now() - timezone.timedelta(days=1)
        }
    ]
    return notifications[:limit]

def _generate_performance_analytics(user, current_session):
    """Helper function to generate performance analytics"""
    analytics = {}
    
    # User engagement score
    user_clubs = Club_member.objects.filter(
        member=user,
        session=current_session.session_id if current_session else None
    ).count()
    
    user_events = Event_info.objects.filter(
        participants__contains=[{'user_id': user.id}],
        session=current_session.session_id if current_session else None
    ).count()
    
    # Calculate engagement score (0-100)
    engagement_score = min(100, (user_clubs * 20) + (user_events * 5))
    analytics['engagement_score'] = engagement_score
    
    # Participation trend
    monthly_participation = []
    for i in range(6):  # Last 6 months
        month_date = timezone.now() - timezone.timedelta(days=30*i)
        month_events = Event_info.objects.filter(
            participants__contains=[{'user_id': user.id}],
            event_date__month=month_date.month,
            event_date__year=month_date.year
        ).count()
        monthly_participation.append({
            'month': month_date.strftime('%b %Y'),
            'events': month_events
        })
    
    analytics['participation_trend'] = list(reversed(monthly_participation))
    
    return analytics

def _is_club_admin(user):
    """Helper function to check if user is club admin"""
    return Core_team.objects.filter(
        member=user,
        position_title__in=['President', 'Vice President', 'Secretary']
    ).exists()

def _is_gymkhana_admin(user):
    """Helper function to check if user is gymkhana admin"""
    try:
        extra_info = ExtraInfo.objects.get(user=user)
        designations = HoldsDesignation.objects.filter(
            user=extra_info,
            designation__name__icontains='gymkhana'
        )
        return designations.exists()
    except ExtraInfo.DoesNotExist:
        return False
```

### 2. Club Management Views

```python
@login_required
def club_list(request):
    """
    CORE PURPOSE: Display comprehensive list of all clubs with filtering and search
    
    BUSINESS LOGIC:
    - Shows all active clubs with detailed information
    - Provides filtering by category, status, and other criteria
    - Supports search functionality across club attributes
    - Includes pagination for performance optimization
    """
    clubs = Club_info.objects.filter(status='active')
    
    # Search functionality
    search_query = request.GET.get('search', '')
    if search_query:
        clubs = clubs.filter(
            Q(club_name__icontains=search_query) |
            Q(club_description__icontains=search_query) |
            Q(category__icontains=search_query)
        )
    
    # Category filter
    category_filter = request.GET.get('category', '')
    if category_filter:
        clubs = clubs.filter(category=category_filter)
    
    # Status filter
    status_filter = request.GET.get('status', '')
    if status_filter:
        clubs = clubs.filter(status=status_filter)
    
    # Sorting
    sort_by = request.GET.get('sort', 'club_name')
    valid_sort_fields = ['club_name', 'category', 'total_members', 'events_conducted', 'created_at']
    if sort_by in valid_sort_fields:
        clubs = clubs.order_by(sort_by)
    
    # Pagination
    paginator = Paginator(clubs, 12)  # 12 clubs per page
    page_number = request.GET.get('page')
    page_clubs = paginator.get_page(page_number)
    
    # Categories for filter dropdown
    categories = Club_info.objects.values_list('category', flat=True).distinct()
    
    # Analytics for each club
    for club in page_clubs:
        club.analytics = club.get_comprehensive_club_analytics()
    
    context = {
        'clubs': page_clubs,
        'categories': categories,
        'search_query': search_query,
        'category_filter': category_filter,
        'status_filter': status_filter,
        'sort_by': sort_by
    }
    
    return render(request, 'gymkhana/club_list.html', context)

@login_required
def club_detail(request, club_id):
    """
    CORE PURPOSE: Detailed view of individual club with comprehensive information
    
    BUSINESS LOGIC:
    - Shows complete club profile and statistics
    - Displays club members, events, and activities
    - Provides join/leave functionality for members
    - Shows financial information for authorized users
    """
    club = get_object_or_404(Club_info, club_id=club_id)
    current_session = Session_info.get_current_session()
    user = request.user
    
    # Check if user is member
    is_member = Club_member.objects.filter(
        club=club,
        member=user,
        session=current_session.session_id if current_session else None
    ).exists()
    
    # Check if user has leadership role
    leadership_role = Core_team.objects.filter(
        club=club,
        member=user,
        session=current_session.session_id if current_session else None
    ).first()
    
    # Get club members
    club_members = Club_member.objects.filter(
        club=club,
        session=current_session.session_id if current_session else None
    ).select_related('member')
    
    # Get core team
    core_team = Core_team.objects.filter(
        club=club,
        session=current_session.session_id if current_session else None
    ).select_related('member')
    
    # Get recent events
    recent_events = Event_info.objects.filter(
        organizing_club=club
    ).order_by('-event_date')[:5]
    
    # Get upcoming events
    upcoming_events = Event_info.objects.filter(
        organizing_club=club,
        event_date__gte=timezone.now()
    ).order_by('event_date')[:5]
    
    # Club analytics
    club_analytics = club.get_comprehensive_club_analytics()
    
    # Financial information (for authorized users)
    financial_info = None
    if leadership_role or _is_gymkhana_admin(user):
        try:
            budget = Club_budget.objects.get(
                club=club,
                session=current_session.session_id if current_session else None
            )
            financial_info = budget.get_comprehensive_budget_analytics()
        except Club_budget.DoesNotExist:
            financial_info = None
    
    # Recent reports
    recent_reports = Club_report.objects.filter(
        club=club
    ).order_by('-created_at')[:3]
    
    context = {
        'club': club,
        'is_member': is_member,
        'leadership_role': leadership_role,
        'club_members': club_members,
        'core_team': core_team,
        'recent_events': recent_events,
        'upcoming_events': upcoming_events,
        'club_analytics': club_analytics,
        'financial_info': financial_info,
        'recent_reports': recent_reports,
        'current_session': current_session
    }
    
    return render(request, 'gymkhana/club_detail.html', context)

@login_required
@require_http_methods(["POST"])
def join_club(request, club_id):
    """
    CORE PURPOSE: Handle club membership requests and approvals
    
    BUSINESS LOGIC:
    - Processes club join requests
    - Validates eligibility criteria
    - Handles approval workflow
    - Updates club statistics
    """
    club = get_object_or_404(Club_info, club_id=club_id)
    user = request.user
    current_session = Session_info.get_current_session()
    
    # Check if already a member
    existing_membership = Club_member.objects.filter(
        club=club,
        member=user,
        session=current_session.session_id if current_session else None
    ).first()
    
    if existing_membership:
        messages.warning(request, 'You are already a member of this club.')
        return redirect('club_detail', club_id=club_id)
    
    # Check eligibility
    eligibility_check = _check_club_eligibility(user, club)
    if not eligibility_check['eligible']:
        messages.error(request, f'Not eligible to join: {eligibility_check["reason"]}')
        return redirect('club_detail', club_id=club_id)
    
    # Create membership request
    membership = Club_member.objects.create(
        club=club,
        member=user,
        session=current_session.session_id if current_session else None,
        member_type='general',
        membership_status='pending',
        joined_date=timezone.now().date()
    )
    
    # Auto-approve or send for approval based on club settings
    if club.auto_approve_members:
        membership.membership_status = 'active'
        membership.approved_by = club.club_coordinator
        membership.approval_date = timezone.now()
        membership.save()
        
        messages.success(request, 'Successfully joined the club!')
    else:
        messages.info(request, 'Membership request submitted. Waiting for approval.')
    
    # Update club statistics
    club.total_members = club.get_total_members()
    club.save()
    
    return redirect('club_detail', club_id=club_id)

def _check_club_eligibility(user, club):
    """Helper function to check club eligibility"""
    try:
        student = Student.objects.get(id=user)
        
        # Check academic standing
        if club.min_cgpa_required > 0:
            # This would check actual CGPA from academic records
            # For now, assume eligible
            pass
        
        # Check year/branch restrictions
        if club.year_restrictions:
            # Check if user's year is in allowed years
            pass
        
        if club.branch_restrictions:
            # Check if user's branch is in allowed branches
            pass
        
        # Check maximum membership limit
        current_members = Club_member.objects.filter(
            club=club,
            membership_status='active'
        ).count()
        
        if club.max_members > 0 and current_members >= club.max_members:
            return {
                'eligible': False,
                'reason': 'Club has reached maximum membership limit'
            }
        
        return {'eligible': True, 'reason': ''}
        
    except Student.DoesNotExist:
        return {
            'eligible': False,
            'reason': 'Student record not found'
        }

@login_required
def create_club(request):
    """
    CORE PURPOSE: Handle new club creation with comprehensive validation
    
    BUSINESS LOGIC:
    - Validates club creation eligibility
    - Processes club registration form
    - Handles approval workflow
    - Sets up initial club structure
    """
    if request.method == 'POST':
        form = ClubRegistrationForm(request.POST, request.FILES)
        if form.is_valid():
            club = form.save(commit=False)
            club.created_by = request.user
            club.status = 'pending_approval'
            club.save()
            
            # Create initial core team entry for creator
            Core_team.objects.create(
                club=club,
                member=request.user,
                position_title='Founder',
                position_level='president',
                session=Session_info.get_current_session().session_id,
                appointed_date=timezone.now().date(),
                term_start_date=timezone.now().date(),
                term_end_date=timezone.now().date() + timezone.timedelta(days=365)
            )
            
            messages.success(request, 'Club registration submitted for approval!')
            return redirect('club_detail', club_id=club.club_id)
    else:
        form = ClubRegistrationForm()
    
    context = {'form': form}
    return render(request, 'gymkhana/create_club.html', context)
```

### 3. Event Management Views

```python
@login_required
def event_list(request):
    """
    CORE PURPOSE: Comprehensive event listing with filtering and analytics
    
    BUSINESS LOGIC:
    - Displays all events with detailed information
    - Provides filtering by date, club, category, status
    - Shows event analytics and participation metrics
    - Supports event registration and management
    """
    events = Event_info.objects.all()
    
    # Date filtering
    date_filter = request.GET.get('date_filter', 'all')
    if date_filter == 'upcoming':
        events = events.filter(event_date__gte=timezone.now())
    elif date_filter == 'past':
        events = events.filter(event_date__lt=timezone.now())
    elif date_filter == 'this_month':
        events = events.filter(
            event_date__month=timezone.now().month,
            event_date__year=timezone.now().year
        )
    
    # Club filtering
    club_filter = request.GET.get('club', '')
    if club_filter:
        events = events.filter(organizing_club__club_id=club_filter)
    
    # Category filtering
    category_filter = request.GET.get('category', '')
    if category_filter:
        events = events.filter(event_category=category_filter)
    
    # Status filtering
    status_filter = request.GET.get('status', '')
    if status_filter:
        events = events.filter(event_status=status_filter)
    
    # Search
    search_query = request.GET.get('search', '')
    if search_query:
        events = events.filter(
            Q(event_name__icontains=search_query) |
            Q(event_description__icontains=search_query) |
            Q(organizing_club__club_name__icontains=search_query)
        )
    
    # Sorting
    sort_by = request.GET.get('sort', '-event_date')
    events = events.order_by(sort_by)
    
    # Pagination
    paginator = Paginator(events, 10)
    page_number = request.GET.get('page')
    page_events = paginator.get_page(page_number)
    
    # Add analytics to each event
    for event in page_events:
        event.analytics = event.get_comprehensive_event_analytics()
    
    # Filter options
    clubs = Club_info.objects.filter(status='active')
    categories = Event_info.objects.values_list('event_category', flat=True).distinct()
    statuses = Event_info.objects.values_list('event_status', flat=True).distinct()
    
    context = {
        'events': page_events,
        'clubs': clubs,
        'categories': categories,
        'statuses': statuses,
        'date_filter': date_filter,
        'club_filter': club_filter,
        'category_filter': category_filter,
        'status_filter': status_filter,
        'search_query': search_query,
        'sort_by': sort_by
    }
    
    return render(request, 'gymkhana/event_list.html', context)

@login_required
def event_detail(request, event_id):
    """
    CORE PURPOSE: Detailed event view with registration and management features
    
    BUSINESS LOGIC:
    - Shows comprehensive event information
    - Handles event registration and participation
    - Provides event analytics and metrics
    - Supports event management for organizers
    """
    event = get_object_or_404(Event_info, event_id=event_id)
    user = request.user
    
    # Check if user is registered
    is_registered = any(
        participant.get('user_id') == user.id 
        for participant in event.participants
    )
    
    # Check if user can manage event
    can_manage = (
        event.event_coordinator == user or
        _is_club_admin(user) or
        _is_gymkhana_admin(user)
    )
    
    # Get event analytics
    event_analytics = event.get_comprehensive_event_analytics()
    
    # Get participant list (for managers)
    participants = None
    if can_manage:
        participants = _get_event_participants(event)
    
    # Registration status
    registration_status = _get_registration_status(event)
    
    # Similar events
    similar_events = Event_info.objects.filter(
        event_category=event.event_category,
        organizing_club=event.organizing_club
    ).exclude(event_id=event_id)[:3]
    
    context = {
        'event': event,
        'is_registered': is_registered,
        'can_manage': can_manage,
        'event_analytics': event_analytics,
        'participants': participants,
        'registration_status': registration_status,
        'similar_events': similar_events
    }
    
    return render(request, 'gymkhana/event_detail.html', context)

@login_required
@require_http_methods(["POST"])
def register_event(request, event_id):
    """
    CORE PURPOSE: Handle event registration with validation and confirmation
    
    BUSINESS LOGIC:
    - Validates registration eligibility
    - Processes registration form
    - Handles payment if required
    - Updates event statistics
    """
    event = get_object_or_404(Event_info, event_id=event_id)
    user = request.user
    
    # Check if already registered
    is_already_registered = any(
        participant.get('user_id') == user.id 
        for participant in event.participants
    )
    
    if is_already_registered:
        return JsonResponse({
            'status': 'error',
            'message': 'Already registered for this event'
        })
    
    # Check registration eligibility
    eligibility_check = _check_event_eligibility(user, event)
    if not eligibility_check['eligible']:
        return JsonResponse({
            'status': 'error',
            'message': eligibility_check['reason']
        })
    
    # Add participant
    participant_data = {
        'user_id': user.id,
        'username': user.username,
        'registration_date': timezone.now().isoformat(),
        'payment_status': 'pending' if event.registration_fee > 0 else 'completed',
        'attendance_status': 'registered'
    }
    
    event.participants.append(participant_data)
    event.registered_participants += 1
    event.save()
    
    # Handle payment if required
    if event.registration_fee > 0:
        # This would integrate with payment system
        payment_url = _initiate_payment(user, event)
        return JsonResponse({
            'status': 'payment_required',
            'message': 'Registration successful. Please complete payment.',
            'payment_url': payment_url
        })
    
    return JsonResponse({
        'status': 'success',
        'message': 'Successfully registered for the event!'
    })

def _check_event_eligibility(user, event):
    """Helper function to check event registration eligibility"""
    # Check registration deadline
    if event.registration_deadline and timezone.now() > event.registration_deadline:
        return {
            'eligible': False,
            'reason': 'Registration deadline has passed'
        }
    
    # Check capacity
    if event.max_participants > 0 and event.registered_participants >= event.max_participants:
        return {
            'eligible': False,
            'reason': 'Event is full'
        }
    
    # Check if event is active
    if event.event_status not in ['approved', 'ongoing']:
        return {
            'eligible': False,
            'reason': 'Event registration is not open'
        }
    
    # Check club membership requirement
    if event.club_members_only:
        is_club_member = Club_member.objects.filter(
            club=event.organizing_club,
            member=user,
            membership_status='active'
        ).exists()
        
        if not is_club_member:
            return {
                'eligible': False,
                'reason': 'This event is only for club members'
            }
    
    return {'eligible': True, 'reason': ''}

def _get_event_participants(event):
    """Helper function to get event participants with details"""
    participants = []
    for participant_data in event.participants:
        try:
            user = User.objects.get(id=participant_data['user_id'])
            participants.append({
                'user': user,
                'registration_date': participant_data.get('registration_date'),
                'payment_status': participant_data.get('payment_status', 'pending'),
                'attendance_status': participant_data.get('attendance_status', 'registered')
            })
        except User.DoesNotExist:
            continue
    
    return participants

def _get_registration_status(event):
    """Helper function to get event registration status"""
    now = timezone.now()
    
    if event.registration_deadline and now > event.registration_deadline:
        return 'closed'
    elif event.max_participants > 0 and event.registered_participants >= event.max_participants:
        return 'full'
    elif event.event_status in ['approved', 'ongoing']:
        return 'open'
    else:
        return 'not_open'

@login_required
def create_event(request):
    """
    CORE PURPOSE: Handle new event creation with comprehensive validation
    
    BUSINESS LOGIC:
    - Validates event creation permissions
    - Processes event creation form
    - Handles approval workflow
    - Sets up event structure and notifications
    """
    # Check if user can create events
    user_clubs = Club_member.objects.filter(
        member=request.user,
        membership_status='active'
    )
    
    if not user_clubs.exists() and not _is_gymkhana_admin(request.user):
        messages.error(request, 'You must be a member of at least one club to create events.')
        return redirect('event_list')
    
    if request.method == 'POST':
        form = EventCreationForm(request.POST, request.FILES, user=request.user)
        if form.is_valid():
            event = form.save(commit=False)
            event.event_coordinator = request.user
            event.created_by = request.user
            event.approval_status = 'pending'
            event.save()
            
            messages.success(request, 'Event created successfully and submitted for approval!')
            return redirect('event_detail', event_id=event.event_id)
    else:
        form = EventCreationForm(user=request.user)
    
    context = {'form': form}
    return render(request, 'gymkhana/create_event.html', context)
```

### 4. Budget and Financial Management Views

```python
@login_required
def budget_overview(request):
    """
    CORE PURPOSE: Comprehensive budget overview and financial analytics
    
    BUSINESS LOGIC:
    - Shows budget allocation across all clubs
    - Provides financial analytics and insights
    - Displays spending patterns and efficiency metrics
    - Supports budget monitoring and approval
    """
    current_session = Session_info.get_current_session()
    
    # Get all club budgets for current session
    club_budgets = Club_budget.objects.filter(
        session=current_session.session_id if current_session else None
    ).select_related('club')
    
    # Calculate overall statistics
    total_allocated = club_budgets.aggregate(
        total=Sum('total_budget')
    )['total'] or 0
    
    total_spent = club_budgets.aggregate(
        total=Sum('total_expenditure')
    )['total'] or 0
    
    # Budget utilization
    budget_utilization = (total_spent / total_allocated * 100) if total_allocated > 0 else 0
    
    # Top spending clubs
    top_spending_clubs = club_budgets.order_by('-total_expenditure')[:5]
    
    # Budget efficiency analysis
    efficiency_analysis = []
    for budget in club_budgets:
        analytics = budget.get_comprehensive_budget_analytics()
        efficiency_analysis.append({
            'club': budget.club,
            'budget': budget,
            'analytics': analytics
        })
    
    # Monthly spending trend
    monthly_trend = _calculate_monthly_spending_trend(current_session)
    
    # Pending budget requests
    pending_requests = Club_budget.objects.filter(
        approval_status='pending'
    ).count()
    
    context = {
        'club_budgets': club_budgets,
        'total_allocated': total_allocated,
        'total_spent': total_spent,
        'budget_utilization': budget_utilization,
        'top_spending_clubs': top_spending_clubs,
        'efficiency_analysis': efficiency_analysis,
        'monthly_trend': monthly_trend,
        'pending_requests': pending_requests,
        'current_session': current_session
    }
    
    return render(request, 'gymkhana/budget_overview.html', context)

def _calculate_monthly_spending_trend(session):
    """Helper function to calculate monthly spending trend"""
    monthly_data = []
    
    for i in range(12):  # Last 12 months
        month_date = timezone.now() - timezone.timedelta(days=30*i)
        
        # This would calculate actual monthly spending
        # For now, return mock data
        monthly_spending = Club_budget.objects.filter(
            session=session.session_id if session else None,
            updated_at__month=month_date.month,
            updated_at__year=month_date.year
        ).aggregate(total=Sum('total_expenditure'))['total'] or 0
        
        monthly_data.append({
            'month': month_date.strftime('%b %Y'),
            'spending': float(monthly_spending)
        })
    
    return list(reversed(monthly_data))

@login_required
def club_budget_detail(request, club_id):
    """
    CORE PURPOSE: Detailed budget view for specific club
    
    BUSINESS LOGIC:
    - Shows comprehensive budget breakdown
    - Provides spending analytics and insights
    - Handles budget modifications and approvals
    - Displays financial performance metrics
    """
    club = get_object_or_404(Club_info, club_id=club_id)
    current_session = Session_info.get_current_session()
    
    try:
        budget = Club_budget.objects.get(
            club=club,
            session=current_session.session_id if current_session else None
        )
    except Club_budget.DoesNotExist:
        budget = None
    
    # Check permissions
    can_view_financial = (
        _is_club_admin(request.user) or
        _is_gymkhana_admin(request.user) or
        Club_member.objects.filter(
            club=club,
            member=request.user,
            membership_status='active'
        ).exists()
    )
    
    if not can_view_financial:
        messages.error(request, 'You do not have permission to view this budget.')
        return redirect('club_detail', club_id=club_id)
    
    # Get budget analytics
    budget_analytics = None
    if budget:
        budget_analytics = budget.get_comprehensive_budget_analytics()
    
    # Get related events and their costs
    club_events = Event_info.objects.filter(
        organizing_club=club,
        session=current_session.session_id if current_session else None
    )
    
    # Spending history
    spending_history = _get_spending_history(club, current_session)
    
    context = {
        'club': club,
        'budget': budget,
        'budget_analytics': budget_analytics,
        'club_events': club_events,
        'spending_history': spending_history,
        'can_edit': _is_club_admin(request.user) or _is_gymkhana_admin(request.user)
    }
    
    return render(request, 'gymkhana/club_budget_detail.html', context)

def _get_spending_history(club, session):
    """Helper function to get spending history"""
    # This would get actual transaction history
    # For now, return mock data based on events
    history = []
    
    events = Event_info.objects.filter(
        organizing_club=club,
        session=session.session_id if session else None
    ).order_by('-event_date')
    
    for event in events:
        if event.estimated_budget > 0:
            history.append({
                'date': event.event_date,
                'description': f"Event: {event.event_name}",
                'amount': event.estimated_budget,
                'type': 'event_expense',
                'status': 'completed' if event.event_status == 'completed' else 'planned'
            })
    
    return history[:10]  # Last 10 transactions
```

This comprehensive view functions implementation covers the core business logic for the gymkhana module.