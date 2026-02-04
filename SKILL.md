---
name: frappe-framework
description: Comprehensive guide for Frappe Framework development. Use this skill when building web applications with Frappe, working with DocTypes, creating custom apps, implementing REST APIs, writing controller logic, database operations, form scripts, or bench commands. Triggers include any mention of "Frappe", "ERPNext", "DocType", "bench", "frappe.get_doc", "frappe.db", or building business applications with Python/JavaScript full-stack framework.
---

# Frappe Framework Development

Frappe is a full-stack, batteries-included web framework written in Python and JavaScript with MariaDB/PostgreSQL as database. It powers ERPNext and is ideal for building database-driven business applications.

## Core Concepts

### Architecture
- **Bench**: Development environment manager (sites, apps, dependencies)
- **Site**: A deployment instance with its own database
- **App**: A module containing DocTypes, controllers, and business logic
- **DocType**: Core building block - defines Model + View (auto-generates DB table, forms, REST API)

### Directory Structure
```
frappe-bench/
├── apps/
│   ├── frappe/           # Core framework
│   └── your_app/         # Custom app
│       ├── your_app/
│       │   ├── your_module/
│       │   │   └── doctype/
│       │   │       └── your_doctype/
│       │   │           ├── your_doctype.json    # DocType definition
│       │   │           ├── your_doctype.py      # Controller
│       │   │           └── your_doctype.js      # Form script
│       │   └── hooks.py
│       └── setup.py
└── sites/
    └── your_site/
        └── site_config.json
```

## Document Operations

### Creating Documents
```python
# Method 1: Using dict
doc = frappe.get_doc({
    'doctype': 'Task',
    'subject': 'New Task',
    'status': 'Open'
})
doc.insert()

# Method 2: Using new_doc
doc = frappe.new_doc('Task')
doc.subject = 'New Task'
doc.insert()

# With child table
doc = frappe.get_doc({
    'doctype': 'Sales Order',
    'customer': 'Customer Name',
    'items': [
        {'item_code': 'ITEM-001', 'qty': 10, 'rate': 100}
    ]
})
doc.insert()
```

### Reading Documents
```python
# Get single document
doc = frappe.get_doc('Task', 'TASK-0001')

# With caching
doc = frappe.get_cached_doc('Task', 'TASK-0001')

# Get single field value
subject = frappe.db.get_value('Task', 'TASK-0001', 'subject')

# Get multiple fields
subject, status = frappe.db.get_value('Task', 'TASK-0001', ['subject', 'status'])

# As dict
task = frappe.db.get_value('Task', 'TASK-0001', ['subject', 'status'], as_dict=True)
```

### Updating Documents
```python
# Full ORM (triggers hooks)
doc = frappe.get_doc('Task', 'TASK-0001')
doc.status = 'Completed'
doc.save()

# Direct DB update (no hooks, faster)
frappe.db.set_value('Task', 'TASK-0001', 'status', 'Completed')

# Bulk update
frappe.db.set_value('Task', 'TASK-0001', {
    'status': 'Completed',
    'priority': 'High'
})
```

### Deleting Documents
```python
frappe.delete_doc('Task', 'TASK-0001')

# Or via document
doc = frappe.get_doc('Task', 'TASK-0001')
doc.delete()

# Bulk delete
frappe.db.delete('Task', {'status': 'Cancelled'})
```

## Database Queries

### get_list / get_all
```python
# get_list applies user permissions, get_all ignores them
tasks = frappe.db.get_list('Task',
    filters={'status': 'Open'},
    fields=['name', 'subject', 'date'],
    order_by='date desc',
    start=0,
    page_length=20
)

# Filter operators
filters = {
    'date': ['>', '2024-01-01'],
    'status': ['in', ['Open', 'Working']],
    'subject': ['like', '%urgent%']
}

# Between dates
filters = [['date', 'between', ['2024-01-01', '2024-12-31']]]

# Pluck single field
names = frappe.db.get_all('Task', filters={'status': 'Open'}, pluck='name')
```

### exists / count
```python
# Check existence
if frappe.db.exists('Task', 'TASK-0001'):
    pass

if frappe.db.exists('Task', {'status': 'Open', 'assigned_to': 'user@example.com'}):
    pass

# Count records
total = frappe.db.count('Task')
open_tasks = frappe.db.count('Task', {'status': 'Open'})
```

### Raw SQL
```python
# Use parameterized queries to prevent SQL injection
data = frappe.db.sql("""
    SELECT t.name, t.subject, u.full_name
    FROM `tabTask` t
    LEFT JOIN `tabUser` u ON t.assigned_to = u.name
    WHERE t.status = %(status)s
""", {'status': 'Open'}, as_dict=True)
```

## Controller Hooks

Create controller at `doctype/your_doctype/your_doctype.py`:

```python
import frappe
from frappe.model.document import Document

class YourDoctype(Document):
    def before_validate(self):
        """Auto-set missing values"""
        if not self.date:
            self.date = frappe.utils.today()
    
    def validate(self):
        """Validation logic - throw errors here"""
        if self.amount < 0:
            frappe.throw("Amount cannot be negative")
    
    def before_save(self):
        """Before saving to database"""
        self.calculate_totals()
    
    def after_insert(self):
        """After new document is created"""
        self.send_notification()
    
    def on_update(self):
        """After document is saved (insert or update)"""
        self.update_related_docs()
    
    def before_submit(self):
        """Before document is submitted"""
        self.validate_for_submission()
    
    def on_submit(self):
        """After document is submitted"""
        self.create_gl_entries()
    
    def on_cancel(self):
        """When submitted document is cancelled"""
        self.reverse_gl_entries()
    
    def on_trash(self):
        """Before document is deleted"""
        self.cleanup_related_data()
```

### Hook Execution Order
**Insert**: before_insert → before_naming → autoname → before_validate → validate → before_save → db_insert → after_insert → on_update → on_change

**Save**: before_validate → validate → before_save → db_update → on_update → on_change

**Submit**: before_validate → validate → before_submit → db_update → on_submit → on_update → on_change

## Form Scripts (Client-Side)

Create at `doctype/your_doctype/your_doctype.js`:

```javascript
frappe.ui.form.on('Your Doctype', {
    // Form events
    setup(frm) {
        // Called once when form is setup
    },
    
    refresh(frm) {
        // Called when form is refreshed/loaded
        if (!frm.is_new()) {
            frm.add_custom_button('Process', () => {
                frm.call('process_document');
            });
        }
    },
    
    validate(frm) {
        // Client-side validation
        if (frm.doc.amount < 0) {
            frappe.throw('Amount cannot be negative');
        }
    },
    
    // Field events
    customer(frm) {
        // When customer field changes
        if (frm.doc.customer) {
            frappe.call({
                method: 'get_customer_details',
                args: { customer: frm.doc.customer },
                callback: (r) => {
                    frm.set_value('customer_name', r.message.customer_name);
                }
            });
        }
    },
    
    // Child table events
    items_add(frm, cdt, cdn) {
        // When row is added to items table
    },
    
    items_remove(frm, cdt, cdn) {
        // When row is removed
    }
});

// Child table field events
frappe.ui.form.on('Your Doctype Item', {
    qty(frm, cdt, cdn) {
        let row = locals[cdt][cdn];
        frappe.model.set_value(cdt, cdn, 'amount', row.qty * row.rate);
        calculate_total(frm);
    }
});
```

## Whitelisted Methods (API Endpoints)

```python
# In your_doctype.py or any Python file

@frappe.whitelist()
def get_customer_details(customer):
    """Callable from client: frappe.call({method: 'app.module.get_customer_details'})"""
    return frappe.db.get_value('Customer', customer, ['customer_name', 'email'], as_dict=True)

@frappe.whitelist(allow_guest=True)
def public_api():
    """Accessible without login"""
    return {'status': 'ok'}

# Document method
class YourDoctype(Document):
    @frappe.whitelist()
    def process_document(self):
        """Callable from client: frm.call('process_document')"""
        self.status = 'Processed'
        self.save()
        return {'message': 'Document processed'}
```

## REST API

### Authentication
```python
# Token auth header
Authorization: token api_key:api_secret

# Or Basic auth
Authorization: Basic base64(username:password)
```

### Endpoints
```
# List documents
GET /api/resource/DocType?filters=[["status","=","Open"]]&fields=["name","subject"]

# Get single document  
GET /api/resource/DocType/DOC-0001

# Create document
POST /api/resource/DocType
Body: {"subject": "New Task", "status": "Open"}

# Update document
PUT /api/resource/DocType/DOC-0001
Body: {"status": "Completed"}

# Delete document
DELETE /api/resource/DocType/DOC-0001

# Call whitelisted method
GET/POST /api/method/app.module.method_name
```

## Bench Commands

```bash
# Site management
bench new-site site_name
bench use site_name
bench drop-site site_name

# App management
bench new-app app_name
bench get-app https://github.com/org/app
bench install-app app_name
bench uninstall-app app_name

# Development
bench start                    # Start development server
bench --site site_name console # Python console
bench --site site_name mariadb # Database console

# Migrations
bench --site site_name migrate
bench --site site_name clear-cache

# Backup/Restore
bench --site site_name backup
bench --site site_name restore /path/to/backup.sql.gz

# Production
bench setup production user
bench setup nginx
bench setup supervisor
```

## Hooks (hooks.py)

```python
app_name = "your_app"
app_title = "Your App"

# Document events
doc_events = {
    "Sales Order": {
        "on_submit": "your_app.handlers.on_sales_order_submit",
        "on_cancel": "your_app.handlers.on_sales_order_cancel"
    },
    "*": {  # All doctypes
        "after_insert": "your_app.handlers.log_creation"
    }
}

# Scheduled tasks
scheduler_events = {
    "daily": ["your_app.tasks.daily_task"],
    "hourly": ["your_app.tasks.hourly_task"],
    "cron": {
        "0 */6 * * *": ["your_app.tasks.every_6_hours"]
    }
}

# Override whitelisted methods
override_whitelisted_methods = {
    "frappe.client.get_count": "your_app.overrides.custom_get_count"
}

# Fixtures (export to app)
fixtures = [
    {"dt": "Custom Field", "filters": [["module", "=", "Your App"]]},
    {"dt": "Property Setter", "filters": [["module", "=", "Your App"]]}
]
```

## Background Jobs

```python
# Enqueue a job
frappe.enqueue(
    'your_app.tasks.long_running_task',
    queue='long',  # short, default, long
    timeout=600,
    arg1='value1',
    arg2='value2'
)

# Enqueue document method
frappe.enqueue_doc('DocType', 'name', 'method_name', queue='short')
```

## Utility Functions

```python
# Date/Time
from frappe.utils import today, now, add_days, date_diff, getdate

today()                        # '2024-01-15'
now()                         # '2024-01-15 10:30:00'
add_days(today(), 7)          # Add 7 days
date_diff(date1, date2)       # Difference in days

# Formatting
from frappe.utils import fmt_money, flt, cint, cstr

fmt_money(1000, currency='USD')  # '$1,000.00'
flt('123.45')                    # 123.45 (float)
cint('123')                      # 123 (int)

# Email
frappe.sendmail(
    recipients=['user@example.com'],
    subject='Subject',
    message='Body',
    template='template_name',
    args={'key': 'value'}
)

# Messages
frappe.msgprint('Message to show')
frappe.throw('Error message')  # Stops execution
```

## Common Patterns

### Permission Check
```python
if not frappe.has_permission('DocType', 'write', doc=doc):
    frappe.throw('Not permitted', frappe.PermissionError)
```

### Transaction Safety
```python
try:
    doc1.save()
    doc2.save()
    frappe.db.commit()
except Exception:
    frappe.db.rollback()
    raise
```

### Caching
```python
# Simple cache
@frappe.whitelist()
def get_settings():
    return frappe.cache().get_value('settings', get_settings_from_db)

# Hget
frappe.cache().hset('namespace', 'key', value)
value = frappe.cache().hget('namespace', 'key')
```

## References

For detailed API documentation, see:
- [references/python-api.md](references/python-api.md) - Document and Database API details
- [references/js-api.md](references/js-api.md) - Form Scripts and Client API
- [references/bench-commands.md](references/bench-commands.md) - Complete bench command reference

Official documentation: https://docs.frappe.io/framework
