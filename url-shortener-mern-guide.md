# URL Shortener Clone - Complete Full-Stack MERN Guide

Project Name: URL Shortener Clone  
Project Type: Full-stack URL shortener app  
Tech Stack: MongoDB, Express.js, React, Node.js, Mongoose, JWT, Tailwind CSS, Axios  
Skill Level: Beginner

---

## What We Will Build

We will build a URL shortener like Bitly.

The user will paste a long URL, and our app will create a short URL. When someone opens the short URL, the backend will redirect them to the original long URL.

Main features:

- Create short links
- Redirect short links to original URLs
- Store links in MongoDB
- Track number of clicks
- Show created links in the frontend
- Add signup and login in Version 2
- Add user dashboard in Version 2
- Protect private routes using JWT

---

## Tech Stack Used

### Frontend

- `React`: builds the user interface.
- `Vite`: creates a fast React development setup.
- `React Router`: handles frontend pages like Home, Login, Signup, Dashboard.
- `Axios`: sends HTTP requests from frontend to backend.
- `Tailwind CSS`: styles the app quickly using utility classes.

### Backend

- `Node.js`: runs JavaScript on the server.
- `Express.js`: creates backend APIs and routes.
- `Mongoose`: connects Node.js to MongoDB using models.
- `Zod`: validates request body data.
- `nanoid`: generates unique short URL codes.
- `bcryptjs`: hashes passwords.
- `jsonwebtoken`: creates and verifies JWT auth tokens.
- `cors`: allows frontend and backend to communicate.
- `dotenv`: reads environment variables from `.env`.

### Database

- `MongoDB`: stores users and shortened URLs.
- `MongoDB Atlas`: cloud MongoDB database.
- `Mongoose`: ODM, meaning Object Data Modeling tool. It lets us define schemas and interact with MongoDB using JavaScript.

---

## Final Project Structure

```text
url-shortener-mern/
|-- backend/
|   |-- src/
|   |   |-- config/
|   |   |   |-- db.js
|   |   |-- controllers/
|   |   |   |-- authController.js
|   |   |   |-- urlController.js
|   |   |-- middleware/
|   |   |   |-- authMiddleware.js
|   |   |-- models/
|   |   |   |-- User.js
|   |   |   |-- Url.js
|   |   |-- routes/
|   |   |   |-- authRoutes.js
|   |   |   |-- urlRoutes.js
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
|   |   |   |-- UrlForm.jsx
|   |   |   |-- UrlList.jsx
|   |   |-- pages/
|   |   |   |-- Home.jsx
|   |   |   |-- Login.jsx
|   |   |   |-- Signup.jsx
|   |   |   |-- Dashboard.jsx
|   |   |-- App.jsx
|   |   |-- main.jsx
|   |-- package.json
```

Important files:

- `server.js`: starts the backend server.
- `db.js`: connects backend to MongoDB.
- `Url.js`: MongoDB model for shortened URLs.
- `User.js`: MongoDB model for registered users.
- `urlController.js`: contains URL shortening and redirect logic.
- `authController.js`: contains signup and login logic.
- `authMiddleware.js`: protects routes using JWT.
- `axios.js`: frontend API configuration.
- `Home.jsx`: public page for creating a short URL.
- `Dashboard.jsx`: private page showing user links.

---

## Prerequisites

Before starting, install:

- Node.js
- npm
- VS Code
- MongoDB Atlas account
- Postman or Thunder Client for API testing

Check your Node version:

```bash
node -v
npm -v
```

Create a MongoDB Atlas database:

1. Go to MongoDB Atlas.
2. Create a free cluster.
3. Create a database user.
4. Allow network access from your IP.
5. Copy the connection string.

Example MongoDB URL:

```env
mongodb+srv://username:password@cluster.mongodb.net/url_shortener
```

---

## Version 1: Basic MVP

In Version 1, we will build the simplest working URL shortener:

- User enters long URL.
- Backend creates short code.
- MongoDB stores original URL and short code.
- User opens short URL.
- Backend redirects to original URL.

No authentication yet.

---

### STEP 1: Create Project Folders

What we are doing:

Creating separate folders for backend and frontend.

Why this is needed:

MERN apps usually keep backend and frontend separate because they run on different ports during development.

Commands:

```bash
mkdir url-shortener-mern
cd url-shortener-mern
mkdir backend frontend
```

How to test:

```bash
ls
```

You should see:

```text
backend
frontend
```

---

## Backend Setup

---

### STEP 2: Initialize Backend

Go into backend:

```bash
cd backend
npm init -y
```

Install packages:

```bash
npm install express mongoose cors dotenv nanoid zod bcryptjs jsonwebtoken
npm install -D nodemon
```

Update `backend/package.json`:

```json
{
  "name": "url-shortener-backend",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "nodemon src/server.js",
    "start": "node src/server.js"
  }
}
```

Why `"type": "module"` is used:

It allows us to use modern import/export syntax:

```js
import express from "express";
```

---

### STEP 3: Setup Environment Variables

Create `backend/.env`:

```env
PORT=5000
MONGO_URI=your_mongodb_connection_string
BASE_URL=http://localhost:5000
JWT_SECRET=your_super_secret_key
```

Explanation:

- `PORT`: backend server port.
- `MONGO_URI`: MongoDB connection string.
- `BASE_URL`: used to create full short URLs.
- `JWT_SECRET`: used later for login tokens.

Important:

Never push `.env` to GitHub.

---

### STEP 4: Connect MongoDB

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

What this does:

- Reads `MONGO_URI`.
- Connects Express app to MongoDB.
- Stops the app if database connection fails.

---

### STEP 5: Create URL Model

Create `backend/src/models/Url.js`:

```js
import mongoose from "mongoose";

const urlSchema = new mongoose.Schema(
  {
    originalUrl: {
      type: String,
      required: true,
    },
    shortCode: {
      type: String,
      required: true,
      unique: true,
    },
    shortUrl: {
      type: String,
      required: true,
    },
    clicks: {
      type: Number,
      default: 0,
    },
    user: {
      type: mongoose.Schema.Types.ObjectId,
      ref: "User",
      default: null,
    },
  },
  { timestamps: true }
);

export const Url = mongoose.model("Url", urlSchema);
```

Explanation:

- `originalUrl`: full URL entered by user.
- `shortCode`: unique short code like `aB12xY`.
- `shortUrl`: complete short URL like `http://localhost:5000/aB12xY`.
- `clicks`: number of times short URL was opened.
- `user`: used later when authentication is added.
- `timestamps`: automatically adds `createdAt` and `updatedAt`.

---

### STEP 6: Create URL Controller

Create `backend/src/controllers/urlController.js`:

```js
import { nanoid } from "nanoid";
import { z } from "zod";
import { Url } from "../models/Url.js";

const createUrlSchema = z.object({
  originalUrl: z.string().url("Please enter a valid URL"),
});

export const createShortUrl = async (req, res) => {
  try {
    const parsed = createUrlSchema.safeParse(req.body);

    if (!parsed.success) {
      return res.status(400).json({
        message: "Validation failed",
        errors: parsed.error.flatten(),
      });
    }

    const { originalUrl } = parsed.data;

    const existingUrl = await Url.findOne({ originalUrl, user: null });

    if (existingUrl) {
      return res.status(200).json(existingUrl);
    }

    const shortCode = nanoid(7);
    const shortUrl = `${process.env.BASE_URL}/${shortCode}`;

    const newUrl = await Url.create({
      originalUrl,
      shortCode,
      shortUrl,
    });

    res.status(201).json(newUrl);
  } catch (error) {
    res.status(500).json({
      message: "Something went wrong",
      error: error.message,
    });
  }
};

export const redirectToOriginalUrl = async (req, res) => {
  try {
    const { shortCode } = req.params;

    const url = await Url.findOne({ shortCode });

    if (!url) {
      return res.status(404).json({ message: "Short URL not found" });
    }

    url.clicks += 1;
    await url.save();

    return res.redirect(url.originalUrl);
  } catch (error) {
    res.status(500).json({
      message: "Something went wrong",
      error: error.message,
    });
  }
};

export const getRecentUrls = async (req, res) => {
  try {
    const urls = await Url.find({ user: null })
      .sort({ createdAt: -1 })
      .limit(10);

    res.json(urls);
  } catch (error) {
    res.status(500).json({
      message: "Something went wrong",
      error: error.message,
    });
  }
};
```

Important logic:

- `nanoid(7)` creates a random short code.
- `z.string().url()` checks that the input is a valid URL.
- `res.redirect(originalUrl)` sends the browser to the original website.
- `clicks += 1` tracks how many times the short link was opened.

---

### STEP 7: Create URL Routes

Create `backend/src/routes/urlRoutes.js`:

```js
import express from "express";
import {
  createShortUrl,
  getRecentUrls,
  redirectToOriginalUrl,
} from "../controllers/urlController.js";

const router = express.Router();

router.post("/shorten", createShortUrl);
router.get("/recent", getRecentUrls);
router.get("/:shortCode", redirectToOriginalUrl);

export default router;
```

Routes:

| Method | URL | Purpose |
|---|---|---|
| `POST` | `/api/urls/shorten` | Create short URL |
| `GET` | `/api/urls/recent` | Get recent public URLs |
| `GET` | `/api/urls/:shortCode` | Redirect to original URL |

---

### STEP 8: Create Express Server

Create `backend/src/server.js`:

```js
import express from "express";
import cors from "cors";
import dotenv from "dotenv";
import { connectDB } from "./config/db.js";
import urlRoutes from "./routes/urlRoutes.js";

dotenv.config();

const app = express();

connectDB();

app.use(cors());
app.use(express.json());

app.get("/", (req, res) => {
  res.json({ message: "URL Shortener API is running" });
});

app.use("/api/urls", urlRoutes);

app.get("/:shortCode", async (req, res, next) => {
  req.url = `/api/urls${req.url}`;
  next();
}, urlRoutes);

const PORT = process.env.PORT || 5000;

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

Why we added `/:shortCode`:

The user should be able to open:

```text
http://localhost:5000/abc123
```

Instead of:

```text
http://localhost:5000/api/urls/abc123
```

---

### STEP 9: Test Backend

Start backend:

```bash
npm run dev
```

Expected output:

```text
MongoDB connected
Server running on http://localhost:5000
```

Test health route:

```text
GET http://localhost:5000/
```

Response:

```json
{
  "message": "URL Shortener API is running"
}
```

Test create short URL:

```text
POST http://localhost:5000/api/urls/shorten
```

Body:

```json
{
  "originalUrl": "https://www.google.com"
}
```

Response:

```json
{
  "_id": "mongo_id",
  "originalUrl": "https://www.google.com",
  "shortCode": "abc1234",
  "shortUrl": "http://localhost:5000/abc1234",
  "clicks": 0
}
```

Open the `shortUrl` in browser. It should redirect to Google.

---

## Database Setup

MongoDB is document-based, not table-based.

Instead of SQL tables, MongoDB uses:

- Database
- Collections
- Documents

In this project:

| SQL idea | MongoDB idea |
|---|---|
| Table | Collection |
| Row | Document |
| Column | Field |

### URL Collection

Model:

```text
Url
```

Fields:

- `originalUrl`: original long URL.
- `shortCode`: unique code.
- `shortUrl`: full shortened URL.
- `clicks`: number of visits.
- `user`: owner of URL, optional in Version 1.
- `createdAt`: created automatically.
- `updatedAt`: updated automatically.

### User Collection

Used in Version 2.

Fields:

- `name`
- `email`
- `password`

Relationship:

One user can have many shortened URLs.

In MongoDB, we store the user ID inside each URL document:

```js
user: {
  type: mongoose.Schema.Types.ObjectId,
  ref: "User"
}
```

This means each URL can belong to one user.

---

## Frontend Setup

---

### STEP 1: Create React App

Open a new terminal:

```bash
cd url-shortener-mern/frontend
npm create vite@latest . -- --template react
npm install
```

Install frontend packages:

```bash
npm install axios react-router-dom
```

Install Tailwind:

```bash
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
```

---

### STEP 2: Setup Axios

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

Why this is useful:

- We do not repeat the backend URL everywhere.
- Later, JWT token is automatically attached to protected requests.

---

### STEP 3: Create URL Form Component

Create `frontend/src/components/UrlForm.jsx`:

```jsx
import { useState } from "react";
import api from "../api/axios";

export default function UrlForm({ onCreated }) {
  const [originalUrl, setOriginalUrl] = useState("");
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState("");
  const [shortUrl, setShortUrl] = useState("");

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError("");
    setShortUrl("");

    if (!originalUrl.trim()) {
      setError("Please enter a URL");
      return;
    }

    try {
      setLoading(true);
      const res = await api.post("/urls/shorten", { originalUrl });
      setShortUrl(res.data.shortUrl);
      setOriginalUrl("");
      onCreated?.();
    } catch (err) {
      setError(err.response?.data?.message || "Failed to shorten URL");
    } finally {
      setLoading(false);
    }
  };

  const copyToClipboard = async () => {
    await navigator.clipboard.writeText(shortUrl);
    alert("Copied!");
  };

  return (
    <div className="bg-white border rounded-lg p-6 shadow-sm">
      <form onSubmit={handleSubmit} className="space-y-4">
        <input
          type="url"
          value={originalUrl}
          onChange={(e) => setOriginalUrl(e.target.value)}
          placeholder="Paste your long URL here"
          className="w-full border rounded-md px-4 py-3 outline-none focus:ring-2 focus:ring-blue-500"
        />

        {error && <p className="text-red-600 text-sm">{error}</p>}

        <button
          disabled={loading}
          className="w-full bg-blue-600 text-white py-3 rounded-md font-medium disabled:opacity-60"
        >
          {loading ? "Shortening..." : "Shorten URL"}
        </button>
      </form>

      {shortUrl && (
        <div className="mt-4 bg-gray-50 border rounded-md p-4">
          <p className="text-sm text-gray-600 mb-2">Your short URL:</p>
          <div className="flex gap-2">
            <a
              href={shortUrl}
              target="_blank"
              className="text-blue-600 underline break-all flex-1"
            >
              {shortUrl}
            </a>
            <button
              onClick={copyToClipboard}
              className="bg-gray-900 text-white px-3 py-1 rounded-md"
            >
              Copy
            </button>
          </div>
        </div>
      )}
    </div>
  );
}
```

Important React concepts:

- `useState` stores form values.
- `onSubmit` handles form submission.
- `axios.post` sends data to backend.
- Loading state prevents double submission.

---

### STEP 4: Create URL List Component

Create `frontend/src/components/UrlList.jsx`:

```jsx
export default function UrlList({ urls }) {
  if (!urls.length) {
    return (
      <div className="text-center text-gray-500 mt-6">
        No URLs created yet.
      </div>
    );
  }

  return (
    <div className="mt-6 space-y-3">
      {urls.map((url) => (
        <div key={url._id} className="bg-white border rounded-lg p-4">
          <a
            href={url.shortUrl}
            target="_blank"
            className="text-blue-600 font-medium underline"
          >
            {url.shortUrl}
          </a>

          <p className="text-gray-600 text-sm break-all mt-1">
            {url.originalUrl}
          </p>

          <p className="text-sm text-gray-500 mt-2">
            Clicks: {url.clicks}
          </p>
        </div>
      ))}
    </div>
  );
}
```

What this does:

- Shows each short URL.
- Shows original URL.
- Shows click count.

---

### STEP 5: Create Home Page

Create `frontend/src/pages/Home.jsx`:

```jsx
import { useEffect, useState } from "react";
import api from "../api/axios";
import UrlForm from "../components/UrlForm";
import UrlList from "../components/UrlList";

export default function Home() {
  const [urls, setUrls] = useState([]);

  const fetchUrls = async () => {
    const res = await api.get("/urls/recent");
    setUrls(res.data);
  };

  useEffect(() => {
    fetchUrls();
  }, []);

  return (
    <main className="min-h-screen bg-gray-100 px-4 py-10">
      <div className="max-w-2xl mx-auto">
        <h1 className="text-3xl font-bold text-center mb-2">
          URL Shortener
        </h1>

        <p className="text-center text-gray-600 mb-8">
          Convert long URLs into short, shareable links.
        </p>

        <UrlForm onCreated={fetchUrls} />
        <UrlList urls={urls} />
      </div>
    </main>
  );
}
```

What this does:

- Fetches recent short URLs when page loads.
- Renders the form.
- Renders the URL list.
- Refreshes the list after creating a URL.

---

### STEP 6: Setup App Routing

Update `frontend/src/App.jsx`:

```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";
import Home from "./pages/Home";

export default function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
      </Routes>
    </BrowserRouter>
  );
}
```

Run frontend:

```bash
npm run dev
```

Open:

```text
http://localhost:5173
```

---

## Authentication, If Needed

Now we will add signup, login, JWT tokens, and a private dashboard.

Authentication flow:

```text
User signs up
Backend hashes password
Backend stores user
Backend creates JWT token
Frontend stores token in localStorage
Frontend sends token with protected requests
Backend verifies token
Protected route returns user data
```

---

## Version 2: Add Important Features

---

### Feature 1: User Model

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

Important concepts:

- Password is never stored as plain text.
- `bcrypt.hash` turns password into a secure hash.
- `matchPassword` compares login password with stored hash.

---

### Feature 2: JWT Token Utility

Create `backend/src/utils/generateToken.js`:

```js
import jwt from "jsonwebtoken";

export const generateToken = (userId) => {
  return jwt.sign({ userId }, process.env.JWT_SECRET, {
    expiresIn: "7d",
  });
};
```

What JWT contains:

```json
{
  "userId": "mongo_user_id"
}
```

The token proves who the user is.

---

### Feature 3: Auth Controller

Create `backend/src/controllers/authController.js`:

```js
import { z } from "zod";
import { User } from "../models/User.js";
import { generateToken } from "../utils/generateToken.js";

const signupSchema = z.object({
  name: z.string().min(1, "Name is required"),
  email: z.string().email("Invalid email"),
  password: z.string().min(6, "Password must be at least 6 characters"),
});

const loginSchema = z.object({
  email: z.string().email("Invalid email"),
  password: z.string().min(1, "Password is required"),
});

export const signup = async (req, res) => {
  try {
    const parsed = signupSchema.safeParse(req.body);

    if (!parsed.success) {
      return res.status(400).json({
        message: "Validation failed",
        errors: parsed.error.flatten(),
      });
    }

    const { name, email, password } = parsed.data;

    const existingUser = await User.findOne({ email });

    if (existingUser) {
      return res.status(400).json({ message: "Email already registered" });
    }

    const user = await User.create({ name, email, password });

    res.status(201).json({
      token: generateToken(user._id),
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
    const parsed = loginSchema.safeParse(req.body);

    if (!parsed.success) {
      return res.status(400).json({
        message: "Validation failed",
        errors: parsed.error.flatten(),
      });
    }

    const { email, password } = parsed.data;

    const user = await User.findOne({ email });

    if (!user || !(await user.matchPassword(password))) {
      return res.status(401).json({ message: "Invalid email or password" });
    }

    res.json({
      token: generateToken(user._id),
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

export const getMe = async (req, res) => {
  res.json(req.user);
};
```

---

### Feature 4: Auth Middleware

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
    res.status(401).json({ message: "Invalid or expired token" });
  }
};
```

What middleware does:

- Reads token from headers.
- Verifies token.
- Finds user.
- Attaches user to `req.user`.
- Allows route to continue.

---

### Feature 5: Auth Routes

Create `backend/src/routes/authRoutes.js`:

```js
import express from "express";
import { getMe, login, signup } from "../controllers/authController.js";
import { protect } from "../middleware/authMiddleware.js";

const router = express.Router();

router.post("/signup", signup);
router.post("/login", login);
router.get("/me", protect, getMe);

export default router;
```

Update `backend/src/server.js`:

```js
import authRoutes from "./routes/authRoutes.js";

app.use("/api/auth", authRoutes);
```

Add that line before URL routes.

Auth routes:

| Method | URL | Purpose |
|---|---|---|
| `POST` | `/api/auth/signup` | Create account |
| `POST` | `/api/auth/login` | Login account |
| `GET` | `/api/auth/me` | Get logged-in user |

---

### Feature 6: Create User-Owned Short URLs

Update `createShortUrl` in `urlController.js`:

```js
const userId = req.user?._id || null;

const existingUrl = await Url.findOne({ originalUrl, user: userId });

const newUrl = await Url.create({
  originalUrl,
  shortCode,
  shortUrl,
  user: userId,
});
```

Create protected dashboard controller:

```js
export const getMyUrls = async (req, res) => {
  try {
    const urls = await Url.find({ user: req.user._id }).sort({ createdAt: -1 });
    res.json(urls);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};
```

Update `urlRoutes.js`:

```js
import { protect } from "../middleware/authMiddleware.js";
import { getMyUrls } from "../controllers/urlController.js";

router.post("/shorten", protect, createShortUrl);
router.get("/my-urls", protect, getMyUrls);
```

If you want public shortening and private shortening both, create two routes:

```js
router.post("/shorten", createShortUrl);
router.post("/shorten/private", protect, createShortUrl);
```

---

### Feature 7: Frontend Signup Page

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
      navigate("/dashboard");
    } catch (err) {
      setError(err.response?.data?.message || "Signup failed");
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="min-h-screen bg-gray-100 flex items-center justify-center px-4">
      <form onSubmit={handleSubmit} className="bg-white p-6 rounded-lg border w-full max-w-md space-y-4">
        <h1 className="text-2xl font-bold">Create Account</h1>

        {error && <p className="text-red-600">{error}</p>}

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
          Already have an account? <Link to="/login" className="text-blue-600">Login</Link>
        </p>
      </form>
    </div>
  );
}
```

---

### Feature 8: Frontend Login Page

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
      navigate("/dashboard");
    } catch (err) {
      setError(err.response?.data?.message || "Login failed");
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="min-h-screen bg-gray-100 flex items-center justify-center px-4">
      <form onSubmit={handleSubmit} className="bg-white p-6 rounded-lg border w-full max-w-md space-y-4">
        <h1 className="text-2xl font-bold">Login</h1>

        {error && <p className="text-red-600">{error}</p>}

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
          New user? <Link to="/signup" className="text-blue-600">Create account</Link>
        </p>
      </form>
    </div>
  );
}
```

---

### Feature 9: Protected Route Component

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

Prevents logged-out users from opening private pages.

---

### Feature 10: Dashboard Page

Create `frontend/src/pages/Dashboard.jsx`:

```jsx
import { useEffect, useState } from "react";
import api from "../api/axios";
import UrlForm from "../components/UrlForm";
import UrlList from "../components/UrlList";

export default function Dashboard() {
  const [urls, setUrls] = useState([]);

  const fetchMyUrls = async () => {
    const res = await api.get("/urls/my-urls");
    setUrls(res.data);
  };

  const logout = () => {
    localStorage.removeItem("token");
    window.location.href = "/login";
  };

  useEffect(() => {
    fetchMyUrls();
  }, []);

  return (
    <main className="min-h-screen bg-gray-100 px-4 py-8">
      <div className="max-w-3xl mx-auto">
        <div className="flex justify-between items-center mb-6">
          <h1 className="text-3xl font-bold">Dashboard</h1>
          <button onClick={logout} className="bg-gray-900 text-white px-4 py-2 rounded-md">
            Logout
          </button>
        </div>

        <UrlForm onCreated={fetchMyUrls} />

        <h2 className="text-xl font-semibold mt-8">My URLs</h2>
        <UrlList urls={urls} />
      </div>
    </main>
  );
}
```

---

### Feature 11: Update App Routes

Update `frontend/src/App.jsx`:

```jsx
import { BrowserRouter, Routes, Route, Link } from "react-router-dom";
import Home from "./pages/Home";
import Login from "./pages/Login";
import Signup from "./pages/Signup";
import Dashboard from "./pages/Dashboard";
import ProtectedRoute from "./components/ProtectedRoute";

function Navbar() {
  return (
    <nav className="bg-white border-b px-6 py-4 flex justify-between">
      <Link to="/" className="font-bold">URL Shortener</Link>
      <div className="flex gap-4">
        <Link to="/login">Login</Link>
        <Link to="/signup">Signup</Link>
        <Link to="/dashboard">Dashboard</Link>
      </div>
    </nav>
  );
}

export default function App() {
  return (
    <BrowserRouter>
      <Navbar />
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/login" element={<Login />} />
        <Route path="/signup" element={<Signup />} />
        <Route
          path="/dashboard"
          element={
            <ProtectedRoute>
              <Dashboard />
            </ProtectedRoute>
          }
        />
      </Routes>
    </BrowserRouter>
  );
}
```

---

## Backend Route Details

### `POST /api/urls/shorten`

Purpose:

Creates a short URL.

Request:

```json
{
  "originalUrl": "https://example.com/very/long/url"
}
```

Response:

```json
{
  "_id": "mongo_id",
  "originalUrl": "https://example.com/very/long/url",
  "shortCode": "xYz1234",
  "shortUrl": "http://localhost:5000/xYz1234",
  "clicks": 0
}
```

---

### `GET /:shortCode`

Purpose:

Redirects user to original URL.

Example:

```text
GET http://localhost:5000/xYz1234
```

Result:

Browser redirects to the original URL.

---

### `GET /api/urls/recent`

Purpose:

Returns recent public short URLs.

Response:

```json
[
  {
    "_id": "mongo_id",
    "originalUrl": "https://google.com",
    "shortUrl": "http://localhost:5000/abc1234",
    "clicks": 2
  }
]
```

---

### `POST /api/auth/signup`

Purpose:

Creates a new user.

Request:

```json
{
  "name": "Varna",
  "email": "varna@example.com",
  "password": "123456"
}
```

Response:

```json
{
  "token": "jwt_token_here",
  "user": {
    "id": "user_id",
    "name": "Varna",
    "email": "varna@example.com"
  }
}
```

---

### `POST /api/auth/login`

Purpose:

Logs in an existing user.

Request:

```json
{
  "email": "varna@example.com",
  "password": "123456"
}
```

Response:

```json
{
  "token": "jwt_token_here",
  "user": {
    "id": "user_id",
    "name": "Varna",
    "email": "varna@example.com"
  }
}
```

---

### `GET /api/urls/my-urls`

Purpose:

Returns URLs created by logged-in user.

Headers:

```text
Authorization: Bearer jwt_token_here
```

Response:

```json
[
  {
    "_id": "url_id",
    "originalUrl": "https://example.com",
    "shortUrl": "http://localhost:5000/abc1234",
    "clicks": 5
  }
]
```

---

## Running the Application

### Terminal 1: Backend

```bash
cd backend
npm run dev
```

Expected output:

```text
MongoDB connected
Server running on http://localhost:5000
```

### Terminal 2: Frontend

```bash
cd frontend
npm run dev
```

Expected output:

```text
Local: http://localhost:5173
```

Open:

```text
http://localhost:5173
```

---

## Testing the Application Manually

### Test MVP

1. Open frontend at `http://localhost:5173`.
2. Paste `https://www.google.com`.
3. Click Shorten URL.
4. Copy the short URL.
5. Open short URL in browser.
6. Confirm it redirects to Google.
7. Refresh frontend.
8. Confirm click count increased.

### Test Auth

1. Open `/signup`.
2. Create account.
3. Confirm redirect to dashboard.
4. Create private short URL.
5. Logout.
6. Login again.
7. Confirm your URLs still appear.

### Test Protected Route

1. Remove token from localStorage.
2. Open `/dashboard`.
3. Confirm you are redirected to `/login`.

---

## Key Concepts for Beginners

### REST API

A REST API is a set of backend URLs that the frontend can call.

Example:

```text
POST /api/urls/shorten
```

Means:

Create a new short URL.

---

### HTTP Methods

- `GET`: read data.
- `POST`: create data.
- `PUT/PATCH`: update data.
- `DELETE`: delete data.

---

### Request and Response

Request is what frontend sends.

Response is what backend sends back.

Example request:

```json
{
  "originalUrl": "https://google.com"
}
```

Example response:

```json
{
  "shortUrl": "http://localhost:5000/abc1234"
}
```

---

### Database Model

A model defines what data looks like in the database.

Example:

```js
originalUrl: String
shortCode: String
clicks: Number
```

This tells MongoDB how URL documents should be structured.

---

### Environment Variables

Environment variables store secret or environment-specific values.

Examples:

```env
MONGO_URI=...
JWT_SECRET=...
```

Why use them:

- Keep secrets out of code.
- Use different values for development and production.

---

### CORS

CORS allows frontend and backend on different ports to talk.

Frontend:

```text
http://localhost:5173
```

Backend:

```text
http://localhost:5000
```

Without `cors()`, browser may block requests.

---

### Middleware

Middleware runs before the final route handler.

Example:

```js
router.get("/my-urls", protect, getMyUrls);
```

Here:

- `protect` checks login.
- `getMyUrls` runs only if login is valid.

---

### JWT Authentication

JWT is a signed token that proves the user is logged in.

Flow:

```text
Login successful
Backend sends token
Frontend stores token
Frontend sends token in Authorization header
Backend verifies token
```

---

### State Management

React state stores changing UI data.

Examples:

- Form input
- Loading status
- Error messages
- URL list

---

### Error Handling

Good apps handle errors clearly.

Examples:

- Invalid URL
- Wrong password
- Database down
- Short code not found

---

## Common Errors and Solutions

### Server not running

Problem:

Frontend says API failed.

Solution:

Start backend:

```bash
cd backend
npm run dev
```

---

### Database connection error

Problem:

Backend says MongoDB connection failed.

Solution:

- Check `MONGO_URI`.
- Check MongoDB Atlas username/password.
- Check IP whitelist.
- Make sure password special characters are encoded.

---

### CORS error

Problem:

Browser blocks API request.

Solution:

Make sure backend has:

```js
app.use(cors());
```

---

### Invalid URL error

Problem:

You entered `google.com`, but validation fails.

Solution:

Use full URL:

```text
https://google.com
```

---

### Token auth error

Problem:

Dashboard says unauthorized.

Solution:

- Login again.
- Check localStorage has `token`.
- Check Axios interceptor adds Authorization header.
- Check `JWT_SECRET` is same while creating and verifying token.

---

### Frontend not updating after creating URL

Problem:

URL is created but list does not refresh.

Solution:

Make sure `UrlForm` calls:

```js
onCreated?.();
```

And `Home` or `Dashboard` passes:

```jsx
<UrlForm onCreated={fetchUrls} />
```

---

### Short URL route not working

Problem:

`http://localhost:5000/abc1234` does not redirect.

Solution:

Make sure backend has:

```js
app.use("/api/urls", urlRoutes);
app.get("/:shortCode", async (req, res, next) => {
  req.url = `/api/urls${req.url}`;
  next();
}, urlRoutes);
```

---

## Final Summary

You built a complete MERN URL shortener app.

You learned how to:

- Create an Express backend.
- Connect MongoDB with Mongoose.
- Design URL and User models.
- Create REST API routes.
- Validate data with Zod.
- Generate short codes using nanoid.
- Redirect short URLs.
- Track clicks.
- Build a React frontend.
- Connect frontend to backend using Axios.
- Add signup and login.
- Hash passwords with bcrypt.
- Protect routes with JWT middleware.
- Store JWT in localStorage.
- Build a private dashboard.

---

## Next Improvements

Useful features to add next:

1. Custom short aliases.
2. Delete short URLs.
3. Edit destination URL.
4. QR code generation.
5. Expiring links.
6. Public/private links.
7. Click analytics by date.
8. Device/browser analytics.
9. Rate limiting to prevent spam.
10. Deployment with Render, Vercel, and MongoDB Atlas.

