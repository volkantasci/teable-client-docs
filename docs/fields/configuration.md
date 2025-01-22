# Field Configuration

This guide covers how to configure fields in Teable tables, including field types, options, and advanced settings.

## Field Types

### Basic Field Types

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Get your table
table = client.get_table("table_id")

# Text Fields
text_field = {
    "name": "Description",
    "type": "text",
    "options": {
        "multiline": True,
        "defaultValue": ""
    }
}

# Number Fields
number_field = {
    "name": "Amount",
    "type": "number",
    "options": {
        "precision": 2,
        "format": "currency",
        "symbol": "$"
    }
}

# Date Fields
date_field = {
    "name": "Due Date",
    "type": "date",
    "options": {
        "format": "YYYY-MM-DD",
        "includeTime": True,
        "timeFormat": "24hour"
    }
}
```

### Selection Fields

```python
# Single Select
single_select = {
    "name": "Status",
    "type": "single_select",
    "options": {
        "choices": [
            {"name": "Active", "color": "green"},
            {"name": "Pending", "color": "yellow"},
            {"name": "Completed", "color": "blue"}
        ]
    }
}

# Multiple Select
multi_select = {
    "name": "Tags",
    "type": "multiple_select",
    "options": {
        "choices": [
            {"name": "Urgent", "color": "red"},
            {"name": "Important", "color": "orange"},
            {"name": "Review", "color": "purple"}
        ]
    }
}
```

### Special Field Types

```python
# Attachment Field
attachment_field = {
    "name": "Files",
    "type": "attachment",
    "options": {
        "maxSize": 10485760,  # 10MB
        "allowedTypes": ["image/*", "application/pdf"]
    }
}

# Link Field
link_field = {
    "name": "Project Manager",
    "type": "link",
    "options": {
        "relationship": "many_to_one",
        "foreignTableId": "employees_table_id",
        "symmetricFieldId": "managed_projects"
    }
}

# Formula Field
formula_field = {
    "name": "Total",
    "type": "formula",
    "options": {
        "expression": "{Quantity} * {Price}",
        "valueType": "number"
    }
}
```

## Field Creation

### Creating Fields

```python
# Create a single field
field = table.create_field(text_field)

# Create multiple fields
fields = [
    text_field,
    number_field,
    date_field,
    single_select
]

for field_config in fields:
    field = table.create_field(field_config)
    print(f"Created field: {field.name}")
```

### Field Options

```python
# Text Field Options
text_options = {
    "multiline": True,        # Allow multiple lines
    "defaultValue": "",       # Default value
    "maxLength": 1000,        # Maximum length
    "unique": False,          # Unique values only
    "required": True         # Required field
}

# Number Field Options
number_options = {
    "precision": 2,           # Decimal places
    "format": "decimal",      # number format
    "minimum": 0,            # Minimum value
    "maximum": 1000,         # Maximum value
    "allowNegative": False   # Allow negative values
}

# Date Field Options
date_options = {
    "includeTime": True,     # Include time component
    "timeFormat": "12hour",  # 12/24 hour format
    "defaultValue": "now",   # Default to current time
    "timezone": "UTC"       # Timezone handling
}
```

## Field Management

### Updating Fields

```python
# Update field configuration
field.update({
    "name": "New Name",
    "description": "Updated description",
    "options": {
        "required": True,
        "unique": True
    }
})
```

### Field Properties

```python
# Access field properties
print(f"Field ID: {field.field_id}")
print(f"Name: {field.name}")
print(f"Type: {field.field_type}")
print(f"Description: {field.description}")
print(f"Required: {field.is_required}")
print(f"Primary: {field.is_primary}")
```

## Advanced Configuration

### Computed Fields

```python
# Lookup Field
lookup_field = {
    "name": "Manager Email",
    "type": "lookup",
    "options": {
        "relationship": "many_to_one",
        "foreignTableId": "employees_table_id",
        "lookupFieldId": "email_field_id"
    }
}

# Rollup Field
rollup_field = {
    "name": "Total Sales",
    "type": "rollup",
    "options": {
        "relationship": "one_to_many",
        "foreignTableId": "sales_table_id",
        "rollupFunction": "sum",
        "field": "amount_field_id"
    }
}
```

### Field Dependencies

```python
# Create dependent fields
status_field = {
    "name": "Status",
    "type": "single_select",
    "options": {
        "choices": [
            {"name": "Active"},
            {"name": "Inactive"}
        ]
    }
}

dependent_field = {
    "name": "Next Action",
    "type": "single_select",
    "options": {
        "choices": [
            {"name": "Follow up"},
            {"name": "Archive"}
        ],
        "dependsOn": {
            "field": "Status",
            "condition": {
                "equals": "Active"
            }
        }
    }
}
```

## Best Practices

1. **Field Design**
   - Use appropriate field types
   - Set meaningful defaults
   - Consider data validation
   - Document field purposes

2. **Performance**
   - Optimize computed fields
   - Use appropriate indexes
   - Consider query performance
   - Monitor field usage

3. **Data Integrity**
   - Validate field dependencies
   - Maintain referential integrity
   - Handle default values
   - Consider data migration

4. **Maintenance**
   - Regular field audits
   - Update field documentation
   - Monitor field usage
   - Clean up unused fields

## Next Steps

- [Field Validation](validation.md)
- [Table Management](../tables/management.md)
- [Record Operations](../records/create.md)
- [Views Configuration](../views/creation.md)
