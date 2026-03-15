
# Daily Expense Tracker - Complete Project Documentation

## Table of Contents

1. [Project Overview](#project-overview)
2. [Tech Stack](#tech-stack)
3. [Project Structure](#project-structure)
4. [Database Design (SQL)](#database-design-sql)
5. [Backend API (Node.js + Express)](#backend-api-nodejs--express)
6. [Frontend (React)](#frontend-react)
7. [Environment Variables](#environment-variables)
8. [Setup & Installation](#setup--installation)
9. [API Endpoints Reference](#api-endpoints-reference)
10. [Authentication Flow](#authentication-flow)
11. [Deployment](#deployment)

---

## Project Overview

**Daily Expense Tracker** is a full-stack web application that allows users to track their daily expenses, categorize spending, set budgets, and view analytics/reports. Users can sign up, log in, add/edit/delete expenses, and visualize their spending patterns over time.

### Core Features

- User registration & login (JWT authentication)
- Add, edit, delete daily expenses
- Categorize expenses (Food, Transport, Bills, Entertainment, Shopping, Health, Others)
- Set monthly budgets per category
- Dashboard with spending summary
- Filter expenses by date range, category, and amount
- Charts & reports (monthly/weekly/daily breakdown)
- Export expenses to CSV
- Responsive design (mobile-friendly)

---

## Tech Stack

| Layer      | Technology                          |
| ---------- | ----------------------------------- |
| Frontend   | React 18, React Router, Axios, Chart.js, Tailwind CSS |
| Backend    | Node.js, Express.js                 |
| Database   | MySQL                               |
| Auth       | JWT (JSON Web Tokens), bcrypt       |
| API        | RESTful API                         |
| Tools      | Vite, ESLint, Prettier, dotenv      |

---

## Project Structure

```
Daily-expensive-tracker/
├── client/                          # Frontend (React)
│   ├── public/
│   │   └── index.html
│   ├── src/
│   │   ├── assets/                  # Images, icons
│   │   ├── components/              # Reusable UI components
│   │   │   ├── Navbar.jsx
│   │   │   ├── Sidebar.jsx
│   │   │   ├── ExpenseCard.jsx
│   │   │   ├── ExpenseForm.jsx
│   │   │   ├── CategoryBadge.jsx
│   │   │   ├── BudgetProgressBar.jsx
│   │   │   ├── ChartContainer.jsx
│   │   │   ├── FilterBar.jsx
│   │   │   ├── ConfirmModal.jsx
│   │   │   └── Loader.jsx
│   │   ├── pages/
│   │   │   ├── Login.jsx
│   │   │   ├── Register.jsx
│   │   │   ├── Dashboard.jsx
│   │   │   ├── Expenses.jsx
│   │   │   ├── AddExpense.jsx
│   │   │   ├── EditExpense.jsx
│   │   │   ├── Budget.jsx
│   │   │   ├── Reports.jsx
│   │   │   └── Profile.jsx
│   │   ├── context/
│   │   │   └── AuthContext.jsx
│   │   ├── hooks/
│   │   │   ├── useAuth.js
│   │   │   └── useExpenses.js
│   │   ├── services/
│   │   │   ├── api.js               # Axios instance
│   │   │   ├── authService.js
│   │   │   ├── expenseService.js
│   │   │   └── budgetService.js
│   │   ├── utils/
│   │   │   ├── formatCurrency.js
│   │   │   ├── formatDate.js
│   │   │   └── exportCSV.js
│   │   ├── App.jsx
│   │   ├── main.jsx
│   │   └── index.css
│   ├── package.json
│   ├── tailwind.config.js
│   └── vite.config.js
│
├── server/                          # Backend (Node.js + Express)
│   ├── config/
│   │   └── db.js                    # MySQL connection pool
│   ├── controllers/
│   │   ├── authController.js
│   │   ├── expenseController.js
│   │   └── budgetController.js
│   ├── middleware/
│   │   ├── authMiddleware.js        # JWT verification
│   │   └── errorHandler.js
│   ├── models/
│   │   ├── userModel.js
│   │   ├── expenseModel.js
│   │   └── budgetModel.js
│   ├── routes/
│   │   ├── authRoutes.js
│   │   ├── expenseRoutes.js
│   │   └── budgetRoutes.js
│   ├── utils/
│   │   └── generateToken.js
│   ├── server.js                    # Entry point
│   └── package.json
│
├── database/
│   └── schema.sql                   # Full database schema
│
├── .env.example
├── .gitignore
├── Daily-Expense-Tracker.md
└── README.md
```

---

## Database Design (SQL)

### Full Schema (`database/schema.sql`)

```sql
-- ================================================
-- Daily Expense Tracker - Database Schema
-- Database: MySQL
-- ================================================

CREATE DATABASE IF NOT EXISTS daily_expense_tracker;
USE daily_expense_tracker;

-- ------------------------------------------------
-- 1. Users Table
-- ------------------------------------------------
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    avatar_url VARCHAR(500) DEFAULT NULL,
    currency VARCHAR(10) DEFAULT 'INR',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- ------------------------------------------------
-- 2. Categories Table
-- ------------------------------------------------
CREATE TABLE categories (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL UNIQUE,
    icon VARCHAR(50) DEFAULT NULL,
    color VARCHAR(7) DEFAULT '#6B7280'
);

-- Seed default categories
INSERT INTO categories (name, icon, color) VALUES
('Food', 'utensils', '#EF4444'),
('Transport', 'car', '#F59E0B'),
('Bills', 'file-text', '#3B82F6'),
('Entertainment', 'film', '#8B5CF6'),
('Shopping', 'shopping-bag', '#EC4899'),
('Health', 'heart', '#10B981'),
('Education', 'book', '#6366F1'),
('Others', 'more-horizontal', '#6B7280');

-- ------------------------------------------------
-- 3. Expenses Table
-- ------------------------------------------------
CREATE TABLE expenses (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    category_id INT NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    description VARCHAR(255),
    expense_date DATE NOT NULL,
    payment_method ENUM('cash', 'card', 'upi', 'net_banking') DEFAULT 'cash',
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (category_id) REFERENCES categories(id) ON DELETE RESTRICT
);

-- Index for faster queries
CREATE INDEX idx_expenses_user_date ON expenses(user_id, expense_date);
CREATE INDEX idx_expenses_category ON expenses(category_id);

-- ------------------------------------------------
-- 4. Budgets Table
-- ------------------------------------------------
CREATE TABLE budgets (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    category_id INT NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    month INT NOT NULL CHECK (month BETWEEN 1 AND 12),
    year INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (category_id) REFERENCES categories(id) ON DELETE RESTRICT,
    UNIQUE KEY unique_budget (user_id, category_id, month, year)
);

-- ------------------------------------------------
-- 5. Useful Views
-- ------------------------------------------------

-- Monthly spending summary per category
CREATE VIEW v_monthly_spending AS
SELECT
    e.user_id,
    c.name AS category,
    MONTH(e.expense_date) AS month,
    YEAR(e.expense_date) AS year,
    SUM(e.amount) AS total_spent,
    COUNT(*) AS transaction_count
FROM expenses e
JOIN categories c ON e.category_id = c.id
GROUP BY e.user_id, c.name, YEAR(e.expense_date), MONTH(e.expense_date);

-- Daily spending summary
CREATE VIEW v_daily_spending AS
SELECT
    e.user_id,
    e.expense_date,
    SUM(e.amount) AS total_spent,
    COUNT(*) AS transaction_count
FROM expenses e
GROUP BY e.user_id, e.expense_date;

-- Budget vs actual spending
CREATE VIEW v_budget_vs_actual AS
SELECT
    b.user_id,
    c.name AS category,
    b.month,
    b.year,
    b.amount AS budget_amount,
    COALESCE(SUM(e.amount), 0) AS actual_spent,
    (b.amount - COALESCE(SUM(e.amount), 0)) AS remaining
FROM budgets b
JOIN categories c ON b.category_id = c.id
LEFT JOIN expenses e ON e.user_id = b.user_id
    AND e.category_id = b.category_id
    AND MONTH(e.expense_date) = b.month
    AND YEAR(e.expense_date) = b.year
GROUP BY b.id, c.name, b.month, b.year, b.amount, b.user_id;
```

### ER Diagram (Text)

```
┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│    users     │       │  categories  │       │   budgets    │
├──────────────┤       ├──────────────┤       ├──────────────┤
│ id (PK)      │──┐    │ id (PK)      │──┐    │ id (PK)      │
│ name         │  │    │ name         │  │    │ user_id (FK) │
│ email        │  │    │ icon         │  │    │ category_id  │
│ password     │  │    │ color        │  │    │ amount       │
│ avatar_url   │  │    └──────────────┘  │    │ month        │
│ currency     │  │                      │    │ year         │
│ created_at   │  │    ┌──────────────┐  │    └──────────────┘
│ updated_at   │  │    │   expenses   │  │
└──────────────┘  │    ├──────────────┤  │
                  ├───>│ user_id (FK) │  │
                  │    │ category_id  │<─┘
                  │    │ amount       │
                  │    │ description  │
                  │    │ expense_date │
                  │    │ payment_method│
                  │    │ notes        │
                  │    └──────────────┘
```

---

## Backend API (Node.js + Express)

### server/config/db.js — MySQL Connection

```javascript
const mysql = require('mysql2/promise');
require('dotenv').config();

const pool = mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  waitForConnections: true,
  connectionLimit: 10,
});

module.exports = pool;
```

### server/server.js — Entry Point

```javascript
const express = require('express');
const cors = require('cors');
require('dotenv').config();

const authRoutes = require('./routes/authRoutes');
const expenseRoutes = require('./routes/expenseRoutes');
const budgetRoutes = require('./routes/budgetRoutes');
const errorHandler = require('./middleware/errorHandler');

const app = express();
const PORT = process.env.PORT || 5000;

// Middleware
app.use(cors({ origin: process.env.CLIENT_URL }));
app.use(express.json());

// Routes
app.use('/api/auth', authRoutes);
app.use('/api/expenses', expenseRoutes);
app.use('/api/budgets', budgetRoutes);

// Health check
app.get('/api/health', (req, res) => {
  res.json({ status: 'ok' });
});

// Error handler
app.use(errorHandler);

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### server/middleware/authMiddleware.js — JWT Auth

```javascript
const jwt = require('jsonwebtoken');

const protect = (req, res, next) => {
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ message: 'Not authorized, no token' });
  }

  try {
    const token = authHeader.split(' ')[1];
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.userId = decoded.id;
    next();
  } catch (error) {
    return res.status(401).json({ message: 'Not authorized, token invalid' });
  }
};

module.exports = protect;
```

### server/middleware/errorHandler.js

```javascript
const errorHandler = (err, req, res, next) => {
  const statusCode = res.statusCode === 200 ? 500 : res.statusCode;
  res.status(statusCode).json({
    message: err.message,
    stack: process.env.NODE_ENV === 'production' ? null : err.stack,
  });
};

module.exports = errorHandler;
```

### server/utils/generateToken.js

```javascript
const jwt = require('jsonwebtoken');

const generateToken = (id) => {
  return jwt.sign({ id }, process.env.JWT_SECRET, { expiresIn: '30d' });
};

module.exports = generateToken;
```

### server/controllers/authController.js

```javascript
const bcrypt = require('bcryptjs');
const pool = require('../config/db');
const generateToken = require('../utils/generateToken');

// POST /api/auth/register
const register = async (req, res, next) => {
  try {
    const { name, email, password } = req.body;

    if (!name || !email || !password) {
      res.status(400);
      throw new Error('All fields are required');
    }

    const [existing] = await pool.query('SELECT id FROM users WHERE email = ?', [email]);
    if (existing.length > 0) {
      res.status(400);
      throw new Error('Email already registered');
    }

    const salt = await bcrypt.genSalt(10);
    const hashedPassword = await bcrypt.hash(password, salt);

    const [result] = await pool.query(
      'INSERT INTO users (name, email, password) VALUES (?, ?, ?)',
      [name, email, hashedPassword]
    );

    res.status(201).json({
      id: result.insertId,
      name,
      email,
      token: generateToken(result.insertId),
    });
  } catch (error) {
    next(error);
  }
};

// POST /api/auth/login
const login = async (req, res, next) => {
  try {
    const { email, password } = req.body;

    const [rows] = await pool.query('SELECT * FROM users WHERE email = ?', [email]);
    if (rows.length === 0) {
      res.status(401);
      throw new Error('Invalid email or password');
    }

    const user = rows[0];
    const isMatch = await bcrypt.compare(password, user.password);
    if (!isMatch) {
      res.status(401);
      throw new Error('Invalid email or password');
    }

    res.json({
      id: user.id,
      name: user.name,
      email: user.email,
      token: generateToken(user.id),
    });
  } catch (error) {
    next(error);
  }
};

// GET /api/auth/profile
const getProfile = async (req, res, next) => {
  try {
    const [rows] = await pool.query(
      'SELECT id, name, email, avatar_url, currency, created_at FROM users WHERE id = ?',
      [req.userId]
    );

    if (rows.length === 0) {
      res.status(404);
      throw new Error('User not found');
    }

    res.json(rows[0]);
  } catch (error) {
    next(error);
  }
};

// PUT /api/auth/profile
const updateProfile = async (req, res, next) => {
  try {
    const { name, currency } = req.body;
    await pool.query('UPDATE users SET name = ?, currency = ? WHERE id = ?', [
      name,
      currency,
      req.userId,
    ]);
    res.json({ message: 'Profile updated' });
  } catch (error) {
    next(error);
  }
};

module.exports = { register, login, getProfile, updateProfile };
```

### server/controllers/expenseController.js

```javascript
const pool = require('../config/db');

// GET /api/expenses
const getExpenses = async (req, res, next) => {
  try {
    const { category, start_date, end_date, payment_method, sort, page = 1, limit = 20 } = req.query;
    const offset = (page - 1) * limit;

    let query = `
      SELECT e.*, c.name AS category_name, c.icon, c.color
      FROM expenses e
      JOIN categories c ON e.category_id = c.id
      WHERE e.user_id = ?
    `;
    const params = [req.userId];

    if (category) {
      query += ' AND e.category_id = ?';
      params.push(category);
    }
    if (start_date) {
      query += ' AND e.expense_date >= ?';
      params.push(start_date);
    }
    if (end_date) {
      query += ' AND e.expense_date <= ?';
      params.push(end_date);
    }
    if (payment_method) {
      query += ' AND e.payment_method = ?';
      params.push(payment_method);
    }

    query += ` ORDER BY e.expense_date ${sort === 'asc' ? 'ASC' : 'DESC'}`;
    query += ' LIMIT ? OFFSET ?';
    params.push(Number(limit), Number(offset));

    const [expenses] = await pool.query(query, params);

    // Total count for pagination
    let countQuery = 'SELECT COUNT(*) AS total FROM expenses WHERE user_id = ?';
    const countParams = [req.userId];
    const [countResult] = await pool.query(countQuery, countParams);

    res.json({
      expenses,
      total: countResult[0].total,
      page: Number(page),
      totalPages: Math.ceil(countResult[0].total / limit),
    });
  } catch (error) {
    next(error);
  }
};

// POST /api/expenses
const addExpense = async (req, res, next) => {
  try {
    const { category_id, amount, description, expense_date, payment_method, notes } = req.body;

    if (!category_id || !amount || !expense_date) {
      res.status(400);
      throw new Error('Category, amount, and date are required');
    }

    const [result] = await pool.query(
      `INSERT INTO expenses (user_id, category_id, amount, description, expense_date, payment_method, notes)
       VALUES (?, ?, ?, ?, ?, ?, ?)`,
      [req.userId, category_id, amount, description, expense_date, payment_method || 'cash', notes]
    );

    res.status(201).json({ id: result.insertId, message: 'Expense added' });
  } catch (error) {
    next(error);
  }
};

// PUT /api/expenses/:id
const updateExpense = async (req, res, next) => {
  try {
    const { id } = req.params;
    const { category_id, amount, description, expense_date, payment_method, notes } = req.body;

    const [existing] = await pool.query('SELECT id FROM expenses WHERE id = ? AND user_id = ?', [
      id,
      req.userId,
    ]);
    if (existing.length === 0) {
      res.status(404);
      throw new Error('Expense not found');
    }

    await pool.query(
      `UPDATE expenses SET category_id = ?, amount = ?, description = ?, expense_date = ?, payment_method = ?, notes = ?
       WHERE id = ? AND user_id = ?`,
      [category_id, amount, description, expense_date, payment_method, notes, id, req.userId]
    );

    res.json({ message: 'Expense updated' });
  } catch (error) {
    next(error);
  }
};

// DELETE /api/expenses/:id
const deleteExpense = async (req, res, next) => {
  try {
    const { id } = req.params;

    const [existing] = await pool.query('SELECT id FROM expenses WHERE id = ? AND user_id = ?', [
      id,
      req.userId,
    ]);
    if (existing.length === 0) {
      res.status(404);
      throw new Error('Expense not found');
    }

    await pool.query('DELETE FROM expenses WHERE id = ? AND user_id = ?', [id, req.userId]);
    res.json({ message: 'Expense deleted' });
  } catch (error) {
    next(error);
  }
};

// GET /api/expenses/summary
const getSummary = async (req, res, next) => {
  try {
    const { month, year } = req.query;
    const m = month || new Date().getMonth() + 1;
    const y = year || new Date().getFullYear();

    const [totalResult] = await pool.query(
      `SELECT COALESCE(SUM(amount), 0) AS total_spent
       FROM expenses WHERE user_id = ? AND MONTH(expense_date) = ? AND YEAR(expense_date) = ?`,
      [req.userId, m, y]
    );

    const [categoryBreakdown] = await pool.query(
      `SELECT c.name, c.color, COALESCE(SUM(e.amount), 0) AS total, COUNT(*) AS count
       FROM expenses e
       JOIN categories c ON e.category_id = c.id
       WHERE e.user_id = ? AND MONTH(e.expense_date) = ? AND YEAR(e.expense_date) = ?
       GROUP BY c.id ORDER BY total DESC`,
      [req.userId, m, y]
    );

    const [dailyTrend] = await pool.query(
      `SELECT expense_date, SUM(amount) AS daily_total
       FROM expenses
       WHERE user_id = ? AND MONTH(expense_date) = ? AND YEAR(expense_date) = ?
       GROUP BY expense_date ORDER BY expense_date`,
      [req.userId, m, y]
    );

    res.json({
      total_spent: totalResult[0].total_spent,
      category_breakdown: categoryBreakdown,
      daily_trend: dailyTrend,
    });
  } catch (error) {
    next(error);
  }
};

// GET /api/expenses/categories
const getCategories = async (req, res, next) => {
  try {
    const [categories] = await pool.query('SELECT * FROM categories ORDER BY name');
    res.json(categories);
  } catch (error) {
    next(error);
  }
};

module.exports = { getExpenses, addExpense, updateExpense, deleteExpense, getSummary, getCategories };
```

### server/controllers/budgetController.js

```javascript
const pool = require('../config/db');

// GET /api/budgets?month=3&year=2026
const getBudgets = async (req, res, next) => {
  try {
    const { month, year } = req.query;
    const m = month || new Date().getMonth() + 1;
    const y = year || new Date().getFullYear();

    const [budgets] = await pool.query(
      `SELECT b.*, c.name AS category_name, c.color,
              COALESCE((SELECT SUM(e.amount) FROM expenses e
                WHERE e.user_id = b.user_id AND e.category_id = b.category_id
                AND MONTH(e.expense_date) = b.month AND YEAR(e.expense_date) = b.year
              ), 0) AS spent
       FROM budgets b
       JOIN categories c ON b.category_id = c.id
       WHERE b.user_id = ? AND b.month = ? AND b.year = ?`,
      [req.userId, m, y]
    );

    res.json(budgets);
  } catch (error) {
    next(error);
  }
};

// POST /api/budgets
const setBudget = async (req, res, next) => {
  try {
    const { category_id, amount, month, year } = req.body;

    if (!category_id || !amount || !month || !year) {
      res.status(400);
      throw new Error('All fields are required');
    }

    await pool.query(
      `INSERT INTO budgets (user_id, category_id, amount, month, year)
       VALUES (?, ?, ?, ?, ?)
       ON DUPLICATE KEY UPDATE amount = VALUES(amount)`,
      [req.userId, category_id, amount, month, year]
    );

    res.status(201).json({ message: 'Budget set successfully' });
  } catch (error) {
    next(error);
  }
};

// DELETE /api/budgets/:id
const deleteBudget = async (req, res, next) => {
  try {
    const { id } = req.params;
    await pool.query('DELETE FROM budgets WHERE id = ? AND user_id = ?', [id, req.userId]);
    res.json({ message: 'Budget deleted' });
  } catch (error) {
    next(error);
  }
};

module.exports = { getBudgets, setBudget, deleteBudget };
```

### server/routes/authRoutes.js

```javascript
const express = require('express');
const router = express.Router();
const { register, login, getProfile, updateProfile } = require('../controllers/authController');
const protect = require('../middleware/authMiddleware');

router.post('/register', register);
router.post('/login', login);
router.get('/profile', protect, getProfile);
router.put('/profile', protect, updateProfile);

module.exports = router;
```

### server/routes/expenseRoutes.js

```javascript
const express = require('express');
const router = express.Router();
const {
  getExpenses, addExpense, updateExpense, deleteExpense, getSummary, getCategories,
} = require('../controllers/expenseController');
const protect = require('../middleware/authMiddleware');

router.use(protect); // All routes require auth

router.get('/summary', getSummary);
router.get('/categories', getCategories);
router.route('/').get(getExpenses).post(addExpense);
router.route('/:id').put(updateExpense).delete(deleteExpense);

module.exports = router;
```

### server/routes/budgetRoutes.js

```javascript
const express = require('express');
const router = express.Router();
const { getBudgets, setBudget, deleteBudget } = require('../controllers/budgetController');
const protect = require('../middleware/authMiddleware');

router.use(protect);

router.route('/').get(getBudgets).post(setBudget);
router.route('/:id').delete(deleteBudget);

module.exports = router;
```

### server/package.json

```json
{
  "name": "daily-expense-tracker-server",
  "version": "1.0.0",
  "description": "Backend API for Daily Expense Tracker",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "dependencies": {
    "bcryptjs": "^2.4.3",
    "cors": "^2.8.5",
    "dotenv": "^16.4.5",
    "express": "^4.21.0",
    "jsonwebtoken": "^9.0.2",
    "mysql2": "^3.11.0"
  },
  "devDependencies": {
    "nodemon": "^3.1.4"
  }
}
```

---

## Frontend (React)

### client/src/services/api.js — Axios Instance

```javascript
import axios from 'axios';

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL || 'http://localhost:5000/api',
});

api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default api;
```

### client/src/services/authService.js

```javascript
import api from './api';

export const loginUser = (data) => api.post('/auth/login', data);
export const registerUser = (data) => api.post('/auth/register', data);
export const getProfile = () => api.get('/auth/profile');
export const updateProfile = (data) => api.put('/auth/profile', data);
```

### client/src/services/expenseService.js

```javascript
import api from './api';

export const getExpenses = (params) => api.get('/expenses', { params });
export const addExpense = (data) => api.post('/expenses', data);
export const updateExpense = (id, data) => api.put(`/expenses/${id}`, data);
export const deleteExpense = (id) => api.delete(`/expenses/${id}`);
export const getSummary = (params) => api.get('/expenses/summary', { params });
export const getCategories = () => api.get('/expenses/categories');
```

### client/src/services/budgetService.js

```javascript
import api from './api';

export const getBudgets = (params) => api.get('/budgets', { params });
export const setBudget = (data) => api.post('/budgets', data);
export const deleteBudget = (id) => api.delete(`/budgets/${id}`);
```

### client/src/context/AuthContext.jsx

```jsx
import { createContext, useState, useEffect } from 'react';
import { getProfile } from '../services/authService';

export const AuthContext = createContext();

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const token = localStorage.getItem('token');
    if (token) {
      getProfile()
        .then((res) => setUser(res.data))
        .catch(() => localStorage.removeItem('token'))
        .finally(() => setLoading(false));
    } else {
      setLoading(false);
    }
  }, []);

  const login = (userData, token) => {
    localStorage.setItem('token', token);
    setUser(userData);
  };

  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, loading, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};
```

### client/src/App.jsx

```jsx
import { BrowserRouter as Router, Routes, Route, Navigate } from 'react-router-dom';
import { AuthProvider } from './context/AuthContext';
import useAuth from './hooks/useAuth';
import Navbar from './components/Navbar';
import Sidebar from './components/Sidebar';
import Login from './pages/Login';
import Register from './pages/Register';
import Dashboard from './pages/Dashboard';
import Expenses from './pages/Expenses';
import AddExpense from './pages/AddExpense';
import EditExpense from './pages/EditExpense';
import Budget from './pages/Budget';
import Reports from './pages/Reports';
import Profile from './pages/Profile';

const ProtectedRoute = ({ children }) => {
  const { user, loading } = useAuth();
  if (loading) return <div className="flex justify-center items-center h-screen">Loading...</div>;
  return user ? children : <Navigate to="/login" />;
};

const AppRoutes = () => (
  <Routes>
    <Route path="/login" element={<Login />} />
    <Route path="/register" element={<Register />} />
    <Route path="/" element={<ProtectedRoute><Dashboard /></ProtectedRoute>} />
    <Route path="/expenses" element={<ProtectedRoute><Expenses /></ProtectedRoute>} />
    <Route path="/expenses/add" element={<ProtectedRoute><AddExpense /></ProtectedRoute>} />
    <Route path="/expenses/edit/:id" element={<ProtectedRoute><EditExpense /></ProtectedRoute>} />
    <Route path="/budget" element={<ProtectedRoute><Budget /></ProtectedRoute>} />
    <Route path="/reports" element={<ProtectedRoute><Reports /></ProtectedRoute>} />
    <Route path="/profile" element={<ProtectedRoute><Profile /></ProtectedRoute>} />
  </Routes>
);

const App = () => (
  <AuthProvider>
    <Router>
      <div className="flex h-screen bg-gray-50">
        <Sidebar />
        <div className="flex-1 flex flex-col overflow-hidden">
          <Navbar />
          <main className="flex-1 overflow-y-auto p-6">
            <AppRoutes />
          </main>
        </div>
      </div>
    </Router>
  </AuthProvider>
);

export default App;
```

### client/src/pages/Dashboard.jsx (Key Page Example)

```jsx
import { useState, useEffect } from 'react';
import { getSummary } from '../services/expenseService';
import { getBudgets } from '../services/budgetService';
import ChartContainer from '../components/ChartContainer';
import BudgetProgressBar from '../components/BudgetProgressBar';

const Dashboard = () => {
  const [summary, setSummary] = useState(null);
  const [budgets, setBudgets] = useState([]);
  const [month, setMonth] = useState(new Date().getMonth() + 1);
  const [year, setYear] = useState(new Date().getFullYear());

  useEffect(() => {
    const fetchData = async () => {
      const [summaryRes, budgetRes] = await Promise.all([
        getSummary({ month, year }),
        getBudgets({ month, year }),
      ]);
      setSummary(summaryRes.data);
      setBudgets(budgetRes.data);
    };
    fetchData();
  }, [month, year]);

  if (!summary) return <div>Loading...</div>;

  return (
    <div className="space-y-6">
      <h1 className="text-2xl font-bold text-gray-800">Dashboard</h1>

      {/* Total Spent Card */}
      <div className="bg-white rounded-xl shadow p-6">
        <p className="text-gray-500 text-sm">Total Spent This Month</p>
        <p className="text-3xl font-bold text-red-500">
          ₹{Number(summary.total_spent).toLocaleString()}
        </p>
      </div>

      {/* Category Breakdown Chart */}
      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <div className="bg-white rounded-xl shadow p-6">
          <h2 className="text-lg font-semibold mb-4">Spending by Category</h2>
          <ChartContainer
            type="doughnut"
            labels={summary.category_breakdown.map((c) => c.name)}
            data={summary.category_breakdown.map((c) => c.total)}
            colors={summary.category_breakdown.map((c) => c.color)}
          />
        </div>

        <div className="bg-white rounded-xl shadow p-6">
          <h2 className="text-lg font-semibold mb-4">Daily Spending Trend</h2>
          <ChartContainer
            type="line"
            labels={summary.daily_trend.map((d) => d.expense_date)}
            data={summary.daily_trend.map((d) => d.daily_total)}
          />
        </div>
      </div>

      {/* Budget Progress */}
      <div className="bg-white rounded-xl shadow p-6">
        <h2 className="text-lg font-semibold mb-4">Budget Status</h2>
        {budgets.length === 0 ? (
          <p className="text-gray-400">No budgets set for this month.</p>
        ) : (
          budgets.map((b) => (
            <BudgetProgressBar
              key={b.id}
              category={b.category_name}
              color={b.color}
              budget={b.amount}
              spent={b.spent}
            />
          ))
        )}
      </div>
    </div>
  );
};

export default Dashboard;
```

### client/src/pages/Login.jsx

```jsx
import { useState } from 'react';
import { useNavigate, Link } from 'react-router-dom';
import useAuth from '../hooks/useAuth';
import { loginUser } from '../services/authService';

const Login = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const { login } = useAuth();
  const navigate = useNavigate();

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError('');
    try {
      const res = await loginUser({ email, password });
      login(res.data, res.data.token);
      navigate('/');
    } catch (err) {
      setError(err.response?.data?.message || 'Login failed');
    }
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50">
      <div className="bg-white p-8 rounded-xl shadow-md w-full max-w-md">
        <h2 className="text-2xl font-bold text-center mb-6">Login to Expense Tracker</h2>
        {error && <p className="text-red-500 text-sm mb-4">{error}</p>}
        <form onSubmit={handleSubmit} className="space-y-4">
          <input
            type="email" placeholder="Email" value={email}
            onChange={(e) => setEmail(e.target.value)}
            className="w-full border rounded-lg px-4 py-2" required
          />
          <input
            type="password" placeholder="Password" value={password}
            onChange={(e) => setPassword(e.target.value)}
            className="w-full border rounded-lg px-4 py-2" required
          />
          <button type="submit" className="w-full bg-blue-600 text-white py-2 rounded-lg hover:bg-blue-700">
            Login
          </button>
        </form>
        <p className="text-center text-sm mt-4">
          Don't have an account? <Link to="/register" className="text-blue-600">Register</Link>
        </p>
      </div>
    </div>
  );
};

export default Login;
```

### client/src/components/ExpenseForm.jsx

```jsx
import { useState, useEffect } from 'react';
import { getCategories } from '../services/expenseService';

const ExpenseForm = ({ initialData = {}, onSubmit, buttonText = 'Save' }) => {
  const [categories, setCategories] = useState([]);
  const [form, setForm] = useState({
    category_id: initialData.category_id || '',
    amount: initialData.amount || '',
    description: initialData.description || '',
    expense_date: initialData.expense_date || new Date().toISOString().split('T')[0],
    payment_method: initialData.payment_method || 'cash',
    notes: initialData.notes || '',
  });

  useEffect(() => {
    getCategories().then((res) => setCategories(res.data));
  }, []);

  const handleChange = (e) => {
    setForm({ ...form, [e.target.name]: e.target.value });
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit(form);
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4 max-w-lg">
      <select name="category_id" value={form.category_id} onChange={handleChange}
        className="w-full border rounded-lg px-4 py-2" required>
        <option value="">Select Category</option>
        {categories.map((c) => (
          <option key={c.id} value={c.id}>{c.name}</option>
        ))}
      </select>

      <input type="number" name="amount" placeholder="Amount (₹)" step="0.01"
        value={form.amount} onChange={handleChange}
        className="w-full border rounded-lg px-4 py-2" required />

      <input type="text" name="description" placeholder="Description"
        value={form.description} onChange={handleChange}
        className="w-full border rounded-lg px-4 py-2" />

      <input type="date" name="expense_date"
        value={form.expense_date} onChange={handleChange}
        className="w-full border rounded-lg px-4 py-2" required />

      <select name="payment_method" value={form.payment_method} onChange={handleChange}
        className="w-full border rounded-lg px-4 py-2">
        <option value="cash">Cash</option>
        <option value="card">Card</option>
        <option value="upi">UPI</option>
        <option value="net_banking">Net Banking</option>
      </select>

      <textarea name="notes" placeholder="Notes (optional)"
        value={form.notes} onChange={handleChange}
        className="w-full border rounded-lg px-4 py-2" rows={3} />

      <button type="submit" className="w-full bg-green-600 text-white py-2 rounded-lg hover:bg-green-700">
        {buttonText}
      </button>
    </form>
  );
};

export default ExpenseForm;
```

### client/src/components/BudgetProgressBar.jsx

```jsx
const BudgetProgressBar = ({ category, color, budget, spent }) => {
  const percentage = Math.min((spent / budget) * 100, 100);
  const isOver = spent > budget;

  return (
    <div className="mb-4">
      <div className="flex justify-between text-sm mb-1">
        <span className="font-medium">{category}</span>
        <span className={isOver ? 'text-red-500 font-bold' : 'text-gray-500'}>
          ₹{Number(spent).toLocaleString()} / ₹{Number(budget).toLocaleString()}
        </span>
      </div>
      <div className="w-full bg-gray-200 rounded-full h-3">
        <div
          className="h-3 rounded-full transition-all duration-300"
          style={{
            width: `${percentage}%`,
            backgroundColor: isOver ? '#EF4444' : color,
          }}
        />
      </div>
    </div>
  );
};

export default BudgetProgressBar;
```

### client/src/utils/exportCSV.js

```javascript
export const exportToCSV = (expenses, filename = 'expenses.csv') => {
  const headers = ['Date', 'Category', 'Description', 'Amount', 'Payment Method', 'Notes'];
  const rows = expenses.map((e) => [
    e.expense_date,
    e.category_name,
    e.description,
    e.amount,
    e.payment_method,
    e.notes || '',
  ]);

  const csv = [headers, ...rows].map((row) => row.map((v) => `"${v}"`).join(',')).join('\n');
  const blob = new Blob([csv], { type: 'text/csv' });
  const url = URL.createObjectURL(blob);

  const a = document.createElement('a');
  a.href = url;
  a.download = filename;
  a.click();
  URL.revokeObjectURL(url);
};
```

### client/package.json

```json
{
  "name": "daily-expense-tracker-client",
  "version": "1.0.0",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "axios": "^1.7.7",
    "chart.js": "^4.4.4",
    "react": "^18.3.1",
    "react-chartjs-2": "^5.2.0",
    "react-dom": "^18.3.1",
    "react-router-dom": "^6.26.2"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.3.1",
    "autoprefixer": "^10.4.20",
    "postcss": "^8.4.45",
    "tailwindcss": "^3.4.10",
    "vite": "^5.4.4"
  }
}
```

---

## Environment Variables

### .env.example

```env
# Server
PORT=5000
NODE_ENV=development
CLIENT_URL=http://localhost:5173

# Database
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=your_mysql_password
DB_NAME=daily_expense_tracker

# JWT
JWT_SECRET=your_super_secret_jwt_key_change_this

# Client (create in client/.env)
# VITE_API_URL=http://localhost:5000/api
```

---

## Setup & Installation

### Prerequisites

- Node.js v18+
- MySQL 8.0+
- npm or yarn

### Step 1: Clone the repository

```bash
git clone https://github.com/samseni/Daily-expensive-tracker.git
cd Daily-expensive-tracker
```

### Step 2: Setup the database

```bash
mysql -u root -p < database/schema.sql
```

### Step 3: Setup the backend

```bash
cd server
cp ../.env.example .env    # Edit .env with your DB credentials
npm install
npm run dev                # Starts on http://localhost:5000
```

### Step 4: Setup the frontend

```bash
cd client
npm install
npm run dev                # Starts on http://localhost:5173
```

### Step 5: Open in browser

Visit `http://localhost:5173`, register a new account, and start tracking expenses.

---

## API Endpoints Reference

| Method   | Endpoint              | Auth | Description                      |
| -------- | --------------------- | ---- | -------------------------------- |
| POST     | /api/auth/register    | No   | Register new user                |
| POST     | /api/auth/login       | No   | Login and get JWT token          |
| GET      | /api/auth/profile     | Yes  | Get current user profile         |
| PUT      | /api/auth/profile     | Yes  | Update user profile              |
| GET      | /api/expenses         | Yes  | List expenses (with filters)     |
| POST     | /api/expenses         | Yes  | Add a new expense                |
| PUT      | /api/expenses/:id     | Yes  | Update an expense                |
| DELETE   | /api/expenses/:id     | Yes  | Delete an expense                |
| GET      | /api/expenses/summary | Yes  | Get monthly spending summary     |
| GET      | /api/expenses/categories | Yes | Get all categories             |
| GET      | /api/budgets          | Yes  | Get budgets for a month          |
| POST     | /api/budgets          | Yes  | Set/update a budget              |
| DELETE   | /api/budgets/:id      | Yes  | Delete a budget                  |

### Query Parameters for GET /api/expenses

| Param           | Type   | Description                     |
| --------------- | ------ | ------------------------------- |
| category        | number | Filter by category ID           |
| start_date      | string | Filter from date (YYYY-MM-DD)   |
| end_date        | string | Filter to date (YYYY-MM-DD)     |
| payment_method  | string | cash, card, upi, net_banking    |
| sort            | string | asc or desc (by date)           |
| page            | number | Page number (default: 1)        |
| limit           | number | Items per page (default: 20)    |

---

## Authentication Flow

```
1. User registers → POST /api/auth/register
   ↓ returns { id, name, email, token }

2. User logs in → POST /api/auth/login
   ↓ returns { id, name, email, token }

3. Token stored in localStorage

4. Every API request includes header:
   Authorization: Bearer <token>

5. authMiddleware.js verifies token on protected routes

6. On 401 response → auto redirect to /login
```

---

## Deployment

### Option 1: Manual VPS Deployment

```bash
# Backend
cd server
npm install --production
NODE_ENV=production node server.js   # Use PM2 for process management

# Frontend
cd client
npm run build                         # Creates dist/ folder
# Serve dist/ with Nginx or any static file server
```

### Option 2: Docker (docker-compose.yml)

```yaml
version: '3.8'
services:
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: daily_expense_tracker
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql
      - ./database/schema.sql:/docker-entrypoint-initdb.d/schema.sql

  server:
    build: ./server
    ports:
      - "5000:5000"
    environment:
      DB_HOST: db
      DB_USER: root
      DB_PASSWORD: rootpassword
      DB_NAME: daily_expense_tracker
      JWT_SECRET: your_jwt_secret
      CLIENT_URL: http://localhost:5173
    depends_on:
      - db

  client:
    build: ./client
    ports:
      - "5173:80"
    depends_on:
      - server

volumes:
  db_data:
```

---

## Summary

| Component | Details |
|-----------|---------|
| **Pages** | Login, Register, Dashboard, Expenses, Add/Edit Expense, Budget, Reports, Profile |
| **API Routes** | 13 endpoints (Auth: 4, Expenses: 6, Budgets: 3) |
| **DB Tables** | 4 tables (users, categories, expenses, budgets) + 3 views |
| **Auth** | JWT-based with bcrypt password hashing |
| **Charts** | Doughnut (category breakdown), Line (daily trend) |
| **Features** | CRUD expenses, budgets, filters, pagination, CSV export |
