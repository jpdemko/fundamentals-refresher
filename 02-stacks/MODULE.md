# Module 2: Stacks — "Maze Generator & Solver"

## Overview

Build a grid-based maze where walls are generated using recursive backtracking (a stack-based algorithm), then watch a robot solve it by pushing coordinates onto a stack as it explores and popping to backtrack at dead ends. The stack state is visible in real-time alongside the maze.

**Why this project?** Stacks power every backtracking algorithm. Maze generation with recursive backtracking is one of the most visual, satisfying demonstrations of a stack in action — you can literally watch it carve paths by pushing, and reverse course by popping. It also plants the seed for DFS, which becomes critical in Module 8 (Graphs).

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| TypeScript | Data structure implementation + maze algorithms |
| React (Vite) | Grid UI and animation |
| HTML Canvas or CSS Grid | Maze rendering |
| Vitest | Unit testing |

**No backend or database** — pure in-memory, like Module 1.

---

## What You'll Learn

### Core Concepts
- **LIFO (Last In, First Out)** — the newest item is always the first removed
- **Push and Pop** — both O(1) operations
- **Backtracking** — explore a path, hit a dead end, pop back to the last decision point
- **Recursive backtracking = DFS with a stack** — the algorithm that generates and solves mazes
- **The JavaScript call stack** — recursion IS a stack; the maze algorithm shows this explicitly
- **Explicit stack vs recursion** — same algorithm, two ways to write it

### Operations

| Operation | Time Complexity | Description |
|-----------|----------------|-------------|
| `push(item)` | O(1) | Add to top |
| `pop()` | O(1) | Remove and return top |
| `peek()` | O(1) | Look at top without removing |
| `isEmpty()` | O(1) | Check if empty |
| `size()` | O(1) | Number of items |

### How Maze Generation Works (Recursive Backtracking)

This is the core algorithm — it's DFS on a grid using a stack:

```
1. Start at a cell, mark it visited, push it onto the stack
2. While the stack is not empty:
   a. Look at the current cell (top of stack)
   b. If it has any UNVISITED neighbors:
      - Pick a random unvisited neighbor
      - Remove the wall between current cell and neighbor
      - Push the neighbor onto the stack (it becomes current)
   c. If ALL neighbors are visited (dead end):
      - Pop the stack (backtrack to previous cell)
3. When the stack is empty, every cell has been visited → maze is complete
```

The beauty: the stack remembers every decision point, so backtracking always takes you to the last place where you had unexplored options.

### How Maze Solving Works (DFS)

Nearly identical algorithm, but instead of carving walls, you're finding a path:

```
1. Push start cell onto the stack
2. While stack is not empty:
   a. Pop a cell, mark it visited
   b. If it's the exit → reconstruct path, done!
   c. Push all unvisited passable neighbors onto the stack
3. If stack empties with no solution → maze is unsolvable
```

---

## Project Architecture

```
02-stacks/
├── MODULE.md
├── package.json
├── vite.config.ts
├── tsconfig.json
├── index.html
├── src/
│   ├── data-structures/
│   │   ├── Stack.ts               — Stack class (array-based)
│   │   ├── StackLinked.ts         — Stack class (linked-list-based)
│   │   ├── Stack.test.ts          — Tests
│   │   └── maze/
│   │       ├── MazeGenerator.ts   — Recursive backtracking maze generation
│   │       ├── MazeSolver.ts      — DFS maze solver
│   │       ├── Grid.ts            — Grid/Cell representation
│   │       └── maze.test.ts       — Tests
│   ├── components/
│   │   ├── MazeGrid.tsx           — Visual grid with walls and paths
│   │   ├── StackVisualizer.tsx    — Real-time stack state display
│   │   ├── Controls.tsx           — Generate, solve, step, speed controls
│   │   ├── AlgorithmInfo.tsx      — Current step explanation
│   │   └── MazeStats.tsx          — Cells visited, backtracks, path length
│   ├── App.tsx
│   └── main.tsx
└── public/
```

---

## Learning Curriculum

### Phase 1: Concept (Understanding the Theory)

**Step 1: What is a stack?**
Think of a stack of plates. You can only add to the top and remove from the top. That's it. Last in, first out.

```typescript
interface Stack<T> {
  push(item: T): void    // put on top
  pop(): T | undefined    // take off top
  peek(): T | undefined   // look at top
  isEmpty(): boolean
  size(): number
}
```

**Step 2: Backtracking = stacks in action**
Imagine you're in a maze. At every intersection, you pick a direction and walk. Dead end? Walk back to the last intersection and try another direction. That "walking back" is popping the stack.

The stack remembers your entire path. Each push is a step forward, each pop is a step back. When you pop all the way back to an intersection with unexplored directions, you try the next one.

**Step 3: The JavaScript call stack**
Every time a function is called, JavaScript pushes a "frame" onto the call stack. When the function returns, it pops. The recursive version of maze generation uses the call stack itself as the stack. The iterative version uses an explicit Stack object — same algorithm, same behavior.

```javascript
// Recursive (uses the call stack implicitly):
function generate(cell) {
  cell.visited = true
  for (const neighbor of shuffledNeighbors(cell)) {
    if (!neighbor.visited) {
      removeWall(cell, neighbor)
      generate(neighbor)  // ← PUSH onto call stack
    }
  }
  // function returns ← POP from call stack (backtrack)
}

// Iterative (uses an explicit stack):
function generate(start) {
  const stack = new Stack()
  stack.push(start)
  while (!stack.isEmpty()) {
    const cell = stack.peek()
    const neighbor = getRandomUnvisitedNeighbor(cell)
    if (neighbor) {
      removeWall(cell, neighbor)
      neighbor.visited = true
      stack.push(neighbor)    // ← PUSH
    } else {
      stack.pop()              // ← POP (backtrack)
    }
  }
}
```

Both produce the same maze. The iterative version lets you visualize the stack and step through it.

**Step 4: Why two implementations (array vs linked list)?**
- **Array-based**: uses `Array.push()` and `Array.pop()` internally. Simple. Good for most cases.
- **Linked-list-based**: each node points to the one below. No resizing. Uses your Node class from Module 1. Shows that data structures are composable — a stack can be built ON TOP of a linked list.

### Phase 2: Implement (Build It From Scratch)

**Step 1: Array-based Stack**
1. `push(item)` — add to end of internal array
2. `pop()` — remove and return last element
3. `peek()` — return last element without removing
4. `isEmpty()` / `size()` — simple checks
5. `toArray()` — for debugging/display
6. `clear()` — empty the stack

**Step 2: Linked-list-based Stack**
1. Use your `Node` class from Module 1
2. `push(item)` — prepend to head (O(1))
3. `pop()` — remove head (O(1))
4. `peek()` — return head value
5. Compare: same interface, different internals

**Step 3: Grid / Cell representation**
1. `Cell` class: row, col, walls (top/right/bottom/left booleans), visited flag
2. `Grid` class: 2D array of Cells
3. `getNeighbors(cell)` — return adjacent cells (up/down/left/right)
4. `getUnvisitedNeighbors(cell)` — filter to only unvisited
5. `removeWall(cellA, cellB)` — remove the wall between two adjacent cells

**Step 4: Maze Generator (Recursive Backtracking)**
1. Initialize grid with all walls up
2. Create a Stack of Cells
3. Push starting cell, mark visited
4. While stack not empty:
   - Peek at top cell
   - Get random unvisited neighbor
   - If found: remove wall between them, push neighbor, mark visited
   - If none: pop (backtrack)
5. Return the completed grid
6. **Crucially**: yield/emit state at each step for visualization

**Step 5: Maze Solver (DFS)**
1. Create a Stack of Cells
2. Push start cell
3. Track parent map for path reconstruction
4. While stack not empty:
   - Pop cell
   - If it's the exit: reconstruct path from parent map, return it
   - For each unvisited passable neighbor: mark visited, record parent, push
5. If stack empties: no solution
6. Yield state at each step for visualization

### Phase 3: Test (Verify Correctness)

**Stack tests:**
- Push/pop ordering (LIFO verified)
- Pop from empty stack returns undefined
- Peek doesn't modify the stack
- Size tracks correctly after push/pop sequences
- Clear empties the stack

**Maze generator tests:**
- Every cell is visited (reachable) after generation
- There exists a path from any cell to any other cell (connected maze)
- Walls are properly removed between adjacent cells only
- Grid dimensions are correct

**Maze solver tests:**
- Finds a path from start to exit (when one exists)
- Path is valid (each step is to an adjacent passable cell)
- Returns no solution for an impossible maze (if you construct one manually)
- Works on 1x1 grid, 2x2 grid, large grids

### Phase 4: Build (React Visualization)

**Maze Grid:**
1. Grid of cells rendered with CSS Grid or Canvas
2. Walls drawn as thick borders between cells
3. Color-coded cells:
   - **White**: unvisited
   - **Blue**: visited (in current generation/solve pass)
   - **Yellow**: current cell (top of stack)
   - **Dark blue**: backtracked (popped from stack)
   - **Green**: start cell
   - **Red**: exit cell
   - **Gold path**: solution route (after solving)
4. Animate generation: watch walls being carved one step at a time
5. Animate solving: watch the robot explore and backtrack

**Stack Visualizer (side panel):**
1. Vertical stack display — top of stack at the top
2. Each entry shows the cell coordinates: `(row, col)`
3. Push animation: new card slides in from top
4. Pop animation: top card slides out
5. Current stack depth displayed
6. Highlight: when the stack pops (backtrack), flash the cell red briefly on the grid

**Controls:**
1. **"Generate Maze"** — start generation with animation
2. **"Solve Maze"** — start DFS solver with animation
3. **Speed slider** — control animation speed (slow for learning, fast for fun)
4. **"Step" button** — advance one step at a time (push or pop)
5. **"Reset"** — clear and start over
6. **Grid size selector** — 10x10, 20x20, 30x30, custom
7. **Toggle**: show/hide stack panel, show/hide algorithm step description

**Algorithm Info Panel:**
1. Current step in plain English:
   - "At cell (3, 4). Neighbors: (3,5) unvisited, (2,4) visited. Choosing (3,5)."
   - "Dead end at (7, 2). Backtracking... popping to (7, 1)."
2. Push/pop count
3. Current phase: Generating / Solving / Complete

**Stats Panel:**
1. Cells visited during generation
2. Total backtracks during generation
3. Cells explored during solve
4. Solution path length
5. Dead ends hit during solve
6. Stack max depth (peak memory usage)

### Phase 5: Challenge (Stretch Goals)

1. **Compare algorithms** — implement BFS solver too (uses a Queue from Module 4 preview). Run DFS and BFS side by side on the same maze. DFS finds A path; BFS finds the SHORTEST path.
2. **Maze generation variants** — Prim's algorithm (uses random selection, not backtracking), Kruskal's algorithm (uses sets). Compare the "feel" of mazes generated by different algorithms.
3. **Recursive vs iterative** — implement the recursive version and show the call stack frames alongside the explicit stack. Prove they produce the same behavior.
4. **Balanced parentheses** — bonus mini-feature: input a string of `()[]{}`, use a stack to check if they're properly nested. Quick, classic stack exercise.
5. **Largest maze** — how big can you go before the animation lags? Profile the performance.

---

## Connection to Module 1

You built linked lists in Module 1. Now you've seen that a stack can be implemented ON TOP of a linked list — `push` is just `prepend`, `pop` is just removing the head. Data structures are composable. This theme continues throughout the course.

## Connection to Module 8

The maze solver IS depth-first search on a graph. The grid is the graph, cells are vertices, passable adjacent cells share edges. When you get to Module 8 (Graphs), DFS will feel familiar — you already built it here with a stack. The difference: Module 8 generalizes from a grid to any graph structure.

---

## Key Takeaways

By the end of this module you should be able to:
- [ ] Explain LIFO and how it enables backtracking
- [ ] Implement a stack using both arrays and linked lists
- [ ] Generate a maze using recursive backtracking (a stack-based algorithm)
- [ ] Solve a maze using DFS with an explicit stack
- [ ] Understand the JavaScript call stack and how recursion relates to stacks
- [ ] Recognize that DFS = stack, BFS = queue (preview of Modules 4 and 8)
