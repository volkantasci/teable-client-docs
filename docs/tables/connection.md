# Table Connections and Relationships

This guide covers how to establish and manage connections between tables in Teable using the Teable-Client library.

## Creating Reference Fields

### Basic Reference Field

```python
from teable import TeableClient, TeableConfig, FieldType

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Get your tables
employees_table = client.tables.get("employees_table_id")
projects_table = client.tables.get("projects_table_id")

# Create a reference field in the projects table
reference_field = client.fields.create(
    table_id=projects_table.table_id,
    name="Project Manager",
    field_type=FieldType.REFERENCE,
    options={
        "table_id": employees_table.table_id
    }
)
```

## Working with Referenced Records

### Creating Records with References

```python
# Create a project and link it to an existing employee
project = client.records.create(
    table_id=projects_table.table_id,
    fields={
        "Project Name": "New Website",
        "Project Manager": ["employee123"]  # Reference employee by ID
    }
)

# Create with multiple references
project = client.records.create(
    table_id=projects_table.table_id,
    fields={
        "Project Name": "Big Initiative",
        "Team Members": ["employee123", "employee456"]
    }
)
```

### Querying Referenced Records

```python
from teable import FilterOperator

# Get all projects managed by a specific employee
projects = client.records.get_records(
    table_id=projects_table.table_id,
    query={
        "filter": {
            "operator": "and",
            "conditions": [
                {
                    "field": "Project Manager",
                    "operator": FilterOperator.EQUALS,
                    "value": "employee123"
                }
            ]
        }
    }
)
```

### Updating Referenced Records

```python
# Update project manager
client.records.update(
    table_id=projects_table.table_id,
    record_id="project123",
    fields={
        "Project Manager": ["new_manager_id"]
    }
)

# Update team members
client.records.update(
    table_id=projects_table.table_id,
    record_id="project123",
    fields={
        "Team Members": ["member1", "member2", "new_member"]
    }
)
```

## Best Practices

1. **Relationship Planning**
   - Plan your table relationships before implementation
   - Choose appropriate field types
   - Consider data integrity implications
   - Document relationship purposes

2. **Field Naming**
   - Use clear, descriptive names for reference fields
   - Maintain consistent naming conventions
   - Document field relationships
   - Consider field organization

3. **Performance**
   - Use appropriate field types
   - Consider query performance implications
   - Monitor reference field usage
   - Optimize queries when possible

4. **Data Integrity**
   - Validate referenced records exist
   - Handle deleted records appropriately
   - Maintain referential integrity
   - Regular relationship audits

## Error Handling

```python
from teable.exceptions import TeableError, ValidationError

try:
    # Create a reference field
    reference_field = client.fields.create(
        table_id=projects_table.table_id,
        name="Project Manager",
        field_type=FieldType.REFERENCE,
        options={
            "table_id": employees_table.table_id
        }
    )
except ValidationError as e:
    print(f"Invalid field configuration: {e}")
except TeableError as e:
    print(f"Error creating reference: {e}")

# Validate referenced records
try:
    table = client.tables.get(projects_table.table_id)
    table.validate_record_fields({
        "Project Name": "New Project",
        "Project Manager": ["invalid_id"]
    })
except ValidationError as e:
    print(f"Invalid reference: {e}")
```

## Next Steps

After setting up table connections, you can:

- [Create and manage records](../records/create.md)
- [Set up views](../views/creation.md)
- [Configure data validation](../fields/validation.md)
- [Work with bulk operations](bulk-operations.md)
