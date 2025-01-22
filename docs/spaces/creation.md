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
space = client.create_space("My Project Space")
print(f"Created space with ID: {space.space_id}")
```

## Managing Space Collaborators

### Adding Collaborators

You can add collaborators to a space with specific roles:

```python
from teable import SpaceRole

# Add user collaborators
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

client.add_space_collaborators(
    space_id="space123",
    collaborators=collaborators,
    role=SpaceRole.EDITOR
)
```

### Listing Collaborators

You can list all collaborators in a space with various filtering options:

```python
# List all collaborators
collaborators = client.list_collaborators(
    space_id="space123",
    include_system=True,  # Include system collaborators
    include_base=True,    # Include base information
    take=100,            # Limit results
    search="john"        # Search by name/email
)

for collaborator in collaborators['collaborators']:
    if collaborator['type'] == 'user':
        print(f"User: {collaborator['userName']} ({collaborator['email']})")
        print(f"Role: {collaborator['role']}")
```

### Updating Collaborator Roles

You can update a collaborator's role:

```python
# Update collaborator role
client.update_collaborator(
    space_id="space123",
    principal_id="user123",
    principal_type="user",
    role=SpaceRole.CREATOR
)
```

### Removing Collaborators

To remove a collaborator from a space:

```python
# Remove collaborator
client.delete_collaborator(
    space_id="space123",
    principal_id="user123",
    principal_type="user"
)
```

## Managing Space Invitations

### Creating Invitation Links

You can create invitation links to share with potential collaborators:

```python
# Create an invitation link
invitation = client.create_invitation(
    space_id="space123",
    role=SpaceRole.EDITOR
)

print(f"Invitation URL: {invitation['inviteUrl']}")
print(f"Invitation Code: {invitation['invitationCode']}")
```

### Sending Email Invitations

You can directly send invitation emails to multiple users:

```python
# Send invitation emails
emails = ["user1@example.com", "user2@example.com"]
invitations = client.send_invitation_emails(
    space_id="space123",
    emails=emails,
    role=SpaceRole.EDITOR
)

for email, invitation in invitations.items():
    print(f"Sent invitation to {email}: {invitation['invitationId']}")
```

### Managing Invitation Links

List, update, and delete invitation links:

```python
# List all invitation links
invitations = client.list_invitations("space123")
for invitation in invitations:
    print(f"Invitation ID: {invitation['invitationId']}")
    print(f"Role: {invitation['role']}")
    print(f"URL: {invitation['inviteUrl']}")

# Update invitation role
updated = client.update_invitation(
    space_id="space123",
    invitation_id="inv123",
    role=SpaceRole.VIEWER
)

# Delete an invitation
client.delete_invitation(
    space_id="space123",
    invitation_id="inv123"
)
```

## Best Practices

1. **Role Management**
   - Assign appropriate roles based on user responsibilities
   - Regularly review and audit collaborator permissions
   - Use the most restrictive role that still allows users to do their work

2. **Invitation Management**
   - Set appropriate expiration times for invitation links
   - Regularly clean up unused invitation links
   - Use email invitations for direct user invites
   - Use invitation links for broader sharing needs

3. **Security Considerations**
   - Regularly audit space access and permissions
   - Remove collaborators who no longer need access
   - Use time-limited invitation links when possible
   - Monitor space activity for unusual patterns

## Error Handling

Handle potential errors when managing spaces:

```python
from teable.exceptions import TeableError, ValidationError, AuthenticationError

try:
    space = client.create_space("New Space")
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
