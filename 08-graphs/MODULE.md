# Module 8: Graphs — "Route Finder / Map Navigator"

## Overview

Build an interactive map where you place locations (nodes) and draw roads (weighted edges) on a canvas. Run BFS for unweighted shortest path, Dijkstra's for weighted shortest path, and DFS for exploration. Watch the algorithms explore the graph in real-time with step-by-step animation.

**Why this project?** Google Maps, package dependency resolution (npm/Composer), social network connections, web crawlers, and network routing all use graph algorithms. Graphs are the most general data structure — trees, linked lists, and even arrays are special cases. This module ties together everything you've learned.

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| TypeScript | Data structure + algorithms |
| React (Vite) | Interactive canvas UI |
| HTML Canvas or SVG | Drawing the map/graph |
| Vitest | Unit testing |
| better-sqlite3 | Optional: save/load maps |

**Backend is optional** for this module. The focus is on the algorithms and visualization. You can save/load map layouts to SQLite if you want persistence.

---

## What You'll Learn

### Core Concepts
- **Graph terminology** — vertices (nodes), edges, directed vs undirected, weighted vs unweighted
- **Representations** — adjacency list vs adjacency matrix (and when to use each)
- **BFS (Breadth-First Search)** — explore level by level, shortest unweighted path
- **DFS (Depth-First Search)** — explore as deep as possible, backtrack
- **Dijkstra's Algorithm** — shortest weighted path (uses a priority queue from Module 5!)
- **Cycle detection** — can you get back to where you started?
- **Topological sort** — ordering dependencies (npm install order)

### Graphs vs Trees

| Property | Tree | Graph |
|----------|------|-------|
| Cycles | Never | Can have cycles |
| Root | Has one | No root (or any can be "start") |
| Parent/child | Each node has one parent | Nodes can connect to anyone |
| Edges | N-1 edges for N nodes | Any number of edges |
| Path | Exactly one path between any two nodes | Multiple paths possible |

A tree is a connected acyclic graph. Everything you learned in Module 7 applies here, but graphs are more general.

### Representations

**Adjacency List** (most common, better for sparse graphs):
```typescript
// Map of node → list of neighbors
{
  "A": [{ node: "B", weight: 4 }, { node: "C", weight: 2 }],
  "B": [{ node: "A", weight: 4 }, { node: "D", weight: 3 }],
  "C": [{ node: "A", weight: 2 }, { node: "D", weight: 1 }],
  "D": [{ node: "B", weight: 3 }, { node: "C", weight: 1 }]
}
```

**Adjacency Matrix** (better for dense graphs or quick edge lookups):
```
    A   B   C   D
A [ 0,  4,  2,  0 ]
B [ 4,  0,  0,  3 ]
C [ 2,  0,  0,  1 ]
D [ 0,  3,  1,  0 ]
```

### Key Algorithms

| Algorithm | Uses | Time Complexity | Finds |
|-----------|------|----------------|-------|
| BFS | Queue | O(V + E) | Shortest unweighted path |
| DFS | Stack/recursion | O(V + E) | Reachability, cycles, topological order |
| Dijkstra's | Priority Queue | O((V + E) log V) | Shortest weighted path |

---

## Project Architecture

```
08-graphs/
├── MODULE.md
├── package.json
├── vite.config.ts
├── tsconfig.json
├── index.html
├── src/
│   ├── data-structures/
│   │   ├── Graph.ts               — Graph class (adjacency list)
│   │   ├── Graph.test.ts          — Tests
│   │   ├── algorithms/
│   │   │   ├── bfs.ts             — Breadth-first search
│   │   │   ├── dfs.ts             — Depth-first search
│   │   │   ├── dijkstra.ts        — Dijkstra's shortest path
│   │   │   ├── topologicalSort.ts — Dependency ordering
│   │   │   ├── cycleDetection.ts  — Detect cycles
│   │   │   └── algorithms.test.ts — Tests for all algorithms
│   │   └── PriorityQueue.ts       — Reuse from Module 5 (or simplified version)
│   ├── components/
│   │   ├── MapCanvas.tsx          — Interactive canvas for placing nodes/edges
│   │   ├── AlgorithmControls.tsx  — Pick algorithm, set start/end, run
│   │   ├── StepControls.tsx       — Play/pause/step through algorithm
│   │   ├── AlgorithmInfo.tsx      — Explanation of current algorithm step
│   │   ├── GraphStats.tsx         — Node count, edge count, connected?
│   │   └── AdjacencyView.tsx      — Show adjacency list/matrix representation
│   ├── App.tsx
│   └── main.tsx
└── public/
```

---

## Learning Curriculum

### Phase 1: Concept (Understanding the Theory)

**Step 1: What is a graph?**
A set of nodes (vertices) connected by edges. That's it. The most general "things connected to things" structure.

Examples you already know:
- Cities connected by roads (weighted, undirected)
- Twitter follows (directed — you can follow someone who doesn't follow you back)
- npm dependencies (directed — package A depends on B, not the other way)
- Web pages connected by hyperlinks (directed)

**Step 2: Directed vs Undirected**
- **Undirected**: roads — if A connects to B, B connects to A
- **Directed**: one-way streets, Twitter follows, dependencies

**Step 3: Weighted vs Unweighted**
- **Unweighted**: all edges equal (social connections)
- **Weighted**: edges have costs (road distances, network latency)

**Step 4: BFS — Breadth-First Search**
Explore all neighbors first, then neighbors' neighbors:
1. Start at source, add to queue
2. Dequeue a node, visit all unvisited neighbors, enqueue them
3. Repeat until queue is empty or target found

BFS finds the shortest path in an **unweighted** graph because it explores one "hop" at a time.

Uses your Queue from Module 4!

**Step 5: DFS — Depth-First Search**
Go as deep as possible, then backtrack:
1. Start at source, push to stack (or use recursion)
2. Pop a node, visit one unvisited neighbor, push it
3. If no unvisited neighbors, backtrack (pop)

DFS is used for: cycle detection, topological sort, maze solving, finding connected components.

Uses your Stack from Module 2! (Or the call stack via recursion.)

**Step 6: Dijkstra's Algorithm**
Shortest weighted path. Like BFS, but uses a priority queue instead of a regular queue:
1. Set distance to source = 0, all others = infinity
2. Add source to priority queue with priority 0
3. Extract-min from priority queue
4. For each neighbor: if current path is shorter than known distance, update and enqueue
5. Repeat until target extracted or queue empty

Uses your Priority Queue from Module 5!

**Step 7: Topological Sort**
Order tasks so that dependencies come first:
```
A depends on nothing
B depends on A
C depends on A
D depends on B and C

Topological order: A → B → C → D (or A → C → B → D)
```
This is how `npm install` determines installation order. Only works on **directed acyclic graphs** (DAGs).

### Phase 2: Implement (Build It From Scratch)

**Step 1: Graph class**
1. `addVertex(id, data?)` — add a node
2. `addEdge(from, to, weight?)` — add an edge (directed or undirected)
3. `removeVertex(id)` — remove node and all its edges
4. `removeEdge(from, to)` — remove specific edge
5. `getNeighbors(id)` — return adjacency list for a node
6. `hasEdge(from, to)` — check if edge exists
7. `getVertices()` — list all nodes
8. `toAdjacencyMatrix()` — convert to matrix representation

**Step 2: BFS**
1. Takes `graph`, `start`, optional `target`
2. Returns visited order AND shortest path (via parent tracking)
3. Track visited set to avoid cycles
4. Return step-by-step state for visualization

**Step 3: DFS**
1. Both iterative (stack) and recursive versions
2. Returns visited order
3. Detect cycles (if we visit a node already in the current path)
4. Return step-by-step state for visualization

**Step 4: Dijkstra's**
1. Takes `graph`, `start`, `target`
2. Uses PriorityQueue (from Module 5 or simplified)
3. Returns shortest distances AND actual path (via parent tracking)
4. Return step-by-step state: current node, frontier, known distances

**Step 5: Topological Sort**
1. DFS-based: finish-time ordering
2. Kahn's algorithm: BFS-based using in-degree counts
3. Detect if graph has a cycle (topological sort impossible)

### Phase 3: Test (Verify Correctness)

- BFS finds shortest unweighted path
- BFS visits all reachable nodes
- DFS visits all reachable nodes
- DFS detects cycles correctly
- Dijkstra's finds shortest weighted path
- Dijkstra's handles negative weights gracefully (or rejects them)
- Topological sort produces valid ordering
- Topological sort detects cycles
- Graph with disconnected components
- Single-node graph
- Graph with self-loops

### Phase 4: Build (React Visualization)

**Interactive Map Canvas:**
1. Click to place location nodes (draggable)
2. Click two nodes to draw an edge between them
3. Input weight for each edge (distance/cost)
4. Toggle directed/undirected mode
5. Right-click to delete nodes/edges
6. Preset maps: "City Grid", "Random", "Maze"

**Algorithm Controls:**
1. Select algorithm: BFS, DFS, Dijkstra's
2. Click a start node (green) and end node (red)
3. "Run" button — animates the algorithm
4. Speed slider — slow for learning, fast for fun

**Step-by-Step Animation:**
1. Play/Pause/Step buttons
2. Current node highlighted (yellow)
3. Explored nodes colored (blue)
4. Frontier/queue shown (light blue)
5. Final path highlighted (green)
6. For Dijkstra's: show current known distances on each node

**Algorithm Info Panel:**
1. Current step description in plain English
2. "Visiting node C. Checking neighbors: D (weight 3), E (weight 7)"
3. Queue/stack/priority queue state displayed
4. Distance table for Dijkstra's

**Adjacency View (toggle):**
1. Switch between visual map and adjacency list/matrix
2. Updates in real-time as you add/remove nodes and edges

### Phase 5: Integrate (Optional Database Connection)

**If you want persistence:**
- Save map layouts to SQLite (nodes with x,y coordinates, edges with weights)
- Load preset maps from DB
- Save "solutions" (shortest paths found)

**SQL practice:**
```sql
-- Simplified map storage
CREATE TABLE map_nodes (
  id INTEGER PRIMARY KEY,
  map_id INTEGER,
  label TEXT,
  x REAL, y REAL
);

CREATE TABLE map_edges (
  id INTEGER PRIMARY KEY,
  map_id INTEGER,
  from_node INTEGER REFERENCES map_nodes(id),
  to_node INTEGER REFERENCES map_nodes(id),
  weight REAL DEFAULT 1,
  directed BOOLEAN DEFAULT 0
);

-- Find all nodes connected to a specific node
SELECT mn2.label, me.weight
FROM map_edges me
JOIN map_nodes mn2 ON me.to_node = mn2.id
WHERE me.from_node = 1;
```

### Phase 6: Challenge (Stretch Goals)

1. **A* search** — Dijkstra's + heuristic (estimated distance to goal). Much faster for maps.
2. **Minimum Spanning Tree** — Prim's or Kruskal's algorithm. "Connect all cities with minimum road."
3. **Connected components** — find groups of nodes that are connected to each other
4. **Shortest path with obstacles** — grid-based pathfinding (like a game)
5. **npm dependency graph** — parse a real `package.json` and visualize the dependency tree

---

## Connection to Other Modules

This module ties together almost everything:
- **Module 2 (Stacks)**: DFS uses a stack
- **Module 4 (Queues)**: BFS uses a queue
- **Module 5 (Priority Queues)**: Dijkstra's uses a priority queue
- **Module 7 (Trees)**: Trees are special cases of graphs (connected, acyclic)
- **Module 6 (Hash Tables)**: Adjacency list is a hash map of node → neighbors

---

## Key Takeaways

By the end of this module you should be able to:
- [ ] Explain vertices, edges, directed/undirected, weighted/unweighted
- [ ] Implement a graph using an adjacency list
- [ ] Implement BFS and understand when it gives shortest paths
- [ ] Implement DFS and use it for cycle detection
- [ ] Implement Dijkstra's algorithm using a priority queue
- [ ] Understand topological sort and dependency ordering
- [ ] Recognize that previous data structures (stacks, queues, heaps) are tools FOR graph algorithms
