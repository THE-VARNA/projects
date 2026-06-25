# Simple Todo MERN App - Complete Full-Stack Guide

Project Name: Simple Todo MERN App  
Project Type: Full-stack todo/task manager  
Skill Level: Beginner  

Tech Stack:

- Frontend: React, Vite, React Router, Axios, Tailwind CSS
- Backend: Node.js, Express.js, JavaScript
- Database: MongoDB with Mongoose
- Authentication: JWT login/signup with bcrypt password hashing
- Deployment: Vercel for frontend, Render/Railway for backend, MongoDB Atlas for database

---

## 1. Project Overview

### What The Application Does

We will build a simple todo application where users can:

- Sign up
- Log in
- Create todos
- View their own todos
- Mark todos as completed or not completed
- Edit todos
- Delete todos
- Log out

Each user will only see their own todos.

### Main Features

- User authentication
- Protected todo APIs
- MongoDB database storage
- React frontend
- Form handling
- Loading and error states
- CRUD operations

CRUD means:

- Create
- Read
- Update
- Delete

### Key Concepts Learned

You will learn:

- How frontend, backend, and database work together
- How Express creates APIs
- How MongoDB stores documents
- How Mongoose models work
- How JWT authentication works
- How React calls backend APIs
- How protected routes work
- How to handle validation and errors
- How to deploy a MERN app

### High-Level Architecture

```text
Browser / React frontend
        |
        | Axios HTTP requests
        v
Node.js / Express backend
        |
        | Mongoose queries
        v
MongoDB database
```

Authentication flow:

```text
User logs in
Backend returns JWT token
Frontend stores token
Frontend sends token with todo requests
Backend verifies token
Backend returns only that user's todos
```

---

## 2. Folder Structure

```text
simple-todo-mern/
|-- backend/
|   |-- src/
|   |   |-- config/
|   |   |   |-- db.js
|   |   |-- controllers/
|   |   |   |-- authController.js
|   |   |   |-- todoController.js
|   |   |-- middleware/
|   |   |   |-- authMiddleware.js
|   |   |   |-- errorMiddleware.js
|   |   |   |-- validateMiddleware.js
|   |   |-- models/
|   |   |   |-- User.js
|   |   |   |-- Todo.js
|   |   |-- routes/
|   |   |   |-- authRoutes.js
|   |   |   |-- todoRoutes.js
|   |   |-- schemas/
|   |   |   |-- authSchemas.js
|   |   |   |-- todoSchemas.js
|   |   |-- utils/
|   |   |   |-- generateToken.js
|   |   |-- server.js
|   |-- .env
|   |-- package.json
|
|-- frontend/
|   |-- src/
|   |   |-- api/
|   |   |   |-- axios.js
|   |   |-- components/
|   |   |   |-- Navbar.jsx
|   |   |   |-- ProtectedRoute.jsx
|   |   |   |-- TodoForm.jsx
|   |   |   |-- TodoItem.jsx
|   |   |-- pages/
|   |   |   |-- Login.jsx
|   |   |   |-- Signup.jsx
|   |   |   |-- Todos.jsx
|   |   |-- App.jsx
|   |   |-- main.jsx
|   |   |-- index.css
|   |-- package.json
```

### Purpose Of Each Folder

- `backend/src/config`: database connection setup.
- `backend/src/controllers`: functions that handle API logic.
- `backend/src/middleware`: reusable request helpers like auth and validation.
- `backend/src/models`: MongoDB data models.
- `backend/src/routes`: API URL definitions.
- `backend/src/schemas`: validation rules.
- `backend/src/utils`: helper functions.
- `frontend/src/api`: Axios API setup.
- `frontend/src/components`: reusable React UI components.
- `frontend/src/pages`: full page screens.

---

## 3. Setup & Installation

### Prerequisites

Install:

- Node.js
- npm
- VS Code
- MongoDB Atlas account
- Postman or Thunder Client

Check installation:

```bash
node -v
npm -v
```

Create project:

```bash
mkdir simple-todo-mern
cd simple-todo-mern
mkdir backend frontend
```

---

### Backend Setup

Go into backend:

```bash
cd backend
npm init -y
```

Install backend packages:

```bash
npm install express mongoose cors dotenv bcryptjs jsonwebtoken zod
npm install -D nodemon
```

What these packages do:

- `express`: creates the backend server and routes.
- `mongoose`: connects Node.js to MongoDB.
- `cors`: allows frontend to call backend.
- `dotenv`: reads `.env` variables.
- `bcryptjs`: hashes passwords.
- `jsonwebtoken`: creates and verifies JWT tokens.
- `zod`: validates request bodies.
- `nodemon`: restarts backend when files change.

Update `backend/package.json`:

```json
{
  "name": "simple-todo-backend",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "nodemon src/server.js",
    "start": "node src/server.js"
  }
}
```

Why `"type": "module"` is needed:

It lets us use modern JavaScript imports:

```js
import express from "express";
```

---

### Backend Environment Variables

Create `backend/.env`:

```env
PORT=5000
MONGO_URI=your_mongodb_atlas_connection_string
JWT_SECRET=replace_this_with_a_long_secret
CLIENT_URL=http://localhost:5173
```

What each variable does:

- `PORT`: backend runs on this port.
- `MONGO_URI`: MongoDB connection string.
- `JWT_SECRET`: secret key for signing tokens.
- `CLIENT_URL`: frontend URL allowed by CORS.

Important:

Do not upload `.env` to GitHub.

---

### Frontend Setup

Open a second terminal:

```bash
cd simple-todo-mern/frontend
npm create vite@latest . -- --template react
npm install
```

Install frontend packages:

```bash
npm install axios react-router-dom
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

Update `frontend/tailwind.config.js`:

```js
export default {
  content: ["./index.html", "./src/**/*.{js,jsx}"],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

Update `frontend/src/index.css`:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

body {
  margin: 0;
  background: #f8fafc;
  color: #0f172a;
  font-family: system-ui, sans-serif;
}
```

---

## 4. Database Design

We need two MongoDB collections:

- `users`
- `todos`

MongoDB stores data as documents.

Example user document:

```json
{
  "_id": "user_id",
  "name": "Alex",
  "email": "alex@example.com",
  "password": "hashed_password"
}
```

Example todo document:

```json
{
  "_id": "todo_id",
  "title": "Learn MERN",
  "completed": false,
  "user": "user_id"
}
```

### Relationship

One user can have many todos.

```text
User
 |-- Todo 1
 |-- Todo 2
 |-- Todo 3
```

In MongoDB, each todo stores the owner's user ID:

```js
user: {
  type: mongoose.Schema.Types.ObjectId,
  ref: "User"
}
```

Why this field exists:

It allows the backend to fetch only the logged-in user's todos.

---

## 5. Backend Development

---

### Step 1: Connect To MongoDB

Concept:

The backend needs a database connection before it can save or read data.

Why it is needed:

Without MongoDB, todos would disappear whenever the server restarts.

Where it fits:

`server.js` will call this connection function before starting the app.

Create `backend/src/config/db.js`:

```js
import mongoose from "mongoose";

export const connectDB = async () => {
  try {
    await mongoose.connect(process.env.MONGO_URI);
    console.log("MongoDB connected");
  } catch (error) {
    console.error("MongoDB connection failed:", error.message);
    process.exit(1);
  }
};
```

Code explanation:

- `mongoose.connect(...)` connects to MongoDB.
- `process.env.MONGO_URI` reads the connection string from `.env`.
- `try/catch` handles connection errors.
- `process.exit(1)` stops the server if database connection fails.

---

### Step 2: Create User Model

Concept:

A model defines what a document should look like in MongoDB.

Why it is needed:

We need a consistent structure for users.

Where it fits:

Auth controllers will use this model during signup and login.

Create `backend/src/models/User.js`:

```js
import mongoose from "mongoose";
import bcrypt from "bcryptjs";

const userSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: true,
      trim: true,
    },
    email: {
      type: String,
      required: true,
      unique: true,
      lowercase: true,
      trim: true,
    },
    password: {
      type: String,
      required: true,
      minlength: 6,
    },
  },
  { timestamps: true }
);

userSchema.pre("save", async function (next) {
  if (!this.isModified("password")) {
    return next();
  }

  this.password = await bcrypt.hash(this.password, 10);
  next();
});

userSchema.methods.matchPassword = async function (enteredPassword) {
  return bcrypt.compare(enteredPassword, this.password);
};

export const User = mongoose.model("User", userSchema);
```

Code explanation:

- `name`: user's display name.
- `email`: unique login email.
- `password`: stored as a hash, not plain text.
- `timestamps`: automatically adds `createdAt` and `updatedAt`.
- `pre("save")`: runs before saving a user.
- `bcrypt.hash`: converts password into a secure hash.
- `matchPassword`: compares login password with stored hash.

---

### Step 3: Create Todo Model

Concept:

Todo documents store task information.

Why it is needed:

The app must save todo title, completed state, and owner.

Where it fits:

Todo controllers will use this model for CRUD operations.

Create `backend/src/models/Todo.js`:

```js
import mongoose from "mongoose";

const todoSchema = new mongoose.Schema(
  {
    title: {
      type: String,
      required: true,
      trim: true,
    },
    completed: {
      type: Boolean,
      default: false,
    },
    user: {
      type: mongoose.Schema.Types.ObjectId,
      ref: "User",
      required: true,
    },
  },
  { timestamps: true }
);

export const Todo = mongoose.model("Todo", todoSchema);
```

Code explanation:

- `title`: task text.
- `completed`: whether the task is done.
- `user`: connects the todo to the owner.
- `timestamps`: lets us sort by creation date.

---

### Step 4: Generate JWT Token

Concept:

JWT is a token that proves the user is logged in.

Why it is needed:

The backend needs a way to know which user is making todo requests.

Where it fits:

Signup and login return this token. Frontend sends it with protected requests.

Create `backend/src/utils/generateToken.js`:

```js
import jwt from "jsonwebtoken";

export const generateToken = (userId) => {
  return jwt.sign({ userId }, process.env.JWT_SECRET, {
    expiresIn: "7d",
  });
};
```

Code explanation:

- `jwt.sign` creates a token.
- `{ userId }` is the token payload.
- `JWT_SECRET` signs the token.
- `expiresIn: "7d"` makes it valid for 7 days.

---

### Step 5: Auth Middleware

Concept:

Middleware is code that runs before a route controller.

Why it is needed:

Todo routes should only work for logged-in users.

Where it fits:

We will place this middleware before todo controllers.

Create `backend/src/middleware/authMiddleware.js`:

```js
import jwt from "jsonwebtoken";
import { User } from "../models/User.js";

export const protect = async (req, res, next) => {
  try {
    const authHeader = req.headers.authorization;

    if (!authHeader || !authHeader.startsWith("Bearer ")) {
      return res.status(401).json({ message: "No token provided" });
    }

    const token = authHeader.split(" ")[1];
    const decoded = jwt.verify(token, process.env.JWT_SECRET);

    const user = await User.findById(decoded.userId).select("-password");

    if (!user) {
      return res.status(401).json({ message: "User not found" });
    }

    req.user = user;
    next();
  } catch (error) {
    return res.status(401).json({ message: "Invalid or expired token" });
  }
};
```

Code explanation:

- Reads `Authorization` header.
- Checks for `Bearer token`.
- Verifies token.
- Finds user in MongoDB.
- Removes password from returned user.
- Stores user on `req.user`.
- Calls `next()` to continue.

---

### Step 6: Validation Middleware

Concept:

Validation checks if request data is correct before saving it.

Why it is needed:

Users might send empty titles, invalid emails, or bad data.

Where it fits:

Routes will use it before controllers.

Create `backend/src/middleware/validateMiddleware.js`:

```js
export const validate = (schema) => {
  return (req, res, next) => {
    const result = schema.safeParse(req.body);

    if (!result.success) {
      return res.status(400).json({
        message: "Validation failed",
        errors: result.error.flatten(),
      });
    }

    req.body = result.data;
    next();
  };
};
```

Code explanation:

- Accepts a Zod schema.
- Validates `req.body`.
- Returns `400` if invalid.
- Replaces `req.body` with validated data.
- Calls `next()` if valid.

---

### Step 7: Error Middleware

Concept:

Error middleware handles unknown routes and unexpected errors.

Why it is needed:

It gives consistent error responses.

Create `backend/src/middleware/errorMiddleware.js`:

```js
export const notFound = (req, res, next) => {
  res.status(404);
  next(new Error(`Route not found: ${req.originalUrl}`));
};

export const errorHandler = (err, req, res, next) => {
  const statusCode = res.statusCode === 200 ? 500 : res.statusCode;

  res.status(statusCode).json({
    message: err.message || "Internal server error",
  });
};
```

---

## 6. API Development

---

## Auth API

### Auth Validation Schemas

Concept:

Auth routes need rules for signup and login data.

Create `backend/src/schemas/authSchemas.js`:

```js
import { z } from "zod";

export const signupSchema = z.object({
  name: z.string().min(1, "Name is required"),
  email: z.string().email("Invalid email"),
  password: z.string().min(6, "Password must be at least 6 characters"),
});

export const loginSchema = z.object({
  email: z.string().email("Invalid email"),
  password: z.string().min(1, "Password is required"),
});
```

---

### Auth Controller

Concept:

Controllers contain the main logic for each route.

Why it is needed:

Routes should stay clean. Controllers do the actual work.

Create `backend/src/controllers/authController.js`:

```js
import { User } from "../models/User.js";
import { generateToken } from "../utils/generateToken.js";

export const signup = async (req, res) => {
  try {
    const { name, email, password } = req.body;

    const existingUser = await User.findOne({ email });

    if (existingUser) {
      return res.status(400).json({ message: "Email already registered" });
    }

    const user = await User.create({ name, email, password });
    const token = generateToken(user._id);

    res.status(201).json({
      token,
      user: {
        id: user._id,
        name: user.name,
        email: user.email,
      },
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const login = async (req, res) => {
  try {
    const { email, password } = req.body;

    const user = await User.findOne({ email });

    if (!user || !(await user.matchPassword(password))) {
      return res.status(401).json({ message: "Invalid email or password" });
    }

    const token = generateToken(user._id);

    res.json({
      token,
      user: {
        id: user._id,
        name: user.name,
        email: user.email,
      },
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const me = async (req, res) => {
  res.json({ user: req.user });
};
```

Important code explanation:

- `User.findOne({ email })`: checks if email exists.
- `User.create(...)`: creates a new user.
- Password hashing happens automatically in the User model.
- `generateToken(user._id)`: creates login token.
- `matchPassword`: compares login password with hashed password.
- `me`: returns current user from auth middleware.

---

### Auth Routes

Create `backend/src/routes/authRoutes.js`:

```js
import express from "express";
import { login, me, signup } from "../controllers/authController.js";
import { protect } from "../middleware/authMiddleware.js";
import { validate } from "../middleware/validateMiddleware.js";
import { loginSchema, signupSchema } from "../schemas/authSchemas.js";

const router = express.Router();

router.post("/signup", validate(signupSchema), signup);
router.post("/login", validate(loginSchema), login);
router.get("/me", protect, me);

export default router;
```

### Endpoint: Signup

Purpose:

Create a new user account.

Request:

```text
POST /api/auth/signup
```

Body:

```json
{
  "name": "Alex",
  "email": "alex@example.com",
  "password": "123456"
}
```

Response:

```json
{
  "token": "jwt_token",
  "user": {
    "id": "user_id",
    "name": "Alex",
    "email": "alex@example.com"
  }
}
```

Errors:

- `400`: invalid input
- `400`: email already registered
- `500`: server error

Data flow:

```text
Frontend signup form -> POST /signup -> validate -> create user -> return token
```

### Endpoint: Login

Request:

```text
POST /api/auth/login
```

Body:

```json
{
  "email": "alex@example.com",
  "password": "123456"
}
```

Response:

```json
{
  "token": "jwt_token",
  "user": {
    "id": "user_id",
    "name": "Alex",
    "email": "alex@example.com"
  }
}
```

Errors:

- `400`: invalid input
- `401`: wrong email or password
- `500`: server error

---

## Todo API

### Todo Validation Schemas

Create `backend/src/schemas/todoSchemas.js`:

```js
import { z } from "zod";

export const createTodoSchema = z.object({
  title: z.string().min(1, "Todo title is required"),
});

export const updateTodoSchema = z.object({
  title: z.string().min(1, "Todo title is required").optional(),
  completed: z.boolean().optional(),
});
```

---

### Todo Controller

Concept:

Todo controllers handle CRUD logic.

Important security rule:

Every query must filter by `user: req.user._id`. This prevents users from reading or editing someone else's todos.

Create `backend/src/controllers/todoController.js`:

```js
import { Todo } from "../models/Todo.js";

export const getTodos = async (req, res) => {
  try {
    const todos = await Todo.find({ user: req.user._id }).sort({ createdAt: -1 });
    res.json({ todos });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const createTodo = async (req, res) => {
  try {
    const todo = await Todo.create({
      title: req.body.title,
      user: req.user._id,
    });

    res.status(201).json({ todo });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const updateTodo = async (req, res) => {
  try {
    const todo = await Todo.findOneAndUpdate(
      {
        _id: req.params.id,
        user: req.user._id,
      },
      req.body,
      { new: true }
    );

    if (!todo) {
      return res.status(404).json({ message: "Todo not found" });
    }

    res.json({ todo });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const deleteTodo = async (req, res) => {
  try {
    const todo = await Todo.findOneAndDelete({
      _id: req.params.id,
      user: req.user._id,
    });

    if (!todo) {
      return res.status(404).json({ message: "Todo not found" });
    }

    res.json({ message: "Todo deleted" });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};
```

Code explanation:

- `getTodos`: gets only logged-in user's todos.
- `createTodo`: attaches logged-in user's ID to the todo.
- `findOneAndUpdate`: updates only if todo belongs to user.
- `findOneAndDelete`: deletes only if todo belongs to user.
- `{ new: true }`: returns updated todo instead of old todo.

---

### Todo Routes

Create `backend/src/routes/todoRoutes.js`:

```js
import express from "express";
import {
  createTodo,
  deleteTodo,
  getTodos,
  updateTodo,
} from "../controllers/todoController.js";
import { protect } from "../middleware/authMiddleware.js";
import { validate } from "../middleware/validateMiddleware.js";
import { createTodoSchema, updateTodoSchema } from "../schemas/todoSchemas.js";

const router = express.Router();

router.use(protect);

router.get("/", getTodos);
router.post("/", validate(createTodoSchema), createTodo);
router.patch("/:id", validate(updateTodoSchema), updateTodo);
router.delete("/:id", deleteTodo);

export default router;
```

Why `router.use(protect)`:

Every todo route requires login.

### Endpoint: Get Todos

Request:

```text
GET /api/todos
```

Headers:

```text
Authorization: Bearer TOKEN
```

Response:

```json
{
  "todos": [
    {
      "_id": "todo_id",
      "title": "Learn MERN",
      "completed": false
    }
  ]
}
```

### Endpoint: Create Todo

Request:

```text
POST /api/todos
```

Body:

```json
{
  "title": "Learn Express"
}
```

Response:

```json
{
  "todo": {
    "_id": "todo_id",
    "title": "Learn Express",
    "completed": false
  }
}
```

### Endpoint: Update Todo

Request:

```text
PATCH /api/todos/:id
```

Body examples:

```json
{
  "completed": true
}
```

or:

```json
{
  "title": "Learn Express deeply"
}
```

### Endpoint: Delete Todo

Request:

```text
DELETE /api/todos/:id
```

Response:

```json
{
  "message": "Todo deleted"
}
```

---

### Server Setup

Concept:

The server combines database connection, middleware, routes, and port listening.

Create `backend/src/server.js`:

```js
import express from "express";
import cors from "cors";
import dotenv from "dotenv";
import { connectDB } from "./config/db.js";
import authRoutes from "./routes/authRoutes.js";
import todoRoutes from "./routes/todoRoutes.js";
import { errorHandler, notFound } from "./middleware/errorMiddleware.js";

dotenv.config();

const app = express();

connectDB();

app.use(
  cors({
    origin: process.env.CLIENT_URL,
    credentials: true,
  })
);

app.use(express.json());

app.get("/", (req, res) => {
  res.json({ message: "Todo API is running" });
});

app.use("/api/auth", authRoutes);
app.use("/api/todos", todoRoutes);

app.use(notFound);
app.use(errorHandler);

const PORT = process.env.PORT || 5000;

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

Code explanation:

- `dotenv.config()`: loads `.env`.
- `connectDB()`: connects MongoDB.
- `cors(...)`: allows frontend requests.
- `express.json()`: reads JSON body.
- `/api/auth`: auth routes.
- `/api/todos`: todo routes.
- `notFound` and `errorHandler`: handle errors.
- `app.listen`: starts the server.

Run backend:

```bash
npm run dev
```

Expected:

```text
MongoDB connected
Server running on http://localhost:5000
```

---

## 7. Frontend Development

---

### Axios API Client

Concept:

Axios sends requests from React to Express.

Why it is needed:

Frontend needs to call signup, login, and todo APIs.

Create `frontend/src/api/axios.js`:

```js
import axios from "axios";

const api = axios.create({
  baseURL: "http://localhost:5000/api",
});

api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  return config;
});

export default api;
```

What this does:

- `baseURL`: avoids repeating backend URL.
- Interceptor runs before every request.
- If token exists, it adds `Authorization` header.

---

### Protected Route Component

Create `frontend/src/components/ProtectedRoute.jsx`:

```jsx
import { Navigate } from "react-router-dom";

export default function ProtectedRoute({ children }) {
  const token = localStorage.getItem("token");

  if (!token) {
    return <Navigate to="/login" replace />;
  }

  return children;
}
```

Purpose:

If a user is not logged in, redirect them to login.

---

### Navbar Component

Create `frontend/src/components/Navbar.jsx`:

```jsx
import { Link, useNavigate } from "react-router-dom";

export default function Navbar() {
  const navigate = useNavigate();
  const token = localStorage.getItem("token");

  const logout = () => {
    localStorage.removeItem("token");
    localStorage.removeItem("user");
    navigate("/login");
  };

  return (
    <nav className="bg-white border-b px-6 py-4 flex justify-between">
      <Link to="/todos" className="font-bold text-xl">
        Simple Todo
      </Link>

      <div className="flex gap-4">
        {token ? (
          <button onClick={logout} className="text-red-600">
            Logout
          </button>
        ) : (
          <>
            <Link to="/login">Login</Link>
            <Link to="/signup">Signup</Link>
          </>
        )}
      </div>
    </nav>
  );
}
```

What this does:

- Shows app name.
- Shows login/signup if logged out.
- Shows logout if logged in.
- Logout removes token.

---

### Signup Page

Create `frontend/src/pages/Signup.jsx`:

```jsx
import { useState } from "react";
import { Link, useNavigate } from "react-router-dom";
import api from "../api/axios";

export default function Signup() {
  const navigate = useNavigate();
  const [form, setForm] = useState({ name: "", email: "", password: "" });
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError("");

    try {
      setLoading(true);
      const res = await api.post("/auth/signup", form);
      localStorage.setItem("token", res.data.token);
      localStorage.setItem("user", JSON.stringify(res.data.user));
      navigate("/todos");
    } catch (err) {
      setError(err.response?.data?.message || "Signup failed");
    } finally {
      setLoading(false);
    }
  };

  return (
    <main className="min-h-[calc(100vh-64px)] flex items-center justify-center px-4">
      <form onSubmit={handleSubmit} className="w-full max-w-md bg-white border rounded-lg p-6 space-y-4">
        <h1 className="text-2xl font-bold">Create account</h1>

        {error && <p className="text-red-600 text-sm">{error}</p>}

        <input
          className="w-full border rounded-md px-3 py-2"
          placeholder="Name"
          value={form.name}
          onChange={(e) => setForm({ ...form, name: e.target.value })}
        />

        <input
          className="w-full border rounded-md px-3 py-2"
          placeholder="Email"
          type="email"
          value={form.email}
          onChange={(e) => setForm({ ...form, email: e.target.value })}
        />

        <input
          className="w-full border rounded-md px-3 py-2"
          placeholder="Password"
          type="password"
          value={form.password}
          onChange={(e) => setForm({ ...form, password: e.target.value })}
        />

        <button className="w-full bg-blue-600 text-white py-2 rounded-md">
          {loading ? "Creating..." : "Sign Up"}
        </button>

        <p className="text-sm">
          Already have an account? <Link className="text-blue-600" to="/login">Login</Link>
        </p>
      </form>
    </main>
  );
}
```

Important explanation:

- `useState` stores form data.
- `handleSubmit` sends data to backend.
- Token is saved after signup.
- User is redirected to `/todos`.

---

### Login Page

Create `frontend/src/pages/Login.jsx`:

```jsx
import { useState } from "react";
import { Link, useNavigate } from "react-router-dom";
import api from "../api/axios";

export default function Login() {
  const navigate = useNavigate();
  const [form, setForm] = useState({ email: "", password: "" });
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError("");

    try {
      setLoading(true);
      const res = await api.post("/auth/login", form);
      localStorage.setItem("token", res.data.token);
      localStorage.setItem("user", JSON.stringify(res.data.user));
      navigate("/todos");
    } catch (err) {
      setError(err.response?.data?.message || "Login failed");
    } finally {
      setLoading(false);
    }
  };

  return (
    <main className="min-h-[calc(100vh-64px)] flex items-center justify-center px-4">
      <form onSubmit={handleSubmit} className="w-full max-w-md bg-white border rounded-lg p-6 space-y-4">
        <h1 className="text-2xl font-bold">Login</h1>

        {error && <p className="text-red-600 text-sm">{error}</p>}

        <input
          className="w-full border rounded-md px-3 py-2"
          placeholder="Email"
          type="email"
          value={form.email}
          onChange={(e) => setForm({ ...form, email: e.target.value })}
        />

        <input
          className="w-full border rounded-md px-3 py-2"
          placeholder="Password"
          type="password"
          value={form.password}
          onChange={(e) => setForm({ ...form, password: e.target.value })}
        />

        <button className="w-full bg-blue-600 text-white py-2 rounded-md">
          {loading ? "Logging in..." : "Login"}
        </button>

        <p className="text-sm">
          New user? <Link className="text-blue-600" to="/signup">Create account</Link>
        </p>
      </form>
    </main>
  );
}
```

---

### Todo Form Component

Create `frontend/src/components/TodoForm.jsx`:

```jsx
import { useState } from "react";

export default function TodoForm({ onCreate }) {
  const [title, setTitle] = useState("");

  const handleSubmit = (e) => {
    e.preventDefault();

    if (!title.trim()) {
      return;
    }

    onCreate(title);
    setTitle("");
  };

  return (
    <form onSubmit={handleSubmit} className="flex gap-2">
      <input
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        placeholder="Add a new todo"
        className="flex-1 border rounded-md px-3 py-2"
      />

      <button className="bg-blue-600 text-white px-4 py-2 rounded-md">
        Add
      </button>
    </form>
  );
}
```

Purpose:

This component only handles the input field. The parent page decides how to call the backend.

---

### Todo Item Component

Create `frontend/src/components/TodoItem.jsx`:

```jsx
import { useState } from "react";

export default function TodoItem({ todo, onToggle, onUpdate, onDelete }) {
  const [editing, setEditing] = useState(false);
  const [title, setTitle] = useState(todo.title);

  const saveEdit = () => {
    if (!title.trim()) return;
    onUpdate(todo._id, title);
    setEditing(false);
  };

  return (
    <div className="bg-white border rounded-lg p-4 flex items-center gap-3">
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => onToggle(todo)}
      />

      {editing ? (
        <input
          value={title}
          onChange={(e) => setTitle(e.target.value)}
          className="flex-1 border rounded-md px-2 py-1"
        />
      ) : (
        <p className={`flex-1 ${todo.completed ? "line-through text-gray-400" : ""}`}>
          {todo.title}
        </p>
      )}

      {editing ? (
        <button onClick={saveEdit} className="text-green-600">
          Save
        </button>
      ) : (
        <button onClick={() => setEditing(true)} className="text-blue-600">
          Edit
        </button>
      )}

      <button onClick={() => onDelete(todo._id)} className="text-red-600">
        Delete
      </button>
    </div>
  );
}
```

What this component does:

- Shows one todo.
- Lets user check/uncheck.
- Lets user edit title.
- Lets user delete todo.

---

### Todos Page

Create `frontend/src/pages/Todos.jsx`:

```jsx
import { useEffect, useState } from "react";
import api from "../api/axios";
import TodoForm from "../components/TodoForm";
import TodoItem from "../components/TodoItem";

export default function Todos() {
  const [todos, setTodos] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState("");

  const fetchTodos = async () => {
    try {
      setLoading(true);
      const res = await api.get("/todos");
      setTodos(res.data.todos);
    } catch (err) {
      setError(err.response?.data?.message || "Failed to load todos");
    } finally {
      setLoading(false);
    }
  };

  const createTodo = async (title) => {
    try {
      const res = await api.post("/todos", { title });
      setTodos([res.data.todo, ...todos]);
    } catch (err) {
      alert(err.response?.data?.message || "Failed to create todo");
    }
  };

  const toggleTodo = async (todo) => {
    const res = await api.patch(`/todos/${todo._id}`, {
      completed: !todo.completed,
    });

    setTodos(
      todos.map((item) => (item._id === todo._id ? res.data.todo : item))
    );
  };

  const updateTodo = async (id, title) => {
    const res = await api.patch(`/todos/${id}`, { title });

    setTodos(
      todos.map((item) => (item._id === id ? res.data.todo : item))
    );
  };

  const deleteTodo = async (id) => {
    await api.delete(`/todos/${id}`);
    setTodos(todos.filter((todo) => todo._id !== id));
  };

  useEffect(() => {
    fetchTodos();
  }, []);

  return (
    <main className="max-w-2xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-2">My Todos</h1>
      <p className="text-gray-600 mb-6">Create and manage your daily tasks.</p>

      <TodoForm onCreate={createTodo} />

      {loading && <p className="mt-6">Loading todos...</p>}
      {error && <p className="mt-6 text-red-600">{error}</p>}

      <div className="mt-6 space-y-3">
        {!loading && todos.length === 0 && (
          <p className="text-gray-500 text-center">No todos yet.</p>
        )}

        {todos.map((todo) => (
          <TodoItem
            key={todo._id}
            todo={todo}
            onToggle={toggleTodo}
            onUpdate={updateTodo}
            onDelete={deleteTodo}
          />
        ))}
      </div>
    </main>
  );
}
```

Important data flow:

```text
Todos page loads
GET /api/todos
Backend verifies token
Backend returns user's todos
React stores todos in state
TodoItem renders each todo
```

---

### App Routes

Update `frontend/src/App.jsx`:

```jsx
import { BrowserRouter, Navigate, Route, Routes } from "react-router-dom";
import Navbar from "./components/Navbar";
import ProtectedRoute from "./components/ProtectedRoute";
import Login from "./pages/Login";
import Signup from "./pages/Signup";
import Todos from "./pages/Todos";

export default function App() {
  return (
    <BrowserRouter>
      <Navbar />
      <Routes>
        <Route path="/" element={<Navigate to="/todos" replace />} />
        <Route path="/login" element={<Login />} />
        <Route path="/signup" element={<Signup />} />
        <Route
          path="/todos"
          element={
            <ProtectedRoute>
              <Todos />
            </ProtectedRoute>
          }
        />
      </Routes>
    </BrowserRouter>
  );
}
```

Update `frontend/src/main.jsx`:

```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App.jsx";
import "./index.css";

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

---

## 8. Core Features

### Feature: Signup

User flow:

```text
User fills signup form
Frontend sends POST /auth/signup
Backend validates data
Backend hashes password
Backend creates user
Backend returns token
Frontend stores token
User goes to todos page
```

Database interaction:

```js
User.create({ name, email, password })
```

### Feature: Login

User flow:

```text
User enters email/password
Backend checks user
Backend compares password
Backend returns token
Frontend stores token
```

### Feature: Create Todo

User flow:

```text
User types todo
Clicks Add
Frontend sends POST /todos
Backend creates todo with req.user._id
Frontend adds new todo to UI
```

### Feature: Complete Todo

User flow:

```text
User clicks checkbox
Frontend sends PATCH /todos/:id
Backend updates completed
Frontend replaces updated todo in state
```

### Feature: Edit Todo

User flow:

```text
User clicks Edit
Changes title
Clicks Save
Frontend sends PATCH /todos/:id
Backend updates title
Frontend updates UI
```

### Feature: Delete Todo

User flow:

```text
User clicks Delete
Frontend sends DELETE /todos/:id
Backend deletes todo
Frontend removes todo from state
```

---

## 9. Authentication & Authorization

### Password Handling

Passwords are hashed before saving.

```js
this.password = await bcrypt.hash(this.password, 10);
```

Why:

If database leaks, users' real passwords are not visible.

### JWT Flow

```text
Login/signup successful
Backend creates token
Frontend stores token
Frontend sends token in Authorization header
Backend verifies token
Backend knows current user
```

### Protected Routes

Backend:

```js
router.use(protect);
```

Frontend:

```jsx
<ProtectedRoute>
  <Todos />
</ProtectedRoute>
```

### Logout

```js
localStorage.removeItem("token");
localStorage.removeItem("user");
```

---

## 10. Validation & Error Handling

### Backend Validation

Zod checks request body:

```js
z.object({
  title: z.string().min(1),
});
```

### Frontend Validation

Frontend prevents empty todos:

```js
if (!title.trim()) {
  return;
}
```

### API Error Responses

Example:

```json
{
  "message": "Validation failed",
  "errors": {
    "fieldErrors": {
      "title": ["Todo title is required"]
    }
  }
}
```

### Edge Cases

- Empty todo title
- Invalid email
- Duplicate email
- Wrong password
- Missing token
- Expired token
- User tries to edit another user's todo

Best practice:

Always check ownership in database queries:

```js
{
  _id: req.params.id,
  user: req.user._id
}
```

---

## 11. Testing & Debugging

### Test Backend With Postman

#### Signup

```text
POST http://localhost:5000/api/auth/signup
```

Body:

```json
{
  "name": "Alex",
  "email": "alex@example.com",
  "password": "123456"
}
```

#### Login

```text
POST http://localhost:5000/api/auth/login
```

Body:

```json
{
  "email": "alex@example.com",
  "password": "123456"
}
```

Copy token.

#### Create Todo

```text
POST http://localhost:5000/api/todos
Authorization: Bearer TOKEN
```

Body:

```json
{
  "title": "Learn MERN"
}
```

#### Get Todos

```text
GET http://localhost:5000/api/todos
Authorization: Bearer TOKEN
```

#### Update Todo

```text
PATCH http://localhost:5000/api/todos/TODO_ID
Authorization: Bearer TOKEN
```

Body:

```json
{
  "completed": true
}
```

#### Delete Todo

```text
DELETE http://localhost:5000/api/todos/TODO_ID
Authorization: Bearer TOKEN
```

### Frontend Testing Checklist

1. Open frontend.
2. Signup.
3. Confirm redirect to todos.
4. Add todo.
5. Mark todo completed.
6. Edit todo.
7. Delete todo.
8. Logout.
9. Login again.
10. Confirm todos are still saved.

### Common Bugs

#### CORS Error

Fix backend:

```js
app.use(cors({ origin: process.env.CLIENT_URL }));
```

#### Unauthorized

Check:

- Token exists in localStorage.
- Axios interceptor sends token.
- Backend `JWT_SECRET` is correct.

#### MongoDB Error

Check:

- `MONGO_URI` is correct.
- IP is allowed in MongoDB Atlas.
- Database user/password is correct.

---

## 12. Running The Complete Project

### Start Backend

```bash
cd backend
npm run dev
```

Expected:

```text
MongoDB connected
Server running on http://localhost:5000
```

### Start Frontend

```bash
cd frontend
npm run dev
```

Expected:

```text
Local: http://localhost:5173
```

Open:

```text
http://localhost:5173
```

Startup sequence:

1. Start MongoDB or use MongoDB Atlas.
2. Start backend.
3. Start frontend.
4. Open browser.

---

## 13. Deployment

### Database Deployment

Use MongoDB Atlas:

1. Create cluster.
2. Create database user.
3. Add network access.
4. Copy connection string.
5. Use it as `MONGO_URI`.

### Backend Deployment

Use Render or Railway.

Environment variables:

```env
PORT=5000
MONGO_URI=your_atlas_url
JWT_SECRET=your_long_secret
CLIENT_URL=https://your-frontend-url.vercel.app
```

Build command:

```bash
npm install
```

Start command:

```bash
npm start
```

### Frontend Deployment

Use Vercel.

Set production API URL by changing Axios:

```js
const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
});
```

Create `frontend/.env`:

```env
VITE_API_URL=http://localhost:5000/api
```

In Vercel, set:

```env
VITE_API_URL=https://your-backend-url.onrender.com/api
```

---

## 14. Next Improvements

### Advanced Features

1. Todo due dates.
2. Todo priorities.
3. Todo categories.
4. Search todos.
5. Filter completed/pending todos.
6. Drag-and-drop reordering.
7. Dark mode.
8. Email verification.
9. Password reset.
10. Refresh tokens.

### Security Improvements

1. Store JWT in HTTP-only cookies.
2. Add rate limiting.
3. Add stronger password rules.
4. Add helmet security headers.
5. Add refresh token rotation.

### Performance Improvements

1. Pagination for todos.
2. Debounced search.
3. React Query for caching.
4. Index MongoDB fields.

### Production Best Practices

1. Use separate production environment variables.
2. Add logging.
3. Add tests.
4. Add CI/CD.
5. Monitor server errors.

---

## Final Summary

You built a complete MERN todo application.

You learned:

- How to create an Express backend.
- How to connect MongoDB.
- How to create Mongoose models.
- How to build REST APIs.
- How to validate input with Zod.
- How to hash passwords with bcrypt.
- How JWT authentication works.
- How React calls backend APIs.
- How protected routes work.
- How CRUD operations work end to end.

The most important idea:

```text
Frontend shows the UI.
Backend protects the data.
Database stores the data.
JWT connects a request to a logged-in user.
```

