# Creating and Managing Spaces

Spaces in Teable are organizational units that contain bases (databases). This guide covers how to create and manage spaces using the Teable-Client library.

## Creating a Space

Creating a new space is straightforward:

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Create a new space
space = client.spaces.create(name="My Project Space")
print(f"Created space with ID: {space.space_id}")
```

## Managing Space Collaborators

### Adding Collaborators

You can add collaborators to a space with specific roles:

```python
from teable import SpaceRole

# Add collaborators
collaborators = [
    {
        "principalId": "user123",
        "principalType": "user"
    },
    {
        "principalId": "user456",
        "principalType": "user"
    }
]

space.add_collaborators(
    collaborators=collaborators,
    role=SpaceRole.EDITOR
)
```

### Listing Collaborators

You can list all collaborators in a space with various filtering options:

```python
# List all collaborators
collaborators, total = space.get_collaborators(
    include_system=True,  # Include system collaborators
    include_base=True,    # Include base information
    take=100,            # Limit results
    search="john"        # Search by name/email
)

for collaborator in collaborators:
    print(f"Name: {collaborator.name}")
    print(f"Role: {collaborator.role}")
```

### Updating Collaborator Roles

You can update a collaborator's role:

```python
from teable.models.collaborator import PrincipalType

# Update collaborator role
space.update_collaborator(
    principal_id="user123",
    principal_type=PrincipalType.USER,
    role=SpaceRole.CREATOR
)
```

### Removing Collaborators

To remove a collaborator from a space:

```python
# Remove collaborator
space.delete_collaborator(
    principal_id="user123",
    principal_type=PrincipalType.USER
)
```

## Managing Space Invitations

### Creating Invitation Links

You can create invitation links to share with potential collaborators:

```python
# Create an invitation link
invitation = space.create_invitation_link(
    role=SpaceRole.EDITOR
)
```

### Sending Email Invitations

You can directly send invitation emails to multiple users:

```python
# Send invitation emails
emails = ["user1@example.com", "user2@example.com"]
result = space.invite_by_email(
    emails=emails,
    role=SpaceRole.EDITOR
)
```

### Listing Invitation Links

List all invitation links:

```python
# List all invitation links
invitations = space.get_invitation_links()
```

## Best Practices

1. **Role Management**
   - Assign appropriate roles based on user responsibilities
   - Regularly review and audit collaborator permissions
   - Use the most restrictive role that still allows users to do their work

2. **Invitation Management**
   - Use email invitations for direct user invites
   - Use invitation links for broader sharing needs
   - Regularly review and clean up invitations

3. **Security Considerations**
   - Regularly audit space access and permissions
   - Remove collaborators who no longer need access
   - Monitor space activity for unusual patterns

## Error Handling

Handle potential errors when managing spaces:

```python
from teable.exceptions import TeableError, ValidationError, AuthenticationError

try:
    space = client.spaces.create(name="New Space")
except ValidationError as e:
    print(f"Invalid space name: {e}")
except AuthenticationError as e:
    print(f"Authentication failed: {e}")
except TeableError as e:
    print(f"Error creating space: {e}")
```

## Next Steps

After creating and configuring your space, you can:

- [Create bases](../bases/creation.md) within the space
- [Manage space configuration](configuration.md)
- [Set up tables](../tables/creation.md) within bases
