# Python API Reference

## Document API

### frappe.get_doc
Returns a Document object for the record.

```python
# Get existing document
doc = frappe.get_doc('Task', 'TASK-0001')

# Get Single DocType (no name needed)
settings = frappe.get_doc('System Settings')

# Create new document (in memory, not saved)
doc = frappe.get_doc({
    'doctype': 'Task',
    'subject': 'New Task',
    'status': 'Open'
})
doc.insert()  # Save to database
```

### frappe.get_cached_doc
Same as get_doc but caches the result. Use for frequently accessed, rarely modified documents.

```python
doc = frappe.get_cached_doc('Company', 'My Company')
```

### frappe.new_doc
Creates a new Document object.

```python
doc = frappe.new_doc('Task')
doc.subject = 'New Task'
doc.insert()
```

### frappe.delete_doc
Deletes a document and its children.

```python
frappe.delete_doc('Task', 'TASK-0001')

# Ignore permissions
frappe.delete_doc('Task', 'TASK-0001', ignore_permissions=True)

# Force delete (bypasses linked document check)
frappe.delete_doc('Task', 'TASK-0001', force=True)
```

### frappe.rename_doc
Renames a document (changes primary key).

```python
frappe.rename_doc('Task', 'TASK-0001', 'TASK-NEW-0001')

# Merge with existing
frappe.rename_doc('Customer', 'Old Name', 'Existing Name', merge=True)
```

### frappe.get_meta
Returns DocType metadata.

```python
meta = frappe.get_meta('Task')
meta.has_field('status')           # True/False
meta.get_field('status')           # Field object
meta.get_custom_fields()           # List of custom fields
meta.get_link_fields()             # Fields of type Link
meta.get_valid_columns()           # All valid column names
```

## Document Methods

### doc.insert()
Insert document into database.

```python
doc.insert(
    ignore_permissions=False,
    ignore_links=False,
    ignore_if_duplicate=False,
    ignore_mandatory=False,
    set_name=None,          # Force specific name
    set_child_names=True
)
```

### doc.save()
Save changes to existing document.

```python
doc.save(
    ignore_permissions=False,
    ignore_version=False
)
```

### doc.submit()
Submit document (for submittable DocTypes).

```python
doc.submit()
```

### doc.cancel()
Cancel submitted document.

```python
doc.cancel()
```

### doc.delete()
Delete the document.

```python
doc.delete(
    ignore_permissions=False,
    force=False,
    delete_permanently=False
)
```

### doc.reload()
Reload document from database.

```python
doc.reload()
```

### doc.get_doc_before_save()
Get document state before changes.

```python
old = doc.get_doc_before_save()
if old.status != doc.status:
    # Status changed
    pass
```

### doc.as_dict()
Convert document to dictionary.

```python
data = doc.as_dict()
```

### doc.get()
Safe attribute access.

```python
value = doc.get('field_name')           # Returns None if not exists
value = doc.get('field_name', 'default')  # With default
```

### doc.append()
Add row to child table.

```python
doc.append('items', {
    'item_code': 'ITEM-001',
    'qty': 10,
    'rate': 100
})
```

### doc.set()
Set field value.

```python
doc.set('status', 'Open')
doc.set('items', [])  # Clear child table
```

### doc.db_set()
Direct database update (no hooks).

```python
doc.db_set('status', 'Closed')
doc.db_set('status', 'Closed', update_modified=False)

# Multiple fields
doc.db_set({
    'status': 'Closed',
    'completion_date': frappe.utils.today()
})
```

### doc.run_method()
Run a method if it exists.

```python
doc.run_method('custom_method', arg1='value')
```

### doc.add_comment()
Add comment to document.

```python
doc.add_comment('Comment', 'This is a comment')
doc.add_comment('Edit', 'Document was edited')
```

### doc.notify_update()
Publish realtime update notification.

```python
doc.notify_update()
```

---

## Database API

### frappe.db.get_list / frappe.get_list
Get list of documents with user permissions applied.

```python
frappe.db.get_list('Task',
    filters={'status': 'Open'},
    or_filters={'priority': 'High'},
    fields=['name', 'subject', 'date'],
    order_by='date desc',
    group_by='status',
    start=0,
    page_length=20,
    as_list=False,      # True returns list of lists
    pluck='name',       # Returns list of single field
    ignore_ifnull=False
)
```

### Filter Operators
```python
# Equals
{'status': 'Open'}
{'status': ['=', 'Open']}

# Not equals
{'status': ['!=', 'Closed']}

# Greater/Less than
{'date': ['>', '2024-01-01']}
{'date': ['>=', '2024-01-01']}
{'date': ['<', '2024-12-31']}
{'date': ['<=', '2024-12-31']}

# In list
{'status': ['in', ['Open', 'Working']]}
{'status': ['not in', ['Closed', 'Cancelled']]}

# Like
{'subject': ['like', '%urgent%']}
{'subject': ['not like', '%test%']}

# Between
[['date', 'between', ['2024-01-01', '2024-12-31']]]

# Is null / Not null
{'assigned_to': ['is', 'set']}
{'assigned_to': ['is', 'not set']}

# Nested filters with or_filters
frappe.db.get_list('Task',
    filters={'status': 'Open'},
    or_filters=[
        {'priority': 'High'},
        {'assigned_to': 'admin@example.com'}
    ]
)
```

### frappe.db.get_all / frappe.get_all
Same as get_list but ignores user permissions.

```python
frappe.db.get_all('Task', filters={'status': 'Open'})
```

### frappe.db.get_value / frappe.get_value
Get field value(s) for a document.

```python
# Single field
subject = frappe.db.get_value('Task', 'TASK-0001', 'subject')

# Multiple fields
subject, status = frappe.db.get_value('Task', 'TASK-0001', ['subject', 'status'])

# As dict
task = frappe.db.get_value('Task', 'TASK-0001', ['subject', 'status'], as_dict=True)

# With filters (returns first match)
subject = frappe.db.get_value('Task', {'status': 'Open'}, 'subject')

# Cache result
subject = frappe.db.get_value('Task', 'TASK-0001', 'subject', cache=True)
```

### frappe.db.get_single_value
Get field from Single DocType.

```python
timezone = frappe.db.get_single_value('System Settings', 'time_zone')
```

### frappe.db.set_value / frappe.db.update
Update field(s) directly in database.

```python
# Single field
frappe.db.set_value('Task', 'TASK-0001', 'status', 'Closed')

# Multiple fields
frappe.db.set_value('Task', 'TASK-0001', {
    'status': 'Closed',
    'priority': 'Low'
})

# Don't update modified timestamp
frappe.db.set_value('Task', 'TASK-0001', 'status', 'Closed', update_modified=False)
```

### frappe.db.exists
Check if document exists.

```python
# By name
frappe.db.exists('Task', 'TASK-0001')

# By filters
frappe.db.exists('Task', {'status': 'Open', 'assigned_to': 'user@example.com'})

# With cache
frappe.db.exists('Task', 'TASK-0001', cache=True)
```

### frappe.db.count
Count documents.

```python
# All
total = frappe.db.count('Task')

# With filters
open_tasks = frappe.db.count('Task', {'status': 'Open'})
```

### frappe.db.delete
Delete records matching filters.

```python
frappe.db.delete('Task', {'status': 'Cancelled'})
frappe.db.delete('Error Log')  # Delete all
```

### frappe.db.truncate
Truncate table (faster than delete, cannot rollback).

```python
frappe.db.truncate('Error Log')
```

### frappe.db.sql
Execute raw SQL.

```python
# Parameterized query (prevents SQL injection)
data = frappe.db.sql("""
    SELECT name, subject, status
    FROM `tabTask`
    WHERE status = %(status)s
    AND date >= %(date)s
""", {
    'status': 'Open',
    'date': '2024-01-01'
}, as_dict=True)

# As list of lists
data = frappe.db.sql("SELECT name, subject FROM `tabTask`", as_list=True)
```

### frappe.db.commit
Commit transaction.

```python
frappe.db.commit()
```

### frappe.db.rollback
Rollback transaction.

```python
frappe.db.rollback()
```

### frappe.db.savepoint
Create savepoint for partial rollback.

```python
frappe.db.savepoint('before_update')
try:
    # operations
except:
    frappe.db.rollback(save_point='before_update')
```

---

## Utility Functions

### Date/Time (frappe.utils)
```python
from frappe.utils import (
    today, now, now_datetime, nowdate, nowtime,
    add_days, add_months, add_years,
    date_diff, time_diff, time_diff_in_hours,
    getdate, get_datetime, get_time,
    format_date, format_datetime, format_time
)

today()                          # '2024-01-15'
now()                           # '2024-01-15 10:30:00.123456'
nowdate()                       # '2024-01-15'
nowtime()                       # '10:30:00.123456'
now_datetime()                  # datetime object

add_days('2024-01-15', 7)       # '2024-01-22'
add_months('2024-01-15', 1)     # '2024-02-15'
date_diff('2024-01-20', '2024-01-15')  # 5

getdate('2024-01-15')           # date object
get_datetime('2024-01-15 10:30:00')  # datetime object
```

### Number Formatting
```python
from frappe.utils import flt, cint, cstr, fmt_money, rounded

flt('123.456')                  # 123.456
flt('123.456', 2)               # 123.46 (precision)
cint('123')                     # 123
cint('abc')                     # 0
cstr(123)                       # '123'
fmt_money(1234.56, currency='USD')  # '$1,234.56'
rounded(123.456, 2)             # 123.46
```

### String Operations
```python
from frappe.utils import strip_html, escape_html, cstr, scrub

strip_html('<p>Hello</p>')      # 'Hello'
escape_html('<script>')         # '&lt;script&gt;'
scrub('My Field Name')          # 'my_field_name'
```

### File Operations
```python
from frappe.utils.file_manager import save_file, get_file

# Save file
file_doc = save_file(
    fname='document.pdf',
    content=file_content,
    dt='Task',
    dn='TASK-0001',
    is_private=1
)

# Get file content
file_name, content = get_file(file_url)
```

---

## Messages and Errors

```python
# Display message (non-blocking)
frappe.msgprint('Operation completed')
frappe.msgprint('Warning', indicator='orange')
frappe.msgprint('Success', indicator='green', title='Done')

# Throw error (stops execution)
frappe.throw('Invalid input')
frappe.throw('Not permitted', frappe.PermissionError)
frappe.throw('Not found', frappe.DoesNotExistError)

# Validation error (collected, shown together)
frappe.validation_error.append(('field', 'Error message'))

# Log error
frappe.log_error('Error details', 'Error Title')
frappe.log_error(frappe.get_traceback())
```

---

## Email

```python
frappe.sendmail(
    recipients=['user@example.com'],
    cc=['cc@example.com'],
    bcc=['bcc@example.com'],
    subject='Email Subject',
    message='Email body in HTML',
    template='template_name',           # Email template
    args={'key': 'value'},             # Template variables
    attachments=[{'fname': 'file.pdf', 'fcontent': content}],
    reference_doctype='Task',
    reference_name='TASK-0001',
    delayed=True,                       # Queue for background sending
    now=False
)
```

---

## Caching

```python
# Simple key-value
frappe.cache().set_value('key', value, expires_in_sec=3600)
value = frappe.cache().get_value('key')
frappe.cache().delete_value('key')

# Hash operations
frappe.cache().hset('namespace', 'key', value)
value = frappe.cache().hget('namespace', 'key')
frappe.cache().hdel('namespace', 'key')
all_values = frappe.cache().hgetall('namespace')

# Decorator
@frappe.cache()
def expensive_function():
    return result
```

---

## Permissions

```python
# Check permission
frappe.has_permission('Task', 'read')
frappe.has_permission('Task', 'write', doc=doc)
frappe.has_permission('Task', 'create', throw=True)

# Get permitted documents
frappe.get_all('Task', filters={'status': 'Open'})  # Auto applies permissions

# Permission levels: read, write, create, delete, submit, cancel, amend, print, email, share

# Run without permission checks
frappe.flags.ignore_permissions = True
# or
doc.insert(ignore_permissions=True)
```

---

## Background Jobs

```python
# Enqueue function
frappe.enqueue(
    'your_app.tasks.process_data',
    queue='long',           # short, default, long
    timeout=600,            # seconds
    is_async=True,
    at_front=False,
    job_name='unique_job_id',
    arg1='value1',
    arg2='value2'
)

# Enqueue document method
frappe.enqueue_doc('Task', 'TASK-0001', 'process', queue='default')

# Check if already enqueued
from frappe.utils.background_jobs import is_job_enqueued
if not is_job_enqueued('job_name'):
    frappe.enqueue(...)
```
