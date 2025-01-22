# Data Insertion and Record Creation

This guide covers how to insert data and create records in Teable tables using the Teable-Client library.

## Single Record Creation

### Basic Record Creation

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Create a single record
record = client.records.create(
    table_id="table_id",
    fields={
        "Name": "John Doe",
        "Email": "john@example.com",
        "Department": "Engineering",
        "Start Date": "2023-01-15"
    }
)

print(f"Created record ID: {record.record_id}")
```

### Record Creation with Validation

```python
from teable.exceptions import ValidationError

try:
    # Get table for validation
    table = client.tables.get("table_id")
    
    # Prepare record data
    data = {
        "Name": "Jane Smith",
        "Email": "jane@example.com",
        "Salary": 75000,
        "Active": True
    }
    
    # Validate data
    table.validate_record_fields(data)
    
    # Create record
    record = client.records.create(
        table_id=table.table_id,
        fields=data
    )
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

## Validation and Error Handling

### Pre-validation

```python
try:
    # Get table for validation
    table = client.tables.get("table_id")
    
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
    # Prepare records
    records = [
        {"Name": "User 1", "Email": "valid@example.com"},
        {"Name": "User 2", "Email": "invalid-email"},
        {"Name": "User 3", "Email": "another@example.com"}
    ]
    
    # Attempt batch creation
    result = client.records.batch_create_records(
        table_id="table_id",
        records=records
    )
    
    # Check results
    print(f"Successfully created: {result.success_count}")
    print(f"Failed to create: {result.failure_count}")
    print(f"Success rate: {result.success_rate}%")
    
except TeableError as e:
    print(f"Batch operation failed: {e}")
```

## Best Practices

1. **Data Validation**
   - Validate data before insertion
   - Handle validation errors gracefully
   - Use appropriate data types
   - Clean and normalize data

2. **Batch Operations**
   - Use batch operations for multiple records
   - Handle partial failures appropriately
   - Monitor performance
   - Consider rate limits

3. **Error Handling**
   - Implement comprehensive error handling
   - Log validation failures
   - Provide meaningful error messages
   - Handle errors appropriately

4. **Performance**
   - Use batch operations when possible
   - Monitor API usage
   - Prepare data efficiently
   - Consider rate limits

## Error Handling Examples

```python
from teable.exceptions import TeableError, ValidationError, ResourceNotFoundError

def safe_create_record(client, table_id, data):
    """Safely create a record with error handling."""
    try:
        # Get table for validation
        table = client.tables.get(table_id)
        
        # Validate first
        table.validate_record_fields(data)
        
        # Create record
        record = client.records.create(
            table_id=table_id,
            fields=data
        )
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
