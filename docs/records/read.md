# Reading and Querying Records

This guide covers how to read and query records in Teable tables using the Teable-Client library.

## Reading Individual Records

### Getting a Single Record

```python
from teable import TeableClient, Record

# Initialize the client
client = TeableClient()

# Get a record by ID and convert to Record object
try:
    record = Record.from_api_response(
        client.records.get_record(
            table_id="table_id",
            record_id="rec123"
        )
    )
    print(f"Record ID: {record.record_id}")
    print(f"Fields: {record.fields}")
except ResourceNotFoundError:
    print("Record not found")

# Get record status
status = client.records.get_record_status(
    table_id="table_id",
    record_id="rec123"
)
print(f"Is visible: {status.is_visible}")
print(f"Is deleted: {status.is_deleted}")

# Get record history
history = client.records.get_record_history(
    table_id="table_id",
    record_id="rec123"
)
print(f"History users: {history.users}")

# Get table record history
table_history = client.records.get_table_record_history(
    table_id="table_id"
)
print(f"Table history users: {table_history.users}")
```

### Specifying Fields to Return

```python
# Get specific fields only
record = client.records.get(
    table_id="table_id",
    record_id="rec123",
    projection=["Name", "Email", "Department"]
)

# Access the fields
print(f"Name: {record.fields.get('Name')}")
print(f"Email: {record.fields.get('Email')}")
```

## Querying Multiple Records

### Basic Query

```python
# Get all records
records = client.records.get_records(
    table_id="table_id"
)

# Process records
for record in records:
    print(f"Record ID: {record.record_id}")
    print(f"Fields: {record.fields}")
    print("---")
```

### Advanced Querying

#### Using Filter Sets

```python
# Get field IDs first
fields = client.fields.get_table_fields(table_id)
category_field = next(f for f in fields if f.name == "Category")

# Create filter parameters
filter_params = {
    "filterSet": [
        {
            "operator": "is",
            "fieldId": category_field.field_id,
            "value": "A"
        }
    ],
    "conjunction": "and"
}

# Get filtered records
filtered_records = client.records.get_records(
    table_id="table_id",
    filter=filter_params
)
```

#### Using Search

```python
# Get field IDs first
fields = client.fields.get_table_fields(table_id)
name_field = next(f for f in fields if f.name == "Name")

# Create search parameters
search_params = {
    "search": [{
        "value": "John",
        "field": name_field.field_id,
        "exact": True  # For exact matching
    }]
}

# Get searched records
searched_records = client.records.get_records(
    table_id="table_id",
    **search_params
)
```

### Pagination

```python
# Skip default empty records (usually first 3)
records = client.records.get_records(
    table_id="table_id",
    skip=3,     # Skip default empty records
    take=50     # Number of records to return
)

# For regular pagination
records = client.records.get_records(
    table_id="table_id",
    skip=100,   # Skip first 100 records
    take=50     # Return next 50 records
)

# Verify pagination results
assert len(records) == 50  # Got requested number of records
```

## Field Formatting

### Cell Format Options

```python
# Get records with JSON formatting
records = client.records.get_records(
    table_id="table_id",
    cell_format="json"
)

# Get records with text formatting
records = client.records.get_records(
    table_id="table_id",
    cell_format="text"
)
```

### Field Key Types

```python
# Use field names as keys
records = client.records.get_records(
    table_id="table_id",
    field_key_type="name"
)

# Use field IDs as keys
records = client.records.get_records(
    table_id="table_id",
    field_key_type="id"
)
```

## Error Handling

```python
from teable.exceptions import TeableError, ResourceNotFoundError

def safe_get_records(client, table_id, **query_params):
    """Safely get records with error handling."""
    try:
        records = client.records.get_records(
            table_id=table_id,
            **query_params
        )
        return records
    except ResourceNotFoundError:
        print("Table not found")
        return []
    except TeableError as e:
        print(f"Error retrieving records: {e}")
        return []
```

## Performance Optimization

### Projection Optimization

```python
def get_minimal_records(client, table_id, required_fields):
    """Get records with only required fields."""
    return client.records.get_records(
        table_id=table_id,
        projection=required_fields,
        cell_format="json"
    )

# Example usage
minimal_records = get_minimal_records(
    client,
    "table_id",
    ["Name", "Email"]
)
```

### Batch Processing

```python
def process_records_in_batches(client, table_id, batch_size=1000):
    """Process records in batches to manage memory."""
    processed = 0
    while True:
        batch = client.records.get_records(
            table_id=table_id,
            skip=processed,
            take=batch_size
        )
        
        if not batch:
            break
            
        for record in batch:
            # Process record
            process_record(record)
            
        processed += len(batch)
        print(f"Processed {processed} records")
```

## Best Practices

1. **Query Optimization**
   - Use projections to limit returned fields
   - Implement pagination for large datasets
   - Use appropriate filters to reduce data transfer
   - Consider caching frequently accessed data

2. **Error Handling**
   - Implement comprehensive error handling
   - Handle resource not found cases
   - Provide meaningful error messages
   - Consider retry strategies

3. **Performance**
   - Use batch processing for large datasets
   - Optimize query filters
   - Monitor query performance
   - Use appropriate cell formats

4. **Data Format**
   - Choose appropriate cell formats
   - Use consistent field key types
   - Handle null values appropriately
   - Document format requirements

## Next Steps

After mastering record querying, you can:

- [Update existing records](update.md)
- [Delete records](delete.md)
- [Work with views](../views/creation.md)
- [Implement data synchronization](../integration/sync.md)
