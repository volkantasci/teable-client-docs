# Updating Records

This guide covers how to update existing records in Teable tables using the Teable-Client library. Learn about updating individual records, batch updates, and handling field modifications.

## Basic Record Updates

### Updating a Single Record

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Get your table
table = client.get_table("table_id")

# Update a record
updated_record = table.update_record(
    record_id="rec123",
    fields={
        "Name": "John Smith",
        "Department": "Marketing",
        "Status": "Active"
    }
)

print(f"Updated record ID: {updated_record.record_id}")
```

### Type Casting

```python
# Update with automatic type conversion
updated_record = table.update_record(
    record_id="rec123",
    fields={
        "Start Date": "2023-01-15",  # String to date
        "Salary": "75000",           # String to number
        "Is Active": "true"          # String to boolean
    },
    typecast=True
)
```

## Field-Specific Updates

### Updating Individual Fields

```python
# Get a record
record = table.get_record("rec123")

# Update specific fields
record.set_field_value("Status", "Inactive")
record.set_field_value("Last Modified", "2023-06-30")

# Save changes
updated_record = table.update_record(
    record_id=record.record_id,
    fields=record.fields
)
```

### Updating Special Field Types

```python
# Update different field types
updated_record = table.update_record(
    record_id="rec123",
    fields={
        # Text fields
        "Name": "Alice Johnson",
        "Description": "Senior Developer",
        
        # Numeric fields
        "Salary": 85000,
        "Years Experience": 5,
        
        # Date fields
        "Last Review": "2023-06-30T14:30:00Z",
        
        # Boolean fields
        "Is Manager": True,
        
        # Select fields
        "Department": "Engineering",
        "Skills": ["Python", "JavaScript", "Docker"],
        
        # Link fields
        "Manager": ["rec456"],
        "Projects": ["rec789", "rec012"]
    }
)
```

## Batch Updates

### Updating Multiple Records

```python
# Prepare updates
updates = [
    {
        "recordId": "rec123",
        "fields": {
            "Status": "Active",
            "Department": "Engineering"
        }
    },
    {
        "recordId": "rec456",
        "fields": {
            "Status": "Inactive",
            "Department": "Marketing"
        }
    }
]

# Perform batch update
updated_records = table.batch_update_records(
    updates=updates,
    field_key_type="name",
    typecast=True
)

print(f"Updated {len(updated_records)} records")
```

### Ordering Updates

```python
# Update records with specific order
updated_records = table.batch_update_records(
    updates=updates,
    order={
        "viewId": "view123",
        "anchorId": "rec789",
        "position": "after"
    }
)
```

## Validation and Error Handling

### Pre-validation

```python
from teable.exceptions import ValidationError

try:
    # Validate fields before update
    table.validate_record_fields({
        "Name": "Test User",
        "Email": "test@example.com",
        "Department": "Engineering"
    })
    
    # Perform update if validation passes
    updated_record = table.update_record(
        record_id="rec123",
        fields=fields
    )
except ValidationError as e:
    print(f"Validation error: {e}")
```

### Safe Updates

```python
from teable.exceptions import TeableError, ResourceNotFoundError

def safe_update_record(table, record_id, fields):
    """Safely update a record with error handling."""
    try:
        # Validate fields first
        table.validate_record_fields(fields)
        
        # Perform update
        record = table.update_record(
            record_id=record_id,
            fields=fields
        )
        return record
        
    except ValidationError as e:
        print(f"Field validation error: {e}")
        return None
    except ResourceNotFoundError:
        print(f"Record {record_id} not found")
        return None
    except TeableError as e:
        print(f"Error updating record: {e}")
        return None
```

## Advanced Updates

### Conditional Updates

```python
def update_if_changed(table, record_id, new_fields):
    """Update record only if fields have changed."""
    # Get current record
    current = table.get_record(record_id)
    
    # Check for changes
    changes = {}
    for field, value in new_fields.items():
        if current.fields.get(field) != value:
            changes[field] = value
    
    # Update if there are changes
    if changes:
        return table.update_record(
            record_id=record_id,
            fields=changes
        )
    return current
```

### Batch Update with Retry

```python
def batch_update_with_retry(table, updates, max_retries=3):
    """Perform batch update with retry logic."""
    failed_updates = updates
    retry_count = 0
    
    while failed_updates and retry_count < max_retries:
        try:
            result = table.batch_update_records(failed_updates)
            
            # Check for partial failures
            failed_updates = [
                updates[error.index]
                for error in result.errors
            ]
            
            if failed_updates:
                retry_count += 1
                print(f"Retry {retry_count}: {len(failed_updates)} updates failed")
            
        except TeableError as e:
            print(f"Batch update error: {e}")
            retry_count += 1
    
    return len(updates) - len(failed_updates)
```

## Best Practices

1. **Data Validation**
   - Validate data before updates
   - Use appropriate data types
   - Handle validation errors gracefully
   - Document validation requirements

2. **Performance**
   - Use batch updates for multiple records
   - Implement retry logic for failures
   - Monitor update performance
   - Consider rate limits

3. **Error Handling**
   - Implement comprehensive error handling
   - Log update failures
   - Provide meaningful error messages
   - Consider recovery strategies

4. **Data Integrity**
   - Validate data relationships
   - Handle linked records appropriately
   - Consider cascading updates
   - Maintain data consistency

## Next Steps

After mastering record updates, you can:

- [Delete records](delete.md)
- [Query records](read.md)
- [Work with bulk operations](../tables/bulk-operations.md)
- [Set up views](../views/creation.md)
