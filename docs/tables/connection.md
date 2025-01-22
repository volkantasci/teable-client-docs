# Table Connections and Relationships

This guide covers how to establish and manage connections between tables in Teable using the Teable-Client library. Table connections allow you to create relationships between different tables and link related data.

## Understanding Table Relationships

Teable supports different types of relationships between tables:

- **One-to-One**: Each record in the first table corresponds to exactly one record in the second table
- **One-to-Many**: Each record in the first table can be linked to multiple records in the second table
- **Many-to-One**: Multiple records in the first table can be linked to a single record in the second table
- **Many-to-Many**: Records in both tables can have multiple connections to each other

## Creating Linked Fields

### Basic Link Field

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Get your tables
employees_table = client.get_table("employees_table_id")
projects_table = client.get_table("projects_table_id")

# Create a link field in the projects table to link to employees
link_field = {
    "name": "Project Manager",
    "type": "link",
    "options": {
        "relationship": "many_to_one",  # Each project has one manager
        "foreignTableId": employees_table.table_id,
        "symmetricFieldId": None  # Optional: ID of the reciprocal field
    }
}

# Add the link field to the projects table
projects_table.create_field(link_field)
```

### Bidirectional Links

Create linked fields that automatically maintain relationships in both directions:

```python
# Create a bidirectional link between employees and projects
manager_field = {
    "name": "Project Manager",
    "type": "link",
    "options": {
        "relationship": "many_to_one",
        "foreignTableId": employees_table.table_id
    }
}

# Create the field and get its ID
manager_link = projects_table.create_field(manager_field)

# Create the reciprocal field in the employees table
managed_projects_field = {
    "name": "Managed Projects",
    "type": "link",
    "options": {
        "relationship": "one_to_many",
        "foreignTableId": projects_table.table_id,
        "symmetricFieldId": manager_link.field_id
    }
}

employees_table.create_field(managed_projects_field)
```

## Working with Linked Records

### Creating Records with Links

```python
# Create a project and link it to an existing employee
project = projects_table.create_record({
    "Project Name": "New Website",
    "Project Manager": ["employee123"]  # Link to employee by ID
})

# Create multiple links
project = projects_table.create_record({
    "Project Name": "Big Initiative",
    "Team Members": ["employee123", "employee456"]  # Many-to-many relationship
})
```

### Querying Linked Records

```python
# Get all projects managed by a specific employee
projects = projects_table.get_records(
    filter={
        "field": "Project Manager",
        "operator": "=",
        "value": "employee123"
    }
)

# Get projects with specific team members
projects = projects_table.get_records(
    filter={
        "field": "Team Members",
        "operator": "contains",
        "value": "employee123"
    }
)
```

### Updating Linked Records

```python
# Update project manager
projects_table.update_record(
    record_id="project123",
    fields={
        "Project Manager": ["new_manager_id"]
    }
)

# Add team members
current_team = project.fields["Team Members"]
projects_table.update_record(
    record_id="project123",
    fields={
        "Team Members": current_team + ["new_member_id"]
    }
)
```

## Advanced Link Features

### Lookup Fields

Create fields that display data from linked records:

```python
# Create a lookup field to show manager's email
lookup_field = {
    "name": "Manager Email",
    "type": "lookup",
    "options": {
        "relationship": "many_to_one",
        "foreignTableId": employees_table.table_id,
        "lookupFieldId": "email_field_id"  # ID of the email field in employees table
    }
}

projects_table.create_field(lookup_field)
```

### Rollup Fields

Create fields that aggregate data from linked records:

```python
# Create a rollup field to count team members
rollup_field = {
    "name": "Team Size",
    "type": "rollup",
    "options": {
        "relationship": "one_to_many",
        "foreignTableId": employees_table.table_id,
        "rollupFunction": "count"
    }
}

projects_table.create_field(rollup_field)
```

## Best Practices

1. **Relationship Planning**
   - Plan your table relationships before implementation
   - Choose appropriate relationship types
   - Consider data integrity implications
   - Document relationship purposes

2. **Field Naming**
   - Use clear, descriptive names for linked fields
   - Maintain consistent naming conventions
   - Consider bidirectional relationship names
   - Document field relationships

3. **Performance**
   - Use appropriate relationship types
   - Consider query performance implications
   - Index frequently queried linked fields
   - Limit unnecessary bidirectional links

4. **Data Integrity**
   - Validate linked records exist
   - Handle deleted records appropriately
   - Maintain referential integrity
   - Regular relationship audits

## Error Handling

```python
from teable.exceptions import TeableError, ValidationError

try:
    # Create a linked field
    link_field = {
        "name": "Project Manager",
        "type": "link",
        "options": {
            "relationship": "many_to_one",
            "foreignTableId": employees_table.table_id
        }
    }
    projects_table.create_field(link_field)
except ValidationError as e:
    print(f"Invalid field configuration: {e}")
except TeableError as e:
    print(f"Error creating link: {e}")

# Validate linked records
try:
    projects_table.validate_record_fields({
        "Project Name": "New Project",
        "Project Manager": ["invalid_id"]
    })
except ValidationError as e:
    print(f"Invalid link reference: {e}")
```

## Next Steps

After setting up table connections, you can:

- [Create and manage records](../records/create.md)
- [Set up views](../views/creation.md)
- [Configure data validation](../fields/validation.md)
- [Work with bulk operations](bulk-operations.md)
