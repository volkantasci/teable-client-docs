# Creating Records

This guide covers how to create and manage individual records in Teable tables using the Teable-Client library. Learn about record creation, field value management, and record properties.

## Basic Record Creation

### Creating a Simple Record

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Get your table
table = client.get_table("table_id")

# Create a record
record = table.create_record({
    "Name": "John Doe",
    "Email": "john@example.com",
    "Department": "Engineering",
    "Start Date": "2023-01-15"
})

print(f"Created record ID: {record.record_id}")
```

### Record Properties

Access various record properties:

```python
# Access record information
print(f"Record ID: {record.record_id}")
print(f"Primary Field Value: {record.name}")
print(f"Auto Number: {record.auto_number}")
print(f"Created Time: {record.created_time}")
print(f"Last Modified Time: {record.last_modified_time}")
print(f"Created By: {record.created_by}")
print(f"Last Modified By: {record.last_modified_by}")
```

## Working with Field Values

### Getting Field Values

```python
# Get field value by name
name = record.get_field_value("Name")
email = record.get_field_value("Email")

# Get field value using Field object
field = table.get_field("field_id")
value = record.get_field_value(field)

# Access raw field values
fields = record.fields
print(f"All field values: {fields}")
```

### Setting Field Values

```python
# Set field value by name
record.set_field_value("Status", "Active")

# Set field value with type casting
record.set_field_value(
    "Start Date",
    "2023-01-15",
    typecast=True  # Automatically convert string to date
)

# Set field value using Field object
field = table.get_field("field_id")
record.set_field_value(field, "New Value")
```

## Record Creation with Validation

```python
from teable.exceptions import ValidationError

try:
    # Create record with validation
    record = table.create_record({
        "Name": "Jane Smith",
        "Email": "jane@example.com",
        "Department": "Marketing",
        "Salary": 75000,
        "Is Active": True
    })
except ValidationError as e:
    print(f"Validation error: {e}")
```

## Advanced Record Creation

### Creating Records with Special Fields

```python
# Create record with different field types
record = table.create_record({
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
    "Skills": ["Python", "JavaScript", "Docker"],
    
    # Link fields (references to other records)
    "Manager": ["rec123"],
    "Projects": ["rec456", "rec789"]
})
```

### Creating Records with Attachments

```python
# Create record with file attachments
record = table.create_record({
    "Name": "Project Documentation",
    "Files": [
        {
            "filename": "document.pdf",
            "url": "https://example.com/files/document.pdf",
            "size": 1024,
            "type": "application/pdf"
        }
    ]
})
```

## Converting Records

### Record to Dictionary

```python
# Convert record to dictionary format
record_dict = record.to_dict()

print("Record as dictionary:")
print(f"ID: {record_dict['id']}")
print(f"Fields: {record_dict['fields']}")
```

## Error Handling

```python
from teable.exceptions import TeableError, ValidationError

def safe_create_record(table, data):
    """Safely create a record with error handling."""
    try:
        # Validate field values first
        table.validate_record_fields(data)
        
        # Create the record
        record = table.create_record(data)
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
   - Validate linked records

3. **Error Handling**
   - Implement comprehensive error handling
   - Provide meaningful error messages
   - Log validation failures
   - Consider recovery strategies

4. **Performance**
   - Use batch operations for multiple records
   - Minimize API calls
   - Handle large attachments appropriately
   - Consider caching strategies

## Helper Functions

```python
def create_record_with_defaults(table, data, defaults=None):
    """Create a record with default values."""
    # Merge defaults with provided data
    record_data = {**(defaults or {}), **data}
    
    try:
        record = table.create_record(record_data)
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
    table,
    {"Name": "New Employee"},
    defaults=defaults
)
```

## Next Steps

After mastering record creation, you can:

- [Read and query records](read.md)
- [Update existing records](update.md)
- [Delete records](delete.md)
- [Work with bulk operations](../tables/bulk-operations.md)
