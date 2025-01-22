# Managing Spaces

This guide covers the various operations available for managing spaces in Teable, including listing spaces, retrieving space information, and managing space content.

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
spaces = client.get_spaces()

# Display space information
for space in spaces:
    print(f"Space ID: {space.space_id}")
    print(f"Name: {space.name}")
    print("---")
```

## Retrieving a Specific Space

Get detailed information about a specific space:

```python
# Get a space by ID
space = client.get_space("space123")

# Access space properties
print(f"Space Name: {space.name}")
print(f"Space ID: {space.space_id}")
```

## Managing Space Content

### Listing Bases in a Space

You can retrieve all bases within a space:

```python
# Get all bases in a space
bases = client.get_space_bases("space123")

for base in bases:
    print(f"Base Name: {base.name}")
    print(f"Base ID: {base.base_id}")
    print("---")
```

### Managing Space Trash

Teable provides trash management capabilities for spaces:

```python
from teable.models.trash import ResourceType

# List items in space trash
trash_items = client.get_trash_items(ResourceType.SPACE)

# List trash items for a specific space
space_trash = client.get_trash_items_for_resource(
    resource_id="space123",
    resource_type=ResourceType.SPACE
)

# Restore an item from trash
client.restore_trash_item("trash_item_id")

# Permanently delete trash items
client.reset_trash_items_for_resource(
    resource_id="space123",
    resource_type=ResourceType.SPACE
)
```

### Permanent Deletion

If needed, you can permanently delete a space:

```python
try:
    # Permanently delete a space
    client.permanently_delete_space("space123")
    print("Space permanently deleted")
except TeableError as e:
    print(f"Error deleting space: {e}")
```

!!! warning "Warning"
    Permanent deletion cannot be undone. Make sure to back up any important data before proceeding with permanent deletion.

## Space Permissions

### Checking Base Permissions

You can check permissions for bases within a space:

```python
# Get permissions for a base
permissions = client.get_base_permission("base123")

# Check specific permissions
for permission, allowed in permissions.items():
    print(f"{permission}: {'Allowed' if allowed else 'Not Allowed'}")
```

## Advanced Operations

### Database Connections

Teable allows you to manage database connections for bases within a space:

```python
# Create a database connection
connection = client.create_db_connection("base123")
print(f"Connection URL: {connection['url']}")

# Get connection information
connection_info = client.get_db_connection("base123")
print(f"Current connections: {connection_info['connection']['current']}")
print(f"Max connections: {connection_info['connection']['max']}")

# Delete a connection when no longer needed
client.delete_db_connection("base123")
```

### Base Ordering

You can manage the order of bases within a space:

```python
# Update base order
client.update_base_order(
    base_id="base123",
    anchor_id="base456",
    position="after"  # or "before"
)
```

### Querying Bases

Execute SQL queries on bases within a space:

```python
# Execute a query
results = client.query_base(
    base_id="base123",
    query="SELECT * FROM table_name WHERE column = 'value'",
    cell_format="json"  # or "text"
)

for row in results:
    print(row)
```

## Best Practices

1. **Resource Management**
   - Regularly clean up unused spaces and bases
   - Use trash management before permanent deletion
   - Monitor database connections and close unused ones

2. **Performance Optimization**
   - Use pagination when listing large numbers of items
   - Close database connections when not in use
   - Use appropriate cell formats when querying data

3. **Organization**
   - Maintain a clear naming convention for spaces
   - Group related bases within the same space
   - Use descriptive names for better searchability

## Error Handling

Implement proper error handling for space management operations:

```python
from teable.exceptions import TeableError, ResourceNotFoundError

try:
    space = client.get_space("space123")
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
