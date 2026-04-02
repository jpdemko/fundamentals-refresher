# Module 7: Trees — "Threaded Comments + Autocomplete"

## Overview

Two projects in one module, both powered by tree data structures:

**Part A — Reddit-style Threaded Comments:** Load comments from the shared database's `comments` table (which already has `parent_comment_id` — it's been a tree all along). Display them as collapsible threaded replies using DFS. This connects directly to Module 3's schema.

**Part B — Autocomplete Search:** Build a search bar with instant type-ahead suggestions powered by a Trie (prefix tree). Terms loaded from a database dictionary table.

**Why this project?** The DOM is a tree. File systems are trees. Comment threads are trees. Your WordPress category/tag hierarchies are trees. React's component tree is a tree. Trees are everywhere in web development, and you'll finally see them clearly.

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| TypeScript | Data structure + backend |
| React (Vite) | UI with tree visualizations |
| Node + Express | API server |
| better-sqlite3 | Shared SQLite database |
| Vitest | Unit testing |

---

## What You'll Learn

### Core Concepts
- **Tree terminology** — root, leaf, parent, child, sibling, depth, height
- **Binary Search Tree (BST)** — left < parent < right, enabling O(log n) search
- **Trie (Prefix Tree)** — each node is a character, paths spell words
- **Tree traversals** — DFS (pre-order, in-order, post-order) and BFS (level-order)
- **Recursion** — trees are inherently recursive structures
- **Self-referential data** — the `comments` table's `parent_comment_id` IS a tree in SQL

### Tree Traversal Orders

Given this tree:
```
        1
       / \
      2   3
     / \
    4   5
```

| Traversal | Order | Use Case |
|-----------|-------|----------|
| Pre-order (DFS) | 1, 2, 4, 5, 3 | Copy/serialize a tree |
| In-order (DFS) | 4, 2, 5, 1, 3 | BST: produces sorted order |
| Post-order (DFS) | 4, 5, 2, 3, 1 | Delete tree (children before parent) |
| Level-order (BFS) | 1, 2, 3, 4, 5 | "Load more replies" one level at a time |

### Database Additions

The `comments` table already exists from Module 3. We add:

```sql
-- Autocomplete dictionary
search_terms (
  id INTEGER PRIMARY KEY,
  term TEXT UNIQUE NOT NULL,
  frequency INTEGER DEFAULT 1,  -- how often this term is searched
  category TEXT                  -- optional grouping
)
```

---

## Project Architecture

```
07-trees/
├── MODULE.md
├── package.json
├── vite.config.ts
├── tsconfig.json
├── index.html
├── src/
│   ├── data-structures/
│   │   ├── BinarySearchTree.ts    — BST implementation
│   │   ├── BST.test.ts            — Tests
│   │   ├── Trie.ts                — Trie (prefix tree) implementation
│   │   ├── Trie.test.ts           — Tests
│   │   ├── CommentTree.ts         — Build tree from flat comment array
│   │   └── CommentTree.test.ts    — Tests
│   ├── api/
│   │   ├── server.ts              — Express server
│   │   └── routes/
│   │       ├── comments.ts        — Threaded comments API
│   │       └── search.ts          — Autocomplete API
│   ├── components/
│   │   ├── ThreadedComments.tsx   — Reddit-style comment display
│   │   ├── CommentNode.tsx        — Single comment with collapse/reply
│   │   ├── SearchBar.tsx          — Autocomplete search input
│   │   ├── TrieVisualizer.tsx     — Visual Trie showing character nodes
│   │   ├── BSTExplorer.tsx        — Interactive BST (insert/search/delete)
│   │   └── TraversalDemo.tsx      — Step-through tree traversal animation
│   ├── App.tsx
│   └── main.tsx
└── public/
```

---

## Learning Curriculum

### Phase 1: Concept (Understanding the Theory)

**Step 1: What is a tree?**
A tree is a node with zero or more children. Unlike linked lists (one next), trees branch.
- **Root**: the top node (no parent)
- **Leaf**: a node with no children
- **Depth**: distance from root (root = depth 0)
- **Height**: longest path from a node to a leaf

**Step 2: Binary Search Tree (BST)**
A binary tree (max 2 children) with the BST property:
- Everything in the left subtree < parent
- Everything in the right subtree > parent
- This means in-order traversal gives sorted output

```
Insert 8, 3, 10, 1, 6, 14:

        8
       / \
      3   10
     / \    \
    1   6   14

Search for 6: 8→left→3→right→6 ✓ (3 comparisons, not 6)
```

**Step 3: Trie (Prefix Tree)**
Each node represents a character. Paths from root to marked nodes spell complete words:

```
Insert: "cat", "car", "card", "care"

        (root)
          |
          c
          |
          a
         / \
        t   r
            |
            d / e    ← "car" ends here, "card" and "care" branch
```

Searching for "car" = follow c→a→r. All words starting with "car": "car", "card", "care".

**Step 4: Trees in your database**
The `comments` table has `parent_comment_id`. This IS a tree — each row is a node, each foreign key is a parent pointer. To display threaded comments:
1. Query all comments for a post
2. Build a tree in memory from the flat array
3. Traverse with DFS to render in thread order

**Step 5: DFS vs BFS on comment threads**
- **DFS (pre-order)**: render a full conversation thread top-to-bottom. "Show full thread."
- **BFS (level-order)**: render all top-level comments first, then all direct replies, etc. "Load more at this depth."

### Phase 2: Implement (Build It From Scratch)

**Step 1: TreeNode class**
```typescript
class TreeNode<T> {
  value: T
  children: TreeNode<T>[] = []
  parent: TreeNode<T> | null = null
}
```

**Step 2: Binary Search Tree**
1. `insert(value)` — compare, go left or right, insert at leaf
2. `search(value)` — compare, go left or right until found or null
3. `inOrder()` — left → node → right (sorted output)
4. `preOrder()` — node → left → right
5. `postOrder()` — left → right → node
6. `bfs()` — level-order using a Queue (from Module 4!)
7. `min()` / `max()` — leftmost / rightmost node
8. `delete(value)` — three cases: leaf, one child, two children (find successor)
9. `height()` — recursive height calculation

**Step 3: Trie**
1. TrieNode: `children: Map<string, TrieNode>`, `isEndOfWord: boolean`
2. `insert(word)` — create path of character nodes
3. `search(word)` — follow path, check `isEndOfWord`
4. `startsWith(prefix)` — follow path (doesn't need `isEndOfWord`)
5. `autocomplete(prefix)` — find prefix node, collect all words below it (DFS)
6. `delete(word)` — remove end marker, clean up unused nodes

**Step 4: CommentTree (flat array → tree)**
1. `buildTree(comments[])` — convert flat DB rows into nested tree
   - Create a map of id → node
   - For each comment, find parent and add as child
   - Return root-level nodes
2. `flattenDFS(node)` — DFS traversal for rendering
3. `flattenBFS(node)` — BFS traversal for level-based loading
4. `getThread(commentId)` — get a single thread (node + all descendants)
5. `getDepth(commentId)` — how deep is this comment nested?

### Phase 3: Test (Verify Correctness)

**BST tests:**
- In-order traversal produces sorted array
- Search finds existing values, returns null for missing
- Delete leaf, delete node with one child, delete node with two children
- Height calculation

**Trie tests:**
- Insert and search single word
- Search for non-existent word returns false
- startsWith returns true for valid prefixes
- Autocomplete returns all matching words
- Delete removes word without affecting other words sharing the prefix

**CommentTree tests:**
- Flat array correctly builds nested tree
- DFS traversal visits all nodes in thread order
- BFS traversal visits level by level
- Handle comments with no replies (leaves)
- Handle deeply nested threads

### Phase 4: Build (React Visualization)

**Part A — Threaded Comments:**
1. Select a post from the database
2. Load its comments and build the tree
3. Display comments with indentation based on depth (like Reddit)
4. Collapse/expand threads (click to hide all children)
5. "Reply" button on each comment — creates a new comment with `parent_comment_id`
6. Toggle between DFS view (full threads) and BFS view (level by level)
7. "Add comment" form at the top for top-level comments

**Part B — Autocomplete Search:**
1. Search input field
2. As you type, show dropdown of matching terms (from Trie)
3. Results sorted by frequency (from `search_terms` table)
4. Click a suggestion to "search" for it (increment frequency in DB)
5. Visual Trie showing character nodes lighting up as you type each letter

**BST Explorer (bonus panel):**
1. Input to insert numbers into the BST
2. Visual binary tree with nodes and edges
3. Search: highlight the path taken to find a value
4. Delete: watch the tree restructure
5. Traversal buttons: step through in-order, pre-order, post-order with animations

**Traversal Demo:**
1. Pick a traversal type
2. Step through one node at a time
3. Each visited node highlights in sequence
4. Show the output array building up

### Phase 5: Integrate (Database Connection)

**API endpoints:**
- `GET /api/posts/:id/comments` — all comments for a post (flat, from DB)
- `POST /api/posts/:id/comments` — add a comment (with optional parent_comment_id)
- `GET /api/search?q=prefix` — autocomplete suggestions
- `POST /api/search/terms` — add a search term to the dictionary
- `PATCH /api/search/terms/:id/hit` — increment frequency on selection

**SQL practice:**
```sql
-- All comments for a post, with author info and reply depth
WITH RECURSIVE comment_tree AS (
  -- Base: top-level comments
  SELECT c.id, c.body, c.parent_comment_id, u.display_name, 0 as depth
  FROM comments c
  JOIN users u ON c.user_id = u.id
  WHERE c.post_id = 1 AND c.parent_comment_id IS NULL

  UNION ALL

  -- Recursive: replies
  SELECT c.id, c.body, c.parent_comment_id, u.display_name, ct.depth + 1
  FROM comments c
  JOIN users u ON c.user_id = u.id
  JOIN comment_tree ct ON c.parent_comment_id = ct.id
)
SELECT * FROM comment_tree ORDER BY depth, id;

-- Most active commenters
SELECT u.display_name, COUNT(*) as comment_count
FROM comments c
JOIN users u ON c.user_id = u.id
GROUP BY u.id
ORDER BY comment_count DESC
LIMIT 10;

-- Top autocomplete terms
SELECT term, frequency FROM search_terms
WHERE term LIKE 'react%'
ORDER BY frequency DESC
LIMIT 5;
```

The recursive CTE above is worth studying — it's how you query tree structures in SQL. WordPress doesn't use CTEs (it queries multiple times), but modern SQL does.

### Phase 6: Challenge (Stretch Goals)

1. **AVL tree** — self-balancing BST with rotation operations
2. **Serialize/deserialize** — convert BST to JSON string and back (common interview question)
3. **Lowest Common Ancestor** — find the shared ancestor of two comment nodes
4. **Trie with deletion cleanup** — remove unused branches when a word is deleted
5. **Word frequency ranking** — combine Trie with a priority queue (Module 5) for top-K suggestions

---

## Connection to Other Modules

- **Module 1 (Linked Lists)**: Tree nodes are like linked list nodes with multiple next pointers
- **Module 3 (SQL)**: The `comments` table IS a tree — `parent_comment_id` is the parent pointer
- **Module 4 (Queues)**: BFS traversal uses a queue
- **Module 2 (Stacks)**: DFS traversal uses a stack (or recursion, which uses the call stack)
- **Module 8 (Graphs)**: Trees are a special case of graphs (connected, acyclic)

---

## Key Takeaways

By the end of this module you should be able to:
- [ ] Explain tree terminology and the BST property
- [ ] Implement a BST with insert, search, delete, and all traversals
- [ ] Implement a Trie with insert, search, and autocomplete
- [ ] Convert a flat database result into a nested tree structure
- [ ] Traverse trees with DFS and BFS and know when to use each
- [ ] Write recursive SQL CTEs to query tree-structured data
