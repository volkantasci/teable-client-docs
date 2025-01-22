# Data Synchronization

This guide covers data synchronization capabilities in Teable, including real-time sync, batch synchronization, and integration patterns.

## Real-time Synchronization

### Watching for Changes

```python
from teable import TeableClient, TeableConfig

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Get your table
table = client.get_table("table_id")

# Set up change listener
def handle_change(change_event):
    """Handle table changes."""
    event_type = change_event['type']  # 'create', 'update', or 'delete'
    record_id = change_event['recordId']
    changes = change_event.get('changes', {})
    
    print(f"Event type: {event_type}")
    print(f"Record ID: {record_id}")
    print(f"Changes: {changes}")

# Start watching for changes
table.watch(callback=handle_change)
```

### Real-time Updates

```python
# Sync handler with real-time updates
def sync_handler(change_event):
    """Handle sync events."""
    if change_event['type'] == 'create':
        # Handle new record
        sync_new_record(change_event['record'])
    elif change_event['type'] == 'update':
        # Handle updated record
        sync_updated_record(
            change_event['recordId'],
            change_event['changes']
        )
    elif change_event['type'] == 'delete':
        # Handle deleted record
        sync_deleted_record(change_event['recordId'])

def sync_new_record(record):
    """Sync new record to external system."""
    # Implement sync logic
    pass

def sync_updated_record(record_id, changes):
    """Sync record updates to external system."""
    # Implement sync logic
    pass

def sync_deleted_record(record_id):
    """Handle deleted record in external system."""
    # Implement sync logic
    pass
```

## Batch Synchronization

### Full Sync

```python
def full_sync(source_table, target_table):
    """Perform full sync between tables."""
    # Get all records from source
    source_records = source_table.get_records()
    
    # Get existing records from target
    target_records = {
        record.record_id: record
        for record in target_table.get_records()
    }
    
    # Prepare batches
    to_create = []
    to_update = []
    to_delete = set(target_records.keys())
    
    # Compare records
    for record in source_records:
        record_id = record.record_id
        if record_id in target_records:
            # Update existing record
            if record.fields != target_records[record_id].fields:
                to_update.append({
                    'recordId': record_id,
                    'fields': record.fields
                })
            to_delete.remove(record_id)
        else:
            # Create new record
            to_create.append(record.fields)
    
    # Execute changes
    if to_create:
        target_table.batch_create_records(to_create)
    if to_update:
        target_table.batch_update_records(to_update)
    if to_delete:
        target_table.batch_delete_records(list(to_delete))
```

### Incremental Sync

```python
def incremental_sync(source_table, target_table, last_sync_time):
    """Perform incremental sync between tables."""
    # Get records modified since last sync
    modified_records = source_table.get_records(
        filter={
            "field": "last_modified_time",
            "operator": ">",
            "value": last_sync_time
        }
    )
    
    # Prepare batches
    to_create = []
    to_update = []
    
    # Get existing record IDs
    existing_ids = {
        record.record_id
        for record in target_table.get_records(
            projection=["record_id"]
        )
    }
    
    # Sort records
    for record in modified_records:
        if record.record_id in existing_ids:
            to_update.append({
                'recordId': record.record_id,
                'fields': record.fields
            })
        else:
            to_create.append(record.fields)
    
    # Execute changes
    if to_create:
        target_table.batch_create_records(to_create)
    if to_update:
        target_table.batch_update_records(to_update)
    
    return modified_records[-1].last_modified_time if modified_records else last_sync_time
```

## Integration Patterns

### External System Integration

```python
class ExternalSystemSync:
    """Sync manager for external system integration."""
    
    def __init__(self, table, external_client):
        self.table = table
        self.external_client = external_client
        self.sync_status = {}
    
    def start_sync(self):
        """Start synchronization process."""
        # Set up change listener
        self.table.watch(callback=self.handle_change)
    
    def handle_change(self, change_event):
        """Handle table changes."""
        try:
            if change_event['type'] == 'create':
                self.sync_to_external(change_event['record'])
            elif change_event['type'] == 'update':
                self.update_external(
                    change_event['recordId'],
                    change_event['changes']
                )
            elif change_event['type'] == 'delete':
                self.delete_from_external(change_event['recordId'])
                
            self.sync_status[change_event['recordId']] = {
                'status': 'success',
                'timestamp': datetime.now()
            }
            
        except Exception as e:
            self.sync_status[change_event['recordId']] = {
                'status': 'error',
                'error': str(e),
                'timestamp': datetime.now()
            }
    
    def sync_to_external(self, record):
        """Sync record to external system."""
        # Implement external system sync
        pass
    
    def update_external(self, record_id, changes):
        """Update record in external system."""
        # Implement external system update
        pass
    
    def delete_from_external(self, record_id):
        """Delete record from external system."""
        # Implement external system deletion
        pass
```

## Best Practices

1. **Sync Design**
   - Use appropriate sync patterns
   - Handle conflicts gracefully
   - Implement error recovery
   - Monitor sync status

2. **Performance**
   - Use batch operations
   - Implement incremental sync
   - Optimize data transfer
   - Monitor sync performance

3. **Error Handling**
   - Implement retry logic
   - Log sync failures
   - Provide status updates
   - Handle edge cases

4. **Maintenance**
   - Regular sync audits
   - Monitor sync logs
   - Clean up sync data
   - Update sync configurations

## Next Steps

- [Workflow Automation](../automation/workflows.md)
- [Error Handling](../advanced/error-handling.md)
- [Best Practices](../advanced/best-practices.md)
- [API Reference](../api/client.md)
