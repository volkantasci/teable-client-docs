# Deleting Records

This guide covers all methods for deleting records from Teable tables.

## Single Record Deletion

### Basic Deletion

```python
from teable import TeableClient, TeableConfig

# Initialize client
client = TeableClient(TeableConfig(api_key="your_api_key"))

# Delete a single record
success = client.records.delete_record(
    table_id="table_id",
    record_id="record_id"
)

if success:
    print("Record deleted successfully")
```

## Batch Record Deletion

### Basic Batch Deletion

```python
# Delete multiple records at once
record_ids = ["record1_id", "record2_id", "record3_id"]

success = client.records.batch_delete_records(
    table_id="table_id",
    record_ids=record_ids
)

if success:
    print("Records deleted successfully")
```

## Record Status Verification

Before or after deletion, you can check a record's status:

```python
# Check record status
status = client.records.get_record_status(
    table_id="table_id",
    record_id="record_id"
)

print(f"Is visible: {status.is_visible}")
print(f"Is deleted: {status.is_deleted}")
```

## Error Handling

```python
from teable.exceptions import ValidationError, APIError

try:
    # Attempt to delete record
    client.records.delete_record(
        table_id="table_id",
        record_id="record_id"
    )
except ValidationError as e:
    print(f"Validation error: {str(e)}")
except APIError as e:
    if e.status_code == 404:
        print("Record not found")
    else:
        print(f"API error: {str(e)}")
        print(f"Status code: {e.status_code}")
        print(f"Error details: {e.details}")
```

## Best Practices

### 1. Use Batch Deletion

```python
# DON'T: Delete records one by one
for record_id in record_ids:
    client.records.delete_record(
        table_id="table_id",
        record_id=record_id
    )

# DO: Use batch deletion
client.records.batch_delete_records(
    table_id="table_id",
    record_ids=record_ids
)
```

### 2. Implement Retry Logic

```python
import time
from teable.exceptions import APIError

def retry_batch_delete(client, table_id, record_ids, max_retries=3):
    """Retry batch deletion with exponential backoff."""
    for attempt in range(max_retries):
        try:
            return client.records.batch_delete_records(
                table_id=table_id,
                record_ids=record_ids
            )
        except APIError as e:
            if e.status_code == 429:  # Rate limit
                if attempt < max_retries - 1:
                    time.sleep(2 ** attempt)  # Exponential backoff
                    continue
            raise
    raise Exception("Max retries exceeded")
```

### 3. Validate Before Deletion

```python
def validate_record_ids(client, table_id, record_ids):
    """Validate records exist before deletion."""
    invalid_ids = []
    for record_id in record_ids:
        try:
            status = client.records.get_record_status(
                table_id=table_id,
                record_id=record_id
            )
            if status.is_deleted:
                invalid_ids.append(record_id)
        except APIError as e:
            if e.status_code == 404:
                invalid_ids.append(record_id)
    
    if invalid_ids:
        raise ValidationError(
            f"Records not found or already deleted: {invalid_ids}"
        )

# Use validation before deletion
try:
    validate_record_ids(client, table_id, record_ids)
    client.records.batch_delete_records(
        table_id=table_id,
        record_ids=record_ids
    )
except ValidationError as e:
    print(f"Validation failed: {str(e)}")
```

### 4. Handle Large Deletions

```python
def delete_records_in_batches(client, table_id, record_ids, batch_size=500):
    """Delete large number of records in batches."""
    results = []
    
    # Split deletions into smaller batches
    for i in range(0, len(record_ids), batch_size):
        batch = record_ids[i:i + batch_size]
        try:
            success = client.records.batch_delete_records(
                table_id=table_id,
                record_ids=batch
            )
            if success:
                results.extend(batch)
                
        except APIError as e:
            print(f"Batch {i//batch_size + 1} failed: {str(e)}")
            # Handle error or retry
    
    return results
```

### 5. Track Deletion History

```python
def delete_with_history(client, table_id, record_id):
    """Delete record and log its history."""
    try:
        # Get record history before deletion
        history = client.records.get_record_history(
            table_id=table_id,
            record_id=record_id
        )
        
        # Store history for audit purposes
        last_modified = history.entries[-1].created_time
        modified_by = history.users[history.entries[-1].user_id].name
        
        # Delete the record
        success = client.records.delete_record(
            table_id=table_id,
            record_id=record_id
        )
        
        if success:
            print(f"Deleted record {record_id}")
            print(f"Last modified: {last_modified}")
            print(f"Modified by: {modified_by}")
            
        return success
        
    except APIError as e:
        print(f"Deletion failed: {str(e)}")
        raise
```

### 6. Implement Soft Delete

If you need to implement a soft delete mechanism:

```python
def soft_delete_record(client, table_id, record_id):
    """Mark record as deleted without actually deleting it."""
    try:
        # Update record status instead of deleting
        client.records.update_record(
            table_id=table_id,
            record_id=record_id,
            fields={
                "is_deleted": True,
                "deleted_at": datetime.now().isoformat(),
                "deleted_by": client.auth.get_user().user_id
            }
        )
        return True
        
    except APIError as e:
        print(f"Soft delete failed: {str(e)}")
        raise

def restore_soft_deleted_record(client, table_id, record_id):
    """Restore a soft-deleted record."""
    try:
        client.records.update_record(
            table_id=table_id,
            record_id=record_id,
            fields={
                "is_deleted": False,
                "deleted_at": None,
                "deleted_by": None
            }
        )
        return True
        
    except APIError as e:
        print(f"Restore failed: {str(e)}")
        raise
