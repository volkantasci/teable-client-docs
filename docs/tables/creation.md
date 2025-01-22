# Creating and Configuring Tables

This guide covers how to create and configure tables in Teable using the Teable-Client library.

## Basic Table Creation

### Creating a Simple Table

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Create a basic table
table = client.tables.create(
    space_id="space123",
    name="Employees",
    description="Employee directory and information"
)

print(f"Created table: {table.table_id}")
```

### Creating a Table with Fields

```python
from teable import FieldType

# Define fields for the table
fields = [
    {
        "name": "Full Name",
        "type": FieldType.SINGLE_LINE_TEXT,
        "required": True
    },
    {
        "name": "Department",
        "type": FieldType.SINGLE_SELECT,
        "options": {
            "choices": ["Engineering", "Marketing", "Sales", "HR"]
        }
    },
    {
        "name": "Start Date",
        "type": FieldType.DATE
    },
    {
        "name": "Salary",
        "type": FieldType.NUMBER,
        "options": {
            "precision": 2,
            "format": "currency"
        }
    }
]

# Create table with fields
table = client.tables.create(
    space_id="space123",
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
    print(f"Field ID: {field.field_id}")
    print(f"Type: {field.field_type}")
    print(f"Required: {field.is_required}")
    print("---")
```

## Field Validation

Tables automatically validate data against field configurations:

```python
try:
    # Get table for validation
    table = client.tables.get("table_id")
    
    # Validate record data
    data = {
        "Full Name": "John Doe",
        "Department": "Engineering",
        "Start Date": "2023-01-15",
        "Salary": 75000
    }
    
    # Validate data
    table.validate_record_fields(data)
    
    # Create record
    record = client.records.create(
        table_id=table.table_id,
        fields=data
    )
    print(f"Created record: {record.record_id}")
except ValidationError as e:
    print(f"Validation error: {e}")
```

## Field Types Reference

Here are the available field types in Teable:

```python
from teable import FieldType

# Text Fields
FieldType.SINGLE_LINE_TEXT    # Single line text
FieldType.LONG_TEXT          # Multi-line text

# Number Fields
FieldType.NUMBER            # Numeric values

# Date Fields
FieldType.DATE              # Date and time

# Selection Fields
FieldType.SINGLE_SELECT     # Single choice
FieldType.MULTIPLE_SELECT   # Multiple choices

# Special Fields
FieldType.CHECKBOX          # Boolean values
FieldType.RATING           # Rating values
FieldType.DURATION         # Time duration
FieldType.BUTTON           # Button field

# System Fields
FieldType.CREATED_TIME     # Record creation time
FieldType.LAST_MODIFIED_TIME  # Last modification time
FieldType.CREATED_BY       # Record creator
FieldType.LAST_MODIFIED_BY  # Last modifier
FieldType.AUTO_NUMBER      # Auto-incrementing number
```

## Best Practices

1. **Field Planning**
   - Plan your field structure before creating tables
   - Use appropriate field types for data
   - Set required fields appropriately
   - Consider field relationships

2. **Data Validation**
   - Use field constraints to ensure data quality
   - Set appropriate field formats
   - Document validation requirements
   - Handle validation errors gracefully

3. **Performance**
   - Create tables with all fields initially when possible
   - Use batch operations for multiple records
   - Consider field indexing for large tables
   - Monitor table performance

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
    table = client.tables.create(
        space_id="space123",
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
        "Department": "Engineering"
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
