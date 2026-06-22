# Google Drive Clone - Complete MERN Full-Stack Guide

Project Name: Google Drive Clone  
Project Type: Cloud file storage app  
Tech Stack: React, Vite, Tailwind CSS, Node.js, Express.js, MongoDB, Mongoose, JWT Auth, Multer  
Skill Level: Beginner

This guide teaches a practical beginner version first. The first version stores uploaded files on the backend server's local disk. That is good for learning. In real production apps, files should be stored in object storage such as AWS S3, Cloudinary, Google Cloud Storage, or Azure Blob Storage.

The attached system-design note talks about large-scale Google Drive ideas such as block servers, resumable upload, delta sync, compression, encryption, metadata database, notifications, and cold storage. We will explain those concepts, but we will implement a beginner-friendly MERN version first.

---

## 1. Project Overview

### What The App Does

We will build a Google Drive-style web app where users can:

- Sign up and log in.
- Upload files.
- Create folders.
- View files and folders.
- Open folders.
- Download files.
- Rename files and folders.
- Move files and folders to trash.
- Restore deleted items.
- Permanently delete items.
- See basic file details such as size, type, owner, and upload date.

In the advanced version, we will also add:

- Share files with another user.
- Show "Shared with me".
- Upload a new version of a file.
- View file revision history.
- Star important files.
- Explain how to upgrade local disk storage to cloud storage.

### Main Features

- JWT authentication
- Protected backend APIs
- MongoDB metadata storage
- Local file upload with Multer
- Folder hierarchy
- File download
- Trash system
- Starred files
- Sharing by email
- File versions
- React dashboard UI
- Axios API client
- Frontend protected routes

### What The User Will Learn

By building this project, you will learn:

- How MERN apps are structured.
- How frontend and backend communicate.
- How file upload works in Express.
- Why uploaded files need metadata.
- How folders can be represented in a database.
- How to protect APIs using JWT.
- How to store auth tokens in the frontend.
- How to build reusable React components.
- How Google Drive-like apps separate file metadata from file storage.

---

## 2. Folder Structure

Recommended project structure:

```text
google-drive-clone-mern/
|-- backend/
|   |-- src/
|   |   |-- config/
|   |   |   |-- db.js
|   |   |-- controllers/
|   |   |   |-- authController.js
|   |   |   |-- driveController.js
|   |   |-- middleware/
|   |   |   |-- authMiddleware.js
|   |   |   |-- errorMiddleware.js
|   |   |   |-- uploadMiddleware.js
|   |   |-- models/
|   |   |   |-- User.js
|   |   |   |-- FileItem.js
|   |   |-- routes/
|   |   |   |-- authRoutes.js
|   |   |   |-- driveRoutes.js
|   |   |-- utils/
|   |   |   |-- generateToken.js
|   |   |   |-- formatBytes.js
|   |   |-- server.js
|   |-- uploads/
|   |-- .env
|   |-- package.json
|
|-- frontend/
|   |-- src/
|   |   |-- api/
|   |   |   |-- axios.js
|   |   |-- components/
|   |   |   |-- FileTable.jsx
|   |   |   |-- Navbar.jsx
|   |   |   |-- ProtectedRoute.jsx
|   |   |   |-- UploadBox.jsx
|   |   |-- context/
|   |   |   |-- AuthContext.jsx
|   |   |-- pages/
|   |   |   |-- Dashboard.jsx
|   |   |   |-- Login.jsx
|   |   |   |-- Signup.jsx
|   |   |   |-- SharedWithMe.jsx
|   |   |   |-- Starred.jsx
|   |   |   |-- Trash.jsx
|   |   |-- App.jsx
|   |   |-- main.jsx
|   |   |-- index.css
|   |-- package.json
```

### What Each Folder Does

- `backend/src/config`: database connection code.
- `backend/src/models`: MongoDB schemas.
- `backend/src/controllers`: actual route logic.
- `backend/src/routes`: route URL definitions.
- `backend/src/middleware`: reusable functions that run before controllers.
- `backend/uploads`: local storage folder for uploaded files.
- `frontend/src/api`: Axios backend connection.
- `frontend/src/context`: global authentication state.
- `frontend/src/components`: reusable UI pieces.
- `frontend/src/pages`: full screens/pages.

---

## 3. Setup Instructions

### Prerequisites

Install:

- Node.js
- npm
- MongoDB Atlas account
- VS Code
- Postman or Thunder Client

Check versions:

```bash
node -v
npm -v
```

Create project folder:

```bash
mkdir google-drive-clone-mern
cd google-drive-clone-mern
mkdir backend frontend
```

---

### Initialize Backend

```bash
cd backend
npm init -y
```

Install backend dependencies:

```bash
npm install express mongoose cors dotenv bcryptjs jsonwebtoken multer zod nanoid morgan
npm install -D nodemon
```

Update `backend/package.json`:

```json
{
  "name": "google-drive-clone-backend",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "nodemon src/server.js",
    "start": "node src/server.js"
  }
}
```

Why we use `"type": "module"`:

It allows modern JavaScript imports:

```js
import express from "express";
```

---

### Configure Environment Variables

Create `backend/.env`:

```env
PORT=5000
MONGO_URI=your_mongodb_atlas_connection_string
JWT_SECRET=your_super_secret_jwt_key
CLIENT_URL=http://localhost:5173
MAX_FILE_SIZE_MB=50
```

Explanation:

- `PORT`: backend server port.
- `MONGO_URI`: MongoDB connection string.
- `JWT_SECRET`: secret key used to sign JWT tokens.
- `CLIENT_URL`: frontend URL allowed by CORS.
- `MAX_FILE_SIZE_MB`: upload size limit for the beginner version.

Important:

Never upload `.env` to GitHub.

---

### Initialize Frontend

Open a new terminal:

```bash
cd google-drive-clone-mern/frontend
npm create vite@latest . -- --template react
npm install
```

Install frontend dependencies:

```bash
npm install axios react-router-dom lucide-react
```

Install Tailwind CSS:

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

body {
  margin: 0;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  background: #f8fafc;
}
```

---

## 4. Database Design

Google Drive has two big parts:

1. File metadata
2. Actual file content

Metadata means information about the file:

- File name
- File size
- MIME type
- Owner
- Parent folder
- Upload date
- Storage path

Actual file content means the real uploaded file bytes.

In our beginner version:

- Metadata goes to MongoDB.
- Actual files go to `backend/uploads`.

In production:

- Metadata still goes to a database.
- Actual files go to S3 or another object storage service.

---

### User Model

Each user can own many files and folders.

Fields:

- `name`: user's display name.
- `email`: user's login email.
- `password`: hashed password.
- `storageUsed`: total storage used by user.
- `storageLimit`: user's allowed storage limit.

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
    storageUsed: {
      type: Number,
      default: 0,
    },
    storageLimit: {
      type: Number,
      default: 1024 * 1024 * 1024,
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

Why password hashing matters:

We never store the real password. If the database leaks, hashed passwords are much harder to abuse than plain text passwords.

---

### FileItem Model

We will use one model for both files and folders.

Why?

Because files and folders share many fields:

- name
- owner
- parent
- created date
- deleted state

Create `backend/src/models/FileItem.js`:

```js
import mongoose from "mongoose";

const versionSchema = new mongoose.Schema(
  {
    versionNumber: Number,
    storagePath: String,
    size: Number,
    uploadedAt: {
      type: Date,
      default: Date.now,
    },
  },
  { _id: false }
);

const sharedUserSchema = new mongoose.Schema(
  {
    user: {
      type: mongoose.Schema.Types.ObjectId,
      ref: "User",
    },
    permission: {
      type: String,
      enum: ["viewer", "editor"],
      default: "viewer",
    },
  },
  { _id: false }
);

const fileItemSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: true,
      trim: true,
    },
    type: {
      type: String,
      enum: ["file", "folder"],
      required: true,
    },
    mimeType: {
      type: String,
      default: null,
    },
    size: {
      type: Number,
      default: 0,
    },
    owner: {
      type: mongoose.Schema.Types.ObjectId,
      ref: "User",
      required: true,
    },
    parent: {
      type: mongoose.Schema.Types.ObjectId,
      ref: "FileItem",
      default: null,
    },
    storagePath: {
      type: String,
      default: null,
    },
    isStarred: {
      type: Boolean,
      default: false,
    },
    isDeleted: {
      type: Boolean,
      default: false,
    },
    deletedAt: {
      type: Date,
      default: null,
    },
    sharedWith: [sharedUserSchema],
    currentVersion: {
      type: Number,
      default: 1,
    },
    versions: [versionSchema],
  },
  { timestamps: true }
);

fileItemSchema.index({ owner: 1, parent: 1, name: 1 });
fileItemSchema.index({ name: "text" });

export const FileItem = mongoose.model("FileItem", fileItemSchema);
```

Important fields:

- `type`: tells if item is file or folder.
- `parent`: stores folder hierarchy. If `parent` is `null`, item is in root.
- `storagePath`: path on server where file is stored.
- `sharedWith`: list of users who can access the item.
- `versions`: history of uploaded versions.
- `isDeleted`: soft delete flag for trash.

Folder relationship:

```text
Root
|-- Documents folder
|   |-- resume.pdf
|-- Photos folder
|   |-- image.png
```

In MongoDB:

```text
Documents.parent = null
resume.pdf parent = Documents._id
Photos.parent = null
image.png parent = Photos._id
```

---

## 5. Backend Development

---

### Create Database Connection

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

Explanation:

- `mongoose.connect` connects to MongoDB.
- `process.env.MONGO_URI` reads the connection string.
- `process.exit(1)` stops the server if database connection fails.

---

### Generate JWT Token

Create `backend/src/utils/generateToken.js`:

```js
import jwt from "jsonwebtoken";

export const generateToken = (userId) => {
  return jwt.sign({ userId }, process.env.JWT_SECRET, {
    expiresIn: "7d",
  });
};
```

JWT means JSON Web Token.

It is a signed string that proves a user is logged in.

---

### Auth Middleware

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

What happens here:

1. Read `Authorization` header.
2. Extract token from `Bearer token`.
3. Verify token.
4. Find user in database.
5. Attach user to `req.user`.
6. Continue to controller.

---

### Upload Middleware

Multer handles multipart file upload.

Create `backend/src/middleware/uploadMiddleware.js`:

```js
import fs from "fs";
import path from "path";
import multer from "multer";
import { nanoid } from "nanoid";

const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    const userId = req.user._id.toString();
    const uploadDir = path.join(process.cwd(), "uploads", userId);

    fs.mkdirSync(uploadDir, { recursive: true });
    cb(null, uploadDir);
  },
  filename: (req, file, cb) => {
    const extension = path.extname(file.originalname);
    const uniqueName = `${Date.now()}-${nanoid(10)}${extension}`;
    cb(null, uniqueName);
  },
});

export const upload = multer({
  storage,
  limits: {
    fileSize: Number(process.env.MAX_FILE_SIZE_MB || 50) * 1024 * 1024,
  },
});
```

Why we create unique names:

Two users may upload files with the same name. Unique disk names prevent overwriting files.

Example:

```text
resume.pdf
```

becomes:

```text
1712345678-AbCdEf1234.pdf
```

The user still sees `resume.pdf` because the original display name is stored in MongoDB.

---

### Error Middleware

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

Why this helps:

It gives consistent error responses instead of crashing the server.

---

### Auth Controller

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
        storageUsed: user.storageUsed,
        storageLimit: user.storageLimit,
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
        storageUsed: user.storageUsed,
        storageLimit: user.storageLimit,
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

Major line explanations:

- `safeParse`: validates request body without throwing.
- `User.findOne({ email })`: checks duplicate account.
- `User.create`: saves user. Password is hashed by the model hook.
- `generateToken(user._id)`: creates login token.

---

### Auth Routes

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

Routes:

| Method | URL | Purpose |
|---|---|---|
| `POST` | `/api/auth/signup` | Create account |
| `POST` | `/api/auth/login` | Login |
| `GET` | `/api/auth/me` | Get current user |

---

### Drive Controller

Create `backend/src/controllers/driveController.js`:

```js
import fs from "fs";
import path from "path";
import { z } from "zod";
import { FileItem } from "../models/FileItem.js";
import { User } from "../models/User.js";

const objectIdSchema = z.string().regex(/^[0-9a-fA-F]{24}$/, "Invalid id");

const createFolderSchema = z.object({
  name: z.string().min(1, "Folder name is required"),
  parent: z.string().optional().nullable(),
});

const renameSchema = z.object({
  name: z.string().min(1, "Name is required"),
});

const shareSchema = z.object({
  email: z.string().email("Invalid email"),
  permission: z.enum(["viewer", "editor"]).default("viewer"),
});

const getParentValue = (value) => {
  if (!value || value === "null" || value === "undefined") {
    return null;
  }

  return value;
};

const canAccessItem = (item, userId) => {
  const isOwner = item.owner.toString() === userId.toString();
  const isShared = item.sharedWith.some(
    (entry) => entry.user.toString() === userId.toString()
  );

  return isOwner || isShared;
};

export const createFolder = async (req, res) => {
  try {
    const parsed = createFolderSchema.safeParse(req.body);

    if (!parsed.success) {
      return res.status(400).json({
        message: "Validation failed",
        errors: parsed.error.flatten(),
      });
    }

    const parent = getParentValue(parsed.data.parent);

    if (parent) {
      const parentFolder = await FileItem.findOne({
        _id: parent,
        owner: req.user._id,
        type: "folder",
        isDeleted: false,
      });

      if (!parentFolder) {
        return res.status(404).json({ message: "Parent folder not found" });
      }
    }

    const folder = await FileItem.create({
      name: parsed.data.name,
      type: "folder",
      owner: req.user._id,
      parent,
    });

    res.status(201).json(folder);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const uploadFile = async (req, res) => {
  try {
    if (!req.file) {
      return res.status(400).json({ message: "No file uploaded" });
    }

    const parent = getParentValue(req.body.parent);

    if (parent) {
      const parentFolder = await FileItem.findOne({
        _id: parent,
        owner: req.user._id,
        type: "folder",
        isDeleted: false,
      });

      if (!parentFolder) {
        return res.status(404).json({ message: "Parent folder not found" });
      }
    }

    if (req.user.storageUsed + req.file.size > req.user.storageLimit) {
      fs.unlinkSync(req.file.path);
      return res.status(400).json({ message: "Storage limit exceeded" });
    }

    const fileItem = await FileItem.create({
      name: req.file.originalname,
      type: "file",
      mimeType: req.file.mimetype,
      size: req.file.size,
      owner: req.user._id,
      parent,
      storagePath: req.file.path,
      versions: [
        {
          versionNumber: 1,
          storagePath: req.file.path,
          size: req.file.size,
        },
      ],
    });

    await User.findByIdAndUpdate(req.user._id, {
      $inc: { storageUsed: req.file.size },
    });

    res.status(201).json(fileItem);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const getItems = async (req, res) => {
  try {
    const parent = getParentValue(req.query.parent);

    const items = await FileItem.find({
      owner: req.user._id,
      parent,
      isDeleted: false,
    }).sort({ type: -1, name: 1 });

    res.json(items);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const getStarredItems = async (req, res) => {
  try {
    const items = await FileItem.find({
      owner: req.user._id,
      isStarred: true,
      isDeleted: false,
    }).sort({ updatedAt: -1 });

    res.json(items);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const getTrashItems = async (req, res) => {
  try {
    const items = await FileItem.find({
      owner: req.user._id,
      isDeleted: true,
    }).sort({ deletedAt: -1 });

    res.json(items);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const downloadFile = async (req, res) => {
  try {
    const parsedId = objectIdSchema.safeParse(req.params.id);

    if (!parsedId.success) {
      return res.status(400).json({ message: "Invalid file id" });
    }

    const item = await FileItem.findById(req.params.id);

    if (!item || item.type !== "file" || item.isDeleted) {
      return res.status(404).json({ message: "File not found" });
    }

    if (!canAccessItem(item, req.user._id)) {
      return res.status(403).json({ message: "Access denied" });
    }

    const absolutePath = path.resolve(item.storagePath);

    if (!fs.existsSync(absolutePath)) {
      return res.status(404).json({ message: "Stored file missing" });
    }

    res.download(absolutePath, item.name);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const renameItem = async (req, res) => {
  try {
    const parsed = renameSchema.safeParse(req.body);

    if (!parsed.success) {
      return res.status(400).json({
        message: "Validation failed",
        errors: parsed.error.flatten(),
      });
    }

    const item = await FileItem.findOneAndUpdate(
      {
        _id: req.params.id,
        owner: req.user._id,
        isDeleted: false,
      },
      { name: parsed.data.name },
      { new: true }
    );

    if (!item) {
      return res.status(404).json({ message: "Item not found" });
    }

    res.json(item);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const toggleStar = async (req, res) => {
  try {
    const item = await FileItem.findOne({
      _id: req.params.id,
      owner: req.user._id,
      isDeleted: false,
    });

    if (!item) {
      return res.status(404).json({ message: "Item not found" });
    }

    item.isStarred = !item.isStarred;
    await item.save();

    res.json(item);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const moveToTrash = async (req, res) => {
  try {
    const item = await FileItem.findOneAndUpdate(
      {
        _id: req.params.id,
        owner: req.user._id,
        isDeleted: false,
      },
      {
        isDeleted: true,
        deletedAt: new Date(),
      },
      { new: true }
    );

    if (!item) {
      return res.status(404).json({ message: "Item not found" });
    }

    res.json({ message: "Moved to trash", item });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const restoreItem = async (req, res) => {
  try {
    const item = await FileItem.findOneAndUpdate(
      {
        _id: req.params.id,
        owner: req.user._id,
        isDeleted: true,
      },
      {
        isDeleted: false,
        deletedAt: null,
      },
      { new: true }
    );

    if (!item) {
      return res.status(404).json({ message: "Trash item not found" });
    }

    res.json(item);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const permanentDelete = async (req, res) => {
  try {
    const item = await FileItem.findOne({
      _id: req.params.id,
      owner: req.user._id,
      isDeleted: true,
    });

    if (!item) {
      return res.status(404).json({ message: "Trash item not found" });
    }

    if (item.type === "file" && item.storagePath && fs.existsSync(item.storagePath)) {
      fs.unlinkSync(item.storagePath);
    }

    await FileItem.deleteOne({ _id: item._id });

    if (item.type === "file") {
      await User.findByIdAndUpdate(req.user._id, {
        $inc: { storageUsed: -item.size },
      });
    }

    res.json({ message: "Permanently deleted" });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const shareItem = async (req, res) => {
  try {
    const parsed = shareSchema.safeParse(req.body);

    if (!parsed.success) {
      return res.status(400).json({
        message: "Validation failed",
        errors: parsed.error.flatten(),
      });
    }

    const targetUser = await User.findOne({ email: parsed.data.email });

    if (!targetUser) {
      return res.status(404).json({ message: "User not found" });
    }

    if (targetUser._id.toString() === req.user._id.toString()) {
      return res.status(400).json({ message: "You cannot share with yourself" });
    }

    const item = await FileItem.findOne({
      _id: req.params.id,
      owner: req.user._id,
      isDeleted: false,
    });

    if (!item) {
      return res.status(404).json({ message: "Item not found" });
    }

    const alreadyShared = item.sharedWith.some(
      (entry) => entry.user.toString() === targetUser._id.toString()
    );

    if (!alreadyShared) {
      item.sharedWith.push({
        user: targetUser._id,
        permission: parsed.data.permission,
      });
    }

    await item.save();
    res.json(item);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const getSharedWithMe = async (req, res) => {
  try {
    const items = await FileItem.find({
      "sharedWith.user": req.user._id,
      isDeleted: false,
    })
      .populate("owner", "name email")
      .sort({ updatedAt: -1 });

    res.json(items);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const uploadNewVersion = async (req, res) => {
  try {
    if (!req.file) {
      return res.status(400).json({ message: "No file uploaded" });
    }

    const item = await FileItem.findOne({
      _id: req.params.id,
      owner: req.user._id,
      type: "file",
      isDeleted: false,
    });

    if (!item) {
      fs.unlinkSync(req.file.path);
      return res.status(404).json({ message: "File not found" });
    }

    item.currentVersion += 1;
    item.storagePath = req.file.path;
    item.size = req.file.size;
    item.mimeType = req.file.mimetype;
    item.versions.push({
      versionNumber: item.currentVersion,
      storagePath: req.file.path,
      size: req.file.size,
    });

    await item.save();
    res.json(item);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const getVersions = async (req, res) => {
  try {
    const item = await FileItem.findOne({
      _id: req.params.id,
      owner: req.user._id,
      type: "file",
      isDeleted: false,
    });

    if (!item) {
      return res.status(404).json({ message: "File not found" });
    }

    res.json(item.versions);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};
```

This controller is the main backend logic for Drive features.

Important beginner concepts:

- We check ownership before editing or deleting.
- We do not trust frontend data.
- We validate IDs and body data.
- We store metadata in MongoDB.
- We store physical files on disk.
- We use soft delete for trash.

---

### Drive Routes

Create `backend/src/routes/driveRoutes.js`:

```js
import express from "express";
import {
  createFolder,
  downloadFile,
  getItems,
  getSharedWithMe,
  getStarredItems,
  getTrashItems,
  getVersions,
  moveToTrash,
  permanentDelete,
  renameItem,
  restoreItem,
  shareItem,
  toggleStar,
  uploadFile,
  uploadNewVersion,
} from "../controllers/driveController.js";
import { protect } from "../middleware/authMiddleware.js";
import { upload } from "../middleware/uploadMiddleware.js";

const router = express.Router();

router.use(protect);

router.get("/", getItems);
router.get("/starred", getStarredItems);
router.get("/trash", getTrashItems);
router.get("/shared-with-me", getSharedWithMe);

router.post("/folders", createFolder);
router.post("/upload", upload.single("file"), uploadFile);

router.get("/:id/download", downloadFile);
router.get("/:id/versions", getVersions);
router.patch("/:id/rename", renameItem);
router.patch("/:id/star", toggleStar);
router.patch("/:id/restore", restoreItem);
router.patch("/:id/share", shareItem);
router.post("/:id/versions", upload.single("file"), uploadNewVersion);
router.delete("/:id", moveToTrash);
router.delete("/:id/permanent", permanentDelete);

export default router;
```

Why `router.use(protect)`:

Every Drive route requires login.

---

### Server Setup

Create `backend/src/server.js`:

```js
import express from "express";
import cors from "cors";
import dotenv from "dotenv";
import morgan from "morgan";
import { connectDB } from "./config/db.js";
import authRoutes from "./routes/authRoutes.js";
import driveRoutes from "./routes/driveRoutes.js";
import { errorHandler, notFound } from "./middleware/errorMiddleware.js";

dotenv.config();

const app = express();

connectDB();

app.use(cors({
  origin: process.env.CLIENT_URL,
  credentials: true,
}));

app.use(express.json());
app.use(morgan("dev"));

app.get("/", (req, res) => {
  res.json({ message: "Google Drive Clone API is running" });
});

app.use("/api/auth", authRoutes);
app.use("/api/drive", driveRoutes);

app.use(notFound);
app.use(errorHandler);

const PORT = process.env.PORT || 5000;

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

How to test:

```bash
npm run dev
```

Expected output:

```text
MongoDB connected
Server running on http://localhost:5000
```

---

## Backend API Routes

### Auth Routes

| Method | URL | Purpose |
|---|---|---|
| `POST` | `/api/auth/signup` | Create account |
| `POST` | `/api/auth/login` | Login |
| `GET` | `/api/auth/me` | Get logged-in user |

### Drive Routes

| Method | URL | Purpose |
|---|---|---|
| `GET` | `/api/drive?parent=null` | List root files/folders |
| `GET` | `/api/drive?parent=folderId` | List folder contents |
| `POST` | `/api/drive/folders` | Create folder |
| `POST` | `/api/drive/upload` | Upload file |
| `GET` | `/api/drive/:id/download` | Download file |
| `PATCH` | `/api/drive/:id/rename` | Rename file/folder |
| `PATCH` | `/api/drive/:id/star` | Star/unstar item |
| `DELETE` | `/api/drive/:id` | Move item to trash |
| `GET` | `/api/drive/trash` | List trash |
| `PATCH` | `/api/drive/:id/restore` | Restore trash item |
| `DELETE` | `/api/drive/:id/permanent` | Permanently delete |
| `PATCH` | `/api/drive/:id/share` | Share with another user |
| `GET` | `/api/drive/shared-with-me` | Items shared with current user |
| `POST` | `/api/drive/:id/versions` | Upload new version |
| `GET` | `/api/drive/:id/versions` | List versions |

---

## 6. Frontend Development

---

### Axios Setup

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

- We write backend URL once.
- Every request automatically gets JWT token.

---

### Auth Context

Create `frontend/src/context/AuthContext.jsx`:

```jsx
import { createContext, useContext, useEffect, useState } from "react";
import api from "../api/axios";

const AuthContext = createContext(null);

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  const loadUser = async () => {
    const token = localStorage.getItem("token");

    if (!token) {
      setLoading(false);
      return;
    }

    try {
      const res = await api.get("/auth/me");
      setUser(res.data);
    } catch {
      localStorage.removeItem("token");
      setUser(null);
    } finally {
      setLoading(false);
    }
  };

  const login = (token, userData) => {
    localStorage.setItem("token", token);
    setUser(userData);
  };

  const logout = () => {
    localStorage.removeItem("token");
    setUser(null);
  };

  useEffect(() => {
    loadUser();
  }, []);

  return (
    <AuthContext.Provider value={{ user, loading, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => useContext(AuthContext);
```

Why context:

Many pages need to know if the user is logged in. Context avoids passing user data manually through every component.

---

### Protected Route

Create `frontend/src/components/ProtectedRoute.jsx`:

```jsx
import { Navigate } from "react-router-dom";
import { useAuth } from "../context/AuthContext";

export default function ProtectedRoute({ children }) {
  const { user, loading } = useAuth();

  if (loading) {
    return <div className="p-6">Loading...</div>;
  }

  if (!user) {
    return <Navigate to="/login" replace />;
  }

  return children;
}
```

Purpose:

Only logged-in users can open dashboard, starred, shared, and trash pages.

---

### Navbar

Create `frontend/src/components/Navbar.jsx`:

```jsx
import { Link, useNavigate } from "react-router-dom";
import { HardDrive, LogOut } from "lucide-react";
import { useAuth } from "../context/AuthContext";

export default function Navbar() {
  const navigate = useNavigate();
  const { user, logout } = useAuth();

  const handleLogout = () => {
    logout();
    navigate("/login");
  };

  return (
    <nav className="h-16 border-b bg-white px-6 flex items-center justify-between">
      <Link to="/" className="flex items-center gap-2 font-bold text-xl">
        <HardDrive className="text-blue-600" />
        Drive Clone
      </Link>

      {user ? (
        <div className="flex items-center gap-4">
          <span className="text-sm text-gray-600">{user.email}</span>
          <button
            onClick={handleLogout}
            className="flex items-center gap-2 text-sm bg-gray-900 text-white px-3 py-2 rounded-md"
          >
            <LogOut size={16} />
            Logout
          </button>
        </div>
      ) : (
        <div className="flex gap-4">
          <Link to="/login">Login</Link>
          <Link to="/signup">Signup</Link>
        </div>
      )}
    </nav>
  );
}
```

---

### Signup Page

Create `frontend/src/pages/Signup.jsx`:

```jsx
import { useState } from "react";
import { Link, useNavigate } from "react-router-dom";
import api from "../api/axios";
import { useAuth } from "../context/AuthContext";

export default function Signup() {
  const navigate = useNavigate();
  const { login } = useAuth();

  const [form, setForm] = useState({
    name: "",
    email: "",
    password: "",
  });

  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError("");

    try {
      setLoading(true);
      const res = await api.post("/auth/signup", form);
      login(res.data.token, res.data.user);
      navigate("/");
    } catch (err) {
      setError(err.response?.data?.message || "Signup failed");
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="min-h-[calc(100vh-4rem)] flex items-center justify-center px-4">
      <form onSubmit={handleSubmit} className="bg-white border rounded-lg p-6 w-full max-w-md space-y-4">
        <h1 className="text-2xl font-bold">Create your account</h1>

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
          Already have an account? <Link to="/login" className="text-blue-600">Login</Link>
        </p>
      </form>
    </div>
  );
}
```

---

### Login Page

Create `frontend/src/pages/Login.jsx`:

```jsx
import { useState } from "react";
import { Link, useNavigate } from "react-router-dom";
import api from "../api/axios";
import { useAuth } from "../context/AuthContext";

export default function Login() {
  const navigate = useNavigate();
  const { login } = useAuth();

  const [form, setForm] = useState({
    email: "",
    password: "",
  });

  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError("");

    try {
      setLoading(true);
      const res = await api.post("/auth/login", form);
      login(res.data.token, res.data.user);
      navigate("/");
    } catch (err) {
      setError(err.response?.data?.message || "Login failed");
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="min-h-[calc(100vh-4rem)] flex items-center justify-center px-4">
      <form onSubmit={handleSubmit} className="bg-white border rounded-lg p-6 w-full max-w-md space-y-4">
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
          New here? <Link to="/signup" className="text-blue-600">Create account</Link>
        </p>
      </form>
    </div>
  );
}
```

---

### UploadBox Component

Create `frontend/src/components/UploadBox.jsx`:

```jsx
import { useState } from "react";
import api from "../api/axios";

export default function UploadBox({ parent, onUploaded }) {
  const [file, setFile] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState("");

  const handleUpload = async (e) => {
    e.preventDefault();
    setError("");

    if (!file) {
      setError("Please choose a file");
      return;
    }

    const formData = new FormData();
    formData.append("file", file);
    formData.append("parent", parent || "");

    try {
      setLoading(true);
      await api.post("/drive/upload", formData, {
        headers: {
          "Content-Type": "multipart/form-data",
        },
      });
      setFile(null);
      onUploaded();
    } catch (err) {
      setError(err.response?.data?.message || "Upload failed");
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleUpload} className="bg-white border rounded-lg p-4 space-y-3">
      <input
        type="file"
        onChange={(e) => setFile(e.target.files[0])}
        className="block w-full text-sm"
      />

      {error && <p className="text-sm text-red-600">{error}</p>}

      <button className="bg-blue-600 text-white px-4 py-2 rounded-md">
        {loading ? "Uploading..." : "Upload File"}
      </button>
    </form>
  );
}
```

Why FormData:

Normal JSON cannot send files. File uploads use `multipart/form-data`.

---

### FileTable Component

Create `frontend/src/components/FileTable.jsx`:

```jsx
import { File, Folder, MoreVertical, Star } from "lucide-react";

const formatBytes = (bytes) => {
  if (!bytes) return "-";

  const units = ["B", "KB", "MB", "GB"];
  let size = bytes;
  let unitIndex = 0;

  while (size >= 1024 && unitIndex < units.length - 1) {
    size = size / 1024;
    unitIndex++;
  }

  return `${size.toFixed(1)} ${units[unitIndex]}`;
};

export default function FileTable({
  items,
  onOpenFolder,
  onDownload,
  onRename,
  onDelete,
  onStar,
  onShare,
}) {
  if (!items.length) {
    return (
      <div className="bg-white border rounded-lg p-8 text-center text-gray-500">
        This folder is empty.
      </div>
    );
  }

  return (
    <div className="bg-white border rounded-lg overflow-hidden">
      <table className="w-full text-sm">
        <thead className="bg-gray-50 border-b">
          <tr>
            <th className="text-left p-3">Name</th>
            <th className="text-left p-3">Type</th>
            <th className="text-left p-3">Size</th>
            <th className="text-left p-3">Modified</th>
            <th className="text-right p-3">Actions</th>
          </tr>
        </thead>

        <tbody>
          {items.map((item) => (
            <tr key={item._id} className="border-b last:border-b-0 hover:bg-gray-50">
              <td className="p-3">
                <button
                  onClick={() => item.type === "folder" && onOpenFolder(item)}
                  className="flex items-center gap-2"
                >
                  {item.type === "folder" ? (
                    <Folder className="text-yellow-500" size={20} />
                  ) : (
                    <File className="text-blue-500" size={20} />
                  )}
                  <span className="font-medium">{item.name}</span>
                  {item.isStarred && <Star size={14} className="text-yellow-500 fill-yellow-500" />}
                </button>
              </td>

              <td className="p-3 capitalize">{item.type}</td>
              <td className="p-3">{formatBytes(item.size)}</td>
              <td className="p-3">{new Date(item.updatedAt).toLocaleString()}</td>

              <td className="p-3">
                <div className="flex justify-end gap-2">
                  {item.type === "file" && (
                    <button onClick={() => onDownload(item)} className="text-blue-600">
                      Download
                    </button>
                  )}
                  <button onClick={() => onStar(item)}>Star</button>
                  <button onClick={() => onRename(item)}>Rename</button>
                  <button onClick={() => onShare(item)}>Share</button>
                  <button onClick={() => onDelete(item)} className="text-red-600">
                    Delete
                  </button>
                  <MoreVertical size={18} />
                </div>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

---

### Dashboard Page

Create `frontend/src/pages/Dashboard.jsx`:

```jsx
import { useEffect, useState } from "react";
import api from "../api/axios";
import FileTable from "../components/FileTable";
import UploadBox from "../components/UploadBox";

export default function Dashboard() {
  const [items, setItems] = useState([]);
  const [parent, setParent] = useState(null);
  const [folderStack, setFolderStack] = useState([]);
  const [folderName, setFolderName] = useState("");
  const [error, setError] = useState("");

  const fetchItems = async () => {
    const res = await api.get(`/drive?parent=${parent || ""}`);
    setItems(res.data);
  };

  const createFolder = async (e) => {
    e.preventDefault();
    setError("");

    if (!folderName.trim()) {
      setError("Folder name is required");
      return;
    }

    try {
      await api.post("/drive/folders", {
        name: folderName,
        parent,
      });
      setFolderName("");
      fetchItems();
    } catch (err) {
      setError(err.response?.data?.message || "Failed to create folder");
    }
  };

  const openFolder = (folder) => {
    setFolderStack([...folderStack, folder]);
    setParent(folder._id);
  };

  const goBack = () => {
    const updatedStack = [...folderStack];
    updatedStack.pop();
    const previousFolder = updatedStack[updatedStack.length - 1];

    setFolderStack(updatedStack);
    setParent(previousFolder?._id || null);
  };

  const downloadFile = async (item) => {
    const res = await api.get(`/drive/${item._id}/download`, {
      responseType: "blob",
    });

    const url = window.URL.createObjectURL(new Blob([res.data]));
    const link = document.createElement("a");
    link.href = url;
    link.setAttribute("download", item.name);
    document.body.appendChild(link);
    link.click();
    link.remove();
  };

  const renameItem = async (item) => {
    const name = prompt("Enter new name", item.name);

    if (!name) return;

    await api.patch(`/drive/${item._id}/rename`, { name });
    fetchItems();
  };

  const deleteItem = async (item) => {
    const confirmed = confirm(`Move "${item.name}" to trash?`);

    if (!confirmed) return;

    await api.delete(`/drive/${item._id}`);
    fetchItems();
  };

  const starItem = async (item) => {
    await api.patch(`/drive/${item._id}/star`);
    fetchItems();
  };

  const shareItem = async (item) => {
    const email = prompt("Enter user email to share with");

    if (!email) return;

    await api.patch(`/drive/${item._id}/share`, {
      email,
      permission: "viewer",
    });

    alert("Shared successfully");
  };

  useEffect(() => {
    fetchItems();
  }, [parent]);

  return (
    <main className="p-6 max-w-6xl mx-auto space-y-6">
      <div>
        <h1 className="text-3xl font-bold">My Drive</h1>
        <p className="text-gray-600">Upload, organize, and manage your files.</p>
      </div>

      <div className="flex gap-3 text-sm">
        <button onClick={() => { setParent(null); setFolderStack([]); }} className="text-blue-600">
          My Drive
        </button>

        {folderStack.map((folder) => (
          <span key={folder._id}>/ {folder.name}</span>
        ))}
      </div>

      {folderStack.length > 0 && (
        <button onClick={goBack} className="bg-gray-200 px-3 py-2 rounded-md">
          Back
        </button>
      )}

      <div className="grid md:grid-cols-2 gap-4">
        <UploadBox parent={parent} onUploaded={fetchItems} />

        <form onSubmit={createFolder} className="bg-white border rounded-lg p-4 space-y-3">
          <input
            value={folderName}
            onChange={(e) => setFolderName(e.target.value)}
            placeholder="New folder name"
            className="w-full border rounded-md px-3 py-2"
          />

          {error && <p className="text-red-600 text-sm">{error}</p>}

          <button className="bg-gray-900 text-white px-4 py-2 rounded-md">
            Create Folder
          </button>
        </form>
      </div>

      <FileTable
        items={items}
        onOpenFolder={openFolder}
        onDownload={downloadFile}
        onRename={renameItem}
        onDelete={deleteItem}
        onStar={starItem}
        onShare={shareItem}
      />
    </main>
  );
}
```

Important frontend data flow:

```text
User clicks upload
UploadBox sends FormData to backend
Backend stores file and metadata
Dashboard refetches items
FileTable shows new file
```

---

### Trash Page

Create `frontend/src/pages/Trash.jsx`:

```jsx
import { useEffect, useState } from "react";
import api from "../api/axios";

export default function Trash() {
  const [items, setItems] = useState([]);

  const fetchTrash = async () => {
    const res = await api.get("/drive/trash");
    setItems(res.data);
  };

  const restoreItem = async (item) => {
    await api.patch(`/drive/${item._id}/restore`);
    fetchTrash();
  };

  const deleteForever = async (item) => {
    const confirmed = confirm(`Permanently delete "${item.name}"?`);

    if (!confirmed) return;

    await api.delete(`/drive/${item._id}/permanent`);
    fetchTrash();
  };

  useEffect(() => {
    fetchTrash();
  }, []);

  return (
    <main className="p-6 max-w-5xl mx-auto">
      <h1 className="text-3xl font-bold mb-6">Trash</h1>

      <div className="bg-white border rounded-lg divide-y">
        {items.map((item) => (
          <div key={item._id} className="p-4 flex justify-between">
            <div>
              <p className="font-medium">{item.name}</p>
              <p className="text-sm text-gray-500">
                Deleted {new Date(item.deletedAt).toLocaleString()}
              </p>
            </div>

            <div className="flex gap-3">
              <button onClick={() => restoreItem(item)} className="text-blue-600">
                Restore
              </button>
              <button onClick={() => deleteForever(item)} className="text-red-600">
                Delete forever
              </button>
            </div>
          </div>
        ))}

        {items.length === 0 && (
          <p className="p-6 text-gray-500 text-center">Trash is empty.</p>
        )}
      </div>
    </main>
  );
}
```

---

### Shared With Me Page

Create `frontend/src/pages/SharedWithMe.jsx`:

```jsx
import { useEffect, useState } from "react";
import api from "../api/axios";

export default function SharedWithMe() {
  const [items, setItems] = useState([]);

  useEffect(() => {
    api.get("/drive/shared-with-me").then((res) => setItems(res.data));
  }, []);

  const downloadFile = async (item) => {
    const res = await api.get(`/drive/${item._id}/download`, {
      responseType: "blob",
    });

    const url = window.URL.createObjectURL(new Blob([res.data]));
    const link = document.createElement("a");
    link.href = url;
    link.setAttribute("download", item.name);
    document.body.appendChild(link);
    link.click();
    link.remove();
  };

  return (
    <main className="p-6 max-w-5xl mx-auto">
      <h1 className="text-3xl font-bold mb-6">Shared with me</h1>

      <div className="bg-white border rounded-lg divide-y">
        {items.map((item) => (
          <div key={item._id} className="p-4 flex justify-between">
            <div>
              <p className="font-medium">{item.name}</p>
              <p className="text-sm text-gray-500">
                Owner: {item.owner?.email}
              </p>
            </div>

            {item.type === "file" && (
              <button onClick={() => downloadFile(item)} className="text-blue-600">
                Download
              </button>
            )}
          </div>
        ))}

        {items.length === 0 && (
          <p className="p-6 text-gray-500 text-center">Nothing shared with you yet.</p>
        )}
      </div>
    </main>
  );
}
```

---

### App Routing

Update `frontend/src/App.jsx`:

```jsx
import { BrowserRouter, Link, Route, Routes } from "react-router-dom";
import Navbar from "./components/Navbar";
import ProtectedRoute from "./components/ProtectedRoute";
import { AuthProvider } from "./context/AuthContext";
import Dashboard from "./pages/Dashboard";
import Login from "./pages/Login";
import SharedWithMe from "./pages/SharedWithMe";
import Signup from "./pages/Signup";
import Trash from "./pages/Trash";

function Sidebar() {
  return (
    <aside className="w-56 border-r bg-white min-h-[calc(100vh-4rem)] p-4 hidden md:block">
      <nav className="space-y-2">
        <Link className="block px-3 py-2 rounded-md hover:bg-gray-100" to="/">My Drive</Link>
        <Link className="block px-3 py-2 rounded-md hover:bg-gray-100" to="/shared">Shared with me</Link>
        <Link className="block px-3 py-2 rounded-md hover:bg-gray-100" to="/trash">Trash</Link>
      </nav>
    </aside>
  );
}

export default function App() {
  return (
    <BrowserRouter>
      <AuthProvider>
        <Navbar />
        <div className="flex">
          <Sidebar />
          <div className="flex-1">
            <Routes>
              <Route
                path="/"
                element={
                  <ProtectedRoute>
                    <Dashboard />
                  </ProtectedRoute>
                }
              />
              <Route path="/login" element={<Login />} />
              <Route path="/signup" element={<Signup />} />
              <Route
                path="/shared"
                element={
                  <ProtectedRoute>
                    <SharedWithMe />
                  </ProtectedRoute>
                }
              />
              <Route
                path="/trash"
                element={
                  <ProtectedRoute>
                    <Trash />
                  </ProtectedRoute>
                }
              />
            </Routes>
          </div>
        </div>
      </AuthProvider>
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

## 7. Core Features

### Feature 1: Signup

What it does:

Creates a new user account.

How it works:

```text
Frontend form
POST /api/auth/signup
Validate data
Check duplicate email
Hash password
Save user
Return JWT token
Frontend stores token
```

Backend code:

```js
router.post("/signup", signup);
```

Frontend code:

```js
const res = await api.post("/auth/signup", form);
login(res.data.token, res.data.user);
```

---

### Feature 2: Login

What it does:

Allows an existing user to access their Drive.

How it works:

```text
User enters email/password
Backend checks email
Backend compares password with bcrypt
Backend returns JWT
Frontend stores JWT
Protected routes become accessible
```

---

### Feature 3: Upload File

What it does:

Uploads a file into current folder.

Backend:

```js
router.post("/upload", upload.single("file"), uploadFile);
```

Frontend:

```js
const formData = new FormData();
formData.append("file", file);
formData.append("parent", parent || "");
await api.post("/drive/upload", formData);
```

Data flow:

```text
Browser file input
FormData
Axios request
Multer saves file on disk
Controller saves metadata in MongoDB
Frontend refetches folder items
```

---

### Feature 4: Create Folder

What it does:

Creates a folder under the current parent folder.

Backend:

```js
router.post("/folders", createFolder);
```

Frontend:

```js
await api.post("/drive/folders", {
  name: folderName,
  parent,
});
```

Why folders do not need `storagePath`:

A folder is only metadata. It does not contain real file bytes on disk.

---

### Feature 5: List Files And Folders

What it does:

Shows all active items inside a folder.

Backend:

```js
const items = await FileItem.find({
  owner: req.user._id,
  parent,
  isDeleted: false,
});
```

Explanation:

- `owner` ensures users only see their own files.
- `parent` controls which folder we are viewing.
- `isDeleted: false` hides trash items.

---

### Feature 6: Download File

What it does:

Downloads file content.

Backend:

```js
res.download(absolutePath, item.name);
```

Frontend:

```js
const res = await api.get(`/drive/${item._id}/download`, {
  responseType: "blob",
});
```

Why `blob`:

The response is file bytes, not JSON.

---

### Feature 7: Trash

What it does:

Moves item to trash instead of deleting immediately.

Why this is useful:

Users often delete by mistake. Trash gives them a chance to restore.

Backend:

```js
isDeleted: true,
deletedAt: new Date()
```

Restore:

```js
isDeleted: false,
deletedAt: null
```

---

### Feature 8: Sharing

What it does:

Allows a file/folder owner to share an item with another user by email.

Backend data:

```js
sharedWith: [
  {
    user: userId,
    permission: "viewer"
  }
]
```

Simple permission model:

- `viewer`: can view/download.
- `editor`: future improvement, can upload new versions or rename.

---

### Feature 9: File Versions

What it does:

Stores a history of file uploads.

Why this matters:

Google Drive lets users see older versions of files. This is useful if a file is accidentally replaced.

Our model:

```js
versions: [
  {
    versionNumber: 1,
    storagePath: "...",
    size: 12345,
    uploadedAt: "..."
  }
]
```

Beginner version:

- Upload new version.
- Save old version record.
- Show list of versions.

Production improvement:

- Allow downloading old versions.
- Limit stored versions to save space.

---

## 8. Authentication And Authorization

### Signup

```text
User submits name, email, password
Backend validates input
Backend hashes password
Backend saves user
Backend returns token
Frontend stores token
```

### Login

```text
User submits email and password
Backend finds user
Backend compares password
Backend returns token
Frontend stores token
```

### Token Handling

Token is stored in:

```js
localStorage.setItem("token", token);
```

Axios sends it:

```js
config.headers.Authorization = `Bearer ${token}`;
```

Backend reads it:

```js
const authHeader = req.headers.authorization;
```

### Protected Routes

Backend:

```js
router.use(protect);
```

Frontend:

```jsx
<ProtectedRoute>
  <Dashboard />
</ProtectedRoute>
```

### Logout

Frontend removes token:

```js
localStorage.removeItem("token");
```

---

## 9. Validation And Error Handling

### Input Validation

We use Zod:

```js
z.object({
  email: z.string().email(),
  password: z.string().min(6),
});
```

Why:

Never trust frontend input. A user can bypass frontend validation and send bad API requests directly.

### API Error Response Format

Example:

```json
{
  "message": "Validation failed",
  "errors": {
    "fieldErrors": {
      "email": ["Invalid email"]
    }
  }
}
```

### Frontend Error State

Example:

```jsx
{error && <p className="text-red-600">{error}</p>}
```

### Common Edge Cases

- User uploads no file.
- User uploads file over size limit.
- Parent folder does not exist.
- User tries to access another user's file.
- File exists in database but missing on disk.
- JWT token expired.
- Duplicate email signup.
- User shares file with non-existing email.

---

## 10. Testing And Debugging

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

#### Create Folder

```text
POST http://localhost:5000/api/drive/folders
Authorization: Bearer TOKEN
```

Body:

```json
{
  "name": "Documents",
  "parent": null
}
```

#### Upload File

```text
POST http://localhost:5000/api/drive/upload
Authorization: Bearer TOKEN
Content-Type: multipart/form-data
```

Form fields:

```text
file: choose file
parent: optional folder id
```

#### List Files

```text
GET http://localhost:5000/api/drive?parent=
Authorization: Bearer TOKEN
```

#### Download File

```text
GET http://localhost:5000/api/drive/FILE_ID/download
Authorization: Bearer TOKEN
```

---

### Test Frontend Flows

1. Open `http://localhost:5173`.
2. Create account.
3. Upload a file.
4. Create a folder.
5. Open folder.
6. Upload file inside folder.
7. Download file.
8. Rename file.
9. Star file.
10. Move file to trash.
11. Restore file.
12. Create second user.
13. Share file with second user's email.
14. Login as second user.
15. Open "Shared with me".

---

### Common Bugs And Fixes

#### CORS Error

Fix:

```js
app.use(cors({
  origin: process.env.CLIENT_URL,
  credentials: true,
}));
```

Check `.env`:

```env
CLIENT_URL=http://localhost:5173
```

#### Unauthorized Error

Cause:

- Token missing.
- Token expired.
- Axios interceptor not working.

Fix:

Check localStorage:

```js
localStorage.getItem("token")
```

#### File Upload Fails

Cause:

- Field name is wrong.

Backend expects:

```js
upload.single("file")
```

Frontend must send:

```js
formData.append("file", file);
```

#### Download Opens Broken File

Cause:

- Axios response type missing.

Fix:

```js
responseType: "blob"
```

#### MongoDB Connection Failed

Fix:

- Check `MONGO_URI`.
- Check MongoDB Atlas IP whitelist.
- Check username/password.
- Restart backend.

---

## 11. Running The Project

### Run Backend

```bash
cd backend
npm run dev
```

Expected:

```text
MongoDB connected
Server running on http://localhost:5000
```

### Run Frontend

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

### Verify Everything Works

Checklist:

- Backend health route works.
- Signup works.
- Login works.
- Token is stored in localStorage.
- Dashboard loads.
- Upload works.
- File appears in MongoDB.
- File appears in `backend/uploads`.
- Download works.
- Trash works.
- Sharing works.

---

## 12. Next Improvements

### Feature Upgrades

1. Drag-and-drop upload.
2. Upload progress bar.
3. Multiple file upload.
4. File preview for images and PDFs.
5. Search files by name.
6. Breadcrumb navigation with full folder path.
7. Folder sharing.
8. Shareable public links.
9. Role-based permissions.
10. Download old file versions.
11. Limit number of stored versions.
12. Auto-delete trash after 30 days.

### Performance Improvements

1. Pagination for large folders.
2. Lazy loading folder contents.
3. File type icons.
4. Background jobs for large files.
5. Redis caching for metadata.
6. Virus scanning before storing files.
7. Rate limiting upload APIs.

### Production Storage Upgrade

Local disk storage is not safe for production because:

- Files can be lost if server is deleted.
- Scaling multiple servers becomes hard.
- Disk space can fill up.

Better production setup:

```text
React frontend
Express API server
MongoDB metadata database
AWS S3 / Cloudinary / Google Cloud Storage for actual files
Redis for cache/rate limit
Queue for background jobs
```

### System Design Upgrade From The Attached Notes

For a real Google Drive-scale system:

- Use object storage like S3.
- Store metadata separately in database.
- Add load balancers.
- Make API servers stateless.
- Use block upload for large files.
- Split large files into chunks.
- Compress chunks.
- Encrypt chunks.
- Add resumable upload.
- Add file revision history.
- Add notification service.
- Use long polling or WebSocket for sync updates.
- Add offline sync support.
- Use cold storage for old versions.

Simple version upload flow:

```text
Client uploads file
Express saves file
MongoDB stores metadata
Frontend refreshes list
```

Production upload flow:

```text
Client requests upload session
Backend creates metadata row
Client uploads chunks
Storage saves chunks
Backend marks file uploaded
Notification service tells other clients
```

This is how a beginner project can grow into a real cloud storage system.

---

## Final Summary

You built a MERN Google Drive clone with:

- User authentication
- JWT protected routes
- File upload
- Folder creation
- File listing
- File download
- Rename
- Star
- Trash
- Restore
- Permanent delete
- Sharing
- File versions

You also learned the most important system design idea behind Google Drive:

Metadata and file content are different things.

MongoDB stores metadata. File storage stores actual bytes. This separation is what makes cloud storage systems easier to scale.

