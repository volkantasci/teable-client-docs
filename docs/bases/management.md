# Managing Bases

This guide covers the various operations available for managing bases in Teable, including listing bases, database connections, and querying capabilities.

## Listing Bases

### Getting All Accessible Bases

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Get all accessible bases
bases = client.get_bases()

for base in bases:
    print(f"Base: {base.name}")
    print(f"ID: {base.base_id}")
    print(f"Space ID: {base.space_id}")
    print(f"Icon: {base.icon or 'No icon'}")
    print("---")
```

### Getting Bases in a Space

```python
# Get bases in a specific space
space_bases = client.get_space_bases("space123")

# Or from a space instance
space = client.get_space("space123")
space_bases = space.get_bases()
```

### Getting Shared Bases

```python
# Get all shared bases
shared_bases = client.get_shared_bases()

for base in shared_bases:
    print(f"Shared Base: {base.name}")
    print(f"ID: {base.base_id}")
```

## Database Connections

Teable allows you to establish database connections for direct data access.

### Creating a Connection

```python
# Create a database connection
connection = client.create_db_connection("base123")

# Access connection details
print(f"Connection URL: {connection['url']}")
print(f"DSN: {connection['dsn']}")
print(f"Current Connections: {connection['connection']['current']}")
print(f"Max Connections: {connection['connection']['max']}")
```

### Managing Connections

```python
# Get connection information
connection_info = client.get_db_connection("base123")

# Delete a connection when no longer needed
client.delete_db_connection("base123")
```

## Querying Bases

Teable provides SQL querying capabilities for bases.

### Basic Queries

```python
# Execute a simple query
results = base.query(
    query="SELECT * FROM table_name LIMIT 10",
    cell_format="text"  # or "json"
)

for row in results:
    print(row)
```

### Advanced Queries

```python
# Complex query with joins and aggregations
query = """
SELECT t1.name, COUNT(t2.id) as count
FROM table1 t1
LEFT JOIN table2 t2 ON t1.id = t2.table1_id
GROUP BY t1.name
HAVING count > 5
ORDER BY count DESC
"""

results = base.query(query, cell_format="json")

# Process results
for row in results:
    print(f"Name: {row['name']}, Count: {row['count']}")
```

## Base Properties

### Checking Base Type

```python
from teable import CollaboratorType

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

## Trash Management

### Managing Deleted Bases

```python
from teable.models.trash import ResourceType

# List bases in trash
trash_items = client.get_trash_items(ResourceType.BASE)

# List trash items for a specific base
base_trash = client.get_trash_items_for_resource(
    resource_id="base123",
    resource_type=ResourceType.BASE
)

# Restore a base from trash
client.restore_trash_item("trash_item_id")

# Permanently delete trash items
client.reset_trash_items_for_resource(
    resource_id="base123",
    resource_type=ResourceType.BASE
)
```

## Best Practices

1. **Database Connections**
   - Close connections when not in use
   - Monitor connection pool usage
   - Use appropriate cell formats for queries
   - Implement connection error handling

2. **Query Optimization**
   - Use appropriate WHERE clauses
   - Limit result sets when possible
   - Consider query performance impact
   - Use indexes effectively

3. **Resource Management**
   - Clean up unused bases
   - Archive inactive bases
   - Monitor base usage
   - Regular backup important bases

4. **Organization**
   - Use consistent naming conventions
   - Maintain logical base ordering
   - Document base relationships
   - Regular access review

## Error Handling

```python
from teable.exceptions import TeableError, ResourceNotFoundError

try:
    # Attempt database operations
    connection = client.create_db_connection("base123")
    results = base.query("SELECT * FROM table_name")
except ResourceNotFoundError:
    print("Base or table not found")
except TeableError as e:
    print(f"Error accessing base: {e}")
finally:
    # Clean up connections
    try:
        client.delete_db_connection("base123")
    except TeableError:
        print("Error closing connection")
```

## Performance Tips

1. **Query Optimization**
   ```python
   # Good - Specific columns, limited results
   results = base.query(
       "SELECT specific_column FROM table_name WHERE condition LIMIT 100"
   )
   
   # Avoid - Select all columns, no limit
   results = base.query("SELECT * FROM table_name")
   ```

2. **Connection Management**
   ```python
   # Create connection only when needed
   connection = client.create_db_connection("base123")
   try:
       # Perform operations
       results = base.query("SELECT * FROM table_name")
   finally:
       # Always clean up
       client.delete_db_connection("base123")
   ```

## Next Steps

After mastering base management, you can:

- [Configure table integration](table-integration.md)
- [Work with records](../records/create.md)
- [Set up views](../views/creation.md)
