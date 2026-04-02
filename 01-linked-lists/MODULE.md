# Module 1: Linked Lists — "Spotify-Style Music Queue"

## Overview

Build a media player UI with a playlist powered by linked lists. Add tracks, skip forward/back, "Play Next" to insert a song right after the current one, toggle "Repeat All" to loop the playlist, and drag-and-drop to reorder — all operations that show exactly why linked lists exist.

**Why this project?** Music players are the perfect linked list demo. "Play Next" is an O(1) insert-after-current — the killer feature of linked lists that arrays can't match without shifting everything. Toggling "Repeat All" literally connects the tail node to the head, making it a circular linked list. You've used Spotify's queue a thousand times; now you'll build the data structure behind it.

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| TypeScript | Data structure implementation |
| React (Vite) | Media player UI |
| Vitest | Unit testing the data structure |
| CSS (no framework) | Simple styling to keep focus on logic |

**No backend or database** — this module is pure in-memory to focus entirely on the data structure fundamentals.

---

## What You'll Learn

### Core Concepts
- **Nodes and pointers** — the building block of most data structures
- **Singly Linked List** — each node points to the next
- **Doubly Linked List** — each node points to both next AND previous (needed for prev track)
- **Circular Linked List** — tail connects back to head (repeat-all mode)
- **Arrays vs Linked Lists** — when each is better and why
- **Time complexity** — O(1) insert/delete at known positions, O(n) search

### Key Operations Comparison
| Operation | Array | Linked List | Music Player Action |
|-----------|-------|-------------|-------------------|
| Access by index | O(1) | O(n) | "Jump to track #7" |
| Insert after current | O(n) | **O(1)** | **"Play Next"** |
| Insert at end | O(1)* | O(1) | "Add to Queue" |
| Remove current | O(n) | **O(1)** | "Remove from Queue" |
| Reorder (move a node) | O(n) | **O(1)** | **Drag-and-drop** |
| Search | O(n) | O(n) | "Find this song" |

*O(1) amortized for dynamic arrays

The bolded rows are where linked lists shine — and they're exactly the operations a music queue needs most.

---

## Project Architecture

```
01-linked-lists/
├── MODULE.md
├── package.json
├── vite.config.ts
├── tsconfig.json
├── index.html
├── src/
│   ├── data-structures/
│   │   ├── SinglyLinkedList.ts       — Singly linked list class
│   │   ├── SinglyLinkedList.test.ts  — Tests
│   │   ├── DoublyLinkedList.ts       — Doubly linked list class
│   │   ├── DoublyLinkedList.test.ts  — Tests
│   │   └── Playlist.ts              — Playlist class (doubly + circular)
│   ├── components/
│   │   ├── MusicPlayer.tsx           — Main player UI (album art, controls)
│   │   ├── PlaylistView.tsx          — Track list with drag-and-drop
│   │   ├── NowPlaying.tsx            — Current track display
│   │   ├── NodeChainViz.tsx          — Visual chain of linked nodes
│   │   ├── PlayerControls.tsx        — Play/pause, prev, next, shuffle, repeat
│   │   └── QueueManager.tsx          — "Play Next" / "Add to Queue" interface
│   ├── App.tsx
│   └── main.tsx
└── public/
    └── tracks/                       — Sample audio or mock track data
```

---

## Learning Curriculum

### Phase 1: Concept (Understanding the Theory)

**Step 1: What is a node?**
A node is just an object that holds two things: a value, and a pointer (reference) to another node.

```typescript
// This is the entire concept:
class Node<T> {
  value: T
  next: Node<T> | null = null
}
```

That's it. A linked list is just nodes pointing to each other in a chain. In our case, each node holds a Track object.

**Step 2: Why not just use an array for a playlist?**
Arrays are great when you need random access (jump to track #7). But for a music queue:
- **"Play Next"** means inserting right after the current track. In an array, every track after that shifts down (O(n)). In a linked list, you just reassign two pointers (O(1)).
- **Removing the current track** from an array shifts everything. From a linked list: reassign pointers, done.
- **Reordering via drag-and-drop** means removing from one position and inserting at another. Arrays: two O(n) shifts. Linked list: pointer reassignment.

Spotify/Apple Music/YouTube Music all face this exact problem at scale.

**Step 3: Singly vs Doubly**
- **Singly**: each node has `next` only. Can only go forward. You can implement "next track" but not "previous track."
- **Doubly**: each node has `next` AND `prev`. Can go both directions. Essential for a music player — you need both skip-forward and skip-back.

**Step 4: Circular — the "Repeat All" toggle**
In a normal list, the last track's `next` is `null` — playback stops. In a circular list, the last track's `next` points back to the first track. Toggle repeat-all = toggle whether `tail.next` is `null` or `head`.

```
Normal:    [Track1] → [Track2] → [Track3] → null
                                              ↑ playback stops

Circular:  [Track1] → [Track2] → [Track3] → [Track1] → ...
                                              ↑ loops forever
```

With a doubly circular list, `head.prev` points to `tail` too — so "previous" from the first track wraps to the last.

**Step 5: When to use linked lists in practice**
- When you need frequent insertion/deletion at arbitrary positions
- When you don't need random access by index
- As building blocks for stacks, queues, hash table chaining, LRU caches
- Music queues, undo systems, browser history, memory allocation

### Phase 2: Implement (Build It From Scratch)

Build in this order — each builds on the previous:

**Step 1: Node class**
- Create `Node<T>` with `value` and `next` properties

**Step 2: Singly Linked List**
Implement these methods one at a time:
1. `append(value)` — add to end
2. `prepend(value)` — add to beginning
3. `toArray()` — convert to array for easy debugging/display
4. `size()` — count nodes by traversal (or track with a counter)
5. `find(predicate)` — search for a node matching a condition
6. `insertAfter(node, value)` — insert directly after a given node (**the key operation**)
7. `remove(node)` — remove a specific node
8. `reverse()` — reverse the list in-place (classic interview question)

**Step 3: Doubly Linked List**
Extend to doubly linked:
1. Add `prev` pointer to Node
2. Update all operations to maintain both `prev` and `next` pointers
3. Add `traverseBackward()` — go from tail to head
4. `moveBefore(node, target)` — move a node to just before target (for drag-and-drop)
5. `moveAfter(node, target)` — move a node to just after target

**Step 4: Playlist class (Doubly + Circular)**
Specialized doubly linked list for the music player:
1. `addTrack(track)` — append to end of playlist
2. `playNext(track)` — insert after current track (the star of the show)
3. `addToQueue(track)` — append to end
4. `skipForward()` — move current pointer to `current.next`
5. `skipBack()` — move current pointer to `current.prev`
6. `removeTrack(node)` — remove and handle current-track edge case
7. `toggleRepeat()` — connect/disconnect tail↔head to make circular/linear
8. `isRepeating()` — check if circular
9. `shuffle()` — randomize order (Fisher-Yates on linked list)
10. `reorder(node, afterTarget)` — move a track to a new position (drag-and-drop)
11. `getCurrent()` — return the currently playing track
12. `getPlaylist()` — return all tracks in order

### Phase 3: Test (Verify Correctness)

Write Vitest tests covering:
- Empty playlist operations (skip on empty, remove from empty)
- Single track: skip forward wraps or stops, skip back wraps or stops
- `playNext` inserts directly after current, not at the end
- `addToQueue` always adds to the end
- After `skipForward` past last track: stops (linear) vs wraps to first (circular)
- After `skipBack` past first track: stops (linear) vs wraps to last (circular)
- `toggleRepeat` switches between circular and linear correctly
- Reorder: move track from position 3 to after position 1
- Remove current track: current advances to next
- Shuffle produces a valid permutation (all tracks present, order changed)
- Reverse: playlist plays in reverse order

### Phase 4: Build (React Visualization)

**The UI should have two main areas:**

**Media Player Panel:**
1. **Now Playing** — current track name, artist, (mock) album art
2. **Progress bar** — simulated playback progress (auto-advances after a few seconds for demo)
3. **Controls** — Previous, Play/Pause, Next
4. **Toggle buttons** — Repeat (circular toggle), Shuffle
5. When a track "finishes," auto-advance to next (or loop if repeat is on)

**Playlist Panel:**
1. **Track list** — ordered list of all tracks in the playlist
2. **Current track highlighted** — visually distinct from others
3. **"Play Next" button** on each track — inserts it after the currently playing track
4. **"Remove" button** on each track
5. **Drag-and-drop reorder** — drag a track to a new position in the list
6. **"Add Track" button** — add from a mock library of available tracks

**Node Chain Visualization (learning panel, toggleable):**
1. Horizontal chain of boxes connected by arrows
2. Each box shows track name
3. `next` arrows going right, `prev` arrows going left
4. Current track highlighted
5. When repeat is ON: show the circular arrow from tail back to head
6. When "Play Next" is used: animate the new node inserting into the chain
7. When reordering: animate the node detaching and reattaching

**Stats Panel:**
1. Playlist length
2. Current mode: Linear / Repeat All
3. Current position in playlist (track 3 of 12)

### Phase 5: Challenge (Stretch Goals)

Once the core project works:
1. **"Play Next" queue vs main playlist** — Spotify actually has two lists: the main playlist and a "Play Next" queue that drains first. Implement this with two linked lists.
2. **History stack** — track what's been played (connects to Module 2: Stacks)
3. **Detect a cycle** — Floyd's tortoise and hare algorithm. Can you detect if the list is circular without checking the `isRepeating` flag?
4. **Find the middle track** — two-pointer technique (fast/slow pointers) — classic interview problem
5. **Merge two sorted playlists** — merge by artist name, maintaining sorted order

---

## Key Takeaways

By the end of this module you should be able to:
- [ ] Explain what nodes and pointers are
- [ ] Implement a singly, doubly, and circular linked list from scratch
- [ ] Know when to use a linked list vs an array (and articulate the tradeoffs)
- [ ] Understand O(1) insert-after and remove vs O(n) for arrays
- [ ] Recognize linked lists as building blocks for future data structures (stacks, queues, hash tables, LRU caches)
