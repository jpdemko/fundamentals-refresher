# Module 4: Queues — "Support Ticket System"

## Overview

Build a helpdesk ticket system where customers submit support tickets (stored in SQLite) and agents process them in FIFO order. Tickets are loaded from the database into an in-memory queue for processing. Includes a circular buffer variant for a live event log viewer.

**Why this project?** Every background job processor, message queue, and helpdesk system uses FIFO queues. WordPress itself uses `wp_cron` — a queue of scheduled tasks. This project connects the data structure to real persistence with the shared database from Module 3.

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| TypeScript | Data structure + backend |
| React (Vite) | UI and visualization |
| Node + Express | API server |
| better-sqlite3 | Shared SQLite database |
| Vitest | Unit testing |

**First module using the shared database.** You'll add a `tickets` table and practice loading/saving data.

---

## What You'll Learn

### Core Concepts
- **FIFO (First In, First Out)** — the opposite of a stack's LIFO
- **Enqueue and Dequeue** — add to back, remove from front
- **Circular Queue (Ring Buffer)** — fixed-size queue that wraps around, overwrites oldest data
- **Queue vs Stack** — when FIFO vs LIFO is the right choice
- **Database-backed queues** — how real systems persist queue items

### Operations

| Operation | Time Complexity | Description |
|-----------|----------------|-------------|
| `enqueue(item)` | O(1) | Add to back |
| `dequeue()` | O(1) | Remove from front |
| `peek()` | O(1) | Look at front without removing |
| `isEmpty()` | O(1) | Check if empty |
| `size()` | O(1) | Number of items |

### Database Addition

```sql
-- Added to the shared database
tickets (
  id INTEGER PRIMARY KEY,
  user_id INTEGER REFERENCES users(id),
  subject TEXT NOT NULL,
  description TEXT NOT NULL,
  priority TEXT DEFAULT 'normal',  -- low, normal, high, urgent
  status TEXT DEFAULT 'waiting',   -- waiting, processing, resolved, closed
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  assigned_agent TEXT,
  resolved_at DATETIME
)
```

---

## Project Architecture

```
04-queues/
├── MODULE.md
├── package.json
├── vite.config.ts
├── tsconfig.json
├── index.html
├── src/
│   ├── data-structures/
│   │   ├── Queue.ts               — Queue class (array + linked list)
│   │   ├── Queue.test.ts          — Tests
│   │   ├── CircularQueue.ts       — Fixed-size ring buffer
│   │   └── CircularQueue.test.ts  — Tests
│   ├── api/
│   │   ├── server.ts              — Express server
│   │   └── routes/
│   │       └── tickets.ts         — CRUD for tickets
│   ├── components/
│   │   ├── TicketSubmitter.tsx     — Form to create new tickets
│   │   ├── QueueVisualizer.tsx     — Visual FIFO queue
│   │   ├── AgentPanel.tsx         — "Process next" button + current ticket
│   │   ├── TicketHistory.tsx       — Resolved tickets list
│   │   └── EventLog.tsx           — Circular buffer log viewer
│   ├── App.tsx
│   └── main.tsx
└── public/
```

---

## Learning Curriculum

### Phase 1: Concept (Understanding the Theory)

**Step 1: What is a queue?**
A line at a store. First person in line is the first served. New people join at the back.

```typescript
interface Queue<T> {
  enqueue(item: T): void   // join the back
  dequeue(): T | undefined  // leave from the front
  peek(): T | undefined     // who's at the front?
  isEmpty(): boolean
  size(): number
}
```

**Step 2: The naive array problem**
Using `Array.shift()` to dequeue is O(n) — every element shifts forward. Two better approaches:
- **Linked list**: head = front, tail = back. Dequeue removes head O(1).
- **Two-pointer array**: track `front` and `back` indices, don't shift elements.

**Step 3: Circular Queue (Ring Buffer)**
A fixed-size array where the back wraps around to the beginning:
- When `back` reaches the end, it wraps to index 0
- When the buffer is full, new items overwrite the oldest
- Used for: log viewers, audio buffers, network packet buffers

```
Capacity 4:  [A] [B] [C] [D]
                               ^ front=0, back=4 (full)
Enqueue E:   [E] [B] [C] [D]
              ^ back wraps to 0, overwrites A, front moves to 1
```

**Step 4: Queue vs Stack — when to use which**
- **Queue (FIFO)**: processing in order — tickets, jobs, messages, BFS
- **Stack (LIFO)**: undoing in reverse — undo/redo, parsing, DFS, call stack

### Phase 2: Implement (Build It From Scratch)

**Step 1: Queue (Linked List based)**
1. Use a linked list with head (front) and tail (back)
2. `enqueue(item)` — append to tail
3. `dequeue()` — remove from head
4. `peek()` — return head value
5. `toArray()` — for display

**Step 2: Queue (Two-pointer array based)**
1. Internal array with `front` and `back` indices
2. `enqueue(item)` — place at `back`, increment `back`
3. `dequeue()` — return at `front`, increment `front`
4. No shifting needed — just move the pointers

**Step 3: Circular Queue**
1. Fixed-size array with `front`, `back`, `count`
2. `enqueue(item)` — place at `back % capacity`, handle full (overwrite or reject)
3. `dequeue()` — return at `front % capacity`
4. `isFull()` — check if count equals capacity

**Step 4: Ticket Queue (integrating with DB)**
1. Load pending tickets from SQLite on startup (ordered by `created_at`)
2. `submitTicket(ticket)` — INSERT into DB, enqueue in memory
3. `processNext()` — dequeue from memory, UPDATE status in DB to 'processing'
4. `resolveTicket(id)` — UPDATE status to 'resolved', set `resolved_at`
5. Sync state between in-memory queue and database

### Phase 3: Test (Verify Correctness)

- FIFO order maintained
- Dequeue from empty returns undefined
- Circular queue wraps correctly
- Circular queue overwrites oldest when full
- Ticket queue syncs with database (insert → select verifies presence)
- Load from DB on startup restores queue state

### Phase 4: Build (React Visualization)

**Ticket Submission Panel:**
1. Form: subject, description, user (dropdown from DB users)
2. Submit button adds ticket to queue and database

**Queue Visualizer:**
1. Horizontal queue — items enter from right, leave from left
2. Each ticket shows subject, user, time waiting
3. "Waiting" → "Processing" → "Resolved" flow
4. Animation on enqueue/dequeue

**Agent Panel:**
1. "Process Next Ticket" button — dequeues front
2. Current ticket details displayed
3. "Resolve" button — marks complete
4. Stats: tickets processed, average wait time

**Event Log (Circular Buffer):**
1. Fixed-height log showing last N events
2. Events: "Ticket #5 submitted", "Ticket #2 resolved"
3. Oldest events disappear as new ones arrive
4. Visual indicator of the ring buffer wrapping

### Phase 5: Integrate (Database Connection)

**API endpoints:**
- `POST /api/tickets` — create ticket (inserts into DB + enqueues)
- `GET /api/tickets?status=waiting` — list tickets by status
- `PATCH /api/tickets/:id` — update status (processing, resolved)
- `GET /api/tickets/stats` — aggregate stats (count by status, avg resolution time)

**SQL practice opportunities:**
```sql
-- Tickets waiting, ordered by submission time
SELECT t.*, u.display_name FROM tickets t
JOIN users u ON t.user_id = u.id
WHERE t.status = 'waiting'
ORDER BY t.created_at ASC;

-- Average resolution time per agent
SELECT assigned_agent, AVG(julianday(resolved_at) - julianday(created_at)) * 24 as avg_hours
FROM tickets WHERE status = 'resolved'
GROUP BY assigned_agent;

-- Tickets submitted per day this week
SELECT DATE(created_at) as day, COUNT(*) as ticket_count
FROM tickets
GROUP BY DATE(created_at)
ORDER BY day DESC;
```

### Phase 6: Challenge (Stretch Goals)

1. **Priority queue preview** — add an "urgent" lane that gets processed first (preview of Module 5)
2. **Rate limiting** — max 3 tickets per user per hour (uses circular buffer to track submissions)
3. **Batch processing** — dequeue N tickets at once for bulk operations
4. **Dead letter queue** — tickets that fail processing go to a separate queue for review
5. **Simulate load** — "rush hour" button that submits 20 tickets rapidly

---

## Key Takeaways

By the end of this module you should be able to:
- [ ] Explain FIFO and implement a queue from scratch
- [ ] Implement a circular buffer / ring buffer
- [ ] Connect an in-memory data structure to a persistent database
- [ ] Build REST API endpoints for CRUD operations
- [ ] Write SQL queries involving JOINs and aggregates against real data
