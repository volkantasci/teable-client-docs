# Bulk Operations

This guide covers all batch operations available in the Teable Client Library for efficient handling of multiple records.

## Batch Creation

### Basic Batch Creation

```python
from teable import TeableClient, TeableConfig

# Initialize client
client = TeableClient(TeableConfig(api_key="your_api_key"))

# Prepare records data
records_data = [
    {
        "Name": "Alice Smith",
        "Email": "alice@example.com",
        "Age": 25
    },
    {
        "Name": "Bob Johnson",
        "Email": "bob@example.com",
        "Age": 35
    }
]

# Create multiple records at once
batch_result = client.records.batch_create_records(
    table_id="table_id",
    records=records_data,
    field_key_type="name",  # Use field names instead of IDs
    typecast=True  # Enable automatic type conversion
)

print(f"Successfully created {batch_result.success_count} records")
if batch_result.failure_count > 0:
    print(f"Failed to create {batch_result.failure_count} records")
    for error in batch_result.errors:
        print(f"Error in record {error.index}: {error.message}")
```

### Batch Creation with Ordering

```python
# Create records with specific ordering
batch_result = client.records.batch_create_records(
    table_id="table_id",
    records=records_data,
    field_key_type="name",
    typecast=True,
    order={
        "viewId": "view_id",
        "position": "after",
        "recordId": "anchor_record_id"
    }
)
```

## Batch Updates

### Basic Batch Update

```python
# Prepare update data
updates = [
    {
        "id": "record1_id",
        "fields": {
            "Status": "Completed",
            "Progress": 100
        }
    },
    {
        "id": "record2_id",
        "fields": {
            "Status": "In Progress",
            "Progress": 50
        }
    }
]

# Update multiple records at once
updated_records = client.records.batch_update_records(
    table_id="table_id",
    updates=updates,
    field_key_type="name",
    typecast=True
)
```

### Batch Update with Ordering

```python
# Update records with specific ordering
updated_records = client.records.batch_update_records(
    table_id="table_id",
    updates=updates,
    field_key_type="name",
    typecast=True,
    order={
        "viewId": "view_id",
        "position": "after",
        "recordId": "anchor_record_id"
    }
)
```

## Batch Deletion

```python
# Delete multiple records at once
record_ids = ["record1_id", "record2_id", "record3_id"]

success = client.records.batch_delete_records(
    table_id="table_id",
    record_ids=record_ids
)
```

## Validation Rules

### Batch Size Limits

```python
# Maximum 2000 records per batch operation
try:
    records = [{"fields": {}} for _ in range(2001)]  # Too many records
    client.records.batch_create_records(
        table_id="table_id",
        records=records
    )
except ValidationError as e:
    print(f"Validation error: {str(e)}")
```

### Field Validation

```python
def validate_batch_data(records):
    """Validate batch data before sending to API."""
    for i, record in enumerate(records):
        if not isinstance(record, dict):
            raise ValidationError(f"Record {i} must be a dictionary")
        
        if "fields" not in record:
            raise ValidationError(f"Record {i} missing 'fields' key")
            
        fields = record["fields"]
        if not isinstance(fields, dict):
            raise ValidationError(f"Fields in record {i} must be a dictionary")
            
        # Validate required fields
        if "Name" in fields and not fields["Name"]:
            raise ValidationError(f"Name in record {i} cannot be empty")
            
        # Validate email format
        if "Email" in fields:
            email = fields["Email"]
            if not "@" in email:
                raise ValidationError(f"Invalid email in record {i}")

# Use validation
try:
    validate_batch_data(records_data)
    batch_result = client.records.batch_create_records(
        table_id="table_id",
        records=records_data
    )
except ValidationError as e:
    print(f"Validation failed: {str(e)}")
```

## Error Handling

### Basic Error Handling

```python
from teable.exceptions import ValidationError, APIError

try:
    batch_result = client.records.batch_create_records(
        table_id="table_id",
        records=records_data
    )
except ValidationError as e:
    print(f"Validation error: {str(e)}")
except APIError as e:
    print(f"API error: {str(e)}")
    print(f"Status code: {e.status_code}")
    print(f"Error details: {e.details}")
```

### Handling Partial Success

```python
def handle_batch_operation(client, table_id, records):
    """Handle batch operation with partial success tracking."""
    try:
        batch_result = client.records.batch_create_records(
            table_id=table_id,
            records=records
        )
        
        # Handle successful records
        if batch_result.success_count > 0:
            print(f"Successfully processed {batch_result.success_count} records")
            
        # Handle failed records
        if batch_result.failure_count > 0:
            print(f"Failed to process {batch_result.failure_count} records")
            failed_records = []
            
            for error in batch_result.errors:
                print(f"Record {error.index}: {error.message}")
                failed_records.append(records[error.index])
            
            # Implement retry logic for failed records
            return retry_failed_records(client, table_id, failed_records)
            
        return batch_result
        
    except APIError as e:
        print(f"Batch operation failed: {str(e)}")
        raise
```

## Best Practices

### 1. Process in Batches

```python
def process_large_dataset(client, table_id, records, batch_size=500):
    """Process large number of records in batches."""
    results = []
    
    # Split into smaller batches
    for i in range(0, len(records), batch_size):
        batch = records[i:i + batch_size]
        try:
            batch_result = client.records.batch_create_records(
                table_id=table_id,
                records=batch
            )
            results.append(batch_result)
            
        except APIError as e:
            print(f"Batch {i//batch_size + 1} failed: {str(e)}")
            # Handle error or retry
    
    return results
```

### 2. Implement Retry Logic

```python
import time
from teable.exceptions import APIError

def retry_batch_operation(operation, max_retries=3):
    """Retry batch operation with exponential backoff."""
    for attempt in range(max_retries):
        try:
            return operation()
        except APIError as e:
            if e.status_code == 429:  # Rate limit
                if attempt < max_retries - 1:
                    time.sleep(2 ** attempt)  # Exponential backoff
                    continue
            raise
    raise Exception("Max retries exceeded")

# Usage
batch_result = retry_batch_operation(
    lambda: client.records.batch_create_records(
        table_id=table_id,
        records=records_data
    )
)
```

### 3. Validate Before Processing

```python
def validate_and_process(client, table_id, records):
    """Validate and process records with error handling."""
    # Group records by operation type
    to_create = []
    to_update = []
    to_delete = []
    
    for record in records:
        try:
            if "id" not in record:
                # New record
                validate_creation_data(record)
                to_create.append(record)
            elif record.get("_delete"):
                # Record to delete
                validate_record_id(record["id"])
                to_delete.append(record["id"])
            else:
                # Record to update
                validate_update_data(record)
                to_update.append(record)
                
        except ValidationError as e:
            print(f"Validation failed for record: {str(e)}")
            continue
    
    # Process each group
    results = {
        "created": [],
        "updated": [],
        "deleted": [],
        "failed": []
    }
    
    if to_create:
        try:
            batch_result = client.records.batch_create_records(
                table_id=table_id,
                records=to_create
            )
            results["created"] = batch_result.successful
        except APIError as e:
            print(f"Creation failed: {str(e)}")
    
    if to_update:
        try:
            updated = client.records.batch_update_records(
                table_id=table_id,
                updates=to_update
            )
            results["updated"] = updated
        except APIError as e:
            print(f"Update failed: {str(e)}")
    
    if to_delete:
        try:
            success = client.records.batch_delete_records(
                table_id=table_id,
                record_ids=to_delete
            )
            if success:
                results["deleted"] = to_delete
        except APIError as e:
            print(f"Deletion failed: {str(e)}")
    
    return results
```

### 4. Monitor Progress

```python
from datetime import datetime

def process_with_monitoring(client, table_id, records, batch_size=500):
    """Process records with progress monitoring."""
    start_time = datetime.now()
    total_batches = (len(records) + batch_size - 1) // batch_size
    processed = 0
    
    print(f"Starting batch processing of {len(records)} records")
    
    for i in range(0, len(records), batch_size):
        batch_start = datetime.now()
        batch = records[i:i + batch_size]
        
        try:
            batch_result = client.records.batch_create_records(
                table_id=table_id,
                records=batch
            )
            processed += batch_result.success_count
            
            # Calculate progress
            batch_num = i // batch_size + 1
            progress = (batch_num / total_batches) * 100
            elapsed = datetime.now() - start_time
            batch_time = datetime.now() - batch_start
            
            print(f"Batch {batch_num}/{total_batches} ({progress:.1f}%)")
            print(f"Processed: {processed}/{len(records)}")
            print(f"Batch time: {batch_time}")
            print(f"Total time: {elapsed}")
            
        except APIError as e:
            print(f"Batch {batch_num} failed: {str(e)}")
    
    total_time = datetime.now() - start_time
    print(f"\nProcessing complete:")
    print(f"Total records: {len(records)}")
    print(f"Successfully processed: {processed}")
    print(f"Total time: {total_time}")
