# Quickstart Guide

This guide will help you get started with the Teable Client Library through practical examples.

## Basic Setup

First, import the necessary classes and create a client instance:

```python
from teable import TeableClient, TeableConfig

# Initialize client with API key
client = TeableClient(TeableConfig(
    api_key="your_api_key_here",
    api_url="https://api.teable.io"
))

# Sign in for full access
client.auth.signin(
    email="your-email@example.com",
    password="your-password"
)
```

## Working with Spaces

Spaces are the top-level containers in Teable. Here's how to work with them:

```python
# Create a new space
space = client.spaces.create_space(name="My Project Space")

# List all spaces
spaces = client.spaces.get_spaces()
for space in spaces:
    print(f"Space: {space.name} (ID: {space.space_id})")

# Get a specific space
space = client.spaces.get_space("space_id")

# Update space name
space.update("Updated Space Name")

# Create an invitation link
invitation = space.create_invitation_link(role="editor")
print(f"Invitation URL: {invitation.invite_url}")

# Invite users by email
space.invite_by_email(
    emails=["user@example.com"],
    role="editor"
)
```

## Managing Bases

Bases are containers for tables within a space:

```python
# Create a base in a space
base = space.create_base(name="My Project Base")

# Get all bases in a space
bases = space.get_bases()
for base in bases:
    print(f"Base: {base.name} (ID: {base.base_id})")

# Delete a base
base.delete()
```

## Working with Tables

Tables store your actual data:

```python
# Create a table with fields
table = client.tables.create_table(
    base_id=base.base_id,
    name="Employees",
    db_table_name="employees",
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
            "type": "number",
            "precision": 0
        }
    ]
)

# Get table fields
fields = table.fields
for field in fields:
    print(f"Field: {field.name} ({field.field_type})")
```

## Managing Records

Records are the individual data entries in a table:

```python
# Create a single record
record = table.create_record({
    "Name": "John Doe",
    "Email": "john@example.com",
    "Age": 30
})

# Batch create records
records_data = [
    {"Name": "Alice Smith", "Email": "alice@example.com", "Age": 25},
    {"Name": "Bob Johnson", "Email": "bob@example.com", "Age": 35}
]
batch_result = table.batch_create_records(records_data)
print(f"Created {batch_result.success_count} records")

# Query records with filtering
filter_data = {
    "filterSet": [
        {
            "fieldId": "Age",
            "operator": "isGreaterEqual",
            "value": 30
        }
    ],
    "conjunction": "and"
}

filtered_records = table.get_records(
    filter=filter_data,
    field_key_type="name"
)

# Update a record
updated_record = table.update_record(
    record.record_id,
    {"Age": 31}
)

# Delete a record
table.delete_record(record.record_id)

# Batch delete records
record_ids = [record.record_id for record in filtered_records]
table.batch_delete_records(record_ids)
```

## Working with Views

Views provide different ways to visualize table data:

```python
# Create a view
view = table.create_view({
    "name": "Age Filter View",
    "type": "grid",
    "filter": {
        "filterSet": [
            {
                "fieldId": "Age",
                "operator": "isGreaterEqual",
                "value": 30
            }
        ],
        "conjunction": "and"
    }
})

# Get all views
views = table.views
for view in views:
    print(f"View: {view.name} (Type: {view.view_type})")
```

## Error Handling

Always handle potential errors in your code:

```python
from teable.exceptions import ValidationError, APIError

try:
    # Attempt to create a record with invalid data
    record = table.create_record({
        "Name": "",  # Required field is empty
        "Email": "invalid-email"
    })
except ValidationError as e:
    print(f"Validation error: {str(e)}")
except APIError as e:
    print(f"API error: {str(e)}")
except Exception as e:
    print(f"Unexpected error: {str(e)}")
```

## Best Practices

1. **Authentication**:
   - Always sign in for operations requiring full access
   - Handle authentication errors appropriately

2. **Resource Management**:
   - Use batch operations for multiple records
   - Clean up resources when no longer needed

3. **Error Handling**:
   - Catch specific exceptions
   - Implement proper error recovery

4. **Performance**:
   - Use pagination for large datasets
   - Implement rate limiting in your application

## Next Steps

- Learn about [Authentication](../advanced/authentication.md)
- Explore [Error Handling](../advanced/error-handling.md)
- Review [Best Practices](../advanced/best-practices.md)
- Check the [API Reference](../api/client.md)
