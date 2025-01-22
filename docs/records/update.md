# Updating Records

This guide covers how to update existing records in Teable tables using the Teable-Client library.

## Basic Record Updates

### Updating a Single Record

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Update a record
updated_record = client.records.update(
    table_id="table_id",
    record_id="rec123",
    fields={
        "Name": "John Smith",
        "Department": "Marketing",
        "Status": "Active"
    }
)

print(f"Updated record ID: {updated_record.record_id}")
```

### Updating Different Field Types

```python
# Update different field types
updated_record = client.records.update(
    table_id="table_id",
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
        "Skills": ["Python", "JavaScript", "Docker"]
    }
)
```

## Batch Updates

### Updating Multiple Records

```python
# Prepare updates
records = [
    {
        "record_id": "rec123",
        "fields": {
            "Status": "Active",
            "Department": "Engineering"
        }
    },
    {
        "record_id": "rec456",
        "fields": {
            "Status": "Inactive",
            "Department": "Marketing"
        }
    }
]

# Perform batch update
result = client.records.batch_update_records(
    table_id="table_id",
    records=records
)

print(f"Successfully updated: {result.success_count}")
print(f"Failed updates: {result.failure_count}")
print(f"Success rate: {result.success_rate}%")

# Process results
for record in result.successful:
    print(f"Updated record: {record.record_id}")

for failure in result.failed:
    print(f"Failed update: {failure}")
```

## Validation and Error Handling

### Pre-validation

```python
from teable.exceptions import ValidationError

try:
    # Get table for validation
    table = client.tables.get("table_id")
    
    # Prepare update data
    fields = {
        "Name": "Test User",
        "Email": "test@example.com",
        "Department": "Engineering"
    }
    
    # Validate fields before update
    table.validate_record_fields(fields)
    
    # Perform update if validation passes
    updated_record = client.records.update(
        table_id=table.table_id,
        record_id="rec123",
        fields=fields
    )
except ValidationError as e:
    print(f"Validation error: {e}")
```

### Safe Updates

```python
from teable.exceptions import TeableError, ResourceNotFoundError

def safe_update_record(client, table_id, record_id, fields):
    """Safely update a record with error handling."""
    try:
        # Get table for validation
        table = client.tables.get(table_id)
        
        # Validate fields first
        table.validate_record_fields(fields)
        
        # Perform update
        record = client.records.update(
            table_id=table_id,
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
def update_if_changed(client, table_id, record_id, new_fields):
    """Update record only if fields have changed."""
    try:
        # Get current record
        current = client.records.get(
            table_id=table_id,
            record_id=record_id
        )
        
        # Check for changes
        changes = {}
        for field, value in new_fields.items():
            if current.fields.get(field) != value:
                changes[field] = value
        
        # Update if there are changes
        if changes:
            return client.records.update(
                table_id=table_id,
                record_id=record_id,
                fields=changes
            )
        return current
        
    except TeableError as e:
        print(f"Error: {e}")
        return None
```

## Best Practices

1. **Data Validation**
   - Validate data before updates
   - Use appropriate data types
   - Handle validation errors gracefully
   - Document validation requirements

2. **Performance**
   - Use batch updates for multiple records
   - Monitor update performance
   - Consider rate limits
   - Handle failures appropriately

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
- [Work with bulk operations](bulk-operations.md)
- [Set up views](../views/creation.md)
