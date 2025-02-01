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

# Get current user info
user = client.auth.get_user()
print(f"Signed in as: {user.email}")
```

## Working with Spaces

Spaces are the top-level containers in Teable:

```python
# Create a new space
space = client.spaces.create_space(name="My Project Space")

# List all spaces
spaces = client.spaces.get_spaces()
for space in spaces:
    print(f"Space: {space.name} (ID: {space.space_id})")

# Get a specific space
space = client.spaces.get_space("space_id")

# Create an invitation link
invitation = space.create_invitation_link(role="editor")
print(f"Invitation URL: {invitation.invite_url}")

# Invite users by email
space.invite_by_email(
    emails=["user@example.com"],
    role="editor"
)
```

## Managing Tables

Tables store your data with specific field types and configurations:

```python
# Create a table with fields
table = client.tables.create_table(
    base_id="base_id",
    name="Employees",
    db_table_name="employees",  # Must be 1-63 chars, start with letter
    description="Employee records",
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

Records are individual data entries in a table:

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
batch_result = table.batch_create_records(
    records_data,
    field_key_type="name",  # Use field names instead of IDs
    typecast=True  # Enable automatic type conversion
)
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

# Get records with various parameters
records = table.get_records(
    field_key_type="name",
    filter=filter_data,
    take=100,  # Pagination: records per page
    skip=0,    # Pagination: records to skip
    cell_format="json"  # Response format (json or text)
)

# Search records
search_params = [{
    "value": "John",
    "field": "Name",
    "exact": True
}]

search_results = table.get_records(
    search=search_params,
    field_key_type="name"
)

# Update records
updated_record = table.update_record(
    record.record_id,
    {"Age": 31}
)

# Batch update records
updates = [
    {
        "id": record.record_id,
        "fields": {"Age": record.fields["Age"] + 1}
    }
    for record in records
]

updated_records = table.batch_update_records(
    updates,
    field_key_type="name",
    typecast=True
)

# Delete records
table.delete_record(record.record_id)

# Batch delete records
record_ids = [record.record_id for record in records]
table.batch_delete_records(record_ids)
```

## Working with Views

Views provide different ways to visualize and filter table data:

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

# Get default view ID
default_view_id = client.tables.get_table_default_view_id(
    base_id="base_id",
    table_id=table.table_id
)
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
    print(f"Status code: {e.status_code}")
    print(f"Error details: {e.details}")
except Exception as e:
    print(f"Unexpected error: {str(e)}")
```

## Resource Management

Always clean up resources when done:

```python
try:
    # Perform operations
    space = client.spaces.create_space(name="Temporary Space")
    base = space.create_base(name="Temporary Base")
    # ... more operations ...
finally:
    # Clean up
    if base:
        base.delete()
    if space:
        client.spaces.permanently_delete_space(space.space_id)
    # Sign out
    client.auth.signout()
```

## Best Practices

1. **Authentication**:
   - Always sign in for operations requiring full access
   - Handle authentication errors appropriately
   - Sign out when done

2. **Resource Management**:
   - Use batch operations for multiple records
   - Clean up temporary resources
   - Implement proper error handling

3. **Performance**:
   - Use pagination for large datasets
   - Implement caching where appropriate
   - Use field projections to limit data transfer

4. **Error Handling**:
   - Catch specific exceptions
   - Implement retry logic for transient failures
   - Log errors appropriately

## Next Steps

- Learn about [Authentication](../advanced/authentication.md)
- Explore [Error Handling](../advanced/error-handling.md)
- Review [Best Practices](../advanced/best-practices.md)
- Check the [API Reference](../api/client.md)
