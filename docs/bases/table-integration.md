# Table Integration with Bases

This guide covers how to integrate and manage tables within bases using the Teable-Client library. Tables are the fundamental data structures within bases that store and organize your information.

## Table Structure in Bases

A base can contain multiple tables, and these tables can be interconnected through relationships. Each table consists of:

- Fields (columns) that define the data structure
- Records (rows) that contain the actual data
- Views that provide different ways to visualize and interact with the data

## Creating Tables in a Base

### Basic Table Creation

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Create a table in a base
table = client.tables.create(
    base_id="base123",
    name="Projects",
    description="Track all company projects"
)
```

### Creating Tables with Fields

```python
# Define fields for the table
fields = [
    {
        "name": "Project Name",
        "type": "singleLineText",
        "required": True
    },
    {
        "name": "Start Date",
        "type": "date"
    },
    {
        "name": "Status",
        "type": "singleSelect",
        "options": {
            "choices": ["Planning", "In Progress", "Completed"]
        }
    }
]

# Create table with fields
table = client.tables.create(
    base_id="base123",
    name="Projects",
    fields=fields
)
```

## Managing Tables

### Getting Table Information

```python
# Get a specific table
table = client.tables.get("table123")

# Access table properties
print(f"Table ID: {table.table_id}")
print(f"Description: {table.description}")

# Access fields
for field in table.fields:
    print(f"Field: {field.field_id}")
    print(f"Type: {field.field_type}")
    print(f"Required: {field.is_required}")
```

### Updating Tables

```python
# Update table properties
table = client.tables.update(
    table_id="table123",
    name="Active Projects",
    description="Track ongoing and planned projects"
)
```

## Field Management

### Creating Fields

```python
from teable import FieldType

# Create a single select field
field = client.fields.create(
    table_id="table123",
    name="Priority",
    field_type=FieldType.SINGLE_SELECT,
    options={
        "choices": ["High", "Medium", "Low"]
    }
)

# Create a number field
field = client.fields.create(
    table_id="table123",
    name="Budget",
    field_type=FieldType.NUMBER,
    options={
        "precision": 2,
        "format": "currency"
    }
)
```

### Updating Fields

```python
# Update field properties
field = client.fields.update(
    field_id="field123",
    name="Project Priority",
    options={
        "choices": ["Critical", "High", "Medium", "Low"]
    }
)
```

## View Management

### Creating Views

```python
from teable import FilterOperator, SortDirection

# Create a view
view = client.views.create(
    table_id="table123",
    name="Active Projects",
    type="grid",
    filter={
        "operator": "and",
        "conditions": [
            {
                "field": "Status",
                "operator": FilterOperator.EQUALS,
                "value": "In Progress"
            }
        ]
    },
    sort=[
        {
            "field": "Start Date",
            "direction": SortDirection.DESCENDING
        }
    ]
)
```

### Managing Views

```python
# Get views from table
views = table.views

# Update view configuration
view = client.views.update(
    view_id="view123",
    name="Current Projects",
    filter={
        "operator": "and",
        "conditions": [
            {
                "field": "Status",
                "operator": FilterOperator.IN,
                "value": ["Planning", "In Progress"]
            }
        ]
    }
)
```

## Best Practices

1. **Table Design**
   - Plan your table structure before creation
   - Use appropriate field types for data
   - Consider relationships between tables
   - Document table purposes

2. **Field Management**
   - Use clear, descriptive field names
   - Set appropriate field constraints
   - Consider field dependencies
   - Document field usage

3. **View Organization**
   - Create views for common use cases
   - Use consistent naming conventions
   - Document view purposes
   - Regular view maintenance

## Error Handling

```python
from teable.exceptions import TeableError, ValidationError

try:
    # Create table with fields
    table = client.tables.create(
        base_id="base123",
        name="Projects",
        fields=fields
    )
except ValidationError as e:
    print(f"Invalid table configuration: {e}")
except TeableError as e:
    print(f"Error creating table: {e}")
```

## Performance Considerations

1. **Batch Operations**
   ```python
   # Good - Batch create records
   records = [
       {"Project Name": f"Project {i}"}
       for i in range(100)
   ]
   client.records.batch_create_records(
       table_id="table123",
       records=records
   )
   
   # Avoid - Individual record creation
   for i in range(100):
       client.records.create(
           table_id="table123",
           fields={"Project Name": f"Project {i}"}
       )
   ```

2. **Field Creation**
   ```python
   # Good - Create table with all fields
   table = client.tables.create(
       base_id="base123",
       name="Projects",
       fields=all_fields
   )
   
   # Avoid - Creating fields one by one
   table = client.tables.create(
       base_id="base123",
       name="Projects"
   )
   for field in fields:
       client.fields.create(
           table_id=table.table_id,
           **field
       )
   ```

## Next Steps

After integrating tables with your base, you can:

- [Create and manage records](../records/create.md)
- [Set up views](../views/creation.md)
- [Configure field types](../fields/configuration.md)
