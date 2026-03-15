# Daily Expense Tracker - Theory & How Everything Works

## 1. What is This Project?

This is a **full-stack web application** built using the **MERN-style architecture** (but with MySQL instead of MongoDB). It lets users track daily expenses, set budgets, and view spending analytics.

**Architecture Pattern:** Client-Server with RESTful API

```
[ React Frontend ]  <--HTTP/JSON-->  [ Express Backend ]  <--SQL-->  [ MySQL Database ]
   (Port 5173)                          (Port 5000)                    (Port 3306)
```

---

## 2. Tech Stack - Why Each Technology?

### Frontend
| Technology | Purpose | Why Used |
|---|---|---|
| **React 18** | UI Library | Component-based architecture, virtual DOM for fast updates, large ecosystem |
| **React Router v6** | Client-side routing | Enables SPA (Single Page Application) - no full page reloads when navigating |
| **Axios** | HTTP client | Cleaner API than fetch(), supports interceptors for auto-attaching JWT tokens |
| **Chart.js + react-chartjs-2** | Data visualization | Lightweight charting library for doughnut and line charts |
| **Tailwind CSS** | Styling | Utility-first CSS framework - write styles directly in JSX without separate CSS files |
| **Vite** | Build tool | Faster than Webpack - uses ES modules for instant hot module replacement (HMR) |

### Backend
| Technology | Purpose | Why Used |
|---|---|---|
| **Node.js** | Runtime | JavaScript on the server - same language as frontend, non-blocking I/O |
| **Express.js** | Web framework | Minimal, flexible - handles routing, middleware, and HTTP request/response cycle |
| **mysql2/promise** | Database driver | Async/await support for MySQL queries, connection pooling for performance |
| **JWT (jsonwebtoken)** | Authentication | Stateless auth tokens - no server-side session storage needed |
| **bcryptjs** | Password security | One-way hashing with salt - passwords never stored in plain text |
| **dotenv** | Configuration | Loads environment variables from .env file - keeps secrets out of code |
| **cors** | Security | Controls which origins (domains) can access the API |

### Database
| Technology | Purpose | Why Used |
|---|---|---|
| **MySQL 8.0** | Relational database | ACID compliance, structured data with relationships, powerful querying with JOINs |

---

## 3. How the Frontend Works

### 3.1 Entry Point Flow

```
main.jsx  -->  App.jsx  -->  AuthProvider (wraps everything)
                               |
                               +--> Router
                                     |
                                     +--> Sidebar + Navbar (always visible)
                                     +--> Routes (page content changes)
```

**main.jsx** is the first file that runs. It renders `App.jsx` into the DOM.

### 3.2 Component Architecture

```
App.jsx
├── AuthProvider          (Context - global auth state)
│   └── Router            (URL-based navigation)
│       ├── Sidebar       (Navigation links)
│       ├── Navbar        (Top bar with user info)
│       └── Routes
│           ├── /login          → Login.jsx
│           ├── /register       → Register.jsx
│           ├── /               → Dashboard.jsx    (Protected)
│           ├── /expenses       → Expenses.jsx     (Protected)
│           ├── /expenses/add   → AddExpense.jsx   (Protected)
│           ├── /expenses/edit/:id → EditExpense.jsx (Protected)
│           ├── /budget         → Budget.jsx       (Protected)
│           ├── /reports        → Reports.jsx      (Protected)
│           └── /profile        → Profile.jsx      (Protected)
```

### 3.3 Protected Routes - How They Work

```jsx
const ProtectedRoute = ({ children }) => {
  const { user, loading } = useAuth();
  if (loading) return <Loading />;
  return user ? children : <Navigate to="/login" />;
};
```

**Theory:** A Protected Route is a wrapper component that checks if the user is logged in before showing the page. If not logged in, it redirects to `/login`. This prevents unauthorized access to app features.

### 3.4 State Management - Context API

Instead of Redux (which is more complex), this project uses React's built-in **Context API**:

```
AuthContext
├── user        → Current logged-in user object (or null)
├── loading     → Boolean - is auth check in progress?
├── login()     → Function to set user + save token
└── logout()    → Function to clear user + remove token
```

**How it works:**
1. On app load, `AuthProvider` checks if a token exists in `localStorage`
2. If yes, it calls `GET /api/auth/profile` to validate the token and get user data
3. If the token is valid, `user` state is set → app renders protected content
4. If invalid/expired, token is removed → user sees login page

### 3.5 Service Layer - API Communication

The frontend uses a **service layer pattern** to separate API calls from UI logic:

```
Pages/Components  →  Services  →  Axios Instance  →  Backend API
                                       |
                                  Interceptors:
                                  - Request: Auto-attach Bearer token
                                  - Response: Auto-redirect on 401
```

**Why interceptors?**
- **Request interceptor:** Every API call automatically includes `Authorization: Bearer <token>` header - no need to manually add it everywhere
- **Response interceptor:** If any API returns 401 (unauthorized), the app automatically logs out the user and redirects to login

### 3.6 How Charts Work

The app uses **Chart.js** (via react-chartjs-2 wrapper) for two chart types:

1. **Doughnut Chart** - Shows spending breakdown by category (Food, Transport, etc.)
2. **Line Chart** - Shows daily spending trend over the month

Data flows: `Backend summary API → Dashboard component → ChartContainer component → Chart.js renders canvas`

---

## 4. How the Backend Works

### 4.1 Express Application Structure

```
Request → CORS → JSON Parser → Route Matcher → Auth Middleware → Controller → Response
                                                                      |
                                                                   Database
                                                                      |
                                                              Error Handler (if error)
```

### 4.2 Middleware Chain - Theory

Middleware are functions that run **before** the route handler. They can:
- Modify the request/response
- End the request cycle
- Pass control to the next middleware

**Middleware in this project:**

| Middleware | Purpose | Applied To |
|---|---|---|
| `cors()` | Allow frontend origin to access API | All routes |
| `express.json()` | Parse JSON request body | All routes |
| `authMiddleware` (protect) | Verify JWT token, extract userId | Protected routes only |
| `errorHandler` | Catch errors and send formatted response | All routes (last) |

### 4.3 MVC Pattern (Model-View-Controller)

This project follows the **MVC pattern** (without traditional views since React handles that):

```
Routes (URL mapping)  →  Controllers (Business Logic)  →  Models (Database Queries)
     │                          │                              │
authRoutes.js              authController.js              Direct SQL via pool
expenseRoutes.js           expenseController.js
budgetRoutes.js            budgetController.js
```

**Routes:** Define which URL maps to which controller function
**Controllers:** Contain the business logic (validate input, call database, format response)
**Models:** In this project, SQL queries are written directly in controllers using the connection pool

### 4.4 Connection Pooling - Theory

```javascript
const pool = mysql.createPool({
  connectionLimit: 10,    // Max 10 simultaneous connections
  waitForConnections: true // Queue requests when all connections busy
});
```

**Why pooling?** Opening a new database connection for every request is slow. A pool maintains a set of reusable connections. When a request needs the database, it borrows a connection from the pool and returns it when done.

### 4.5 Route Organization

```
/api/auth/
├── POST   /register       → Create new user account
├── POST   /login           → Authenticate and get token
├── GET    /profile         → Get logged-in user's profile  [Protected]
└── PUT    /profile         → Update profile                [Protected]

/api/expenses/              [All Protected]
├── GET    /                → List expenses (with filters & pagination)
├── POST   /                → Add new expense
├── PUT    /:id             → Update an expense
├── DELETE /:id             → Delete an expense
├── GET    /summary         → Monthly spending analytics
└── GET    /categories      → List all categories

/api/budgets/               [All Protected]
├── GET    /                → Get budgets for a month
├── POST   /                → Set/update a budget (upsert)
└── DELETE /:id             → Delete a budget
```

---

## 5. How Authentication Works (JWT)

### 5.1 What is JWT?

JWT (JSON Web Token) is a compact, self-contained token for securely transmitting information. It has 3 parts:

```
Header.Payload.Signature
  |       |        |
  |       |        └── HMAC-SHA256(header + payload, SECRET_KEY)
  |       └── { id: 1, iat: timestamp, exp: timestamp }
  └── { alg: "HS256", typ: "JWT" }
```

### 5.2 Complete Auth Flow

```
REGISTRATION:
  User fills form → POST /api/auth/register
  → Server validates input
  → Server checks if email exists (prevents duplicates)
  → Server hashes password with bcrypt (10 salt rounds)
  → Server inserts user into database
  → Server generates JWT token (30-day expiry)
  → Returns { id, name, email, token }
  → Frontend stores token in localStorage

LOGIN:
  User fills form → POST /api/auth/login
  → Server finds user by email
  → Server compares password hash with bcrypt.compare()
  → If match: generates new JWT token → returns user + token
  → If no match: returns 401 error

EVERY SUBSEQUENT REQUEST:
  Frontend Axios interceptor adds: Authorization: Bearer <token>
  → Server authMiddleware extracts token from header
  → jwt.verify(token, SECRET) decodes it
  → Extracts userId from decoded payload
  → Attaches req.userId for controller to use
  → If token invalid/expired: returns 401

LOGOUT:
  Frontend removes token from localStorage
  → User state set to null
  → ProtectedRoute redirects to /login
```

### 5.3 Why bcrypt? Why Not Just Hash?

Regular hashing (like SHA-256) is **too fast** - attackers can try billions of guesses per second. bcrypt is intentionally slow and adds a random **salt** to each password, making:
- Rainbow table attacks useless (each password has unique salt)
- Brute force attacks extremely slow

```
Password: "hello123"
bcrypt output: "$2a$10$N9qo8uLOickgx2ZMRZoMy.MrPXhK5J7h0qm..." (includes salt + hash)
```

---

## 6. How the Database Works

### 6.1 Entity Relationship (ER) Diagram

```
┌──────────┐         ┌──────────────┐         ┌──────────┐
│  users   │ 1───M   │   expenses   │   M───1 │categories│
│──────────│         │──────────────│         │──────────│
│ id (PK)  │────────>│ user_id (FK) │         │ id (PK)  │
│ name     │         │ category_id  │<────────│ name     │
│ email    │         │ amount       │         │ icon     │
│ password │         │ description  │         │ color    │
│ currency │         │ expense_date │         └──────────┘
└──────────┘         │ payment_method│              │
      │              │ notes        │              │
      │              └──────────────┘              │
      │                                            │
      │    1───M  ┌──────────┐   M───1             │
      └──────────>│ budgets  │<────────────────────┘
                  │──────────│
                  │ user_id  │
                  │category_id│
                  │ amount   │
                  │ month    │
                  │ year     │
                  └──────────┘
```

### 6.2 Relationships Explained

| Relationship | Type | Meaning |
|---|---|---|
| users → expenses | One-to-Many | One user can have many expenses |
| users → budgets | One-to-Many | One user can set many budgets |
| categories → expenses | One-to-Many | One category can have many expenses |
| categories → budgets | One-to-Many | One category can have many budgets |

### 6.3 Foreign Key Constraints

```sql
FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
-- If a user is deleted, all their expenses are automatically deleted

FOREIGN KEY (category_id) REFERENCES categories(id) ON DELETE RESTRICT
-- Cannot delete a category if expenses exist for it (prevents data loss)
```

### 6.4 Database Views - Pre-computed Queries

Views are **saved SQL queries** that act like virtual tables:

| View | Purpose | Used For |
|---|---|---|
| `v_monthly_spending` | Monthly totals grouped by category | Reports page |
| `v_daily_spending` | Daily totals per user | Trend charts |
| `v_budget_vs_actual` | Budget amount vs actual spending | Budget tracking |

### 6.5 Indexes - Why They Matter

```sql
CREATE INDEX idx_expenses_user_date ON expenses(user_id, expense_date);
CREATE INDEX idx_expenses_category ON expenses(category_id);
```

**Theory:** Without indexes, MySQL scans every row in the table (full table scan). Indexes create a sorted data structure (B-tree) that allows the database to find rows in O(log n) time instead of O(n). Think of it like a book's index vs reading every page.

---

## 7. How Data Flows Through the App

### 7.1 Adding an Expense (Full Journey)

```
Step 1: User fills ExpenseForm (category, amount, date, etc.)
          ↓
Step 2: Form submits → calls expenseService.addExpense(formData)
          ↓
Step 3: Axios sends POST /api/expenses with JSON body
        + Authorization: Bearer <token> (auto-added by interceptor)
          ↓
Step 4: Express receives request
        → cors middleware allows it
        → express.json() parses body
        → authMiddleware verifies token, sets req.userId
          ↓
Step 5: expenseController.addExpense() runs
        → Validates required fields (category_id, amount, expense_date)
        → Executes INSERT INTO expenses (...) VALUES (...)
        → Returns { id, message: 'Expense added' }
          ↓
Step 6: Frontend receives 201 response
        → Navigates to /expenses page
        → Expenses list re-fetches from API
        → New expense appears in the list
```

### 7.2 Dashboard Loading (Full Journey)

```
Step 1: User navigates to / (Dashboard)
          ↓
Step 2: ProtectedRoute checks auth → user exists → renders Dashboard
          ↓
Step 3: Dashboard useEffect fires two parallel API calls:
        - GET /api/expenses/summary?month=3&year=2026
        - GET /api/budgets?month=3&year=2026
          ↓
Step 4: Backend processes each:
        Summary → 3 SQL queries:
          1. Total spent this month (SUM)
          2. Category breakdown (GROUP BY category)
          3. Daily trend (GROUP BY date)
        Budgets → 1 SQL query with subquery for actual spending
          ↓
Step 5: Dashboard renders:
        - Total spent card (big number)
        - Doughnut chart (category breakdown)
        - Line chart (daily trend)
        - Budget progress bars (budget vs actual)
```

---

## 8. Filtering & Pagination - How They Work

### 8.1 Dynamic Query Building

The `GET /api/expenses` endpoint builds SQL dynamically based on query parameters:

```
Base query: SELECT ... FROM expenses WHERE user_id = ?

If category param exists:   + AND category_id = ?
If start_date param exists: + AND expense_date >= ?
If end_date param exists:   + AND expense_date <= ?
If payment_method exists:   + AND payment_method = ?

Then: + ORDER BY expense_date DESC/ASC
Then: + LIMIT 20 OFFSET 0    (pagination)
```

### 8.2 Pagination Math

```
Page 1: LIMIT 20 OFFSET 0    → rows 1-20
Page 2: LIMIT 20 OFFSET 20   → rows 21-40
Page 3: LIMIT 20 OFFSET 40   → rows 41-60

Formula: offset = (page - 1) * limit
Total pages = CEIL(total_rows / limit)
```

---

## 9. Budget System - Upsert Pattern

The budget system uses an **upsert** (update or insert) pattern:

```sql
INSERT INTO budgets (user_id, category_id, amount, month, year)
VALUES (?, ?, ?, ?, ?)
ON DUPLICATE KEY UPDATE amount = VALUES(amount)
```

**Theory:** The `UNIQUE KEY (user_id, category_id, month, year)` constraint ensures only one budget per category per month. If a user sets a budget that already exists, it updates the amount instead of creating a duplicate. This eliminates the need for separate "check if exists → update or insert" logic.

---

## 10. Error Handling Strategy

### Backend

```
Controller throws error → Express error handler middleware catches it
                                    ↓
                            Sends JSON response:
                            {
                              message: "Error description",
                              stack: (only in development mode)
                            }
```

### Frontend

```
API call fails → Axios response interceptor catches it
                        ↓
                 401 error? → Auto-logout + redirect to /login
                 Other error? → Component catches it → Shows error message to user
```

---

## 11. Security Measures

| Threat | Protection |
|---|---|
| **Password theft** | bcrypt hashing (never stored in plain text) |
| **Unauthorized access** | JWT tokens on every protected API call |
| **Cross-origin attacks** | CORS configured to allow only the frontend origin |
| **Data isolation** | Every query filters by `user_id` - users can only see their own data |
| **SQL injection** | Parameterized queries (`?` placeholders) - never string concatenation |
| **Token expiry** | Tokens expire after 30 days - limits damage if token is stolen |

---

## 12. CSV Export - How It Works

```
1. Frontend has all expense data in memory (from API)
2. exportCSV.js maps expenses to rows: [Date, Category, Description, Amount, Method, Notes]
3. Joins rows with commas, wraps values in quotes
4. Creates a Blob (binary data) with CSV content
5. Creates a temporary download URL (URL.createObjectURL)
6. Creates an invisible <a> tag, sets href to URL, triggers click
7. Browser downloads the file as "expenses.csv"
8. Cleans up the temporary URL
```

---

## 13. Deployment Architecture

### Development

```
localhost:5173 (Vite dev server)  ──>  localhost:5000 (Express)  ──>  localhost:3306 (MySQL)
```

### Production (Docker)

```
┌─────────────────────────────────────────────┐
│              Docker Network                  │
│                                              │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐ │
│  │  Nginx   │──>│  Express │──>│  MySQL   │ │
│  │ (React   │   │  Server  │   │   8.0    │ │
│  │  static) │   │ Port 5000│   │ Port 3306│ │
│  │ Port 80  │   └──────────┘   └──────────┘ │
│  └──────────┘                                │
│       ↑                                      │
│   Port 5173                                  │
└─────────────────────────────────────────────┘
```

### Production (VPS with PM2)

```
Internet → Nginx (reverse proxy + static files)
              ├── /            → Serves React build (dist/)
              └── /api/*       → Proxies to Express (PM2 managed)
                                      └── MySQL (local or remote)
```

---

## 14. Key Design Patterns Used

| Pattern | Where Used | Purpose |
|---|---|---|
| **MVC** | Backend structure | Separation of concerns (Routes → Controllers → Database) |
| **Service Layer** | Frontend services/ | Abstracts API calls from UI components |
| **Context Pattern** | AuthContext | Global state without prop drilling |
| **Interceptor Pattern** | Axios interceptors | Cross-cutting concerns (auth headers, error handling) |
| **Protected Route** | App.jsx | Authorization at the route level |
| **Upsert** | Budget controller | Insert or update in a single query |
| **Connection Pool** | db.js | Reuse database connections for performance |
| **Component Composition** | React components | Build complex UIs from small, reusable pieces |

---

## 15. Summary - How Everything Connects

```
USER OPENS APP
      │
      ▼
  React App loads (Vite serves it)
      │
      ▼
  AuthContext checks localStorage for token
      │
      ├── No token → Show Login page
      │                    │
      │                    ▼
      │              User submits credentials
      │                    │
      │                    ▼
      │              POST /api/auth/login
      │                    │
      │                    ▼
      │              Server validates with bcrypt
      │                    │
      │                    ▼
      │              Returns JWT token
      │                    │
      │                    ▼
      │              Token saved to localStorage
      │
      ├── Has token → Validate via GET /api/auth/profile
      │                    │
      │                    ▼
      │              Token valid? → Set user state → Show Dashboard
      │              Token invalid? → Clear token → Show Login
      │
      ▼
  USER IS LOGGED IN
      │
      ├── Dashboard: Fetches summary + budgets → Renders charts
      ├── Expenses: Fetches filtered list → CRUD operations
      ├── Budget: Set/view monthly budgets per category
      ├── Reports: Detailed analytics with charts
      └── Profile: View/edit user settings
      │
      ▼
  ALL API CALLS: Axios adds Bearer token → Express verifies → Controller queries MySQL → JSON response
```
