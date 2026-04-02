# Module 6: Hash Tables — "API Cache + Rate Limiter"

## Overview

Build two tools powered by hash tables: (1) An LRU cache that stores API responses so repeated requests are instant, using a hash map + doubly linked list for eviction. (2) A rate limiter that tracks how many requests each IP address has made in a sliding time window. Both backed by the shared SQLite database for persistence.

**Why this project?** Hash tables are THE most frequently used data structure in software. PHP's associative arrays, JavaScript objects and Maps, Python dictionaries, Redis — all hash tables. WordPress's transients API is literally a key-value cache. You use hash tables every day without realizing it.

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| TypeScript | Data structure + backend |
| React (Vite) | UI with cache hit/miss visualization |
| Node + Express | API server with caching layer |
| better-sqlite3 | Shared SQLite database |
| Vitest | Unit testing |

---

## What You'll Learn

### Core Concepts
- **Hash functions** — converting any key into an array index
- **Collision handling** — what happens when two keys hash to the same index
- **Chaining** — each bucket is a linked list of entries (uses your Module 1 knowledge!)
- **Load factor** — ratio of entries to buckets, and when to resize
- **O(1) average operations** — and why worst case is O(n)
- **LRU Cache** — Least Recently Used eviction using hash map + doubly linked list
- **Rate limiting** — sliding window counter pattern

### How Hashing Works

```
Key: "hello"
  ↓ hash function
Hash: 2348710293
  ↓ modulo bucket count
Index: 2348710293 % 16 = 5
  ↓
Bucket 5: [("hello", value)]
```

Two keys CAN hash to the same index (collision). Chaining handles this with a linked list per bucket:
```
Bucket 5: [("hello", 1)] → [("world", 2)] → null
```

### Operations

| Operation | Average | Worst Case | Description |
|-----------|---------|------------|-------------|
| `set(key, value)` | O(1) | O(n) | Insert or update |
| `get(key)` | O(1) | O(n) | Retrieve by key |
| `delete(key)` | O(1) | O(n) | Remove by key |
| `has(key)` | O(1) | O(n) | Check existence |

Worst case happens when ALL keys hash to the same bucket (everything in one linked list). A good hash function and proper resizing prevent this.

### Database Additions

```sql
-- Persistent cache entries
api_cache (
  id INTEGER PRIMARY KEY,
  cache_key TEXT UNIQUE NOT NULL,
  endpoint TEXT NOT NULL,
  response_data TEXT NOT NULL,    -- JSON stringified
  hit_count INTEGER DEFAULT 0,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  expires_at DATETIME NOT NULL
)

-- Rate limit tracking
rate_limit_log (
  id INTEGER PRIMARY KEY,
  ip_address TEXT NOT NULL,
  endpoint TEXT NOT NULL,
  request_count INTEGER DEFAULT 1,
  window_start DATETIME NOT NULL,
  window_end DATETIME NOT NULL
)
```

---

## Project Architecture

```
06-hash-tables/
├── MODULE.md
├── package.json
├── vite.config.ts
├── tsconfig.json
├── index.html
├── src/
│   ├── data-structures/
│   │   ├── HashTable.ts           — Hash table with chaining
│   │   ├── HashTable.test.ts      — Tests
│   │   ├── LRUCache.ts            — LRU cache (hash map + doubly linked list)
│   │   ├── LRUCache.test.ts       — Tests
│   │   └── hash-functions.ts      — Different hash function implementations
│   ├── api/
│   │   ├── server.ts              — Express server
│   │   ├── middleware/
│   │   │   ├── cache.ts           — Caching middleware using LRU cache
│   │   │   └── rateLimiter.ts     — Rate limiting middleware
│   │   └── routes/
│   │       ├── posts.ts           — Blog posts API (cached)
│   │       ├── cache.ts           — Cache stats/management
│   │       └── rateLimit.ts       — Rate limit stats
│   ├── components/
│   │   ├── ApiTester.tsx          — Make API requests, see cache behavior
│   │   ├── HashTableViz.tsx       — Visual hash table with buckets and chains
│   │   ├── LRUCacheViz.tsx        — Visual LRU with access order
│   │   ├── RateLimiterViz.tsx     — Request counter per IP
│   │   ├── CacheStats.tsx         — Hit rate, evictions, etc.
│   │   └── HashFunctionDemo.tsx   — See how different inputs hash
│   ├── App.tsx
│   └── main.tsx
└── public/
```

---

## Learning Curriculum

### Phase 1: Concept (Understanding the Theory)

**Step 1: The key insight**
Arrays give you O(1) access IF you know the index. Hash tables let you convert ANY key (string, number, object) into an array index. That's the whole trick.

**Step 2: What makes a good hash function?**
- **Deterministic**: same key always produces same hash
- **Uniform distribution**: keys spread evenly across buckets
- **Fast to compute**: the hash function runs on every operation
- Common approach: multiply char codes, use prime numbers, modulo by bucket count

**Step 3: Collision handling — Chaining**
Each bucket holds a linked list. On collision, new entries append to the chain:
1. Hash the key to find the bucket
2. Walk the linked list in that bucket
3. If key found → update value
4. If not found → append new node

**Step 4: Load factor and resizing**
- Load factor = entries / buckets
- When load factor exceeds ~0.75, double the bucket count and rehash everything
- This keeps chains short and maintains O(1) average performance

**Step 5: LRU Cache — the hash map + linked list combo**
The most elegant data structure combination:
- **Hash map** → O(1) lookup by key
- **Doubly linked list** → O(1) move-to-front (most recently used) and remove-from-back (least recently used)
- On `get(key)`: look up in hash map, move node to front of linked list
- On `set(key, value)`: if full, remove tail of linked list (LRU item), delete from hash map
- This is literally how Redis LRU eviction works

### Phase 2: Implement (Build It From Scratch)

**Step 1: Hash function**
1. Simple hash: sum of char codes modulo size
2. Better hash: DJB2 algorithm (multiply by 33, add char code)
3. Compare distribution of both with a visualization

**Step 2: HashTable class**
1. Internal array of buckets (each bucket is a linked list or array)
2. `hash(key)` — convert key to index
3. `set(key, value)` — hash, find bucket, insert or update
4. `get(key)` — hash, find bucket, search chain
5. `delete(key)` — hash, find bucket, remove from chain
6. `has(key)` — boolean check
7. `resize()` — double buckets, rehash all entries
8. `keys()`, `values()`, `entries()` — iterators
9. Track size and load factor

**Step 3: LRU Cache class**
1. Combine a `Map<string, DoublyLinkedListNode>` with a `DoublyLinkedList`
2. `get(key)` — if exists, move to front, return value. If not, return undefined.
3. `put(key, value)` — if exists, update and move to front. If full, evict tail. Insert at front.
4. `evict()` — remove tail node, delete from map
5. `getStats()` — hits, misses, evictions, hit rate

**Step 4: Rate Limiter**
1. Hash map of IP → { count, windowStart }
2. `isAllowed(ip)` — check if under limit within current window
3. `recordRequest(ip)` — increment count
4. `resetWindow(ip)` — reset when window expires
5. Sliding window: use timestamps, not fixed intervals

### Phase 3: Test (Verify Correctness)

- Set/get returns correct values
- Overwrite existing key
- Delete removes entry
- Hash collision doesn't lose data (two different keys, same bucket)
- Resize maintains all entries
- LRU evicts least recently USED (not least recently INSERTED)
- LRU get() updates recency
- Rate limiter allows up to limit, blocks after
- Rate limiter resets after window expires

### Phase 4: Build (React Visualization)

**API Tester:**
1. Dropdown of blog API endpoints (from Module 3's posts data)
2. "Send Request" button
3. Response display with: cache HIT (green, instant) or MISS (red, slow with artificial delay)
4. Request count and cache hit rate

**Hash Table Visualizer:**
1. Grid of buckets (array indices 0 to N)
2. Each bucket shows its chain of key-value pairs
3. Highlight the bucket when a key is looked up
4. Color-code by chain length (green=0-1, yellow=2-3, red=4+)
5. Load factor meter — watch it grow toward the resize threshold

**LRU Cache Visualizer:**
1. Horizontal linked list showing cache entries in recency order
2. Most recent on left, least recent on right
3. Capacity bar showing how full the cache is
4. On access: watch the node move to the front (animated)
5. On eviction: watch the tail node disappear
6. Stats: hit count, miss count, hit rate percentage, eviction count

**Hash Function Demo:**
1. Input field: type any string
2. Show the hash value and which bucket it maps to
3. Try different hash functions and compare distribution
4. Scatter plot: hash N random strings, visualize bucket distribution

### Phase 5: Integrate (Database Connection)

**API endpoints:**
- `GET /api/posts` — returns posts (cached with LRU)
- `GET /api/posts/:id` — single post (cached)
- `GET /api/cache/stats` — hit rate, entries, evictions
- `DELETE /api/cache` — flush the cache
- `GET /api/rate-limit/status` — current rate limit state per IP

**SQL practice:**
```sql
-- Most cached endpoints
SELECT endpoint, hit_count, created_at, expires_at
FROM api_cache
ORDER BY hit_count DESC
LIMIT 10;

-- Rate limit violations (IPs that hit the limit)
SELECT ip_address, endpoint, request_count, window_start
FROM rate_limit_log
WHERE request_count >= 100
ORDER BY request_count DESC;

-- Cache efficiency: what percentage of entries are still valid?
SELECT
  COUNT(*) as total_entries,
  SUM(CASE WHEN expires_at > CURRENT_TIMESTAMP THEN 1 ELSE 0 END) as valid,
  SUM(CASE WHEN expires_at <= CURRENT_TIMESTAMP THEN 1 ELSE 0 END) as expired
FROM api_cache;
```

### Phase 6: Challenge (Stretch Goals)

1. **Open addressing** — implement linear probing instead of chaining, compare performance
2. **Consistent hashing** — used in distributed systems to assign data to servers
3. **Bloom filter** — probabilistic "is this key in the set?" with no false negatives
4. **Two-tier cache** — memory LRU + database cache (check memory first, then DB, then origin)
5. **Cache warming** — on startup, pre-populate the cache with the most frequently accessed items from DB

---

## Connection to Other Modules

- **Module 1 (Linked Lists)**: Chaining uses linked lists; LRU cache uses a doubly linked list
- **Module 3 (SQL)**: You're caching query results from the blog CMS database
- **Module 9 (B-Trees)**: Database indexes are another O(log n) lookup structure — compare with hash tables' O(1)

---

## Key Takeaways

By the end of this module you should be able to:
- [ ] Explain how hash functions convert keys to array indices
- [ ] Implement a hash table with chaining from scratch
- [ ] Handle collisions and know when/how to resize
- [ ] Build an LRU cache using a hash map + doubly linked list
- [ ] Implement a sliding window rate limiter
- [ ] Understand why hash tables are O(1) average but O(n) worst case
