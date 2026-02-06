# Task Entity Display Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Extend `viewer.html` to display "Task" entities alongside "Issue" entities, with Tasks having a distinct background color to differentiate them from Issues.

**Architecture:** Modify the existing card-based viewer to:
1. Detect whether the JSON contains `issues`, `tasks`, or both arrays
2. Render both entity types in a unified grid with visual differentiation
3. Apply a CSS background color change (green tint) for Task cards vs Issue cards (white)
4. Group tasks by User Stories, and User Stories by Milestones (Sprints)

**Tech Stack:** Plain HTML/CSS/JavaScript (no frameworks)

---

### Task 1: Add Task-specific CSS styling

**Files:**
- Modify: `/Users/gordon/Documents/repos/viewers/tiaga.html`

**Step 1: Add CSS for Task card background**

Add this CSS before `</style>`:
```css
/* Task card - different background from Issue */
.card.task {
    background: #e8f5e9;
}
.card.task:hover {
    background: #c8e6c9;
}

/* Keep Issue card with white background */
.card.issue {
    background: white;
}

/* Entity type label styling */
.entity-type {
    font-size: 10px;
    padding: 2px 6px;
    border-radius: 4px;
    text-transform: uppercase;
    font-weight: 600;
}

.card.issue .entity-type {
    background: #e3f2fd;
    color: #1565c0;
}

.card.task .entity-type {
    background: #e8f5e9;
    color: #2e7d32;
}
```

**Step 2: Run verification**

Open `viewer.html` in editor to confirm CSS was added correctly.

**Step 3: Commit**

```bash
git add viewer.html
git commit -m "style: add task card background styling"
```

---

### Task 2: Modify render function to support Task entity with User Story grouping

**Files:**
- Modify: `/Users/gordon/Documents/repos/viewers/tiaga.html:210-345`

**Step 1: Add renderCards function with User Story grouping**

The function signature is:
```javascript
function renderCards(issues, tasks, userStories)
```

Key implementation:
- Creates a unified `.card-grid` for Issue cards (rendered first as ungroupped)
- Groups tasks by User Stories using `storyMap` lookup
- Groups User Stories by Milestones (Sprints)
- Orphan tasks (not assigned to a User Story) appear in "Backlog" section
- Tasks under each User Story can be collapsed/expanded

**Step 2: Helper function for Task card HTML**

```javascript
function createTaskCardHtml(task) {
    return `
        <div class="card task" onclick="toggleCard(this)">
            <div class="card-header">
                <span class="ref">#${task.ref}</span>
                <span class="status" style="background:${getStatusColor(task.status)}20;color:${getStatusColor(task.status)}">
                    ${task.status || 'Unknown'}
                </span>
                <span class="entity-type">Task</span>
            </div>
            <div class="subject">${task.subject || ''}</div>
            <div class="card-footer" style="margin-top:auto;">
                <span class="priority" style="color:${getStatusColor(task.priority)}">
                    ${task.priority || ''}
                </span>
                ${task.attachments?.length > 0
                    ? `<span class="has-attachments">📎 ${task.attachments.length}</span>`
                    : ''}
            </div>
            <div class="details" id="details-${task.ref}">
                <div class="meta">
                    ${task.owner ? `<span class="meta-item"><strong>Owner:</strong> ${task.owner}</span>` : ''}
                    ${task.assigned_to ? `<span class="meta-item"><strong>Assigned:</strong> ${task.assigned_to}</span>` : ''}
                    ${task.user_story ? `<span class="meta-item"><strong>User Story:</strong> ${
                        typeof task.user_story === 'object'
                            ? (task.user_story.subject || task.user_story.ref || 'Unknown')
                            : task.user_story
                    }</span>` : ''}
                </div>
                <div class="description">${task.description || ''}</div>
                ${issue.attachments?.length > 0 ? `
                    <div class="attachments">
                        ${issue.attachments.map((att, i) => `
                            <span class="attachment" onclick="downloadAttachment(${issue.ref}, ${i}, event)">
                                📄 ${att.attached_file?.name || 'unnamed-file'}
                            </span>
                        `).join('')}
                    </div>
                ` : ''}
            </div>
        </div>
    `;
}
```

**Step 3: Add toggleStory function for collapsible User Story sections**

```javascript
function toggleStory(header) {
    const storyTasks = header.parentElement.querySelector('.story-tasks');
    if (storyTasks) {
        storyTasks.classList.toggle('collapsed');
    }
}
```

**Step 4: Add collapsed CSS styling**

```css
.story-tasks.collapsed {
    display: none;
}
```

**Step 5: Verify code compiles**

Open `viewer.html` to confirm JavaScript syntax is correct.

**Step 6: Commit**

```bash
git add viewer.html
git commit -m "feat: add renderCards function with User Story grouping"
```

---

### Task 3: Update handleFile to support combined data

**Files:**
- Modify: `/Users/gordon/Documents/repos/viewers/tiaga.html:392-419`

**Step 1: Replace handleFile function**

```javascript
function handleFile(file) {
    if (!file || !file.name.endsWith('.json')) {
        alert('Please drop a JSON file');
        return;
    }

    const reader = new FileReader();
    reader.onload = (e) => {
        try {
            const data = JSON.parse(e.target.result);
            window.issues = data.issues || [];
            window.tasks = data.tasks || [];
            window.userStories = data.user_stories || [];

            if (issues.length === 0 && tasks.length === 0 && userStories.length === 0) {
                alert('No issues, tasks or user stories found in this file');
                return;
            }

            renderCards(issues, tasks, userStories);
            dropzone.classList.add('hidden');
            results.classList.remove('hidden');
        } catch (err) {
            alert('Invalid JSON file: ' + err.message);
        }
    };
    reader.readAsText(file);
}
```

**Step 2: Commit**

```bash
git add viewer.html
git commit -m "feat: update handleFile to support issues, tasks and user_stories"
```

---

### Task 4: Update page title dynamically

**Files:**
- Modify: `/Users/gordon/Documents/repos/viewers/tiaga.html:6,153,445-457`

**Step 1: Add updateTitle function**

```javascript
function updateTitle() {
    const hasIssues = issues && issues.length > 0;
    const hasTasks = tasks && tasks.length > 0;

    if (hasIssues && hasTasks) {
        document.title = 'Taiga Issues & Tasks Viewer';
        document.getElementById('page-title').textContent = 'Taiga Issues & Tasks Viewer';
    } else if (hasTasks) {
        document.title = 'Taiga Task Viewer';
        document.getElementById('page-title').textContent = 'Taiga Task Viewer';
    } else {
        document.title = 'Taiga Issue Viewer';
        document.getElementById('page-title').textContent = 'Taiga Issue Viewer';
    }
}
```

Call `updateTitle()` at the end of `renderCards()` after setting `grid.innerHTML`.

**Step 2: Commit**

```bash
git add viewer.html
git commit -m "feat: add dynamic title based on entity types"
```

---

### Task 5: Test with sample data

**Step 1: Create test JSON with issues, tasks and user stories**

```bash
cat > /tmp/test-combined.json << 'EOF'
{
  "issues": [
    {"ref": 1, "subject": "Sample Issue", "status": "New", "priority": "High", "description": "Test issue description", "owner": "test@example.com"}
  ],
  "tasks": [
    {"ref": 101, "subject": "Sample Task", "status": "In progress", "priority": "Normal", "description": "Test task description", "assigned_to": "dev@example.com", "user_story": 201}
  ],
  "user_stories": [
    {"ref": 201, "subject": "User Story Example", "status": "In progress", "milestone": "Sprint 1"}
  ]
}
EOF
```

**Step 2: Test by opening viewer.html in browser**

Verify:
- Issue card has white background with blue "Issue" label
- Task card has green background with green "Task" label
- Tasks are grouped under their User Story
- User Stories are grouped by Milestone/Sprint
- Clicking User Story header collapses/expands tasks
- Both cards expand to show details
- No console errors
- Page title updates based on entity types present

**Step 3: Commit**

```bash
git add -A
git commit -m "test: verify combined issue/task/user_story rendering"
```

---

### Summary of Changes

| File | Change |
|------|--------|
| `viewer.html` | Added Task card CSS styling with green background |
| `viewer.html` | Added `renderCards()` function with User Story grouping |
| `viewer.html` | Added `createTaskCardHtml()` helper function |
| `viewer.html` | Added `toggleStory()` for collapsible story sections |
| `viewer.html` | Added `updateTitle()` for dynamic page titles |
| `viewer.html` | Updated `handleFile()` to support `user_stories` array |
| `viewer.html` | Added attachment support for Task entities |
| `viewer.html` | Updated `downloadAttachment()` to search both issues and tasks |