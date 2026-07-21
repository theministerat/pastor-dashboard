---
name: pastor-dashboard-refresh
description: Refreshes Aaron's Personal Dashboard - processes action queue, reads data sources (Calendar, Email, Tasks, Sermon, Pastoral Check-Ins, Prayer List), and generates updated HTML for deployment to GitHub Pages.
---

You are refreshing Aaron Taylor's Personal Dashboard for Ford Street United Methodist Church. This skill reads from his data sources, processes any pending actions, and generates updated HTML ready for deployment.

# 0. Important Context

## Data Locations
- **Google Drive mounted locally**: `G:\My Drive` (check if this path exists; if not, look for alternative paths like `G:\My Drive\...` or check OneDrive/Dropbox folders)
- **Work folder**: `G:\My Drive\[02] Work\` - contains all dashboard data files
- **Skills folder**: Look for `Claude Skills` or similar in the Drive

## Action Queue
The dashboard stores pending actions in localStorage in the browser. When refreshing, check for action data in two ways:
1. Ask the user if they have any pending actions to process
2. Look for a queue file in the Drive (e.g., `Dashboard Actions/queue.json`)

## The Six Dashboard Regions
1. **Calendar** - Today's and tomorrow's events from Google Calendar
2. **Email** - Threads needing personal reply from Gmail
3. **Worship** - This week's sermon tracker
4. **Tasks** - Task list from vault
5. **Pastoral Check-Ins** - 3 people to reach out to
6. **Prayer List** - Prayer requests

## Output
Generate complete HTML for the dashboard and save it to `C:\Users\aaron\.minimax\workspace\pastor-dashboard\dashboard.html`. This file can then be deployed to GitHub Pages.

---

# 1. Check for Pending Actions

Ask the user: "Do you have any pending actions in your dashboard? Check localStorage key 'pastor_dashboard_action_queue' or look for a queue file."

If yes, read the action data and process each action:

## Action Types and How to Apply Them

### add-task
- Read the Task List file from the vault
- Append: `- [ ] TEXT _(source: manual, added Month D, YYYY)_`
- If `alsoIgnoreThread` is present, also add to ignored threads

### mark-task-complete
- Read Task List
- Find the line with matching task text
- Change `- [ ]` to `- [x]`

### add-prayer
- Read Prayer List file
- Append: `- [ ] **NAME** — REQUEST _(added Month D, YYYY, PR-N)_`
- Find the highest PR number and increment

### mark-prayer-answered
- Read Prayer List
- Find entry with matching PR-N
- Change `- [ ]` to `- [x]`

### mark-messaged
- Read Message Log file
- Append row: `| Month D, YYYY | NAME | PERSON_ID |`

### log-note
- Read Pastoral Notes file
- Prepend new note entry with date

### ignore-thread
- Read Ignored Email Threads file
- Append the summary line

### sermon-checkbox
- Read Sermon Workflow Tracker
- Find the matching date entry
- Check/uncheck the specified line

### start-week
- Read Sermon Workflow Tracker
- Archive current week if exists
- Create new week entry with series, scripture, etc.

### run-research
- Run the sermon-research skill for the scripture
- Generate PDF and upload to Drive
- Update tracker with link

---

# 2. Get Current Dashboard HTML

Read the current dashboard file:
`C:\Users\aaron\.minimax\workspace\pastor-dashboard\dashboard.html`

This file has six regions marked by HTML comments:
- `<!-- CALENDAR_SECTION_START -->`
- `<!-- CALENDAR_SECTION_END -->`
- `<!-- EMAIL_SECTION_START -->`
- `<!-- EMAIL_SECTION_END -->`
- `<!-- WORSHIP_SECTION_START -->`
- `<!-- WORSHIP_SECTION_END -->`
- `<!-- TASKS_SECTION_START -->`
- `<!-- TASKS_SECTION_END -->`
- `<!-- TEXT3_SECTION_START -->`
- `<!-- TEXT3_SECTION_END -->`
- `<!-- PRAYER_SECTION_START -->`
- `<!-- PRAYER_SECTION_END -->`

Also find the sermon-data JSON:
`<script type="application/json" id="sermon-data">`

Keep everything outside these regions exactly as-is.

---

# 3. Calendar Region (CALENDAR_SECTION)

**Data source**: Google Calendar (via lark-tools or direct access if available)

For today and tomorrow:
- Get events for each day
- Format as HTML list items

**HTML format**:
```html
<ul class="items">
  <li class="item tier-low">
    <div class="event-time"><b>H:MM</b>&ndash;H:MM AM/PM</div>
    <div class="item-body">
      <div class="item-top"><span class="item-name">EVENT TITLE</span></div>
      <p class="item-desc">One short line: attendees/prep/stakes, if known</p>
    </div>
  </li>
</ul>
```

If no events: `<div class="empty-state">Nothing on your calendar today.</div>`

For tomorrow, add `<div class="subhead">Coming Up</div>` before the list.

---

# 4. Email Region (EMAIL_SECTION)

**Data source**: Gmail (via lark-tools or direct access if available)

Filter for:
- Inbox messages from last 3 days
- Exclude sent, newsletters, marketing
- Keep threads where a real person is waiting on Aaron

Classify by urgency:
- **tier-critical**: overdue, time-sensitive, or followed up multiple times
- **tier-pastoral**: Gloo (pastoral platform) or congregant matters
- **tier-low**: real but not urgent

Show max 6 threads.

**HTML format**:
```html
<ul class="items">
  <li class="item tier-critical">
    <div class="tier-dot" aria-hidden="true"></div>
    <div class="item-body">
      <div class="item-top">
        <span class="item-name">SENDER NAME</span>
        <span class="item-org">ORG OR CONTEXT</span>
        <time class="item-time">Day H:MM AM/PM</time>
      </div>
      <p class="item-desc">One paraphrased sentence</p>
      <span class="item-tag">SHORT TAG</span>
      <div class="item-actions">
        <!-- Add to Prayer, Add as Task, Ignore buttons -->
      </div>
    </div>
  </li>
</ul>
```

Action buttons use `data-action="queue-action"` with JSON payloads.

---

# 5. Worship/Sermon Region (WORSHIP_SECTION)

**Data source**: `G:\My Drive\[02] Work\Sermon Workflow Tracker.md`

This region has:
1. Navigation row (prev/next week)
2. Sermon info (series, title, scripture, main idea)
3. Phase tracker (Study → Plan → Write → Prepare → Preach → Content)
4. Next action button
5. Deliverables row (links to generated content)

**Determine active date**: Look for `# This Week` entry date, or use next Sunday.

**Build weeks JSON**: Create a JSON object with week dates as keys, containing the HTML for each week.

**Phase tracker states**:
- `done`: all items in phase checked
- `current`: first phase with unchecked item
- `upcoming`: phases after current

**Next action buttons**:
- Research: queue-action to run research skill
- Brainstorm, Editor, etc.: copy-prompt to clipboard
- Mark checkbox: queue-action

**HTML format**:
```html
<div class="sermon-nav">
  <button class="sermon-nav-arrow" data-action="sermon-nav" data-dir="-1">←</button>
  <span class="sermon-nav-date">Sunday, July 27, 2026 <span style="font-size:10.5px;font-weight:700;letter-spacing:0.04em;color:var(--accent);margin-left:8px;">THIS WEEK</span></span>
  <button class="sermon-nav-arrow" data-action="sermon-nav" data-dir="1">→</button>
</div>
<div class="sermon-info">
  <div class="info-field"><span class="info-label">Series</span><span class="info-value">SERIES NAME</span></div>
  <div class="info-field"><span class="info-label">Title</span><span class="info-value">SERMON TITLE</span></div>
  <div class="info-field"><span class="info-label">Scripture</span><span class="info-value">PASSAGE</span></div>
  <div class="info-field"><span class="info-label">Main Idea</span><span class="info-value">MAIN IDEA</span></div>
</div>
<div class="tracker">
  <div class="tracker-line"></div>
  <div class="tracker-node done"><div class="tracker-dot"></div><span class="tracker-label">Study</span></div>
  <div class="tracker-node current"><div class="tracker-dot"></div><span class="tracker-label">Plan</span></div>
  <div class="tracker-node upcoming"><div class="tracker-dot"></div><span class="tracker-label">Write</span></div>
  <!-- etc -->
</div>
<div class="cta-row">
  <button class="cta-button" data-action="queue-action" data-payload='{...}'>NEXT ACTION</button>
  <span class="cta-toast">Action queued</span>
</div>
<!-- Deliverables row if content exists -->
```

**Update sermon-data JSON** with the new weeks object:
```html
<script type="application/json" id="sermon-data">
{"activeDate":"2026-07-27","weeks":{"2026-07-27":"...HTML...","2026-07-20":"...archived HTML..."},"notStartedActiveHtml":"...","pastEmptyHtml":"...","futureEmptyHtml":"..."}
</script>
```

---

# 6. Tasks Region (TASKS_SECTION)

**Data source**: `G:\My Drive\[02] Work\Task List.md`

Format: `- [ ] TASK TEXT _(source: SOURCE, added DATE)_`

- Show max 8 open tasks, newest first
- Each task has a checkbox button

**HTML format**:
```html
<div class="task-list">
  <div class="task-row">
    <span class="task-text">Task text<span class="task-source">From today's notes</span></span>
    <button class="icon-button" data-action="queue-action" data-payload='{"type":"mark-task-complete","text":"TASK TEXT"}'>
      <!-- check icon -->
    </button>
  </div>
</div>
```

If no tasks: `<div class="empty-state">Nothing on your task list right now.</div>`

---

# 7. Pastoral Check-Ins Region (TEXT3_SECTION)

**Data source**: `G:\My Drive\[02] Work\Pastoral Care\Congregation Contacts.csv` and `G:\My Drive\[02] Work\Pastoral Care\Message Log.md`

Find 3 people to contact:
- Never messaged, OR
- Last messaged 60+ days ago

Sort: never-messaged first (alphabetical by last name), then by oldest message date.

**Privacy**: Show full names (first + last), but phone numbers go in data attributes only, never visible text.

**HTML format**:
```html
<div class="checkin-person">
  <div class="checkin-row">
    <div class="checkin-info">
      <span class="item-name" data-action="text-checkin" data-phone="+1XXXXXXXXXX" data-first-name="FIRST" style="cursor:pointer;text-decoration:underline;text-underline-offset:2px;">FIRST LAST</span>
      <span class="item-org">Never messaged</span>
    </div>
    <div class="checkin-actions">
      <button class="icon-button" data-action="queue-action" data-payload='{"type":"mark-messaged","personId":"ID","name":"FIRST LAST"}'>...</button>
      <button class="icon-button" data-action="toggle-note" data-target="note-row-1">...</button>
    </div>
  </div>
  <div class="note-row" id="note-row-1">
    <textarea class="note-input" id="note-1" placeholder="If they respond, jot a quick note..."></textarea>
    <button class="cta-button-secondary" data-action="copy-note" data-target="note-1" data-person="ID">Log Response</button>
  </div>
</div>
```

If everyone contacted in last 60 days: `<div class="empty-state">Everyone's been messaged in the last 60 days.</div>`

---

# 8. Prayer List Region (PRAYER_SECTION)

**Data source**: `G:\My Drive\[02] Work\Pastoral Care\Prayer List.md`

Format: `- [ ] **Name** — Request _(added Date, PR-N)_`

- Show max 6 active requests, newest first
- Each has "Mark Answered" button

**HTML format**:
```html
<ul class="items">
  <li class="item tier-low">
    <div class="item-body">
      <div class="item-top">
        <span class="item-name">FIRST LAST</span>
        <span class="item-org">Added DATE</span>
      </div>
      <p class="item-desc">REQUEST TEXT</p>
      <div class="item-actions">
        <button class="cta-button-secondary" data-action="queue-action" data-payload='{"type":"mark-prayer-answered","prId":"PR-N"}'>Mark Answered</button>
      </div>
    </div>
  </li>
</ul>
```

If no requests: `<div class="empty-state">No active prayer requests.</div>`

---

# 9. Generate and Save

Assemble the complete HTML:
1. Keep everything before CALENDAR_SECTION_START
2. Insert Calendar HTML between CALENDAR_SECTION comments
3. Keep everything between CALENDAR_SECTION_END and EMAIL_SECTION_START
4. Insert Email HTML between EMAIL_SECTION comments
5. Continue for all six regions
6. Keep everything after PRAYER_SECTION_END

Save to: `C:\Users\aaron\.minimax\workspace\pastor-dashboard\dashboard.html`

---

# 10. Summary

Provide a brief summary:
- How many actions were processed
- What data was refreshed in each region
- Any issues or notes

Tell the user the dashboard has been updated and is ready for deployment to GitHub Pages.