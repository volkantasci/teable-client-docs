# Reading Records

This guide covers all methods for retrieving records from Teable tables.

## Getting Records

### Basic Record Retrieval

```python
from teable import TeableClient, TeableConfig

# Initialize client
client = TeableClient(TeableConfig(api_key="your_api_key"))

# Get records with default parameters
records = client.records.get_records(
    table_id="table_id",
    field_key_type="name"  # Use field names instead of IDs
)
```

### Advanced Query Parameters

```python
# Get records with all available parameters
records = client.records.get_records(
    table_id="table_id",
    projection=["Name", "Email"],  # Only return specific fields
    cell_format="json",  # 'json' or 'text'
    field_key_type="name",  # 'name' or 'id'
    view_id="view_id",  # Filter by view
    ignore_view_query=False,  # Whether to ignore view's filter/sort
    filter_by_tql="Name = 'John'",  # Filter using Teable Query Language
    filter={  # Complex filter object
        "filterSet": [
            {
                "fieldId": "Age",
                "operator": "isGreaterEqual",
                "value": 30
            }
        ],
        "conjunction": "and"
    },
    search=[  # Search parameters
        {
            "value": "John",
            "field": "Name",
            "exact": True
        }
    ],
    filter_link_cell_candidate="linked_record_id",  # Filter by link field candidates
    filter_link_cell_selected="selected_record_id",  # Filter by link field selection
    selected_record_ids=["record1", "record2"],  # Filter by specific records
    order_by="Name",  # Sort specification
    group_by="Department",  # Group specification
    collapsed_group_ids=["group1", "group2"],  # List of collapsed group IDs
    take=100,  # Number of records to return (max 2000)
    skip=0  # Number of records to skip
)
```

### Getting a Single Record

```python
# Get a single record by ID
record = client.records.get_record(
    table_id="table_id",
    record_id="record_id",
    projection=["Name", "Email"],  # Optional: specific fields
    cell_format="json",  # Optional: format
    field_key_type="name"  # Optional: key type
)
```

## Record Status and History

### Checking Record Status

```python
# Get record visibility and deletion status
status = client.records.get_record_status(
    table_id="table_id",
    record_id="record_id"
)

print(f"Is visible: {status.is_visible}")
print(f"Is deleted: {status.is_deleted}")
```

### Getting Record History

```python
# Get history for a specific record
history = client.records.get_record_history(
    table_id="table_id",
    record_id="record_id"
)

# Print history entries
for entry in history.entries:
    print(f"Changed by: {history.users[entry.user_id].name}")
    print(f"Action: {entry.action}")
    print(f"Time: {entry.created_time}")

# Get history for all records in a table
table_history = client.records.get_table_record_history(
    table_id="table_id"
)
```

## Search and Filter Options

### Text Search

```python
# Simple text search
records = client.records.get_records(
    table_id="table_id",
    search=[{
        "value": "John",
        "field": "Name",
        "exact": True  # Exact match
    }]
)

# Multiple search criteria
records = client.records.get_records(
    table_id="table_id",
    search=[
        {
            "value": "John",
            "field": "Name",
            "exact": False  # Partial match
        },
        {
            "value": "Developer",
            "field": "Title",
            "exact": True
        }
    ]
)
```

### Complex Filtering

```python
# Filter with multiple conditions
records = client.records.get_records(
    table_id="table_id",
    filter={
        "filterSet": [
            {
                "fieldId": "Age",
                "operator": "isGreaterEqual",
                "value": 30
            },
            {
                "fieldId": "Department",
                "operator": "is",
                "value": "Engineering"
            }
        ],
        "conjunction": "and"
    }
)

# Filter with TQL (Teable Query Language)
records = client.records.get_records(
    table_id="table_id",
    filter_by_tql="Age >= 30 AND Department = 'Engineering'"
)
```

## Pagination

```python
def fetch_all_records(client, table_id):
    """Fetch all records using pagination."""
    all_records = []
    page_size = 100
    skip = 0
    
    while True:
        records = client.records.get_records(
            table_id=table_id,
            take=page_size,
            skip=skip
        )
        
        if not records:
            break
            
        all_records.extend(records)
        skip += page_size
        
        if len(records) < page_size:
            break
    
    return all_records
```

## Error Handling

```python
from teable.exceptions import ValidationError, APIError

try:
    records = client.records.get_records(
        table_id="table_id",
        take=3000  # Exceeds maximum
    )
except ValidationError as e:
    print(f"Validation error: {str(e)}")
except APIError as e:
    print(f"API error: {str(e)}")
    print(f"Status code: {e.status_code}")
    print(f"Error details: {e.details}")
```

## Best Practices

### 1. Use Field Projections

```python
# DON'T: Fetch all fields when not needed
records = client.records.get_records(table_id="table_id")

# DO: Only fetch required fields
records = client.records.get_records(
    table_id="table_id",
    projection=["Name", "Email"]  # Only fetch needed fields
)
```

### 2. Implement Pagination

```python
# DON'T: Fetch all records at once
records = client.records.get_records(table_id="table_id")

# DO: Use pagination
def process_records_in_batches(client, table_id, batch_size=100):
    skip = 0
    while True:
        batch = client.records.get_records(
            table_id=table_id,
            take=batch_size,
            skip=skip
        )
        
        if not batch:
            break
            
        process_batch(batch)
        skip += batch_size
```

### 3. Use Efficient Filtering

```python
# DON'T: Filter in memory
records = client.records.get_records(table_id="table_id")
filtered = [r for r in records if r["Age"] > 30]  # Inefficient

# DO: Filter at API level
records = client.records.get_records(
    table_id="table_id",
    filter={
        "filterSet": [
            {
                "fieldId": "Age",
                "operator": "isGreaterThan",
                "value": 30
            }
        ]
    }
)
```

### 4. Handle Rate Limits

```python
import time
from teable.exceptions import APIError

def get_records_with_retry(client, table_id, max_retries=3):
    for attempt in range(max_retries):
        try:
            return client.records.get_records(table_id=table_id)
        except APIError as e:
            if e.status_code == 429:  # Rate limit
                if attempt < max_retries - 1:
                    time.sleep(2 ** attempt)  # Exponential backoff
                    continue
            raise
    raise Exception("Max retries exceeded")
```

### 5. Cache Results When Appropriate

```python
from functools import lru_cache
from datetime import datetime, timedelta

@lru_cache(maxsize=100)
def get_cached_record(client, table_id, record_id):
    """Cache individual record lookups."""
    return client.records.get_record(
        table_id=table_id,
        record_id=record_id
    )

class RecordCache:
    """Time-based cache for record queries."""
    def __init__(self, ttl_seconds=300):
        self.cache = {}
        self.ttl = timedelta(seconds=ttl_seconds)
    
    def get_records(self, client, table_id, **query_params):
        cache_key = f"{table_id}:{str(query_params)}"
        now = datetime.now()
        
        if cache_key in self.cache:
            result, timestamp = self.cache[cache_key]
            if now - timestamp < self.ttl:
                return result
        
        result = client.records.get_records(
            table_id=table_id,
            **query_params
        )
        self.cache[cache_key] = (result, now)
        return result
