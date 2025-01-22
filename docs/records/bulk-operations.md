# Bulk Record Operations

This guide covers how to perform bulk operations on records in Teable tables.

## Batch Record Creation

### Creating Multiple Records

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Prepare multiple records
records = [
    {
        "Name": "John Doe",
        "Email": "john@example.com",
        "Department": "Engineering"
    },
    {
        "Name": "Jane Smith",
        "Email": "jane@example.com",
        "Department": "Marketing"
    }
]

# Batch create records
result = client.records.batch_create_records(
    table_id="table_id",
    records=records
)

print(f"Successfully created: {result.success_count}")
print(f"Failed to create: {result.failure_count}")
print(f"Success rate: {result.success_rate}%")
```

### Processing Results

```python
# Process successful records
for record in result.successful:
    print(f"Created record: {record.record_id}")
    print(f"Fields: {record.fields}")

# Process failed records
for failure in result.failed:
    print(f"Failed record: {failure}")
```

## Batch Record Updates

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
print(f"Failed to update: {result.failure_count}")
print(f"Success rate: {result.success_rate}%")
```

## Batch Record Deletion

### Deleting Multiple Records

```python
# Delete multiple records
record_ids = ["rec123", "rec456", "rec789"]

success = client.records.batch_delete_records(
    table_id="table_id",
    record_ids=record_ids
)

if success:
    print(f"Successfully deleted {len(record_ids)} records")
```

## Data Validation

### Pre-validation

```python
def validate_records(client, table_id, records):
    """Validate records before batch operation."""
    # Get table for validation
    table = client.tables.get(table_id)
    
    valid_records = []
    invalid_records = []
    
    for record in records:
        try:
            table.validate_record_fields(record['fields'])
            valid_records.append(record)
        except ValidationError as e:
            invalid_records.append({
                'record': record,
                'error': str(e)
            })
    
    return valid_records, invalid_records

# Usage
records = [
    {
        "fields": {
            "Name": "John Doe",
            "Email": "john@example.com"
        }
    },
    {
        "fields": {
            "Name": "Jane Smith",
            "Email": "invalid-email"  # Invalid email
        }
    }
]

valid, invalid = validate_records(client, "table_id", records)

# Create valid records
if valid:
    result = client.records.batch_create_records(
        table_id="table_id",
        records=valid
    )

# Log invalid records
if invalid:
    print("Invalid records:")
    for item in invalid:
        print(f"Record: {item['record']}")
        print(f"Error: {item['error']}")
```

## Best Practices

1. **Batch Size**
   - Use reasonable batch sizes
   - Consider API rate limits
   - Monitor performance
   - Handle failures appropriately

2. **Error Handling**
   - Validate data before operations
   - Handle validation errors
   - Log failed operations
   - Provide meaningful error messages

3. **Performance**
   - Use batch operations for multiple records
   - Monitor operation timing
   - Consider rate limits
   - Optimize data preparation

4. **Data Integrity**
   - Validate data before operations
   - Maintain referential integrity
   - Handle duplicates appropriately
   - Back up important data

## Next Steps

- [Query Records](read.md)
- [Update Records](update.md)
- [Delete Records](delete.md)
- [Table Operations](../tables/creation.md)
