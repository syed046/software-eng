# Employee Attendance & Analytics API

A high-performance Employee Attendance & Analytics API built with **Python 3.11+**, **FastAPI**, and **MongoDB (6.0+)**, implementing punch-in/out business rules, audit trails, and MongoDB aggregation pipelines.

---

## Deliverables in this Repository

- **`app/main.py`**: Complete FastAPI application containing all models, business logic (rules R1–R10), concurrency-safe handlers, and MongoDB aggregation pipelines.
- **`requirements.txt`**: Python dependencies.
- **`REVIEW.md`**: Detailed review of defects identified and fixed from the starter codebase.
- **`DECISIONS.md`**: Architectural and design rationale for indexing, concurrency handling, tie resolution, headcount calculation, and 100x scalability.
- **`README.md`**: Setup and run instructions.

---

## Prerequisites

- **Python 3.11+**
- **MongoDB 6.0+** (Local instance, Docker container, or MongoDB Atlas cluster)

---

## Setup & Running

1. **Clone the repository**:
   ```bash
   git clone https://github.com/syed046/hrone-attendance-api.git
   cd hrone-attendance-api
   ```

2. **Create and activate a virtual environment**:
   ```bash
   python -m venv .venv
   # Windows:
   .venv\Scripts\activate
   # Linux/macOS:
   source .venv/bin/activate
   ```

3. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

4. **Configure environment variables (optional)**:
   By default, the application connects to:
   - `MONGO_URI`: `mongodb://localhost:27017`
   - `MONGO_DB`: `attendance_db`

   To override:
   ```bash
   export MONGO_URI="mongodb://localhost:27017"
   export MONGO_DB="attendance_db"
   ```

5. **Start the server**:
   ```bash
   uvicorn app.main:app --port 8000
   ```

All indexes are created idempotently at application startup.

---

## API Overview

### Attendance & Employees
- `GET /health`: Readiness probe testing database connectivity.
- `POST /employees`: Create employee (enforces unique `emp_code`).
- `GET /employees`: List employees with pagination and department filtering.
- `POST /attendance/punch-in`: Race-safe punch-in with R1/R2 attendance date and late calculations.
- `POST /attendance/punch-out`: Atomic punch-out with overtime (R3), work hours (R4), and half-day (R5) logic.
- `GET /attendance`: Indexed attendance list with pagination.
- `PATCH /attendance/{emp_code}/{date}`: Manual regularization with full audit history.

### Analytics Aggregations
- `GET /analytics/employees/{emp_code}/monthly`: Monthly attendance summary with working days calculation.
- `GET /analytics/departments/summary`: Department summary including zero-log headcount.
- `GET /analytics/leaderboard/late`: Competition ranking (`$rank`) for late arrivals with tie support.
- `GET /analytics/departments/{department}/trend`: Continuous daily trend with gap-filling (`$densify`) and 7-day moving average.
- `GET /admin/explain/{endpoint}`: MongoDB query execution plan verification (`IXSCAN`).
