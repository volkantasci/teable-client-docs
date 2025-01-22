# Managing Spaces

This guide covers the various operations available for managing spaces in Teable.

## Listing Spaces

You can retrieve all accessible spaces:

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Get all accessible spaces
spaces = client.spaces.get_all()

# Display space information
for space in spaces:
    print(f"Space ID: {space.space_id}")
```

## Retrieving a Specific Space

Get detailed information about a specific space:

```python
# Get a space by ID
space = client.spaces.get("space123")

# Access space properties
print(f"Space ID: {space.space_id}")
```

## Managing Space Content

### Listing Bases in a Space

You can retrieve all bases within a space:

```python
# Get all bases in a space
bases = client.tables.get_all(space_id="space123")

for base in bases:
    print(f"Base ID: {base.table_id}")
```

## Space Permissions

### Managing Collaborators

```python
from teable import SpaceRole
from teable.models.collaborator import PrincipalType

# Get a space
space = client.spaces.get("space123")

# Get collaborators
collaborators, total = space.get_collaborators(
    include_system=True,
    take=100,
    search="john"
)

# Add collaborators
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

space.add_collaborators(
    collaborators=collaborators,
    role=SpaceRole.EDITOR
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
```

## Best Practices

1. **Resource Management**
   - Regularly clean up unused spaces and bases
   - Monitor space usage
   - Back up important data

2. **Organization**
   - Maintain a clear naming convention for spaces
   - Group related bases within the same space
   - Use descriptive names for better searchability

3. **Access Control**
   - Regularly review collaborator access
   - Use appropriate roles for different users
   - Document access policies

## Error Handling

Implement proper error handling for space management operations:

```python
from teable.exceptions import TeableError, ResourceNotFoundError

try:
    space = client.spaces.get("space123")
except ResourceNotFoundError:
    print("Space not found")
except TeableError as e:
    print(f"Error accessing space: {e}")
```

## Next Steps

After mastering space management, you might want to explore:

- [Space Configuration](configuration.md)
- [Base Creation](../bases/creation.md)
- [Table Management](../tables/management.md)
