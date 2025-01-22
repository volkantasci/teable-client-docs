# Reading and Querying Records

This guide covers how to read and query records in Teable tables using the Teable-Client library.

## Reading Individual Records

### Getting a Single Record

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Get a record by ID
try:
    record = client.records.get(
        table_id="table_id",
        record_id="rec123",
        cell_format="json"  # or "text"
    )
    print(f"Record ID: {record.record_id}")
    print(f"Fields: {record.fields}")
except ResourceNotFoundError:
    print("Record not found")
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

### Using Query Builder

```python
from teable import FilterOperator, SortDirection

# Get table for query builder
table = client.tables.get("table_id")

# Create a query using the query builder
query = table.query()

# Add filters
query.filter(
    field="Department",
    operator=FilterOperator.EQUALS,
    value="Engineering"
)
query.filter(
    field="Salary",
    operator=FilterOperator.GREATER_THAN,
    value=75000
)

# Add sorting
query.sort(
    field="Name",
    direction=SortDirection.ASCENDING
)

# Add pagination
query.paginate(take=10)

# Execute the query
records = client.records.get_records(
    table_id="table_id",
    query=query.build()
)
```

### Pagination

```python
# Get records with pagination
records = client.records.get_records(
    table_id="table_id",
    take=50,    # Number of records to return
    skip=100    # Number of records to skip
)
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
