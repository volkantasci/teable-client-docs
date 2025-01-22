# Deleting Records

This guide covers how to delete records from Teable tables using the Teable-Client library. Learn about single record deletion, batch deletions, and handling the deletion process safely.

## Basic Record Deletion

### Deleting a Single Record

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Get your table
table = client.get_table("table_id")

# Delete a record
try:
    success = table.delete_record("rec123")
    if success:
        print("Record successfully deleted")
except TeableError as e:
    print(f"Error deleting record: {e}")
```

### Safe Record Deletion

```python
from teable.exceptions import TeableError, ResourceNotFoundError

def safe_delete_record(table, record_id):
    """Safely delete a record with error handling."""
    try:
        # Verify record exists
        record = table.get_record(record_id)
        
        # Delete record
        success = table.delete_record(record_id)
        return success
        
    except ResourceNotFoundError:
        print(f"Record {record_id} not found")
        return False
    except TeableError as e:
        print(f"Error deleting record: {e}")
        return False
```

## Batch Deletions

### Deleting Multiple Records

```python
# Delete multiple records
record_ids = ["rec123", "rec456", "rec789"]

try:
    success = table.batch_delete_records(record_ids)
    if success:
        print(f"Successfully deleted {len(record_ids)} records")
except TeableError as e:
    print(f"Error during batch deletion: {e}")
```

### Batch Deletion with Validation

```python
def validate_and_delete_records(table, record_ids):
    """Validate records before deletion."""
    # Verify all records exist
    existing_records = []
    missing_records = []
    
    for record_id in record_ids:
        try:
            table.get_record(record_id)
            existing_records.append(record_id)
        except ResourceNotFoundError:
            missing_records.append(record_id)
    
    if missing_records:
        print(f"Warning: Records not found: {missing_records}")
    
    if existing_records:
        success = table.batch_delete_records(existing_records)
        return success, len(existing_records)
    
    return False, 0
```

## Advanced Deletion

### Conditional Deletion

```python
def delete_records_by_condition(table, condition):
    """Delete records that match a condition."""
    # Get records matching condition
    records = table.get_records(filter=condition)
    
    if not records:
        print("No records match the condition")
        return 0
    
    # Delete matching records
    record_ids = [record.record_id for record in records]
    success = table.batch_delete_records(record_ids)
    
    if success:
        return len(record_ids)
    return 0

# Example usage
deleted_count = delete_records_by_condition(
    table,
    {
        "field": "Status",
        "operator": "=",
        "value": "Inactive"
    }
)
print(f"Deleted {deleted_count} inactive records")
```

### Chunked Deletion

```python
def delete_records_in_chunks(table, record_ids, chunk_size=1000):
    """Delete large number of records in chunks."""
    total_deleted = 0
    
    # Process records in chunks
    for i in range(0, len(record_ids), chunk_size):
        chunk = record_ids[i:i + chunk_size]
        try:
            success = table.batch_delete_records(chunk)
            if success:
                total_deleted += len(chunk)
                print(f"Deleted chunk {i//chunk_size + 1}: {len(chunk)} records")
        except TeableError as e:
            print(f"Error deleting chunk: {e}")
    
    return total_deleted
```

## Error Handling

### Comprehensive Error Handling

```python
from teable.exceptions import TeableError, ResourceNotFoundError, ValidationError

def handle_record_deletion(table, record_id):
    """Handle record deletion with comprehensive error handling."""
    try:
        # Attempt to delete record
        success = table.delete_record(record_id)
        
        if success:
            print(f"Successfully deleted record {record_id}")
            return True
        else:
            print(f"Failed to delete record {record_id}")
            return False
            
    except ResourceNotFoundError:
        print(f"Record {record_id} not found")
        return False
    except ValidationError as e:
        print(f"Validation error: {e}")
        return False
    except TeableError as e:
        print(f"Teable error: {e}")
        return False
    except Exception as e:
        print(f"Unexpected error: {e}")
        return False
```

### Batch Error Recovery

```python
def batch_delete_with_recovery(table, record_ids, max_retries=3):
    """Attempt batch deletion with retry for failed records."""
    remaining_ids = record_ids
    retry_count = 0
    total_deleted = 0
    
    while remaining_ids and retry_count < max_retries:
        try:
            success = table.batch_delete_records(remaining_ids)
            if success:
                total_deleted += len(remaining_ids)
                remaining_ids = []
            else:
                retry_count += 1
                
        except TeableError as e:
            print(f"Error on attempt {retry_count + 1}: {e}")
            retry_count += 1
            
    if remaining_ids:
        print(f"Failed to delete {len(remaining_ids)} records after {max_retries} attempts")
        
    return total_deleted
```

## Best Practices

1. **Data Validation**
   - Verify records exist before deletion
   - Handle missing records gracefully
   - Consider data relationships
   - Document deletion requirements

2. **Performance**
   - Use batch deletions for multiple records
   - Process large deletions in chunks
   - Monitor deletion performance
   - Consider rate limits

3. **Error Handling**
   - Implement comprehensive error handling
   - Log deletion failures
   - Provide meaningful error messages
   - Consider recovery strategies

4. **Data Integrity**
   - Consider cascading deletions
   - Validate data relationships
   - Maintain referential integrity
   - Back up important data

## Next Steps

After mastering record deletion, you can:

- [Query records](read.md)
- [Update records](update.md)
- [Work with bulk operations](../tables/bulk-operations.md)
- [Set up views](../views/creation.md)
