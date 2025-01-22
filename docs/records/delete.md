# Deleting Records

This guide covers how to delete records from Teable tables using the Teable-Client library.

## Basic Record Deletion

### Deleting a Single Record

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Delete a record
try:
    success = client.records.delete(
        table_id="table_id",
        record_id="rec123"
    )
    if success:
        print("Record successfully deleted")
except TeableError as e:
    print(f"Error deleting record: {e}")
```

### Safe Record Deletion

```python
from teable.exceptions import TeableError, ResourceNotFoundError

def safe_delete_record(client, table_id, record_id):
    """Safely delete a record with error handling."""
    try:
        # Verify record exists
        client.records.get(
            table_id=table_id,
            record_id=record_id
        )
        
        # Delete record
        success = client.records.delete(
            table_id=table_id,
            record_id=record_id
        )
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
    success = client.records.batch_delete_records(
        table_id="table_id",
        record_ids=record_ids
    )
    if success:
        print(f"Successfully deleted {len(record_ids)} records")
except TeableError as e:
    print(f"Error during batch deletion: {e}")
```

### Batch Deletion with Validation

```python
def validate_and_delete_records(client, table_id, record_ids):
    """Validate records before deletion."""
    # Verify all records exist
    existing_records = []
    missing_records = []
    
    for record_id in record_ids:
        try:
            client.records.get(
                table_id=table_id,
                record_id=record_id
            )
            existing_records.append(record_id)
        except ResourceNotFoundError:
            missing_records.append(record_id)
    
    if missing_records:
        print(f"Warning: Records not found: {missing_records}")
    
    if existing_records:
        success = client.records.batch_delete_records(
            table_id=table_id,
            record_ids=existing_records
        )
        return success, len(existing_records)
    
    return False, 0
```

## Error Handling

### Comprehensive Error Handling

```python
from teable.exceptions import TeableError, ResourceNotFoundError, ValidationError

def handle_record_deletion(client, table_id, record_id):
    """Handle record deletion with comprehensive error handling."""
    try:
        # Attempt to delete record
        success = client.records.delete(
            table_id=table_id,
            record_id=record_id
        )
        
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

## Best Practices

1. **Data Validation**
   - Verify records exist before deletion
   - Handle missing records gracefully
   - Consider data relationships
   - Document deletion requirements

2. **Performance**
   - Use batch deletions for multiple records
   - Monitor deletion performance
   - Consider rate limits
   - Handle failures appropriately

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
- [Work with bulk operations](bulk-operations.md)
- [Set up views](../views/creation.md)
