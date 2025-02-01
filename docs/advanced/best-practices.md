# Best Practices

This guide outlines recommended practices for using the Teable Client Library effectively and safely.

## Authentication

### API Key Management

```python
# DON'T: Hardcode API keys
client = TeableClient(TeableConfig(
    api_key="1234567890abcdef"  # Bad practice
))

# DO: Use environment variables
from dotenv import load_dotenv
load_dotenv()

client = TeableClient.from_env()  # Reads from environment variables
```

### Authentication Flow

```python
# DO: Implement proper authentication flow
try:
    # Initialize with API key
    client = TeableClient(TeableConfig(
        api_key=os.getenv('TEABLE_API_KEY')
    ))
    
    # Sign in for operations requiring full access
    client.auth.signin(
        email=os.getenv('TEABLE_EMAIL'),
        password=os.getenv('TEABLE_PASSWORD')
    )
    
    # Perform operations
    
finally:
    # Always sign out when done
    client.auth.signout()
```

## Resource Management

### Cleanup

```python
# DO: Always clean up resources
def manage_temporary_space():
    space = None
    try:
        space = client.spaces.create_space(name="Temporary")
        # ... perform operations ...
    finally:
        if space:
            client.spaces.permanently_delete_space(space.space_id)
```

### Resource Hierarchy

```python
# DO: Follow the resource hierarchy
space = client.spaces.create_space(name="Project")
base = space.create_base(name="Database")
table = client.tables.create_table(
    base_id=base.base_id,
    name="Employees"
)
```

## Data Operations

### Batch Operations

```python
# DON'T: Create records one by one in a loop
for data in records_data:
    table.create_record(data)  # Inefficient

# DO: Use batch operations
batch_result = table.batch_create_records(records_data)
print(f"Created {batch_result.success_count} records")
```

### Query Optimization

```python
# DON'T: Fetch all records and filter in memory
all_records = table.get_records()
filtered = [r for r in all_records if r.fields["Age"] > 30]

# DO: Use API-level filtering
filter_data = {
    "filterSet": [
        {
            "fieldId": "Age",
            "operator": "isGreaterThan",
            "value": 30
        }
    ]
}
filtered_records = table.get_records(filter=filter_data)
```

### Pagination

```python
# DO: Use pagination for large datasets
def fetch_all_records(table):
    records = []
    skip = 0
    take = 100
    
    while True:
        batch = table.get_records(skip=skip, take=take)
        if not batch:
            break
        records.extend(batch)
        skip += take
    
    return records
```

## Error Handling

### Specific Exceptions

```python
# DO: Handle specific exceptions
from teable.exceptions import ValidationError, APIError

try:
    record = table.create_record(data)
except ValidationError as e:
    # Handle validation errors
    logger.error(f"Validation error: {str(e)}")
except APIError as e:
    # Handle API errors
    logger.error(f"API error: {str(e)}")
except Exception as e:
    # Handle unexpected errors
    logger.error(f"Unexpected error: {str(e)}")
```

### Retry Logic

```python
# DO: Implement retry logic for transient failures
def retry_operation(operation, max_retries=3):
    for attempt in range(max_retries):
        try:
            return operation()
        except APIError as e:
            if e.status_code == 429:  # Rate limit
                time.sleep(2 ** attempt)  # Exponential backoff
                continue
            raise
    raise Exception("Max retries exceeded")
```

## Performance

### Connection Management

```python
# DON'T: Create multiple clients
client1 = TeableClient(config)
client2 = TeableClient(config)  # Unnecessary

# DO: Reuse client instance
client = TeableClient(config)
# Use the same client throughout your application
```

### Caching

```python
# DO: Cache frequently accessed data
from functools import lru_cache

@lru_cache(maxsize=100)
def get_table_schema(table_id):
    return client.tables.get_table(table_id)
```

## Testing

### Test Environment

```python
# DO: Use a separate test environment
def setup_test_client():
    return TeableClient(TeableConfig(
        api_key=os.getenv('TEABLE_TEST_API_KEY'),
        api_url=os.getenv('TEABLE_TEST_API_URL')
    ))
```

### Test Data Management

```python
# DO: Clean up test data
@pytest.fixture
def test_space(client):
    space = client.spaces.create_space(name="Test Space")
    yield space
    client.spaces.permanently_delete_space(space.space_id)
```

## Security

### Credential Management

```python
# DON'T: Store credentials in code
credentials = {
    "api_key": "secret_key",
    "password": "secret_pass"
}

# DO: Use secure credential management
from keyring import get_password

api_key = get_password("teable", "api_key")
client = TeableClient(TeableConfig(api_key=api_key))
```

### Permission Checks

```python
# DO: Check permissions before operations
def create_record_if_allowed(table, data):
    permissions = client.tables.get_table_permission(
        table.base_id,
        table.table_id
    )
    
    if permissions["record"]["create"]:
        return table.create_record(data)
    else:
        raise PermissionError("No create permission")
```

## Code Organization

### Module Structure

```python
# DO: Organize code logically
from teable import TeableClient
from teable.exceptions import APIError
from teable.models import Table, Record

class TeableManager:
    def __init__(self):
        self.client = TeableClient.from_env()
    
    def setup_workspace(self):
        """Create and configure workspace"""
        pass
    
    def manage_records(self):
        """Handle record operations"""
        pass
```

### Configuration Management

```python
# DO: Centralize configuration
class TeableConfig:
    def __init__(self):
        load_dotenv()
        
        self.api_key = os.getenv('TEABLE_API_KEY')
        self.api_url = os.getenv('TEABLE_API_URL')
        self.email = os.getenv('TEABLE_EMAIL')
        self.password = os.getenv('TEABLE_PASSWORD')

config = TeableConfig()
client = TeableClient(TeableConfig(
    api_key=config.api_key,
    api_url=config.api_url
))
```

## Logging

### Structured Logging

```python
# DO: Implement proper logging
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def perform_operation():
    try:
        result = client.spaces.create_space(name="New Space")
        logger.info("Space created", extra={
            "space_id": result.space_id,
            "name": result.name
        })
        return result
    except Exception as e:
        logger.error("Failed to create space", exc_info=True)
        raise
```

## Documentation

### Code Comments

```python
# DO: Document code properly
def process_records(table, filter_criteria=None):
    """
    Process records from a table with optional filtering.
    
    Args:
        table: Table instance to process
        filter_criteria: Optional dict of filter parameters
        
    Returns:
        List of processed records
        
    Raises:
        APIError: If API request fails
        ValidationError: If filter criteria is invalid
    """
    pass
```

## Version Control

### Dependencies

```python
# DO: Specify version requirements
# requirements.txt
teable-client>=1.0.0,<2.0.0
python-dotenv>=0.19.0
```

### Git Integration

```python
# DO: Add proper .gitignore
# .gitignore
.env
__pycache__/
*.pyc
.pytest_cache/
