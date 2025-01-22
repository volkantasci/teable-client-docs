# Creating Records

This guide covers how to create records in Teable tables using the Teable-Client library.

## Basic Record Creation

### Creating a Simple Record

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Create a record
record = client.records.create(
    table_id="table_id",
    fields={
        "Name": "John Doe",
        "Email": "john@example.com",
        "Department": "Engineering",
        "Start Date": "2023-01-15"
    }
)

print(f"Created record ID: {record.record_id}")
```

### Record Properties

Access record properties:

```python
# Access record information
print(f"Record ID: {record.record_id}")

# Access field values
fields = record.fields
print(f"All field values: {fields}")
```

## Record Creation with Validation

```python
from teable.exceptions import ValidationError

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
    record = client.records.create(
        table_id=table.table_id,
        fields=data
    )
except ValidationError as e:
    print(f"Validation error: {e}")
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
