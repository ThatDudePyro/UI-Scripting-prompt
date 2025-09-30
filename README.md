# Nighty UI Scripting Framework Reference v1.0

**Created by:** thatdudepyro using https://claude.ai

**Last Updated:** September 2025

---

## 1. Overview

Nighty UI Scripting is a Python-based framework for creating dynamic custom tabs within the Nighty Discord selfbot application. Scripts use a hierarchical component structure to build responsive layouts with interactive elements.

**Core Principle**: Components must be created through parent methods, never directly instantiated.

### 1.1 Prohibited Patterns

The following will cause script failure:

```python
# NEVER do this - Direct instantiation is forbidden
tab = Tab()
container = CardContainer()
card = Card()
group = Group()
element = UI.Text()
```

Always use parent creation methods:
```python
# CORRECT - Use parent methods
tab = Tab(name="MyTab")
container = tab.create_container()
card = container.create_card()
group = card.create_group()
element = card.create_ui_element(UI.Text, content="Hello")
```

### 1.2 Critical Requirements

* **Hierarchy Enforcement**: Tab → CardContainer → Card → UI Elements/Groups
* **Unique Names**: Tab names must be unique across the entire UI (max 36 characters)
* **Final Render**: Always call `tab.render()` as the last step
* **At Least One Container**: Every Tab requires at least one CardContainer
* **No Card Nesting**: Cards cannot contain other Cards

### 1.3 Framework Limitations

* CardContainer nesting beyond 3 levels degrades UI performance
* Limited to 39 predefined icons for tabs
* Cards must be direct children of CardContainers only
* UI Elements must be direct children of Cards or Groups

---

## 2. Component Hierarchy

### 2.1 Required Structure

```python
Tab (Root - Required)
├── CardContainer (At least 1 required)
│   ├── Card (Content container)
│   │   ├── Group (Optional organizer)
│   │   │   ├── UI Elements
│   │   │   └── Nested Groups (unlimited depth)
│   │   └── Direct UI Elements
│   └── Nested CardContainers (max 3 levels)
└── Additional CardContainers
```

### 2.2 Layout Philosophy

The framework uses flexbox-based layouts:
- **Containers**: Manage spatial arrangement of Cards
- **Cards**: Primary content holders with alignment control
- **Groups**: Organize UI elements into logical units
- **Elements**: Interactive components (buttons, inputs, text, etc.)

---

## 3. Core Components

### 3.1 Tab Class (Root Component)

The top-level container representing a custom tab in Nighty's interface.

```python
tab = Tab(
    name="UniqueTabName",        # Required: string, max 36 chars, globally unique
    icon="users",                # Optional: predefined icon (see Section 3.1.1)
    gap=6,                       # Optional: 0-10, spacing between containers
    title="Display Title"        # Optional: shown above cards
)

# Methods
container = tab.create_container(type="columns", **kwargs)
tab.toast(title="Alert", description="Message", type="INFO")  # "INFO"|"ERROR"|"SUCCESS"
tab.render()  # REQUIRED - call after all elements are defined
```

**Parameters**:
- `name` (Required): Unique identifier, max 36 characters
- `icon` (Optional): Visual tab icon from predefined set
- `gap` (Optional, 0-10, default 6): Spacing between CardContainer elements
- `title` (Optional): Display title shown above cards

**Methods**:
- `create_container(type, **kwargs)`: Creates CardContainer instances
- `render()`: Initializes the tab (mandatory final step)
- `toast(title, description, type)`: Shows notifications

#### 3.1.1 Available Icons (39 Total)

report, chart, two_way, inbox, star, search, variants, message, users, history, clean, magic, eraser, convert, cloud, calc, calendar, key, gift, trophy, book, sun, cart, pill, bookmark, mail, bear, heart, music, umbrella, tube, palette, lock, fire, preferences, power, share, bell, trash, nitro

### 3.2 CardContainer Class (Layout Manager)

Manages spatial arrangement of Card components using flexbox layouts.

```python
# Created via tab.create_container() or container.create_container()
container = parent.create_container(
    type="columns",              # "columns"|"rows", default "columns"
    width="full",               # "auto"|"full", default "full"
    height="full",              # "auto"|"full", default "full"
    gap=6                       # 0-10, default 6
)

# Methods
nested_container = container.create_container(type="rows", **kwargs)
card = container.create_card(**kwargs)
```

**Layout Types**:
- `"columns"`: Horizontal arrangement (side-by-side)
- `"rows"`: Vertical arrangement (stacked)

**Size Control**:
- `"auto"`: Size based on content
- `"full"`: Occupy all available space

### 3.3 Card Class (Content Container)

Primary content container that houses UI elements and Groups.

```python
# Created via container.create_card()
card = container.create_card(
    type="columns",              # "columns"|"rows", default "columns"
    height="full",              # "auto"|"full", default "full"
    width="full",               # "auto"|"full", default "full"
    vertical_align="start",     # "start"|"center"|"end", default "start"
    horizontal_align="start",   # "start"|"center"|"end", default "start"
    gap=6,                      # 0-10, default 6
    disallow_shrink=False       # True|False, prevents auto-resizing
)

# Methods
group = card.create_group(type="columns", **kwargs)
element = card.create_ui_element(UI.Text, content="Hello", **kwargs)
```

**Alignment Options**:
- `"start"`: Align to beginning (top/left)
- `"center"`: Center alignment
- `"end"`: Align to end (bottom/right)

### 3.4 Group Class (Element Organizer)

Organizes multiple UI elements into cohesive units within Cards.

```python
# Created via card.create_group() or group.create_group()
group = parent.create_group(
    type="columns",              # "columns"|"rows", default "columns"
    vertical_align="start",     # "start"|"center"|"end", default "start"
    horizontal_align="start",   # "start"|"center"|"end", default "start"
    gap=6,                      # 0-10, default 6
    full_width=False            # True|False, forces full width
)

# Methods
nested_group = group.create_group(type="rows", **kwargs)
element = group.create_ui_element(UI.Button, label="Click", **kwargs)
```

**Nesting**: Groups support unlimited nesting depth for complex layouts.

---

## 4. UI Elements

All UI elements are created via `parent.create_ui_element(UI.ElementType, **kwargs)` where parent is a Card or Group.

### 4.1 UI.Text

Text display component with comprehensive typography control.

```python
text = parent.create_ui_element(UI.Text,
    content="Hello World\nNew line",    # string, use \n for line breaks
    size="base",                        # "tiny"|"sm"|"base"|"lg"|"xl"|"2xl"
    weight="normal",                    # "thin"|"light"|"normal"|"medium"|"bold"|"extrabold"|"black"
    color="#FFFFFF",                    # CSS color value
    align="left",                       # "left"|"center"|"right"
    margin="m-0",                       # Tailwind margin classes
    visible=True                        # True|False
)

# All attributes are read/write
text.content = "Updated text"
text.size = "xl"
```

### 4.2 UI.Button

Interactive button component with event handling and visual variants.

```python
button = parent.create_ui_element(UI.Button,
    label="Click Me",                   # string
    variant="solid",                    # "solid"|"bordered"|"ghost"|"light"|"flat"|"cta"
    color="primary",                    # "primary"|"default"|"success"|"danger"
    size="md",                          # "sm"|"md"
    full_width=False,                   # True|False
    disabled=False,                     # True|False
    loading=False,                      # True|False, shows spinner
    margin="m-0",                       # Tailwind margin classes
    visible=True                        # True|False
)

# Event handling (supports async functions)
def click_handler():
    button.loading = True
    # Perform work...
    button.loading = False

async def async_click_handler():
    button.loading = True
    # Async work like API calls...
    button.loading = False

button.onClick = click_handler  # or async_click_handler
```

### 4.3 UI.Input

Text input field with validation and state management.

```python
input_field = parent.create_ui_element(UI.Input,
    label="Email",                      # string|None
    placeholder="Enter email...",       # string|None
    value="",                          # string|None, current value
    description="Help text",           # string|None
    show_clear_button=False,           # True|False
    invalid=False,                     # True|False, validation state
    disabled=False,                    # True|False
    readonly=False,                    # True|False
    required=False,                    # True|False
    full_width=False,                  # True|False
    error_message="Error text",        # string|None
    margin="m-0",                      # Tailwind margin classes
    visible=True                       # True|False
)

# Event handling
def on_input_change(value):
    print(f"Input changed to: {value}")

input_field.onInput = on_input_change
# Access current value: input_field.value
```

### 4.4 UI.Image

Image display component with styling and layout options.

```python
image = parent.create_ui_element(UI.Image,
    url="https://example.com/image.jpg", # Required: valid URL
    alt="Description",                   # string, default "Image"
    circle=False,                       # True|False, circular cropping
    shadow=False,                       # True|False, drop shadow
    width="auto",                       # "50%"|"75px"|"auto"
    height="auto",                      # "50%"|"75px"|"auto"
    rounded="md",                       # "none"|"sm"|"md"|"lg"|"xl"|"2xl"
    fill_type="contain",               # "cover"|"contain"
    border_color="#FFFFFF",            # CSS color value
    border_width=0,                    # 0-10
    margin="m-0",                      # Tailwind margin classes
    visible=True                       # True|False
)
```

### 4.5 UI.Toggle

Binary toggle switch component.

```python
toggle = parent.create_ui_element(UI.Toggle,
    label="Enable feature",            # string
    checked=False,                     # True|False, toggle state
    disabled=False,                    # True|False
    visible=True                       # True|False
)

# Event handling
def on_toggle_change(checked):
    print(f"Toggle is now: {checked}")

toggle.onChange = on_toggle_change
```

### 4.6 UI.Select

Dropdown selection component supporting single/multiple selection modes.

```python
select = parent.create_ui_element(UI.Select,
    label="Choose option",             # Required: string
    items=[                           # Required: list of dicts
        {
            "id": "option1",          # Required: unique string
            "title": "Option 1",      # Required: display text
            "iconUrl": "https://..."  # Optional: icon URL
        },
        {"id": "option2", "title": "Option 2"}
    ],
    selected_items=["option1"],        # list of IDs currently selected
    disabled_items=[],                 # list of IDs to disable
    mode="single",                     # "single"|"multiple"
    description="Help text",           # string|None
    loading=False,                     # True|False, loading state
    disabled=False,                    # True|False
    invalid=False,                     # True|False, validation state
    error_message="Error text",        # string|None
    full_width=False,                  # True|False
    margin="m-0",                      # Tailwind margin classes
    visible=True                       # True|False
)

# Event handling
def on_selection_change(selected_items):
    print(f"Selected: {selected_items}")

select.onChange = on_selection_change
```

**Important**: All item IDs must be unique to prevent rendering issues.

### 4.7 UI.Checkbox

Binary checkbox input component.

```python
checkbox = parent.create_ui_element(UI.Checkbox,
    label="I agree",                   # Required: string
    checked=False,                     # True|False, checked state
    disabled=False,                    # True|False
    invalid=False,                     # True|False, validation state
    visible=True                       # True|False
)

# Event handling
def on_checkbox_change(checked):
    print(f"Checkbox is: {checked}")

checkbox.onChange = on_checkbox_change
```

### 4.8 UI.Table

Complex data table component with search, pagination, and selection capabilities.

```python
table = parent.create_ui_element(UI.Table,
    columns=[                          # Required: list of column definitions
        {
            "type": "text",            # "tag"|"text"|"button"
            "label": "Name",           # Column header
        },
        {
            "type": "button",
            "label": "Actions",
            "buttons": [               # For "button" type columns
                {
                    "label": "Edit",
                    "color": "default", # "default"|"danger"
                    "onClick": edit_function
                }
            ]
        }
    ],
    rows=[                            # List of row data
        {
            "id": "row1",             # Required: unique ID per row
            "cells": [                # Must match column structure
                {"text": "John", "subtext": "Admin", "imageUrl": "url"}, # text type
                {"text": "Active", "color": "green"},                    # tag type
                {}                                                       # button type
            ]
        }
    ],
    search=True,                      # True|False, enable search
    items_per_page=50,               # int, pagination size
    selectable=False,                # True|False, row selection
    visible=True                     # True|False
)

# Event handling
def on_selection_change(selected_row_ids):
    print(f"Selected rows: {selected_row_ids}")

table.onSelectionChange = on_selection_change
```

**Column Types**:
- `"text"`: Display text with optional subtext and image
- `"tag"`: Colored label display
- `"button"`: Action buttons with click handlers

---

## 5. Shared Attributes

### 5.1 Gap Property

Universal spacing control between child elements (range 0-10):
- `0`: No spacing
- `10`: Maximum spacing
- Applied to: Tab, CardContainer, Card, Group

### 5.2 Margin Property

Tailwind CSS margin class support:
- Available prefixes: `m-` `mt-` `mr-` `mb-` `ml-` `mx-` `my-`
- Range: 0-10 (e.g., `"m-4"`, `"mx-3"`, `"mb-2"`)
- Applied to: All UI Elements

**Examples**:
- `"m-4"`: 4 units margin on all sides
- `"mx-3"`: 3 units margin on left and right
- `"mt-2"`: 2 units margin on top

### 5.3 Visible Property

Universal visibility control:
- `True`: Element is displayed
- `False`: Element is hidden
- Applied to: All UI Elements

---

## 6. Common Layout Patterns

### 6.1 Basic Single Card Layout

```python
tab = Tab(name="SimpleTab")
container = tab.create_container()
card = container.create_card()
card.create_ui_element(UI.Text, content="Hello World")
tab.render()
```

### 6.2 Two Column Layout

```python
tab = Tab(name="TwoColumns")
container = tab.create_container(type="columns")

# Left column
left_card = container.create_card()
left_card.create_ui_element(UI.Text, content="Left Content")

# Right column  
right_card = container.create_card()
right_card.create_ui_element(UI.Text, content="Right Content")

tab.render()
```

### 6.3 Form with Grouped Elements

```python
tab = Tab(name="ContactForm")
container = tab.create_container()
card = container.create_card(vertical_align="center", horizontal_align="center")

# Form title
card.create_ui_element(UI.Text, content="Contact Information", size="xl", weight="bold")

# Input and button in same row using group
input_group = card.create_group(type="columns", gap=2)
email_input = input_group.create_ui_element(UI.Input, 
    label="Email", 
    placeholder="your@email.com",
    full_width=True
)
submit_button = input_group.create_ui_element(UI.Button, 
    label="Submit", 
    variant="cta"
)

def handle_submit():
    email = email_input.value
    if email:
        submit_button.loading = True
        # Process submission...
        submit_button.loading = False
    else:
        email_input.invalid = True
        email_input.error_message = "Email is required"

submit_button.onClick = handle_submit
tab.render()
```

### 6.4 Complex Nested Layout

```python
tab = Tab(name="Dashboard", icon="chart")

# Main container with side-by-side layout
main_container = tab.create_container(type="columns")

# Left sidebar with vertical stack
sidebar_container = main_container.create_container(type="rows")
sidebar_container.width = "auto"  # Let it size based on content

# Navigation card
nav_card = sidebar_container.create_card()
nav_card.create_ui_element(UI.Text, content="Navigation", size="lg", weight="bold")
nav_card.create_ui_element(UI.Button, label="Dashboard", variant="ghost", full_width=True)
nav_card.create_ui_element(UI.Button, label="Reports", variant="ghost", full_width=True)

# Settings card
settings_card = sidebar_container.create_card()
settings_card.create_ui_element(UI.Text, content="Settings", size="lg", weight="bold")
settings_card.create_ui_element(UI.Toggle, label="Dark Mode")
settings_card.create_ui_element(UI.Toggle, label="Notifications")

# Main content area
content_card = main_container.create_card()
content_card.create_ui_element(UI.Text, content="Main Content Area", size="2xl")

tab.render()
```

### 6.5 Interactive Data Table

```python
def edit_user():
    print("Edit user clicked")

def delete_user():
    print("Delete user clicked")

table = card.create_ui_element(UI.Table,
    columns=[
        {"type": "text", "label": "User"},
        {"type": "tag", "label": "Status"},
        {"type": "button", "label": "Actions", "buttons": [
            {"label": "Edit", "color": "default", "onClick": edit_user},
            {"label": "Delete", "color": "danger", "onClick": delete_user}
        ]}
    ],
    rows=[
        {"id": "user1", "cells": [
            {"text": "John Doe", "subtext": "Administrator", "imageUrl": "https://example.com/avatar1.png"},
            {"text": "Active", "color": "green"},
            {}  # Button column cells are empty
        ]},
        {"id": "user2", "cells": [
            {"text": "Jane Smith", "subtext": "User"},
            {"text": "Inactive", "color": "red"},
            {}
        ]}
    ],
    search=True,
    selectable=True,
    items_per_page=25
)

def on_user_selection(selected_ids):
    print(f"Selected users: {selected_ids}")

table.onSelectionChange = on_user_selection
```

---

## 7. Best Practices

### 7.1 Component Creation

1. **Always use parent methods**: Never instantiate components directly
2. **Unique identification**: Ensure Tab names and table/select item IDs are unique
3. **Proper nesting**: Follow the required hierarchy strictly
4. **Final render**: Always call `tab.render()` as the last step

### 7.2 Layout Design

1. **Container limits**: Avoid nesting CardContainers beyond 3 levels
2. **Responsive sizing**: Mix `"auto"` and `"full"` width/height for responsive behavior
3. **Alignment control**: Use alignment properties for precise positioning
4. **Gap management**: Use consistent gap values for visual cohesion

### 7.3 Event Handling

1. **Async support**: Event handlers can be synchronous or asynchronous functions
2. **Loading states**: Use loading attributes for buttons during async operations
3. **Validation feedback**: Use invalid/error_message attributes for form validation
4. **State management**: Leverage read/write attributes for dynamic UI updates

### 7.4 Performance Considerations

1. **Efficient filtering**: Use conditional rendering with the `visible` attribute
2. **Minimal nesting**: Keep component hierarchy as flat as reasonable
3. **Event optimization**: Avoid heavy computations in event handlers
4. **State batching**: Update multiple attributes together when possible

---

## 8. Common Pitfalls

### 8.1 Creation Errors

```python
# WRONG - Direct instantiation
tab = Tab()

# CORRECT - Proper instantiation
tab = Tab(name="MyTab")
```

### 8.2 Hierarchy Violations

```python
# WRONG - Card nested in Card
outer_card = container.create_card()
inner_card = outer_card.create_card()  # Not allowed!

# CORRECT - Use Groups for organization
outer_card = container.create_card()
group = outer_card.create_group()
```

### 8.3 Missing Required Parameters

```python
# WRONG - Missing required parameters
select = card.create_ui_element(UI.Select)  # Missing label

# CORRECT - Include required parameters
select = card.create_ui_element(UI.Select, label="Choose Option", items=[])
```

### 8.4 Forgetting Final Render

```python
tab = Tab(name="MyTab")
container = tab.create_container()
card = container.create_card()
# WRONG - Missing tab.render()

# CORRECT - Always include final render
tab.render()
```

---

## 9. Debugging Tips

### 9.1 Component Verification

1. **Check hierarchy**: Ensure proper parent-child relationships
2. **Verify unique IDs**: Confirm all Tab names and item IDs are unique
3. **Validate parameters**: Check that all required parameters are provided
4. **Test incrementally**: Build UI step by step to isolate issues

### 9.2 Layout Debugging

1. **Gap visualization**: Use different gap values to see spacing
2. **Alignment testing**: Try different alignment combinations
3. **Size debugging**: Switch between "auto" and "full" sizing
4. **Container inspection**: Check container types and nesting levels

### 9.3 Event Debugging

1. **Console logging**: Add print statements in event handlers
2. **State inspection**: Check attribute values before/after events
3. **Async handling**: Ensure async functions are properly awaited
4. **Error catching**: Wrap event handlers in try-catch blocks

---

This reference provides complete coverage of the Nighty UI Scripting framework for AI assistants helping users create dynamic, interactive custom tabs within the Nighty Discord selfbot application.
