# Bulk Operations

This guide covers how to perform bulk operations in Teable tables.

## Batch Record Operations

### Batch Creation

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
        "Name": f"User {i}",
        "Email": f"user{i}@example.com",
        "Department": "Engineering"
    }
    for i in range(1, 101)  # Create 100 records
]

# Batch create records
result = client.records.batch_create_records(
    table_id="table_id",
    records=records
)

print(f"Successfully created: {result.success_count}")
print(f"Failed to create: {result.failure_count}")
print(f"Success rate: {result.success_rate}%")

# Process successful records
for record in result.successful:
    print(f"Created record: {record.record_id}")

# Process failed records
for failure in result.failed:
    print(f"Failed record: {failure}")
```

### Batch Updates

```python
# Prepare updates for multiple records
records = [
    {
        "record_id": "rec123",
        "fields": {
            "Status": "Active",
            "Last Updated": "2023-01-15"
        }
    },
    {
        "record_id": "rec456",
        "fields": {
            "Status": "Inactive",
            "Last Updated": "2023-01-15"
        }
    }
]

# Batch update records
result = client.records.batch_update_records(
    table_id="table_id",
    records=records
)

print(f"Successfully updated: {result.success_count}")
print(f"Failed to update: {result.failure_count}")
print(f"Success rate: {result.success_rate}%")
```

### Batch Deletion

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

## Error Handling

### Handling Batch Operations

```python
from teable.exceptions import TeableError, ValidationError

def safe_batch_operation(client, table_id, records):
    """Safely perform batch operations with error handling."""
    try:
        # Get table for validation
        table = client.tables.get(table_id)
        
        # Validate records
        for record in records:
            table.validate_record_fields(record['fields'])
        
        # Perform batch creation
        result = client.records.batch_create_records(
            table_id=table_id,
            records=records
        )
        
        print(f"Successfully created: {result.success_count}")
        print(f"Failed to create: {result.failure_count}")
        print(f"Success rate: {result.success_rate}%")
        
        return result
        
    except ValidationError as e:
        print(f"Validation error: {e}")
        return None
    except TeableError as e:
        print(f"Batch operation failed: {e}")
        return None
```

## Best Practices

1. **Batch Processing**
   - Use batch operations for multiple records
   - Handle partial failures gracefully
   - Monitor API rate limits
   - Consider performance impact

2. **Data Preparation**
   - Clean data before processing
   - Validate data structure
   - Handle missing values
   - Convert data types appropriately

3. **Error Handling**
   - Implement comprehensive error handling
   - Log failures for review
   - Provide meaningful error messages
   - Consider recovery strategies

4. **Performance**
   - Monitor API usage
   - Consider rate limits
   - Track operation progress
   - Handle failures appropriately

## Next Steps

After mastering bulk operations, you can:

- [Configure data validation](../fields/validation.md)
- [Work with views](../views/creation.md)
- [Implement data synchronization](../integration/sync.md)
