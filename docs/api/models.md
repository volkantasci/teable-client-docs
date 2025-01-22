# API Reference: Models

This document provides detailed API reference for all model classes in the Teable-Client library.

## Space Models

### Space

Represents a space in Teable, which is a container for bases and other resources.

```python
from teable import Space, SpaceRole

# Space properties
space.space_id: str          # Unique identifier
space.name: str              # Display name
space.role: SpaceRole        # User's role in the space
space.organization: Optional[Organization]  # Associated organization
```

#### Methods

```python
# Update space information
space.update(name: str) -> Space

# Delete space
space.delete() -> bool

# Delete permanently
space.delete_permanent() -> bool

# Get invitation links
space.get_invitation_links() -> List[Invitation]

# Create invitation link
space.create_invitation_link(role: SpaceRole) -> Invitation

# Send email invitations
space.invite_by_email(
    emails: List[str],
    role: SpaceRole
) -> Dict[str, Dict[str, str]]  # Map of email to invitation info

# Get collaborators
space.get_collaborators(
    include_system: Optional[bool] = None,
    include_base: Optional[bool] = None,
    skip: Optional[int] = None,
    take: Optional[int] = None,
    search: Optional[str] = None,
    collaborator_type: Optional[PrincipalType] = None
) -> Tuple[List[Collaborator], int]  # Returns collaborators and total count

# Update collaborator role
space.update_collaborator(
    principal_id: str,
    principal_type: PrincipalType,
    role: SpaceRole
) -> None

# Delete collaborator
space.delete_collaborator(
    principal_id: str,
    principal_type: PrincipalType
) -> None

# Add collaborators
space.add_collaborators(
    collaborators: List[Dict[str, str]],  # List of collaborator info dicts
    role: SpaceRole
) -> None

# Create base
space.create_base(
    name: Optional[str] = None,
    icon: Optional[str] = None
) -> Base

# Get bases
space.get_bases() -> List[Base]
```

### SpaceRole

Enumeration of possible roles in a space.

```python
from teable import SpaceRole

SpaceRole.OWNER      # Full control
SpaceRole.CREATOR    # Can create and manage content
SpaceRole.EDITOR     # Can edit content
SpaceRole.COMMENTER  # Can comment on content
SpaceRole.VIEWER     # Can only view content
```

### Organization

Represents an organization in Teable.

```python
from teable import Organization

# Organization properties
organization.org_id: str  # Unique identifier
organization.name: str    # Display name
```

## Base Models

### Base

Represents a base in Teable, which is a container for tables.

```python
from teable import Base, CollaboratorType, Position

# Base properties
base.base_id: str           # Unique identifier
base.name: str              # Display name
base.space_id: str          # ID of containing space
base.icon: Optional[str]    # Base icon
base.collaborator_type: Optional[CollaboratorType]  # Type of collaborator
base.is_unrestricted: bool  # Access restriction flag
```

#### Methods

```python
# Update base information
base.update(
    name: Optional[str] = None,
    icon: Optional[str] = None
) -> Base

# Delete base
base.delete() -> bool

# Delete permanently
base.delete_permanent() -> bool

# Update order
base.update_order(
    anchor_id: str,
    position: Position
) -> None

# Duplicate base
base.duplicate(
    space_id: str,
    name: Optional[str] = None,
    with_records: bool = False
) -> Base

# Get collaborators
base.get_collaborators(
    include_system: Optional[bool] = None,
    skip: Optional[int] = None,
    take: Optional[int] = None,
    search: Optional[str] = None,
    collaborator_type: Optional[PrincipalType] = None
) -> Tuple[List[Collaborator], int]  # Returns collaborators and total count

# Update collaborator role
base.update_collaborator(
    principal_id: str,
    principal_type: PrincipalType,
    role: str
) -> None

# Delete collaborator
base.delete_collaborator(
    principal_id: str,
    principal_type: PrincipalType
) -> None

# Get permissions
base.get_permissions() -> Dict[str, bool]

# Execute SQL query
base.query(
    query: str,
    cell_format: str = 'text'
) -> List[Dict[str, Any]]

# Get invitation links
base.get_invitation_links() -> List[Invitation]

# Create invitation link
base.create_invitation_link(role: str) -> Invitation

# Send email invitations
base.send_email_invitations(
    emails: List[str],
    role: str
) -> Dict[str, Dict[str, str]]

# Add collaborators
base.add_collaborators(
    collaborators: List[Dict[str, str]],
    role: str
) -> None
```

### CollaboratorType

Enumeration of collaborator types.

```python
from teable import CollaboratorType

CollaboratorType.SPACE  # Space collaborator
CollaboratorType.BASE   # Base collaborator
```

### Position

Enumeration of position options for ordering.

```python
from teable import Position

Position.BEFORE  # Position before anchor
Position.AFTER   # Position after anchor
```

## Table Models

### Table

Represents a table in Teable, which stores records and fields.

```python
from teable import Table

# Table properties
table.table_id: str              # Unique identifier
table.name: str                  # Display name
table.description: Optional[str]  # Table description
table._fields: Optional[List[Field]]  # Cached fields
table._views: Optional[List[View]]    # Cached views
```

#### Methods

```python
# Get fields (cached)
@property
def fields(self) -> List[Field]

# Get views (cached)
@property
def views(self) -> List[View]

# Get specific field
def get_field(self, field_id: str) -> Field

# Get specific view
def get_view(self, view_id: str) -> View

# Validate record fields
def validate_record_fields(self, fields: Dict[str, Any]) -> None

# Get records
def get_records(
    self,
    projection: Optional[List[str]] = None,
    cell_format: str = 'json',
    field_key_type: str = 'name',
    view_id: Optional[str] = None,
    ignore_view_query: Optional[bool] = None,
    filter_by_tql: Optional[str] = None,
    filter: Optional[Dict[str, Any]] = None,
    search: Optional[List[Any]] = None,
    filter_link_cell_candidate: Optional[Union[str, List[str]]] = None,
    filter_link_cell_selected: Optional[Union[str, List[str]]] = None,
    selected_record_ids: Optional[List[str]] = None,
    order_by: Optional[str] = None,
    group_by: Optional[str] = None,
    collapsed_group_ids: Optional[List[str]] = None,
    take: Optional[int] = None,
    skip: Optional[int] = None,
    query: Optional[Union[QueryBuilder, Dict[str, Any]]] = None
) -> List[Record]

# Get single record
def get_record(
    self,
    record_id: str,
    projection: Optional[List[str]] = None,
    cell_format: str = 'json',
    field_key_type: str = 'name'
) -> Record

# Create record
def create_record(
    self,
    fields: Dict[str, Any]
) -> Record

# Update record
def update_record(
    self,
    record_id: str,
    fields: Dict[str, Any],
    field_key_type: str = 'name',
    typecast: bool = False,
    order: Optional[Dict[str, Any]] = None
) -> Record

# Duplicate record
def duplicate_record(
    self,
    record_id: str,
    view_id: str,
    anchor_id: str,
    position: str
) -> Record

# Delete record
def delete_record(
    self,
    record_id: str
) -> bool

# Batch create records
def batch_create_records(
    self,
    records: List[Dict[str, Any]],
    field_key_type: str = 'name',
    typecast: bool = False,
    order: Optional[Dict[str, Any]] = None
) -> RecordBatch

# Batch update records
def batch_update_records(
    self,
    updates: List[Dict[str, Any]],
    field_key_type: str = 'name',
    typecast: bool = False,
    order: Optional[Dict[str, Any]] = None
) -> List[Record]

# Batch delete records
def batch_delete_records(
    self,
    record_ids: List[str]
) -> bool

# Create query builder
def query(self) -> QueryBuilder

# Clear cache
def clear_cache(self) -> None
```

## Record Models

### RecordStatus

Represents a record's visibility and deletion status.

```python
from teable import RecordStatus

# Properties
status.is_visible: bool  # Whether the record is visible
status.is_deleted: bool  # Whether the record is deleted
```

### Record

Represents a record in a table.

```python
from teable import Record

# Record properties
record.record_id: str          # Unique identifier
record.fields: Dict[str, Any]  # Field values
record.name: Optional[str]     # Primary field value
record.auto_number: Optional[int]  # Auto-incrementing number
record.created_time: Optional[datetime]  # Creation timestamp
record.last_modified_time: Optional[datetime]  # Last modification timestamp
record.created_by: Optional[str]  # Creator user ID
record.last_modified_by: Optional[str]  # Last modifier user ID
```

#### Methods

```python
# Get field value
def get_field_value(
    self,
    field: Union[str, Field],
    cell_format: str = 'json'
) -> Any

# Set field value
def set_field_value(
    self,
    field: Union[str, Field],
    value: Any,
    typecast: bool = False
) -> None

# Convert to dictionary
def to_dict(self) -> Dict[str, Any]
```

### RecordBatch

Represents results from batch record operations.

```python
from teable import RecordBatch

# Properties
batch.successful: List[Record]  # Successfully processed records
batch.failed: List[Dict]       # Failed operations with error details
batch.total: int               # Total number of records in batch

# Computed properties
@property
def success_count(self) -> int  # Number of successful operations

@property
def failure_count(self) -> int  # Number of failed operations

@property
def success_rate(self) -> float  # Success rate as percentage

# String representation
def __str__(self) -> str  # Returns formatted batch results

# Create from API response
@classmethod
def from_api_response(
    cls,
    response: Dict[str, Any],
    total: int
) -> 'RecordBatch'
```

## Field Models

### FieldType

Enumeration of supported field types in Teable.

```python
from teable import FieldType

FieldType.SINGLE_LINE_TEXT    # Single line text field
FieldType.LONG_TEXT          # Multi-line text field
FieldType.USER               # User reference field
FieldType.ATTACHMENT         # File attachment field
FieldType.CHECKBOX          # Boolean checkbox field
FieldType.MULTIPLE_SELECT    # Multiple choice field
FieldType.SINGLE_SELECT     # Single choice field
FieldType.DATE              # Date/time field
FieldType.NUMBER            # Numeric field
FieldType.DURATION         # Duration field
FieldType.RATING           # Rating field
FieldType.FORMULA          # Formula field
FieldType.ROLLUP           # Rollup field
FieldType.COUNT            # Count field
FieldType.LINK             # Record link field
FieldType.CREATED_TIME     # Creation timestamp field
FieldType.LAST_MODIFIED_TIME  # Last modified timestamp field
FieldType.CREATED_BY       # Creator reference field
FieldType.LAST_MODIFIED_BY  # Last modifier reference field
FieldType.AUTO_NUMBER      # Auto-incrementing number field
FieldType.BUTTON           # Button field
```

### Field Options

Base class and specialized options for different field types.

```python
from teable import FieldOption, SelectOption, NumberOption, DateOption

# Select field options
select_options = SelectOption(
    choices=["Option 1", "Option 2"],
    default_value="Option 1"  # or ["Option 1"] for multiple select
)

# Number field options
number_options = NumberOption(
    precision=2,
    min_value=0,
    max_value=100,
    format="0.00"
)

# Date field options
date_options = DateOption(
    format="YYYY-MM-DD",
    include_time=True,
    timezone="UTC"
)
```

### Field

Base class for all field types.

```python
from teable import Field

# Field properties
field.field_id: str           # Unique identifier
field.name: str               # Display name
field.field_type: FieldType   # Type of the field
field.description: Optional[str]  # Field description
field.options: Optional[FieldOption]  # Field-specific options
field.is_required: bool       # Required flag
field.is_primary: bool        # Primary field flag
field.is_computed: bool       # Computed field flag
```

#### Methods

```python
# Validate field value
def validate_value(self, value: Any) -> None

# Convert to dictionary
def to_dict(self) -> Dict[str, Any]

# Create from API response
@classmethod
def from_api_response(cls, data: Dict[str, Any]) -> Field
```

The validate_value() method performs type-specific validation:
- Required fields cannot be None
- Computed fields cannot be set directly
- Select fields validate against choices
- Number fields validate against min/max values
- Date fields validate format
- Text fields validate type is string
- Checkbox fields validate type is boolean

## View Models

### Position (View)

Enumeration of position options for view ordering.

```python
from teable import Position

Position.BEFORE  # Position before anchor
Position.AFTER   # Position after anchor
```

### SortDirection

Enumeration of sort directions.

```python
from teable import SortDirection

SortDirection.ASCENDING   # Sort in ascending order
SortDirection.DESCENDING  # Sort in descending order
```

### FilterOperator

Enumeration of filter operators.

```python
from teable import FilterOperator

FilterOperator.EQUALS                  # =
FilterOperator.NOT_EQUALS             # !=
FilterOperator.GREATER_THAN           # >
FilterOperator.GREATER_THAN_OR_EQUALS # >=
FilterOperator.LESS_THAN              # <
FilterOperator.LESS_THAN_OR_EQUALS    # <=
FilterOperator.CONTAINS               # contains
FilterOperator.NOT_CONTAINS           # notContains
FilterOperator.STARTS_WITH            # startsWith
FilterOperator.ENDS_WITH              # endsWith
FilterOperator.IS_EMPTY              # isEmpty
FilterOperator.IS_NOT_EMPTY          # isNotEmpty
FilterOperator.IN                    # in
FilterOperator.NOT_IN                # notIn
FilterOperator.BETWEEN               # between
```

### FilterCondition

Represents a single filter condition.

```python
from teable import FilterCondition

# Create a filter condition
condition = FilterCondition(
    field="field_id",  # or Field object
    operator=FilterOperator.EQUALS,
    value="some value"
)
```

### SortCondition

Represents a sort condition.

```python
from teable import SortCondition

# Create a sort condition
sort = SortCondition(
    field="field_id",  # or Field object
    direction=SortDirection.ASCENDING
)
```

### QueryBuilder

Builder pattern implementation for constructing API queries.

```python
from teable import QueryBuilder

# Create and configure a query
query = QueryBuilder()
    .filter("field1", FilterOperator.EQUALS, "value1")
    .filter("field2", FilterOperator.GREATER_THAN, 100)
    .sort("field3", SortDirection.DESCENDING)
    .paginate(take=50, skip=0)
    .search("search text", field="field4")
    .set_view("view_id")

# Build final query parameters
params = query.build()
```

### View

Represents a view in a table.

```python
from teable import View

# View properties
view.view_id: str            # Unique identifier
view.name: str               # Display name
view.description: Optional[str]  # View description
view.filters: List[FilterCondition]  # View's filter conditions
view.sorts: List[SortCondition]    # View's sort conditions
```

#### Methods

```python
# Create a query builder from view settings
def create_query(self) -> QueryBuilder

# Update view order
def update_order(
    self,
    table_id: str,
    anchor_id: str,
    position: Position
) -> None

# Convert to dictionary
def to_dict(self) -> Dict[str, Any]

# Create from API response
@classmethod
def from_api_response(
    cls,
    data: Dict[str, Any],
    client: Any = None
) -> View
```

## Next Steps

- [Client Reference](client.md)
- [Exceptions Reference](exceptions.md)
- [Best Practices](../advanced/best-practices.md)
