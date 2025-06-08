# Express.js REST API Tutorial
*Build a CRUD API in 30 minutes*

## What We're Building
A simple API to manage items with Create, Read, Update, Delete operations.

## Setup (5 minutes)

### 1. Create Project
```bash
mkdir my-api
cd my-api
npm init -y
npm install express body-parser mysql2
```

### 2. Create Database
```sql
CREATE DATABASE test_db;
USE test_db;
CREATE TABLE items (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL
);
```

## Code Tutorial

### Step 1: Basic Setup
Create `app.js`:

```javascript
const express = require("express");
const bodyParser = require("body-parser");
const mysql = require("mysql2");

const app = express();
app.use(bodyParser.json()); // Parse JSON requests

// Database connection
const db = mysql.createConnection({
  host: "localhost",
  user: "root",
  password: "",
  database: "test_db",
});

db.connect((err) => {
  if (err) throw err;
  console.log("Database connected!");
});
```

**What this does:** Sets up Express server and connects to MySQL database.

### Step 2: READ Operations (GET)

```javascript
// Get all items
app.get("/items", (req, res) => {
  db.query("SELECT * FROM items", (err, results) => {
    if (err) return res.status(500).json({ error: err });
    res.json(results);
  });
});

// Get one item by ID
app.get("/items/:id", (req, res) => {
  const id = req.params.id;
  db.query("SELECT * FROM items WHERE id = ?", [id], (err, results) => {
    if (err) return res.status(500).json({ error: err });
    if (results.length > 0) {
      res.json(results[0]);
    } else {
      res.status(404).json({ message: "Item not found" });
    }
  });
});
```

**Key concepts:**
- `req.params.id` gets ID from URL
- `?` prevents SQL injection
- Always handle errors

### Step 3: CREATE Operation (POST)

```javascript
// Create new item
app.post("/items", (req, res) => {
  const { name } = req.body;
  db.query("INSERT INTO items (name) VALUES (?)", [name], (err, result) => {
    if (err) return res.status(500).json({ error: err });
    res.status(201).json({ id: result.insertId, name });
  });
});
```

**What this does:** Takes `name` from request body, saves to database, returns new item with ID.

### Step 4: UPDATE Operation (PUT)

```javascript
// Update item
app.put("/items/:id", (req, res) => {
  const id = req.params.id;
  const { name } = req.body;
  db.query("UPDATE items SET name = ? WHERE id = ?", [name, id], (err, result) => {
    if (err) return res.status(500).json({ error: err });
    if (result.affectedRows > 0) {
      res.json({ id: parseInt(id), name });
    } else {
      res.status(404).json({ message: "Item not found" });
    }
  });
});
```

**Key concept:** `affectedRows` tells us if item was actually updated.

### Step 5: DELETE Operations

```javascript
// Delete one item
app.delete("/items/:id", (req, res) => {
  const id = req.params.id;
  db.query("DELETE FROM items WHERE id = ?", [id], (err, result) => {
    if (err) return res.status(500).json({ error: err });
    res.json({ message: "Item deleted" });
  });
});

// Delete all items
app.delete("/items", (req, res) => {
  db.query("DELETE FROM items", (err) => {
    if (err) return res.status(500).json({ error: err });
    res.json({ message: "All items deleted" });
  });
});
```

### Step 6: Start Server

```javascript
app.listen(3000, () => {
  console.log("Server running at http://localhost:3000");
});
```

## Testing Your API

### Start server:
```bash
node app.js
```

### Test with curl:

**Get all items:**
```bash
curl http://localhost:3000/items
```

**Create item:**
```bash
curl -X POST http://localhost:3000/items \
  -H "Content-Type: application/json" \
  -d '{"name": "Test Item"}'
```

**Update item:**
```bash
curl -X PUT http://localhost:3000/items/1 \
  -H "Content-Type: application/json" \
  -d '{"name": "Updated Item"}'
```

**Delete item:**
```bash
curl -X DELETE http://localhost:3000/items/1
```

## Complete Code

```javascript
const express = require("express");
const bodyParser = require("body-parser");
const mysql = require("mysql2");

const app = express();
app.use(bodyParser.json());

const db = mysql.createConnection({
  host: "localhost",
  user: "root",
  password: "",
  database: "test_db",
});

db.connect((err) => {
  if (err) throw err;
  console.log("Connected to MySQL database");
});

// GET all items
app.get("/items", (req, res) => {
  db.query("SELECT * FROM items", (err, results) => {
    if (err) return res.status(500).json({ error: err });
    res.json(results);
  });
});

// GET single item
app.get("/items/:id", (req, res) => {
  const id = req.params.id;
  db.query("SELECT * FROM items WHERE id = ?", [id], (err, results) => {
    if (err) return res.status(500).json({ error: err });
    if (results.length > 0) {
      res.json(results[0]);
    } else {
      res.status(404).json({ message: "Item not found" });
    }
  });
});

// POST new item
app.post("/items", (req, res) => {
  const { name } = req.body;
  db.query("INSERT INTO items (name) VALUES (?)", [name], (err, result) => {
    if (err) return res.status(500).json({ error: err });
    res.status(201).json({ id: result.insertId, name });
  });
});

// PUT update item
app.put("/items/:id", (req, res) => {
  const id = req.params.id;
  const { name } = req.body;
  db.query("UPDATE items SET name = ? WHERE id = ?", [name, id], (err, result) => {
    if (err) return res.status(500).json({ error: err });
    if (result.affectedRows > 0) {
      res.json({ id: parseInt(id), name });
    } else {
      res.status(404).json({ message: "Item not found" });
    }
  });
});

// DELETE single item
app.delete("/items/:id", (req, res) => {
  const id = req.params.id;
  db.query("DELETE FROM items WHERE id = ?", [id], (err, result) => {
    if (err) return res.status(500).json({ error: err });
    res.json({ message: "Item deleted" });
  });
});

// DELETE all items
app.delete("/items", (req, res) => {
  db.query("DELETE FROM items", (err) => {
    if (err) return res.status(500).json({ error: err });
    res.json({ message: "All items deleted" });
  });
});

app.listen(3000, () => {
  console.log("Server running at http://localhost:3000");
});
```

## Key Learning Points

1. **CRUD Operations**: Create (POST), Read (GET), Update (PUT), Delete (DELETE)
2. **Route Parameters**: Use `:id` to capture URL parameters
3. **Request Body**: Use `req.body` to get POST/PUT data
4. **Error Handling**: Always check for database errors
5. **Status Codes**: 200 (OK), 201 (Created), 404 (Not Found), 500 (Error)
6. **SQL Security**: Use `?` placeholders to prevent injection

## Practice Exercises

1. Add input validation for the `name` field
2. Add a `description` field to items
3. Add pagination to GET all items
4. Add search functionality

**Congratulations!** You've built a complete REST API! 🎉