# Module 5: Priority Queues — "Game Server Login Queue"

## Overview

Build a game server login screen where players wait to join, but VIP and premium players get bumped up. Under the hood, a binary heap determines who enters next. Player data lives in the shared SQLite database; the heap runs in memory for fast O(log n) operations.

**Why this project?** If you've ever waited in a WoW or FFXIV login queue and seen premium players skip ahead, that's a priority queue. OS task schedulers, emergency rooms, and Dijkstra's shortest path algorithm (Module 8) all use heaps. This is one of the most practically useful data structures.

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| TypeScript | Data structure + backend |
| React (Vite) | UI with heap tree visualization |
| Node + Express | API server |
| better-sqlite3 | Shared SQLite database |
| Vitest | Unit testing |

---

## What You'll Learn

### Core Concepts
- **Priority Queue** — like a queue, but items come out in priority order, not insertion order
- **Binary Heap** — a complete binary tree where every parent is smaller (min) or larger (max) than its children
- **Array-based tree** — a binary tree stored as a flat array (no pointers needed!)
- **Heapify up / Heapify down** — the operations that maintain the heap property
- **O(log n) insert and extract** — vs O(n) for a naive sorted array

### Array-Based Binary Tree

The key insight: a binary tree can live in a flat array. For a node at index `i`:
- **Parent**: `Math.floor((i - 1) / 2)`
- **Left child**: `2 * i + 1`
- **Right child**: `2 * i + 2`

```
Array:  [10, 20, 30, 25, 35, 32, 40]

Tree:        10
           /    \
         20      30
        /  \    /  \
      25   35  32   40
```

### Operations

| Operation | Time Complexity | Description |
|-----------|----------------|-------------|
| `insert(item)` | O(log n) | Add item, bubble up to maintain heap |
| `extractMin/Max()` | O(log n) | Remove root, sink replacement down |
| `peek()` | O(1) | Look at highest priority item |
| `size()` | O(1) | Number of items |

### Database Additions

```sql
-- Player accounts
players (
  id INTEGER PRIMARY KEY,
  user_id INTEGER REFERENCES users(id),
  gamertag TEXT UNIQUE NOT NULL,
  tier TEXT DEFAULT 'free',  -- free, premium, vip
  level INTEGER DEFAULT 1,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
)

-- Game servers
servers (
  id INTEGER PRIMARY KEY,
  name TEXT NOT NULL,
  region TEXT NOT NULL,            -- us-east, eu-west, etc.
  max_capacity INTEGER DEFAULT 100,
  current_players INTEGER DEFAULT 0,
  status TEXT DEFAULT 'online'     -- online, full, maintenance
)

-- Queue history for analytics
login_queue_log (
  id INTEGER PRIMARY KEY,
  player_id INTEGER REFERENCES players(id),
  server_id INTEGER REFERENCES servers(id),
  priority_score INTEGER NOT NULL,
  position_entered INTEGER,
  entered_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  admitted_at DATETIME,
  wait_seconds INTEGER
)
```

---

## Project Architecture

```
05-priority-queues/
├── MODULE.md
├── package.json
├── vite.config.ts
├── tsconfig.json
├── index.html
├── src/
│   ├── data-structures/
│   │   ├── MinHeap.ts             — Min-heap implementation
│   │   ├── MaxHeap.ts             — Max-heap implementation
│   │   ├── PriorityQueue.ts       — Priority queue using a heap
│   │   ├── Heap.test.ts           — Tests
│   │   └── PriorityQueue.test.ts  — Tests
│   ├── api/
│   │   ├── server.ts              — Express server
│   │   └── routes/
│   │       ├── players.ts         — Player CRUD
│   │       ├── servers.ts         — Server status
│   │       └── queue.ts           — Join/admit/status endpoints
│   ├── components/
│   │   ├── ServerSelect.tsx       — Pick a server to join
│   │   ├── LoginQueue.tsx         — The queue waiting room
│   │   ├── HeapTree.tsx           — Visual binary tree of the heap
│   │   ├── ArrayView.tsx          — The underlying array representation
│   │   ├── PlayerCard.tsx         — Individual player in queue
│   │   └── QueueStats.tsx         — Wait times, queue depth, etc.
│   ├── App.tsx
│   └── main.tsx
└── public/
```

---

## Learning Curriculum

### Phase 1: Concept (Understanding the Theory)

**Step 1: Why not just sort the queue?**
You could maintain a sorted array, but:
- Inserting into a sorted array is O(n) (shift elements)
- A heap gives you O(log n) insert AND O(log n) extract-min/max
- You only need the BEST item, not all items sorted

**Step 2: The heap property**
- **Min-heap**: every parent ≤ its children. Root is the minimum.
- **Max-heap**: every parent ≥ its children. Root is the maximum.
- For our login queue: higher priority = lower number (VIP=1, Premium=2, Free=3). Use a min-heap.

**Step 3: Heapify Up (Bubble Up)**
After inserting at the end:
1. Compare with parent
2. If smaller than parent, swap
3. Repeat until heap property restored

```
Insert 5 into [10, 20, 30]:
Step 0: [10, 20, 30, 5]     — 5 added at end
Step 1: [10, 5, 30, 20]     — 5 < 20 (parent), swap
Step 2: [5, 10, 30, 20]     — 5 < 10 (parent), swap
Done: 5 is now the root
```

**Step 4: Heapify Down (Sink Down)**
After extracting the root (replace with last element):
1. Compare with both children
2. Swap with the smaller child (min-heap)
3. Repeat until heap property restored

**Step 5: Priority scoring**
For our game queue, priority is calculated from:
- Tier: VIP = 100, Premium = 50, Free = 10
- Wait time bonus: +1 per 30 seconds waiting (so free players eventually get in)
- Level bonus: +0.1 per player level

### Phase 2: Implement (Build It From Scratch)

**Step 1: MinHeap class**
1. Internal array storage
2. Helper methods: `parent(i)`, `leftChild(i)`, `rightChild(i)`, `swap(i, j)`
3. `insert(value)` — push to end, heapify up
4. `extractMin()` — save root, move last to root, heapify down
5. `peek()` — return root
6. `heapifyUp(index)` — bubble up from given index
7. `heapifyDown(index)` — sink down from given index

**Step 2: MaxHeap class**
1. Same structure, just flip the comparison
2. Or: implement as MinHeap with negated priorities

**Step 3: PriorityQueue class**
1. Wraps a MinHeap but stores `{ value, priority }` pairs
2. `enqueue(value, priority)` — insert with priority
3. `dequeue()` — extract min priority item
4. `changePriority(value, newPriority)` — update priority (used for wait time bonus)
5. Custom comparator support

**Step 4: GameLoginQueue**
1. `joinQueue(player)` — calculate priority score, enqueue
2. `admitNext(serverId)` — dequeue highest priority, UPDATE server current_players
3. `getPosition(playerId)` — where are you in the queue?
4. `updateWaitBonuses()` — periodically recalculate priorities based on wait time
5. `getQueueState()` — return current heap as array + tree for visualization

### Phase 3: Test (Verify Correctness)

- Heap property maintained after every insert
- Extract always returns min (or max)
- Insert + extract sequence produces sorted output (heap sort!)
- Priority queue respects priority ordering
- VIP players always ahead of free players (at same wait time)
- Wait time bonus eventually moves free players up
- Edge cases: empty heap extract, single element, duplicate priorities

### Phase 4: Build (React Visualization)

**Server Selection:**
1. List of game servers with name, region, capacity, current players
2. "Full" servers show a join queue button
3. Server status from the database

**Login Queue Waiting Room:**
1. "Add Player" button — pick a player from DB, join queue
2. "Auto-generate players" — rapidly add random players
3. Queue position display: "You are #7 in queue"
4. Estimated wait time

**Heap Tree Visualization:**
1. Binary tree drawn with nodes as circles
2. Each node shows player name + priority score
3. Color-coded by tier (gold=VIP, blue=Premium, gray=Free)
4. **Animate** heapify up/down — watch nodes swap step by step
5. Toggle between tree view and array view

**Array View:**
1. Show the underlying flat array
2. Highlight parent/child relationships with connecting lines
3. Show index numbers and the parent/child formulas

**Queue Stats Panel:**
1. Total in queue, by tier
2. Average wait time by tier
3. Admission rate (players/minute)
4. Historical chart of queue depth over time (from `login_queue_log`)

### Phase 5: Integrate (Database Connection)

**API endpoints:**
- `GET /api/players` — list players from DB
- `POST /api/players` — create player
- `GET /api/servers` — list servers with capacity info
- `POST /api/queue/join` — player joins queue (logs to DB)
- `POST /api/queue/admit` — admit next player (logs admission time)
- `GET /api/queue/status` — current queue state
- `GET /api/queue/history` — admission history with wait times

**SQL practice:**
```sql
-- Average wait time by tier
SELECT p.tier, AVG(l.wait_seconds) as avg_wait
FROM login_queue_log l
JOIN players p ON l.player_id = p.id
WHERE l.admitted_at IS NOT NULL
GROUP BY p.tier;

-- Players currently in queue, ordered by priority
SELECT p.gamertag, p.tier, l.priority_score, l.entered_at
FROM login_queue_log l
JOIN players p ON l.player_id = p.id
WHERE l.admitted_at IS NULL
ORDER BY l.priority_score DESC;

-- Server fill rate over time
SELECT s.name,
  strftime('%H', l.admitted_at) as hour,
  COUNT(*) as admissions
FROM login_queue_log l
JOIN servers s ON l.server_id = s.id
WHERE l.admitted_at IS NOT NULL
GROUP BY s.name, hour;
```

### Phase 6: Challenge (Stretch Goals)

1. **Heap sort** — use the heap to sort an array (extract all elements = sorted order)
2. **K-th largest element** — use a min-heap of size K to find the K-th largest in a stream
3. **Merge K sorted lists** — use a min-heap to efficiently merge (common interview problem)
4. **Decrease-key operation** — needed for Dijkstra's algorithm (preview of Module 8)
5. **D-ary heap** — generalize from binary (2 children) to D children — experiment with performance

---

## Connection to Other Modules

- **Module 1 (Linked Lists)**: Heaps use arrays instead of pointers — contrast the two approaches
- **Module 4 (Queues)**: Priority queue is a generalization of the regular queue
- **Module 8 (Graphs)**: Dijkstra's algorithm uses a priority queue to pick the next nearest node

---

## Key Takeaways

By the end of this module you should be able to:
- [ ] Explain the heap property and why it enables efficient priority operations
- [ ] Implement a binary heap from scratch using an array
- [ ] Perform heapify up and heapify down operations
- [ ] Build a priority queue on top of a heap
- [ ] Store and query game/queue analytics data in SQLite
- [ ] Visualize a tree structure stored as a flat array
