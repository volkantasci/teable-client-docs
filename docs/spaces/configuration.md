# Space Configuration

This guide covers the configuration options and settings available for spaces in Teable, including space roles, organization management, and space updates.

## Space Roles

Teable defines several roles for space access control:

```python
from teable import SpaceRole

# Available roles
SpaceRole.OWNER       # Full control over the space
SpaceRole.CREATOR     # Can create and manage content
SpaceRole.EDITOR      # Can edit existing content
SpaceRole.COMMENTER   # Can comment on content
SpaceRole.VIEWER      # Can only view content
```

## Basic Space Configuration

### Updating Space Information

You can update a space's basic information:

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Get a space
space = client.get_space("space123")

# Update space name
updated_space = space.update(name="New Space Name")
print(f"Space updated: {updated_space.name}")
```

### Organization Management

Spaces can be associated with organizations:

```python
# Access organization information
space = client.get_space("space123")
if space.organization:
    print(f"Organization ID: {space.organization.org_id}")
    print(f"Organization Name: {space.organization.name}")
```

## Access Control

### Role-Based Access Control

```python
# Check user's role in a space
space = client.get_space("space123")
print(f"Current user's role: {space.role}")

# Determine if user has sufficient permissions
if space.role in [SpaceRole.OWNER, SpaceRole.CREATOR]:
    print("User can create and manage content")
elif space.role == SpaceRole.EDITOR:
    print("User can edit content")
elif space.role == SpaceRole.COMMENTER:
    print("User can comment on content")
else:
    print("User can only view content")
```

### Managing Collaborators

```python
from teable.models.collaborator import PrincipalType

# Get collaborators with filtering options
collaborators, total_count = space.get_collaborators(
    include_system=True,    # Include system collaborators
    include_base=True,      # Include base information
    skip=0,                 # Pagination offset
    take=100,              # Number of results to return
    search="john",         # Search term
    collaborator_type=PrincipalType.USER  # Filter by type
)

# Update collaborator role
space.update_collaborator(
    principal_id="user123",
    principal_type=PrincipalType.USER,
    role=SpaceRole.EDITOR
)

# Remove collaborator
space.delete_collaborator(
    principal_id="user123",
    principal_type=PrincipalType.USER
)

# Add new collaborators
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
space.add_collaborators(collaborators, role=SpaceRole.EDITOR)
```

## Invitation Management

### Creating and Managing Invitations

```python
# Create invitation link
invitation = space.create_invitation_link(role=SpaceRole.EDITOR)
print(f"Invitation URL: {invitation.invite_url}")
print(f"Invitation Code: {invitation.invitation_code}")

# List all invitation links
invitations = space.get_invitation_links()
for inv in invitations:
    print(f"ID: {inv.invitation_id}")
    print(f"Role: {inv.role}")
    print(f"Created by: {inv.created_by}")
    print(f"Created time: {inv.created_time}")
    print("---")

# Send email invitations
emails = ["user1@example.com", "user2@example.com"]
result = space.invite_by_email(emails, role=SpaceRole.EDITOR)
for email, info in result.items():
    print(f"Invited {email}: {info['invitationId']}")
```

## Space Management

### Creating Bases

```python
# Create a new base in the space
base = space.create_base(
    name="Project Database",
    icon="📊"  # Optional emoji or icon identifier
)
print(f"Created base: {base.name} ({base.base_id})")

# List all bases in the space
bases = space.get_bases()
for base in bases:
    print(f"Base: {base.name}")
    print(f"ID: {base.base_id}")
    print("---")
```

## Best Practices

1. **Role Assignment**
   - Follow the principle of least privilege
   - Regularly audit user roles
   - Use appropriate roles for automation and system accounts

2. **Organization Management**
   - Keep organization information up to date
   - Use consistent naming conventions
   - Document organization-space relationships

3. **Access Control**
   - Regularly review collaborator access
   - Clean up unused invitation links
   - Use time-limited invitations when possible
   - Document access control policies

4. **Space Structure**
   - Organize bases logically within spaces
   - Use clear, descriptive names
   - Maintain consistent structure across spaces

## Error Handling

```python
from teable.exceptions import TeableError, ValidationError

try:
    # Update space configuration
    space.update(name="New Name")
except ValidationError as e:
    print(f"Invalid configuration: {e}")
except TeableError as e:
    print(f"Error updating space: {e}")
```

## Next Steps

After configuring your space, you can:

- [Create and manage bases](../bases/creation.md)
- [Set up tables](../tables/creation.md)
- [Manage records](../records/create.md)
