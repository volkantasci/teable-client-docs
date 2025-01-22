# Creating and Managing Views

This guide covers how to create and manage views in Teable tables.

## Creating Views

### Basic View Creation

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Create a grid view
view = client.views.create(
    table_id="table_id",
    name="My Grid View",
    type="grid"
)

print(f"Created view: {view.view_id}")
```

## View Configuration

### Filtering Records

```python
from teable import FilterOperator

# Create view with filters
view = client.views.create(
    table_id="table_id",
    name="Active Projects",
    type="grid",
    filter={
        "operator": "and",
        "conditions": [
            {
                "field": "Status",
                "operator": FilterOperator.EQUALS,
                "value": "Active"
            },
            {
                "field": "Due Date",
                "operator": FilterOperator.GREATER_THAN,
                "value": "2023-01-01"
            }
        ]
    }
)
```

### Sorting Records

```python
from teable import SortDirection

# Create view with sorting
view = client.views.create(
    table_id="table_id",
    name="Projects by Priority",
    type="grid",
    sort=[
        {
            "field": "Priority",
            "direction": SortDirection.DESCENDING
        },
        {
            "field": "Due Date",
            "direction": SortDirection.ASCENDING
        }
    ]
)
```

### Field Visibility

```python
# Create view with specific visible fields
view = client.views.create(
    table_id="table_id",
    name="Simple View",
    type="grid",
    fields={
        "visible": ["Name", "Status", "Due Date"],
        "hidden": ["Internal Notes", "Created By"]
    }
)
```

## Managing Views

### Getting Views

```python
# Get table
table = client.tables.get("table_id")

# Get all views in a table
views = table.views
```

### Updating Views

```python
from teable import FilterOperator

# Update view configuration
view = client.views.update(
    view_id="view_id",
    name="Updated View Name",
    filter={
        "operator": "and",
        "conditions": [
            {
                "field": "Status",
                "operator": FilterOperator.IN,
                "value": ["Active", "In Progress"]
            }
        ]
    }
)
```

### Deleting Views

```python
# Delete a view
client.views.delete(view_id="view_id")
```

## Working with View Data

### Getting Records from a View

```python
# Get records using view's configuration
records = client.records.get_records(
    table_id="table_id",
    view_id="view_id"
)
```

### Pagination in Views

```python
# Get paginated records from a view
records = client.records.get_records(
    table_id="table_id",
    view_id="view_id",
    take=50,    # Number of records to return
    skip=100    # Number of records to skip
)
```

## Best Practices

1. **View Organization**
   - Use clear, descriptive names
   - Group related views together
   - Consider user permissions
   - Document view purposes

2. **Performance**
   - Optimize filters for performance
   - Use appropriate pagination
   - Consider data volume
   - Monitor view usage

3. **User Experience**
   - Configure meaningful defaults
   - Provide clear field visibility
   - Consider sorting and filtering
   - Keep views organized

4. **Maintenance**
   - Regular view audits
   - Clean up unused views
   - Update view configurations
   - Monitor view usage

## Next Steps

- [Table Management](../tables/management.md)
- [Record Operations](../records/create.md)
- [Field Configuration](../fields/configuration.md)
- [Advanced Topics](../advanced/best-practices.md)
