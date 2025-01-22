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
) -> Dict[str, Dict[str, str]]

# Get collaborators
space.get_collaborators(
    include_system: Optional[bool] = None,
    include_base: Optional[bool] = None,
    skip: Optional[int] = None,
    take: Optional[int] = None,
    search: Optional[str] = None
) -> Tuple[List[Collaborator], int]
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

## Base Models

### Base

Represents a base in Teable, which is a container for tables.

```python
from teable import Base, CollaboratorType

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
```

#### Methods

```python
# Get fields
@property
def fields(self) -> List[Field]

# Get views
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
    filter: Optional[Dict[str, Any]] = None,
    order_by: Optional[str] = None,
    take: Optional[int] = None,
    skip: Optional[int] = None
) -> List[Record]
```

## Record Models

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
```

## Field Models

### Field

Base class for all field types.

```python
from teable import Field

# Field properties
field.field_id: str           # Unique identifier
field.name: str               # Display name
field.field_type: str         # Field type identifier
field.description: Optional[str]  # Field description
field.is_required: bool       # Required flag
field.is_primary: bool        # Primary field flag
field.is_computed: bool       # Computed field flag
```

#### Methods

```python
# Validate field value
def validate_value(self, value: Any) -> None

# Convert value to API format
def to_api_value(self, value: Any) -> Any

# Convert value from API format
def from_api_value(self, value: Any) -> Any
```

## View Models

### View

Represents a view of a table.

```python
from teable import View

# View properties
view.view_id: str            # Unique identifier
view.name: str               # Display name
view.type: str              # View type
view.filter: Optional[Dict]  # View filter configuration
view.sort: Optional[List]    # View sort configuration
```

#### Methods

```python
# Update view
def update(self, config: Dict[str, Any]) -> View

# Delete view
def delete(self) -> bool
```

## Next Steps

- [Client Reference](client.md)
- [Exceptions Reference](exceptions.md)
- [Best Practices](../advanced/best-practices.md)
