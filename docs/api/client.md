# API Reference: TeableClient

This document provides detailed API reference for the TeableClient class, which is the main entry point for interacting with the Teable API.

## TeableClient

The TeableClient class provides the primary interface for interacting with Teable services through various specialized managers.

### Constructor

The TeableClient can be initialized in two ways:

#### Using TeableConfig Object

```python
from teable import TeableClient, TeableConfig

client = TeableClient(
    config=TeableConfig(
        api_url="https://your-teable-instance.com/api",
        api_key="your-api-key",
        timeout=45.0,
        max_retries=5
    )
)
```

#### Using Dictionary Configuration

```python
from teable import TeableClient

client = TeableClient({
    'api_url': "https://your-teable-instance.com/api",
    'api_key': "your-api-key",
    'timeout': 60.0,
    'max_retries': 5
})
```

### Configuration Options

The configuration accepts the following parameters:

| Parameter | Type | Required | Description | Validation Rules |
|-----------|------|----------|-------------|-----------------|
| api_url | str | Yes | The base URL of your Teable instance API | Must be a valid URL |
| api_key | str | Yes | Your API key for authentication | Must start with 'teable_' |
| timeout | float | No | Request timeout in seconds (default: 30) | Must be positive |
| max_retries | int | No | Maximum number of retry attempts (default: 3) | Must be non-negative |

### Configuration Validation

The client performs validation on the configuration parameters:

```python
# Invalid API key format
client = TeableClient({
    'api_key': 'invalid_key',  # Will raise ConfigurationError
    'api_url': 'https://api.teable.io'
})

# Invalid API URL
client = TeableClient({
    'api_key': 'teable_test_key',
    'api_url': 'not_a_url'  # Will raise ConfigurationError
})

# Invalid timeout
client = TeableClient({
    'api_key': 'teable_test_key',
    'api_url': 'https://api.teable.io',
    'timeout': -1  # Will raise ConfigurationError
})
```

## Available Managers

The TeableClient provides access to various specialized managers for different aspects of the API. Here's a complete list of available managers:

| Manager | Description |
|---------|-------------|
| spaces | Space management operations |
| tables | Table and base operations |
| records | Record CRUD operations |
| fields | Field configuration and management |
| views | View creation and management |
| attachments | File attachment operations |
| selection | Record selection operations |
| notifications | Notification management |
| access_tokens | Access token management |
| imports | Data import operations |
| exports | Data export operations |
| pins | Pin management |
| billing | Billing and subscription operations |
| admin | Administrative operations |
| usage | Usage statistics and monitoring |
| oauth | OAuth integration management |
| undo_redo | Operation history management |
| plugins | Plugin management |
| comments | Comment operations |
| organizations | Organization management |
| ai | AI-powered operations |
| integrity | Data integrity checks |
| aggregation | Data aggregation operations |

Example usage of these managers:

### Space Operations (client.spaces)

```python
# Get all spaces
spaces = client.spaces.get_all()

# Get a specific space
space = client.spaces.get("space_id")

# Create a new space
space = client.spaces.create(name="My New Space")
```

### Base Operations (client.tables)

```python
# Get all bases
bases = client.tables.get_all()

# Get a specific base
base = client.tables.get("base_id")

# Create a new base
base = client.tables.create(
    space_id="space_id",
    name="My New Base",
    icon="📊"  # Optional
)
```

### Table Operations (client.tables)

```python
# Get a specific table
table = client.tables.get("table_id")

# Create a new table
table = client.tables.create(
    base_id="base_id",
    name="My New Table",
    description="Table description",
    fields=[
        {
            "name": "Name",
            "type": "text",
            "required": True
        },
        {
            "name": "Age",
            "type": "number"
        }
    ]
)
```

### Record Operations (client.records)

```python
# Get a specific record
record = client.records.get(
    table_id="table_id",
    record_id="record_id"
)

# Create a new record
record = client.records.create(
    table_id="table_id",
    fields={
        "Name": "John Doe",
        "Age": 30
    }
)
```

### Field Operations (client.fields)

```python
# Get field information
field = client.fields.get("field_id")

# Create a new field
field = client.fields.create(
    table_id="table_id",
    name="New Field",
    type="text"
)
```

### View Operations (client.views)

```python
# Get view information
view = client.views.get("view_id")

# Create a new view
view = client.views.create(
    table_id="table_id",
    name="New View",
    type="grid"
)
```

### Attachment Operations (client.attachments)

```python
# Upload an attachment
attachment = client.attachments.upload(file_path="path/to/file")

# Get attachment information
attachment = client.attachments.get("attachment_id")
```

### Selection Operations (client.selection)

```python
# Create a selection
selection = client.selection.create(
    table_id="table_id",
    record_ids=["record1", "record2"]
)
```

### Notification Operations (client.notifications)

```python
# Get notifications
notifications = client.notifications.get_all()

# Mark notification as read
client.notifications.mark_read("notification_id")
```

### Organization Operations (client.organizations)

```python
# Get organization information
org = client.organizations.get("org_id")

# Update organization settings
client.organizations.update("org_id", settings={})
```

### AI Operations (client.ai)

```python
# Generate content using AI
result = client.ai.generate(prompt="Your prompt here")
```

### Plugin Operations (client.plugins)

```python
# Get installed plugins
plugins = client.plugins.get_all()

# Install a plugin
client.plugins.install("plugin_id")
```

### Comment Operations (client.comments)

```python
# Add a comment
comment = client.comments.create(
    record_id="record_id",
    content="Comment text"
)

# Get comments
comments = client.comments.get_all(record_id="record_id")
```

### Cache Management

The client maintains caches for various resources to improve performance. You can clear these caches when needed:

```python
# Clear all caches
client.clear_cache()
```

## Error Handling

The client may raise the following exceptions:

- `TeableError`: Base exception for all Teable errors
- `ValidationError`: Data validation errors
- `AuthenticationError`: Authentication failures
- `ResourceNotFoundError`: Resource not found errors
- `RateLimitError`: API rate limit exceeded
- `NetworkError`: Network-related errors

Example error handling:

```python
from teable.exceptions import TeableError, ValidationError

try:
    record = client.records.create(table_id, fields)
except ValidationError as e:
    print(f"Invalid data: {e}")
except TeableError as e:
    print(f"Operation failed: {e}")
```

## Next Steps

- [Models Reference](models.md)
- [Exceptions Reference](exceptions.md)
- [Best Practices](../advanced/best-practices.md)
