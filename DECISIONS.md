# DECISIONS.md

1. **Indexes.**
   We created:
   - `employees: {"emp_code": 1}` (unique): Guarantees unique employee identifiers and enables fast lookups.
   - `employees: {"department": 1, "joined_on": 1}`: Accelerates department filtering and monthly eligibility checks.
   - `attendance_logs: {"emp_code": 1, "date": 1}` (unique): Enforces the one-record-per-day rule and prevents concurrent duplicate punches.
   - `attendance_logs: {"date": -1, "emp_code": 1}`: Supports sorted pagination on `GET /attendance` entirely via index scan (`IXSCAN`).
   - `attendance_logs: {"date": 1, "late_minutes": 1, "emp_code": 1}`: Powers monthly late leaderboard matching and grouping.
   *Rejected*: An index on `attendance_logs: {"punch_in": 1, "punch_out": 1}` was rejected because queries filter on calendar `date` and `emp_code`, making raw timestamp indexes redundant overhead.

2. **Punch-in race.**
   Both requests compute the same attendance `date` from the punch-in instant. Both execute atomic `insert_one` calls against MongoDB. WiredTiger serializes the write on the `(emp_code, date)` unique index: the first request succeeds and returns 201. The second request triggers a `DuplicateKeyError`, which our handler intercepts and maps directly to 409 Conflict.

3. **Ties.**
   The late leaderboard uses MongoDB's `$rank` window function for standard competition ranking (1, 2, 2, 4). The `limit` filter (`rank <= limit`) evaluates *after* ranking. If two employees tie at the cutoff rank, both are returned, potentially yielding more rows than `limit`. Tied employees are sorted deterministically by `emp_code` ascending.

4. **Headcount.**
   The aggregation pipeline starts on the `employees` collection (filtering `joined_on <= month_end`), not `attendance_logs`. Each active employee increments headcount by 1 before performing a `$lookup` to join their monthly logs. Employees with zero logs are fully preserved.

5. **One thing you would change for 100x data.**
   At 100x volume (~10M logs), shard `attendance_logs` on `{emp_code: 1, date: 1}` to distribute write throughput. For analytics, replace real-time cross-collection joins with incremental materialized daily rollups updated via MongoDB change streams or scheduled `$merge` tasks.
