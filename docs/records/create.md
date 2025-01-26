# Creating Records

This guide covers how to create records in Teable tables using the Teable-Client library.

## Basic Record Creation

### Creating a Simple Record

```python
from teable import TeableClient, Record

# Initialize the client
client = TeableClient()

# Create a record and convert API response to Record object
record = Record.from_api_response(
    client.records.create_record(
        table_id="table_id",
        fields={
            "Name": "John Doe",
            "Email": "john@example.com",
            "Department": "Engineering",
            "Start Date": "2023-01-15"
        }
    )
)

print(f"Created record ID: {record.record_id}")
```

### Record Properties and Field Access

```python
# Access record information
print(f"Record ID: {record.record_id}")

# Access all field values
fields = record.fields
print(f"All field values: {fields}")

# Get specific field value
name = record.get_field_value("Name")
print(f"Name: {name}")

# Update specific field value
record.set_field_value("Department", "Sales")
print(f"Updated department: {record.get_field_value('Department')}")

# Handle non-existent fields
try:
    value = record.get_field_value("Non-existent Field")
except KeyError as e:
    print(f"Field not found: {e}")
```

## Record Creation with Validation

### Required Fields

```python
from teable.exceptions import ValidationError

# Create a table with required field
table = client.tables.create_table(
    base_id="base_id",
    name="Validation Test Table",
    fields=[
        {
            "name": "Required Field",
            "type": "singleLineText",
            "required": True
        }
    ]
)

# Test missing required field
try:
    client.records.create_record(table.table_id, {})  # Empty data
except ValidationError as e:
    print(f"Validation error: {e}")  # Will fail due to missing required field

# Correct way with required field
record = client.records.create_record(
    table.table_id,
    {"Required Field": "Value"}
)
```

### Field Validation

```python
from teable.exceptions import ValidationError, APIError

try:
    # Get table for validation
    table = client.tables.get("table_id")
    
    # Prepare record data
    data = {
        "Name": "Jane Smith",
        "Email": "jane@example.com",
        "Department": "Marketing",
        "Salary": 75000,
        "Is Active": True
    }
    
    # Validate data first
    table.validate_record_fields(data)
    
    # Create record
    record = Record.from_api_response(
        client.records.create_record(
            table_id=table.table_id,
            fields=data
        )
    )
except ValidationError as e:
    print(f"Validation error: {e}")
except APIError as e:
    print(f"API error (e.g., invalid field name): {e}")
```

### Batch Creation Validation

```python
from teable.exceptions import ValidationError

# Validate batch size
try:
    # Empty batch
    client.records.batch_create_records(table_id, [])  # Will raise ValidationError
except ValidationError as e:
    print(f"Empty batch error: {e}")

try:
    # Too many records (over 2000)
    too_many_records = [{"field": f"value{i}"} for i in range(2001)]
    client.records.batch_create_records(table_id, too_many_records)  # Will raise ValidationError
except ValidationError as e:
    print(f"Batch size error: {e}")

# Correct batch size
valid_batch = [{"field": f"value{i}"} for i in range(100)]
batch_result = client.records.batch_create_records(table_id, valid_batch)
print(f"Successfully created {batch_result.success_count} records")
```

## Advanced Record Creation

### Creating Records with Different Field Types

```python
# Create record with different field types
record = client.records.create(
    table_id="table_id",
    fields={
        # Text fields
        "Name": "Alice Johnson",
        "Description": "Senior Developer",
        
        # Numeric fields
        "Salary": 85000,
        "Years Experience": 5,
        
        # Date fields
        "Start Date": "2023-01-15",
        "Last Review": "2023-06-30T14:30:00Z",
        
        # Boolean fields
        "Is Manager": True,
        "Remote Worker": False,
        
        # Select fields
        "Department": "Engineering",
        "Skills": ["Python", "JavaScript", "Docker"]
    }
)
```

## Error Handling

```python
from teable.exceptions import TeableError, ValidationError

def safe_create_record(client, table_id, data):
    """Safely create a record with error handling."""
    try:
        # Get table for validation
        table = client.tables.get(table_id)
        
        # Validate field values first
        table.validate_record_fields(data)
        
        # Create the record
        record = client.records.create(
            table_id=table_id,
            fields=data
        )
        return record
        
    except ValidationError as e:
        print(f"Field validation error: {e}")
        return None
    except TeableError as e:
        print(f"Error creating record: {e}")
        return None
```

## Best Practices

1. **Data Validation**
   - Validate data before creation
   - Use appropriate data types
   - Handle validation errors gracefully
   - Document validation requirements

2. **Field Management**
   - Use consistent field naming
   - Handle required fields appropriately
   - Consider field dependencies
   - Use correct field types

3. **Error Handling**
   - Implement comprehensive error handling
   - Provide meaningful error messages
   - Log validation failures
   - Consider recovery strategies

4. **Performance**
   - Use batch operations for multiple records
   - Minimize API calls
   - Consider caching strategies

## Helper Functions

```python
from datetime import datetime

def create_record_with_defaults(client, table_id, data, defaults=None):
    """Create a record with default values."""
    # Merge defaults with provided data
    record_data = {**(defaults or {}), **data}
    
    try:
        record = client.records.create(
            table_id=table_id,
            fields=record_data
        )
        return record
    except ValidationError as e:
        print(f"Validation error: {e}")
        return None

# Example usage
defaults = {
    "Status": "Active",
    "Created Date": datetime.now().isoformat(),
    "Department": "Unassigned"
}

record = create_record_with_defaults(
    client,
    "table_id",
    {"Name": "New Employee"},
    defaults=defaults
)
```

## Next Steps

After mastering record creation, you can:

- [Read and query records](read.md)
- [Update existing records](update.md)
- [Delete records](delete.md)
- [Work with bulk operations](bulk-operations.md)
