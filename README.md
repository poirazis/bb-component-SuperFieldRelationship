# Super Field - Relationship

A powerful relationship field component for Budibase applications supporting database links, table relationships, and data connections.

## 🚀 Features

### Relationship Management

- **Database Links**: Connect to related tables and records
- **Self-Referencing**: Support for hierarchical data structures
- **Multiple Control Types**: Select dropdown or table picker interfaces
- **Dynamic Filtering**: Filter related records based on conditions
- **Default Values**: Pre-selected relationship values

### Data Selection

- **Table Picker**: Visual table interface for record selection
- **Search & Filter**: Quick finding of related records
- **Column Configuration**: Customize displayed columns in picker
- **Item Limits**: Control number of displayed options
- **Wide Mode**: Expanded picker interface

### Advanced Features

- **Create New**: Inline creation of new related records
- **Reordering**: Drag-and-drop reordering of relationships
- **Numbering**: Automatic numbering for ordered relationships
- **Event Handling**: On change and create new events
- **Validation**: Relationship-specific validation rules

### User Experience

- **Intuitive Interfaces**: Familiar selection and picker controls
- **Visual Feedback**: Clear relationship display and management
- **Auto-complete**: Quick record finding and selection
- **Context Actions**: Row-specific action buttons
- **Accessibility**: Full keyboard navigation support

### Display Modes

- **Text Mode**: Simple text display of relationships
- **Pills Mode**: Visual pill-style relationship indicators
- **Icon Support**: Visual indicators for relationship fields
- **Help Text**: Guidance for relationship selection

### Styling & Layout

- **Flexible Positioning**: Label placement options
- **Field Modes**: Form input or inline editing
- **Size Configuration**: Adjustable component width
- **Theme Integration**: Consistent with Budibase design

## 📝 Usage Instructions

### Basic Setup

1. Add the Super Field - Relationship component to your form
2. Bind to a relationship/link field
3. Select control type (select or table picker)
4. Configure filtering and display options

### Advanced Configuration

- **Table Selection**: Set up related table connections
- **Filtering**: Add conditions for related record filtering
- **Picker Columns**: Customize columns shown in table picker
- **Create New**: Enable inline record creation
- **Events**: Attach actions to relationship changes

### Common Use Cases

- **Parent-Child Relationships**: Hierarchical data structures
- **Lookup Tables**: Reference data selection
- **User Assignments**: Assign users to tasks or projects
- **Category Associations**: Link items to categories
- **Document Relationships**: Connect related documents

## 🔧 Configuration Options

| Setting        | Type    | Description                      |
| -------------- | ------- | -------------------------------- |
| Field          | Link    | Relationship field binding       |
| Label          | String  | Display label text               |
| Placeholder    | String  | Selection guidance text          |
| Default Value  | Link    | Pre-selected relationship        |
| Help Text      | String  | Help/instruction text            |
| Validation     | Rules   | Relationship validation          |
| Filter         | Filter  | Related record filtering         |
| Control Type   | Select  | Select/Table picker interface    |
| Picker Columns | Columns | Display columns for table picker |
| Limit Items    | Number  | Maximum displayed items          |
| Wide           | Boolean | Expanded picker mode             |
| Disabled       | Boolean | Disable relationship selection   |
| Read Only      | Boolean | Read-only mode                   |
| Reordering     | Boolean | Enable drag-and-drop reordering  |
| Numbering      | Boolean | Show item numbering              |
| Create New     | Boolean | Allow inline record creation     |
| Icon           | Icon    | Visual indicator icon            |
| Field Mode     | Select  | Form or inline input style       |
| Label Position | Select  | Label placement                  |
| Size           | Number  | Component width span             |
| Mode           | Select  | Text or Pills display mode       |

## 📋 Events

### On Change

Triggered when relationship selection changes.

**Context:**

- `value`: Selected relationship value
- `field`: The bound field information

### On Create New

Triggered when creating a new related record.

**Context:**

- Related record creation context

## 🎨 Styling

The component supports advanced styling options:

- **Picker Interface**: Customizable table picker appearance
- **Relationship Display**: Text or pill-style visualization
- **Action Buttons**: Styled row action controls
- **Responsive**: Mobile-optimized selection interfaces

## 🔍 Best Practices

- Use table picker for complex relationship selection
- Implement filtering to improve performance
- Consider user permissions for record creation
- Provide clear relationship labels and descriptions
- Test with various relationship sizes and complexity
- Use appropriate display modes for different contexts
