# Creating and Managing Bases

A base in Teable is a container for tables and other resources within a space. This guide covers how to create and manage bases using the Teable-Client library.

## Creating a Base

### Basic Base Creation

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Create a base in a space
base = client.tables.create(
    space_id="space123",
    name="Project Tracker",
    icon="📊"  # Optional emoji or icon identifier
)

print(f"Created base: {base.name} (ID: {base.base_id})")
```

## Duplicating Bases

You can create a copy of an existing base:

```python
# Duplicate a base
duplicated_base = client.tables.duplicate(
    from_base_id="base123",
    space_id="space123",
    name="Project Tracker Copy",
    with_records=True  # Include existing records in the copy
)
```

## Base Configuration

### Updating Base Information

```python
# Get a base
base = client.tables.get("base123")

# Update base information
updated_base = base.update(
    name="New Project Tracker",
    icon="🚀"
)
```

### Managing Base Order

You can arrange bases within a space:

```python
from teable import Position

# Update base position
base.update_order(
    anchor_id="base456",  # The base to position relative to
    position=Position.AFTER  # or Position.BEFORE
)
```

## Base Permissions

### Checking Permissions

```python
# Get permissions for a base
permissions = base.get_permissions()

# Check specific permissions
for permission, allowed in permissions.items():
    print(f"{permission}: {'Allowed' if allowed else 'Not Allowed'}")
```

## Base Collaborators

### Adding Collaborators

```python
# Add collaborators to a base
collaborators = [
    {
        "principalId": "user123",
        "principalType": "user"
    },
    {
        "principalId": "dept456",
        "principalType": "department"
    }
]

base.add_collaborators(
    collaborators=collaborators,
    role="editor"  # Options: creator, editor, commenter, viewer
)
```

### Managing Collaborators

```python
from teable.models.collaborator import PrincipalType

# List collaborators
collaborators, total = base.get_collaborators(
    include_system=True,
    take=100,
    search="john"
)

# Update collaborator role
base.update_collaborator(
    principal_id="user123",
    principal_type=PrincipalType.USER,
    role="editor"
)

# Remove collaborator
base.delete_collaborator(
    principal_id="user123",
    principal_type=PrincipalType.USER
)
```

## Base Invitations

### Creating Invitation Links

```python
# Create an invitation link
invitation = base.create_invitation_link(role="editor")

# List all invitation links
invitations = base.get_invitation_links()
```

### Sending Email Invitations

```python
# Send invitation emails
emails = ["user1@example.com", "user2@example.com"]
result = base.send_email_invitations(
    emails=emails,
    role="editor"
)
```

## Base Deletion

### Moving to Trash

```python
# Move base to trash (can be restored later)
base.delete()
```

### Permanent Deletion

```python
# Permanently delete a base (cannot be undone)
base.delete_permanent()
```

!!! warning "Warning"
    Permanent deletion cannot be undone. Make sure to back up any important data before proceeding.

## Best Practices

1. **Base Organization**
   - Use clear, descriptive names for bases
   - Organize bases logically within spaces
   - Use appropriate icons for visual identification
   - Document base purposes and relationships

2. **Access Control**
   - Regularly review collaborator access
   - Use appropriate roles for different user types
   - Clean up unused invitation links
   - Document access policies

3. **Data Management**
   - Consider whether to include records when duplicating
   - Back up important bases before major changes
   - Maintain consistent naming conventions

## Error Handling

```python
from teable.exceptions import TeableError, ValidationError

try:
    base = client.tables.create(
        space_id="space123",
        name="New Project"
    )
except ValidationError as e:
    print(f"Invalid base configuration: {e}")
except TeableError as e:
    print(f"Error creating base: {e}")
```

## Next Steps

After creating and configuring your base, you can:

- [Manage base tables](../tables/creation.md)
- [Configure table integration](table-integration.md)
- [Work with records](../records/create.md)
