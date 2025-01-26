# Creating and Managing Bases

A base in Teable is a container for tables and other resources within a space. This guide covers how to create and manage bases using the Teable-Client library.

## Creating a Base

### Basic Base Creation

```python
from teable import TeableClient

# Initialize the client and get spaces
client = TeableClient()
spaces = client.spaces.get_spaces()
space = spaces[0]  # Get first space

# Create a base in the space
base = space.create_base(
    name="Project Tracker",
    icon="📊"  # Optional emoji or icon identifier
)

print(f"Created base: {base.name} (ID: {base.base_id})")

# Access base attributes
print(f"Space ID: {base.space_id}")
print(f"Is Unrestricted: {base.is_unrestricted}")
print(f"Collaborator Type: {base.collaborator_type}")

# Convert to dictionary representation
base_dict = base.to_dict()
```

## Duplicating Bases

You can create a copy of an existing base:

```python
# Duplicate a base
duplicated_base = base.duplicate(
    space_id=space.space_id,
    name="Project Tracker Copy",
    with_records=False  # Whether to include existing records in the copy
)

# Verify duplication
assert duplicated_base.name == "Project Tracker Copy"
assert duplicated_base.space_id == space.space_id
assert duplicated_base.base_id != base.base_id  # Different ID from original
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

## Base Querying

You can execute SQL queries on a base:

```python
# Execute a query with default cell format
results = base.query("SELECT * FROM your_table")

# Execute a query with JSON cell format
results_json = base.query("SELECT * FROM your_table", cell_format='json')
```

## Base Deletion

```python
# Delete a base
result = base.delete()
assert result is True  # Verify deletion was successful
```

!!! warning "Warning"
    Base deletion cannot be undone. Make sure to back up any important data before proceeding.

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
