# Workflow Automation

This guide covers workflow automation capabilities in Teable, including triggers, actions, and automation patterns.

## Triggers

### Record-Based Triggers

```python
from teable import TeableClient, TeableConfig, Trigger

# Initialize the client
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))

# Get your table
table = client.get_table("table_id")

# Create record trigger
trigger = Trigger(
    name="New Record Trigger",
    event="record.created",
    table_id=table.table_id,
    conditions=[
        {
            "field": "Status",
            "operator": "=",
            "value": "New"
        }
    ]
)

# Register trigger handler
@trigger.handler
def handle_new_record(event):
    """Handle new record creation."""
    record = event.record
    print(f"New record created: {record.record_id}")
    # Implement trigger logic
```

### Field Change Triggers

```python
# Create field change trigger
field_trigger = Trigger(
    name="Status Change Trigger",
    event="field.changed",
    table_id=table.table_id,
    field_id="status_field_id",
    conditions=[
        {
            "from": "In Progress",
            "to": "Completed"
        }
    ]
)

@field_trigger.handler
def handle_status_change(event):
    """Handle status field changes."""
    record = event.record
    old_value = event.old_value
    new_value = event.new_value
    print(f"Status changed from {old_value} to {new_value}")
    # Implement trigger logic
```

## Actions

### Record Actions

```python
from teable import Action

# Create record action
action = Action(
    name="Update Related Records",
    description="Update related records when status changes"
)

@action.execute
def update_related_records(context):
    """Update related records."""
    record = context.record
    related_records = record.get_field_value("Related Records")
    
    # Update each related record
    for related_id in related_records:
        related_record = table.get_record(related_id)
        table.update_record(
            record_id=related_id,
            fields={
                "Status": "Updated",
                "Last Updated": context.timestamp
            }
        )
```

### Notification Actions

```python
# Create notification action
notification_action = Action(
    name="Send Notification",
    description="Send notification when record is assigned"
)

@notification_action.execute
def send_notification(context):
    """Send notification to assigned user."""
    record = context.record
    assignee = record.get_field_value("Assignee")
    
    if assignee:
        notification = {
            "user_id": assignee,
            "title": "New Assignment",
            "message": f"You have been assigned to {record.name}",
            "link": f"/record/{record.record_id}"
        }
        
        client.send_notification(notification)
```

## Workflow Definitions

### Creating Workflows

```python
from teable import Workflow

# Create workflow
workflow = Workflow(
    name="Task Assignment Workflow",
    description="Handle task assignments and notifications"
)

# Add triggers and actions
workflow.add_trigger(trigger)
workflow.add_action(action)
workflow.add_action(notification_action)

# Set workflow order
workflow.set_execution_order([
    "update_related_records",
    "send_notification"
])

# Enable workflow
workflow.enable()
```

### Conditional Workflows

```python
# Create conditional workflow
conditional_workflow = Workflow(
    name="Priority Task Workflow",
    description="Handle high priority tasks"
)

@conditional_workflow.condition
def check_priority(context):
    """Check if task is high priority."""
    record = context.record
    priority = record.get_field_value("Priority")
    return priority == "High"

@conditional_workflow.action
def handle_priority_task(context):
    """Handle high priority task."""
    record = context.record
    
    # Notify management
    notification = {
        "user_id": "manager_id",
        "title": "High Priority Task",
        "message": f"New high priority task: {record.name}",
        "priority": "urgent"
    }
    
    client.send_notification(notification)
    
    # Update task tracking
    table.update_record(
        record_id=record.record_id,
        fields={
            "Tracked": True,
            "Track Date": context.timestamp
        }
    )
```

## Automation Patterns

### Sequential Workflows

```python
def create_sequential_workflow(table, steps):
    """Create workflow with sequential steps."""
    workflow = Workflow(
        name="Sequential Process",
        description="Handle multi-step process"
    )
    
    current_step = 0
    
    @workflow.trigger
    def step_completed(event):
        """Trigger when step is completed."""
        nonlocal current_step
        record = event.record
        
        if current_step < len(steps):
            # Execute current step
            step = steps[current_step]
            step(record)
            current_step += 1
            
            # Update progress
            table.update_record(
                record_id=record.record_id,
                fields={
                    "Current Step": current_step,
                    "Progress": (current_step / len(steps)) * 100
                }
            )
    
    return workflow
```

### Parallel Workflows

```python
def create_parallel_workflow(table, tasks):
    """Create workflow with parallel tasks."""
    workflow = Workflow(
        name="Parallel Process",
        description="Handle parallel tasks"
    )
    
    @workflow.trigger
    def start_tasks(event):
        """Start parallel tasks."""
        record = event.record
        
        # Create task records
        for task in tasks:
            table.create_record({
                "Parent": record.record_id,
                "Task": task["name"],
                "Status": "Pending",
                "Assigned To": task["assignee"]
            })
    
    @workflow.action
    def check_completion(context):
        """Check if all tasks are complete."""
        record = context.record
        
        # Get task records
        task_records = table.get_records(
            filter={
                "field": "Parent",
                "operator": "=",
                "value": record.record_id
            }
        )
        
        # Check completion
        all_complete = all(
            task.get_field_value("Status") == "Complete"
            for task in task_records
        )
        
        if all_complete:
            table.update_record(
                record_id=record.record_id,
                fields={"Status": "Complete"}
            )
    
    return workflow
```

## Best Practices

1. **Workflow Design**
   - Keep workflows focused
   - Handle errors gracefully
   - Document workflow logic
   - Test thoroughly

2. **Performance**
   - Optimize trigger conditions
   - Use batch operations
   - Monitor execution time
   - Handle timeouts

3. **Error Handling**
   - Implement retry logic
   - Log workflow errors
   - Provide status updates
   - Handle edge cases

4. **Maintenance**
   - Regular workflow audits
   - Monitor execution logs
   - Update documentation
   - Version control workflows

## Next Steps

- [Data Synchronization](../integration/sync.md)
- [Error Handling](../advanced/error-handling.md)
- [Best Practices](../advanced/best-practices.md)
- [API Reference](../api/client.md)
