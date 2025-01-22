# Field Validation

This guide covers field validation in Teable tables.

## Field Validation

### Basic Validation

```python
from teable import TeableClient, TeableConfig, FieldType

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Create a required field
field = client.fields.create(
    table_id="table_id",
    name="Name",
    field_type=FieldType.SINGLE_LINE_TEXT,
    is_required=True
)

# Validate a value
try:
    field.validate_value("John Doe")
    print("Value is valid")
except ValidationError as e:
    print(f"Validation error: {e}")
```

### Number Field Validation

```python
# Create a number field with precision
number_field = client.fields.create(
    table_id="table_id",
    name="Score",
    field_type=FieldType.NUMBER,
    options={
        "precision": 2  # Allow 2 decimal places
    }
)

# Validate number values
try:
    number_field.validate_value(75.50)  # Valid
    number_field.validate_value("invalid")  # Will raise ValidationError
except ValidationError as e:
    print(f"Validation error: {e}")
```

### Select Field Validation

```python
# Create a single select field
select_field = client.fields.create(
    table_id="table_id",
    name="Status",
    field_type=FieldType.SINGLE_SELECT,
    options={
        "choices": ["Active", "Pending", "Completed"]
    }
)

# Validate select values
try:
    select_field.validate_value("Active")  # Valid
    select_field.validate_value("Invalid Status")  # Will raise ValidationError
except ValidationError as e:
    print(f"Validation error: {e}")
```

### Date Field Validation

```python
# Create a date field
date_field = client.fields.create(
    table_id="table_id",
    name="Due Date",
    field_type=FieldType.DATE,
    options={
        "format": "YYYY-MM-DD",
        "include_time": True
    }
)

# Validate date values
try:
    date_field.validate_value("2024-01-23")  # Valid
    date_field.validate_value("invalid date")  # Will raise ValidationError
except ValidationError as e:
    print(f"Validation error: {e}")
```

## Record Validation

### Validating Record Fields

```python
from teable.exceptions import ValidationError

# Get a table
table = client.tables.get("table_id")

# Validate record fields
try:
    table.validate_record_fields({
        "Name": "John Doe",
        "Score": 75.50,
        "Status": "Active",
        "Due Date": "2024-01-23"
    })
    print("Record is valid")
except ValidationError as e:
    print(f"Validation error: {e}")
```

### Batch Record Validation

```python
def validate_records(table, records):
    """Validate multiple records."""
    results = {
        'valid': [],
        'invalid': []
    }
    
    for record in records:
        try:
            table.validate_record_fields(record)
            results['valid'].append(record)
        except ValidationError as e:
            results['invalid'].append({
                'record': record,
                'error': str(e)
            })
    
    return results

# Use the validation function
records = [
    {
        "Name": "John Doe",
        "Score": 75.50,
        "Status": "Active"
    },
    {
        "Name": "Jane Smith",
        "Score": "invalid",  # Invalid score
        "Status": "Unknown"  # Invalid status
    }
]

results = validate_records(table, records)
print(f"Valid records: {len(results['valid'])}")
print(f"Invalid records: {len(results['invalid'])}")
```

## Best Practices

1. **Field Validation**
   - Use appropriate field types for data
   - Set required fields appropriately
   - Configure field options correctly
   - Validate data before saving

2. **Error Handling**
   - Implement comprehensive error handling
   - Provide meaningful error messages
   - Log validation failures
   - Handle validation errors gracefully

3. **Performance**
   - Use batch validation for multiple records
   - Validate data early in the process
   - Handle validation efficiently
   - Monitor validation performance

## Next Steps

- [Field Configuration](configuration.md)
- [Table Management](../tables/management.md)
- [Record Operations](../records/create.md)
- [Error Handling](../advanced/error-handling.md)
