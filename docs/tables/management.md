# Table Management

This guide covers how to manage tables in Teable, including listing, updating, and organizing tables.

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
table = client.get_table("table_id")

# Get all tables in a base
base = client.get_base("base_id")
tables = base.get_tables()

# Process tables
for table in tables:
    print(f"Table: {table.name}")
    print(f"ID: {table.table_id}")
    print(f"Description: {table.description}")
    print("---")
```

### Updating Tables

```python
# Update table properties
table.update(
    name="New Table Name",
    description="Updated description"
)

# Update table configuration
table.update_config({
    "displayMode": "grid",
    "showSystemFields": False,
    "defaultSortField": "created_time",
    "defaultSortDirection": "desc"
})
```

## Table Organization

### Managing Fields

```python
# Get all fields
fields = table.fields

# Get specific field
field = table.get_field("field_id")

# Create new field
new_field = table.create_field({
    "name": "Status",
    "type": "single_select",
    "options": {
        "choices": [
            {"name": "Active"},
            {"name": "Inactive"}
        ]
    }
})

# Update field
field.update({
    "name": "New Name",
    "description": "Updated description"
})

# Delete field
field.delete()
```

### Managing Views

```python
# Get all views
views = table.views

# Get specific view
view = table.get_view("view_id")

# Create new view
new_view = table.create_view({
    "name": "Custom View",
    "type": "grid",
    "filter": {
        "operator": "and",
        "conditions": [
            {
                "field": "Status",
                "operator": "=",
                "value": "Active"
            }
        ]
    }
})

# Update view
view.update({
    "name": "Updated View",
    "sort": [
        {
            "field": "created_time",
            "direction": "desc"
        }
    ]
})

# Delete view
view.delete()
```

## Data Management

### Record Operations

```python
# Get record count
count = table.get_record_count()

# Get records with filtering
records = table.get_records(
    filter={
        "field": "Status",
        "operator": "=",
        "value": "Active"
    }
)

# Export data
table.export_csv("output.csv")

# Import data
with open("input.csv", "rb") as file:
    table.import_csv(file)
```

### Batch Operations

```python
# Batch create records
records = [
    {"Name": f"Record {i}"} for i in range(100)
]
result = table.batch_create_records(records)

# Batch update records
updates = [
    {
        "recordId": f"rec{i}",
        "fields": {"Status": "Updated"}
    }
    for i in range(100)
]
result = table.batch_update_records(updates)

# Batch delete records
record_ids = [f"rec{i}" for i in range(100)]
result = table.batch_delete_records(record_ids)
```

## Table Relationships

### Managing Links

```python
# Create linked field
linked_field = table.create_field({
    "name": "Related Items",
    "type": "link",
    "options": {
        "relationship": "many_to_many",
        "foreignTableId": "other_table_id"
    }
})

# Update relationship
linked_field.update({
    "options": {
        "relationship": "one_to_many"
    }
})
```

### Querying Related Records

```python
# Get records with related data
records = table.get_records(
    filter={
        "field": "Related Items",
        "operator": "has",
        "value": ["rec123"]
    }
)

# Query through relationship
related_records = table.query(
    "SELECT t1.*, t2.name FROM current_table t1 " +
    "JOIN other_table t2 ON t1.related_items = t2.id"
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
   - Validate data before import
   - Maintain relationships
   - Regular data cleanup
   - Backup important data

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
