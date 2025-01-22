# API Reference: TeableClient

This document provides detailed API reference for the TeableClient class, which is the main entry point for interacting with the Teable API.

## TeableClient

The TeableClient class provides the primary interface for interacting with Teable services.

### Constructor

```python
from teable import TeableClient, TeableConfig

client = TeableClient(
    config=TeableConfig(
        api_url="https://your-teable-instance.com/api",
        api_key="your-api-key"
    )
)
```

### Configuration Options

The `TeableConfig` class accepts the following parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| api_url | str | Yes | The base URL of your Teable instance API |
| api_key | str | Yes | Your API key for authentication |
| timeout | int | No | Request timeout in seconds (default: 30) |
| max_retries | int | No | Maximum number of retry attempts (default: 3) |
| cache_enabled | bool | No | Enable response caching (default: True) |

## Space Operations

### Get Spaces

Retrieve all accessible spaces.

```python
spaces = client.get_spaces()
```

Returns: `List[Space]`

### Get Space

Get a specific space by ID.

```python
space = client.get_space("space_id")
```

Parameters:
- space_id (str): The ID of the space to retrieve

Returns: `Space`

### Create Space

Create a new space.

```python
space = client.create_space("My New Space")
```

Parameters:
- name (str): The name for the new space

Returns: `Space`

## Base Operations

### Get Bases

Retrieve all accessible bases.

```python
bases = client.get_bases()
```

Returns: `List[Base]`

### Get Base

Get a specific base by ID.

```python
base = client.get_base("base_id")
```

Parameters:
- base_id (str): The ID of the base to retrieve

Returns: `Base`

### Create Base

Create a new base in a space.

```python
base = client.create_base(
    space_id="space_id",
    name="My New Base",
    icon="📊"  # Optional
)
```

Parameters:
- space_id (str): The ID of the space to create the base in
- name (str, optional): The name for the new base
- icon (str, optional): An emoji or icon identifier

Returns: `Base`

### Duplicate Base

Create a copy of an existing base.

```python
base = client.duplicate_base(
    from_base_id="source_base_id",
    space_id="target_space_id",
    name="Duplicated Base",
    with_records=True
)
```

Parameters:
- from_base_id (str): The ID of the base to duplicate
- space_id (str): The ID of the space to create the duplicate in
- name (str, optional): The name for the duplicated base
- with_records (bool): Whether to include records in the duplicate

Returns: `Base`

## Table Operations

### Get Table

Get a specific table by ID.

```python
table = client.get_table("table_id")
```

Parameters:
- table_id (str): The ID of the table to retrieve

Returns: `Table`

### Create Table

Create a new table in a base.

```python
table = client.create_table(
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

Parameters:
- base_id (str): The ID of the base to create the table in
- name (str): The name for the new table
- description (str, optional): A description of the table
- fields (List[Dict], optional): Field definitions for the table

Returns: `Table`

## Record Operations

### Get Record

Get a specific record by ID.

```python
record = client.get_record(
    table_id="table_id",
    record_id="record_id",
    cell_format="json"
)
```

Parameters:
- table_id (str): The ID of the table containing the record
- record_id (str): The ID of the record to retrieve
- cell_format (str, optional): Format for cell values ('json' or 'text')

Returns: `Record`

### Create Record

Create a new record in a table.

```python
record = client.create_record(
    table_id="table_id",
    fields={
        "Name": "John Doe",
        "Age": 30
    }
)
```

Parameters:
- table_id (str): The ID of the table to create the record in
- fields (Dict): The field values for the new record

Returns: `Record`

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
    record = client.create_record(table_id, fields)
except ValidationError as e:
    print(f"Invalid data: {e}")
except TeableError as e:
    print(f"Operation failed: {e}")
```

## Next Steps

- [Models Reference](models.md)
- [Exceptions Reference](exceptions.md)
- [Best Practices](../advanced/best-practices.md)
