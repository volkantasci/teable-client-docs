# Bulk Operations

This guide covers how to perform bulk operations in Teable tables, including batch record operations, imports, and exports. Learn how to efficiently manage large datasets and perform operations on multiple records simultaneously.

## Batch Record Operations

### Batch Creation

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
        "Name": f"User {i}",
        "Email": f"user{i}@example.com",
        "Department": "Engineering"
    }
    for i in range(1, 101)  # Create 100 records
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

### Batch Updates

```python
# Prepare updates for multiple records
updates = [
    {
        "recordId": "rec123",
        "fields": {
            "Status": "Active",
            "Last Updated": "2023-01-15"
        }
    },
    {
        "recordId": "rec456",
        "fields": {
            "Status": "Inactive",
            "Last Updated": "2023-01-15"
        }
    }
]

# Batch update records
updated_records = table.batch_update_records(
    updates=updates,
    field_key_type="name",
    typecast=True
)

print(f"Updated {len(updated_records)} records")
```

### Batch Deletion

```python
# Delete multiple records
record_ids = ["rec123", "rec456", "rec789"]
success = table.batch_delete_records(record_ids)

if success:
    print(f"Successfully deleted {len(record_ids)} records")
```

## Handling Large Datasets

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

### Error Handling in Batch Operations

```python
from teable.exceptions import TeableError, ValidationError

def safe_batch_operation(table, records):
    """Safely perform batch operations with error handling."""
    results = {
        'successful': [],
        'failed': []
    }
    
    try:
        # Attempt batch creation
        result = table.batch_create_records(records)
        
        # Process results
        results['successful'] = result.successful_records
        
        # Handle partial failures
        for error in result.errors:
            results['failed'].append({
                'record': records[error.index],
                'error': error.message
            })
            
    except TeableError as e:
        print(f"Batch operation failed: {e}")
        results['failed'].extend([
            {'record': record, 'error': str(e)}
            for record in records
        ])
        
    return results
```

## Data Import and Export

### CSV Import

```python
def import_csv_data(table, csv_file_path, chunk_size=1000):
    """Import data from a CSV file."""
    import csv
    
    records = []
    with open(csv_file_path, 'r') as file:
        reader = csv.DictReader(file)
        
        for row in reader:
            # Clean and prepare the data
            record = {
                k: v.strip() if isinstance(v, str) else v
                for k, v in row.items()
                if v
            }
            records.append(record)
            
            # Process in chunks
            if len(records) >= chunk_size:
                result = table.batch_create_records(records)
                print(f"Imported {result.success_count} records")
                records = []
    
    # Process remaining records
    if records:
        result = table.batch_create_records(records)
        print(f"Imported {result.success_count} records")
```

### Data Export

```python
def export_table_data(table, output_file):
    """Export table data to a file."""
    # Get all records
    records = table.get_records()
    
    # Write to file
    with open(output_file, 'w') as file:
        if records:
            # Get field names from first record
            fields = records[0].fields.keys()
            
            # Write header
            file.write(','.join(fields) + '\n')
            
            # Write records
            for record in records:
                values = [str(record.fields.get(field, '')) for field in fields]
                file.write(','.join(values) + '\n')
```

## Performance Optimization

### Efficient Batch Processing

```python
def optimize_batch_operation(records, validate=True):
    """Optimize records for batch processing."""
    optimized = []
    
    for record in records:
        # Remove null values
        cleaned = {k: v for k, v in record.items() if v is not None}
        
        # Convert data types if needed
        if 'date' in cleaned:
            cleaned['date'] = cleaned['date'].isoformat()
            
        optimized.append(cleaned)
        
    return optimized

# Example usage
raw_records = [
    {"name": "User 1", "age": 25, "notes": None},
    {"name": "User 2", "age": 30, "notes": ""}
]

optimized_records = optimize_batch_operation(raw_records)
result = table.batch_create_records(optimized_records)
```

## Best Practices

1. **Batch Processing**
   - Use appropriate chunk sizes
   - Handle partial failures gracefully
   - Monitor memory usage
   - Implement retry logic

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
   - Optimize batch sizes
   - Use efficient data structures
   - Monitor API rate limits
   - Implement progress tracking

## Error Recovery

```python
def recover_failed_operations(failed_records, retry_count=3):
    """Attempt to recover failed batch operations."""
    for attempt in range(retry_count):
        if not failed_records:
            break
            
        print(f"Retry attempt {attempt + 1}")
        retry_records = failed_records
        failed_records = []
        
        try:
            result = table.batch_create_records(retry_records)
            failed_records.extend([
                retry_records[error.index]
                for error in result.errors
            ])
        except TeableError as e:
            print(f"Retry failed: {e}")
            failed_records.extend(retry_records)
            
    return failed_records
```

## Next Steps

After mastering bulk operations, you can:

- [Set up automated workflows](../automation/workflows.md)
- [Configure data validation](../fields/validation.md)
- [Work with views](../views/creation.md)
- [Implement data synchronization](../integration/sync.md)
