# Module 3: SQL Fundamentals — "Blog CMS Database"

## Overview

Design and build a blog CMS database from scratch using SQLite — users, posts, categories, tags, comments. Write progressively complex queries through an interactive SQL console. This module establishes the **shared database** that all future modules build upon.

**Why this project?** You work with WordPress databases every day. The `wp_posts`, `wp_terms`, `wp_term_relationships` tables follow the exact same relational patterns you'll build here. Understanding schema design and JOINs from first principles makes you faster at debugging WordPress queries and building custom solutions.

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| TypeScript | Backend API and types |
| React (Vite) | Interactive SQL console UI |
| Node + Express | API server for running queries |
| better-sqlite3 | SQLite driver (synchronous, simple, no setup) |
| Vitest | Testing queries and schema |

**This module introduces the backend and database.** From here on, most modules will have both a frontend and backend.

---

## What You'll Learn

### Core Concepts
- **Schema design** — tables, columns, data types, constraints
- **Primary keys** — unique identifiers for every row
- **Foreign keys** — relationships between tables
- **Relationships** — one-to-one, one-to-many, many-to-many
- **Normalization** — why you split data across tables instead of one mega-table
- **JOIN types** — INNER, LEFT, RIGHT (with visual explanations)
- **Aggregates** — COUNT, SUM, AVG, GROUP BY, HAVING
- **Subqueries and CTEs** — queries within queries
- **Query execution order** — FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY

### The Schema

```sql
-- Core tables
users (
  id INTEGER PRIMARY KEY,
  username TEXT UNIQUE NOT NULL,
  email TEXT UNIQUE NOT NULL,
  display_name TEXT NOT NULL,
  role TEXT DEFAULT 'subscriber',  -- admin, editor, author, subscriber
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
)

posts (
  id INTEGER PRIMARY KEY,
  author_id INTEGER REFERENCES users(id),
  title TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  body TEXT NOT NULL,
  status TEXT DEFAULT 'draft',  -- draft, published, archived
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
)

categories (
  id INTEGER PRIMARY KEY,
  name TEXT UNIQUE NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  description TEXT
)

-- Many-to-many junction table
post_categories (
  post_id INTEGER REFERENCES posts(id),
  category_id INTEGER REFERENCES categories(id),
  PRIMARY KEY (post_id, category_id)
)

tags (
  id INTEGER PRIMARY KEY,
  name TEXT UNIQUE NOT NULL,
  slug TEXT UNIQUE NOT NULL
)

-- Many-to-many junction table
post_tags (
  post_id INTEGER REFERENCES posts(id),
  tag_id INTEGER REFERENCES tags(id),
  PRIMARY KEY (post_id, tag_id)
)

-- Self-referential for threaded comments (used again in Module 7)
comments (
  id INTEGER PRIMARY KEY,
  post_id INTEGER REFERENCES posts(id),
  user_id INTEGER REFERENCES users(id),
  parent_comment_id INTEGER REFERENCES comments(id),  -- NULL = top-level
  body TEXT NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
)
```

### WordPress Parallels

| This Project | WordPress Equivalent |
|--------------|---------------------|
| `users` | `wp_users` |
| `posts` | `wp_posts` |
| `categories` + `tags` | `wp_terms` + `wp_term_taxonomy` |
| `post_categories` + `post_tags` | `wp_term_relationships` |
| `comments` | `wp_comments` |

---

## Project Architecture

```
03-sql-fundamentals/
├── MODULE.md
├── package.json
├── vite.config.ts
├── tsconfig.json
├── index.html
├── src/
│   ├── api/
│   │   ├── server.ts              — Express server
│   │   ├── routes/
│   │   │   ├── query.ts           — Execute arbitrary SQL (for the console)
│   │   │   ├── posts.ts           — CRUD endpoints for posts
│   │   │   └── users.ts           — CRUD endpoints for users
│   │   └── db.ts                  — Database connection
│   ├── components/
│   │   ├── SqlConsole.tsx         — Interactive SQL input + results table
│   │   ├── SchemaViewer.tsx       — Visual ER diagram of tables
│   │   ├── JoinVisualizer.tsx     — Venn-diagram-style JOIN explorer
│   │   ├── QueryChallenge.tsx     — Guided SQL challenges
│   │   └── ResultsTable.tsx       — Render query results as a table
│   ├── App.tsx
│   └── main.tsx
├── shared/
│   └── db/
│       ├── schema.sql             — CREATE TABLE statements
│       └── seed.sql               — Sample data (10+ users, 30+ posts, etc.)
└── public/
```

---

## Learning Curriculum

### Phase 1: Schema Design (Think Before You Build)

**Step 1: Entities and attributes**
Before writing any SQL, identify what "things" exist in a blog:
- Users (who writes and comments)
- Posts (the content)
- Categories (broad groupings)
- Tags (specific labels)
- Comments (reader responses)

**Step 2: Relationships**
- A user **has many** posts (one-to-many)
- A post **belongs to** one author (many-to-one)
- A post **has many** categories; a category **has many** posts (many-to-many → junction table)
- A comment **belongs to** a post AND a user
- A comment **can have** a parent comment (self-referential → threaded replies)

**Step 3: Why not one big table?**
Imagine storing the author's email in every post row. If they change their email, you'd have to update 50 rows. That's **data redundancy**. Normalization splits data into related tables to avoid this.

**Step 4: Junction tables**
Many-to-many relationships need a third table. `post_categories` holds pairs of (post_id, category_id). This is exactly how WordPress's `wp_term_relationships` works.

### Phase 2: Create the Database

**Step 1: Write the schema**
Write `CREATE TABLE` statements with proper constraints:
- `PRIMARY KEY` for unique IDs
- `REFERENCES` for foreign keys
- `NOT NULL` for required fields
- `DEFAULT` for sensible defaults
- `UNIQUE` for fields that can't be duplicated

**Step 2: Write seed data**
Insert realistic sample data:
- 10+ users with different roles
- 30+ posts across different statuses (draft, published, archived)
- 5+ categories, 15+ tags
- Post-category and post-tag relationships
- 50+ comments with some threaded replies (parent_comment_id)

**Step 3: Set up the backend**
- Install `better-sqlite3` (synchronous SQLite driver for Node)
- Create the Express server with a route to execute SQL queries
- Initialize the database from schema.sql and seed.sql on first run

### Phase 3: Learn SQL Queries (Progressive Difficulty)

**Level 1: Basic SELECT**
```sql
-- All published posts
SELECT * FROM posts WHERE status = 'published';

-- Users who are authors or admins
SELECT username, role FROM users WHERE role IN ('author', 'admin');

-- Posts ordered by newest first
SELECT title, created_at FROM posts ORDER BY created_at DESC;

-- First 10 posts (pagination)
SELECT title FROM posts LIMIT 10 OFFSET 0;
```

**Level 2: JOINs**
```sql
-- Posts with their author's name
SELECT p.title, u.display_name
FROM posts p
INNER JOIN users u ON p.author_id = u.id;

-- All users and their post count (including users with 0 posts)
SELECT u.username, COUNT(p.id) as post_count
FROM users u
LEFT JOIN posts p ON u.id = p.author_id
GROUP BY u.id;

-- Posts with their categories
SELECT p.title, c.name as category
FROM posts p
JOIN post_categories pc ON p.id = pc.post_id
JOIN categories c ON pc.category_id = c.id;
```

**Level 3: Aggregates and Grouping**
```sql
-- How many posts per category
SELECT c.name, COUNT(pc.post_id) as post_count
FROM categories c
LEFT JOIN post_categories pc ON c.id = pc.category_id
GROUP BY c.id
ORDER BY post_count DESC;

-- Authors with more than 5 published posts
SELECT u.display_name, COUNT(*) as published
FROM users u
JOIN posts p ON u.id = p.author_id
WHERE p.status = 'published'
GROUP BY u.id
HAVING published > 5;
```

**Level 4: Subqueries and CTEs**
```sql
-- Posts that have more comments than the average
SELECT p.title, comment_count
FROM (
  SELECT post_id, COUNT(*) as comment_count
  FROM comments
  GROUP BY post_id
) sub
JOIN posts p ON sub.post_id = p.id
WHERE comment_count > (SELECT AVG(c) FROM (SELECT COUNT(*) as c FROM comments GROUP BY post_id));

-- CTE: same thing but more readable
WITH comment_counts AS (
  SELECT post_id, COUNT(*) as cnt
  FROM comments
  GROUP BY post_id
),
avg_comments AS (
  SELECT AVG(cnt) as avg_cnt FROM comment_counts
)
SELECT p.title, cc.cnt
FROM comment_counts cc
JOIN posts p ON cc.post_id = p.id
CROSS JOIN avg_comments ac
WHERE cc.cnt > ac.avg_cnt;
```

### Phase 4: Build the UI

**Interactive SQL Console:**
1. Text area for writing SQL queries
2. "Run" button (Ctrl+Enter)
3. Results displayed as a formatted table
4. Error messages for invalid SQL
5. Query history (recent queries list)

**Schema Viewer:**
1. Visual diagram showing all tables and their relationships
2. Click a table to see its columns, types, and constraints
3. Lines between tables showing foreign key relationships

**JOIN Visualizer:**
1. Pick two tables from dropdowns
2. Pick a JOIN type (INNER, LEFT, RIGHT)
3. See a Venn diagram highlighting which rows are included
4. See the actual query result below

**Query Challenges:**
1. Guided challenges with increasing difficulty
2. Description of what to query ("Find all posts by authors who joined in 2024")
3. Hint system (reveal the hint step by step)
4. Check your answer against the expected result

### Phase 5: Challenge (Stretch Goals)

1. **CRUD API** — Build full REST endpoints: create post, update post, delete post (with cascading deletes)
2. **Full-text search** — Use SQLite's FTS5 extension to search post content
3. **Views** — Create SQL views for common queries (e.g., `published_posts_with_authors`)
4. **Transactions** — Understand BEGIN/COMMIT/ROLLBACK with a "transfer category" operation
5. **Data migration** — Write an ALTER TABLE migration and a data backfill script

---

## Connection to Future Modules

This database grows with you:
- **Module 4 (Queues)** adds a `tickets` table for the support ticket system
- **Module 5 (Priority Queues)** adds `players` and `servers` tables
- **Module 6 (Hash Tables)** adds `api_cache` and `rate_limit_log` tables
- **Module 7 (Trees)** uses the `comments` table (it's already a tree!) and adds `search_terms`
- **Module 9 (B-Trees)** adds indexes to all these tables and measures performance

---

## Key Takeaways

By the end of this module you should be able to:
- [ ] Design a relational schema with proper normalization
- [ ] Explain and use one-to-many and many-to-many relationships
- [ ] Write SELECT queries with JOINs, WHERE, GROUP BY, HAVING, ORDER BY
- [ ] Understand the difference between INNER JOIN, LEFT JOIN, and RIGHT JOIN
- [ ] Use subqueries and CTEs for complex data retrieval
- [ ] Recognize WordPress database patterns in your own schema
