The actual library can be found here https://greasyfork.org/en/scripts/535798-immediategui

# ImmediateGUI Documentation

Generated: 2025-10-04

ImmediateGUI is a lightweight, immediate-mode UI helper for building small tool panels in the browser. It renders a floating, draggable panel with themable controls and layout primitives.

- File: ImmediateGUI.js
- Class: ImmediateGUI

---

## Table of Contents

- [Overview](#overview)
- [Properties](#properties)
- [Static Methods](#static-methods)
- [Constructor](#constructor)
- [Private/Internal Methods](#privateinternal-methods)
- [Layout Methods](#layout-methods)
- [Tab Methods](#tab-methods)
- [Control Methods](#control-methods)
- [Panel Lifecycle](#panel-lifecycle)
- [Modal/Prompt](#modalprompt)
- [Theming](#theming)

---

## Overview

- Floating, fixed-position container with title bar and content area
- Dark and light themes using CSS variables
- Controls rendered directly to DOM
- Sections, rows, indentation, and tabs for layout
- Draggable by mouse, keeps itself within viewport

---

## Properties

- static OccupiedElementIds: string[]
  - Tracks generated IDs to avoid collisions.

- theme: object
  - Active theme object from this.themes.

- themes: object
  - Built-in theme definitions (light, dark).

- options: object
  - Runtime options provided to constructor.

- container: HTMLElement
  - Root panel element.

- contentContainer: HTMLElement
  - Inner content area for controls.

- currentSection: HTMLElement | null
  - Target container for controls when inside a section.

- currentRow: HTMLElement | null
  - Target container for controls when inside a row.

- currentTab: { wrapper, panes, activeTab, currentTabIndex } | null
  - Current tab context while building tabs.

- indentationLevel: number
  - Current indentation level (applied as left margin).

- indentationSize: number
  - Pixels per indentation level.

- isCustomIndentationLevel: boolean
  - Whether the indentation level was set explicitly.

- maxHeight: string
  - Max height for the panel.

- titleBar: HTMLElement
  - Title bar element.

- minimizeBtn: HTMLButtonElement
  - Minimize toggle button.

- isMinimized: boolean
  - Current minimized state.

- escapeHTMLPolicy: Trusted Types policy or shim
  - Used to safely set innerHTML for symbols.

---

## Static Methods

### ImmediateGUI.GenerateId(prefix = "gui_")
Generates a unique element ID and registers it to avoid reuse.

- Parameters:
  - prefix: string (optional) ID prefix
- Returns: string
- Notes: Uses base36 timestamp + random part and falls back to recursion on collision.

---

## Constructor

### new ImmediateGUI(options = {})
Creates a new floating panel with a title bar and content area.

- Options:
  - theme: 'dark' | 'light' (default 'dark')
  - position: 'right' | 'left' (default 'right')
  - width: number | string (default 300)
  - draggable: boolean (default true)
  - title: string (default 'Immediate GUI')
  - titleLeftAligned: boolean (default true)
- Side Effects:
  - Injects scoped styles
  - Applies CSS variables
  - Initializes DOM structure
  - Enables dragging if configured

---

## Private/Internal Methods

Note: Methods prefixed with underscore are internal.

### _createTitleBar(leftAlign = true)
Builds the sticky title bar with a minimize toggle.

- Parameters:
  - leftAlign: boolean
- Side Effects:
  - Appends title bar to this.container
  - Wires minimize button click handler

### _flashTitleBar(duration = 2000, flashCount = 6)
Temporarily flashes the title bar background to draw attention.

- Parameters:
  - duration: number (ms)
  - flashCount: number of on/off flashes (pairs)
- Returns: this

### _toggleMinimize(forceState = null)
Toggles minimized state of the panel.

- Parameters:
  - forceState: boolean | null
- Behavior:
  - Hides/shows contentContainer
  - Adjusts container maxHeight, position, overflow
  - Updates button icon/title
  - Calls _keepInView()

### _updateCSSVariables()
Applies current theme values to document root CSS variables.

- Returns: void

### _applyGlobalStyles()
Injects a scoped <style> element with base control styles and resets.

- Returns: void
- Notes: Scoped to current container id; also styles scrollbars.

### _setupDragging()
Enables dragging the panel with the mouse.

- Behavior:
  - Starts on mousedown outside controls
  - Moves on mousemove
  - Ends on mouseup/leave
  - Calls _keepInView() after drag
- Returns: void

### _keepInView()
Repositions the panel to ensure visibility within the viewport.

- Behavior:
  - Keeps at least 50px visible on each screen edge
- Returns: void

### _applyIndentation(element)
Applies current left margin based on indentation level.

- Parameters:
  - element: HTMLElement
- Returns: HTMLElement

### _applyThemeToElements()
Updates in-DOM styles for controls to reflect current theme.

- Behavior:
  - Updates container, buttons, inputs, sections, text, toggles
- Returns: this

---

## Layout Methods

### BeginSection(title, collapsible = false, collapsedByDefault = false, tooltip = '')
Starts a new section block. Optionally collapsible.

- Parameters:
  - title: string
  - collapsible: boolean
  - collapsedByDefault: boolean
  - tooltip: string
- Behavior:
  - Non-collapsible: renders header + section container
  - Collapsible: renders header with indicator and an inner content container
- Returns: this
- Notes:
  - When collapsible, controls are appended to section.contentContainer
  - Click the header to toggle collapsed state

### EndSection()
Ends the current section.

- Returns: this

### BeginRow(gap = 2)
Starts a new horizontal row layout.

- Parameters:
  - gap: number (px)
- Behavior:
  - Children wrappers inside a row are flexed to fill width
- Returns: this

### EndRow()
Ends the current row and normalizes child sizing.

- Returns: this

### BeginIndentation(level = -1)
Increases indentation level or sets a custom level.

- Parameters:
  - level: number; -1 increments by one
- Returns: this

### EndIndentation()
Decreases indentation level or resets custom level.

- Returns: this

---

## Tab Methods

### BeginTabs(tabs = [], defaultTab = 0)
Creates a tabbed container with headers and panes.

- Parameters:
  - tabs: string[]
  - defaultTab: number (index)
- Behavior:
  - Builds clickable headers
  - Prepares panes; sets the active one
  - Sets currentSection to first pane initially
  - Tracks currentTabIndex separately for build-time targeting
- Returns: this

### SetActiveTab(tabIndexOrTabName)
Sets the pane for subsequent control additions (build-time context).

- Parameters:
  - tabIndexOrTabName: number | string
- Behavior:
  - If name provided, resolves index by pane.tabName
  - Sets currentTab.currentTabIndex and currentSection
- Returns: this
- Errors:
  - Logs error on invalid index/name

### EndTabs()
Ends the current tabset context.

- Returns: this

---

## Control Targeting

### _getTargetContainer()
Resolves the current DOM container for new controls.

- Priority:
  1. currentRow
  2. currentTab.panes[currentTab.currentTabIndex]
  3. currentSection.contentContainer (if collapsible)
  4. currentSection
  5. contentContainer
- Returns: HTMLElement

### GetControlContainer()
Gets the root panel element.

- Returns: HTMLElement

### GetControls(OfType = null)
Returns all wrapper nodes added to the panel.

- Parameters:
  - OfType: string | null (reserved for future filtering)
- Returns: NodeListOf<HTMLElement>

---

## Control Methods

All control methods append to the current target container and return a primary element (or an API object), unless noted. Most methods apply indentation to the top-level wrapper.

### Separator(plain = false)
Horizontal rule separator.

- Parameters:
  - plain: boolean (if false, draws border-top)
- Returns: HTMLHRElement

### Header(text, level = 1)
Heading element (h1–h6).

- Parameters:
  - text: string
  - level: number (1..6)
- Returns: HTMLHeadingElement

### Button(text, callback, tooltip = '')
Clickable button.

- Parameters:
  - text: string
  - callback: function (click handler)
  - tooltip: string
- Returns: HTMLButtonElement
- Notes: Adds simple hover styling

### Textbox(placeholder, defaultValue = "", tooltip = '')
Single-line text input.

- Parameters:
  - placeholder: string
  - defaultValue: string
  - tooltip: string
- Returns: HTMLInputElement (type="text")

### TextArea(placeholder = "", defaultValue = "", rows = 4, tooltip = '')
Multi-line text input.

- Parameters:
  - placeholder: string
  - defaultValue: string
  - rows: number
  - tooltip: string
- Returns: HTMLTextAreaElement
- Notes:
  - Limits max height relative to panel
  - Adjusts border on mouseenter

### Label(text)
Static label.

- Parameters:
  - text: string
- Returns: HTMLLabelElement

### ProgressBar(value = 0, min = 0, max = 100, showText = true, tooltip = '')
Progress bar with optional percentage text.

- Parameters:
  - value: number
  - min: number
  - max: number
  - showText: boolean
  - tooltip: string
- Returns: HTMLElement (bar element) augmented with:
  - setValue(newValue)
  - getValue()
  - increment()
  - decrement()
  - textElement: HTMLElement | null
  - min, max: numbers
- Notes:
  - Visually implemented with nested divs

### ColorPicker(defaultValue = '#000000', tooltip = '')
Color input paired with hex text.

- Parameters:
  - defaultValue: string (#RRGGBB)
  - tooltip: string
- Returns: HTMLInputElement (type="color")

### DatePicker(defaultValue = today, tooltip = '')
Date selector.

- Parameters:
  - defaultValue: string (YYYY-MM-DD)
  - tooltip: string
- Returns: HTMLInputElement (type="date")

### Dropdown(options = [], defaultValue = null, tooltip = '')
Select dropdown.

- Parameters:
  - options: Array<string | { text|label, value }>
  - defaultValue: string | number | null
  - tooltip: string
- Returns: HTMLSelectElement

### NumberInput(label, defaultValue = 0, min = null, max = null, tooltip = '')
Labeled numeric input.

- Parameters:
  - label: string
  - defaultValue: number
  - min: number | null
  - max: number | null
  - tooltip: string
- Returns: HTMLInputElement (type="number")
- Notes:
  - Keyup handler coerces values into [min, max] if provided

### Slider(minValue = 0, maxValue = 100, defaultValue = 50, tooltip = '')
Range slider with live value label.

- Parameters:
  - minValue: number
  - maxValue: number
  - defaultValue: number
  - tooltip: string
- Returns: HTMLInputElement (type="range")
- Augments:
  - slider.label: HTMLElement (value display)

### Checkbox(label, checked = false, tooltip = '')
Checkbox with label and theme accent.

- Parameters:
  - label: string
  - checked: boolean
  - tooltip: string
- Returns: HTMLInputElement (type="checkbox")
- Augments:
  - checkbox.label: HTMLLabelElement
- Notes:
  - Uses accent-color and a clip-path for style

### ToggleSwitch(label, checked = false, tooltip = '', onChangeCallback = null)
Custom toggle switch control.

- Parameters:
  - label: string
  - checked: boolean
  - tooltip: string
  - onChangeCallback: function(checked: boolean)
- Returns: HTMLInputElement (hidden checkbox)
- Augments (on returned input):
  - toggle(): void
  - setChecked(value: boolean): void
  - label: HTMLLabelElement
  - track: HTMLElement
  - thumb: HTMLElement
- Accessibility:
  - role="switch", tabindex=0, Space/Enter toggles
  - aria-checked maintained

### RadioButtons(options = [], defaultValue = null, tooltip = '')
Radio button group API.

- Parameters:
  - options: Array<string | { text|label, value }>
  - defaultValue: any
  - tooltip: string
- Returns: object API:
  - getChecked(): value | null
  - setChecked(value, checked): void
- Notes:
  - Creates unique group name per call

### Image(src, alt = '', width = 'auto', height = 'auto', tooltip = '')
Displays an image.

- Parameters:
  - src: string
  - alt: string
  - width: string
  - height: string
  - tooltip: string
- Returns: HTMLImageElement

### ListBox(items = [], defaultSelected = null, tooltip = '', onChange = null, itemType = 'text')
Scrollable list of items with selectable or interactive rows.

- Parameters:
  - items: any[] (strings or objects; depends on itemType)
  - defaultSelected: number | any (index or value for text type)
  - tooltip: string
  - onChange: function
    - text: (item, idx)
    - checkbox: (item, idx, checked)
    - custom: depends on implementer
  - itemType: 'text' | 'checkbox' | function (custom renderer)
- Behavior:
  - text: highlights selected row on click
  - checkbox: uses this.Checkbox() for consistent styling; label removed and a span label added; onChange fires with checked state
  - custom: invokes function(item, idx) to produce a child node
- Returns: HTMLUListElement, augmented with:
  - getSelected(): { item, index } | null
  - setSelected(idxOrItem): void
- Notes:
  - Hover/active styling matches theme
  - For text type, defaultSelected supports index or item equality

---

## Panel Lifecycle

### Show()
Appends the panel to document if not present and makes it visible.

- Behavior:
  - Cleans up empty wrappers
  - Warns if called mid-layout (indent/section/row)
  - Removes bottom margin from last control wrapper
- Returns: this

### Remove()
Removes the panel from the DOM.

- Returns: void

### Hide()
Hides the panel (display: none).

- Returns: this

---

## Modal/Prompt

### ShowModal(message, title = '', options = {})
Displays a modal dialog overlay.

- Parameters:
  - message: string | Node
  - title: string
  - options:
    - type: 'info' | 'warning' | 'error' | 'success' (default 'info')
    - buttons: string[] | { text, primary?, callback? }[]
    - closeOnBackdropClick: boolean (default true)
    - width: number (default 400)
- Returns: { close: Function, element: HTMLElement, backdrop: HTMLElement }
- Behavior:
  - Fades in backdrop and modal
  - Esc closes the modal
  - Button clicks invoke callbacks and close

### ShowPrompt(message, title, defaultValue = '', options = {})
Simple prompt placeholder.

- Parameters:
  - message: string
  - title: string (unused in fallback)
  - defaultValue: string
  - options: object (unused)
- Returns: string | null
- Notes:
  - Currently falls back to window.prompt with a console warning

---

## Theming

### SetTheme(themeName)
Switches active theme and updates all controls in-place.

- Parameters:
  - themeName: 'light' | 'dark'
- Returns: this
- Side Effects:
  - Updates CSS variables and restyles existing elements

### _updateCSSVariables()
Sets CSS custom properties on :root for current theme colors.

- Returns: void

### _applyThemeToElements()
Restyles DOM nodes under the container to match active theme.

- Returns: this

---

## Notes and Conventions

- All control builders append to the current build context determined by:
  - Row > Current Tab Pane (by build index) > Section (or its content) > Panel Content
- Most methods return the created input/control for further wiring.
- Wrapper divs use class .imgui-wrapper and controls use .imgui-control.
- IDs are unique per control for label-association and querying.
- Dragging is initiated by clicking on the container outside typical inputs.

---
