# Creating and Managing Bases

A base in Teable is a container for tables and other resources within a space. This guide covers how to create and manage bases using the Teable-Client library.

## Creating a Base

### Basic Base Creation

The simplest way to create a base is within a space:

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Create a base in a space
base = client.create_base(
    space_id="space123",
    name="Project Tracker",
    icon="📊"  # Optional emoji or icon identifier
)

print(f"Created base: {base.name} (ID: {base.base_id})")
```

### Creating from a Space Instance

You can also create a base directly from a space instance:

```python
# Get a space
space = client.get_space("space123")

# Create a base in this space
base = space.create_base(
    name="Customer Database",
    icon="👥"
)
```

## Duplicating Bases

You can create a copy of an existing base:

```python
# Duplicate a base
duplicated_base = client.duplicate_base(
    from_base_id="base123",
    space_id="space123",
    name="Project Tracker Copy",
    with_records=True  # Include existing records in the copy
)
```

Or duplicate from a base instance:

```python
# Get an existing base
base = client.get_base("base123")

# Create a duplicate
duplicate = base.duplicate(
    space_id="space123",
    name="Project Tracker - Development",
    with_records=False  # Start with empty tables
)
```

## Creating from Templates

You can create a new base from an existing template:

```python
# Create a base from a template
base = client.create_base_from_template(
    space_id="space123",
    template_id="template123",
    with_records=True  # Include template records
)
```

## Base Configuration

### Updating Base Information

You can update a base's name and icon:

```python
# Get a base
base = client.get_base("base123")

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
print(f"Invitation URL: {invitation.invite_url}")

# List all invitation links
invitations = base.get_invitation_links()
for inv in invitations:
    print(f"ID: {inv.invitation_id}")
    print(f"Role: {inv.role}")
```

### Sending Email Invitations

```python
# Send invitation emails
emails = ["user1@example.com", "user2@example.com"]
result = base.send_email_invitations(
    emails=emails,
    role="editor"
)

for email, info in result.items():
    print(f"Invited {email}: {info['invitationId']}")
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
   - Use templates for standardized base structures
   - Maintain consistent naming conventions

## Error Handling

```python
from teable.exceptions import TeableError, ValidationError

try:
    base = client.create_base(
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
