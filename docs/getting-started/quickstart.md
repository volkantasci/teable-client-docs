# Quick Start Guide

This guide will help you get started with the Teable-Client library by walking through common operations and basic usage patterns.

## Basic Setup

First, import the necessary classes and initialize the client:

```python
from teable import TeableClient, TeableConfig

# Initialize the client
config = TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
)
client = TeableClient(config)
```

## Working with Tables

### Accessing a Table

```python
# Get a table by ID
table = client.get_table("table_id")

# List all tables in a base
base_tables = client.list_tables("base_id")
```

### Creating Records

```python
# Create a single record
record = table.create_record({
    "name": "John Doe",
    "email": "john@example.com",
    "age": 30
})
print(f"Created record ID: {record.id}")

# Batch create multiple records
records_data = [
    {"name": "Alice", "age": 28},
    {"name": "Bob", "age": 32}
]
batch_result = table.batch_create_records(records_data)
print(f"Created {batch_result.success_count} records")
```

### Querying Records

```python
# Simple query
records = table.get_records()

# Advanced query with filters and sorting
query = table.query()\
    .filter("age", ">", 25)\
    .filter("department", "=", "Engineering")\
    .sort("name")\
    .paginate(take=10)
    
records = table.get_records(query)

# Process query results
for record in records:
    print(f"Name: {record.name}, Age: {record.age}")
```

## Working with Fields

### Getting Field Information

```python
# List all fields in a table
fields = table.fields

# Access field properties
for field in fields:
    print(f"{field.name} ({field.field_type})")
    if field.is_required:
        print("  Required field")
    if field.is_primary:
        print("  Primary field")
```

### Field Validation

```python
from teable import ValidationError

# Validate field values
field = table.get_field("field_id")
try:
    field.validate_value("test@example.com")
    print("Value is valid")
except ValidationError as e:
    print(f"Invalid value: {e}")
```

## Working with Views

### Creating and Managing Views

```python
# Create a new view
view = table.create_view({
    "name": "Engineering Team",
    "type": "grid",
    "filter": {
        "operator": "and",
        "conditions": [
            {
                "field": "department",
                "operator": "=",
                "value": "Engineering"
            }
        ]
    }
})

# Get records from a specific view
view_records = table.get_records(view_id=view.id)
```

## Data Aggregation

### Basic Aggregation

```python
# Get aggregated data
result = client.aggregation.get_aggregation(
    table_id="table_id",
    group_by=["category"],
    metrics=[
        {"field": "amount", "function": "sum"},
        {"field": "quantity", "function": "avg"}
    ]
)

# Process aggregation results
for group in result.groups:
    print(f"Category: {group.category}")
    print(f"Total Amount: {group.metrics.amount_sum}")
    print(f"Average Quantity: {group.metrics.quantity_avg}")
```

### Calendar View

```python
# Get calendar data
calendar = client.aggregation.get_calendar_daily_collection(
    table_id="table_id",
    date_field="due_date"
)

# Process calendar data
for date, events in calendar.items():
    print(f"Date: {date}")
    for event in events:
        print(f"  - {event.title}")
```

## Error Handling

```python
from teable.exceptions import TeableError, ValidationError, AuthenticationError

try:
    # Attempt an operation
    record = table.create_record({"invalid": "data"})
except ValidationError as e:
    print(f"Validation error: {e}")
except AuthenticationError as e:
    print(f"Authentication failed: {e}")
except TeableError as e:
    print(f"General error: {e}")
```

## Best Practices

1. **Use Batch Operations**: When working with multiple records, use batch operations instead of individual calls:
   ```python
   # Good - batch operation
   records = [{"name": f"User {i}"} for i in range(100)]
   result = table.batch_create_records(records)
   
   # Avoid - individual calls
   for i in range(100):
       table.create_record({"name": f"User {i}"})
   ```

2. **Handle Rate Limits**: The client automatically handles rate limiting, but you should still structure your code to work efficiently:
   ```python
   # Process large datasets in chunks
   chunk_size = 1000
   for i in range(0, len(records), chunk_size):
       chunk = records[i:i + chunk_size]
       table.batch_create_records(chunk)
   ```

3. **Use Query Builder**: Leverage the query builder for complex queries instead of filtering in memory:
   ```python
   # Good - server-side filtering
   query = table.query()\
       .filter("age", ">", 25)\
       .sort("name")\
       .paginate(take=100)
   
   # Avoid - client-side filtering
   all_records = table.get_records()
   filtered = [r for r in all_records if r.age > 25]
   ```

## Next Steps

Now that you're familiar with the basics, explore the following topics for more advanced usage:

- [Space Management](../spaces/creation.md)
- [Base Operations](../bases/creation.md)
- [Table Operations](../tables/creation.md)
- [Record Management](../records/create.md)
- [Advanced Topics](../advanced/authentication.md)
