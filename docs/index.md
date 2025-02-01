# Teable Client Library

A professional Python client library for interacting with the Teable API. This library provides a comprehensive, object-oriented interface for managing spaces, bases, tables, records, fields, and views in Teable.

## Overview

The Teable Client Library is designed to provide a seamless interface to Teable's API, offering:

### 🔐 Authentication & Authorization
- API key-based authentication for basic operations
- Full email/password authentication for advanced operations
- Role-based access control and permissions management

### 📊 Data Management
- Complete CRUD operations for spaces, bases, tables, and records
- Efficient batch operations for bulk data handling
- Advanced querying with filtering, sorting, and pagination
- Field type validation and configuration

### 🎯 Key Features
- **Hierarchical Structure**: Follows Teable's Space → Base → Table → Record hierarchy
- **Type Safety**: Comprehensive type hints for better IDE support
- **Error Handling**: Detailed error messages and exception handling
- **Resource Management**: Efficient handling of API resources and rate limits
- **Batch Operations**: Optimized bulk operations for records and tables
- **Query Builder**: Intuitive interface for complex data queries
- **Field Types**: Support for all Teable field types with validation

## Requirements

- Python 3.7+
- Valid Teable API key
- For advanced operations: Teable account credentials

## Quick Installation

```bash
pip install teable-client
```

## Basic Usage

```python
from teable import TeableClient, TeableConfig

# Initialize with API key
client = TeableClient(TeableConfig(
    api_key="your-api-key",
    api_url="https://api.teable.io"
))

# For operations requiring authentication
client.auth.signin(email="your-email", password="your-password")

# Get a table
table = client.get_table("table_id")

# Create a record
record = table.create_record({
    "Name": "John Doe",
    "Email": "john@example.com"
})

# Query records
results = table.get_records(
    filter={"fieldName": "Email", "operator": "contains", "value": "@example.com"},
    sort=[{"field": "Name", "order": "asc"}],
    take=10
)
```

## Important Notes

1. **Authentication Levels**:
   - API key alone provides limited access
   - Full access requires both API key and user authentication
   - Some operations (like space management) always require user authentication

2. **Resource Hierarchy**:
   - Spaces contain bases
   - Bases contain tables
   - Tables contain records
   - Each level has its own permissions and access controls

3. **Best Practices**:
   - Always handle exceptions appropriately
   - Use batch operations for bulk data management
   - Implement proper rate limiting in your application
   - Close the client when finished to free resources

## Documentation Structure

This documentation is organized into several sections:

- **Getting Started**: Basic setup and quick start guide
- **Core Concepts**: Understanding the fundamental components
- **Advanced Topics**: Detailed guides for specific features
- **API Reference**: Complete API documentation
- **Examples**: Real-world usage examples

Choose a section from the navigation menu to get started!
