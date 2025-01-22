# Field Validation

This guide covers field validation in Teable tables, including validation rules, custom validation, and error handling.

## Basic Validation

### Required Fields

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Create a required field
required_field = {
    "name": "Email",
    "type": "email",
    "options": {
        "required": True,
        "unique": True
    }
}

# Create field
table = client.get_table("table_id")
field = table.create_field(required_field)
```

### Field Type Validation

```python
# Email validation
email_field = {
    "name": "Contact Email",
    "type": "email",
    "options": {
        "allowMultiple": False  # Single email only
    }
}

# URL validation
url_field = {
    "name": "Website",
    "type": "url",
    "options": {
        "allowedProtocols": ["http", "https"]
    }
}

# Number validation
number_field = {
    "name": "Score",
    "type": "number",
    "options": {
        "minimum": 0,
        "maximum": 100,
        "precision": 0,  # Integer only
        "allowNegative": False
    }
}
```

## Advanced Validation

### Custom Validation Rules

```python
# Regular expression validation
regex_field = {
    "name": "Product Code",
    "type": "text",
    "options": {
        "pattern": "^[A-Z]{2}\\d{4}$",  # Format: AA1234
        "patternError": "Must be 2 letters followed by 4 digits"
    }
}

# Range validation
range_field = {
    "name": "Age",
    "type": "number",
    "options": {
        "minimum": 18,
        "maximum": 100,
        "minimumError": "Must be at least 18",
        "maximumError": "Must be no more than 100"
    }
}

# Length validation
length_field = {
    "name": "Description",
    "type": "text",
    "options": {
        "minLength": 10,
        "maxLength": 1000,
        "lengthError": "Must be between 10 and 1000 characters"
    }
}
```

### Conditional Validation

```python
# Validation based on other fields
conditional_field = {
    "name": "End Date",
    "type": "date",
    "options": {
        "validation": {
            "condition": "{End Date} > {Start Date}",
            "message": "End date must be after start date"
        }
    }
}

# Required based on condition
conditional_required = {
    "name": "Reason",
    "type": "text",
    "options": {
        "requiredWhen": {
            "field": "Status",
            "equals": "Rejected"
        }
    }
}
```

## Validation Methods

### Field Value Validation

```python
# Validate a single value
try:
    field.validate_value("test@example.com")
    print("Value is valid")
except ValidationError as e:
    print(f"Validation error: {e}")

# Validate multiple values
def validate_values(field, values):
    """Validate multiple values for a field."""
    results = {
        'valid': [],
        'invalid': []
    }
    
    for value in values:
        try:
            field.validate_value(value)
            results['valid'].append(value)
        except ValidationError as e:
            results['invalid'].append({
                'value': value,
                'error': str(e)
            })
    
    return results
```

### Record Validation

```python
# Validate record fields
try:
    table.validate_record_fields({
        "Name": "John Doe",
        "Email": "john@example.com",
        "Age": 25
    })
    print("Record is valid")
except ValidationError as e:
    print(f"Validation error: {e}")

# Batch validation
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
```

## Error Handling

### Validation Error Types

```python
from teable.exceptions import (
    ValidationError,
    RequiredFieldError,
    UniqueFieldError,
    TypeMismatchError
)

try:
    # Attempt validation
    field.validate_value(value)
except RequiredFieldError as e:
    print(f"Required field missing: {e}")
except UniqueFieldError as e:
    print(f"Value must be unique: {e}")
except TypeMismatchError as e:
    print(f"Invalid type: {e}")
except ValidationError as e:
    print(f"General validation error: {e}")
```

### Custom Error Messages

```python
# Field with custom error messages
custom_errors_field = {
    "name": "Phone",
    "type": "text",
    "options": {
        "pattern": "^\\+?\\d{10,15}$",
        "errors": {
            "required": "Phone number is required",
            "pattern": "Invalid phone number format",
            "unique": "This phone number is already registered"
        }
    }
}
```

## Best Practices

1. **Validation Design**
   - Use appropriate validation rules
   - Provide clear error messages
   - Consider user experience
   - Document validation requirements

2. **Performance**
   - Validate early
   - Use batch validation
   - Cache validation results
   - Monitor validation performance

3. **Error Handling**
   - Implement comprehensive error handling
   - Provide meaningful error messages
   - Log validation failures
   - Consider recovery strategies

4. **Maintenance**
   - Regular validation rule review
   - Update validation documentation
   - Monitor validation patterns
   - Clean up unused rules

## Next Steps

- [Field Configuration](configuration.md)
- [Table Management](../tables/management.md)
- [Record Operations](../records/create.md)
- [Error Handling](../advanced/error-handling.md)
