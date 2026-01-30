# Task Entity Display Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Extend `viewer.html` to display "Task" entities alongside "Issue" entities, with Tasks having a distinct background color to differentiate them from Issues.

**Architecture:** Modify the existing card-based viewer to:
1. Detect whether the JSON contains `issues`, `tasks`, or both arrays
2. Render both entity types in a unified grid with visual differentiation
3. Apply a CSS background color change (e.g., light green tint) for Task cards vs Issue cards (light blue tint)

**Tech Stack:** Plain HTML/CSS/JavaScript (no frameworks)

---

### Task 1: Add Task-specific CSS styling

**Files:**
- Modify: `/Users/gordon/Documents/repos/viewers/viewer.html`

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
```

**Step 2: Run verification**

Open `viewer.html` in editor to confirm CSS was added correctly.

**Step 3: Commit**

```bash
git add viewer.html
git commit -m "style: add task card background styling"
```

---

### Task 2: Modify render function to support Task entity

**Files:**
- Modify: `/Users/gordon/Documents/repos/viewers/viewer.html:165-203`

**Step 1: Replace renderIssues function with renderCards function**

Replace `renderIssues()` function with:
```javascript
function renderCards(issues, tasks) {
    let html = '';

    // Render Issues
    if (issues && issues.length > 0) {
        html += issues.map(issue => `
            <div class="card issue" onclick="toggleCard(this)">
                <div class="card-header">
                    <span class="ref">#${issue.ref}</span>
                    <span class="status" style="background:${getStatusColor(issue.status)}20;color:${getStatusColor(issue.status)}">
                        ${issue.status || 'Unknown'}
                    </span>
                    <span class="entity-type">Issue</span>
                </div>
                <div class="subject">${issue.subject || ''}</div>
                <div class="card-footer">
                    <span class="priority" style="color:${getStatusColor(issue.priority)}">
                        ${issue.priority || ''}
                    </span>
                    ${issue.attachments?.length > 0
                        ? `<span class="has-attachments">📎 ${issue.attachments.length}</span>`
                        : ''}
                </div>
                <div class="details" id="details-${issue.ref}">
                    <div class="meta">
                        ${issue.owner ? `<span class="meta-item"><strong>Owner:</strong> ${issue.owner}</span>` : ''}
                        ${issue.type ? `<span class="meta-item"><strong>Type:</strong> ${issue.type}</span>` : ''}
                        ${issue.severity ? `<span class="meta-item"><strong>Severity:</strong> ${issue.severity}</span>` : ''}
                    </div>
                    <div class="description">${issue.description || ''}</div>
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
        `).join('');
    }

    // Render Tasks
    if (tasks && tasks.length > 0) {
        html += tasks.map(task => `
            <div class="card task" onclick="toggleCard(this)">
                <div class="card-header">
                    <span class="ref">#${task.ref}</span>
                    <span class="status" style="background:${getStatusColor(task.status)}20;color:${getStatusColor(task.status)}">
                        ${task.status || 'Unknown'}
                    </span>
                    <span class="entity-type">Task</span>
                </div>
                <div class="subject">${task.subject || ''}</div>
                <div class="card-footer">
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
                        ${task.user_story ? `<span class="meta-item"><strong>User Story:</strong> ${task.user_story}</span>` : ''}
                    </div>
                    <div class="description">${task.description || ''}</div>
                    ${task.attachments?.length > 0 ? `
                        <div class="attachments">
                            ${task.attachments.map((att, i) => `
                                <span class="attachment" onclick="downloadAttachment(${task.ref}, ${i}, event)">
                                    📄 ${att.attached_file?.name || 'unnamed-file'}
                                </span>
                            `).join('')}
                        </div>
                    ` : ''}
                </div>
            </div>
        `).join('');
    }

    grid.innerHTML = html;
}
```

**Step 2: Verify code compiles**

Open `viewer.html` to confirm JavaScript syntax is correct.

**Step 3: Commit**

```bash
git add viewer.html
git commit -m "feat: add renderCards function for both Issue and Task entities"
```

---

### Task 3: Add entity-type label CSS

**Files:**
- Modify: `/Users/gordon/Documents/repos/viewers/viewer.html`

**Step 1: Add entity-type label styling**

Add before `</style>`:
```css
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

**Step 2: Commit**

```bash
git add viewer.html
git commit -m "style: add entity-type label styling"
```

---

### Task 4: Update handleFile to support combined data

**Files:**
- Modify: `/Users/gordon/Documents/repos/viewers/viewer.html:205-231`

**Step 1: Replace handleFile function**

Replace `handleFile()` with:
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
            const issues = data.issues || [];
            const tasks = data.tasks || [];

            if (issues.length === 0 && tasks.length === 0) {
                alert('No issues or tasks found in this file');
                return;
            }

            renderCards(issues, tasks);
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
git commit -m "feat: update handleFile to support both issues and tasks"
```

---

### Task 5: Update page title dynamically

**Files:**
- Modify: `/Users/gordon/Documents/repos/viewers/viewer.html:6,91`

**Step 1: Make title dynamic**

Replace `<title>Taiga Issue Viewer</title>` with `<title>Taiga Viewer</title>` and `<h1>Taiga Issue Viewer</h1>` with:
```javascript
const entityCount = (issues.length > 0 ? 1 : 0) + (tasks.length > 0 ? 1 : 0);
document.title = entityCount > 1 ? 'Taiga Issues & Tasks Viewer' :
                 issues.length > 0 ? 'Taiga Issue Viewer' : 'Taiga Task Viewer';
```

**Step 2: Commit**

```bash
git add viewer.html
git commit -m "feat: add dynamic title based on entity types"
```

---

### Task 6: Test with sample data

**Step 1: Create test JSON with both issues and tasks**

```bash
cat > /tmp/test-combined.json << 'EOF'
{
  "issues": [
    {"ref": 1, "subject": "Sample Issue", "status": "New", "priority": "High", "description": "Test issue description", "owner": "test@example.com"}
  ],
  "tasks": [
    {"ref": 101, "subject": "Sample Task", "status": "In progress", "priority": "Normal", "description": "Test task description", "assigned_to": "dev@example.com"}
  ]
}
EOF
```

**Step 2: Test by opening viewer.html in browser**

Verify:
- Issue card has white background with blue "Issue" label
- Task card has green background with green "Task" label
- Both cards expand to show details
- No console errors

**Step 3: Commit**

```bash
git add -A
git commit -m "test: verify combined issue/task rendering"
```
