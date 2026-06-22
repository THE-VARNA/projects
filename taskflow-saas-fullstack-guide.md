# TaskFlow SaaS - Complete Full-Stack Project Guide

Project Name: TaskFlow SaaS  
Project Type: Multi-tenant team project management platform  
Skill Level: Beginner to intermediate  

Tech Stack:

- Frontend: React, TypeScript, Vite, Tailwind CSS, React Router, TanStack Query, Zustand, dnd-kit, Socket.IO client, React Hook Form, Zod
- Backend: Node.js, Express, TypeScript, REST API, Socket.IO, Prisma ORM, Zod, BullMQ, Multer or cloud upload integration
- Database: PostgreSQL
- Cache/Queue: Redis
- Auth: JWT access tokens, refresh tokens, bcrypt password hashing, role-based access control
- Deployment: Docker, Nginx, Vercel, Render/Railway/Fly.io, Neon/Supabase/Railway Postgres, Upstash/Redis Cloud, AWS S3 or Cloudinary

---

## 1. Project Overview

### What The App Does

TaskFlow SaaS is a project management platform for teams.

Users can:

- Sign up and log in.
- Create organizations/workspaces.
- Invite team members.
- Assign roles such as owner, admin, member, and viewer.
- Create projects inside organizations.
- Create task boards for projects.
- Move tasks between kanban columns.
- Assign tasks to team members.
- Add due dates, priorities, labels, comments, and attachments.
- Receive real-time notifications.
- View project analytics.
- Track activity logs.
- Optionally manage billing for premium organizations.

### Main Features

We will build this in versions.

Version 1: Core MVP

- Auth with JWT access and refresh tokens
- Organization creation
- Membership and role-based access control
- Project CRUD
- Kanban task CRUD
- Drag-and-drop task status updates

Version 2: Collaboration

- Comments
- File attachments
- Activity logs
- Real-time notifications with Socket.IO
- Dashboard analytics

Version 3: SaaS Production Features

- Invitation flow
- Background jobs with BullMQ
- File storage with S3 or Cloudinary
- Redis caching
- Billing/subscriptions
- Deployment with Docker and cloud services

### What The User Will Learn

By the end, you will understand:

- How a real SaaS app is structured.
- What multi-tenancy means.
- How to model organizations, users, projects, tasks, and roles.
- How JWT authentication works.
- How refresh tokens keep users logged in securely.
- How role-based access control protects team data.
- How frontend state and server state are different.
- How React Query manages API data.
- How Zustand manages client-only app state.
- How drag-and-drop kanban boards work.
- How Socket.IO sends real-time updates.
- How Redis and BullMQ support queues and background jobs.
- How to prepare the app for deployment.

---

## 2. Folder Structure

Recommended monorepo-style structure:

```text
taskflow-saas/
|-- backend/
|   |-- prisma/
|   |   |-- schema.prisma
|   |   |-- seed.ts
|   |-- src/
|   |   |-- config/
|   |   |   |-- env.ts
|   |   |   |-- redis.ts
|   |   |-- controllers/
|   |   |   |-- auth.controller.ts
|   |   |   |-- organization.controller.ts
|   |   |   |-- project.controller.ts
|   |   |   |-- task.controller.ts
|   |   |   |-- comment.controller.ts
|   |   |-- jobs/
|   |   |   |-- notification.queue.ts
|   |   |   |-- notification.worker.ts
|   |   |-- lib/
|   |   |   |-- prisma.ts
|   |   |   |-- socket.ts
|   |   |   |-- tokens.ts
|   |   |   |-- password.ts
|   |   |-- middleware/
|   |   |   |-- auth.middleware.ts
|   |   |   |-- error.middleware.ts
|   |   |   |-- membership.middleware.ts
|   |   |   |-- validate.middleware.ts
|   |   |-- routes/
|   |   |   |-- auth.routes.ts
|   |   |   |-- organization.routes.ts
|   |   |   |-- project.routes.ts
|   |   |   |-- task.routes.ts
|   |   |   |-- comment.routes.ts
|   |   |-- schemas/
|   |   |   |-- auth.schema.ts
|   |   |   |-- organization.schema.ts
|   |   |   |-- project.schema.ts
|   |   |   |-- task.schema.ts
|   |   |-- types/
|   |   |   |-- express.d.ts
|   |   |-- uploads/
|   |   |-- app.ts
|   |   |-- server.ts
|   |-- docker-compose.yml
|   |-- package.json
|   |-- tsconfig.json
|   |-- .env
|
|-- frontend/
|   |-- src/
|   |   |-- api/
|   |   |   |-- client.ts
|   |   |   |-- auth.api.ts
|   |   |   |-- organizations.api.ts
|   |   |   |-- projects.api.ts
|   |   |   |-- tasks.api.ts
|   |   |-- components/
|   |   |   |-- layout/
|   |   |   |   |-- AppLayout.tsx
|   |   |   |   |-- Sidebar.tsx
|   |   |   |   |-- Topbar.tsx
|   |   |   |-- kanban/
|   |   |   |   |-- KanbanBoard.tsx
|   |   |   |   |-- KanbanColumn.tsx
|   |   |   |   |-- TaskCard.tsx
|   |   |   |-- forms/
|   |   |   |   |-- LoginForm.tsx
|   |   |   |   |-- RegisterForm.tsx
|   |   |   |   |-- TaskForm.tsx
|   |   |-- hooks/
|   |   |   |-- useSocket.ts
|   |   |-- pages/
|   |   |   |-- LoginPage.tsx
|   |   |   |-- RegisterPage.tsx
|   |   |   |-- OrganizationsPage.tsx
|   |   |   |-- ProjectPage.tsx
|   |   |   |-- DashboardPage.tsx
|   |   |-- routes/
|   |   |   |-- AppRoutes.tsx
|   |   |   |-- ProtectedRoute.tsx
|   |   |-- stores/
|   |   |   |-- auth.store.ts
|   |   |   |-- workspace.store.ts
|   |   |-- types/
|   |   |   |-- index.ts
|   |   |-- utils/
|   |   |   |-- queryClient.ts
|   |   |-- App.tsx
|   |   |-- main.tsx
|   |   |-- index.css
|   |-- package.json
|   |-- vite.config.ts
|   |-- tailwind.config.js
|
|-- README.md
```

### Why This Structure Works

- `backend/controllers`: keeps business logic separate from route definitions.
- `backend/routes`: maps URLs to controllers.
- `backend/middleware`: reusable request checks such as auth, validation, and permissions.
- `backend/schemas`: Zod validation rules.
- `backend/lib`: reusable helpers like Prisma client and token utilities.
- `frontend/api`: all backend calls live in one place.
- `frontend/stores`: Zustand client state such as current organization.
- `frontend/pages`: route-level screens.
- `frontend/components`: reusable UI pieces.

---

## 3. Setup Instructions

### Prerequisites

Install:

- Node.js 20 or newer
- npm or pnpm
- Docker Desktop
- PostgreSQL client, optional
- Redis client, optional
- VS Code
- Postman or Thunder Client

Check versions:

```bash
node -v
npm -v
docker -v
```

Create root folder:

```bash
mkdir taskflow-saas
cd taskflow-saas
mkdir backend frontend
```

---

### Backend Initialization

```bash
cd backend
npm init -y
```

Install backend dependencies:

```bash
npm install express cors dotenv cookie-parser bcrypt jsonwebtoken zod prisma @prisma/client socket.io bullmq ioredis multer morgan
npm install -D typescript tsx nodemon @types/node @types/express @types/cors @types/cookie-parser @types/bcrypt @types/jsonwebtoken @types/multer @types/morgan
```

Create TypeScript config:

```bash
npx tsc --init
```

Use this simplified `backend/tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "rootDir": "src",
    "outDir": "dist",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src", "prisma"]
}
```

Update `backend/package.json` scripts:

```json
{
  "scripts": {
    "dev": "tsx watch src/server.ts",
    "build": "tsc",
    "start": "node dist/server.js",
    "prisma:generate": "prisma generate",
    "prisma:migrate": "prisma migrate dev",
    "prisma:studio": "prisma studio",
    "seed": "tsx prisma/seed.ts"
  }
}
```

What this does:

- `dev`: runs backend in development mode.
- `build`: compiles TypeScript to JavaScript.
- `prisma:migrate`: creates database tables from Prisma schema.
- `prisma:studio`: opens a visual database browser.

---

### Docker Setup For PostgreSQL And Redis

Create `backend/docker-compose.yml`:

```yaml
services:
  postgres:
    image: postgres:16
    container_name: taskflow-postgres
    restart: always
    environment:
      POSTGRES_USER: taskflow
      POSTGRES_PASSWORD: taskflow
      POSTGRES_DB: taskflow
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7
    container_name: taskflow-redis
    restart: always
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

Start services:

```bash
docker compose up -d
```

Why Docker is useful:

You do not need to install PostgreSQL and Redis manually. Docker runs them in containers.

---

### Backend Environment Variables

Create `backend/.env`:

```env
NODE_ENV=development
PORT=5000
CLIENT_URL=http://localhost:5173

DATABASE_URL=postgresql://taskflow:taskflow@localhost:5432/taskflow
REDIS_URL=redis://localhost:6379

JWT_ACCESS_SECRET=access_secret_change_me
JWT_REFRESH_SECRET=refresh_secret_change_me
ACCESS_TOKEN_EXPIRES_IN=15m
REFRESH_TOKEN_EXPIRES_IN=7d

UPLOAD_DIR=uploads
```

Explanation:

- `DATABASE_URL`: Prisma uses this to connect to PostgreSQL.
- `REDIS_URL`: BullMQ and caching use this.
- `JWT_ACCESS_SECRET`: signs short-lived access tokens.
- `JWT_REFRESH_SECRET`: signs longer-lived refresh tokens.
- `CLIENT_URL`: allowed frontend origin for CORS.

---

### Prisma Initialization

```bash
npx prisma init
```

This creates:

```text
prisma/schema.prisma
```

We will replace it in the database design section.

---

### Frontend Initialization

Open a second terminal:

```bash
cd taskflow-saas/frontend
npm create vite@latest . -- --template react-ts
npm install
```

Install frontend dependencies:

```bash
npm install react-router-dom @tanstack/react-query zustand axios socket.io-client @dnd-kit/core @dnd-kit/sortable @dnd-kit/utilities react-hook-form zod @hookform/resolvers lucide-react
npm install -D tailwindcss postcss autoprefixer
```

Initialize Tailwind:

```bash
npx tailwindcss init -p
```

Update `frontend/tailwind.config.js`:

```js
export default {
  content: ["./index.html", "./src/**/*.{ts,tsx}"],
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

html,
body,
#root {
  min-height: 100%;
}

body {
  margin: 0;
  background: #f8fafc;
  color: #0f172a;
}
```

---

## 4. Database Design

### Multi-Tenancy Explained

Multi-tenancy means one app serves many separate teams or organizations.

Example:

- Startup A has its own organization, projects, tasks, and members.
- Agency B has its own organization, projects, tasks, and members.
- Data must never leak between organizations.

In our app, the tenant is `Organization`.

Most important data belongs to an organization:

```text
Organization
|-- Members
|-- Projects
|   |-- Tasks
|       |-- Comments
|       |-- Attachments
|-- Activity Logs
|-- Notifications
```

### Prisma Schema

Replace `backend/prisma/schema.prisma` with:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

enum OrganizationRole {
  OWNER
  ADMIN
  MEMBER
  VIEWER
}

enum ProjectRole {
  MANAGER
  CONTRIBUTOR
  VIEWER
}

enum TaskStatus {
  TODO
  IN_PROGRESS
  IN_REVIEW
  DONE
}

enum TaskPriority {
  LOW
  MEDIUM
  HIGH
  URGENT
}

enum NotificationType {
  TASK_ASSIGNED
  COMMENT_ADDED
  MEMBER_INVITED
  TASK_UPDATED
}

model User {
  id             String          @id @default(uuid())
  name           String
  email          String          @unique
  passwordHash   String
  avatarUrl      String?
  createdAt      DateTime        @default(now())
  updatedAt      DateTime        @updatedAt

  memberships    Membership[]
  refreshTokens  RefreshToken[]
  assignedTasks  Task[]          @relation("TaskAssignee")
  createdTasks   Task[]          @relation("TaskCreator")
  comments       Comment[]
  notifications  Notification[]
  activityLogs   ActivityLog[]
}

model RefreshToken {
  id         String   @id @default(uuid())
  tokenHash  String
  userId     String
  expiresAt  DateTime
  createdAt  DateTime @default(now())

  user       User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model Organization {
  id          String       @id @default(uuid())
  name        String
  slug        String       @unique
  createdAt   DateTime     @default(now())
  updatedAt   DateTime     @updatedAt

  memberships Membership[]
  projects    Project[]
  activityLogs ActivityLog[]
}

model Membership {
  id             String           @id @default(uuid())
  userId         String
  organizationId String
  role           OrganizationRole @default(MEMBER)
  createdAt      DateTime         @default(now())

  user           User             @relation(fields: [userId], references: [id], onDelete: Cascade)
  organization   Organization     @relation(fields: [organizationId], references: [id], onDelete: Cascade)

  @@unique([userId, organizationId])
}

model Project {
  id             String       @id @default(uuid())
  organizationId String
  name           String
  description    String?
  color          String       @default("#2563eb")
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt

  organization   Organization @relation(fields: [organizationId], references: [id], onDelete: Cascade)
  tasks          Task[]
}

model Task {
  id          String       @id @default(uuid())
  projectId   String
  title       String
  description String?
  status      TaskStatus   @default(TODO)
  priority    TaskPriority @default(MEDIUM)
  position    Int          @default(0)
  dueDate     DateTime?
  assigneeId  String?
  creatorId   String
  createdAt   DateTime     @default(now())
  updatedAt   DateTime     @updatedAt

  project     Project      @relation(fields: [projectId], references: [id], onDelete: Cascade)
  assignee    User?        @relation("TaskAssignee", fields: [assigneeId], references: [id], onDelete: SetNull)
  creator     User         @relation("TaskCreator", fields: [creatorId], references: [id], onDelete: Cascade)
  comments    Comment[]
  attachments Attachment[]
}

model Comment {
  id        String   @id @default(uuid())
  taskId    String
  authorId  String
  body      String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  task      Task     @relation(fields: [taskId], references: [id], onDelete: Cascade)
  author    User     @relation(fields: [authorId], references: [id], onDelete: Cascade)
}

model Attachment {
  id          String   @id @default(uuid())
  taskId      String
  fileName    String
  fileUrl     String
  fileKey     String?
  mimeType    String?
  size        Int
  createdAt   DateTime @default(now())

  task        Task     @relation(fields: [taskId], references: [id], onDelete: Cascade)
}

model Notification {
  id        String           @id @default(uuid())
  userId    String
  type      NotificationType
  title     String
  message   String
  readAt    DateTime?
  createdAt DateTime         @default(now())

  user      User             @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model ActivityLog {
  id             String       @id @default(uuid())
  organizationId String
  actorId        String
  action         String
  entityType     String
  entityId       String
  metadata       Json?
  createdAt      DateTime     @default(now())

  organization   Organization @relation(fields: [organizationId], references: [id], onDelete: Cascade)
  actor          User         @relation(fields: [actorId], references: [id], onDelete: Cascade)
}
```

### Relationship Explanation

- `User` can belong to many organizations through `Membership`.
- `Organization` has many members and many projects.
- `Project` belongs to one organization and has many tasks.
- `Task` belongs to one project.
- `Task` can have one assignee and one creator.
- `Comment` belongs to one task and one author.
- `Attachment` belongs to one task.
- `Notification` belongs to one user.
- `ActivityLog` belongs to one organization and one actor.

### Why Important Fields Exist

- `Organization.slug`: useful for URLs like `/org/acme`.
- `Membership.role`: controls what a user can do in an organization.
- `Task.position`: lets us order tasks inside kanban columns.
- `Task.status`: controls which kanban column the task appears in.
- `RefreshToken.tokenHash`: stores refresh token securely.
- `ActivityLog.metadata`: stores flexible details such as old status and new status.

Run migration:

```bash
npm run prisma:migrate -- --name init
npm run prisma:generate
```

Open Prisma Studio:

```bash
npm run prisma:studio
```

---

## 5. Backend Development

### Environment Validation

Create `backend/src/config/env.ts`:

```ts
import dotenv from "dotenv";
import { z } from "zod";

dotenv.config();

const envSchema = z.object({
  NODE_ENV: z.string().default("development"),
  PORT: z.coerce.number().default(5000),
  CLIENT_URL: z.string().url(),
  DATABASE_URL: z.string().min(1),
  REDIS_URL: z.string().min(1),
  JWT_ACCESS_SECRET: z.string().min(20),
  JWT_REFRESH_SECRET: z.string().min(20),
  ACCESS_TOKEN_EXPIRES_IN: z.string().default("15m"),
  REFRESH_TOKEN_EXPIRES_IN: z.string().default("7d"),
  UPLOAD_DIR: z.string().default("uploads"),
});

export const env = envSchema.parse(process.env);
```

What this does:

- Loads `.env`.
- Validates required environment variables.
- Converts `PORT` from string to number.
- Fails early if a required variable is missing.

Why this is useful:

Without env validation, you may only discover a missing secret after a user hits a broken route.

---

### Prisma Client

Create `backend/src/lib/prisma.ts`:

```ts
import { PrismaClient } from "@prisma/client";

export const prisma = new PrismaClient();
```

What this does:

It creates one Prisma client instance so controllers can query PostgreSQL.

Example:

```ts
await prisma.user.findUnique({ where: { email } });
```

---

### Password Helpers

Create `backend/src/lib/password.ts`:

```ts
import bcrypt from "bcrypt";

export const hashPassword = async (password: string) => {
  return bcrypt.hash(password, 12);
};

export const verifyPassword = async (password: string, hash: string) => {
  return bcrypt.compare(password, hash);
};
```

What this does:

- `hashPassword`: turns a plain password into a secure hash.
- `verifyPassword`: checks login password against the stored hash.

Important:

Never store plain passwords in the database.

---

### Token Helpers

Create `backend/src/lib/tokens.ts`:

```ts
import jwt from "jsonwebtoken";
import crypto from "crypto";
import { env } from "../config/env.js";

export type AccessTokenPayload = {
  userId: string;
};

export const signAccessToken = (payload: AccessTokenPayload) => {
  return jwt.sign(payload, env.JWT_ACCESS_SECRET, {
    expiresIn: env.ACCESS_TOKEN_EXPIRES_IN,
  });
};

export const signRefreshToken = (payload: AccessTokenPayload) => {
  return jwt.sign(payload, env.JWT_REFRESH_SECRET, {
    expiresIn: env.REFRESH_TOKEN_EXPIRES_IN,
  });
};

export const verifyAccessToken = (token: string) => {
  return jwt.verify(token, env.JWT_ACCESS_SECRET) as AccessTokenPayload;
};

export const verifyRefreshToken = (token: string) => {
  return jwt.verify(token, env.JWT_REFRESH_SECRET) as AccessTokenPayload;
};

export const hashToken = (token: string) => {
  return crypto.createHash("sha256").update(token).digest("hex");
};
```

What this does:

- Access tokens are short-lived and used for API requests.
- Refresh tokens are longer-lived and used to create new access tokens.
- We hash refresh tokens before storing them, similar to passwords.

---

### Express Type Extension

Create `backend/src/types/express.d.ts`:

```ts
import type { User } from "@prisma/client";

declare global {
  namespace Express {
    interface Request {
      user?: User;
      organizationId?: string;
    }
  }
}

export {};
```

Why this exists:

Our auth middleware adds `req.user`. TypeScript does not know that by default, so we extend Express types.

---

### Error Middleware

Create `backend/src/middleware/error.middleware.ts`:

```ts
import type { NextFunction, Request, Response } from "express";
import { ZodError } from "zod";

export class AppError extends Error {
  statusCode: number;

  constructor(message: string, statusCode = 500) {
    super(message);
    this.statusCode = statusCode;
  }
}

export const notFound = (req: Request, res: Response, next: NextFunction) => {
  next(new AppError(`Route not found: ${req.originalUrl}`, 404));
};

export const errorHandler = (
  err: unknown,
  req: Request,
  res: Response,
  next: NextFunction
) => {
  if (err instanceof ZodError) {
    return res.status(400).json({
      message: "Validation failed",
      errors: err.flatten(),
    });
  }

  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      message: err.message,
    });
  }

  console.error(err);

  return res.status(500).json({
    message: "Internal server error",
  });
};
```

What this does:

- `AppError` lets us throw known errors with status codes.
- `ZodError` returns clean validation errors.
- Unknown errors become `500 Internal server error`.

---

### Validation Middleware

Create `backend/src/middleware/validate.middleware.ts`:

```ts
import type { NextFunction, Request, Response } from "express";
import type { ZodSchema } from "zod";

export const validateBody = (schema: ZodSchema) => {
  return (req: Request, res: Response, next: NextFunction) => {
    req.body = schema.parse(req.body);
    next();
  };
};
```

What this does:

It validates `req.body` before the controller runs.

Example:

```ts
router.post("/login", validateBody(loginSchema), login);
```

If the body is invalid, Zod throws an error and the error middleware handles it.

---

### Auth Middleware

Create `backend/src/middleware/auth.middleware.ts`:

```ts
import type { NextFunction, Request, Response } from "express";
import { prisma } from "../lib/prisma.js";
import { verifyAccessToken } from "../lib/tokens.js";
import { AppError } from "./error.middleware.js";

export const requireAuth = async (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  const header = req.headers.authorization;

  if (!header || !header.startsWith("Bearer ")) {
    return next(new AppError("Unauthorized", 401));
  }

  const token = header.split(" ")[1];
  const payload = verifyAccessToken(token);

  const user = await prisma.user.findUnique({
    where: { id: payload.userId },
  });

  if (!user) {
    return next(new AppError("Unauthorized", 401));
  }

  req.user = user;
  next();
};
```

What this does:

1. Reads `Authorization` header.
2. Extracts JWT token.
3. Verifies token.
4. Finds the user.
5. Adds user to `req.user`.

---

### Membership And RBAC Middleware

Create `backend/src/middleware/membership.middleware.ts`:

```ts
import type { NextFunction, Request, Response } from "express";
import type { OrganizationRole } from "@prisma/client";
import { prisma } from "../lib/prisma.js";
import { AppError } from "./error.middleware.js";

const roleRank: Record<OrganizationRole, number> = {
  VIEWER: 1,
  MEMBER: 2,
  ADMIN: 3,
  OWNER: 4,
};

export const requireOrganizationRole = (minimumRole: OrganizationRole) => {
  return async (req: Request, res: Response, next: NextFunction) => {
    if (!req.user) {
      return next(new AppError("Unauthorized", 401));
    }

    const organizationId = req.params.organizationId || req.body.organizationId;

    if (!organizationId) {
      return next(new AppError("Organization id is required", 400));
    }

    const membership = await prisma.membership.findUnique({
      where: {
        userId_organizationId: {
          userId: req.user.id,
          organizationId,
        },
      },
    });

    if (!membership) {
      return next(new AppError("You are not a member of this organization", 403));
    }

    if (roleRank[membership.role] < roleRank[minimumRole]) {
      return next(new AppError("You do not have permission", 403));
    }

    req.organizationId = organizationId;
    next();
  };
};
```

What RBAC means:

RBAC means role-based access control.

Example:

- Owner can delete organization.
- Admin can invite members.
- Member can create tasks.
- Viewer can only read.

---

### Auth Schemas

Create `backend/src/schemas/auth.schema.ts`:

```ts
import { z } from "zod";

export const registerSchema = z.object({
  name: z.string().min(1, "Name is required"),
  email: z.string().email("Invalid email"),
  password: z.string().min(8, "Password must be at least 8 characters"),
});

export const loginSchema = z.object({
  email: z.string().email("Invalid email"),
  password: z.string().min(1, "Password is required"),
});

export const refreshSchema = z.object({
  refreshToken: z.string().min(1),
});
```

What this does:

It defines the shape of valid auth requests.

---

### Auth Controller

Create `backend/src/controllers/auth.controller.ts`:

```ts
import type { Request, Response } from "express";
import { addDays } from "date-fns";
import { prisma } from "../lib/prisma.js";
import { hashPassword, verifyPassword } from "../lib/password.js";
import {
  hashToken,
  signAccessToken,
  signRefreshToken,
  verifyRefreshToken,
} from "../lib/tokens.js";
import { AppError } from "../middleware/error.middleware.js";

const createTokenPair = async (userId: string) => {
  const accessToken = signAccessToken({ userId });
  const refreshToken = signRefreshToken({ userId });

  await prisma.refreshToken.create({
    data: {
      userId,
      tokenHash: hashToken(refreshToken),
      expiresAt: addDays(new Date(), 7),
    },
  });

  return { accessToken, refreshToken };
};

export const register = async (req: Request, res: Response) => {
  const { name, email, password } = req.body;

  const existingUser = await prisma.user.findUnique({
    where: { email },
  });

  if (existingUser) {
    throw new AppError("Email already registered", 400);
  }

  const passwordHash = await hashPassword(password);

  const user = await prisma.user.create({
    data: {
      name,
      email,
      passwordHash,
    },
    select: {
      id: true,
      name: true,
      email: true,
      createdAt: true,
    },
  });

  const tokens = await createTokenPair(user.id);

  res.status(201).json({ user, ...tokens });
};

export const login = async (req: Request, res: Response) => {
  const { email, password } = req.body;

  const user = await prisma.user.findUnique({
    where: { email },
  });

  if (!user) {
    throw new AppError("Invalid email or password", 401);
  }

  const isValidPassword = await verifyPassword(password, user.passwordHash);

  if (!isValidPassword) {
    throw new AppError("Invalid email or password", 401);
  }

  const tokens = await createTokenPair(user.id);

  res.json({
    user: {
      id: user.id,
      name: user.name,
      email: user.email,
    },
    ...tokens,
  });
};

export const refresh = async (req: Request, res: Response) => {
  const { refreshToken } = req.body;
  const payload = verifyRefreshToken(refreshToken);

  const storedToken = await prisma.refreshToken.findFirst({
    where: {
      userId: payload.userId,
      tokenHash: hashToken(refreshToken),
      expiresAt: {
        gt: new Date(),
      },
    },
  });

  if (!storedToken) {
    throw new AppError("Invalid refresh token", 401);
  }

  const accessToken = signAccessToken({ userId: payload.userId });

  res.json({ accessToken });
};

export const logout = async (req: Request, res: Response) => {
  const { refreshToken } = req.body;

  if (refreshToken) {
    await prisma.refreshToken.deleteMany({
      where: { tokenHash: hashToken(refreshToken) },
    });
  }

  res.json({ message: "Logged out" });
};

export const me = async (req: Request, res: Response) => {
  res.json({ user: req.user });
};
```

Explanation:

- `register`: creates a user and immediately returns tokens.
- `login`: checks password and returns tokens.
- `refresh`: creates a new access token when the old one expires.
- `logout`: deletes the refresh token from the database.
- `me`: returns the current authenticated user.

Note:

This snippet uses `date-fns`, so install it:

```bash
npm install date-fns
```

---

### Auth Routes

Create `backend/src/routes/auth.routes.ts`:

```ts
import { Router } from "express";
import { login, logout, me, refresh, register } from "../controllers/auth.controller.js";
import { requireAuth } from "../middleware/auth.middleware.js";
import { validateBody } from "../middleware/validate.middleware.js";
import { loginSchema, refreshSchema, registerSchema } from "../schemas/auth.schema.js";

export const authRoutes = Router();

authRoutes.post("/register", validateBody(registerSchema), register);
authRoutes.post("/login", validateBody(loginSchema), login);
authRoutes.post("/refresh", validateBody(refreshSchema), refresh);
authRoutes.post("/logout", logout);
authRoutes.get("/me", requireAuth, me);
```

Route summary:

| Method | URL | Purpose |
|---|---|---|
| `POST` | `/api/auth/register` | Create account |
| `POST` | `/api/auth/login` | Log in |
| `POST` | `/api/auth/refresh` | Get new access token |
| `POST` | `/api/auth/logout` | Log out |
| `GET` | `/api/auth/me` | Get current user |

---

### Organization Schemas

Create `backend/src/schemas/organization.schema.ts`:

```ts
import { z } from "zod";

export const createOrganizationSchema = z.object({
  name: z.string().min(2, "Organization name is required"),
});

export const inviteMemberSchema = z.object({
  email: z.string().email("Invalid email"),
  role: z.enum(["ADMIN", "MEMBER", "VIEWER"]).default("MEMBER"),
});
```

---

### Organization Controller

Create `backend/src/controllers/organization.controller.ts`:

```ts
import type { Request, Response } from "express";
import { prisma } from "../lib/prisma.js";
import { AppError } from "../middleware/error.middleware.js";

const slugify = (value: string) => {
  return value
    .toLowerCase()
    .trim()
    .replace(/[^a-z0-9]+/g, "-")
    .replace(/(^-|-$)+/g, "");
};

export const createOrganization = async (req: Request, res: Response) => {
  const { name } = req.body;
  const baseSlug = slugify(name);
  const slug = `${baseSlug}-${Math.random().toString(36).slice(2, 7)}`;

  const organization = await prisma.organization.create({
    data: {
      name,
      slug,
      memberships: {
        create: {
          userId: req.user!.id,
          role: "OWNER",
        },
      },
    },
    include: {
      memberships: true,
    },
  });

  res.status(201).json({ organization });
};

export const getMyOrganizations = async (req: Request, res: Response) => {
  const memberships = await prisma.membership.findMany({
    where: { userId: req.user!.id },
    include: { organization: true },
    orderBy: { createdAt: "asc" },
  });

  res.json({ organizations: memberships.map((m) => m.organization) });
};

export const inviteMember = async (req: Request, res: Response) => {
  const { email, role } = req.body;
  const { organizationId } = req.params;

  const userToInvite = await prisma.user.findUnique({
    where: { email },
  });

  if (!userToInvite) {
    throw new AppError("User with this email does not exist", 404);
  }

  const membership = await prisma.membership.upsert({
    where: {
      userId_organizationId: {
        userId: userToInvite.id,
        organizationId,
      },
    },
    update: { role },
    create: {
      userId: userToInvite.id,
      organizationId,
      role,
    },
  });

  await prisma.activityLog.create({
    data: {
      organizationId,
      actorId: req.user!.id,
      action: "MEMBER_INVITED",
      entityType: "Membership",
      entityId: membership.id,
      metadata: { email, role },
    },
  });

  res.status(201).json({ membership });
};
```

Explanation:

- Creating an organization also creates an owner membership.
- `upsert` means "update if exists, otherwise create".
- Activity logs record important actions for auditing.

---

### Organization Routes

Create `backend/src/routes/organization.routes.ts`:

```ts
import { Router } from "express";
import {
  createOrganization,
  getMyOrganizations,
  inviteMember,
} from "../controllers/organization.controller.js";
import { requireAuth } from "../middleware/auth.middleware.js";
import { requireOrganizationRole } from "../middleware/membership.middleware.js";
import { validateBody } from "../middleware/validate.middleware.js";
import {
  createOrganizationSchema,
  inviteMemberSchema,
} from "../schemas/organization.schema.js";

export const organizationRoutes = Router();

organizationRoutes.use(requireAuth);

organizationRoutes.get("/", getMyOrganizations);
organizationRoutes.post("/", validateBody(createOrganizationSchema), createOrganization);
organizationRoutes.post(
  "/:organizationId/invite",
  requireOrganizationRole("ADMIN"),
  validateBody(inviteMemberSchema),
  inviteMember
);
```

---

### Project Schemas

Create `backend/src/schemas/project.schema.ts`:

```ts
import { z } from "zod";

export const createProjectSchema = z.object({
  organizationId: z.string().uuid(),
  name: z.string().min(2, "Project name is required"),
  description: z.string().optional(),
  color: z.string().optional(),
});

export const updateProjectSchema = z.object({
  name: z.string().min(2).optional(),
  description: z.string().optional(),
  color: z.string().optional(),
});
```

---

### Project Controller

Create `backend/src/controllers/project.controller.ts`:

```ts
import type { Request, Response } from "express";
import { prisma } from "../lib/prisma.js";
import { AppError } from "../middleware/error.middleware.js";

export const createProject = async (req: Request, res: Response) => {
  const { organizationId, name, description, color } = req.body;

  const project = await prisma.project.create({
    data: {
      organizationId,
      name,
      description,
      color,
    },
  });

  await prisma.activityLog.create({
    data: {
      organizationId,
      actorId: req.user!.id,
      action: "PROJECT_CREATED",
      entityType: "Project",
      entityId: project.id,
      metadata: { name },
    },
  });

  res.status(201).json({ project });
};

export const getProjects = async (req: Request, res: Response) => {
  const { organizationId } = req.params;

  const projects = await prisma.project.findMany({
    where: { organizationId },
    orderBy: { createdAt: "desc" },
  });

  res.json({ projects });
};

export const updateProject = async (req: Request, res: Response) => {
  const { projectId } = req.params;

  const project = await prisma.project.update({
    where: { id: projectId },
    data: req.body,
  });

  res.json({ project });
};

export const deleteProject = async (req: Request, res: Response) => {
  const { projectId } = req.params;

  await prisma.project.delete({
    where: { id: projectId },
  });

  res.json({ message: "Project deleted" });
};
```

---

### Project Routes

Create `backend/src/routes/project.routes.ts`:

```ts
import { Router } from "express";
import {
  createProject,
  deleteProject,
  getProjects,
  updateProject,
} from "../controllers/project.controller.js";
import { requireAuth } from "../middleware/auth.middleware.js";
import { requireOrganizationRole } from "../middleware/membership.middleware.js";
import { validateBody } from "../middleware/validate.middleware.js";
import { createProjectSchema, updateProjectSchema } from "../schemas/project.schema.js";

export const projectRoutes = Router();

projectRoutes.use(requireAuth);

projectRoutes.get(
  "/organization/:organizationId",
  requireOrganizationRole("VIEWER"),
  getProjects
);

projectRoutes.post(
  "/",
  requireOrganizationRole("MEMBER"),
  validateBody(createProjectSchema),
  createProject
);

projectRoutes.patch(
  "/:projectId",
  validateBody(updateProjectSchema),
  updateProject
);

projectRoutes.delete("/:projectId", deleteProject);
```

Best practice note:

For `updateProject` and `deleteProject`, a production app should load the project, find its organization, then check membership against that organization. The beginner version keeps the route simple, but do not skip this check in a real app.

---

### Task Schemas

Create `backend/src/schemas/task.schema.ts`:

```ts
import { z } from "zod";

export const createTaskSchema = z.object({
  projectId: z.string().uuid(),
  title: z.string().min(1, "Task title is required"),
  description: z.string().optional(),
  priority: z.enum(["LOW", "MEDIUM", "HIGH", "URGENT"]).default("MEDIUM"),
  assigneeId: z.string().uuid().optional().nullable(),
  dueDate: z.string().datetime().optional().nullable(),
});

export const updateTaskSchema = z.object({
  title: z.string().min(1).optional(),
  description: z.string().optional().nullable(),
  status: z.enum(["TODO", "IN_PROGRESS", "IN_REVIEW", "DONE"]).optional(),
  priority: z.enum(["LOW", "MEDIUM", "HIGH", "URGENT"]).optional(),
  assigneeId: z.string().uuid().optional().nullable(),
  dueDate: z.string().datetime().optional().nullable(),
  position: z.number().int().optional(),
});

export const moveTaskSchema = z.object({
  status: z.enum(["TODO", "IN_PROGRESS", "IN_REVIEW", "DONE"]),
  position: z.number().int().min(0),
});
```

---

### Socket Helper

Create `backend/src/lib/socket.ts`:

```ts
import type { Server } from "socket.io";

let io: Server | null = null;

export const setSocketServer = (server: Server) => {
  io = server;
};

export const getSocketServer = () => {
  if (!io) {
    throw new Error("Socket server not initialized");
  }

  return io;
};
```

What this does:

It lets controllers emit real-time events without passing the Socket.IO server everywhere manually.

---

### Task Controller

Create `backend/src/controllers/task.controller.ts`:

```ts
import type { Request, Response } from "express";
import { prisma } from "../lib/prisma.js";
import { getSocketServer } from "../lib/socket.js";
import { AppError } from "../middleware/error.middleware.js";

const getProjectWithOrganization = async (projectId: string) => {
  const project = await prisma.project.findUnique({
    where: { id: projectId },
    include: { organization: true },
  });

  if (!project) {
    throw new AppError("Project not found", 404);
  }

  return project;
};

export const getProjectTasks = async (req: Request, res: Response) => {
  const { projectId } = req.params;

  const tasks = await prisma.task.findMany({
    where: { projectId },
    include: {
      assignee: {
        select: { id: true, name: true, email: true },
      },
      creator: {
        select: { id: true, name: true, email: true },
      },
    },
    orderBy: [{ status: "asc" }, { position: "asc" }],
  });

  res.json({ tasks });
};

export const createTask = async (req: Request, res: Response) => {
  const { projectId, title, description, priority, assigneeId, dueDate } = req.body;
  const project = await getProjectWithOrganization(projectId);

  const lastTask = await prisma.task.findFirst({
    where: { projectId, status: "TODO" },
    orderBy: { position: "desc" },
  });

  const task = await prisma.task.create({
    data: {
      projectId,
      title,
      description,
      priority,
      assigneeId,
      dueDate: dueDate ? new Date(dueDate) : null,
      creatorId: req.user!.id,
      position: lastTask ? lastTask.position + 1 : 0,
    },
    include: {
      assignee: { select: { id: true, name: true, email: true } },
      creator: { select: { id: true, name: true, email: true } },
    },
  });

  await prisma.activityLog.create({
    data: {
      organizationId: project.organizationId,
      actorId: req.user!.id,
      action: "TASK_CREATED",
      entityType: "Task",
      entityId: task.id,
      metadata: { title },
    },
  });

  if (assigneeId) {
    await prisma.notification.create({
      data: {
        userId: assigneeId,
        type: "TASK_ASSIGNED",
        title: "New task assigned",
        message: `You were assigned to "${title}"`,
      },
    });
  }

  const io = getSocketServer();
  io.to(`project:${projectId}`).emit("task:created", task);

  res.status(201).json({ task });
};

export const updateTask = async (req: Request, res: Response) => {
  const { taskId } = req.params;

  const existingTask = await prisma.task.findUnique({
    where: { id: taskId },
    include: { project: true },
  });

  if (!existingTask) {
    throw new AppError("Task not found", 404);
  }

  const data = {
    ...req.body,
    dueDate: req.body.dueDate ? new Date(req.body.dueDate) : req.body.dueDate,
  };

  const task = await prisma.task.update({
    where: { id: taskId },
    data,
    include: {
      assignee: { select: { id: true, name: true, email: true } },
      creator: { select: { id: true, name: true, email: true } },
    },
  });

  await prisma.activityLog.create({
    data: {
      organizationId: existingTask.project.organizationId,
      actorId: req.user!.id,
      action: "TASK_UPDATED",
      entityType: "Task",
      entityId: task.id,
      metadata: req.body,
    },
  });

  getSocketServer().to(`project:${task.projectId}`).emit("task:updated", task);

  res.json({ task });
};

export const moveTask = async (req: Request, res: Response) => {
  const { taskId } = req.params;
  const { status, position } = req.body;

  const task = await prisma.task.update({
    where: { id: taskId },
    data: { status, position },
  });

  getSocketServer().to(`project:${task.projectId}`).emit("task:moved", task);

  res.json({ task });
};

export const deleteTask = async (req: Request, res: Response) => {
  const { taskId } = req.params;

  const task = await prisma.task.delete({
    where: { id: taskId },
  });

  getSocketServer().to(`project:${task.projectId}`).emit("task:deleted", {
    id: task.id,
  });

  res.json({ message: "Task deleted" });
};
```

Important explanation:

- `position` controls the order of cards in a column.
- Socket events update other connected users in real time.
- Activity logs make the system auditable.
- Notifications are created when a user is assigned.

---

### Task Routes

Create `backend/src/routes/task.routes.ts`:

```ts
import { Router } from "express";
import {
  createTask,
  deleteTask,
  getProjectTasks,
  moveTask,
  updateTask,
} from "../controllers/task.controller.js";
import { requireAuth } from "../middleware/auth.middleware.js";
import { validateBody } from "../middleware/validate.middleware.js";
import { createTaskSchema, moveTaskSchema, updateTaskSchema } from "../schemas/task.schema.js";

export const taskRoutes = Router();

taskRoutes.use(requireAuth);

taskRoutes.get("/project/:projectId", getProjectTasks);
taskRoutes.post("/", validateBody(createTaskSchema), createTask);
taskRoutes.patch("/:taskId", validateBody(updateTaskSchema), updateTask);
taskRoutes.patch("/:taskId/move", validateBody(moveTaskSchema), moveTask);
taskRoutes.delete("/:taskId", deleteTask);
```

---

### App Setup

Create `backend/src/app.ts`:

```ts
import express from "express";
import cors from "cors";
import cookieParser from "cookie-parser";
import morgan from "morgan";
import { env } from "./config/env.js";
import { authRoutes } from "./routes/auth.routes.js";
import { organizationRoutes } from "./routes/organization.routes.js";
import { projectRoutes } from "./routes/project.routes.js";
import { taskRoutes } from "./routes/task.routes.js";
import { errorHandler, notFound } from "./middleware/error.middleware.js";

export const app = express();

app.use(cors({
  origin: env.CLIENT_URL,
  credentials: true,
}));

app.use(express.json());
app.use(cookieParser());
app.use(morgan("dev"));

app.get("/health", (req, res) => {
  res.json({ status: "ok" });
});

app.use("/api/auth", authRoutes);
app.use("/api/organizations", organizationRoutes);
app.use("/api/projects", projectRoutes);
app.use("/api/tasks", taskRoutes);

app.use(notFound);
app.use(errorHandler);
```

What each middleware does:

- `cors`: allows frontend to call backend.
- `express.json`: lets backend read JSON bodies.
- `cookieParser`: useful if you later move refresh tokens to HTTP-only cookies.
- `morgan`: logs requests during development.

---

### Server With Socket.IO

Create `backend/src/server.ts`:

```ts
import http from "http";
import { Server } from "socket.io";
import { app } from "./app.js";
import { env } from "./config/env.js";
import { setSocketServer } from "./lib/socket.js";

const httpServer = http.createServer(app);

const io = new Server(httpServer, {
  cors: {
    origin: env.CLIENT_URL,
    credentials: true,
  },
});

setSocketServer(io);

io.on("connection", (socket) => {
  console.log("Socket connected", socket.id);

  socket.on("project:join", (projectId: string) => {
    socket.join(`project:${projectId}`);
  });

  socket.on("project:leave", (projectId: string) => {
    socket.leave(`project:${projectId}`);
  });

  socket.on("disconnect", () => {
    console.log("Socket disconnected", socket.id);
  });
});

httpServer.listen(env.PORT, () => {
  console.log(`API running on http://localhost:${env.PORT}`);
});
```

What this does:

- Creates an HTTP server.
- Attaches Express to it.
- Attaches Socket.IO to the same server.
- Allows users to join project-specific rooms.

Why rooms matter:

Only users viewing the same project should receive task updates for that project.

---

## 6. Frontend Development

### Query Client

Create `frontend/src/utils/queryClient.ts`:

```ts
import { QueryClient } from "@tanstack/react-query";

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 1,
      refetchOnWindowFocus: false,
    },
  },
});
```

What this does:

TanStack Query uses this client to cache API responses.

---

### Axios Client

Create `frontend/src/api/client.ts`:

```ts
import axios from "axios";
import { useAuthStore } from "../stores/auth.store";

export const api = axios.create({
  baseURL: "http://localhost:5000/api",
});

api.interceptors.request.use((config) => {
  const token = useAuthStore.getState().accessToken;

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  return config;
});
```

What this does:

- Sets backend base URL.
- Automatically adds access token to every request.

Why use an interceptor:

Without it, every API call would need repeated token code.

---

### Auth Store With Zustand

Create `frontend/src/stores/auth.store.ts`:

```ts
import { create } from "zustand";
import { persist } from "zustand/middleware";

type User = {
  id: string;
  name: string;
  email: string;
};

type AuthState = {
  user: User | null;
  accessToken: string | null;
  refreshToken: string | null;
  setAuth: (data: { user: User; accessToken: string; refreshToken: string }) => void;
  logout: () => void;
};

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      accessToken: null,
      refreshToken: null,
      setAuth: ({ user, accessToken, refreshToken }) => {
        set({ user, accessToken, refreshToken });
      },
      logout: () => {
        set({ user: null, accessToken: null, refreshToken: null });
      },
    }),
    {
      name: "taskflow-auth",
    }
  )
);
```

What Zustand does:

It stores client-side state. Here it remembers the logged-in user and tokens.

What `persist` does:

It saves the auth state to localStorage so refresh does not log the user out.

---

### Auth API

Create `frontend/src/api/auth.api.ts`:

```ts
import { api } from "./client";

export type LoginInput = {
  email: string;
  password: string;
};

export type RegisterInput = {
  name: string;
  email: string;
  password: string;
};

export const loginRequest = async (data: LoginInput) => {
  const response = await api.post("/auth/login", data);
  return response.data;
};

export const registerRequest = async (data: RegisterInput) => {
  const response = await api.post("/auth/register", data);
  return response.data;
};
```

What this does:

It keeps auth-related HTTP calls in one file.

---

### Login Form

Create `frontend/src/components/forms/LoginForm.tsx`:

```tsx
import { zodResolver } from "@hookform/resolvers/zod";
import { useMutation } from "@tanstack/react-query";
import { useForm } from "react-hook-form";
import { useNavigate } from "react-router-dom";
import { z } from "zod";
import { loginRequest } from "../../api/auth.api";
import { useAuthStore } from "../../stores/auth.store";

const loginSchema = z.object({
  email: z.string().email("Invalid email"),
  password: z.string().min(1, "Password is required"),
});

type LoginFormValues = z.infer<typeof loginSchema>;

export const LoginForm = () => {
  const navigate = useNavigate();
  const setAuth = useAuthStore((state) => state.setAuth);

  const form = useForm<LoginFormValues>({
    resolver: zodResolver(loginSchema),
    defaultValues: {
      email: "",
      password: "",
    },
  });

  const mutation = useMutation({
    mutationFn: loginRequest,
    onSuccess: (data) => {
      setAuth(data);
      navigate("/organizations");
    },
  });

  const onSubmit = (values: LoginFormValues) => {
    mutation.mutate(values);
  };

  return (
    <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
      <div>
        <label className="block text-sm font-medium">Email</label>
        <input
          className="mt-1 w-full rounded-md border px-3 py-2"
          {...form.register("email")}
        />
        <p className="text-sm text-red-600">{form.formState.errors.email?.message}</p>
      </div>

      <div>
        <label className="block text-sm font-medium">Password</label>
        <input
          type="password"
          className="mt-1 w-full rounded-md border px-3 py-2"
          {...form.register("password")}
        />
        <p className="text-sm text-red-600">{form.formState.errors.password?.message}</p>
      </div>

      {mutation.isError && (
        <p className="text-sm text-red-600">Login failed. Check your credentials.</p>
      )}

      <button
        type="submit"
        disabled={mutation.isPending}
        className="w-full rounded-md bg-blue-600 px-4 py-2 text-white disabled:opacity-60"
      >
        {mutation.isPending ? "Logging in..." : "Login"}
      </button>
    </form>
  );
};
```

Important concepts:

- React Hook Form controls the form.
- Zod validates form data.
- React Query mutation sends the API request.
- Zustand stores auth data after success.

---

### Login Page

Create `frontend/src/pages/LoginPage.tsx`:

```tsx
import { Link } from "react-router-dom";
import { LoginForm } from "../components/forms/LoginForm";

export const LoginPage = () => {
  return (
    <main className="flex min-h-screen items-center justify-center bg-slate-100 px-4">
      <div className="w-full max-w-md rounded-lg border bg-white p-6 shadow-sm">
        <h1 className="text-2xl font-bold">Login to TaskFlow</h1>
        <p className="mt-1 text-sm text-slate-600">
          Manage projects, teams, and tasks in one workspace.
        </p>

        <div className="mt-6">
          <LoginForm />
        </div>

        <p className="mt-4 text-sm">
          No account? <Link className="text-blue-600" to="/register">Create one</Link>
        </p>
      </div>
    </main>
  );
};
```

---

### Protected Route

Create `frontend/src/routes/ProtectedRoute.tsx`:

```tsx
import { Navigate } from "react-router-dom";
import { useAuthStore } from "../stores/auth.store";

type Props = {
  children: React.ReactNode;
};

export const ProtectedRoute = ({ children }: Props) => {
  const accessToken = useAuthStore((state) => state.accessToken);

  if (!accessToken) {
    return <Navigate to="/login" replace />;
  }

  return children;
};
```

What this does:

It blocks private pages when the user is not logged in.

---

### App Routes

Create `frontend/src/routes/AppRoutes.tsx`:

```tsx
import { Navigate, Route, Routes } from "react-router-dom";
import { LoginPage } from "../pages/LoginPage";
import { RegisterPage } from "../pages/RegisterPage";
import { OrganizationsPage } from "../pages/OrganizationsPage";
import { ProjectPage } from "../pages/ProjectPage";
import { ProtectedRoute } from "./ProtectedRoute";

export const AppRoutes = () => {
  return (
    <Routes>
      <Route path="/" element={<Navigate to="/organizations" replace />} />
      <Route path="/login" element={<LoginPage />} />
      <Route path="/register" element={<RegisterPage />} />
      <Route
        path="/organizations"
        element={
          <ProtectedRoute>
            <OrganizationsPage />
          </ProtectedRoute>
        }
      />
      <Route
        path="/projects/:projectId"
        element={
          <ProtectedRoute>
            <ProjectPage />
          </ProtectedRoute>
        }
      />
    </Routes>
  );
};
```

---

### App Entry

Update `frontend/src/App.tsx`:

```tsx
import { BrowserRouter } from "react-router-dom";
import { AppRoutes } from "./routes/AppRoutes";

export const App = () => {
  return (
    <BrowserRouter>
      <AppRoutes />
    </BrowserRouter>
  );
};
```

Update `frontend/src/main.tsx`:

```tsx
import React from "react";
import ReactDOM from "react-dom/client";
import { QueryClientProvider } from "@tanstack/react-query";
import { App } from "./App";
import { queryClient } from "./utils/queryClient";
import "./index.css";

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <App />
    </QueryClientProvider>
  </React.StrictMode>
);
```

What this does:

- React Router handles page navigation.
- QueryClientProvider enables TanStack Query everywhere.

---

### Task API

Create `frontend/src/api/tasks.api.ts`:

```ts
import { api } from "./client";

export type TaskStatus = "TODO" | "IN_PROGRESS" | "IN_REVIEW" | "DONE";

export type Task = {
  id: string;
  projectId: string;
  title: string;
  description?: string | null;
  status: TaskStatus;
  priority: "LOW" | "MEDIUM" | "HIGH" | "URGENT";
  position: number;
  assignee?: {
    id: string;
    name: string;
    email: string;
  } | null;
};

export const getProjectTasks = async (projectId: string) => {
  const response = await api.get(`/tasks/project/${projectId}`);
  return response.data.tasks as Task[];
};

export const moveTaskRequest = async (taskId: string, data: {
  status: TaskStatus;
  position: number;
}) => {
  const response = await api.patch(`/tasks/${taskId}/move`, data);
  return response.data.task as Task;
};
```

---

### Kanban Board With dnd-kit

Create `frontend/src/components/kanban/KanbanBoard.tsx`:

```tsx
import { DndContext, DragEndEvent } from "@dnd-kit/core";
import { useMutation, useQuery, useQueryClient } from "@tanstack/react-query";
import { getProjectTasks, moveTaskRequest, Task, TaskStatus } from "../../api/tasks.api";

const columns: { id: TaskStatus; title: string }[] = [
  { id: "TODO", title: "Todo" },
  { id: "IN_PROGRESS", title: "In Progress" },
  { id: "IN_REVIEW", title: "In Review" },
  { id: "DONE", title: "Done" },
];

type Props = {
  projectId: string;
};

export const KanbanBoard = ({ projectId }: Props) => {
  const queryClient = useQueryClient();

  const tasksQuery = useQuery({
    queryKey: ["tasks", projectId],
    queryFn: () => getProjectTasks(projectId),
  });

  const moveMutation = useMutation({
    mutationFn: ({ taskId, status, position }: {
      taskId: string;
      status: TaskStatus;
      position: number;
    }) => moveTaskRequest(taskId, { status, position }),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["tasks", projectId] });
    },
  });

  const tasks = tasksQuery.data ?? [];

  const onDragEnd = (event: DragEndEvent) => {
    const { active, over } = event;

    if (!over) return;

    const taskId = String(active.id);
    const newStatus = String(over.id) as TaskStatus;

    moveMutation.mutate({
      taskId,
      status: newStatus,
      position: 0,
    });
  };

  if (tasksQuery.isLoading) {
    return <div>Loading board...</div>;
  }

  return (
    <DndContext onDragEnd={onDragEnd}>
      <div className="grid gap-4 md:grid-cols-4">
        {columns.map((column) => (
          <div
            key={column.id}
            id={column.id}
            className="min-h-[500px] rounded-lg border bg-slate-100 p-3"
          >
            <h2 className="mb-3 font-semibold">{column.title}</h2>

            <div className="space-y-3">
              {tasks
                .filter((task) => task.status === column.id)
                .map((task) => (
                  <TaskCard key={task.id} task={task} />
                ))}
            </div>
          </div>
        ))}
      </div>
    </DndContext>
  );
};

const TaskCard = ({ task }: { task: Task }) => {
  return (
    <div
      id={task.id}
      className="rounded-md border bg-white p-3 shadow-sm"
      draggable
    >
      <h3 className="font-medium">{task.title}</h3>
      <p className="mt-1 text-xs text-slate-500">{task.priority}</p>
      {task.assignee && (
        <p className="mt-2 text-xs text-slate-600">Assigned to {task.assignee.name}</p>
      )}
    </div>
  );
};
```

Important note:

This is a simplified dnd-kit example. For production-quality sortable cards, use `@dnd-kit/sortable` with sortable contexts for columns and cards. The main concept is still the same: when a card moves, call the backend to update `status` and `position`.

---

### Socket Hook

Create `frontend/src/hooks/useSocket.ts`:

```ts
import { useEffect } from "react";
import { io } from "socket.io-client";
import { useQueryClient } from "@tanstack/react-query";

const socket = io("http://localhost:5000", {
  autoConnect: false,
});

export const useProjectSocket = (projectId: string) => {
  const queryClient = useQueryClient();

  useEffect(() => {
    socket.connect();
    socket.emit("project:join", projectId);

    const refreshTasks = () => {
      queryClient.invalidateQueries({ queryKey: ["tasks", projectId] });
    };

    socket.on("task:created", refreshTasks);
    socket.on("task:updated", refreshTasks);
    socket.on("task:moved", refreshTasks);
    socket.on("task:deleted", refreshTasks);

    return () => {
      socket.emit("project:leave", projectId);
      socket.off("task:created", refreshTasks);
      socket.off("task:updated", refreshTasks);
      socket.off("task:moved", refreshTasks);
      socket.off("task:deleted", refreshTasks);
    };
  }, [projectId, queryClient]);
};
```

What this does:

- Connects to Socket.IO.
- Joins a project room.
- Refetches tasks when another user changes the board.

---

### Project Page

Create `frontend/src/pages/ProjectPage.tsx`:

```tsx
import { useParams } from "react-router-dom";
import { KanbanBoard } from "../components/kanban/KanbanBoard";
import { useProjectSocket } from "../hooks/useSocket";

export const ProjectPage = () => {
  const { projectId } = useParams();

  if (!projectId) {
    return <div>Project not found</div>;
  }

  useProjectSocket(projectId);

  return (
    <main className="p-6">
      <div className="mb-6">
        <h1 className="text-2xl font-bold">Project Board</h1>
        <p className="text-sm text-slate-600">Drag tasks between columns to update progress.</p>
      </div>

      <KanbanBoard projectId={projectId} />
    </main>
  );
};
```

---

## 7. Core Features

### Feature 1: Authentication

What it does:

Lets users register, log in, refresh tokens, and log out.

How it works:

```text
User submits form
Frontend sends request
Backend validates body
Backend hashes or verifies password
Backend returns tokens
Frontend stores tokens
Frontend sends access token with future requests
```

Backend code:

```ts
authRoutes.post("/login", validateBody(loginSchema), login);
```

Frontend code:

```ts
const response = await api.post("/auth/login", data);
setAuth(response.data);
```

Data flow:

```text
LoginForm -> auth.api.ts -> Express route -> auth.controller.ts -> Prisma -> PostgreSQL
```

---

### Feature 2: Organizations

What it does:

Creates workspaces for teams.

How it works:

- User creates organization.
- Backend creates organization row.
- Backend creates owner membership.
- User can invite other users.

Backend code:

```ts
const organization = await prisma.organization.create({
  data: {
    name,
    slug,
    memberships: {
      create: {
        userId: req.user!.id,
        role: "OWNER",
      },
    },
  },
});
```

Why this matters:

The organization is the tenant boundary. Most permission checks start from the organization.

---

### Feature 3: Role-Based Access Control

What it does:

Protects sensitive actions based on user role.

Example:

- Viewer can read projects.
- Member can create tasks.
- Admin can invite members.
- Owner can manage billing or delete organization.

Backend code:

```ts
organizationRoutes.post(
  "/:organizationId/invite",
  requireOrganizationRole("ADMIN"),
  validateBody(inviteMemberSchema),
  inviteMember
);
```

Data flow:

```text
Request -> requireAuth -> requireOrganizationRole -> controller
```

---

### Feature 4: Projects

What it does:

Groups tasks under an organization.

Backend code:

```ts
const project = await prisma.project.create({
  data: {
    organizationId,
    name,
    description,
    color,
  },
});
```

Why projects need `organizationId`:

This ensures every project belongs to one tenant.

---

### Feature 5: Tasks

What it does:

Stores work items on a kanban board.

Important task fields:

- `title`: what needs to be done.
- `status`: kanban column.
- `priority`: urgency.
- `position`: order in column.
- `assigneeId`: who owns the task.
- `dueDate`: deadline.

Backend code:

```ts
const task = await prisma.task.create({
  data: {
    projectId,
    title,
    creatorId: req.user!.id,
    assigneeId,
    position,
  },
});
```

Frontend code:

```tsx
moveMutation.mutate({
  taskId,
  status: newStatus,
  position: 0,
});
```

---

### Feature 6: Kanban Drag And Drop

What it does:

Lets users move tasks between columns.

How it works:

```text
User drags card
dnd-kit detects drop target
Frontend sends PATCH /api/tasks/:taskId/move
Backend updates status and position
Socket.IO notifies other users
React Query refetches tasks
```

---

### Feature 7: Comments

What it does:

Lets team members discuss a task.

Backend model:

```prisma
model Comment {
  id        String   @id @default(uuid())
  taskId    String
  authorId  String
  body      String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

API routes to add:

```text
GET /api/tasks/:taskId/comments
POST /api/tasks/:taskId/comments
DELETE /api/comments/:commentId
```

Data flow:

```text
CommentForm -> POST comment -> create notification -> emit socket event -> update UI
```

---

### Feature 8: File Attachments

What it does:

Lets users upload files to a task.

Beginner option:

- Use Multer and local `uploads` folder.

Production option:

- Upload to S3 or Cloudinary.
- Store only the file URL/key in PostgreSQL.

Backend model:

```prisma
model Attachment {
  id       String @id @default(uuid())
  taskId   String
  fileName String
  fileUrl  String
  size     Int
}
```

---

### Feature 9: Real-Time Notifications

What it does:

Updates connected users immediately when tasks change.

Backend:

```ts
io.to(`project:${projectId}`).emit("task:updated", task);
```

Frontend:

```ts
socket.on("task:updated", () => {
  queryClient.invalidateQueries({ queryKey: ["tasks", projectId] });
});
```

Why this is useful:

Remote teams can see board changes without refreshing the page.

---

### Feature 10: Activity Logs

What it does:

Records important actions.

Examples:

- User created project.
- User moved task.
- User invited member.
- User uploaded attachment.

Backend:

```ts
await prisma.activityLog.create({
  data: {
    organizationId,
    actorId: req.user!.id,
    action: "TASK_UPDATED",
    entityType: "Task",
    entityId: task.id,
    metadata: req.body,
  },
});
```

Why it matters:

SaaS teams need audit history for accountability.

---

### Feature 11: Analytics Dashboard

What it does:

Shows project progress.

Useful metrics:

- Total tasks
- Completed tasks
- Overdue tasks
- Tasks by status
- Tasks by priority
- Tasks per member

Example backend query:

```ts
const grouped = await prisma.task.groupBy({
  by: ["status"],
  where: { projectId },
  _count: { id: true },
});
```

What this does:

It asks PostgreSQL to count tasks grouped by status.

---

### Feature 12: Background Jobs With BullMQ

What it does:

Runs slow work outside the normal API request.

Good use cases:

- Send invitation emails.
- Send notification emails.
- Generate reports.
- Clean expired refresh tokens.

Create `backend/src/config/redis.ts`:

```ts
import IORedis from "ioredis";
import { env } from "./env.js";

export const redisConnection = new IORedis(env.REDIS_URL, {
  maxRetriesPerRequest: null,
});
```

Create `backend/src/jobs/notification.queue.ts`:

```ts
import { Queue } from "bullmq";
import { redisConnection } from "../config/redis.js";

export const notificationQueue = new Queue("notifications", {
  connection: redisConnection,
});
```

Create `backend/src/jobs/notification.worker.ts`:

```ts
import { Worker } from "bullmq";
import { redisConnection } from "../config/redis.js";

export const notificationWorker = new Worker(
  "notifications",
  async (job) => {
    console.log("Processing notification job", job.data);
  },
  {
    connection: redisConnection,
  }
);
```

What this does:

- Queue stores jobs.
- Worker processes jobs.
- Redis coordinates the queue.

---

## 8. Authentication And Authorization

### Signup

```text
User enters name, email, password
Backend validates data
Backend checks duplicate email
Backend hashes password
Backend creates user
Backend returns access and refresh tokens
Frontend stores tokens
```

### Login

```text
User enters email and password
Backend finds user
Backend compares password with bcrypt
Backend creates token pair
Frontend stores token pair
```

### Token Handling

Access token:

- Short lived
- Sent with every protected API request
- Stored in Zustand/localStorage in this beginner version

Refresh token:

- Longer lived
- Used to get a new access token
- Stored hashed in database

Production best practice:

Store refresh token in an HTTP-only secure cookie. This reduces the risk of JavaScript stealing it.

### Protected Routes

Backend:

```ts
taskRoutes.use(requireAuth);
```

Frontend:

```tsx
<ProtectedRoute>
  <ProjectPage />
</ProtectedRoute>
```

### Logout

Logout should:

1. Delete refresh token from database.
2. Clear frontend auth state.
3. Redirect to login page.

Frontend:

```ts
useAuthStore.getState().logout();
```

---

## 9. Validation And Error Handling

### Input Validation

Backend validation:

```ts
export const createTaskSchema = z.object({
  projectId: z.string().uuid(),
  title: z.string().min(1),
});
```

Frontend validation:

```ts
const taskSchema = z.object({
  title: z.string().min(1, "Task title is required"),
});
```

Why validate both places:

- Frontend validation improves user experience.
- Backend validation protects the database.

### API Error Responses

Good error response:

```json
{
  "message": "Validation failed",
  "errors": {
    "fieldErrors": {
      "title": ["Task title is required"]
    }
  }
}
```

### Frontend Error States

Example:

```tsx
{mutation.isError && (
  <p className="text-sm text-red-600">Something went wrong.</p>
)}
```

### Common Edge Cases

- User tries to access organization they do not belong to.
- Viewer tries to create task.
- Task assignee is not a member of the organization.
- Access token expires.
- Refresh token is invalid.
- Redis is down.
- Socket disconnects.
- Two users move the same task at the same time.
- Uploaded file is too large.

---

## 10. Testing And Debugging

### Test APIs With Postman

#### Health Check

```text
GET http://localhost:5000/health
```

Expected:

```json
{
  "status": "ok"
}
```

#### Register

```text
POST http://localhost:5000/api/auth/register
```

Body:

```json
{
  "name": "Alex",
  "email": "alex@example.com",
  "password": "password123"
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
  "password": "password123"
}
```

Copy `accessToken`.

#### Create Organization

```text
POST http://localhost:5000/api/organizations
Authorization: Bearer ACCESS_TOKEN
```

Body:

```json
{
  "name": "Acme Startup"
}
```

#### Create Project

```text
POST http://localhost:5000/api/projects
Authorization: Bearer ACCESS_TOKEN
```

Body:

```json
{
  "organizationId": "ORG_ID",
  "name": "Website Redesign",
  "description": "Marketing website rebuild"
}
```

#### Create Task

```text
POST http://localhost:5000/api/tasks
Authorization: Bearer ACCESS_TOKEN
```

Body:

```json
{
  "projectId": "PROJECT_ID",
  "title": "Design landing page",
  "priority": "HIGH"
}
```

#### Move Task

```text
PATCH http://localhost:5000/api/tasks/TASK_ID/move
Authorization: Bearer ACCESS_TOKEN
```

Body:

```json
{
  "status": "IN_PROGRESS",
  "position": 0
}
```

### Test Frontend Flows

1. Register a new user.
2. Login.
3. Create an organization.
4. Create a project.
5. Open project board.
6. Create tasks.
7. Move tasks between columns.
8. Open the app in two browser windows.
9. Move a task in one window.
10. Confirm the other window updates.

### Common Bugs And Fixes

#### CORS Error

Fix:

```ts
app.use(cors({
  origin: env.CLIENT_URL,
  credentials: true,
}));
```

Also check:

```env
CLIENT_URL=http://localhost:5173
```

#### Prisma Cannot Connect

Fix:

```bash
docker compose up -d
npm run prisma:migrate
```

Check `DATABASE_URL`.

#### Access Token Unauthorized

Fix:

- Make sure frontend sends `Authorization: Bearer token`.
- Make sure `JWT_ACCESS_SECRET` did not change.
- Login again.

#### Socket Not Updating

Fix:

- Confirm backend Socket.IO server is running.
- Confirm frontend calls `socket.emit("project:join", projectId)`.
- Confirm backend emits to the same room name.

#### Drag And Drop Does Not Work

Fix:

- Confirm droppable IDs match status values.
- Confirm backend accepts status enum.
- Confirm React Query invalidates task query after mutation.

---

## 11. Running The Project

### Start Database And Redis

```bash
cd backend
docker compose up -d
```

### Run Migrations

```bash
npm run prisma:migrate -- --name init
npm run prisma:generate
```

### Run Backend

```bash
npm run dev
```

Expected:

```text
API running on http://localhost:5000
```

### Run Frontend

Open another terminal:

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

- PostgreSQL container is running.
- Redis container is running.
- Backend `/health` route returns ok.
- Frontend opens.
- Register works.
- Login works.
- Organization creation works.
- Project creation works.
- Task creation works.
- Kanban move works.
- Socket updates work in two browser windows.

---

## 12. Next Improvements

### Feature Upgrades

1. Full invitation flow with email links.
2. Google/GitHub OAuth login.
3. Task labels.
4. Task checklists.
5. Due date reminders.
6. Recurring tasks.
7. Advanced search.
8. File previews.
9. Public share links.
10. Organization settings page.
11. Member management UI.
12. Billing with Stripe.

### Performance Improvements

1. Add pagination to task, activity, and notification APIs.
2. Cache organization membership in Redis.
3. Add database indexes for common queries.
4. Move notification emails into BullMQ.
5. Add rate limiting to auth endpoints.
6. Use optimistic updates for task movement.
7. Use virtualized lists for very large boards.

### Deployment Suggestions

Frontend:

- Deploy to Vercel.
- Set `VITE_API_URL` to backend URL.

Backend:

- Deploy to Render, Railway, or Fly.io.
- Use managed PostgreSQL from Neon, Supabase, or Railway.
- Use Redis Cloud or Upstash.

Files:

- Use AWS S3 or Cloudinary.
- Store file URL and key in the database.

Docker:

- Build backend image.
- Use Nginx as reverse proxy.
- Add SSL with Let's Encrypt.

Production checklist:

- Use HTTPS.
- Use secure JWT secrets.
- Store refresh token in HTTP-only cookie.
- Add request rate limiting.
- Add logging and monitoring.
- Add database backups.
- Add error tracking such as Sentry.
- Add tests before deployment.

---

## Final Summary

You built the architecture for TaskFlow SaaS, a real-world multi-tenant project management platform.

You learned:

- How to design SaaS database models.
- How organizations and memberships create multi-tenancy.
- How JWT auth and refresh tokens work.
- How role-based access control protects data.
- How to build REST APIs with Express and TypeScript.
- How Prisma connects backend code to PostgreSQL.
- How React Query manages server data.
- How Zustand manages client auth state.
- How dnd-kit powers a kanban board.
- How Socket.IO enables real-time collaboration.
- How Redis and BullMQ support background jobs.
- How to grow an MVP into a production SaaS platform.

The simplest useful version is auth plus organizations plus projects plus tasks. Once that works, add comments, notifications, files, analytics, billing, and deployment one layer at a time.

