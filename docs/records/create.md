# Creating Records

This guide covers all methods for creating records in Teable tables.

## Single Record Creation

```python
from teable import TeableClient, TeableConfig

# Initialize client
client = TeableClient(TeableConfig(api_key="your_api_key"))

# Create a single record
record = client.records.create_record(
    table_id="table_id",
    fields={
        "Name": "John Doe",
        "Email": "john@example.com",
        "Age": 30
    }
)
```

## Batch Record Creation

```python
# Create multiple records at once
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

batch_result = client.records.batch_create_records(
    table_id="table_id",
    records=records_data,
    field_key_type="name",  # Use field names instead of IDs
    typecast=True,  # Enable automatic type conversion
    order={  # Optional ordering configuration
        "viewId": "view_id",
        "position": "after",
        "recordId": "anchor_record_id"
    }
)

print(f"Successfully created {batch_result.success_count} records")
if batch_result.failure_count > 0:
    print(f"Failed to create {batch_result.failure_count} records")
    for error in batch_result.errors:
        print(f"Error in record {error.index}: {error.message}")
```

## Record Duplication

```python
# Duplicate an existing record
duplicated = client.records.duplicate_record(
    table_id="table_id",
    record_id="record_to_duplicate",
    view_id="view_id",
    anchor_id="anchor_record_id",
    position="after"  # 'before' or 'after'
)
```

## Attachment Upload

```python
# Upload file attachment
with open("document.pdf", "rb") as f:
    file_data = f.read()
    
result = client.records.upload_attachment(
    table_id="table_id",
    record_id="record_id",
    field_id="attachment_field_id",
    file=file_data,
    mime_type="application/pdf"
)

# Or upload from URL
result = client.records.upload_attachment(
    table_id="table_id",
    record_id="record_id",
    field_id="attachment_field_id",
    file_url="https://example.com/document.pdf"
)
```

## Validation Rules

### Field Values

- Must be provided as a dictionary
- Cannot be empty
- Keys must match field names or IDs (based on field_key_type)
- Values must match field type requirements

```python
# Example of field validation
try:
    record = client.records.create_record(
        table_id="table_id",
        fields={}  # Empty fields - will raise ValidationError
    )
except ValidationError as e:
    print(f"Validation error: {str(e)}")
```

### Batch Operations

- Records must be provided as a list
- List cannot be empty
- Maximum 2000 records per batch operation
- Each record must follow field validation rules

```python
# Example of batch validation
try:
    batch_result = client.records.batch_create_records(
        table_id="table_id",
        records=[{} for _ in range(2001)]  # Too many records
    )
except ValidationError as e:
    print(f"Validation error: {str(e)}")
```

## Error Handling

```python
from teable.exceptions import ValidationError, APIError

try:
    record = client.records.create_record(
        table_id="table_id",
        fields={
            "Name": "",  # Empty required field
            "Email": "invalid-email"
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

### 1. Use Batch Operations

```python
# DON'T: Create records one by one
for data in records_data:
    client.records.create_record(table_id, data)  # Inefficient

# DO: Use batch creation
batch_result = client.records.batch_create_records(
    table_id,
    records_data
)
```

### 2. Enable Typecast

```python
# Enable automatic type conversion
batch_result = client.records.batch_create_records(
    table_id="table_id",
    records=records_data,
    typecast=True  # Converts values to correct types
)
```

### 3. Handle Partial Success

```python
batch_result = client.records.batch_create_records(
    table_id="table_id",
    records=records_data
)

# Handle successful and failed records
print(f"Created {batch_result.success_count} records")
if batch_result.failure_count > 0:
    print(f"Failed to create {batch_result.failure_count} records:")
    for error in batch_result.errors:
        print(f"Record {error.index}: {error.message}")
        # Retry failed records or log errors
```

### 4. Implement Retry Logic

```python
import time
from teable.exceptions import APIError

def retry_batch_create(client, table_id, records, max_retries=3):
    for attempt in range(max_retries):
        try:
            return client.records.batch_create_records(
                table_id=table_id,
                records=records
            )
        except APIError as e:
            if e.status_code == 429:  # Rate limit
                if attempt < max_retries - 1:
                    time.sleep(2 ** attempt)  # Exponential backoff
                    continue
            raise
    raise Exception("Max retries exceeded")
```

### 5. Validate Data Before Creation

```python
def validate_record_data(data):
    required_fields = ["Name", "Email"]
    for field in required_fields:
        if not data.get(field):
            raise ValidationError(f"Missing required field: {field}")
    
    # Validate email format
    email = data.get("Email")
    if email and not "@" in email:
        raise ValidationError("Invalid email format")

# Use validation before creation
try:
    for record in records_data:
        validate_record_data(record)
    
    batch_result = client.records.batch_create_records(
        table_id="table_id",
        records=records_data
    )
except ValidationError as e:
    print(f"Data validation failed: {str(e)}")
