# Data Insertion and Record Creation

This guide covers how to insert data and create records in Teable tables using the Teable-Client library. Learn about single record creation, batch operations, and data validation.

## Single Record Creation

### Basic Record Creation

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Get your table
table = client.get_table("table_id")

# Create a single record
record = table.create_record({
    "Name": "John Doe",
    "Email": "john@example.com",
    "Department": "Engineering",
    "Start Date": "2023-01-15"
})

print(f"Created record ID: {record.id}")
```

### Record Creation with Validation

```python
from teable.exceptions import ValidationError

try:
    # Create record with validation
    record = table.create_record({
        "Name": "Jane Smith",
        "Email": "jane@example.com",
        "Salary": 75000,
        "Active": True
    })
except ValidationError as e:
    print(f"Validation error: {e}")
```

## Batch Record Creation

### Creating Multiple Records

```python
# Prepare multiple records
records = [
    {
        "Name": "Alice Johnson",
        "Email": "alice@example.com",
        "Department": "Marketing"
    },
    {
        "Name": "Bob Wilson",
        "Email": "bob@example.com",
        "Department": "Sales"
    },
    {
        "Name": "Carol Brown",
        "Email": "carol@example.com",
        "Department": "Engineering"
    }
]

# Batch create records
result = table.batch_create_records(
    records=records,
    field_key_type="name",  # Use field names instead of IDs
    typecast=True  # Automatically convert data types
)

print(f"Successfully created {result.success_count} records")
```

### Batch Creation with Order

```python
# Create records with specific order
result = table.batch_create_records(
    records=records,
    order={
        "viewId": "view123",
        "anchorId": "record123",
        "position": "after"
    }
)
```

## Advanced Data Insertion

### Type Casting

```python
# Create record with automatic type casting
record = table.create_record(
    fields={
        "Name": "David Lee",
        "Hire Date": "2023-01-15",  # String date will be converted
        "Salary": "75000",          # String number will be converted
        "Active": "true"            # String boolean will be converted
    },
    typecast=True
)
```

### Field Key Types

```python
# Using field IDs instead of names
record = table.create_record(
    fields={
        "fld123": "David Lee",      # Field ID for Name
        "fld456": "david@example.com"  # Field ID for Email
    },
    field_key_type="id"
)
```

## Validation and Error Handling

### Pre-validation

```python
try:
    # Validate fields before creation
    table.validate_record_fields({
        "Name": "Test User",
        "Email": "invalid-email",
        "Department": "Invalid Dept"
    })
except ValidationError as e:
    print(f"Validation failed: {e}")
```

### Batch Error Handling

```python
from teable.exceptions import TeableError

try:
    # Attempt batch creation
    records = [
        {"Name": "User 1", "Email": "valid@example.com"},
        {"Name": "User 2", "Email": "invalid-email"},
        {"Name": "User 3", "Email": "another@example.com"}
    ]
    
    result = table.batch_create_records(records)
    
    # Check for partial success
    print(f"Successful: {result.success_count}")
    print(f"Failed: {result.failure_count}")
    
    # Handle failed records
    for error in result.errors:
        print(f"Error in record {error.index}: {error.message}")
        
except TeableError as e:
    print(f"Batch operation failed: {e}")
```

## Performance Optimization

### Chunked Batch Operations

```python
def create_records_in_chunks(table, records, chunk_size=1000):
    """Create records in chunks to handle large datasets."""
    for i in range(0, len(records), chunk_size):
        chunk = records[i:i + chunk_size]
        result = table.batch_create_records(chunk)
        print(f"Processed chunk {i//chunk_size + 1}: {result.success_count} successful")
```

### Efficient Data Preparation

```python
# Prepare data efficiently for batch insertion
from typing import List, Dict, Any

def prepare_records(data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
    """Prepare records for batch insertion with validation."""
    prepared_records = []
    
    for item in data:
        # Basic data cleaning
        record = {
            k: v.strip() if isinstance(v, str) else v
            for k, v in item.items()
            if v is not None
        }
        
        # Add any default values
        record.setdefault("Status", "Active")
        
        prepared_records.append(record)
    
    return prepared_records

# Use the prepared data
raw_data = [{"Name": "User 1 "}, {"Name": "User 2 "}]  # Note trailing spaces
records = prepare_records(raw_data)
result = table.batch_create_records(records)
```

## Best Practices

1. **Data Validation**
   - Validate data before insertion
   - Handle validation errors gracefully
   - Use appropriate data types
   - Clean and normalize data

2. **Batch Operations**
   - Use batch operations for multiple records
   - Process large datasets in chunks
   - Handle partial failures appropriately
   - Monitor performance

3. **Error Handling**
   - Implement comprehensive error handling
   - Log validation failures
   - Provide meaningful error messages
   - Handle retries for transient failures

4. **Performance**
   - Use appropriate chunk sizes
   - Minimize API calls
   - Prepare data efficiently
   - Monitor memory usage

## Error Handling Examples

```python
from teable.exceptions import TeableError, ValidationError, ResourceNotFoundError

def safe_create_record(table, data):
    """Safely create a record with error handling."""
    try:
        # Validate first
        table.validate_record_fields(data)
        
        # Create record
        record = table.create_record(data)
        return record
        
    except ValidationError as e:
        print(f"Data validation failed: {e}")
        return None
    except ResourceNotFoundError as e:
        print(f"Table or field not found: {e}")
        return None
    except TeableError as e:
        print(f"Operation failed: {e}")
        return None
```

## Next Steps

After mastering data insertion, you can:

- [Query and filter records](../records/read.md)
- [Update existing records](../records/update.md)
- [Delete records](../records/delete.md)
- [Work with views](../views/creation.md)
