# Reading and Querying Records

This guide covers how to read and query records in Teable tables using the Teable-Client library. Learn about retrieving individual records, querying multiple records, and using various filtering and sorting options.

## Reading Individual Records

### Getting a Single Record

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Get your table
table = client.get_table("table_id")

# Get a record by ID
try:
    record = table.get_record(
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
record = table.get_record(
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
records = table.get_records()

# Process records
for record in records:
    print(f"Record ID: {record.record_id}")
    print(f"Name: {record.fields.get('Name')}")
    print("---")
```

### Filtering Records

```python
# Filter by field value
records = table.get_records(
    filter={
        "field": "Department",
        "operator": "=",
        "value": "Engineering"
    }
)

# Multiple conditions
records = table.get_records(
    filter={
        "operator": "and",
        "conditions": [
            {
                "field": "Department",
                "operator": "=",
                "value": "Engineering"
            },
            {
                "field": "Salary",
                "operator": ">",
                "value": 75000
            }
        ]
    }
)
```

### Sorting Records

```python
# Sort by a single field
records = table.get_records(
    order_by="Name"  # Ascending order
)

# Sort by multiple fields
records = table.get_records(
    order_by=[
        {"field": "Department", "direction": "asc"},
        {"field": "Salary", "direction": "desc"}
    ]
)
```

### Pagination

```python
# Get records with pagination
records = table.get_records(
    take=50,    # Number of records to return
    skip=100    # Number of records to skip
)
```

## Advanced Querying

### Using Query Builder

```python
# Create a query using the query builder
query = table.query()\
    .filter("Department", "=", "Engineering")\
    .filter("Salary", ">", 75000)\
    .sort("Name")\
    .paginate(take=10)

# Execute the query
records = table.get_records(query=query)
```

### Search Functionality

```python
# Search across fields
records = table.get_records(
    search=["John", "Name", True]  # [value, field, exact match]
)

# Full-text search
records = table.get_records(
    filter_by_tql="John"  # Teable Query Language
)
```

### Filtering Link Fields

```python
# Filter by linked record candidates
records = table.get_records(
    filter_link_cell_candidate="rec123"
)

# Filter by selected linked records
records = table.get_records(
    filter_link_cell_selected=["rec123", "rec456"]
)
```

### View-Based Queries

```python
# Get records from a specific view
records = table.get_records(
    view_id="viw123"
)

# Get records from a view, ignoring view's query
records = table.get_records(
    view_id="viw123",
    ignore_view_query=True
)
```

## Field Formatting

### Cell Format Options

```python
# Get records with JSON formatting
records = table.get_records(
    cell_format="json"
)

# Get records with text formatting
records = table.get_records(
    cell_format="text"
)
```

### Field Key Types

```python
# Use field names as keys
records = table.get_records(
    field_key_type="name"
)

# Use field IDs as keys
records = table.get_records(
    field_key_type="id"
)
```

## Error Handling

```python
from teable.exceptions import TeableError, ResourceNotFoundError

def safe_get_records(table, **query_params):
    """Safely get records with error handling."""
    try:
        records = table.get_records(**query_params)
        return records
    except ResourceNotFoundError:
        print("Table or view not found")
        return []
    except TeableError as e:
        print(f"Error retrieving records: {e}")
        return []
```

## Performance Optimization

### Projection Optimization

```python
def get_minimal_records(table, required_fields):
    """Get records with only required fields."""
    return table.get_records(
        projection=required_fields,
        cell_format="json"
    )

# Example usage
minimal_records = get_minimal_records(
    table,
    ["Name", "Email"]
)
```

### Batch Processing

```python
def process_records_in_batches(table, batch_size=1000):
    """Process records in batches to manage memory."""
    processed = 0
    while True:
        batch = table.get_records(
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
   - Consider view-based queries
   - Monitor query performance

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
