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

## Working with Spaces and Bases

### Managing Spaces

```python
# Get all spaces
spaces = client.spaces.get_all()
print(f"Found {len(spaces)} spaces")

# Create a new space
space = client.spaces.create(name="My New Space")
print(f"Created space: {space.name}")

# Get a specific space
space = client.spaces.get("space_id")
```

### Managing Bases

```python
# Create a base in a space
base = client.tables.create(
    space_id="space_id",
    name="My New Base"
)
print(f"Created base: {base.name}")

# Get a specific base
base = client.tables.get("base_id")
```

## Working with Tables

### Managing Tables

```python
# Create a table in a base
table = client.tables.create(
    base_id="base_id",
    name="Employees",
    fields=[
        {
            "name": "Name",
            "type": "singleLineText",
            "required": True
        },
        {
            "name": "Email",
            "type": "singleLineText"
        },
        {
            "name": "Age",
            "type": "number"
        }
    ]
)
print(f"Created table: {table.name}")

# Get a specific table
table = client.tables.get("table_id")
```

### Working with Records

```python
# Create a single record
record = client.records.create(
    table_id="table_id",
    fields={
        "Name": "John Doe",
        "Email": "john@example.com",
        "Age": 30
    }
)
print(f"Created record ID: {record.record_id}")

# Batch create multiple records
records_data = [
    {"Name": "Alice", "Age": 28},
    {"Name": "Bob", "Age": 32}
]
batch_result = client.records.batch_create_records(
    table_id="table_id",
    records=records_data
)
print(f"Created {batch_result.success_count} records")

# Get records with filtering and sorting
records = client.tables.get_records(
    table_id="table_id",
    filter={
        "operator": "and",
        "conditions": [
            {
                "field": "Age",
                "operator": ">",
                "value": 25
            },
            {
                "field": "Department",
                "operator": "=",
                "value": "Engineering"
            }
        ]
    },
    order_by="Name"
)

# Process records
for record in records:
    print(f"Name: {record.fields['Name']}, Age: {record.fields['Age']}")
```

## Working with Fields

### Managing Fields

```python
# Get all fields in a table
fields = table.fields

# Access field properties
for field in fields:
    print(f"{field.name} ({field.field_type})")
    if field.is_required:
        print("  Required field")
    if field.is_primary:
        print("  Primary field")

# Get a specific field
field = table.get_field("field_id")
```

### Field Validation

```python
from teable.exceptions import ValidationError

# Validate field values
try:
    field.validate_value("test@example.com")
    print("Value is valid")
except ValidationError as e:
    print(f"Invalid value: {e}")
```

## Working with Views

### Managing Views

```python
# Create a new view
view = client.views.create(
    table_id="table_id",
    name="Engineering Team",
    type="grid",
    filter={
        "operator": "and",
        "conditions": [
            {
                "field": "Department",
                "operator": "=",
                "value": "Engineering"
            }
        ]
    }
)

# Get records from a specific view
records = client.tables.get_records(
    table_id="table_id",
    view_id=view.view_id
)
```

## Using Query Builder

```python
# Create a query builder
query = table.query()

# Add filters
query.filter(
    field="Age",
    operator=FilterOperator.GREATER_THAN,
    value=25
)
query.filter(
    field="Department",
    operator=FilterOperator.EQUALS,
    value="Engineering"
)

# Add sorting
query.sort(
    field="Name",
    direction=SortDirection.ASCENDING
)

# Add pagination
query.paginate(take=10, skip=0)

# Execute the query
records = table.get_records(query=query.build())
```

## Error Handling

```python
from teable.exceptions import (
    TeableError,
    ValidationError,
    AuthenticationError,
    ResourceNotFoundError,
    RateLimitError
)

try:
    # Attempt an operation
    record = client.records.create(
        table_id="table_id",
        fields={"invalid": "data"}
    )
except ValidationError as e:
    print(f"Validation error: {e}")
except AuthenticationError as e:
    print(f"Authentication failed: {e}")
except ResourceNotFoundError as e:
    print(f"Resource not found: {e}")
except RateLimitError as e:
    print(f"Rate limit exceeded: {e}")
except TeableError as e:
    print(f"General error: {e}")
```

## Best Practices

1. **Use Batch Operations**: When working with multiple records, use batch operations instead of individual calls:
   ```python
   # Good - batch operation
   records = [{"Name": f"User {i}"} for i in range(100)]
   result = client.records.batch_create_records(
       table_id="table_id",
       records=records
   )
   
   # Avoid - individual calls
   for i in range(100):
       client.records.create(
           table_id="table_id",
           fields={"Name": f"User {i}"}
       )
   ```

2. **Handle Rate Limits**: The client automatically handles rate limiting, but you should still structure your code to work efficiently:
   ```python
   # Process large datasets in chunks
   chunk_size = 1000
   for i in range(0, len(records), chunk_size):
       chunk = records[i:i + chunk_size]
       client.records.batch_create_records(
           table_id="table_id",
           records=chunk
       )
   ```

3. **Use Query Builder**: Leverage the query builder for complex queries instead of filtering in memory:
   ```python
   # Good - server-side filtering
   query = table.query()\
       .filter("Age", FilterOperator.GREATER_THAN, 25)\
       .sort("Name", SortDirection.ASCENDING)\
       .paginate(take=100)
   records = table.get_records(query=query.build())
   
   # Avoid - client-side filtering
   all_records = table.get_records()
   filtered = [r for r in all_records if r.fields['Age'] > 25]
   ```

## Next Steps

Now that you're familiar with the basics, explore the following topics for more advanced usage:

- [Space Management](../spaces/creation.md)
- [Base Operations](../bases/creation.md)
- [Table Operations](../tables/creation.md)
- [Record Management](../records/create.md)
- [Advanced Topics](../advanced/authentication.md)
