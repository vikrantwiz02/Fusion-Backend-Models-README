# Estate Module System - Complete Documentation

## Overview
The Estate Module is a comprehensive infrastructure and asset management system that handles building lifecycle management, construction/maintenance work tracking, and inventory control for an educational institution. This system manages the complete lifecycle of infrastructure from initial project issuing through construction, completion, and operational phases.

## Database Models

### 1. Building Model
**PostgreSQL Table: `estate_module_building`**

#### Fields
| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| `id` | AutoField | Primary key identifier | NOT NULL, UNIQUE |
| `name` | CharField(max_length=100) | Building name | NOT NULL |
| `dateIssued` | DateField | Date when building project was issued | NULL, BLANK |
| `dateConstructionStarted` | DateField | Date construction began | NULL, BLANK |
| `dateConstructionCompleted` | DateField | Date construction finished | NULL, BLANK |
| `dateOperational` | DateField | Date building became operational | NULL, BLANK |
| `area` | IntegerField | Building area in square units | NULL, BLANK |
| `constructionCostEstimated` | IntegerField | Estimated construction cost | NULL, BLANK |
| `constructionCostActual` | IntegerField | Actual construction cost | NULL, BLANK |
| `numRooms` | IntegerField | Number of rooms in building | NULL, BLANK |
| `numWashrooms` | IntegerField | Number of washrooms | NULL, BLANK |
| `remarks` | TextField | Additional notes and comments | NULL, BLANK |
| `verified` | BooleanField | Verification status | DEFAULT False |

#### Status Constants
```python
STATUS_CHOICES = (
    ('On Schedule', 'On Schedule'),
    ('Delayed', 'Delayed'),
)
```

#### Business Methods
- `status()`: Determines if building is on schedule or delayed based on current date vs expected completion
- `works()`: Returns QuerySet of all Work objects associated with this building
- `cost()`: Returns actual cost if available, otherwise estimated cost

#### Core Logic
**HOW IT WORKS:**
1. **Project Lifecycle Management**: Buildings progress through distinct phases - Issue → Construction Start → Construction Complete → Operational
2. **Date Validation**: System enforces chronological order of dates (Issue < Start < Complete < Operational)
3. **Status Calculation**: Automatically determines if project is on schedule by comparing current date with expected completion
4. **Cost Tracking**: Maintains both estimated and actual costs, with fallback logic for reporting

**BUSINESS PURPOSE:**
The Building model serves as the foundation for infrastructure lifecycle management, enabling administrators to track construction projects from initial approval through operational deployment. It provides critical data for budget management, project monitoring, and facility planning.

### 2. Work Model
**PostgreSQL Table: `estate_module_work`**

#### Fields
| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| `id` | AutoField | Primary key identifier | NOT NULL, UNIQUE |
| `name` | CharField(max_length=100) | Work/project name | NOT NULL |
| `workType` | CharField(max_length=25) | Type of work being performed | NOT NULL, CHOICES |
| `building` | ForeignKey(Building) | Associated building | NULL, BLANK, CASCADE |
| `contractorName` | CharField(max_length=100) | Name of contractor | NOT NULL |
| `dateIssued` | DateField | Date work was issued | NULL, BLANK |
| `dateStarted` | DateField | Date work began | NULL, BLANK |
| `dateCompleted` | DateField | Date work was completed | NULL, BLANK |
| `costEstimated` | IntegerField | Estimated work cost | NULL, BLANK |
| `costActual` | IntegerField | Actual work cost | NULL, BLANK |
| `remarks` | TextField | Additional notes | NULL, BLANK |

#### Work Type Constants
```python
WORK_TYPE_CHOICES = (
    ('Construction Work', 'Construction Work'),
    ('Maintenance Work', 'Maintenance Work'),
)
STATUS_CHOICES = (
    ('On Schedule', 'On Schedule'),  
    ('Delayed', 'Delayed'),
)
```

#### Business Methods
- `status()`: Calculates schedule status based on current date vs completion timeline
- `subWorks()`: Returns all SubWork objects under this work
- `cost()`: Returns actual cost if available, otherwise estimated cost

#### Core Logic
**HOW IT WORKS:**
1. **Work Classification**: Distinguishes between construction (new building work) and maintenance (upkeep of existing structures)
2. **Contractor Management**: Tracks contractor assignments for accountability and payment processing
3. **Progress Monitoring**: Maintains timeline from issue through completion with status calculation
4. **Cost Control**: Dual-tracking of estimated vs actual costs for budget variance analysis

**BUSINESS PURPOSE:**
The Work model manages specific construction and maintenance projects, providing granular tracking of institutional infrastructure development. It enables contract management, progress monitoring, and financial oversight for all building-related activities.

### 3. SubWork Model
**PostgreSQL Table: `estate_module_subwork`**

#### Fields
| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| `id` | AutoField | Primary key identifier | NOT NULL, UNIQUE |
| `name` | CharField(max_length=100) | SubWork name | NOT NULL |
| `work` | ForeignKey(Work) | Parent work project | NOT NULL, CASCADE |
| `dateIssued` | DateField | Date subwork was issued | NULL, BLANK |
| `dateStarted` | DateField | Date subwork began | NULL, BLANK |
| `dateCompleted` | DateField | Date subwork was completed | NULL, BLANK |
| `costEstimated` | IntegerField | Estimated subwork cost | NULL, BLANK |
| `costActual` | IntegerField | Actual subwork cost | NULL, BLANK |
| `remarks` | TextField | Additional notes | NULL, BLANK |

#### Business Methods
- `status()`: Determines if subwork is on schedule or delayed
- `cost()`: Returns actual cost if available, otherwise estimated cost

#### Core Logic
**HOW IT WORKS:**
1. **Task Decomposition**: Breaks down major work projects into manageable sub-components
2. **Hierarchical Tracking**: Maintains parent-child relationship with main Work projects
3. **Granular Progress**: Enables detailed monitoring of work completion at sub-task level
4. **Resource Allocation**: Tracks costs and timelines for individual work components

**BUSINESS PURPOSE:**
SubWork enables detailed project management by decomposing complex construction or maintenance projects into specific tasks. This granular approach improves progress tracking, resource allocation, and helps identify bottlenecks in project execution.

### 4. InventoryType Model
**PostgreSQL Table: `estate_module_inventorytype`**

#### Fields
| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| `id` | AutoField | Primary key identifier | NOT NULL, UNIQUE |
| `name` | CharField(max_length=100) | Inventory item name | NOT NULL |
| `manufacturer` | CharField(max_length=100) | Manufacturer name | NULL, BLANK |
| `model` | CharField(max_length=100) | Model name/number | NULL, BLANK |
| `rate` | IntegerField | Cost per unit | NULL, BLANK |
| `remarks` | TextField | Additional specifications | NULL, BLANK |

#### Business Methods
- `__str__()`: Returns formatted string with name, manufacturer, and model

#### Core Logic
**HOW IT WORKS:**
1. **Standardization**: Creates standardized inventory categories with consistent specifications
2. **Cost Management**: Maintains unit rates for budget planning and procurement
3. **Vendor Tracking**: Associates items with manufacturers for supplier management
4. **Specification Control**: Ensures consistent procurement through model specifications

**BUSINESS PURPOSE:**
InventoryType serves as a master catalog for all institutional assets and consumables, enabling standardized procurement, accurate cost estimation, and consistent inventory management across all projects and buildings.

### 5. InventoryCommon Model (Abstract Base)
**PostgreSQL Table: Abstract - No direct table created**

#### Fields
| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| `inventoryType` | ForeignKey(InventoryType) | Type of inventory item | NOT NULL, CASCADE |
| `building` | ForeignKey(Building) | Associated building | NULL, BLANK, CASCADE |
| `work` | ForeignKey(Work) | Associated work project | NULL, BLANK, CASCADE |
| `quantity` | IntegerField | Total quantity | NOT NULL |
| `dateOrdered` | DateField | Date item was ordered | NULL, BLANK |
| `dateReceived` | DateField | Date item was received | NULL, BLANK |
| `remarks` | TextField | Additional notes | NULL, BLANK |

#### Business Methods
- `__str__()`: Returns inventory type name for display
- `cost()`: Calculates total cost (quantity × rate)

#### Core Logic
**HOW IT WORKS:**
1. **Common Framework**: Provides shared functionality for both consumable and non-consumable inventory
2. **Project Association**: Links inventory items to specific buildings and work projects
3. **Procurement Tracking**: Monitors ordering and receiving dates for supply chain management
4. **Cost Calculation**: Automatically computes total costs based on quantity and unit rates

**BUSINESS PURPOSE:**
InventoryCommon establishes the foundational framework for all inventory management, ensuring consistent tracking methodologies and cost calculations across different inventory types while maintaining project-specific associations.

### 6. InventoryConsumable Model
**PostgreSQL Table: `estate_module_inventoryconsumable`**

#### Fields (Inherits from InventoryCommon plus)
| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| `presentQuantity` | IntegerField | Current available quantity | NOT NULL |

#### Core Logic
**HOW IT WORKS:**
1. **Usage Tracking**: Monitors consumption by tracking present quantity vs original quantity
2. **Depletion Management**: Enables identification of items needing replenishment
3. **Consumption Analysis**: Supports usage pattern analysis for better procurement planning
4. **Stock Control**: Maintains real-time inventory levels for operational planning

**BUSINESS PURPOSE:**
InventoryConsumable manages items that are used up during construction or maintenance activities (like cement, paint, electrical components). It enables consumption tracking, automatic reorder point identification, and ensures adequate supplies for ongoing projects.

### 7. InventoryNonConsumable Model
**PostgreSQL Table: `estate_module_inventorynonconsumable`**

#### Fields (Inherits from InventoryCommon plus)
| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| `serial_no` | CharField(max_length=20) | Unique serial number | NOT NULL |
| `dateLastVerified` | DateField | Last verification date | NOT NULL |
| `issued_to` | ForeignKey(User) | User who has the item | NULL, BLANK, SET_NULL |

#### Core Logic
**HOW IT WORKS:**
1. **Asset Tracking**: Uses serial numbers for unique identification of individual items
2. **Assignment Management**: Tracks which user currently possesses each item
3. **Verification Scheduling**: Maintains verification dates for audit compliance
4. **Accountability**: Ensures responsibility assignment for valuable assets

**BUSINESS PURPOSE:**
InventoryNonConsumable manages durable assets like tools, equipment, and furniture that retain value over time. It provides accountability through user assignment, supports audit requirements through verification tracking, and enables asset lifecycle management.

## View Functions

### 1. estate() - Main Dashboard
**URL: `/estate/`**

#### Core Logic
**HOW IT WORKS:**
1. **Authentication Check**: Verifies user login and restricts student access
2. **Data Aggregation**: Collects all buildings, works, and inventory data
3. **Tab Organization**: Groups data into logical categories (All, Complete, Incomplete, etc.)
4. **Form Integration**: Prepares forms for each entity type
5. **Context Assembly**: Combines all data into structured context for template rendering

**BUSINESS PURPOSE:**
Provides a comprehensive dashboard for estate managers to view all infrastructure and inventory data in organized tabs, enabling quick assessment of project status, inventory levels, and overall estate management.

### 2. Building Management Views

#### newBuilding()
**URL: `/estate/new/building`**
- **Method**: POST only
- **Function**: Creates new building records with form validation
- **Validation**: Enforces date chronology and required fields
- **Success**: Redirects to dashboard with success message
- **Error**: Returns to dashboard with validation errors

#### editBuilding()
**URL: `/estate/edit/building/<building_id>`**
- **Method**: POST only  
- **Function**: Updates existing building records
- **Validation**: Same as creation plus existence check
- **Success**: Updates record and redirects with confirmation

#### deleteBuilding()
**URL: `/estate/delete/building/<building_id>`**
- **Method**: POST only
- **Function**: Removes building record from database
- **Safety**: Confirms building exists before deletion
- **Cleanup**: Handles cascading deletions for related records

### 3. Work Management Views

#### newWork(), editWork(), deleteWork()
**URLs**: `/estate/new/work`, `/estate/edit/work/<work_id>`, `/estate/delete/work/<work_id>`
- **Function**: Complete CRUD operations for Work records
- **Type Handling**: Distinguishes between Construction and Maintenance work
- **Contractor Tracking**: Maintains contractor assignments
- **Cost Management**: Handles estimated vs actual cost tracking

### 4. SubWork Management Views

#### newSubWork(), editSubWork(), deleteSubWork()
**URLs**: `/estate/new/subWork`, `/estate/edit/subWork/<subWork_id>`, `/estate/delete/subWork/<subWork_id>`
- **Function**: Manages granular work breakdown
- **Parent Relationship**: Maintains connection to parent Work
- **Progress Tracking**: Enables detailed project monitoring

### 5. Inventory Management Views

#### Inventory Type Management
- **newInventoryType()**, **editInventoryType()**, **deleteInventoryType()**
- **Function**: Manages master inventory catalog
- **Standardization**: Ensures consistent item specifications

#### Consumable Inventory Management  
- **newInventoryConsumable()**, **editInventoryConsumable()**, **deleteInventoryConsumable()**
- **Function**: Tracks usage-based inventory
- **Consumption Monitoring**: Updates present quantities

#### Non-Consumable Inventory Management
- **newInventoryNonConsumable()**, **editInventoryNonConsumable()**, **deleteInventoryNonConsumable()**
- **Function**: Manages durable assets
- **Assignment Tracking**: Maintains user responsibility

## Forms System

### 1. BuildingForm
**Model**: Building
**Features**:
- Date validation with chronological enforcement
- Cost input with text type for UI compatibility
- Semantic UI integration with placeholder text
- Custom clean() method for date sequence validation

**Validation Logic**:
```python
# Ensures: dateIssued < dateConstructionStarted < dateConstructionCompleted < dateOperational
```

### 2. WorkForm
**Model**: Work
**Features**:
- Work type selection (Construction/Maintenance)
- Building association dropdown
- Contractor name input
- Date and cost tracking

### 3. SubWorkForm
**Model**: SubWork
**Features**:
- Parent work selection
- Independent timeline tracking
- Cost breakdown capability

### 4. InventoryTypeForm
**Model**: InventoryType
**Features**:
- Manufacturer and model specification
- Rate setting for cost calculations

### 5. InventoryConsumableForm & InventoryNonConsumableForm
**Models**: InventoryConsumable, InventoryNonConsumable
**Features**:
- Type selection from InventoryType catalog
- Project association (building/work)
- Quantity and date tracking
- Serial number management (non-consumable only)

## URL Patterns

### Structure
```python
# Main dashboard
path('', views.estate, name="estate_module_home")

# Creation endpoints
path('new/<entity>', views.new<Entity>, name="new_<entity>")

# Edit endpoints  
path('edit/<entity>/<entity_id>', views.edit<Entity>, name="edit_<entity>")

# Delete endpoints
path('delete/<entity>/<entity_id>', views.delete<Entity>, name="delete_<entity>")
```

### Complete URL Mapping
- **Buildings**: 4 URLs (view, new, edit, delete)
- **Works**: 4 URLs (view, new, edit, delete)  
- **SubWorks**: 4 URLs (view, new, edit, delete)
- **InventoryTypes**: 4 URLs (view, new, edit, delete)
- **Consumable Inventory**: 4 URLs (view, new, edit, delete)
- **Non-Consumable Inventory**: 4 URLs (view, new, edit, delete)

## Admin Integration

### Registered Models
```python
admin.site.register(Building)
admin.site.register(Work) 
admin.site.register(SubWork)
admin.site.register(InventoryType)
admin.site.register(InventoryConsumable)
admin.site.register(InventoryNonConsumable)
```

### Administrative Features
- **Full CRUD Access**: Complete create, read, update, delete capabilities
- **Search and Filter**: Built-in Django admin search functionality
- **Bulk Operations**: Mass updates and deletions
- **Data Export**: CSV/JSON export capabilities
- **Audit Trail**: Change history tracking

## Database Relationships

### Foreign Key Relationships
1. **Work → Building**: Many works can be associated with one building
2. **SubWork → Work**: Many subworks belong to one parent work
3. **InventoryCommon → InventoryType**: Many inventory items of one type
4. **InventoryCommon → Building**: Inventory can be associated with buildings
5. **InventoryCommon → Work**: Inventory can be associated with specific work
6. **InventoryNonConsumable → User**: Tracks item assignment to users

### Cascade Behaviors
- **Building deletion**: Cascades to associated Works and Inventory
- **Work deletion**: Cascades to SubWorks and associated Inventory  
- **InventoryType deletion**: Cascades to all Inventory items of that type
- **User deletion**: Sets InventoryNonConsumable.issued_to to NULL

## Business Logic Integration

### Status Calculation Algorithm
```python
def status(self):
    if not self.dateConstructionCompleted and self.dateConstructionStarted:
        expected_completion = calculate_expected_date()
        if timezone.now().date() > expected_completion:
            return 'Delayed'
    return 'On Schedule'
```

### Cost Calculation Logic
```python
def cost(self):
    return self.costActual if self.costActual else self.costEstimated

def total_inventory_cost(self):
    return self.quantity * self.inventoryType.rate if self.inventoryType.rate else 0
```

### Data Validation Framework
1. **Date Sequences**: Enforces logical date progression
2. **Cost Consistency**: Validates estimated vs actual cost relationships
3. **Quantity Logic**: Ensures present quantity ≤ original quantity for consumables
4. **Association Integrity**: Validates building-work-inventory relationships

## System Integration Points

### Dependencies
- **globals.models**: User, ExtraInfo, HoldsDesignation for authentication
- **Django Auth**: User authentication and authorization
- **Django Messages**: User feedback system
- **Django Forms**: Data validation and user input

### External Connections
- **Financial Systems**: Cost data feeds to accounting modules
- **Procurement Systems**: Inventory data for purchase requisitions
- **Maintenance Scheduling**: Work data for preventive maintenance
- **Asset Management**: Non-consumable inventory for institutional assets

## Core System Functions

### Project Lifecycle Management
1. **Initiation**: Building projects start with dateIssued
2. **Planning**: Cost estimation and contractor assignment
3. **Execution**: Construction tracking with progress monitoring
4. **Completion**: Final cost reconciliation and operational handover
5. **Maintenance**: Ongoing work management through Work model

### Inventory Control System
1. **Cataloging**: Standardized inventory types with specifications
2. **Procurement**: Order tracking from requisition to receipt
3. **Distribution**: Assignment tracking for non-consumable items
4. **Consumption**: Usage monitoring for consumable items
5. **Verification**: Periodic audits for accountability

### Financial Management
1. **Budgeting**: Estimated cost tracking for planning
2. **Monitoring**: Actual cost capture for variance analysis
3. **Reporting**: Cost summaries for management decisions
4. **Control**: Approval workflows for cost overruns

### Database Migration Schema
**Migration File: `0001_initial.py`**
**Created: July 16, 2024 (Django 3.1.5)**

#### Table Creation Order & Dependencies
```python
# Dependencies: AUTH_USER_MODEL only
# Creation Order:
1. Building (no foreign keys)
2. InventoryType (no foreign keys) 
3. Work (FK to Building)
4. SubWork (FK to Work)
5. InventoryNonConsumable (FK to InventoryType, Building, Work, User)
6. InventoryConsumable (FK to InventoryType, Building, Work)
```

#### Exact PostgreSQL Field Specifications
```sql
-- estate_module_building
CREATE TABLE "estate_module_building" (
    "id" SERIAL PRIMARY KEY,
    "name" VARCHAR(100) NOT NULL,
    "dateIssued" DATE NOT NULL,
    "dateConstructionStarted" DATE NULL,
    "dateConstructionCompleted" DATE NULL, 
    "dateOperational" DATE NULL,
    "status" VARCHAR(2) DEFAULT 'OS' CHECK (status IN ('OS', 'DL')),
    "area" INTEGER NULL,
    "constructionCostEstimated" INTEGER NULL,
    "constructionCostActual" INTEGER NULL,
    "numRooms" INTEGER NULL,
    "numWashrooms" INTEGER NULL,
    "remarks" TEXT NULL,
    "verified" BOOLEAN DEFAULT FALSE
);
CREATE INDEX ON "estate_module_building" ("id" DESC);

-- estate_module_work  
CREATE TABLE "estate_module_work" (
    "id" SERIAL PRIMARY KEY,
    "name" VARCHAR(100) NOT NULL,
    "workType" VARCHAR(2) DEFAULT 'MW' CHECK (workType IN ('CW', 'MW')),
    "contractorName" VARCHAR(100) NOT NULL,
    "status" VARCHAR(2) DEFAULT 'OS' CHECK (status IN ('OS', 'DL')),
    "dateIssued" DATE NOT NULL,
    "dateStarted" DATE NULL,
    "dateCompleted" DATE NULL,
    "costEstimated" INTEGER NULL,
    "costActual" INTEGER NULL,
    "remarks" TEXT NULL,
    "verified" BOOLEAN DEFAULT FALSE,
    "building_id" INTEGER NULL REFERENCES "estate_module_building"("id") ON DELETE CASCADE
);
CREATE INDEX ON "estate_module_work" ("id" DESC);
CREATE INDEX ON "estate_module_work" ("building_id");

-- estate_module_subwork
CREATE TABLE "estate_module_subwork" (
    "id" SERIAL PRIMARY KEY,
    "name" VARCHAR(100) NOT NULL,
    "dateIssued" DATE NOT NULL,
    "dateStarted" DATE NULL,
    "dateCompleted" DATE NULL,
    "costEstimated" INTEGER NULL,
    "costActual" INTEGER NULL,
    "remarks" TEXT NULL,
    "work_id" INTEGER NOT NULL REFERENCES "estate_module_work"("id") ON DELETE CASCADE
);
CREATE INDEX ON "estate_module_subwork" ("id" DESC);
CREATE INDEX ON "estate_module_subwork" ("work_id");

-- estate_module_inventorytype
CREATE TABLE "estate_module_inventorytype" (
    "id" SERIAL PRIMARY KEY,
    "name" VARCHAR(100) NOT NULL,
    "rate" INTEGER NOT NULL,
    "manufacturer" VARCHAR(100) NULL,
    "model" VARCHAR(100) NULL,
    "remarks" TEXT NULL
);

-- estate_module_inventoryconsumable
CREATE TABLE "estate_module_inventoryconsumable" (
    "id" SERIAL PRIMARY KEY,
    "quantity" INTEGER NOT NULL,
    "dateOrdered" DATE NOT NULL,
    "dateReceived" DATE NULL,
    "remarks" TEXT NULL,
    "presentQuantity" INTEGER NOT NULL,
    "building_id" INTEGER NULL REFERENCES "estate_module_building"("id") ON DELETE CASCADE,
    "inventoryType_id" INTEGER NOT NULL REFERENCES "estate_module_inventorytype"("id") ON DELETE CASCADE,
    "work_id" INTEGER NULL REFERENCES "estate_module_work"("id") ON DELETE CASCADE
);
CREATE INDEX ON "estate_module_inventoryconsumable" ("id" DESC);

-- estate_module_inventorynonconsumable  
CREATE TABLE "estate_module_inventorynonconsumable" (
    "id" SERIAL PRIMARY KEY,
    "quantity" INTEGER NOT NULL,
    "dateOrdered" DATE NOT NULL,
    "dateReceived" DATE NULL,
    "remarks" TEXT NULL,
    "serial_no" VARCHAR(20) NOT NULL,
    "dateLastVerified" DATE NOT NULL,
    "building_id" INTEGER NULL REFERENCES "estate_module_building"("id") ON DELETE CASCADE,
    "inventoryType_id" INTEGER NOT NULL REFERENCES "estate_module_inventorytype"("id") ON DELETE CASCADE,
    "issued_to_id" INTEGER NULL REFERENCES "auth_user"("id") ON DELETE SET NULL,
    "work_id" INTEGER NULL REFERENCES "estate_module_work"("id") ON DELETE CASCADE
);
CREATE INDEX ON "estate_module_inventorynonconsumable" ("id" DESC);
```

### Complete Template System Architecture

#### Template Directory Structure
```
templates/estate_module/
├── home.html (Main dashboard)
├── Building/
│   ├── home.html
│   ├── tab.html  
│   ├── form.html
│   ├── new_modal.html
│   ├── edit_modal.html
│   ├── edit_button.html
│   └── detail_modal.html
├── Work/
│   ├── home.html
│   ├── tab.html
│   ├── form.html
│   ├── new.html
│   ├── edit.html
│   ├── edit_button.html
│   └── detail_modal.html
├── SubWork/
│   ├── form.html
│   ├── new_modal.html
│   ├── edit_modal.html
│   ├── edit_button.html
│   └── detail_modal.html
├── InventoryType/
│   ├── tab.html
│   ├── detail.html
│   ├── edit_modal.html
│   └── edit_button.html
└── Inventory/
    ├── Consumable/
    │   ├── form.html
    │   ├── new_modal.html
    │   ├── edit_modal.html
    │   ├── edit_button.html
    │   └── detail.html
    └── NonConsumable/
        ├── form.html
        ├── new_modal.html
        ├── edit_modal.html
        ├── edit_button.html
        └── detail.html
```

#### Frontend JavaScript Integration

##### Main Dashboard Template Logic
```html
<!-- home.html structure -->
{% extends 'globals/base.html' %}
{% load static %}

<!-- CSS/JS Dependencies -->
<link rel="stylesheet" href="{% static 'estate_module/semantic.min.css' %}" />
<script defer src="{% static 'estate_module/jquery.min.js' %}"></script>
<script defer src="{% static 'estate_module/semantic.min.js' %}"></script>
<script defer src="{% static 'globals/js/tab.js' %}"></script>
<script defer src="{% static 'globals/js/tablesort.js' %}"></script>
<script defer src="{% static 'estate_module/sidebar.js' %}"></script>

<!-- Dynamic Menu Generation -->
{% for menuitem, menuitem_data in menuitems.items %}
<a class="{% if forloop.counter0 is 0 %}active{% endif %} item" data-tab="{{ menuitem|slugify }}">
    {{ menuitem }}
    <i class="right floated chevron right icon"></i>
</a>
{% endfor %}

<!-- Toast Messages System -->
{% for message in messages %}
<script type='module'>
    $('body').toast({
        position: 'top left',
        title: '{{ message.tags|title }}!',
        message: '{{ message }}',
        class: '{{ message.tags }}',
        showProgress: 'bottom',
        displayTime: 6000,
    });
</script>
{% endfor %}
```

##### Form Validation JavaScript Framework
```javascript
// Building Form Validation
$('#building_form').form({
    fields: {
        name: {
            identifier: 'name',
            rules: [{ type: 'empty', prompt: 'Building name cannot be blank' }]
        },
        dateIssued: {
            identifier: 'dateIssued', 
            rules: [{ type: 'empty', prompt: 'Date Issued cannot be blank' }]
        },
        area: {
            identifier: 'area',
            optional: true,
            rules: [{ type: 'integer', prompt: 'Area must be an integer' }]
        },
        constructionCostEstimated: {
            identifier: 'constructionCostEstimated',
            optional: true, 
            rules: [{ type: 'integer', prompt: 'Estimated Cost must be an integer' }]
        }
    }
});

// Work Form Validation
$('#work_form').form({
    fields: {
        name: { 
            identifier: 'name',
            rules: [{ type: 'empty', prompt: 'Work name cannot be blank' }]
        },
        workType: {
            identifier: 'workType',
            rules: [{ type: 'empty', prompt: 'Work Type cannot be blank' }]
        },
        contractorName: {
            identifier: 'contractorName', 
            rules: [{ type: 'empty', prompt: 'Contractor name cannot be blank' }]
        },
        dateIssued: {
            identifier: 'dateIssued',
            rules: [{ type: 'empty', prompt: 'Date Issued cannot be blank' }]
        }
    }
});

// SubWork Form Validation 
$('#subwork_form').form({
    fields: {
        name: {
            identifier: 'name',
            rules: [{ type: 'empty', prompt: 'Sub-Work name cannot be blank' }]
        },
        work: {
            identifier: 'work',
            rules: [{ type: 'empty', prompt: 'Work cannot be blank' }] 
        },
        dateIssued: {
            identifier: 'dateIssued',
            rules: [{ type: 'empty', prompt: 'Date Issued cannot be blank' }]
        },
        costEstimated: {
            identifier: 'costEstimated',
            optional: true,
            rules: [{ type: 'integer', prompt: 'Estimated Cost must be an integer' }]
        }
    }
});

// Inventory Consumable Form Validation
$('#consumable_form').form({
    fields: {
        inventoryType: {
            identifier: 'inventoryType',
            rules: [{ type: 'empty', prompt: 'Inventory name cannot be blank' }]
        },
        dateOrdered: {
            identifier: 'dateOrdered', 
            rules: [{ type: 'empty', prompt: 'Date Ordered cannot be blank' }]
        },
        quantity: {
            identifier: 'quantity',
            rules: [
                { type: 'empty', prompt: 'Quantity cannot be blank' },
                { type: 'integer', prompt: 'Quantity must be an integer' }
            ]
        },
        presentQuantity: {
            identifier: 'presentQuantity',
            rules: [
                { type: 'empty', prompt: 'Present Quantity cannot be blank' },
                { type: 'integer', prompt: 'Present Quantity must be an integer' }
            ]
        }
    }
});
```

##### Modal System Implementation
```javascript
// Building Edit Modal
$('.edit_building.button').on('click', function() {
    $('.edit_building.modal')
        .modal({ centered: false })
        .modal({
            onShow: function() {
                // Pre-fill form with building data
                $('.edit_building.modal form').form('set values', {
                    name: '{{ building.name }}',
                    dateIssued: '{{ building.dateIssued }}',
                    dateConstructionStarted: '{{ building.dateConstructionStarted }}',
                    dateConstructionCompleted: '{{ building.dateConstructionCompleted }}',
                    area: '{{ building.area }}',
                    constructionCostEstimated: '{{ building.constructionCostEstimated }}',
                    verified: '{{ building.verified }}'
                });
            }
        })
        .modal('setting', 'closable', false)
        .modal('show');
});

// Work Detail Modal with Tabs
$('.details.work_{{ work.id }}.button').on('click', function() {
    $('.details.work_{{ work.id }}.modal')
        .modal({ centered: false })
        .modal('show');
});
```

### Complete Static Assets Structure

#### CSS Framework
```
static/estate_module/
├── semantic.css (Full Semantic UI)
├── semantic.min.css (Minified)
├── jquery.min.js (jQuery 3.x)
├── semantic.js (Semantic UI JS)
├── semantic.min.js (Minified)
├── sidebar.js (Custom sidebar functionality)
├── components/ (Semantic UI Components)
└── themes/ (Semantic UI Themes)
```

#### Custom JavaScript Functionality
```javascript
// sidebar.js - Custom Estate Module Navigation
$(document).ready(function() {
    $('.ui.sidebar').sidebar({
        context: $('.pusher'),
        transition: 'push'
    });
    
    $('.menu .item').tab();
    
    // Calendar initialization for date fields
    $('.ui.calendar').calendar({
        type: 'date',
        formatter: {
            date: function(date, settings) {
                return date.toLocaleDateString('en-CA'); // YYYY-MM-DD format
            }
        }
    });
});
```

### Complete Form Widget System

#### Advanced Widget Configuration
```python
# BuildingForm Widgets with Semantic UI Integration
widgets = {
    'name': TextInput(attrs={
        'placeholder': 'Enter name of building',
        'class': 'ui input'
    }),
    'dateIssued': DateInput(attrs={
        'placeholder': 'Enter date',
        'class': 'ui calendar input'
    }),
    'area': NumberInput(attrs={
        'placeholder': 'Enter area',
        'type': 'text',  # Semantic UI compatibility
        'class': 'ui input'
    }),
    'constructionCostEstimated': NumberInput(attrs={
        'placeholder': 'Enter estimated construction cost',
        'type': 'text',  # Prevents HTML5 number validation conflicts
        'class': 'ui input'
    }),
    'verified': CheckboxInput(attrs={
        'class': 'ui checkbox'
    }),
    'remarks': Textarea(attrs={
        'placeholder': 'Enter remarks',
        'rows': 3,
        'class': 'ui textarea'
    })
}

# Form Labels with Semantic UI Icons
labels = {
    'name': 'Building name',
    'dateIssued': 'Date Issued',
    'dateConstructionStarted': 'Construction Started', 
    'dateConstructionCompleted': 'Construction Completed',
    'dateOperational': 'Date Operational',
    'area': 'Area (sq. ft.)',
    'constructionCostEstimated': 'Estimated Construction Cost (₹)',
    'constructionCostActual': 'Actual Construction Cost (₹)',
    'numRooms': 'Number of Rooms',
    'numWashrooms': 'Number of Washrooms',
    'remarks': 'Remarks',
    'verified': 'Verified'
}
```

#### Custom Form Validation Logic
```python
def clean(self):
    super(BuildingForm, self).clean()
    
    dateIssued = self.cleaned_data.get('dateIssued')
    dateConstructionStarted = self.cleaned_data.get('dateConstructionStarted')
    dateConstructionCompleted = self.cleaned_data.get('dateConstructionCompleted')
    dateOperational = self.cleaned_data.get('dateOperational')
    
    # Date sequence validation
    if dateIssued and dateConstructionStarted and dateIssued > dateConstructionStarted:
        self._errors['dateConstructionStarted'] = self.error_class([
            'Construction date must be after issue date'
        ])
    
    if dateConstructionCompleted and dateConstructionStarted and dateConstructionStarted >= dateConstructionCompleted:
        self._errors['dateConstructionCompleted'] = self.error_class([
            'Construction completion date must be after start date'
        ])
    
    if dateConstructionCompleted and dateOperational and dateConstructionCompleted >= dateOperational:
        self._errors['dateOperational'] = self.error_class([
            'Operational date must be after completion date'
        ])
    
    return self.cleaned_data
```

### App Configuration Details

#### Django App Configuration
```python
# apps.py
from django.apps import AppConfig

class EstateModuleConfig(AppConfig):
    name = 'applications.estate_module'
    default_auto_field = 'django.db.models.BigAutoField'
    verbose_name = 'Estate Management System'
    
    def ready(self):
        # Register signals if any
        import applications.estate_module.signals
```

### Authentication & Authorization Framework

#### User Access Control
```python
# View-level security implementation
@login_required(login_url='/accounts/login/')
def estate(request):
    current_user = get_object_or_404(User, username=request.user.username)
    extraInfo = ExtraInfo.objects.get(user=current_user)
    
    # Student access restriction
    if extraInfo.user_type == "student":
        return HttpResponseRedirect('/')
    
    # Only faculty/staff can access estate management
    # User types: 'student', 'faculty', 'staff'
```

#### Permission Matrix
```python
# User Type Access Control
USER_PERMISSIONS = {
    'student': {
        'can_view': False,
        'can_create': False, 
        'can_edit': False,
        'can_delete': False
    },
    'faculty': {
        'can_view': True,
        'can_create': False,
        'can_edit': False, 
        'can_delete': False
    },
    'staff': {
        'can_view': True,
        'can_create': True,
        'can_edit': True,
        'can_delete': True
    },
    'admin': {
        'can_view': True,
        'can_create': True,
        'can_edit': True,
        'can_delete': True,
        'can_verify': True
    }
}
```

### Advanced Status Calculation Algorithms

#### Building Status Logic
```python
def status(self):
    from django.utils import timezone
    
    if not self.dateConstructionCompleted:
        if self.dateConstructionStarted:
            # Calculate expected completion (assume 365 days construction period)
            expected_completion = self.dateConstructionStarted + timedelta(days=365)
            if timezone.now().date() > expected_completion:
                return 'Delayed'
        elif self.dateIssued:
            # If construction hasn't started but should have (assume 30 days prep)
            expected_start = self.dateIssued + timedelta(days=30)
            if timezone.now().date() > expected_start:
                return 'Delayed'
    
    return 'On Schedule'
```

#### Work Progress Calculation
```python
def progress_percentage(self):
    if not self.dateStarted:
        return 0
    if self.dateCompleted:
        return 100
        
    # Calculate based on subworks completion
    subworks = self.subwork_set.all()
    if subworks:
        completed_subworks = subworks.filter(dateCompleted__isnull=False).count()
        return (completed_subworks / subworks.count()) * 100
    
    # Fallback: time-based calculation
    if self.dateStarted:
        total_days = (timezone.now().date() - self.dateStarted).days
        estimated_days = 180  # Default 6 months
        return min((total_days / estimated_days) * 100, 95)  # Cap at 95% until complete
    
    return 0
```

### Complete Error Handling System

#### View Error Management
```python
# Error handling in CRUD operations
@require_POST
def newBuilding(request):
    try:
        new_building_form = BuildingForm(request.POST)
        if new_building_form.is_valid():
            new_building = new_building_form.save()
            messages.success(request, f'New Building Created: {new_building.name}')
            return redirect('estate_module_home')
        
        # Form validation errors
        for label, errors in new_building_form.errors.items():
            for error in errors:
                messages.error(request, f'{label}: {error}')
                
    except ValidationError as e:
        messages.error(request, f'Validation Error: {str(e)}')
    except IntegrityError as e:
        messages.error(request, f'Database Error: {str(e)}')
    except Exception as e:
        messages.error(request, f'Unexpected Error: {str(e)}')
        
    return redirect('estate_module_home')
```

#### Form Error Display
```html
<!-- Error message display in templates -->
<div class="ui error message">
    <div class="header">Please correct the following errors:</div>
    <ul class="list">
        {% for field, errors in form.errors.items %}
            {% for error in errors %}
                <li>{{ field|title }}: {{ error }}</li>
            {% endfor %}
        {% endfor %}
    </ul>
</div>
```

### Complete Test Framework Structure

#### Model Tests
```python
# tests.py (Currently empty - should include)
from django.test import TestCase, Client
from django.contrib.auth.models import User
from django.urls import reverse
from .models import Building, Work, SubWork, InventoryType
from datetime import date, timedelta

class BuildingModelTests(TestCase):
    def setUp(self):
        self.building = Building.objects.create(
            name="Test Building",
            dateIssued=date.today(),
            area=1000,
            constructionCostEstimated=5000000
        )
    
    def test_building_creation(self):
        self.assertEqual(self.building.name, "Test Building")
        self.assertEqual(self.building.verified, False)
    
    def test_status_calculation(self):
        # Test on schedule
        self.assertEqual(self.building.status(), 'On Schedule')
        
        # Test delayed
        self.building.dateConstructionStarted = date.today() - timedelta(days=400)
        self.assertEqual(self.building.status(), 'Delayed')

class WorkModelTests(TestCase):
    def setUp(self):
        self.building = Building.objects.create(name="Test Building", dateIssued=date.today())
        self.work = Work.objects.create(
            name="Test Work",
            workType="CW",
            contractorName="Test Contractor", 
            dateIssued=date.today(),
            building=self.building
        )
    
    def test_work_creation(self):
        self.assertEqual(self.work.name, "Test Work")
        self.assertEqual(self.work.workType, "CW")

class ViewTests(TestCase):
    def setUp(self):
        self.client = Client()
        self.user = User.objects.create_user(username='testuser', password='testpass')
    
    def test_estate_view_requires_login(self):
        response = self.client.get(reverse('estate_module_home'))
        self.assertRedirects(response, '/accounts/login/?next=/estate/')
    
    def test_estate_view_authenticated(self):
        self.client.login(username='testuser', password='testpass')
        response = self.client.get(reverse('estate_module_home'))
        self.assertEqual(response.status_code, 200)
```

### Data Export/Import Capabilities

#### CSV Export Functionality
```python
import csv
from django.http import HttpResponse

def export_buildings_csv(request):
    response = HttpResponse(content_type='text/csv')
    response['Content-Disposition'] = 'attachment; filename="buildings.csv"'
    
    writer = csv.writer(response)
    writer.writerow(['Name', 'Date Issued', 'Status', 'Area', 'Estimated Cost', 'Actual Cost'])
    
    buildings = Building.objects.all()
    for building in buildings:
        writer.writerow([
            building.name,
            building.dateIssued,
            building.status(),
            building.area,
            building.constructionCostEstimated,
            building.constructionCostActual
        ])
    
    return response
```

### Performance Optimization Features

#### Database Query Optimization
```python
# Optimized queries in views
def estate(request):
    # Use select_related and prefetch_related for efficiency
    buildings = Building.objects.select_related().prefetch_related('work_set__subwork_set')
    works = Work.objects.select_related('building').prefetch_related('subwork_set') 
    inventory_consumable = InventoryConsumable.objects.select_related('inventoryType', 'building', 'work')
    inventory_non_consumable = InventoryNonConsumable.objects.select_related('inventoryType', 'building', 'work', 'issued_to')
```

This Estate Module provides comprehensive infrastructure lifecycle management, enabling educational institutions to effectively plan, execute, and maintain their physical assets while ensuring accountability and cost control throughout all processes.
