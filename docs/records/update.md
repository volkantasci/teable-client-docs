# Updating Records

This guide covers all methods for updating records in Teable tables.

## Single Record Updates

### Basic Update

```python
from teable import TeableClient, TeableConfig

# Initialize client
client = TeableClient(TeableConfig(api_key="your_api_key"))

# Update a single record
updated_record = client.records.update_record(
    table_id="table_id",
    record_id="record_id",
    fields={
        "Name": "Updated Name",
        "Email": "updated@example.com"
    }
)
```

### Advanced Update Options

```python
# Update with type conversion and field keys
updated_record = client.records.update_record(
    table_id="table_id",
    record_id="record_id",
    fields={
        "Age": "30",  # Will be converted to number
        "IsActive": "true"  # Will be converted to boolean
    },
    field_key_type="name",  # Use field names instead of IDs
    typecast=True,  # Enable automatic type conversion
    order={  # Optional record ordering
        "viewId": "view_id",
        "position": "after",
        "recordId": "anchor_record_id"
    }
)
```

## Batch Record Updates

### Basic Batch Update

```python
# Update multiple records at once
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

## Validation Rules

### Field Values

```python
from teable.exceptions import ValidationError

# Example of field validation
try:
    client.records.update_record(
        table_id="table_id",
        record_id="record_id",
        fields={}  # Empty fields - will raise ValidationError
    )
except ValidationError as e:
    print(f"Validation error: {str(e)}")
```

### Batch Operations

```python
# Batch update validation
try:
    # Too many records in one batch
    updates = [{"id": f"id_{i}", "fields": {}} for i in range(2001)]
    client.records.batch_update_records(
        table_id="table_id",
        updates=updates
    )
except ValidationError as e:
    print(f"Validation error: {str(e)}")
```

## Error Handling

```python
from teable.exceptions import ValidationError, APIError

try:
    client.records.update_record(
        table_id="table_id",
        record_id="record_id",
        fields={
            "Email": "invalid-email",
            "Age": "not-a-number"
        }
    )
except ValidationError as e:
    print(f"Validation error: {str(e)}")
except APIError as e:
    print(f"API error: {str(e)}")
    print(f"Status code: {e.status_code}")
    print(f"Error details: {e.details}")
```

## Best Practices

### 1. Use Batch Updates

```python
# DON'T: Update records one by one
for record_id, new_data in updates.items():
    client.records.update_record(
        table_id="table_id",
        record_id=record_id,
        fields=new_data
    )

# DO: Use batch update
batch_updates = [
    {"id": record_id, "fields": new_data}
    for record_id, new_data in updates.items()
]
client.records.batch_update_records(
    table_id="table_id",
    updates=batch_updates
)
```

### 2. Enable Typecast

```python
# Enable automatic type conversion
client.records.update_record(
    table_id="table_id",
    record_id="record_id",
    fields={
        "Age": "30",  # String will be converted to number
        "IsActive": "true",  # String will be converted to boolean
        "Date": "2023-01-01"  # String will be converted to date
    },
    typecast=True
)
```

### 3. Implement Retry Logic

```python
import time
from teable.exceptions import APIError

def retry_batch_update(client, table_id, updates, max_retries=3):
    for attempt in range(max_retries):
        try:
            return client.records.batch_update_records(
                table_id=table_id,
                updates=updates
            )
        except APIError as e:
            if e.status_code == 429:  # Rate limit
                if attempt < max_retries - 1:
                    time.sleep(2 ** attempt)  # Exponential backoff
                    continue
            raise
    raise Exception("Max retries exceeded")
```

### 4. Validate Before Updating

```python
def validate_update_data(data):
    """Validate update data before sending to API."""
    required_fields = ["Name", "Email"]
    for field in required_fields:
        if field in data and not data[field]:
            raise ValidationError(f"Field '{field}' cannot be empty")
    
    if "Email" in data:
        email = data["Email"]
        if not "@" in email:
            raise ValidationError("Invalid email format")
    
    if "Age" in data:
        try:
            age = int(data["Age"])
            if age < 0 or age > 120:
                raise ValidationError("Age must be between 0 and 120")
        except ValueError:
            raise ValidationError("Age must be a number")

# Use validation before update
try:
    update_data = {
        "Name": "John Doe",
        "Email": "john@example.com",
        "Age": 30
    }
    
    validate_update_data(update_data)
    
    client.records.update_record(
        table_id="table_id",
        record_id="record_id",
        fields=update_data
    )
except ValidationError as e:
    print(f"Validation failed: {str(e)}")
```

### 5. Handle Partial Success in Batch Updates

```python
def handle_batch_update(client, table_id, updates):
    """Handle batch updates with error tracking."""
    try:
        results = client.records.batch_update_records(
            table_id=table_id,
            updates=updates
        )
        
        # Track successful updates
        successful_ids = [r["id"] for r in results]
        
        # Find failed updates
        failed_ids = set(u["id"] for u in updates) - set(successful_ids)
        
        if failed_ids:
            print(f"Failed to update records: {failed_ids}")
            # Implement retry logic for failed records
            retry_updates = [
                u for u in updates
                if u["id"] in failed_ids
            ]
            # Retry failed updates...
            
        return results
        
    except APIError as e:
        print(f"Batch update failed: {str(e)}")
        raise
```

### 6. Optimize Large Updates

```python
def update_records_in_batches(client, table_id, updates, batch_size=500):
    """Update large number of records in batches."""
    results = []
    
    # Split updates into smaller batches
    for i in range(0, len(updates), batch_size):
        batch = updates[i:i + batch_size]
        try:
            batch_results = client.records.batch_update_records(
                table_id=table_id,
                updates=batch
            )
            results.extend(batch_results)
            
        except APIError as e:
            print(f"Batch {i//batch_size + 1} failed: {str(e)}")
            # Handle error or retry
    
    return results
