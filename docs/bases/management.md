# Managing Bases

This guide covers the various operations available for managing bases in Teable.

## Base Properties

### Checking Base Type

```python
from teable import CollaboratorType

# Get a base
base = client.tables.get("base123")

# Check base type
if base.collaborator_type == CollaboratorType.SPACE:
    print("This is a space-level base")
elif base.collaborator_type == CollaboratorType.BASE:
    print("This is a base-level base")

# Check if base is unrestricted
if base.is_unrestricted:
    print("This base has no access restrictions")
```

## Base Organization

### Managing Base Order

```python
from teable import Position

# Reorder bases
base.update_order(
    anchor_id="base456",
    position=Position.BEFORE  # or Position.AFTER
)
```

## Base Collaborators

### Managing Access

```python
# List collaborators
collaborators, total = base.get_collaborators(
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

base.add_collaborators(
    collaborators=collaborators,
    role="editor"  # Options: creator, editor, commenter, viewer
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

## Best Practices

1. **Organization**
   - Use consistent naming conventions
   - Maintain logical base ordering
   - Document base relationships
   - Regular access review

2. **Access Control**
   - Regularly review collaborator access
   - Use appropriate roles for different user types
   - Document access policies
   - Monitor base permissions

3. **Resource Management**
   - Clean up unused bases
   - Archive inactive bases
   - Monitor base usage
   - Regular backup important bases

## Error Handling

```python
from teable.exceptions import TeableError, ResourceNotFoundError

try:
    # Get base information
    base = client.tables.get("base123")
    
    # Update base
    base.update(name="New Name")
    
except ResourceNotFoundError:
    print("Base not found")
except TeableError as e:
    print(f"Error accessing base: {e}")
```

## Next Steps

After mastering base management, you can:

- [Configure table integration](table-integration.md)
- [Work with records](../records/create.md)
- [Set up views](../views/creation.md)
