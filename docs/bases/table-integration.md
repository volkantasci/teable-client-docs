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

# Get a base
base = client.get_base("base123")

# Create a table
table = base.create_table(
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
        "type": "text",
        "required": True
    },
    {
        "name": "Start Date",
        "type": "date"
    },
    {
        "name": "Status",
        "type": "single_select",
        "options": {
            "choices": [
                {"name": "Planning"},
                {"name": "In Progress"},
                {"name": "Completed"}
            ]
        }
    }
]

# Create table with fields
table = base.create_table(
    name="Projects",
    fields=fields
)
```

## Managing Tables

### Listing Tables

```python
# Get all tables in a base
tables = base.get_tables()

for table in tables:
    print(f"Table: {table.name}")
    print(f"ID: {table.table_id}")
    print("Fields:")
    for field in table.fields:
        print(f"  - {field.name} ({field.type})")
    print("---")
```

### Updating Tables

```python
# Update table properties
table.update(
    name="Active Projects",
    description="Track ongoing and planned projects"
)
```

## Table Relationships

### Creating Linked Fields

```python
# Create a linked field to another table
linked_field = {
    "name": "Project Manager",
    "type": "link",
    "options": {
        "relationship": "many_to_one",
        "foreignTableId": "employees_table_id",
        "symmetricFieldId": "managed_projects"  # Optional
    }
}

table.create_field(linked_field)
```

### Managing Relationships

```python
# Update relationship configuration
table.update_field(
    field_id="field123",
    options={
        "relationship": "many_to_many",
        "symmetricFieldId": "new_symmetric_field"
    }
)
```

## Views Integration

### Creating Views

```python
# Create a view for the table
view = table.create_view({
    "name": "Active Projects",
    "type": "grid",
    "filter": {
        "operator": "and",
        "conditions": [
            {
                "field": "Status",
                "operator": "=",
                "value": "In Progress"
            }
        ]
    },
    "sort": [
        {
            "field": "Start Date",
            "direction": "desc"
        }
    ]
})
```

### Managing Views

```python
# List views
views = table.get_views()

# Update view configuration
view.update({
    "name": "Current Projects",
    "filter": {
        "operator": "and",
        "conditions": [
            {
                "field": "Status",
                "operator": "in",
                "value": ["Planning", "In Progress"]
            }
        ]
    }
})
```

## Data Integration

### Importing Data

```python
# Import data from CSV
with open('projects.csv', 'rb') as file:
    table.import_csv(
        file,
        chunk_size=1000,  # Optional: Process in chunks
        field_mapping={    # Optional: Map CSV columns to fields
            "Project": "Project Name",
            "Start": "Start Date"
        }
    )
```

### Exporting Data

```python
# Export table data
export = table.export_csv()

# Save to file
with open('projects_export.csv', 'wb') as file:
    file.write(export.content)
```

## Best Practices

1. **Table Design**
   - Plan your table structure before creation
   - Use appropriate field types for data
   - Consider relationships between tables
   - Document table purposes and relationships

2. **Field Management**
   - Use clear, descriptive field names
   - Set appropriate field constraints
   - Consider field dependencies
   - Document field usage and requirements

3. **Data Integration**
   - Validate data before import
   - Use appropriate chunk sizes for large imports
   - Maintain data consistency across tables
   - Regular data validation and cleanup

4. **View Organization**
   - Create views for common use cases
   - Use consistent naming conventions
   - Document view purposes
   - Regular view maintenance

## Error Handling

```python
from teable.exceptions import TeableError, ValidationError

try:
    # Create table with fields
    table = base.create_table(
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
   records = [{"Project Name": f"Project {i}"} for i in range(100)]
   table.batch_create_records(records)
   
   # Avoid - Individual record creation
   for i in range(100):
       table.create_record({"Project Name": f"Project {i}"})
   ```

2. **Field Optimization**
   ```python
   # Good - Create all fields at once
   table = base.create_table(
       name="Projects",
       fields=all_fields
   )
   
   # Avoid - Creating fields one by one
   table = base.create_table(name="Projects")
   for field in fields:
       table.create_field(field)
   ```

## Next Steps

After integrating tables with your base, you can:

- [Create and manage records](../records/create.md)
- [Set up views](../views/creation.md)
- [Configure field types](../fields/configuration.md)
