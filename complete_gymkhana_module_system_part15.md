# Complete Gymkhana Module System Documentation - Fusion IIIT (Part 15)

## Advanced View Functions, API Endpoints, and Templates

### 5. Reporting and Analytics Views

```python
@login_required
def reports_dashboard(request):
    """
    CORE PURPOSE: Comprehensive reporting dashboard with advanced analytics
    
    BUSINESS LOGIC:
    - Provides centralized access to all reports and analytics
    - Supports custom report generation and filtering
    - Displays key performance indicators and trends
    - Handles report scheduling and distribution
    """
    current_session = Session_info.get_current_session()
    user = request.user
    
    # Get user's report access permissions
    accessible_clubs = _get_user_accessible_clubs(user)
    
    # Recent reports
    recent_reports = Club_report.objects.filter(
        club__in=accessible_clubs
    ).order_by('-created_at')[:10]
    
    # Other reports
    other_reports = Other_report.objects.filter(
        Q(requested_by=user) | Q(prepared_by=user) | Q(confidentiality_level='public')
    ).order_by('-created_at')[:10]
    
    # Report analytics
    report_analytics = _generate_report_analytics(accessible_clubs, current_session)
    
    # Scheduled reports
    scheduled_reports = _get_scheduled_reports(user)
    
    # Report templates
    report_templates = _get_available_report_templates()
    
    context = {
        'recent_reports': recent_reports,
        'other_reports': other_reports,
        'report_analytics': report_analytics,
        'scheduled_reports': scheduled_reports,
        'report_templates': report_templates,
        'accessible_clubs': accessible_clubs
    }
    
    return render(request, 'gymkhana/reports_dashboard.html', context)

def _get_user_accessible_clubs(user):
    """Helper function to get clubs user can access reports for"""
    accessible_clubs = []
    
    # Clubs where user is a member
    member_clubs = Club_info.objects.filter(
        club_members__member=user,
        club_members__membership_status='active'
    )
    accessible_clubs.extend(member_clubs)
    
    # Clubs where user has leadership role
    leadership_clubs = Club_info.objects.filter(
        core_team__member=user
    )
    accessible_clubs.extend(leadership_clubs)
    
    # All clubs if gymkhana admin
    if _is_gymkhana_admin(user):
        accessible_clubs = Club_info.objects.filter(status='active')
    
    return list(set(accessible_clubs))

def _generate_report_analytics(clubs, session):
    """Helper function to generate report analytics"""
    analytics = {
        'total_reports': Club_report.objects.filter(club__in=clubs).count(),
        'pending_reports': Club_report.objects.filter(
            club__in=clubs,
            submission_status='pending'
        ).count(),
        'overdue_reports': Club_report.objects.filter(
            club__in=clubs,
            submission_deadline__lt=timezone.now(),
            submission_status__in=['draft', 'pending']
        ).count()
    }
    
    # Report submission trends
    monthly_submissions = []
    for i in range(6):
        month_date = timezone.now() - timezone.timedelta(days=30*i)
        month_reports = Club_report.objects.filter(
            club__in=clubs,
            created_at__month=month_date.month,
            created_at__year=month_date.year
        ).count()
        monthly_submissions.append({
            'month': month_date.strftime('%b %Y'),
            'reports': month_reports
        })
    
    analytics['submission_trend'] = list(reversed(monthly_submissions))
    
    return analytics

@login_required
def generate_custom_report(request):
    """
    CORE PURPOSE: Custom report generation with flexible parameters
    
    BUSINESS LOGIC:
    - Allows users to create custom reports with specific criteria
    - Supports multiple data sources and analytics
    - Provides export functionality in various formats
    - Handles report scheduling and automation
    """
    if request.method == 'POST':
        # Get report parameters
        report_type = request.POST.get('report_type')
        date_range = request.POST.get('date_range')
        club_filter = request.POST.getlist('clubs')
        metrics = request.POST.getlist('metrics')
        export_format = request.POST.get('export_format', 'html')
        
        # Generate report data
        report_data = _generate_custom_report_data(
            report_type, date_range, club_filter, metrics
        )
        
        # Handle export
        if export_format == 'pdf':
            return _export_report_pdf(report_data)
        elif export_format == 'excel':
            return _export_report_excel(report_data)
        elif export_format == 'csv':
            return _export_report_csv(report_data)
        else:
            # Return HTML view
            context = {'report_data': report_data}
            return render(request, 'gymkhana/custom_report_view.html', context)
    
    # GET request - show report generation form
    clubs = Club_info.objects.filter(status='active')
    report_types = [
        ('club_performance', 'Club Performance'),
        ('event_analytics', 'Event Analytics'),
        ('financial_summary', 'Financial Summary'),
        ('member_engagement', 'Member Engagement'),
        ('comparative_analysis', 'Comparative Analysis')
    ]
    
    context = {
        'clubs': clubs,
        'report_types': report_types
    }
    
    return render(request, 'gymkhana/generate_custom_report.html', context)

def _generate_custom_report_data(report_type, date_range, club_filter, metrics):
    """Helper function to generate custom report data"""
    # Parse date range
    start_date, end_date = _parse_date_range(date_range)
    
    # Filter clubs
    clubs = Club_info.objects.filter(club_id__in=club_filter) if club_filter else Club_info.objects.filter(status='active')
    
    report_data = {
        'report_type': report_type,
        'date_range': f"{start_date} to {end_date}",
        'clubs': clubs,
        'generated_at': timezone.now(),
        'data': {}
    }
    
    if report_type == 'club_performance':
        report_data['data'] = _generate_club_performance_data(clubs, start_date, end_date, metrics)
    elif report_type == 'event_analytics':
        report_data['data'] = _generate_event_analytics_data(clubs, start_date, end_date, metrics)
    elif report_type == 'financial_summary':
        report_data['data'] = _generate_financial_summary_data(clubs, start_date, end_date, metrics)
    elif report_type == 'member_engagement':
        report_data['data'] = _generate_member_engagement_data(clubs, start_date, end_date, metrics)
    elif report_type == 'comparative_analysis':
        report_data['data'] = _generate_comparative_analysis_data(clubs, start_date, end_date, metrics)
    
    return report_data

def _generate_club_performance_data(clubs, start_date, end_date, metrics):
    """Helper function to generate club performance report data"""
    performance_data = []
    
    for club in clubs:
        club_analytics = club.get_comprehensive_club_analytics()
        
        club_data = {
            'club': club,
            'analytics': club_analytics,
            'member_count': club.get_total_members(),
            'events_conducted': Event_info.objects.filter(
                organizing_club=club,
                event_date__range=[start_date, end_date]
            ).count(),
            'budget_utilization': 0,  # Would calculate from actual budget
            'performance_score': club_analytics.get('overall_performance', {}).get('composite_score', 0)
        }
        
        # Add budget information if available
        try:
            budget = Club_budget.objects.get(club=club)
            budget_analytics = budget.get_comprehensive_budget_analytics()
            club_data['budget_analytics'] = budget_analytics
            club_data['budget_utilization'] = budget_analytics.get('utilization_metrics', {}).get('overall_utilization_percentage', 0)
        except Club_budget.DoesNotExist:
            club_data['budget_analytics'] = None
        
        performance_data.append(club_data)
    
    # Sort by performance score
    performance_data.sort(key=lambda x: x['performance_score'], reverse=True)
    
    return {
        'club_performance': performance_data,
        'summary': {
            'total_clubs': len(performance_data),
            'avg_performance_score': sum(c['performance_score'] for c in performance_data) / len(performance_data) if performance_data else 0,
            'top_performer': performance_data[0] if performance_data else None,
            'total_events': sum(c['events_conducted'] for c in performance_data),
            'avg_budget_utilization': sum(c['budget_utilization'] for c in performance_data) / len(performance_data) if performance_data else 0
        }
    }

@login_required
def voting_management(request):
    """
    CORE PURPOSE: Comprehensive voting and election management system
    
    BUSINESS LOGIC:
    - Manages all voting polls and elections
    - Provides voting analytics and results
    - Handles election administration and oversight
    - Supports democratic decision-making processes
    """
    user = request.user
    current_session = Session_info.get_current_session()
    
    # Get user's voting permissions
    can_create_polls = _can_create_voting_polls(user)
    can_manage_elections = _is_gymkhana_admin(user)
    
    # Active polls
    active_polls = Voting_poll.objects.filter(
        poll_status__in=['registration_open', 'voting_active']
    ).order_by('voting_end_date')
    
    # Recent polls
    recent_polls = Voting_poll.objects.filter(
        poll_status='results_announced'
    ).order_by('-result_announcement_date')[:5]
    
    # Polls user can vote in
    eligible_polls = []
    for poll in active_polls:
        if _can_user_vote_in_poll(user, poll):
            eligible_polls.append(poll)
    
    # Election statistics
    election_stats = _generate_election_statistics(current_session)
    
    # Upcoming elections
    upcoming_elections = Voting_poll.objects.filter(
        poll_type='election',
        voting_start_date__gt=timezone.now()
    ).order_by('voting_start_date')[:3]
    
    context = {
        'can_create_polls': can_create_polls,
        'can_manage_elections': can_manage_elections,
        'active_polls': active_polls,
        'recent_polls': recent_polls,
        'eligible_polls': eligible_polls,
        'election_stats': election_stats,
        'upcoming_elections': upcoming_elections
    }
    
    return render(request, 'gymkhana/voting_management.html', context)

def _can_create_voting_polls(user):
    """Helper function to check if user can create voting polls"""
    return (
        _is_club_admin(user) or
        _is_gymkhana_admin(user) or
        Core_team.objects.filter(
            member=user,
            position_title__in=['President', 'Secretary', 'Election Officer']
        ).exists()
    )

def _can_user_vote_in_poll(user, poll):
    """Helper function to check if user can vote in a specific poll"""
    # Check if user meets eligibility criteria
    if poll.eligible_voter_criteria:
        criteria = poll.eligible_voter_criteria
        
        # Check club membership if required
        if criteria.get('club_membership_required'):
            is_club_member = Club_member.objects.filter(
                club=poll.club,
                member=user,
                membership_status='active'
            ).exists()
            if not is_club_member:
                return False
        
        # Check minimum membership duration
        if poll.minimum_membership_duration > 0:
            membership = Club_member.objects.filter(
                club=poll.club,
                member=user,
                membership_status='active'
            ).first()
            if membership:
                membership_days = (timezone.now().date() - membership.joined_date).days
                if membership_days < poll.minimum_membership_duration:
                    return False
            else:
                return False
    
    return True

def _generate_election_statistics(session):
    """Helper function to generate election statistics"""
    stats = {}
    
    # Total elections conducted
    stats['total_elections'] = Voting_poll.objects.filter(
        poll_type='election',
        session=session.session_id if session else None
    ).count()
    
    # Average voter turnout
    completed_elections = Voting_poll.objects.filter(
        poll_type='election',
        poll_status='results_announced',
        session=session.session_id if session else None
    )
    
    if completed_elections.exists():
        avg_turnout = completed_elections.aggregate(
            avg_turnout=Avg('voter_turnout_percentage')
        )['avg_turnout'] or 0
        stats['average_turnout'] = round(avg_turnout, 2)
    else:
        stats['average_turnout'] = 0
    
    # Elections by club
    stats['elections_by_club'] = Voting_poll.objects.filter(
        poll_type='election',
        session=session.session_id if session else None
    ).values('club__club_name').annotate(
        election_count=Count('poll_id')
    ).order_by('-election_count')[:5]
    
    return stats
```

### 6. Inventory and Asset Management Views

```python
@login_required
def inventory_management(request):
    """
    CORE PURPOSE: Comprehensive inventory and asset management system
    
    BUSINESS LOGIC:
    - Manages all club assets and equipment
    - Provides asset tracking and utilization analytics
    - Handles asset allocation and maintenance scheduling
    - Supports inter-club asset sharing and optimization
    """
    user = request.user
    
    # Get user's accessible clubs
    accessible_clubs = _get_user_accessible_clubs(user)
    
    # Filter assets by club access
    assets = Inventory.objects.filter(club__in=accessible_clubs)
    
    # Apply filters
    club_filter = request.GET.get('club')
    if club_filter:
        assets = assets.filter(club__club_id=club_filter)
    
    category_filter = request.GET.get('category')
    if category_filter:
        assets = assets.filter(asset_category=category_filter)
    
    status_filter = request.GET.get('status')
    if status_filter:
        assets = assets.filter(asset_status=status_filter)
    
    condition_filter = request.GET.get('condition')
    if condition_filter:
        assets = assets.filter(asset_condition=condition_filter)
    
    # Search functionality
    search_query = request.GET.get('search', '')
    if search_query:
        assets = assets.filter(
            Q(asset_name__icontains=search_query) |
            Q(asset_description__icontains=search_query) |
            Q(brand__icontains=search_query) |
            Q(model_number__icontains=search_query)
        )
    
    # Sorting
    sort_by = request.GET.get('sort', 'asset_name')
    assets = assets.order_by(sort_by)
    
    # Pagination
    paginator = Paginator(assets, 20)
    page_number = request.GET.get('page')
    page_assets = paginator.get_page(page_number)
    
    # Add analytics to each asset
    for asset in page_assets:
        asset.analytics = asset.get_comprehensive_asset_analytics()
    
    # Inventory statistics
    inventory_stats = _generate_inventory_statistics(accessible_clubs)
    
    # Maintenance alerts
    maintenance_alerts = _get_maintenance_alerts(accessible_clubs)
    
    # Filter options
    categories = Inventory.objects.filter(
        club__in=accessible_clubs
    ).values_list('asset_category', flat=True).distinct()
    
    statuses = Inventory.objects.filter(
        club__in=accessible_clubs
    ).values_list('asset_status', flat=True).distinct()
    
    conditions = Inventory.objects.filter(
        club__in=accessible_clubs
    ).values_list('asset_condition', flat=True).distinct()
    
    context = {
        'assets': page_assets,
        'accessible_clubs': accessible_clubs,
        'categories': categories,
        'statuses': statuses,
        'conditions': conditions,
        'inventory_stats': inventory_stats,
        'maintenance_alerts': maintenance_alerts,
        'filters': {
            'club': club_filter,
            'category': category_filter,
            'status': status_filter,
            'condition': condition_filter,
            'search': search_query,
            'sort': sort_by
        }
    }
    
    return render(request, 'gymkhana/inventory_management.html', context)

def _generate_inventory_statistics(clubs):
    """Helper function to generate inventory statistics"""
    assets = Inventory.objects.filter(club__in=clubs)
    
    stats = {
        'total_assets': assets.count(),
        'total_value': assets.aggregate(
            total=Sum('current_value')
        )['total'] or 0,
        'available_assets': assets.filter(asset_status='active').count(),
        'assets_under_maintenance': assets.filter(
            asset_status__in=['maintenance', 'repair']
        ).count()
    }
    
    # Asset distribution by category
    stats['category_distribution'] = assets.values('asset_category').annotate(
        count=Count('asset_id'),
        total_value=Sum('current_value')
    ).order_by('-count')
    
    # Asset condition breakdown
    stats['condition_breakdown'] = assets.values('asset_condition').annotate(
        count=Count('asset_id')
    )
    
    # Utilization metrics
    total_assets_with_usage = assets.exclude(total_usage_hours=0).count()
    if total_assets_with_usage > 0:
        avg_utilization = assets.exclude(total_usage_hours=0).aggregate(
            avg_utilization=Avg('utilization_rate')
        )['avg_utilization'] or 0
        stats['average_utilization'] = round(avg_utilization, 2)
    else:
        stats['average_utilization'] = 0
    
    return stats

def _get_maintenance_alerts(clubs):
    """Helper function to get maintenance alerts"""
    alerts = []
    
    # Assets needing immediate maintenance
    overdue_maintenance = Inventory.objects.filter(
        club__in=clubs,
        next_maintenance_due__lt=timezone.now().date()
    )
    
    for asset in overdue_maintenance:
        days_overdue = (timezone.now().date() - asset.next_maintenance_due).days
        alerts.append({
            'type': 'overdue_maintenance',
            'asset': asset,
            'message': f"Maintenance overdue by {days_overdue} days",
            'priority': 'high' if days_overdue > 30 else 'medium'
        })
    
    # Assets approaching maintenance
    upcoming_maintenance = Inventory.objects.filter(
        club__in=clubs,
        next_maintenance_due__range=[
            timezone.now().date(),
            timezone.now().date() + timezone.timedelta(days=14)
        ]
    )
    
    for asset in upcoming_maintenance:
        days_until = (asset.next_maintenance_due - timezone.now().date()).days
        alerts.append({
            'type': 'upcoming_maintenance',
            'asset': asset,
            'message': f"Maintenance due in {days_until} days",
            'priority': 'low'
        })
    
    # Assets with poor condition
    poor_condition_assets = Inventory.objects.filter(
        club__in=clubs,
        asset_condition__in=['poor', 'damaged']
    )
    
    for asset in poor_condition_assets:
        alerts.append({
            'type': 'poor_condition',
            'asset': asset,
            'message': f"Asset in {asset.asset_condition} condition",
            'priority': 'medium'
        })
    
    return sorted(alerts, key=lambda x: {'high': 3, 'medium': 2, 'low': 1}[x['priority']], reverse=True)

@login_required
def asset_detail(request, asset_id):
    """
    CORE PURPOSE: Detailed asset view with management capabilities
    
    BUSINESS LOGIC:
    - Shows comprehensive asset information and analytics
    - Provides asset allocation and booking functionality
    - Handles maintenance scheduling and tracking
    - Supports asset sharing and collaboration
    """
    asset = get_object_or_404(Inventory, asset_id=asset_id)
    user = request.user
    
    # Check permissions
    can_manage = (
        _is_club_admin(user) or
        _is_gymkhana_admin(user) or
        Club_member.objects.filter(
            club=asset.club,
            member=user,
            membership_status='active'
        ).exists()
    )
    
    if not can_manage:
        messages.error(request, 'You do not have permission to view this asset.')
        return redirect('inventory_management')
    
    # Get asset analytics
    asset_analytics = asset.get_comprehensive_asset_analytics()
    
    # Get allocation history
    allocation_history = _get_asset_allocation_history(asset)
    
    # Get maintenance history
    maintenance_history = asset.maintenance_history
    
    # Check current allocations
    current_allocations = _get_current_asset_allocations(asset)
    
    # Booking availability
    booking_calendar = _generate_booking_calendar(asset)
    
    # Similar assets
    similar_assets = Inventory.objects.filter(
        asset_category=asset.asset_category,
        club=asset.club
    ).exclude(asset_id=asset_id)[:3]
    
    context = {
        'asset': asset,
        'can_manage': can_manage,
        'asset_analytics': asset_analytics,
        'allocation_history': allocation_history,
        'maintenance_history': maintenance_history,
        'current_allocations': current_allocations,
        'booking_calendar': booking_calendar,
        'similar_assets': similar_assets
    }
    
    return render(request, 'gymkhana/asset_detail.html', context)

def _get_asset_allocation_history(asset):
    """Helper function to get asset allocation history"""
    # This would get actual allocation records
    # For now, return mock data based on allocation_history field
    history = []
    
    for allocation in asset.allocation_history:
        history.append({
            'allocated_to': allocation.get('allocated_to'),
            'allocation_date': allocation.get('allocation_date'),
            'return_date': allocation.get('return_date'),
            'purpose': allocation.get('purpose'),
            'status': allocation.get('status', 'completed')
        })
    
    return sorted(history, key=lambda x: x['allocation_date'], reverse=True)

def _get_current_asset_allocations(asset):
    """Helper function to get current asset allocations"""
    current = []
    
    for allocation in asset.current_allocations:
        current.append({
            'allocated_to': allocation.get('allocated_to'),
            'allocation_date': allocation.get('allocation_date'),
            'expected_return_date': allocation.get('expected_return_date'),
            'purpose': allocation.get('purpose'),
            'contact_info': allocation.get('contact_info')
        })
    
    return current

def _generate_booking_calendar(asset):
    """Helper function to generate booking calendar"""
    if not asset.is_bookable:
        return None
    
    # Generate next 30 days
    calendar_data = []
    for i in range(30):
        date = timezone.now().date() + timezone.timedelta(days=i)
        
        # Check if date is available (simplified logic)
        is_available = True
        for allocation in asset.current_allocations:
            alloc_start = datetime.datetime.strptime(allocation.get('allocation_date'), '%Y-%m-%d').date()
            alloc_end = datetime.datetime.strptime(allocation.get('expected_return_date'), '%Y-%m-%d').date()
            
            if alloc_start <= date <= alloc_end:
                is_available = False
                break
        
        calendar_data.append({
            'date': date,
            'is_available': is_available,
            'day_name': date.strftime('%a')
        })
    
    return calendar_data
```

### 7. API Endpoints

```python
# API Views for mobile and external integrations
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from rest_framework import status
from django.http import JsonResponse

@api_view(['GET'])
@permission_classes([IsAuthenticated])
def api_club_list(request):
    """
    API endpoint for club listing with filtering
    """
    clubs = Club_info.objects.filter(status='active')
    
    # Apply filters
    category = request.GET.get('category')
    if category:
        clubs = clubs.filter(category=category)
    
    # Serialize data
    club_data = []
    for club in clubs:
        analytics = club.get_comprehensive_club_analytics()
        club_data.append({
            'club_id': club.club_id,
            'club_name': club.club_name,
            'category': club.category,
            'description': club.club_description,
            'total_members': club.get_total_members(),
            'logo_url': club.club_logo.url if club.club_logo else None,
            'performance_score': analytics.get('overall_performance', {}).get('composite_score', 0),
            'events_conducted': analytics.get('event_metrics', {}).get('total_events_conducted', 0)
        })
    
    return Response({
        'status': 'success',
        'count': len(club_data),
        'clubs': club_data
    })

@api_view(['GET'])
@permission_classes([IsAuthenticated])
def api_event_list(request):
    """
    API endpoint for event listing with filtering
    """
    events = Event_info.objects.all()
    
    # Apply filters
    club_id = request.GET.get('club_id')
    if club_id:
        events = events.filter(organizing_club__club_id=club_id)
    
    status_filter = request.GET.get('status')
    if status_filter:
        events = events.filter(event_status=status_filter)
    
    # Date filtering
    upcoming = request.GET.get('upcoming', 'false').lower() == 'true'
    if upcoming:
        events = events.filter(event_date__gte=timezone.now())
    
    events = events.order_by('event_date')[:20]  # Limit to 20 events
    
    # Serialize data
    event_data = []
    for event in events:
        event_data.append({
            'event_id': event.event_id,
            'event_name': event.event_name,
            'event_date': event.event_date.isoformat(),
            'organizing_club': event.organizing_club.club_name,
            'venue': event.venue,
            'registration_fee': float(event.registration_fee),
            'max_participants': event.max_participants,
            'registered_participants': event.registered_participants,
            'status': event.event_status,
            'can_register': event.event_status in ['approved', 'ongoing'] and 
                           event.registered_participants < event.max_participants
        })
    
    return Response({
        'status': 'success',
        'count': len(event_data),
        'events': event_data
    })

@api_view(['POST'])
@permission_classes([IsAuthenticated])
def api_event_register(request, event_id):
    """
    API endpoint for event registration
    """
    try:
        event = Event_info.objects.get(event_id=event_id)
    except Event_info.DoesNotExist:
        return Response({
            'status': 'error',
            'message': 'Event not found'
        }, status=status.HTTP_404_NOT_FOUND)
    
    user = request.user
    
    # Check if already registered
    is_already_registered = any(
        participant.get('user_id') == user.id 
        for participant in event.participants
    )
    
    if is_already_registered:
        return Response({
            'status': 'error',
            'message': 'Already registered for this event'
        }, status=status.HTTP_400_BAD_REQUEST)
    
    # Check eligibility
    eligibility_check = _check_event_eligibility(user, event)
    if not eligibility_check['eligible']:
        return Response({
            'status': 'error',
            'message': eligibility_check['reason']
        }, status=status.HTTP_400_BAD_REQUEST)
    
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
    
    return Response({
        'status': 'success',
        'message': 'Successfully registered for the event!',
        'requires_payment': event.registration_fee > 0,
        'registration_fee': float(event.registration_fee)
    })

@api_view(['GET'])
@permission_classes([IsAuthenticated])
def api_user_dashboard(request):
    """
    API endpoint for user dashboard data
    """
    user = request.user
    current_session = Session_info.get_current_session()
    
    # User's clubs
    user_clubs = Club_member.objects.filter(
        member=user,
        session=current_session.session_id if current_session else None,
        membership_status='active'
    ).select_related('club')
    
    clubs_data = []
    for membership in user_clubs:
        club = membership.club
        clubs_data.append({
            'club_id': club.club_id,
            'club_name': club.club_name,
            'category': club.category,
            'member_type': membership.member_type,
            'joined_date': membership.joined_date.isoformat()
        })
    
    # User's upcoming events
    user_event_ids = []
    for membership in user_clubs:
        club_events = Event_info.objects.filter(
            organizing_club=membership.club,
            event_date__gte=timezone.now()
        )[:3]
        for event in club_events:
            user_event_ids.append(event.event_id)
    
    upcoming_events = Event_info.objects.filter(
        event_id__in=user_event_ids
    ).order_by('event_date')[:5]
    
    events_data = []
    for event in upcoming_events:
        events_data.append({
            'event_id': event.event_id,
            'event_name': event.event_name,
            'event_date': event.event_date.isoformat(),
            'organizing_club': event.organizing_club.club_name,
            'venue': event.venue
        })
    
    # User statistics
    stats = {
        'clubs_joined': len(clubs_data),
        'events_participated': Event_info.objects.filter(
            participants__contains=[{'user_id': user.id}]
        ).count(),
        'leadership_positions': Core_team.objects.filter(member=user).count()
    }
    
    return Response({
        'status': 'success',
        'user_clubs': clubs_data,
        'upcoming_events': events_data,
        'statistics': stats
    })

@api_view(['GET'])
@permission_classes([IsAuthenticated])
def api_club_analytics(request, club_id):
    """
    API endpoint for club analytics data
    """
    try:
        club = Club_info.objects.get(club_id=club_id)
    except Club_info.DoesNotExist:
        return Response({
            'status': 'error',
            'message': 'Club not found'
        }, status=status.HTTP_404_NOT_FOUND)
    
    # Check permissions
    user = request.user
    can_view = (
        _is_club_admin(user) or
        _is_gymkhana_admin(user) or
        Club_member.objects.filter(
            club=club,
            member=user,
            membership_status='active'
        ).exists()
    )
    
    if not can_view:
        return Response({
            'status': 'error',
            'message': 'Permission denied'
        }, status=status.HTTP_403_FORBIDDEN)
    
    # Get comprehensive analytics
    analytics = club.get_comprehensive_club_analytics()
    
    return Response({
        'status': 'success',
        'club_id': club_id,
        'club_name': club.club_name,
        'analytics': analytics
    })

# WebSocket support for real-time notifications
@csrf_exempt
def websocket_notifications(request):
    """
    WebSocket endpoint for real-time notifications
    """
    # This would implement WebSocket functionality
    # For now, return a placeholder response
    return JsonResponse({
        'message': 'WebSocket endpoint for real-time notifications',
        'status': 'not_implemented'
    })
```

This comprehensive Part 15 covers advanced view functions, API endpoints, and system integrations.