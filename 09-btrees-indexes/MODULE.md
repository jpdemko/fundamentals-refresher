# Module 9: B-Trees & Indexes — "Database Performance Explorer"

## Overview

Return to the shared SQLite database from Modules 3-7 and explore WHY some queries are fast and others are slow. Add indexes, compare query plans with EXPLAIN, and visualize the B-tree structure that powers database lookups under the hood. Measure real performance differences as you scale the dataset.

**Why this project?** This is the "aha" module that connects data structures to real performance. Every time a WordPress site slows down with 100K+ posts, the answer is usually "add an index." But most developers can't explain WHY an index helps. After this module, you can — because you'll have built the data structure yourself.

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| TypeScript | B-tree implementation + backend |
| React (Vite) | Performance dashboard UI |
| Node + Express | API for running queries |
| better-sqlite3 | Shared SQLite database |
| Vitest | Unit testing |

---

## What You'll Learn

### Core Concepts
- **Full table scan** — reading every row to find matches (O(n))
- **Indexed lookup** — jumping directly to matching rows (O(log n))
- **B-tree structure** — self-balancing, multi-way tree optimized for disk reads
- **B+ tree** — the variant databases actually use (data only in leaf nodes, leaves linked)
- **EXPLAIN / query plans** — reading how the database will execute your query
- **Index types** — single column, composite, unique, covering
- **Write overhead** — indexes speed up reads but slow down writes (every INSERT updates the index)

### B-Tree vs Binary Tree

| Property | BST (Module 7) | B-Tree |
|----------|----------------|--------|
| Children per node | 2 | Many (often 100+) |
| Height | O(log2 n) | O(logM n) where M is branching factor |
| Optimized for | In-memory | Disk I/O (fewer reads = fewer disk seeks) |
| Keys per node | 1 | Multiple (keeps node size ≈ disk page size) |
| Balance | May become unbalanced | Always balanced (splits and merges) |

A B-tree with branching factor 100 and 1 million keys has height ~3. That's 3 disk reads to find any record. A BST would need ~20.

### How an Index Works

Without index (full table scan):
```
SELECT * FROM posts WHERE title = 'Hello World';
→ Read every row in posts table, check each title
→ 100,000 rows = 100,000 comparisons
```

With index on `title`:
```
CREATE INDEX idx_posts_title ON posts(title);
SELECT * FROM posts WHERE title = 'Hello World';
→ Look up 'Hello World' in B-tree index (3 comparisons)
→ Follow pointer to the actual row
→ 100,000 rows = ~3 comparisons
```

---

## Project Architecture

```
09-btrees-indexes/
├── MODULE.md
├── package.json
├── vite.config.ts
├── tsconfig.json
├── index.html
├── src/
│   ├── data-structures/
│   │   ├── BTree.ts               — B-tree implementation
│   │   ├── BTree.test.ts          — Tests
│   │   └── BPlusTree.ts           — B+ tree variant (optional)
│   ├── api/
│   │   ├── server.ts              — Express server
│   │   └── routes/
│   │       ├── query.ts           — Run queries with timing
│   │       ├── explain.ts         — EXPLAIN query plans
│   │       ├── indexes.ts         — Create/drop indexes
│   │       └── benchmark.ts       — Performance benchmarks
│   ├── components/
│   │   ├── QueryRunner.tsx        — SQL input with timing display
│   │   ├── ExplainViewer.tsx      — Visual EXPLAIN output
│   │   ├── IndexManager.tsx       — Create/drop indexes, see all indexes
│   │   ├── BTreeVisualizer.tsx    — Visual B-tree with insert animation
│   │   ├── PerformanceDash.tsx    — Before/after comparison charts
│   │   ├── DataScaler.tsx         — Generate N rows to test scaling
│   │   └── WPComparison.tsx       — WordPress-specific index examples
│   ├── App.tsx
│   └── main.tsx
└── public/
```

---

## Learning Curriculum

### Phase 1: Concept (Understanding the Theory)

**Step 1: Why do databases need special trees?**
BSTs from Module 7 work great in memory. But databases store data on disk, and disk reads are SLOW (milliseconds vs nanoseconds for memory). B-trees minimize disk reads by:
- Storing many keys per node (one node ≈ one disk page, typically 4-16KB)
- Having many children per node (high branching factor → short tree)
- Staying perfectly balanced (guaranteed O(log n) height)

**Step 2: B-tree properties**
For a B-tree of order `m` (max children per node):
- Each node has at most `m` children and `m-1` keys
- Each node (except root) has at least `⌈m/2⌉` children
- All leaves are at the same depth
- Keys within a node are sorted
- Keys in children follow the BST property (left subtree < key < right subtree)

**Step 3: B-tree insert**
1. Find the correct leaf node
2. Insert the key in sorted order
3. If the node overflows (too many keys), **split** it:
   - Middle key moves up to parent
   - Node splits into two halves
4. Splitting may cascade up to the root (the only way the tree grows taller)

**Step 4: B+ tree — the database variant**
- All actual data lives in **leaf nodes** only
- Internal nodes only store keys for navigation
- Leaf nodes are **linked** (like a linked list!) for efficient range scans
- This is what SQLite, MySQL, PostgreSQL actually use

**Step 5: EXPLAIN**
Every database has an EXPLAIN command that shows the query plan:
```sql
EXPLAIN QUERY PLAN SELECT * FROM posts WHERE title = 'Hello';
```
Output tells you:
- Is it doing a full table scan? (`SCAN TABLE posts`)
- Is it using an index? (`SEARCH TABLE posts USING INDEX idx_posts_title`)
- How many rows does it estimate?

**Step 6: The write overhead tradeoff**
Every INSERT/UPDATE/DELETE must also update every index on that table. More indexes = faster reads, slower writes. This is why you don't index every column.

### Phase 2: Implement (Build It From Scratch)

**Step 1: B-Tree Node**
```typescript
class BTreeNode<T> {
  keys: T[]                    // sorted keys in this node
  children: BTreeNode<T>[]     // child pointers
  isLeaf: boolean
}
```

**Step 2: B-Tree class**
1. Constructor with `order` (max children)
2. `search(key)` — traverse from root, binary search within each node
3. `insert(key)` — find leaf, insert, split if overflow
4. `splitChild(parent, index)` — split a full child node
5. `traverse()` — in-order traversal (sorted output)
6. `getHeight()` — count levels
7. `toVisualization()` — return tree structure for React rendering

**Step 3: Insert with split (the tricky part)**
This is the most interesting algorithm in this module:
1. If root is full, create new root, split old root
2. Walk down the tree, proactively splitting full nodes on the way down
3. Insert into the (guaranteed non-full) leaf

**Step 4: B+ Tree (optional but illuminating)**
1. Data only in leaves
2. Leaf nodes linked in a doubly linked list
3. Range queries: find start key, follow leaf links to end key
4. Show why this is better for `SELECT * FROM posts WHERE created_at BETWEEN '2024-01-01' AND '2024-12-31'`

### Phase 3: Test (Verify Correctness)

- In-order traversal produces sorted output
- Search finds existing keys, returns null for missing
- Insert maintains B-tree properties (check after every insert)
- Node split correctness: right number of keys in each half
- Tree height grows logarithmically with insertions
- Insert 10,000 random values, verify all are searchable

### Phase 4: Build (React Visualization)

**B-Tree Visualizer:**
1. Input to insert keys one at a time
2. "Bulk insert" button — add N random keys
3. Visual multi-way tree — each node is a box with multiple keys
4. **Animate splits** — watch a node overflow, middle key rise, children separate
5. Order selector — change branching factor and see how tree shape changes
6. Color-coded: full nodes (yellow), recently split (red), search path (green)

**Query Runner:**
1. SQL input field (pre-populated with useful examples)
2. "Run" button — executes query, displays results AND execution time
3. Side-by-side comparison: run same query with and without index
4. Time difference highlighted

**EXPLAIN Viewer:**
1. Enter any SELECT query
2. See the EXPLAIN QUERY PLAN output
3. Visual interpretation: "This query scans 50,000 rows" vs "This query uses index, touches 3 rows"
4. Suggestions: "Consider adding an index on column X"

**Index Manager:**
1. List all current indexes in the database
2. Create new index: pick table, pick column(s)
3. Drop index
4. Show index size (bytes)

**Performance Dashboard:**
1. Pick a query
2. Slider: dataset size (1K, 10K, 100K, 500K rows)
3. Generate data, run query, chart execution time vs dataset size
4. Without index: linear growth. With index: logarithmic. See it.
5. Bar chart: read time improvement vs write time overhead

**WordPress Comparison Panel:**
1. Common slow WordPress queries and their index solutions
2. `wp_postmeta` problem: why `meta_key + meta_value` queries are slow
3. The `autoload` column issue in `wp_options`
4. How WooCommerce's custom tables solve the meta table problem

### Phase 5: Integrate (Add Indexes to the Shared Database)

**Indexes to create and test:**
```sql
-- Module 3 tables
CREATE INDEX idx_posts_author ON posts(author_id);
CREATE INDEX idx_posts_status ON posts(status);
CREATE INDEX idx_posts_created ON posts(created_at);
CREATE INDEX idx_comments_post ON comments(post_id);
CREATE INDEX idx_comments_parent ON comments(parent_comment_id);

-- Module 4 tables
CREATE INDEX idx_tickets_status ON tickets(status);
CREATE INDEX idx_tickets_user ON tickets(user_id);

-- Module 5 tables
CREATE INDEX idx_players_tier ON players(tier);
CREATE INDEX idx_queue_log_player ON login_queue_log(player_id);

-- Composite indexes
CREATE INDEX idx_posts_status_created ON posts(status, created_at);
CREATE INDEX idx_tickets_status_created ON tickets(status, created_at);
```

**SQL practice:**
```sql
-- Compare with and without index
EXPLAIN QUERY PLAN
SELECT * FROM posts WHERE status = 'published' ORDER BY created_at DESC;

-- Before composite index: SCAN TABLE + SORT
-- After idx_posts_status_created: SEARCH TABLE USING INDEX (no sort needed!)

-- Check all indexes on a table
SELECT * FROM sqlite_master WHERE type = 'index' AND tbl_name = 'posts';

-- Generate test data (100K posts)
-- (Script provided to INSERT random data for benchmarking)

-- Measure with timing
.timer on
SELECT * FROM posts WHERE author_id = 5 AND status = 'published';
```

### Phase 6: Challenge (Stretch Goals)

1. **B-tree deletion** — the merge/redistribution algorithm (complex but satisfying)
2. **Covering index** — an index that contains ALL columns needed by a query (no table lookup needed)
3. **Index-only scan** — demonstrate a query that never touches the actual table
4. **Full-text search index** — use SQLite FTS5 to build a text search feature
5. **Compare hash index vs B-tree index** — hash is O(1) for equality but can't do range queries

---

## Connection to Other Modules

This module is the grand unification:
- **Module 1 (Linked Lists)**: B+ tree leaf nodes are linked in a doubly linked list for range scans
- **Module 5 (Priority Queues)**: Both heaps and B-trees are tree structures, but optimized for different access patterns
- **Module 6 (Hash Tables)**: Hash tables give O(1) lookup vs B-tree's O(log n), but B-trees support ordered operations (range queries, MIN/MAX)
- **Module 7 (Trees)**: B-trees are a generalization of BSTs with higher branching factor for disk efficiency
- **Modules 3-7 (All DB modules)**: You're indexing the exact tables you built throughout the course

---

## Key Takeaways

By the end of this module you should be able to:
- [ ] Explain why B-trees exist (disk I/O optimization) vs BSTs (in-memory)
- [ ] Implement a B-tree with search and insert (including node splits)
- [ ] Read and interpret EXPLAIN query plans
- [ ] Know when to add an index (and when NOT to)
- [ ] Understand composite indexes and column order
- [ ] Recognize common WordPress performance issues and their index solutions
- [ ] Articulate the read/write tradeoff of indexing
