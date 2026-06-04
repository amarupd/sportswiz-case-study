# Database Indexing Strategy

## Overview

Database indexing is one of the most important performance optimization techniques used within Sportswiz.

As the platform grows, millions of rows can accumulate across match events, player statistics, scorecards, and tournament data.

Without proper indexing, even simple queries can become slow and negatively impact user experience.

The indexing strategy was designed to:

* Reduce query execution time
* Improve API response performance
* Minimize database resource consumption
* Support realtime operations
* Enable efficient analytics

---

# Indexing Goals

The indexing strategy focuses on:

### Fast Lookups

Retrieve records quickly using indexed columns.

### Efficient Filtering

Support common filtering operations.

### Optimized Sorting

Reduce expensive sort operations.

### Reduced Full Table Scans

Avoid unnecessary database scans.

### Scalable Query Performance

Maintain acceptable performance as data volume grows.

---

# Query Analysis Approach

Before creating indexes, query patterns were analyzed.

Questions considered:

### What data is queried most often?

Examples:

```text
Live Match Data
Player Statistics
Tournament Fixtures
Scorecards
Rankings
```

---

### Which columns are used in filters?

Examples:

```text
match_id
player_id
team_id
tournament_id
status
```

---

### Which columns are used in sorting?

Examples:

```text
created_at
start_time
runs
wickets
points
```

---

# Core Indexing Principles

The platform follows several principles.

### Index Frequently Queried Columns

High-traffic queries receive dedicated indexes.

### Use Composite Indexes

Support multi-column filtering.

### Avoid Over-Indexing

Excessive indexes increase write overhead.

### Monitor Query Plans

Indexes are continuously validated using query analysis.

---

# Match Table Indexes

## Matches Table

Common Query:

```sql
SELECT *
FROM matches
WHERE status = 'LIVE';
```

Index:

```sql
CREATE INDEX idx_match_status
ON matches(status);
```

Benefits:

* Fast live match retrieval
* Reduced scan time

---

## Upcoming Matches

Common Query:

```sql
SELECT *
FROM matches
WHERE start_time > NOW()
ORDER BY start_time;
```

Index:

```sql
CREATE INDEX idx_match_start_time
ON matches(start_time);
```

Benefits:

* Faster scheduling queries
* Efficient sorting

---

# Ball Events Indexes

The balls table is typically one of the largest tables in the platform.

Every delivery generates a record.

---

## Match Ball Retrieval

Common Query:

```sql
SELECT *
FROM balls
WHERE match_id = 101;
```

Index:

```sql
CREATE INDEX idx_ball_match
ON balls(match_id);
```

---

## Innings Ball Retrieval

Common Query:

```sql
SELECT *
FROM balls
WHERE innings_id = 5;
```

Index:

```sql
CREATE INDEX idx_ball_innings
ON balls(innings_id);
```

---

# Composite Indexes

Composite indexes support multi-column filtering.

---

## Match Commentary

Common Query:

```sql
SELECT *
FROM balls
WHERE match_id = 101
ORDER BY created_at DESC;
```

Index:

```sql
CREATE INDEX idx_match_commentary
ON balls(match_id, created_at);
```

Benefits:

* Fast filtering
* Efficient ordering

---

## Player Match Performance

Common Query:

```sql
SELECT *
FROM player_stats
WHERE player_id = 501
AND match_id = 101;
```

Index:

```sql
CREATE INDEX idx_player_match
ON player_stats(player_id, match_id);
```

Benefits:

* Fast statistics retrieval

---

# Player Statistics Indexes

Player statistics are frequently queried.

---

## Player Profile Queries

Common Query:

```sql
SELECT *
FROM player_stats
WHERE player_id = ?;
```

Index:

```sql
CREATE INDEX idx_player_stats
ON player_stats(player_id);
```

---

## Rankings Queries

Common Query:

```sql
SELECT *
FROM player_stats
ORDER BY runs DESC;
```

Index:

```sql
CREATE INDEX idx_runs_ranking
ON player_stats(runs);
```

Benefits:

* Faster leaderboard generation

---

# Team Statistics Indexes

Common Query:

```sql
SELECT *
FROM team_stats
WHERE team_id = ?;
```

Index:

```sql
CREATE INDEX idx_team_stats
ON team_stats(team_id);
```

---

# Tournament Indexes

Common Query:

```sql
SELECT *
FROM matches
WHERE tournament_id = ?;
```

Index:

```sql
CREATE INDEX idx_tournament_matches
ON matches(tournament_id);
```

Benefits:

* Faster tournament pages
* Efficient fixture retrieval

---

# Notification Indexes

Notifications are frequently queried per user.

Common Query:

```sql
SELECT *
FROM notifications
WHERE user_id = ?
ORDER BY created_at DESC;
```

Index:

```sql
CREATE INDEX idx_user_notifications
ON notifications(user_id, created_at);
```

Benefits:

* Fast notification feeds

---

# Covering Indexes

In some cases, covering indexes eliminate table lookups.

Example:

```sql
SELECT id, status
FROM matches
WHERE tournament_id = ?;
```

Index:

```sql
CREATE INDEX idx_match_covering
ON matches(tournament_id, id, status);
```

Benefits:

* Faster reads
* Reduced I/O

---

# Foreign Key Indexes

All frequently used foreign keys are indexed.

Examples:

```text
team_id
player_id
match_id
innings_id
tournament_id
user_id
```

Benefits:

* Faster joins
* Improved relationship queries

---

# Query Optimization Through Indexing

Before Index:

```text
Full Table Scan
Rows Examined: 1,000,000+
```

---

After Index:

```text
Index Lookup
Rows Examined: < 100
```

Benefits:

* Significant performance improvement
* Reduced CPU utilization

---

# EXPLAIN Plan Analysis

Indexes are validated using:

```sql
EXPLAIN
SELECT *
FROM matches
WHERE status = 'LIVE';
```

Metrics analyzed:

### Type

Desired:

```text
ref
range
const
```

Avoid:

```text
ALL
```

---

### Rows Examined

Lower values indicate better performance.

---

### Key Used

Confirms correct index selection.

---

# Slow Query Monitoring

Slow queries are continuously monitored.

Threshold Example:

```text
> 500ms
```

Actions:

* Analyze execution plan
* Add indexes
* Rewrite query
* Optimize joins

---

# Index Maintenance

Indexes require regular maintenance.

Tasks include:

### Monitoring

Track index effectiveness.

### Cleanup

Remove unused indexes.

### Optimization

Adjust indexes as query patterns evolve.

---

# Write Performance Considerations

Indexes improve reads but increase write cost.

Each insert or update must maintain indexes.

Balance is maintained by:

* Indexing only necessary columns
* Avoiding redundant indexes
* Reviewing usage regularly

---

# Scaling Strategy

As data volume increases:

### Partitioning

Large tables can be partitioned.

Examples:

```text
balls
player_stats
notifications
```

---

### Archiving

Historical data can be archived.

Benefits:

* Smaller active tables
* Faster queries

---

### Read Replicas

Read-heavy workloads move to replicas.

Benefits:

* Reduced primary database load

---

# Common Indexes Summary

| Table         | Index                 |
| ------------- | --------------------- |
| matches       | status                |
| matches       | start_time            |
| matches       | tournament_id         |
| balls         | match_id              |
| balls         | innings_id            |
| balls         | match_id + created_at |
| player_stats  | player_id             |
| player_stats  | player_id + match_id  |
| notifications | user_id + created_at  |

---

# Engineering Outcome

A well-designed indexing strategy enables Sportswiz to:

* Support realtime score retrieval
* Handle large datasets efficiently
* Maintain low query latency
* Reduce infrastructure costs
* Scale with increasing traffic

By carefully analyzing query patterns and applying targeted indexes, database performance remains predictable even as the platform grows significantly in size and usage.
