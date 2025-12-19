# TaskBoard Project - Complete Interview Q&A Guide

## 📋 Table of Contents
1. [Project Overview & Architecture](#1-project-overview--architecture)
2. [Component Structure & Responsibilities](#2-component-structure--responsibilities)
3. [State Management](#3-state-management)
4. [Data Flow & Props](#4-data-flow--props)
5. [Specific Features Implementation](#5-specific-features-implementation)
6. [Drag & Drop Implementation](#6-drag--drop-implementation)
7. [Filtering & Search Logic](#7-filtering--search-logic)
8. [Sorting Algorithm](#8-sorting-algorithm)
9. [Form Handling & Validation](#9-form-handling--validation)
10. [Edge Cases & Error Handling](#10-edge-cases--error-handling)
11. [Testing Strategy](#11-testing-strategy)
12. [Performance & Optimization](#12-performance--optimization)
13. [Code Quality & Best Practices](#13-code-quality--best-practices)
14. [Potential Improvements](#14-potential-improvements)

---

## 1. Project Overview & Architecture

### Q1: What is this project about?
**A:** This is a Kanban-style task board management application built with React. It allows users to create, edit, delete, and organize tasks across three columns: "To Do", "In Progress", and "Completed". The application includes features like drag-and-drop, filtering, searching, sorting, and tag management.

### Q2: What is the tech stack used?
**A:** 
- **React 19.1.1** - UI library
- **Vite 7.1.7** - Build tool and dev server
- **Vitest 3.2.4** - Testing framework
- **React Testing Library** - Component testing utilities
- **ESLint** - Code linting
- No external state management library (uses React hooks only)

### Q3: What is the overall architecture pattern?
**A:** The application follows a **component-based architecture** with a **top-down data flow** pattern:
- `TaskBoard` is the **container component** (smart component) that manages all state and business logic
- `AddTaskModal`, `TaskColumn`, and `TaskCard` are **presentational components** (dumb components) that receive data and callbacks via props
- All state changes flow down through props, and user interactions flow up through callback functions
- No global state management - everything is managed locally in `TaskBoard` using React hooks

### Q4: Why did you choose this architecture?
**A:** 
- **Simplicity**: For a single-page application with moderate complexity, local state management is sufficient
- **No external dependencies**: Reduces bundle size and complexity
- **Clear data flow**: Makes it easy to understand where state lives and how it changes
- **Easy to test**: Components can be tested in isolation with mocked props
- **Scalability**: If needed later, we can easily refactor to use Context API or Redux

---

## 2. Component Structure & Responsibilities

### Q5: What are the main components and their responsibilities?

**A:** 

1. **TaskBoard.jsx** (Container Component)
   - Manages all application state (tasks, filters, modal state)
   - Implements all business logic (CRUD operations, filtering, sorting)
   - Handles data transformation and processing
   - Renders the board header with filters and search
   - Renders three TaskColumn components
   - Manages AddTaskModal visibility

2. **AddTaskModal.jsx** (Form Component)
   - Manages local form state (title, description, assignee, priority, tags)
   - Handles form validation
   - Implements tag selection/deselection logic
   - Supports both "add" and "edit" modes
   - Calls parent callbacks on form submission

3. **TaskColumn.jsx** (Layout Component)
   - Acts as a drop zone for drag-and-drop
   - Displays column header with task count
   - Renders TaskCard components for tasks in that status
   - Handles drag-over and drop events

4. **TaskCard.jsx** (Display Component)
   - Displays individual task information
   - Implements drag start functionality
   - Provides quick move buttons
   - Manages dropdown menu for edit/delete actions
   - Handles user interactions (edit, delete, move)

### Q6: Why is TaskBoard the container component?
**A:** TaskBoard is the natural container because:
- It needs to coordinate multiple child components
- It owns the source of truth (tasks array)
- It needs to implement complex logic (filtering, sorting) that affects multiple columns
- It's the only component that needs to know about all tasks simultaneously
- It manages the modal state which is shared across the application

### Q7: How do components communicate with each other?
**A:** 
- **Parent to Child**: Data flows down via props (tasks, callbacks, state values)
- **Child to Parent**: Events flow up via callback functions (onAddTask, onMoveTask, onDeleteTask, onEditTask)
- **No sibling communication**: Siblings communicate through their common parent (TaskBoard)

---

## 3. State Management

### Q8: What state is managed in TaskBoard and why?

**A:** TaskBoard manages:

1. **`tasks`** - Array of all task objects (source of truth)
2. **`isModalOpen`** - Boolean to control modal visibility
3. **`editingTask`** - Task object being edited, or null for add mode
4. **`searchTerm`** - String for search input
5. **`filterAssignee`** - String for assignee filter
6. **`filterPriority`** - String for priority filter
7. **`filterDate`** - String (YYYY-MM-DD) for date filter
8. **`filterTags`** - Array of selected tag strings
9. **`isTagsDropdownOpen`** - Boolean for UI state (tags filter dropdown)

**Why:** All this state is in TaskBoard because:
- Tasks array is the single source of truth
- Filters need to work together and affect all columns
- Modal state needs to be accessible from multiple places
- Centralized state makes it easier to implement combined filtering logic

### Q9: Why not use Context API or Redux?
**A:** 
- **Overkill for this project**: The component tree is shallow (only 2-3 levels)
- **No prop drilling issues**: We're not passing props through many layers
- **Simpler mental model**: Easier to understand where state lives
- **Better performance**: No unnecessary re-renders from context updates
- **Easier testing**: Can test components in isolation with simple props

However, if the app grew to have:
- Multiple pages/routes
- Deep component nesting
- Shared state across unrelated components
- Complex async operations

Then Context API or Redux would be beneficial.

### Q10: How is state updated when tasks change?
**A:** State updates use React's `setState` with functional updates:
- **Adding task**: `setTasks((prevTasks) => [newTask, ...prevTasks])` - Prepends new task
- **Updating task**: `setTasks((prevTasks) => prevTasks.map(...))` - Maps over array, updates matching task
- **Deleting task**: `setTasks((prevTasks) => prevTasks.filter(...))` - Filters out deleted task
- **Moving task**: Same as update, but changes status property

Using functional updates ensures we always work with the latest state, especially important for concurrent updates.

---

## 4. Data Flow & Props

### Q11: Walk me through the data flow when adding a new task.

**A:** 
1. User clicks "Add Task" button in TaskBoard
2. `setIsModalOpen(true)` is called, opening AddTaskModal
3. User fills form in AddTaskModal (local state: title, description, assignee, priority, tags)
4. User clicks "Add Task" submit button
5. `handleSubmit` in AddTaskModal calls `onAddTask(title, description, assignee, priority, tags)`
6. `addTask` function in TaskBoard receives the parameters
7. `addTask` creates new task object with:
   - `id: Date.now()` (unique ID)
   - `status: 'todo'` (always starts in To Do)
   - `createdAt: new Date().toISOString()` (current timestamp)
   - All other fields from form
8. `setTasks` updates the tasks array (prepends new task)
9. React re-renders TaskBoard with new tasks
10. Filtering and sorting logic runs on new tasks array
11. TaskColumn components receive filtered/sorted tasks via props
12. New task appears in "To Do" column
13. Modal closes (`setIsModalOpen(false)`)

### Q12: How does editing a task work?

**A:**
1. User clicks "Edit" from TaskCard dropdown menu
2. `handleEdit` calls `onEditTask(task)` with the task object
3. `editTask` in TaskBoard sets `editingTask` to the task and opens modal
4. `useEffect` in AddTaskModal detects `editingTask` change
5. Form fields are pre-filled with `editingTask` values
6. User modifies fields (local state in AddTaskModal)
7. User clicks "Update Task"
8. `handleSubmit` calls `onUpdateTask(editingTask.id, updates)`
9. `updateTask` in TaskBoard maps over tasks, updates matching task with new values
10. `setTasks` triggers re-render
11. Updated task appears in the board
12. Modal closes and `editingTask` is reset to null

### Q13: What props does each component receive?

**A:**

**TaskColumn:**
- `title` - String (column name)
- `status` - String ('todo' | 'inprogress' | 'completed')
- `tasks` - Array of filtered/sorted tasks for this column
- `onMoveTask` - Callback to move task to new status
- `onDeleteTask` - Callback to delete task
- `onEditTask` - Callback to open edit modal

**TaskCard:**
- `task` - Task object
- `onMoveTask` - Callback to move task
- `onDeleteTask` - Callback to delete task
- `onEditTask` - Callback to edit task

**AddTaskModal:**
- `onAddTask` - Callback for adding new task
- `onUpdateTask` - Callback for updating existing task
- `onClose` - Callback to close modal
- `editingTask` - Task object if editing, null if adding

---

## 5. Specific Features Implementation

### Q14: How does tag selection work in AddTaskModal?

**A:**
- Tags are stored in local state as `selectedTags` array (array of tag strings)
- Each tag button has `onClick={() => toggleTag(tag)}`
- `toggleTag` function:
  ```javascript
  const toggleTag = (tag) => {
    setSelectedTags((prevTags) =>
      prevTags.includes(tag)
        ? prevTags.filter((t) => t !== tag)  // Remove if exists
        : [...prevTags, tag]                   // Add if doesn't exist
    );
  };
  ```
- Visual feedback: Tag button gets `'selected'` CSS class when `selectedTags.includes(tag)`
- Multiple tags can be selected simultaneously
- In edit mode, `useEffect` pre-fills `selectedTags` from `editingTask.tags`

### Q15: How are task IDs generated and why?

**A:**
- New tasks use `id: Date.now()` 
- **Why Date.now()?**
  - Simple and doesn't require external dependencies
  - Generates unique IDs in most cases (millisecond precision)
  - Numeric IDs are easy to work with
- **Limitations:**
  - If two tasks are created in the same millisecond, IDs could collide
  - Better alternatives: UUID library, incremental counter, or server-generated IDs
- For this project scope, Date.now() is sufficient

### Q16: How does the task count update in column headers?

**A:**
- Each TaskColumn receives a `tasks` prop (filtered array for that column)
- Column header displays: `<span className="task-count">({tasks.length})</span>`
- Since `tasks` prop is derived from filtered/sorted tasks in TaskBoard, the count automatically updates when:
  - Tasks are added/deleted
  - Tasks move between columns
  - Filters change (affecting which tasks are visible)
- The count is reactive - no manual calculation needed

### Q17: How are quick move buttons determined in TaskCard?

**A:**
- `getAvailableActions()` function analyzes current task status
- Returns array of actions excluding the current status:
  ```javascript
  if (task.status !== 'todo') {
    actions.push({ label: 'Move to To Do', status: 'todo' });
  }
  // Similar for inprogress and completed
  ```
- This ensures users can't "move" a task to its current status
- Buttons are rendered dynamically based on available actions

### Q18: How does the delete confirmation work?

**A:**
- Uses browser's native `window.confirm()` dialog
- `handleDelete` function:
  ```javascript
  const handleDelete = () => {
    const shouldDelete = window.confirm('Are you sure...?');
    if (shouldDelete) {
      onDeleteTask(task.id);
    }
  };
  ```
- **Why window.confirm?**
  - Simple, no dependencies
  - Works immediately
- **Better alternative:** Custom modal component for better UX and styling control

---

## 6. Drag & Drop Implementation

### Q19: How is drag-and-drop implemented?

**A:** Uses HTML5 Drag and Drop API:

**In TaskCard (drag source):**
- Card has `draggable` attribute
- `onDragStart`: Sets task ID in dataTransfer
  ```javascript
  e.dataTransfer.setData('application/json', JSON.stringify({ taskId: task.id }));
  e.dataTransfer.effectAllowed = 'move';
  ```

**In TaskColumn (drop target):**
- `onDragOver`: Prevents default, allows drop
  ```javascript
  e.preventDefault();
  e.dataTransfer.dropEffect = 'move';
  ```
- `onDrop`: Extracts task ID, calls onMoveTask
  ```javascript
  const { taskId } = JSON.parse(e.dataTransfer.getData('application/json'));
  onMoveTask(taskId, status);
  ```

### Q20: Why use JSON for dataTransfer instead of just the ID?

**A:**
- More extensible - can add more data later (e.g., source column, metadata)
- Consistent format - same structure for all drag operations
- Type safety - object structure is clear
- Future-proof - if we need to pass complex data, JSON handles it

### Q21: What happens if drag-and-drop fails (e.g., invalid data)?

**A:**
- `handleDrop` has try-catch around JSON.parse
- If parsing fails, function returns early (silent failure)
- Task doesn't move, no error shown to user
- **Better approach:** Could show error toast or log to console for debugging

### Q22: Can you drag a task to its current column?

**A:**
- Technically yes - the drop handler doesn't check current status
- However, `moveTask` function updates the task even if status is the same
- This causes a re-render but no visual change
- **Optimization:** Could add check: `if (task.status === newStatus) return;`

---

## 7. Filtering & Search Logic

### Q23: How does the search functionality work?

**A:**
- Search input is bound to `searchTerm` state
- Filtering logic:
  ```javascript
  const normalizedSearch = searchTerm.trim().toLowerCase();
  const matchesSearch = !normalizedSearch || 
    task.title.toLowerCase().includes(normalizedSearch) ||
    task.description.toLowerCase().includes(normalizedSearch);
  ```
- **Case-insensitive**: Both search term and task text are lowercased
- **Searches both fields**: Matches if title OR description contains term
- **Empty search shows all**: If searchTerm is empty, all tasks pass

### Q24: How do multiple filters work together?

**A:** All filters use **AND logic** - task must match ALL active filters:
```javascript
return (
  matchesSearch && 
  matchesAssignee && 
  matchesPriority && 
  matchesDate && 
  matchesTags
);
```

**Example:**
- Search: "design" → matches tasks with "design" in title/description
- Assignee: "Sarah Johnson" → matches tasks assigned to Sarah
- Result: Only shows tasks that match BOTH conditions

### Q25: How does the assignee filter work?

**A:**
- `uniqueAssignees` is derived: `[...new Set(tasks.map(task => task.assignee.name))]`
- Dropdown shows "All Assignees" (empty value) + all unique assignee names
- Filter logic: `!filterAssignee || task.assignee?.name === filterAssignee`
- Empty filter shows all tasks
- Selected filter shows only tasks where assignee name exactly matches

### Q26: How does the priority filter work?

**A:**
- Dropdown has options: "All Priorities", "high", "medium", "low"
- Filter logic: `!filterPriority || task.priority === filterPriority`
- Exact match required (case-sensitive, but values are lowercase)
- Empty filter shows all priorities

### Q27: How does the date filter work?

**A:**
- Uses HTML5 date input (`<input type="date">`)
- Stores value as YYYY-MM-DD string
- Filter logic:
  ```javascript
  !filterDate || 
  new Date(task.createdAt).toISOString().slice(0, 10) === filterDate
  ```
- Compares task's creation date (converted to YYYY-MM-DD) with selected date
- Shows only tasks created on that exact date

### Q28: How does the tags filter work?

**A:**
- Multi-select checkboxes in dropdown
- `filterTags` is array of selected tag strings
- Filter logic:
  ```javascript
  filterTags.length === 0 || 
  (task.tags || []).some(tag => filterTags.includes(tag))
  ```
- **OR logic within tags**: Task matches if it has ANY of the selected tags
- **AND logic with other filters**: Must match tags AND all other active filters
- Empty selection (no tags checked) shows all tasks

### Q29: What happens when no tasks match the filters?

**A:**
- `filteredTasks` becomes empty array
- Each TaskColumn receives empty `tasks` array
- Columns render with count "(0)" and no task cards
- No error message shown (could add "No tasks found" message)

---

## 8. Sorting Algorithm

### Q30: How are tasks sorted within each column?

**A:** Two-level sorting:

1. **Primary sort: Priority** (High → Medium → Low)
   - Uses priority rank map: `{ high: 0, medium: 1, low: 2 }`
   - Lower number = higher priority = appears first
   
2. **Secondary sort: Date** (Newest first)
   - When priorities are equal, sort by `createdAt`
   - `new Date(b.createdAt) - new Date(a.createdAt)` (descending)

```javascript
const sortedTasks = [...filteredTasks].sort((a, b) => {
  const priorityComparison = priorityRank[a.priority] - priorityRank[b.priority];
  if (priorityComparison !== 0) return priorityComparison;
  return new Date(b.createdAt) - new Date(a.createdAt);
});
```

### Q31: Why create a copy with `[...filteredTasks]` before sorting?

**A:**
- `Array.sort()` mutates the original array
- React state should be immutable
- Creating a copy ensures we don't accidentally mutate state
- Follows React best practices

### Q32: What if a task has an invalid priority value?

**A:**
- `priorityRank[a.priority]` returns `undefined` for invalid values
- `undefined ?? Number.MAX_SAFE_INTEGER` would handle it (though not explicitly in code)
- Invalid priorities would sort to the end
- **Better approach:** Add validation or default to 'medium'

---

## 9. Form Handling & Validation

### Q33: How is form validation implemented?

**A:** Two levels of validation:

1. **HTML5 validation:**
   - `required` attribute on title, description, assignee inputs
   - Browser prevents submission if fields are empty

2. **JavaScript validation:**
   - Submit button is disabled if:
     ```javascript
     !title.trim() || !description.trim() || !selectedAssignee
     ```
   - Prevents submission even if HTML5 validation is bypassed

### Q34: How does the form handle edit mode vs add mode?

**A:**
- `editingTask` prop determines mode (null = add, object = edit)
- `useEffect` watches `editingTask`:
  ```javascript
  useEffect(() => {
    if (editingTask) {
      // Pre-fill all fields from editingTask
    } else {
      // Reset to empty/default values
    }
  }, [editingTask]);
  ```
- Form title changes: "Add New Task" vs "Edit Task"
- Submit button text: "Add Task" vs "Update Task"
- `handleSubmit` checks `editingTask` to call `onAddTask` or `onUpdateTask`

### Q35: How is the assignee parsed from the select dropdown?

**A:**
- Dropdown shows: `"Sarah Johnson (SJ)"`
- On change, finds matching user object:
  ```javascript
  const user = mockUsers.find(u => u.name === e.target.value);
  setSelectedAssignee(user || null);
  ```
- Stores full user object `{ name, initials }`, not just the string
- This ensures task has both name and initials for display

### Q36: What happens if form submission fails validation?

**A:**
- HTML5 `required` fields prevent submission
- Disabled submit button prevents click
- No error messages shown (could add inline validation messages)
- Form stays open, user can correct and retry

---

## 10. Edge Cases & Error Handling

### Q37: What edge cases are handled in the code?

**A:**

1. **Empty filters**: All filters check for empty values and show all tasks
2. **Missing task properties**: Uses optional chaining (`task.assignee?.name`)
3. **Empty tags array**: Checks `task.tags || []` to avoid errors
4. **Drag drop errors**: Try-catch around JSON.parse
5. **Date parsing**: Uses `new Date()` which handles various formats
6. **Empty search**: Trims and checks for empty string

### Q38: What edge cases are NOT handled?

**A:**

1. **Duplicate task IDs**: No check if Date.now() generates duplicate
2. **Invalid date formats**: Assumes createdAt is valid ISO string
3. **Very long task titles/descriptions**: No length limits
4. **Special characters in search**: Could break with regex special chars
5. **Concurrent edits**: No locking if same task edited simultaneously
6. **Network errors**: No error handling (assumes all-local)
7. **Browser compatibility**: No polyfills for older browsers

### Q39: How would you handle a task with missing required fields?

**A:**
- Current code: Would display but might break (e.g., missing assignee)
- **Better approach:**
  - Add default values: `assignee: task.assignee || { name: 'Unassigned', initials: 'U' }`
  - Validate on load: Filter out invalid tasks
  - Show warning/error in UI for invalid tasks

### Q40: What if a user tries to move a task that no longer exists?

**A:**
- `moveTask` uses `map()` - if task ID doesn't exist, array unchanged
- No error thrown, task simply doesn't move
- **Better approach:** Could log warning or show error message

---

## 11. Testing Strategy

### Q41: What tests are included and what do they cover?

**A:** Three test suites:

1. **TaskBoard.test.jsx:**
   - Search filters by title/description
   - Assignee filter shows only matching tasks
   - Priority filter shows only matching tasks

2. **AddTaskModal.test.jsx:**
   - Form submission with correct parameters (including tags)
   - Tag selection/deselection toggles CSS class

3. **TaskCard.test.jsx:**
   - Move button calls onMoveTask with correct parameters

### Q42: What testing approach is used?

**A:**
- **React Testing Library**: Focuses on user behavior, not implementation
- **User-centric**: Tests what user sees and does, not internal state
- **Integration-style**: Tests components with their props, not in isolation
- Uses `userEvent` for realistic user interactions

### Q43: What is NOT tested that should be?

**A:**
- Delete functionality (confirmation dialog)
- Drag and drop
- Combined filters (multiple filters together)
- Edit mode (form pre-filling)
- Date filter
- Tags filter
- Sorting logic
- Edge cases (empty states, invalid data)
- Error handling

### Q44: How would you test drag-and-drop?

**A:**
- Use `fireEvent` or `userEvent` to simulate drag events:
  ```javascript
  fireEvent.dragStart(card);
  fireEvent.dragOver(column);
  fireEvent.drop(column);
  ```
- Verify `onMoveTask` was called with correct parameters
- Test that task appears in new column after drop

---

## 12. Performance & Optimization

### Q45: Are there any performance concerns in the current implementation?

**A:** Potential issues:

1. **Re-renders on every filter change**: All columns re-render even if their tasks didn't change
2. **No memoization**: Filtering/sorting runs on every render
3. **Large arrays**: If thousands of tasks, filtering could be slow
4. **No virtualization**: All tasks render even if off-screen

### Q46: How would you optimize the filtering logic?

**A:**
- **useMemo** for filtered/sorted tasks:
  ```javascript
  const filteredTasks = useMemo(() => {
    return tasks.filter(...);
  }, [tasks, searchTerm, filterAssignee, ...]);
  ```
- **useMemo** for derived data (uniqueAssignees, column tasks)
- Prevents recalculation on every render

### Q47: How would you optimize re-renders?

**A:**
- **React.memo** for TaskCard and TaskColumn:
  ```javascript
  export default React.memo(TaskCard);
  ```
- Only re-render if props actually change
- **useCallback** for handler functions to prevent recreation:
  ```javascript
  const moveTask = useCallback((taskId, newStatus) => {
    // ...
  }, []);
  ```

### Q48: What about performance with large datasets?

**A:**
- Current: All tasks in memory, all rendered
- **Solutions:**
  - Virtual scrolling (react-window, react-virtualized)
  - Pagination
  - Lazy loading
  - Server-side filtering
  - Debounce search input (wait for user to stop typing)

---

## 13. Code Quality & Best Practices

### Q49: What React best practices are followed?

**A:**
- ✅ Functional components with hooks
- ✅ Immutable state updates (map, filter, spread)
- ✅ Controlled components (inputs bound to state)
- ✅ Separation of concerns (container vs presentational)
- ✅ Props validation would be good (PropTypes or TypeScript)
- ✅ Consistent naming conventions
- ✅ Single responsibility principle

### Q50: What code smells or issues exist?

**A:**
1. **Magic strings**: Status values ('todo', 'inprogress') repeated throughout
   - **Fix**: Constants object `const STATUS = { TODO: 'todo', ... }`
2. **Date.now() for IDs**: Could collide
   - **Fix**: UUID or incremental counter
3. **window.confirm**: Not customizable
   - **Fix**: Custom modal component
4. **No error boundaries**: App crashes on error
   - **Fix**: Wrap in ErrorBoundary
5. **Hardcoded tag list**: In multiple places
   - **Fix**: Single source of truth (constant or config)

### Q51: How is code organized and structured?

**A:**
- **File structure**: One component per folder with CSS
- **Component structure**: 
  - Imports at top
  - Component definition
  - State declarations
  - Handler functions
  - Derived values
  - Render/return
- **Naming**: Clear, descriptive names (handleDelete, moveTask)
- **Comments**: Minimal (could use more for complex logic)

### Q52: What accessibility considerations are there?

**A:**
- ✅ Semantic HTML (buttons, inputs, labels)
- ✅ Labels for form inputs
- ✅ Keyboard navigation (native HTML elements)
- ❌ No ARIA labels for drag-and-drop
- ❌ No keyboard shortcuts
- ❌ No focus management in modal
- ❌ No screen reader announcements

---

## 14. Potential Improvements

### Q53: What features would you add next?

**A:**
1. **Persistence**: localStorage or backend API
2. **Due dates**: Add date picker, show overdue tasks
3. **Task details view**: Expandable card or modal
4. **Bulk operations**: Select multiple tasks, move/delete together
5. **Task history**: Track status changes over time
6. **Comments**: Add comments to tasks
7. **Attachments**: Upload files to tasks
8. **Notifications**: Toast messages for actions
9. **Keyboard shortcuts**: Quick actions via keyboard
10. **Dark mode**: Theme toggle

### Q54: How would you add localStorage persistence?

**A:**
```javascript
// Load from localStorage on mount
useEffect(() => {
  const saved = localStorage.getItem('tasks');
  if (saved) setTasks(JSON.parse(saved));
}, []);

// Save to localStorage whenever tasks change
useEffect(() => {
  localStorage.setItem('tasks', JSON.stringify(tasks));
}, [tasks]);
```

### Q55: How would you refactor to use TypeScript?

**A:**
- Define interfaces:
  ```typescript
  interface Task {
    id: number;
    title: string;
    status: 'todo' | 'inprogress' | 'completed';
    // ...
  }
  ```
- Type all props and state
- Get compile-time error checking
- Better IDE autocomplete

### Q56: How would you add a backend API?

**A:**
1. Create API service layer:
   ```javascript
   // api/tasks.js
   export const fetchTasks = () => fetch('/api/tasks');
   export const createTask = (task) => fetch('/api/tasks', { method: 'POST', body: JSON.stringify(task) });
   ```
2. Replace local state with API calls
3. Add loading states
4. Add error handling
5. Use React Query or SWR for caching/refetching

### Q57: What would you change about the current architecture?

**A:**
1. **Extract custom hooks**: `useTasks`, `useFilters`, `useModal`
2. **Constants file**: Move all magic strings/arrays to config
3. **Utils file**: Extract filtering/sorting logic
4. **Context API**: If adding auth or theme
5. **Error boundary**: Wrap app for error handling
6. **Loading states**: Show spinners during operations
7. **Optimistic updates**: Update UI immediately, sync with server later

---

## Quick Reference: Key Functions

### TaskBoard.jsx
- `moveTask(taskId, newStatus)` - Updates task status
- `addTask(...)` - Creates new task, adds to array
- `deleteTask(taskId)` - Removes task from array
- `editTask(task)` - Opens modal in edit mode
- `updateTask(taskId, updates)` - Updates existing task
- `toggleFilterTag(tag)` - Toggles tag in filter array

### AddTaskModal.jsx
- `handleSubmit(e)` - Validates and submits form
- `toggleTag(tag)` - Adds/removes tag from selection

### TaskCard.jsx
- `handleDragStart(e)` - Initiates drag operation
- `handleQuickMove(newStatus)` - Moves task via button
- `handleDelete()` - Shows confirmation, deletes task
- `handleEdit()` - Opens edit modal
- `getAvailableActions()` - Returns possible move actions

### TaskColumn.jsx
- `handleDragOver(e)` - Allows drop on column
- `handleDrop(e)` - Processes dropped task
- `handleDragLeave(e)` - Cleans up drag state

---

## Common Interview Questions

### Q58: Why did you choose React hooks over class components?
**A:** Hooks provide:
- Simpler syntax (less boilerplate)
- Better code reuse (custom hooks)
- Easier testing
- Modern React standard (class components are legacy)
- Better performance with functional components

### Q59: Explain the difference between controlled and uncontrolled components.
**A:**
- **Controlled**: Input value is bound to state, onChange updates state
  - Example: `<input value={title} onChange={e => setTitle(e.target.value)} />`
- **Uncontrolled**: Input manages its own state via refs
  - Example: `<input ref={inputRef} />`
- This project uses controlled components for predictable state

### Q60: What is the virtual DOM and why does React use it?
**A:**
- Virtual DOM is a JavaScript representation of the real DOM
- React compares virtual DOM trees to find changes (diffing)
- Only updates actual DOM where changes occurred
- Benefits: Performance, declarative programming, cross-platform (React Native)

### Q61: How does React's reconciliation work?
**A:**
- React compares new virtual DOM with previous
- Uses "diffing algorithm" to find minimal changes
- Updates only changed nodes in real DOM
- Uses keys to track list items efficiently

### Q62: What are the differences between useState and useRef?
**A:**
- **useState**: 
  - Triggers re-render on change
  - Returns [value, setter]
  - Used for data that affects UI
- **useRef**:
  - Doesn't trigger re-render
  - Returns mutable object { current: value }
  - Used for DOM references, timers, previous values

### Q63: When would you use useEffect vs useMemo vs useCallback?
**A:**
- **useEffect**: Side effects (API calls, subscriptions, DOM manipulation)
- **useMemo**: Expensive calculations (filtering, sorting)
- **useCallback**: Memoize functions passed as props to prevent re-renders

---

## Final Tips for Interview

1. **Be honest**: If you don't know something, say so and explain how you'd find out
2. **Think aloud**: Walk through your thought process
3. **Ask clarifying questions**: Show you think about edge cases
4. **Discuss trade-offs**: Every decision has pros/cons
5. **Show learning mindset**: Mention what you'd improve or learn next
6. **Reference the code**: Point to specific lines/files when explaining
7. **Be confident**: You built this, you understand it!

Good luck with your interview! 🚀




