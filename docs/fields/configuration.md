# Field Configuration

This guide covers how to configure fields in Teable tables, including field types and options.

## Field Types

### Basic Field Types

```python
from teable import TeableClient, TeableConfig, FieldType

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Get your table
table = client.tables.get("table_id")

# Single Line Text Field
text_field = client.fields.create(
    table_id=table.table_id,
    name="Description",
    field_type=FieldType.SINGLE_LINE_TEXT,
    is_required=True
)

# Long Text Field
long_text_field = client.fields.create(
    table_id=table.table_id,
    name="Details",
    field_type=FieldType.LONG_TEXT
)

# Number Field
number_field = client.fields.create(
    table_id=table.table_id,
    name="Amount",
    field_type=FieldType.NUMBER,
    options={
        "precision": 2,
        "format": "currency"
    }
)

# Date Field
date_field = client.fields.create(
    table_id=table.table_id,
    name="Due Date",
    field_type=FieldType.DATE,
    options={
        "format": "YYYY-MM-DD",
        "include_time": True
    }
)
```

### Selection Fields

```python
# Single Select Field
single_select = client.fields.create(
    table_id=table.table_id,
    name="Status",
    field_type=FieldType.SINGLE_SELECT,
    options={
        "choices": ["Active", "Pending", "Completed"]
    }
)

# Multiple Select Field
multi_select = client.fields.create(
    table_id=table.table_id,
    name="Tags",
    field_type=FieldType.MULTIPLE_SELECT,
    options={
        "choices": ["Urgent", "Important", "Review"]
    }
)
```

### Special Field Types

```python
# Checkbox Field
checkbox_field = client.fields.create(
    table_id=table.table_id,
    name="Is Active",
    field_type=FieldType.CHECKBOX
)

# Rating Field
rating_field = client.fields.create(
    table_id=table.table_id,
    name="Priority",
    field_type=FieldType.RATING
)

# Duration Field
duration_field = client.fields.create(
    table_id=table.table_id,
    name="Time Spent",
    field_type=FieldType.DURATION
)

# Button Field
button_field = client.fields.create(
    table_id=table.table_id,
    name="Action",
    field_type=FieldType.BUTTON
)
```

## Field Management

### Getting Field Information

```python
# Get field from table
field = table.get_field("field_id")

# Access field properties
print(f"Field ID: {field.field_id}")
print(f"Type: {field.field_type}")
print(f"Required: {field.is_required}")
print(f"Primary: {field.is_primary}")
print(f"Computed: {field.is_computed}")
```

### Updating Fields

```python
# Update field configuration
field = client.fields.update(
    field_id="field_id",
    name="New Name",
    is_required=True,
    options={
        "choices": ["Option 1", "Option 2", "Option 3"]  # For select fields
    }
)
```

## Field Types Reference

### Available Field Types

```python
from teable import FieldType

# Text Fields
FieldType.SINGLE_LINE_TEXT    # Single line text
FieldType.LONG_TEXT          # Multi-line text

# Number Fields
FieldType.NUMBER            # Numeric values

# Date Fields
FieldType.DATE              # Date and time

# Selection Fields
FieldType.SINGLE_SELECT     # Single choice
FieldType.MULTIPLE_SELECT   # Multiple choices

# Special Fields
FieldType.CHECKBOX          # Boolean values
FieldType.RATING           # Rating values
FieldType.DURATION         # Time duration
FieldType.BUTTON           # Button field

# System Fields
FieldType.CREATED_TIME     # Record creation time
FieldType.LAST_MODIFIED_TIME  # Last modification time
FieldType.CREATED_BY       # Record creator
FieldType.LAST_MODIFIED_BY  # Last modifier
FieldType.AUTO_NUMBER      # Auto-incrementing number
```

## Best Practices

1. **Field Design**
   - Choose appropriate field types for your data
   - Use clear, descriptive field names
   - Set required fields appropriately
   - Document field purposes

2. **Field Options**
   - Configure field options based on data requirements
   - Use appropriate formats for numbers and dates
   - Set meaningful choices for select fields
   - Consider validation requirements

3. **Field Management**
   - Regularly review field configurations
   - Update field settings as needed
   - Monitor field usage
   - Clean up unused fields

## Error Handling

```python
from teable.exceptions import TeableError, ValidationError

try:
    # Create a field
    field = client.fields.create(
        table_id="table_id",
        name="New Field",
        field_type=FieldType.SINGLE_LINE_TEXT,
        is_required=True
    )
except ValidationError as e:
    print(f"Invalid field configuration: {e}")
except TeableError as e:
    print(f"Error creating field: {e}")
```

## Next Steps

- [Field Validation](validation.md)
- [Table Management](../tables/management.md)
- [Record Operations](../records/create.md)
- [Views Configuration](../views/creation.md)
