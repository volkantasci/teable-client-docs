# Best Practices

This guide covers best practices and recommended patterns for using the Teable-Client library effectively.

## Client Configuration

### Environment-Based Configuration

```python
from teable import TeableClient, TeableConfig
import os

def create_client(environment='production'):
    """Create a properly configured client instance."""
    configs = {
        'production': {
            'api_url': os.getenv('TEABLE_PROD_API_URL'),
            'api_key': os.getenv('TEABLE_PROD_API_KEY')
        },
        'staging': {
            'api_url': os.getenv('TEABLE_STAGING_API_URL'),
            'api_key': os.getenv('TEABLE_STAGING_API_KEY')
        }
    }
    
    env_config = configs.get(environment)
    if not env_config:
        raise ValueError(f"Invalid environment: {environment}")
        
    config = TeableConfig(
        api_url=env_config['api_url'],
        api_key=env_config['api_key']
    )
    
    return TeableClient(config)
```

## Performance Optimization

### Batch Operations

```python
def batch_process_records(client, table_id, records, batch_size=1000):
    """Process records in batches for optimal performance."""
    for i in range(0, len(records), batch_size):
        batch = records[i:i + batch_size]
        result = client.records.batch_create_records(
            table_id=table_id,
            records=batch
        )
        print(f"Batch {i//batch_size + 1}:")
        print(f"- Successful: {result.success_count}")
        print(f"- Failed: {result.failure_count}")
        print(f"- Success rate: {result.success_rate}%")

# Usage
records = [
    {"Name": f"Record {i}", "Status": "Active"}
    for i in range(10000)
]

batch_process_records(client, "table_id", records)
```

### Query Optimization

```python
from teable import FilterOperator, SortDirection

def get_filtered_records(client, table_id):
    """Get records with optimized filtering."""
    # Use query builder for complex queries
    query = client.tables.get(table_id).query()
    
    # Add filters
    query.filter(
        field="Status",
        operator=FilterOperator.EQUALS,
        value="Active"
    )
    
    # Add sorting
    query.sort(
        field="CreatedTime",
        direction=SortDirection.DESCENDING
    )
    
    # Add pagination
    query.paginate(take=100)
    
    # Execute query
    return client.tables.get_records(
        table_id=table_id,
        query=query.build()
    )
```

## Data Validation

### Field Validation

```python
from teable.exceptions import ValidationError

def validate_record_data(table, data):
    """Validate record data before creation/update."""
    # Get required fields
    required_fields = [
        f for f in table.fields
        if f.is_required and not f.is_computed
    ]
    
    # Check required fields
    for field in required_fields:
        if field.field_id not in data:
            raise ValidationError(
                f"Missing required field: {field.field_id}"
            )
    
    # Validate each field value
    for field_id, value in data.items():
        field = table.get_field(field_id)
        field.validate_value(value)
    
    return True

# Usage
try:
    table = client.tables.get("table_id")
    data = {
        "field1": "value1",
        "field2": "value2"
    }
    
    validate_record_data(table, data)
    record = client.records.create(
        table_id=table.table_id,
        fields=data
    )
except ValidationError as e:
    print(f"Validation error: {e}")
```

## Best Practices

1. **Use Batch Operations**
   - Process multiple records using batch operations
   - Monitor batch operation results
   - Handle partial failures appropriately

2. **Optimize Queries**
   - Use query builder for complex queries
   - Add appropriate filters to reduce data transfer
   - Implement pagination for large datasets

3. **Validate Data**
   - Validate data before sending to API
   - Handle validation errors gracefully
   - Check required fields and data types

4. **Error Handling**
   - Implement comprehensive error handling
   - Log errors with context
   - Handle rate limits appropriately

5. **Security**
   - Use environment variables for sensitive data
   - Keep API keys secure
   - Use different keys for different environments

## Next Steps

- [Authentication](authentication.md)
- [Error Handling](error-handling.md)
- [API Reference](../api/client.md)
