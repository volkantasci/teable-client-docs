# Table Management

This guide covers how to manage tables in Teable.

## Basic Table Management

### Getting Tables

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Get a specific table
table = client.tables.get("table_id")

# Get all tables in a space
tables = client.tables.get_all(space_id="space123")

# Process tables
for table in tables:
    print(f"Table ID: {table.table_id}")
```

### Updating Tables

```python
# Update table properties
table = client.tables.update(
    table_id="table_id",
    name="New Table Name",
    description="Updated description"
)
```

## Field Management

### Managing Fields

```python
from teable import FieldType

# Get all fields
fields = table.fields

# Create new field
new_field = client.fields.create(
    table_id=table.table_id,
    name="Status",
    field_type=FieldType.SINGLE_SELECT,
    options={
        "choices": ["Active", "Inactive"]
    }
)

# Update field
field = client.fields.update(
    field_id="field_id",
    name="New Name"
)

# Delete field
client.fields.delete(field_id="field_id")
```

### Managing Views

```python
from teable import FilterOperator, SortDirection

# Get all views
views = table.views

# Create new view
new_view = client.views.create(
    table_id=table.table_id,
    name="Custom View",
    type="grid",
    filter={
        "operator": "and",
        "conditions": [
            {
                "field": "Status",
                "operator": FilterOperator.EQUALS,
                "value": "Active"
            }
        ]
    }
)

# Update view
view = client.views.update(
    view_id="view_id",
    name="Updated View",
    sort=[
        {
            "field": "CreatedTime",
            "direction": SortDirection.DESCENDING
        }
    ]
)

# Delete view
client.views.delete(view_id="view_id")
```

## Data Management

### Record Operations

```python
from teable import FilterOperator

# Get records with filtering
records = client.records.get_records(
    table_id=table.table_id,
    query={
        "filter": {
            "operator": "and",
            "conditions": [
                {
                    "field": "Status",
                    "operator": FilterOperator.EQUALS,
                    "value": "Active"
                }
            ]
        }
    }
)
```

### Batch Operations

```python
# Batch create records
records = [
    {"Name": f"Record {i}"} for i in range(100)
]
result = client.records.batch_create_records(
    table_id=table.table_id,
    records=records
)

# Batch update records
updates = [
    {
        "record_id": f"rec{i}",
        "fields": {"Status": "Updated"}
    }
    for i in range(100)
]
result = client.records.batch_update_records(
    table_id=table.table_id,
    records=updates
)

# Batch delete records
record_ids = [f"rec{i}" for i in range(100)]
result = client.records.batch_delete_records(
    table_id=table.table_id,
    record_ids=record_ids
)
```

## Best Practices

1. **Table Design**
   - Plan table structure carefully
   - Use appropriate field types
   - Consider relationships
   - Document table purpose

2. **Performance**
   - Use batch operations
   - Implement pagination
   - Optimize queries
   - Monitor table size

3. **Data Integrity**
   - Validate data before operations
   - Regular data cleanup
   - Backup important data
   - Handle errors appropriately

4. **Organization**
   - Use clear naming conventions
   - Group related tables
   - Maintain documentation
   - Regular maintenance

## Next Steps

- [Table Connections](connection.md)
- [Data Insertion](data-insertion.md)
- [Bulk Operations](bulk-operations.md)
- [Views Creation](../views/creation.md)
