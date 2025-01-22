# Bulk Record Operations

This guide covers how to perform bulk operations on records in Teable tables, including batch creation, updates, and deletions.

## Batch Record Creation

### Creating Multiple Records

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Get your table
table = client.get_table("table_id")

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
result = table.batch_create_records(
    records=records,
    field_key_type="name",  # Use field names instead of IDs
    typecast=True          # Automatically convert data types
)

print(f"Successfully created {result.success_count} records")
print(f"Failed to create {result.failure_count} records")
```

### Handling Batch Results

```python
# Process batch creation results
for record in result.successful:
    print(f"Created record: {record.record_id}")
    print(f"Fields: {record.fields}")

for error in result.failed:
    print(f"Failed record: {error['record']}")
    print(f"Error: {error['error']}")
```

## Batch Record Updates

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
result = table.batch_update_records(
    updates=updates,
    field_key_type="name",
    typecast=True
)

print(f"Updated {len(result.successful)} records")
```

### Ordering Updates

```python
# Update records with specific order
result = table.batch_update_records(
    updates=updates,
    order={
        "viewId": "view123",
        "anchorId": "rec789",
        "position": "after"
    }
)
```

## Batch Record Deletion

### Deleting Multiple Records

```python
# Delete multiple records
record_ids = ["rec123", "rec456", "rec789"]
success = table.batch_delete_records(record_ids)

if success:
    print(f"Successfully deleted {len(record_ids)} records")
```

## Performance Optimization

### Chunked Operations

```python
def process_in_chunks(records, chunk_size=1000):
    """Process records in chunks to manage memory and API limits."""
    for i in range(0, len(records), chunk_size):
        chunk = records[i:i + chunk_size]
        yield chunk

# Example usage with large dataset
large_dataset = [
    {"Name": f"User {i}", "Email": f"user{i}@example.com"}
    for i in range(10000)
]

total_success = 0
total_failure = 0

for chunk in process_in_chunks(large_dataset):
    result = table.batch_create_records(chunk)
    total_success += result.success_count
    total_failure += result.failure_count
    print(f"Processed chunk: {result.success_count} successful, {result.failure_count} failed")

print(f"Total: {total_success} successful, {total_failure} failed")
```

### Error Recovery

```python
def batch_operation_with_retry(table, records, max_retries=3):
    """Perform batch operation with retry for failed records."""
    all_results = {
        'successful': [],
        'failed': []
    }
    
    remaining = records
    retry_count = 0
    
    while remaining and retry_count < max_retries:
        result = table.batch_create_records(remaining)
        
        # Add successful records to results
        all_results['successful'].extend(result.successful)
        
        # Prepare failed records for retry
        if result.failed:
            remaining = [
                failed['record']
                for failed in result.failed
            ]
            retry_count += 1
            print(f"Retry {retry_count}: {len(remaining)} records")
        else:
            remaining = []
    
    # Add any remaining failed records
    if remaining:
        all_results['failed'].extend(remaining)
    
    return all_results
```

## Data Validation

### Pre-validation

```python
def validate_records(table, records):
    """Validate records before batch operation."""
    valid_records = []
    invalid_records = []
    
    for record in records:
        try:
            table.validate_record_fields(record)
            valid_records.append(record)
        except ValidationError as e:
            invalid_records.append({
                'record': record,
                'error': str(e)
            })
    
    return valid_records, invalid_records

# Usage
valid, invalid = validate_records(table, records)
if valid:
    result = table.batch_create_records(valid)
if invalid:
    print("Invalid records:")
    for item in invalid:
        print(f"Record: {item['record']}")
        print(f"Error: {item['error']}")
```

## Best Practices

1. **Batch Size**
   - Use appropriate chunk sizes (500-1000 records)
   - Monitor memory usage
   - Consider API rate limits
   - Balance performance and reliability

2. **Error Handling**
   - Implement retry mechanisms
   - Log failed operations
   - Provide meaningful error messages
   - Consider partial success scenarios

3. **Performance**
   - Use chunked operations
   - Implement parallel processing when appropriate
   - Monitor operation timing
   - Optimize data preparation

4. **Data Integrity**
   - Validate data before operations
   - Maintain referential integrity
   - Handle duplicates appropriately
   - Consider transaction boundaries

## Next Steps

- [Query Records](read.md)
- [Update Records](update.md)
- [Delete Records](delete.md)
- [Table Operations](../tables/creation.md)
