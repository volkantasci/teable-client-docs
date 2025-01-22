# Creating and Configuring Tables

This guide covers how to create and configure tables in Teable using the Teable-Client library. Tables are the core data structures that store and organize your information.

## Basic Table Creation

### Creating a Simple Table

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Get a base
base = client.get_base("base123")

# Create a basic table
table = base.create_table(
    name="Employees",
    description="Employee directory and information"
)

print(f"Created table: {table.name} (ID: {table.table_id})")
```

### Creating a Table with Fields

```python
# Define fields for the table
fields = [
    {
        "name": "Full Name",
        "type": "text",
        "required": True,
        "description": "Employee's full name"
    },
    {
        "name": "Email",
        "type": "email",
        "required": True,
        "unique": True
    },
    {
        "name": "Department",
        "type": "single_select",
        "options": {
            "choices": [
                {"name": "Engineering"},
                {"name": "Marketing"},
                {"name": "Sales"},
                {"name": "HR"}
            ]
        }
    },
    {
        "name": "Start Date",
        "type": "date",
        "description": "Employee's start date"
    },
    {
        "name": "Salary",
        "type": "number",
        "description": "Annual salary",
        "options": {
            "format": "currency",
            "precision": 2
        }
    }
]

# Create table with fields
table = base.create_table(
    name="Employees",
    description="Employee directory and information",
    fields=fields
)
```

## Field Management

### Getting Table Fields

```python
# Access all fields in a table
fields = table.fields

for field in fields:
    print(f"Field: {field.name}")
    print(f"Type: {field.field_type}")
    print(f"Required: {field.is_required}")
    print(f"Description: {field.description or 'No description'}")
    print("---")
```

### Getting a Specific Field

```python
try:
    # Get field by ID
    field = table.get_field("field123")
    print(f"Field name: {field.name}")
    print(f"Field type: {field.field_type}")
except ResourceNotFoundError:
    print("Field not found")
```

## Field Validation

Tables automatically validate data against field configurations:

```python
try:
    # Create a record with field validation
    record = table.create_record({
        "Full Name": "John Doe",
        "Email": "john@example.com",
        "Department": "Engineering",
        "Start Date": "2023-01-15",
        "Salary": 75000
    })
    print(f"Created record: {record.id}")
except ValidationError as e:
    print(f"Validation error: {e}")
```

## Table Properties

### Accessing Table Information

```python
# Get table properties
print(f"Table ID: {table.table_id}")
print(f"Name: {table.name}")
print(f"Description: {table.description}")
```

### Clearing Cache

Tables cache field and view information for performance. You can clear this cache when needed:

```python
# Clear cached data
table.clear_cache()

# Access fresh data
fields = table.fields  # Will fetch fresh field data
views = table.views    # Will fetch fresh view data
```

## Common Field Types

Here are examples of different field types you can use when creating tables:

```python
fields = [
    # Text Fields
    {
        "name": "Simple Text",
        "type": "text"
    },
    {
        "name": "Long Text",
        "type": "text",
        "options": {
            "multiline": True
        }
    },
    
    # Numeric Fields
    {
        "name": "Number",
        "type": "number",
        "options": {
            "precision": 2
        }
    },
    {
        "name": "Currency",
        "type": "number",
        "options": {
            "format": "currency",
            "precision": 2,
            "symbol": "$"
        }
    },
    
    # Selection Fields
    {
        "name": "Single Select",
        "type": "single_select",
        "options": {
            "choices": [
                {"name": "Option 1"},
                {"name": "Option 2"}
            ]
        }
    },
    {
        "name": "Multi Select",
        "type": "multiple_select",
        "options": {
            "choices": [
                {"name": "Tag 1"},
                {"name": "Tag 2"}
            ]
        }
    },
    
    # Date and Time Fields
    {
        "name": "Date",
        "type": "date"
    },
    {
        "name": "Date Time",
        "type": "date",
        "options": {
            "include_time": True
        }
    },
    
    # Special Fields
    {
        "name": "Email",
        "type": "email"
    },
    {
        "name": "URL",
        "type": "url"
    },
    {
        "name": "Attachment",
        "type": "attachment"
    },
    {
        "name": "Checkbox",
        "type": "checkbox"
    }
]
```

## Best Practices

1. **Field Planning**
   - Plan your field structure before creating tables
   - Use appropriate field types for data
   - Set required fields appropriately
   - Add field descriptions for clarity

2. **Data Validation**
   - Use field constraints to ensure data quality
   - Implement unique constraints where needed
   - Set appropriate field formats
   - Document validation requirements

3. **Performance**
   - Create tables with all fields initially when possible
   - Use batch operations for multiple records
   - Clear cache only when necessary
   - Consider field indexing for large tables

4. **Documentation**
   - Add clear table descriptions
   - Document field purposes
   - Maintain field naming conventions
   - Keep field options documented

## Error Handling

```python
from teable.exceptions import TeableError, ValidationError, ResourceNotFoundError

try:
    # Create table with fields
    table = base.create_table(
        name="Employees",
        fields=fields
    )
except ValidationError as e:
    print(f"Invalid field configuration: {e}")
except TeableError as e:
    print(f"Error creating table: {e}")

# Field validation
try:
    table.validate_record_fields({
        "Full Name": "John Doe",
        "Email": "invalid-email"
    })
except ValidationError as e:
    print(f"Field validation error: {e}")
```

## Next Steps

After creating your table, you can:

- [Configure table connections](connection.md)
- [Manage records](../records/create.md)
- [Set up views](../views/creation.md)
- [Configure bulk operations](bulk-operations.md)
