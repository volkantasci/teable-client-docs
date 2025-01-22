# Creating and Managing Views

This guide covers how to create and manage views in Teable tables. Views provide different ways to visualize and interact with your table data.

## Creating Views

### Basic View Creation

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Get your table
table = client.get_table("table_id")

# Create a basic grid view
view = table.create_view({
    "name": "My Grid View",
    "type": "grid"
})

print(f"Created view: {view.view_id}")
```

### View Types

```python
# Grid View
grid_view = table.create_view({
    "name": "Grid View",
    "type": "grid"
})

# Calendar View
calendar_view = table.create_view({
    "name": "Calendar View",
    "type": "calendar",
    "options": {
        "dateField": "due_date"  # Field to use for calendar
    }
})

# Kanban View
kanban_view = table.create_view({
    "name": "Project Status",
    "type": "kanban",
    "options": {
        "groupField": "status"  # Field to group by
    }
})

# Gallery View
gallery_view = table.create_view({
    "name": "Image Gallery",
    "type": "gallery",
    "options": {
        "coverField": "image"  # Field to use as cover
    }
})
```

## View Configuration

### Filtering Records

```python
# Create view with filters
view = table.create_view({
    "name": "Active Projects",
    "type": "grid",
    "filter": {
        "operator": "and",
        "conditions": [
            {
                "field": "Status",
                "operator": "=",
                "value": "Active"
            },
            {
                "field": "Due Date",
                "operator": ">",
                "value": "2023-01-01"
            }
        ]
    }
})
```

### Sorting Records

```python
# Create view with sorting
view = table.create_view({
    "name": "Projects by Priority",
    "type": "grid",
    "sort": [
        {
            "field": "Priority",
            "direction": "desc"
        },
        {
            "field": "Due Date",
            "direction": "asc"
        }
    ]
})
```

### Field Visibility

```python
# Create view with specific visible fields
view = table.create_view({
    "name": "Simple View",
    "type": "grid",
    "fields": {
        "visible": ["Name", "Status", "Due Date"],
        "hidden": ["Internal Notes", "Created By"]
    }
})
```

## Managing Views

### Getting Views

```python
# Get all views in a table
views = table.views

# Get a specific view
view = table.get_view("view_id")

# Access view properties
print(f"View Name: {view.name}")
print(f"View Type: {view.type}")
print(f"View Filter: {view.filter}")
```

### Updating Views

```python
# Update view configuration
view.update({
    "name": "Updated View Name",
    "filter": {
        "operator": "and",
        "conditions": [
            {
                "field": "Status",
                "operator": "in",
                "value": ["Active", "In Progress"]
            }
        ]
    }
})
```

### Deleting Views

```python
# Delete a view
success = view.delete()
if success:
    print("View deleted successfully")
```

## Working with View Data

### Getting Records from a View

```python
# Get records using view's configuration
records = table.get_records(view_id=view.view_id)

# Get records ignoring view's query
records = table.get_records(
    view_id=view.view_id,
    ignore_view_query=True
)
```

### Pagination in Views

```python
# Get paginated records from a view
records = table.get_records(
    view_id=view.view_id,
    take=50,    # Number of records to return
    skip=100    # Number of records to skip
)
```

## View Options

### Grid View Options

```python
grid_view = table.create_view({
    "name": "Custom Grid",
    "type": "grid",
    "options": {
        "frozenColumns": 2,      # Number of frozen columns
        "rowHeight": "medium",   # Row height: "short", "medium", "tall"
        "showSystemFields": False  # Show/hide system fields
    }
})
```

### Calendar View Options

```python
calendar_view = table.create_view({
    "name": "Event Calendar",
    "type": "calendar",
    "options": {
        "dateField": "event_date",
        "titleField": "event_name",
        "descriptionField": "details",
        "colorField": "category"
    }
})
```

### Kanban View Options

```python
kanban_view = table.create_view({
    "name": "Task Board",
    "type": "kanban",
    "options": {
        "groupField": "status",
        "stackField": "assignee",
        "hideEmptyGroups": True,
        "cardFields": ["title", "priority", "due_date"]
    }
})
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
   - Cache view configurations

3. **User Experience**
   - Choose appropriate view types
   - Configure meaningful defaults
   - Provide clear field visibility
   - Consider sorting and filtering

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
