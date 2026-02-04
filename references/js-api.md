# JavaScript API Reference

## Form Scripts

Form scripts customize the behavior of DocType forms. Create at `doctype/<doctype_name>/<doctype_name>.js`.

### Form Object (frm)

```javascript
frappe.ui.form.on('DocType Name', {
    // Called once when form loads
    setup(frm) {
        frm.set_query('customer', () => {
            return { filters: { 'status': 'Active' } };
        });
    },
    
    // Called on every refresh
    refresh(frm) {
        // Add custom button
        if (!frm.is_new()) {
            frm.add_custom_button(__('Action'), () => {
                // action code
            }, __('Group'));
        }
        
        // Hide field
        frm.set_df_property('field_name', 'hidden', 1);
        
        // Set field read only
        frm.set_df_property('field_name', 'read_only', 1);
    },
    
    // Before save
    validate(frm) {
        if (frm.doc.amount < 0) {
            frappe.throw(__('Amount cannot be negative'));
            validated = false;
        }
    },
    
    // Before submit
    before_submit(frm) {
        return new Promise((resolve, reject) => {
            frappe.confirm('Are you sure?', resolve, reject);
        });
    },
    
    // After save
    after_save(frm) {
        frappe.show_alert({
            message: __('Document saved'),
            indicator: 'green'
        });
    },
    
    // On load
    onload(frm) {
        // Initial setup
    },
    
    // Before load
    before_load(frm) {
        // Before data loaded
    },
    
    // On submit
    on_submit(frm) {
        // After submit
    },
    
    // After cancel
    after_cancel(frm) {
        // After document cancelled
    }
});
```

### Field Events

```javascript
frappe.ui.form.on('DocType Name', {
    // When field value changes
    field_name(frm) {
        if (frm.doc.field_name) {
            // Do something
            frm.set_value('other_field', computed_value);
        }
    },
    
    // Before value change
    before_field_name_change(frm) {
        // Validate before change
    }
});
```

### Child Table Events

```javascript
frappe.ui.form.on('Child DocType Name', {
    // When row is added
    items_add(frm, cdt, cdn) {
        let row = locals[cdt][cdn];
        frappe.model.set_value(cdt, cdn, 'date', frappe.datetime.get_today());
    },
    
    // When row is removed
    items_remove(frm, cdt, cdn) {
        calculate_total(frm);
    },
    
    // When field in row changes
    qty(frm, cdt, cdn) {
        let row = locals[cdt][cdn];
        frappe.model.set_value(cdt, cdn, 'amount', row.qty * row.rate);
    },
    
    // Before row is moved
    items_before_move(frm) {
        // Custom logic
    },
    
    // After row is moved
    items_move(frm) {
        // Recalculate idx
    }
});

// Access child table rows
let items = frm.doc.items || [];
items.forEach(row => {
    console.log(row.item_code, row.qty);
});
```

---

## frm Methods

### Field Manipulation

```javascript
// Set value
frm.set_value('field_name', value);
frm.set_value({
    field1: value1,
    field2: value2
});

// Get value
let value = frm.doc.field_name;

// Set field properties
frm.set_df_property('field_name', 'read_only', 1);
frm.set_df_property('field_name', 'hidden', 1);
frm.set_df_property('field_name', 'reqd', 1);
frm.set_df_property('field_name', 'label', 'New Label');
frm.set_df_property('field_name', 'options', 'Option1\nOption2\nOption3');
frm.set_df_property('field_name', 'description', 'Help text');

// Toggle display
frm.toggle_display('field_name', true);  // Show
frm.toggle_display(['field1', 'field2'], false);  // Hide multiple

// Toggle required
frm.toggle_reqd('field_name', true);

// Toggle read only
frm.toggle_enable('field_name', false);  // Read only
```

### Link Field Queries

```javascript
// Filter link field options
frm.set_query('customer', () => {
    return {
        filters: {
            'status': 'Active',
            'territory': frm.doc.territory
        }
    };
});

// Dynamic query
frm.set_query('item_code', 'items', () => {
    return {
        filters: {
            'item_group': 'Products'
        }
    };
});

// Query with custom method
frm.set_query('customer', () => {
    return {
        query: 'app.module.get_customers',
        filters: { 'type': 'Retail' }
    };
});
```

### Custom Buttons

```javascript
// Add button
frm.add_custom_button(__('Process'), () => {
    // Action
});

// Button in group
frm.add_custom_button(__('Email'), () => {
    // Send email
}, __('Actions'));

// Primary button
frm.page.set_primary_action(__('Submit'), () => {
    frm.save('Submit');
});

// Remove button
frm.remove_custom_button(__('Process'));
frm.remove_custom_button(__('Email'), __('Actions'));

// Clear all buttons in group
frm.clear_custom_buttons();
```

### Dashboard

```javascript
frm.dashboard.set_headline_alert('Alert message', 'orange');

frm.dashboard.add_indicator(__('Status'), 'blue');

frm.dashboard.add_comment('This is a comment');

// Add section
frm.dashboard.add_section({
    label: __('Info'),
    body: '<p>Dashboard content</p>'
});
```

### Server Calls

```javascript
// Call document method
frm.call({
    method: 'custom_method',
    args: { param1: 'value' },
    callback: (r) => {
        if (r.message) {
            console.log(r.message);
        }
    },
    freeze: true,
    freeze_message: __('Processing...')
});

// Call whitelisted method
frappe.call({
    method: 'app.module.method_name',
    args: { docname: frm.doc.name },
    callback: (r) => {
        frm.reload_doc();
    }
});
```

### Form State

```javascript
frm.is_new()         // True if unsaved new doc
frm.is_dirty()       // True if has unsaved changes
frm.doc.docstatus    // 0=Draft, 1=Submitted, 2=Cancelled

frm.save()           // Save document
frm.save('Submit')   // Submit document
frm.reload_doc()     // Reload from server

frm.scroll_to_field('field_name');
frm.set_intro('Introductory text');
```

---

## Global frappe Methods

### Dialogs

```javascript
// Simple message
frappe.msgprint(__('Message'));

// With indicator
frappe.msgprint({
    title: __('Info'),
    message: __('Document created'),
    indicator: 'green'  // green, blue, orange, red
});

// Alert (auto-close)
frappe.show_alert({
    message: __('Saved'),
    indicator: 'green'
}, 5);  // seconds

// Confirmation
frappe.confirm(
    __('Are you sure?'),
    () => {
        // Yes callback
    },
    () => {
        // No callback
    }
);

// Prompt
frappe.prompt(
    {
        fieldname: 'value',
        fieldtype: 'Data',
        label: __('Enter Value'),
        reqd: 1
    },
    (values) => {
        console.log(values.value);
    },
    __('Enter Details'),
    __('Submit')
);

// Multiple fields prompt
frappe.prompt([
    {
        fieldname: 'name',
        fieldtype: 'Data',
        label: __('Name'),
        reqd: 1
    },
    {
        fieldname: 'type',
        fieldtype: 'Select',
        label: __('Type'),
        options: 'Type A\nType B\nType C'
    }
], (values) => {
    console.log(values);
}, __('Enter Details'));

// Throw error
frappe.throw(__('Error message'));
```

### Custom Dialog

```javascript
let dialog = new frappe.ui.Dialog({
    title: __('Custom Dialog'),
    fields: [
        {
            fieldname: 'customer',
            fieldtype: 'Link',
            options: 'Customer',
            label: __('Customer'),
            reqd: 1
        },
        {
            fieldname: 'column_break',
            fieldtype: 'Column Break'
        },
        {
            fieldname: 'date',
            fieldtype: 'Date',
            label: __('Date'),
            default: frappe.datetime.get_today()
        },
        {
            fieldname: 'section_break',
            fieldtype: 'Section Break',
            label: __('Details')
        },
        {
            fieldname: 'remarks',
            fieldtype: 'Text',
            label: __('Remarks')
        }
    ],
    primary_action_label: __('Submit'),
    primary_action(values) {
        console.log(values);
        dialog.hide();
    }
});

dialog.show();
```

### AJAX Calls

```javascript
// Call whitelisted method
frappe.call({
    method: 'app.module.method_name',
    args: {
        param1: 'value1',
        param2: 'value2'
    },
    async: true,
    freeze: true,
    freeze_message: __('Loading...'),
    callback: (r) => {
        if (r.message) {
            console.log(r.message);
        }
    },
    error: (r) => {
        console.error(r);
    }
});

// With promise
frappe.call({
    method: 'app.module.method_name',
    args: { param: 'value' }
}).then(r => {
    console.log(r.message);
});

// Using async/await
let response = await frappe.call({
    method: 'app.module.method_name',
    args: { param: 'value' }
});
console.log(response.message);

// xcall (promise-based shorthand)
let result = await frappe.xcall('app.module.method_name', { param: 'value' });
```

### Document Operations

```javascript
// Get document
frappe.db.get_doc('DocType', 'name').then(doc => {
    console.log(doc);
});

// Get value
frappe.db.get_value('DocType', 'name', 'field').then(r => {
    console.log(r.message.field);
});

// Get list
frappe.db.get_list('DocType', {
    filters: { status: 'Active' },
    fields: ['name', 'title'],
    order_by: 'creation desc',
    limit: 10
}).then(data => {
    console.log(data);
});

// Count
frappe.db.count('DocType', { status: 'Open' }).then(count => {
    console.log(count);
});

// Insert
frappe.db.insert({
    doctype: 'ToDo',
    description: 'New Task'
}).then(doc => {
    console.log(doc.name);
});

// Set value
frappe.db.set_value('DocType', 'name', 'field', 'value');
frappe.db.set_value('DocType', 'name', { field1: 'value1', field2: 'value2' });

// Delete
frappe.db.delete_doc('DocType', 'name');

// Exists
frappe.db.exists('DocType', 'name').then(exists => {
    if (exists) {
        // do something
    }
});
```

### Utilities

```javascript
// Format values
frappe.format(value, { fieldtype: 'Currency' });
frappe.format(value, { fieldtype: 'Date' });

// Date operations
frappe.datetime.get_today();
frappe.datetime.now_datetime();
frappe.datetime.add_days(date, days);
frappe.datetime.add_months(date, months);
frappe.datetime.get_diff(date1, date2);
frappe.datetime.str_to_obj(datestring);

// User info
frappe.session.user;
frappe.session.user_fullname;
frappe.user.has_role('Role Name');

// Navigation
frappe.set_route('Form', 'DocType', 'name');
frappe.set_route('List', 'DocType');
frappe.set_route('query-report', 'Report Name');

// URL
frappe.urllib.get_full_url('/api/method/...');
```

### Loading Indicator

```javascript
// Show loading
frappe.dom.freeze('Loading...');

// Hide loading
frappe.dom.unfreeze();
```

---

## Field Types

Available fieldtypes for dialog/form fields:

```javascript
// Basic
'Data'           // Single line text
'Text'           // Multi-line text
'Long Text'      // Large text area
'Text Editor'    // HTML editor
'Markdown Editor'
'Code'           // Code editor
'Password'       // Password field

// Numbers
'Int'            // Integer
'Float'          // Decimal
'Currency'       // Currency with formatting
'Percent'        // Percentage

// Date/Time
'Date'           // Date picker
'Time'           // Time picker
'Datetime'       // Date and time

// Selection
'Select'         // Dropdown
'Link'           // Link to DocType
'Dynamic Link'   // Dynamic DocType link
'Table'          // Child table
'Table MultiSelect'

// Others
'Check'          // Checkbox
'Color'          // Color picker
'Attach'         // File attachment
'Attach Image'   // Image attachment
'Signature'      // Signature field
'Rating'         // Star rating
'Geolocation'    // Map location
'HTML'           // HTML content
'Read Only'      // Display only
'Button'         // Button

// Layout
'Section Break'
'Column Break'
'Tab Break'
```
