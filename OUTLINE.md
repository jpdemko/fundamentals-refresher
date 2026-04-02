# DSA Practice — Project Outline

Hands-on data structures & algorithms through practical, real-world mini projects.
Each module implements the data structure from scratch in TypeScript, then builds a React UI to visualize and interact with it.

**Stack:** TypeScript, React (Vite), Node/Express (Modules 3+), SQLite via better-sqlite3 (Modules 3+), Vitest for testing

---

## Architecture

### Shared SQLite Database (Modules 3+)

Module 3 introduces the shared database with a blog CMS schema. Each subsequent module adds its own tables to the same database, creating a realistic growing application.

```
shared/
└── db/
    ├── database.ts      — DB connection + initialization
    ├── schema.sql       — Full schema (grows with each module)
    ├── seed.sql         — Seed data for all modules
    └── dsa-practice.db  — The SQLite database file (gitignored)
```

**Database schema progression:**

| Module | Tables Added |
|--------|-------------|
| 3 - SQL | `users`, `posts`, `categories`, `post_categories`, `tags`, `post_tags`, `comments` |
| 4 - Queues | `tickets` |
| 5 - Priority Queues | `players`, `servers`, `login_queue_log` |
| 6 - Hash Tables | `api_cache`, `rate_limit_log` |
| 7 - Trees | `search_terms` (autocomplete dictionary) |
| 9 - B-Trees | Indexes added to existing tables |

### Per-Module Structure

Each module is a standalone Vite + React + TypeScript app:

```
XX-module-name/
├── MODULE.md                — Detailed curriculum and learning guide
├── package.json             — Module-specific dependencies
├── vite.config.ts
├── tsconfig.json
├── index.html
├── src/
│   ├── data-structures/     — Raw DS implementation (the learning core)
│   │   ├── *.ts             — Data structure classes
│   │   └── *.test.ts        — Unit tests (Vitest)
│   ├── api/                 — Express backend (Modules 3+)
│   │   └── server.ts
│   ├── components/          — React UI components
│   │   └── *.tsx
│   ├── App.tsx              — Main app
│   └── main.tsx             — Entry point
└── public/
```

### Learning Approach (Per Module)

Every module follows the same 5-phase structure:

1. **Concept** — Understand the data structure: what it is, why it exists, where it's used
2. **Implement** — Build the data structure from scratch in TypeScript (no libraries)
3. **Test** — Write unit tests to verify correctness and edge cases
4. **Build** — Create the React UI that visualizes and interacts with the data structure
5. **Integrate** — (Modules 4+) Connect to the shared SQLite database for realistic persistence
6. **Challenge** — Stretch exercises and variations to deepen understanding

---

## Module Overview

### Module 1: Linked Lists — "Spotify-Style Music Queue"
**Database:** None (pure in-memory)

Build a media player UI with playlist management powered by linked lists. "Play Next" inserts a track directly after the current one (O(1) — arrays can't do this). Toggle "Repeat All" to connect tail→head (circular linked list). Skip forward/back through tracks with prev/next pointers. Drag-and-drop reorder by reassigning pointers.

**Core concepts:** Nodes, pointers, singly vs doubly vs circular linked lists, O(1) insert-after-current, arrays vs linked lists tradeoffs

---

### Module 2: Stacks — "Maze Generator & Solver"
**Database:** None (pure in-memory)

Build a grid-based maze where walls are generated using recursive backtracking (a stack-based algorithm), then solve it with a visual robot that pushes coordinates as it explores and pops to backtrack at dead ends. Watch the stack grow and shrink in real-time as the maze is generated and solved.

**Core concepts:** LIFO, push/pop O(1), backtracking algorithms, DFS with explicit stack, call stack visualization

---

### Module 3: SQL Fundamentals — "Blog CMS Database"
**Database:** Creates the shared SQLite database

Design and build a blog CMS database from scratch — users, posts, categories, tags, comments. Write progressively complex queries through an interactive SQL console. This module establishes the shared database that all future modules build on.

**Core concepts:** Schema design, primary/foreign keys, relationships (1:1, 1:N, M:N), JOINs, aggregates, GROUP BY, subqueries, CTEs

---

### Module 4: Queues — "Support Ticket System"
**Database:** Adds `tickets` table

A helpdesk where customers submit tickets (stored in SQLite) and agents process them FIFO. Load tickets from the database into an in-memory queue. Includes a circular buffer variant for a live log viewer.

**Core concepts:** FIFO, enqueue/dequeue, circular queue/ring buffer, queue vs stack

---

### Module 5: Priority Queues — "Game Server Login Queue"
**Database:** Adds `players`, `servers`, `login_queue_log` tables

Players (stored in DB with tier info) wait to join game servers. VIP/premium players get higher priority. Binary heap manages the in-memory queue; the database tracks players, servers, and admission history.

**Core concepts:** Heap property, array-based binary tree, heapify up/down, O(log n) insert/extract

---

### Module 6: Hash Tables — "API Cache + Rate Limiter"
**Database:** Adds `api_cache`, `rate_limit_log` tables

Cache database query results in an LRU cache (hash map + doubly linked list). Rate-limit API requests per IP. The database stores cache entries and rate limit logs; the in-memory hash table provides O(1) lookups.

**Core concepts:** Hash functions, collision handling (chaining), O(1) average ops, load factor, LRU eviction

---

### Module 7: Trees — "Threaded Comments + Autocomplete"
**Database:** Uses existing `comments` table, adds `search_terms` table

Two projects in one: (A) Reddit-style threaded comments loaded from the `comments` table — the `parent_comment_id` column IS a tree structure in the database. Display with DFS, collapse/expand threads. (B) Autocomplete search powered by a Trie, with terms loaded from a `search_terms` dictionary table.

**Core concepts:** Tree terminology, BST property, Trie (prefix tree), DFS/BFS traversals, tree vs graph

---

### Module 8: Graphs — "Route Finder / Map Navigator"
**Database:** Optional (save/load maps)

Build an interactive map where you place locations (nodes) and draw roads (weighted edges) on a canvas. Run BFS for unweighted shortest path, Dijkstra's for weighted. Watch the algorithm explore the graph in real-time.

**Core concepts:** Vertices/edges, directed/undirected, weighted, adjacency list vs matrix, BFS, DFS, Dijkstra's, cycle detection

---

### Module 9: B-Trees & Indexes — "Database Performance Explorer"
**Database:** Adds indexes to existing tables, analyzes performance

Return to the shared database and explore WHY queries are fast or slow. Add indexes, compare query plans with EXPLAIN, and visualize the B-tree structure that powers database lookups. Measure real performance differences as data grows.

**Core concepts:** B-tree structure, B+ tree, index types, EXPLAIN plans, write vs read tradeoffs, composite indexes

---

## Suggested Learning Order

The modules are numbered in deliberate order — each builds on the previous:

1. **Linked Lists** — foundational node/pointer concept via a music queue
2. **Stacks** — LIFO and backtracking via maze generation/solving, plants the DFS seed for Module 8
3. **SQL Fundamentals** — establishes the shared database for all future modules
4. **Queues** — FIFO counterpart to stacks, first module using the database
5. **Priority Queues** — queues + tree structure (array-based binary tree)
6. **Hash Tables** — most-used DS in practice, uses linked lists for chaining, caches DB queries
7. **Trees** — builds on node/pointer concepts, uses DB for threaded comments
8. **Graphs** — generalizes trees, uses BFS (from queues) and DFS (from stacks)
9. **B-Trees & Indexes** — combines tree knowledge + hash table intuition for database performance
