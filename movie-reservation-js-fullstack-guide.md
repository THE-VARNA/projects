# CineReserve - Complete Full-Stack Movie Reservation Guide

Project Name: CineReserve  
Project Type: Movie ticket reservation and seat booking system  
Tech Stack: React, Vite, Tailwind CSS, Node.js, Express, JavaScript, PostgreSQL, Prisma ORM, JWT, bcrypt, Zod, Axios  
Skill Level: Beginner

This guide uses JavaScript, not TypeScript.

---

## What We Will Build

We will build a movie reservation service where:

- Users can sign up and log in.
- Users can browse movies.
- Users can view showtimes for a specific date.
- Users can view available seats for a showtime.
- Users can reserve seats.
- Users can see their reservations.
- Users can cancel upcoming reservations.
- Admins can create, update, and delete movies.
- Admins can create showtimes.
- Admins can see reservation reports, capacity, and revenue.

The most important learning goal is understanding business logic:

- How movies relate to showtimes.
- How auditoriums contain seats.
- How reservations lock specific seats.
- How to prevent overbooking.
- How to write queries for reporting.

---

## Tech Stack Used

### Frontend

- `React`: builds the user interface.
- `Vite`: fast React project setup.
- `React Router`: handles pages like login, movies, showtime details, reservations.
- `Axios`: sends HTTP requests from frontend to backend.
- `Tailwind CSS`: styles pages quickly.

### Backend

- `Node.js`: runs JavaScript on the server.
- `Express`: creates REST API routes.
- `Prisma`: ORM for PostgreSQL.
- `Zod`: validates request data.
- `bcryptjs`: hashes passwords.
- `jsonwebtoken`: creates and verifies login tokens.
- `cors`: allows frontend and backend to communicate.
- `dotenv`: loads environment variables.

### Database

- `PostgreSQL`: relational database.

Why PostgreSQL is a good choice:

- Movie reservation systems have strong relationships.
- Seat booking needs transactions.
- Reporting needs joins and aggregations.
- We need constraints to prevent duplicate seat bookings.

### Authentication

- JWT access token
- Password hashing with bcrypt
- Role-based access control with `USER` and `ADMIN`

### Deployment

Beginner deployment path:

- Frontend: Vercel or Netlify
- Backend: Render, Railway, or Fly.io
- Database: Neon, Supabase, Railway Postgres

---

## Final Project Structure

```text
cinereserve/
|-- backend/
|   |-- prisma/
|   |   |-- schema.prisma
|   |   |-- seed.js
|   |-- src/
|   |   |-- controllers/
|   |   |   |-- authController.js
|   |   |   |-- movieController.js
|   |   |   |-- showtimeController.js
|   |   |   |-- reservationController.js
|   |   |   |-- reportController.js
|   |   |-- middleware/
|   |   |   |-- authMiddleware.js
|   |   |   |-- roleMiddleware.js
|   |   |   |-- validateMiddleware.js
|   |   |   |-- errorMiddleware.js
|   |   |-- routes/
|   |   |   |-- authRoutes.js
|   |   |   |-- movieRoutes.js
|   |   |   |-- showtimeRoutes.js
|   |   |   |-- reservationRoutes.js
|   |   |   |-- reportRoutes.js
|   |   |-- schemas/
|   |   |   |-- authSchemas.js
|   |   |   |-- movieSchemas.js
|   |   |   |-- showtimeSchemas.js
|   |   |   |-- reservationSchemas.js
|   |   |-- utils/
|   |   |   |-- generateToken.js
|   |   |-- db.js
|   |   |-- app.js
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
|   |   |   |-- SeatGrid.jsx
|   |   |   |-- MovieCard.jsx
|   |   |-- pages/
|   |   |   |-- Login.jsx
|   |   |   |-- Signup.jsx
|   |   |   |-- Movies.jsx
|   |   |   |-- MovieDetails.jsx
|   |   |   |-- ShowtimeSeats.jsx
|   |   |   |-- MyReservations.jsx
|   |   |   |-- AdminMovies.jsx
|   |   |   |-- AdminReports.jsx
|   |   |-- App.jsx
|   |   |-- main.jsx
|   |   |-- index.css
|   |-- package.json
```

Important files:

- `schema.prisma`: database models.
- `seed.js`: creates initial admin, genres, auditorium, and seats.
- `authMiddleware.js`: checks JWT token.
- `roleMiddleware.js`: blocks non-admin users from admin routes.
- `reservationController.js`: contains the most important seat booking logic.
- `SeatGrid.jsx`: lets users select seats visually.

---

## Prerequisites

Install:

- Node.js 18 or newer
- npm
- PostgreSQL
- VS Code
- Postman or Thunder Client

Check versions:

```bash
node -v
npm -v
psql --version
```

Create project folders:

```bash
mkdir cinereserve
cd cinereserve
mkdir backend frontend
```

---

## Version 1: Basic MVP

The first version includes:

- Auth
- Movies
- Showtimes
- Seats
- Reservations
- Cancellation
- Admin reports

We will build the backend first because the main challenge is data modeling and reservation logic.

---

### STEP 1: Initialize Backend

What we are doing:

Creating an Express backend project.

Why this step is needed:

The backend will store users, movies, showtimes, seats, and reservations. The frontend will call backend APIs.

Commands:

```bash
cd backend
npm init -y
npm install express cors dotenv bcryptjs jsonwebtoken zod prisma @prisma/client
npm install -D nodemon
```

Update `backend/package.json`:

```json
{
  "name": "cinereserve-backend",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "nodemon src/server.js",
    "start": "node src/server.js",
    "prisma:generate": "prisma generate",
    "prisma:migrate": "prisma migrate dev",
    "prisma:studio": "prisma studio",
    "seed": "node prisma/seed.js"
  }
}
```

Why `"type": "module"`:

It lets us use modern import syntax:

```js
import express from "express";
```

---

### STEP 2: Configure Environment Variables

Create `backend/.env`:

```env
PORT=5000
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/cinereserve
JWT_SECRET=replace_with_a_long_secret_key
CLIENT_URL=http://localhost:5173
```

What each variable means:

- `PORT`: backend server port.
- `DATABASE_URL`: PostgreSQL connection string used by Prisma.
- `JWT_SECRET`: secret key used to sign tokens.
- `CLIENT_URL`: frontend URL allowed by CORS.

Create PostgreSQL database:

```bash
createdb cinereserve
```

If you use pgAdmin, create a database named `cinereserve` manually.

---

### STEP 3: Initialize Prisma

Command:

```bash
npx prisma init
```

This creates:

```text
prisma/schema.prisma
```

We will replace it with our movie reservation schema.

---

## Database Setup

### Database Design Overview

Main entities:

- `User`
- `Genre`
- `Movie`
- `Auditorium`
- `Seat`
- `Showtime`
- `Reservation`
- `ReservationSeat`

Relationship summary:

```text
User -> many Reservations
Genre -> many Movies
Movie -> many Showtimes
Auditorium -> many Seats
Auditorium -> many Showtimes
Showtime -> many Reservations
Reservation -> many ReservationSeats
Seat -> many ReservationSeats
```

Why this model:

- Movies can have many showtimes.
- Each showtime happens in one auditorium.
- Each auditorium has fixed seats.
- A reservation belongs to one user and one showtime.
- A reservation can include many seats.
- A seat can be used in many showtimes, but only once per showtime.

---

### Prisma Schema

Replace `backend/prisma/schema.prisma`:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

enum Role {
  USER
  ADMIN
}

enum ReservationStatus {
  ACTIVE
  CANCELLED
}

model User {
  id           String        @id @default(uuid())
  name         String
  email        String        @unique
  passwordHash String
  role         Role          @default(USER)
  createdAt    DateTime      @default(now())
  updatedAt    DateTime      @updatedAt

  reservations Reservation[]
}

model Genre {
  id        String   @id @default(uuid())
  name      String   @unique
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  movies    Movie[]
}

model Movie {
  id          String     @id @default(uuid())
  title       String
  description String
  posterUrl   String?
  durationMin Int
  genreId     String
  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt

  genre       Genre      @relation(fields: [genreId], references: [id])
  showtimes   Showtime[]
}

model Auditorium {
  id        String     @id @default(uuid())
  name      String     @unique
  createdAt DateTime   @default(now())
  updatedAt DateTime   @updatedAt

  seats     Seat[]
  showtimes Showtime[]
}

model Seat {
  id           String      @id @default(uuid())
  auditoriumId String
  row          String
  number       Int
  label        String
  createdAt    DateTime    @default(now())

  auditorium   Auditorium  @relation(fields: [auditoriumId], references: [id], onDelete: Cascade)
  reservationSeats ReservationSeat[]

  @@unique([auditoriumId, row, number])
}

model Showtime {
  id           String        @id @default(uuid())
  movieId      String
  auditoriumId String
  startsAt     DateTime
  endsAt       DateTime
  priceCents   Int
  createdAt    DateTime      @default(now())
  updatedAt    DateTime      @updatedAt

  movie        Movie         @relation(fields: [movieId], references: [id], onDelete: Cascade)
  auditorium   Auditorium    @relation(fields: [auditoriumId], references: [id])
  reservations Reservation[]
  reservedSeats ReservationSeat[]

  @@index([startsAt])
}

model Reservation {
  id          String            @id @default(uuid())
  userId      String
  showtimeId  String
  status      ReservationStatus @default(ACTIVE)
  totalCents  Int
  createdAt   DateTime          @default(now())
  cancelledAt DateTime?

  user        User              @relation(fields: [userId], references: [id], onDelete: Cascade)
  showtime    Showtime          @relation(fields: [showtimeId], references: [id], onDelete: Cascade)
  seats       ReservationSeat[]
}

model ReservationSeat {
  id            String      @id @default(uuid())
  reservationId String
  showtimeId    String
  seatId        String
  seatLabel     String
  createdAt     DateTime    @default(now())

  reservation   Reservation @relation(fields: [reservationId], references: [id], onDelete: Cascade)
  showtime      Showtime    @relation(fields: [showtimeId], references: [id], onDelete: Cascade)
  seat          Seat        @relation(fields: [seatId], references: [id], onDelete: Cascade)

  @@unique([showtimeId, seatId])
}
```

### Critical Constraint For Avoiding Overbooking

This line is very important:

```prisma
@@unique([showtimeId, seatId])
```

It means:

```text
The same seat can be reserved only once for the same showtime.
```

Example:

- Seat A1 can be booked for 10 AM showtime.
- Seat A1 can also be booked for 2 PM showtime.
- Seat A1 cannot be booked twice for the same 10 AM showtime.

When a reservation is cancelled, we will delete its `ReservationSeat` rows. That makes seats available again.

Production note:

In a more advanced system, you may keep all seat history and use a PostgreSQL partial unique index for active reservations only. For a beginner system, deleting reservation-seat lock rows on cancellation is easier to understand.

---

### Run Migration

```bash
npm run prisma:migrate -- --name init
npm run prisma:generate
```

Open database browser:

```bash
npm run prisma:studio
```

---

### Seed Admin, Genres, Auditorium, And Seats

Create `backend/prisma/seed.js`:

```js
import bcrypt from "bcryptjs";
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

async function main() {
  const passwordHash = await bcrypt.hash("admin123", 10);

  await prisma.user.upsert({
    where: { email: "admin@cine.com" },
    update: {},
    create: {
      name: "Admin User",
      email: "admin@cine.com",
      passwordHash,
      role: "ADMIN",
    },
  });

  const genres = ["Action", "Comedy", "Drama", "Horror", "Sci-Fi"];

  for (const name of genres) {
    await prisma.genre.upsert({
      where: { name },
      update: {},
      create: { name },
    });
  }

  const auditorium = await prisma.auditorium.upsert({
    where: { name: "Screen 1" },
    update: {},
    create: { name: "Screen 1" },
  });

  const rows = ["A", "B", "C", "D", "E"];

  for (const row of rows) {
    for (let number = 1; number <= 10; number++) {
      await prisma.seat.upsert({
        where: {
          auditoriumId_row_number: {
            auditoriumId: auditorium.id,
            row,
            number,
          },
        },
        update: {},
        create: {
          auditoriumId: auditorium.id,
          row,
          number,
          label: `${row}${number}`,
        },
      });
    }
  }

  console.log("Seed completed");
}

main()
  .catch((error) => {
    console.error(error);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

Run seed:

```bash
npm run seed
```

Admin login:

```text
email: admin@cine.com
password: admin123
```

---

## Backend Setup

### STEP 1: Prisma Client

Create `backend/src/db.js`:

```js
import { PrismaClient } from "@prisma/client";

export const prisma = new PrismaClient();
```

What this does:

It creates a Prisma client instance. Controllers will use this to talk to PostgreSQL.

Example:

```js
await prisma.movie.findMany();
```

---

### STEP 2: Token Utility

Create `backend/src/utils/generateToken.js`:

```js
import jwt from "jsonwebtoken";

export const generateToken = (user) => {
  return jwt.sign(
    {
      userId: user.id,
      role: user.role,
    },
    process.env.JWT_SECRET,
    { expiresIn: "7d" }
  );
};
```

What this does:

- Creates a signed token.
- Stores `userId` and `role` inside the token.
- Expires after 7 days.

---

### STEP 3: Auth Middleware

Create `backend/src/middleware/authMiddleware.js`:

```js
import jwt from "jsonwebtoken";
import { prisma } from "../db.js";

export const protect = async (req, res, next) => {
  try {
    const authHeader = req.headers.authorization;

    if (!authHeader || !authHeader.startsWith("Bearer ")) {
      return res.status(401).json({ message: "No token provided" });
    }

    const token = authHeader.split(" ")[1];
    const decoded = jwt.verify(token, process.env.JWT_SECRET);

    const user = await prisma.user.findUnique({
      where: { id: decoded.userId },
      select: {
        id: true,
        name: true,
        email: true,
        role: true,
      },
    });

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

What this does:

1. Reads `Authorization` header.
2. Extracts token.
3. Verifies token.
4. Finds user.
5. Attaches user to `req.user`.
6. Allows request to continue.

---

### STEP 4: Role Middleware

Create `backend/src/middleware/roleMiddleware.js`:

```js
export const requireAdmin = (req, res, next) => {
  if (!req.user || req.user.role !== "ADMIN") {
    return res.status(403).json({ message: "Admin access required" });
  }

  next();
};
```

What this does:

It blocks regular users from admin-only routes.

Example:

```js
router.post("/", protect, requireAdmin, createMovie);
```

---

### STEP 5: Validation Middleware

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

What this does:

- Receives a Zod schema.
- Checks `req.body`.
- Returns `400` if body is invalid.
- Replaces `req.body` with clean validated data.

---

### STEP 6: Error Middleware

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

Why this is useful:

It gives consistent error responses instead of random server crashes.

---

## Authentication Routes

### Auth Schemas

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

Create `backend/src/controllers/authController.js`:

```js
import bcrypt from "bcryptjs";
import { prisma } from "../db.js";
import { generateToken } from "../utils/generateToken.js";

export const signup = async (req, res) => {
  try {
    const { name, email, password } = req.body;

    const existingUser = await prisma.user.findUnique({
      where: { email },
    });

    if (existingUser) {
      return res.status(400).json({ message: "Email already registered" });
    }

    const passwordHash = await bcrypt.hash(password, 10);

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
        role: true,
      },
    });

    const token = generateToken(user);

    res.status(201).json({ user, token });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const login = async (req, res) => {
  try {
    const { email, password } = req.body;

    const user = await prisma.user.findUnique({
      where: { email },
    });

    if (!user) {
      return res.status(401).json({ message: "Invalid email or password" });
    }

    const isValidPassword = await bcrypt.compare(password, user.passwordHash);

    if (!isValidPassword) {
      return res.status(401).json({ message: "Invalid email or password" });
    }

    const safeUser = {
      id: user.id,
      name: user.name,
      email: user.email,
      role: user.role,
    };

    const token = generateToken(user);

    res.json({ user: safeUser, token });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const me = async (req, res) => {
  res.json({ user: req.user });
};
```

Important logic:

- Password is hashed before saving.
- Login compares plain password with hashed password.
- Response never returns `passwordHash`.
- JWT token includes user ID and role.

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

Route details:

| Method | URL | Purpose |
|---|---|---|
| `POST` | `/api/auth/signup` | Create account |
| `POST` | `/api/auth/login` | Login |
| `GET` | `/api/auth/me` | Get current user |

Signup request:

```json
{
  "name": "Alex",
  "email": "alex@example.com",
  "password": "123456"
}
```

Signup response:

```json
{
  "user": {
    "id": "user-id",
    "name": "Alex",
    "email": "alex@example.com",
    "role": "USER"
  },
  "token": "jwt-token"
}
```

---

## Movie Management Routes

### Movie Schemas

Create `backend/src/schemas/movieSchemas.js`:

```js
import { z } from "zod";

export const createMovieSchema = z.object({
  title: z.string().min(1, "Title is required"),
  description: z.string().min(1, "Description is required"),
  posterUrl: z.string().url().optional(),
  durationMin: z.number().int().positive(),
  genreId: z.string().uuid(),
});

export const updateMovieSchema = createMovieSchema.partial();
```

---

### Movie Controller

Create `backend/src/controllers/movieController.js`:

```js
import { prisma } from "../db.js";

export const getMovies = async (req, res) => {
  try {
    const movies = await prisma.movie.findMany({
      include: {
        genre: true,
      },
      orderBy: {
        createdAt: "desc",
      },
    });

    res.json({ movies });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const getMovieById = async (req, res) => {
  try {
    const movie = await prisma.movie.findUnique({
      where: { id: req.params.id },
      include: {
        genre: true,
        showtimes: {
          orderBy: { startsAt: "asc" },
        },
      },
    });

    if (!movie) {
      return res.status(404).json({ message: "Movie not found" });
    }

    res.json({ movie });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const createMovie = async (req, res) => {
  try {
    const movie = await prisma.movie.create({
      data: req.body,
    });

    res.status(201).json({ movie });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const updateMovie = async (req, res) => {
  try {
    const movie = await prisma.movie.update({
      where: { id: req.params.id },
      data: req.body,
    });

    res.json({ movie });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const deleteMovie = async (req, res) => {
  try {
    await prisma.movie.delete({
      where: { id: req.params.id },
    });

    res.json({ message: "Movie deleted" });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};
```

Important logic:

- Public users can read movies.
- Only admins can create, update, and delete.
- `include: { genre: true }` joins genre data.

---

### Movie Routes

Create `backend/src/routes/movieRoutes.js`:

```js
import express from "express";
import {
  createMovie,
  deleteMovie,
  getMovieById,
  getMovies,
  updateMovie,
} from "../controllers/movieController.js";
import { protect } from "../middleware/authMiddleware.js";
import { requireAdmin } from "../middleware/roleMiddleware.js";
import { validate } from "../middleware/validateMiddleware.js";
import { createMovieSchema, updateMovieSchema } from "../schemas/movieSchemas.js";

const router = express.Router();

router.get("/", getMovies);
router.get("/:id", getMovieById);
router.post("/", protect, requireAdmin, validate(createMovieSchema), createMovie);
router.patch("/:id", protect, requireAdmin, validate(updateMovieSchema), updateMovie);
router.delete("/:id", protect, requireAdmin, deleteMovie);

export default router;
```

Route details:

| Method | URL | Purpose | Access |
|---|---|---|---|
| `GET` | `/api/movies` | List movies | Public |
| `GET` | `/api/movies/:id` | Movie details | Public |
| `POST` | `/api/movies` | Create movie | Admin |
| `PATCH` | `/api/movies/:id` | Update movie | Admin |
| `DELETE` | `/api/movies/:id` | Delete movie | Admin |

---

## Showtime Routes

### Showtime Schemas

Create `backend/src/schemas/showtimeSchemas.js`:

```js
import { z } from "zod";

export const createShowtimeSchema = z.object({
  movieId: z.string().uuid(),
  auditoriumId: z.string().uuid(),
  startsAt: z.string().datetime(),
  priceCents: z.number().int().positive(),
});
```

Why `startsAt` is a string:

JSON does not have a Date type. The frontend sends an ISO date string like:

```text
2026-06-25T18:30:00.000Z
```

The backend converts it to a JavaScript Date.

---

### Showtime Controller

Create `backend/src/controllers/showtimeController.js`:

```js
import { prisma } from "../db.js";

export const createShowtime = async (req, res) => {
  try {
    const { movieId, auditoriumId, startsAt, priceCents } = req.body;

    const movie = await prisma.movie.findUnique({
      where: { id: movieId },
    });

    if (!movie) {
      return res.status(404).json({ message: "Movie not found" });
    }

    const startDate = new Date(startsAt);
    const endDate = new Date(startDate.getTime() + movie.durationMin * 60 * 1000);

    const overlappingShowtime = await prisma.showtime.findFirst({
      where: {
        auditoriumId,
        startsAt: { lt: endDate },
        endsAt: { gt: startDate },
      },
    });

    if (overlappingShowtime) {
      return res.status(400).json({
        message: "Auditorium already has a showtime during this time",
      });
    }

    const showtime = await prisma.showtime.create({
      data: {
        movieId,
        auditoriumId,
        startsAt: startDate,
        endsAt: endDate,
        priceCents,
      },
      include: {
        movie: true,
        auditorium: true,
      },
    });

    res.status(201).json({ showtime });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const getShowtimesByDate = async (req, res) => {
  try {
    const { date } = req.query;

    if (!date) {
      return res.status(400).json({ message: "date query is required" });
    }

    const start = new Date(`${date}T00:00:00.000Z`);
    const end = new Date(`${date}T23:59:59.999Z`);

    const showtimes = await prisma.showtime.findMany({
      where: {
        startsAt: {
          gte: start,
          lte: end,
        },
      },
      include: {
        movie: {
          include: { genre: true },
        },
        auditorium: true,
      },
      orderBy: {
        startsAt: "asc",
      },
    });

    res.json({ showtimes });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const getShowtimeSeats = async (req, res) => {
  try {
    const showtime = await prisma.showtime.findUnique({
      where: { id: req.params.id },
      include: {
        auditorium: {
          include: {
            seats: {
              orderBy: [{ row: "asc" }, { number: "asc" }],
            },
          },
        },
        reservedSeats: true,
      },
    });

    if (!showtime) {
      return res.status(404).json({ message: "Showtime not found" });
    }

    const reservedSeatIds = new Set(
      showtime.reservedSeats.map((reserved) => reserved.seatId)
    );

    const seats = showtime.auditorium.seats.map((seat) => ({
      id: seat.id,
      row: seat.row,
      number: seat.number,
      label: seat.label,
      isReserved: reservedSeatIds.has(seat.id),
    }));

    res.json({
      showtimeId: showtime.id,
      seats,
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};
```

Important logic:

- `createShowtime` calculates `endsAt` from movie duration.
- It prevents overlapping showtimes in the same auditorium.
- `getShowtimeSeats` returns every seat with `isReserved`.

Overlap logic:

```js
startsAt < newEnd AND endsAt > newStart
```

This means two time ranges overlap.

---

### Showtime Routes

Create `backend/src/routes/showtimeRoutes.js`:

```js
import express from "express";
import {
  createShowtime,
  getShowtimeSeats,
  getShowtimesByDate,
} from "../controllers/showtimeController.js";
import { protect } from "../middleware/authMiddleware.js";
import { requireAdmin } from "../middleware/roleMiddleware.js";
import { validate } from "../middleware/validateMiddleware.js";
import { createShowtimeSchema } from "../schemas/showtimeSchemas.js";

const router = express.Router();

router.get("/", getShowtimesByDate);
router.get("/:id/seats", getShowtimeSeats);
router.post("/", protect, requireAdmin, validate(createShowtimeSchema), createShowtime);

export default router;
```

Route details:

| Method | URL | Purpose | Access |
|---|---|---|---|
| `GET` | `/api/showtimes?date=2026-06-25` | Showtimes for date | Public |
| `GET` | `/api/showtimes/:id/seats` | Seats for showtime | Public |
| `POST` | `/api/showtimes` | Create showtime | Admin |

---

## Reservation Management Routes

### Reservation Schema

Create `backend/src/schemas/reservationSchemas.js`:

```js
import { z } from "zod";

export const createReservationSchema = z.object({
  showtimeId: z.string().uuid(),
  seatIds: z.array(z.string().uuid()).min(1, "Select at least one seat"),
});
```

---

### Reservation Controller

Create `backend/src/controllers/reservationController.js`:

```js
import { Prisma } from "@prisma/client";
import { prisma } from "../db.js";

export const createReservation = async (req, res) => {
  try {
    const { showtimeId, seatIds } = req.body;
    const uniqueSeatIds = [...new Set(seatIds)];

    const result = await prisma.$transaction(
      async (tx) => {
        const showtime = await tx.showtime.findUnique({
          where: { id: showtimeId },
          include: {
            auditorium: true,
          },
        });

        if (!showtime) {
          throw new Error("Showtime not found");
        }

        if (showtime.startsAt <= new Date()) {
          throw new Error("Cannot reserve seats for a past showtime");
        }

        const seats = await tx.seat.findMany({
          where: {
            id: { in: uniqueSeatIds },
            auditoriumId: showtime.auditoriumId,
          },
        });

        if (seats.length !== uniqueSeatIds.length) {
          throw new Error("One or more seats are invalid for this auditorium");
        }

        const alreadyReserved = await tx.reservationSeat.findMany({
          where: {
            showtimeId,
            seatId: { in: uniqueSeatIds },
          },
        });

        if (alreadyReserved.length > 0) {
          throw new Error("One or more seats are already reserved");
        }

        const totalCents = showtime.priceCents * uniqueSeatIds.length;

        const reservation = await tx.reservation.create({
          data: {
            userId: req.user.id,
            showtimeId,
            totalCents,
          },
        });

        await tx.reservationSeat.createMany({
          data: seats.map((seat) => ({
            reservationId: reservation.id,
            showtimeId,
            seatId: seat.id,
            seatLabel: seat.label,
          })),
        });

        return tx.reservation.findUnique({
          where: { id: reservation.id },
          include: {
            seats: true,
            showtime: {
              include: {
                movie: true,
                auditorium: true,
              },
            },
          },
        });
      },
      {
        isolationLevel: Prisma.TransactionIsolationLevel.Serializable,
      }
    );

    res.status(201).json({ reservation: result });
  } catch (error) {
    if (error.code === "P2002") {
      return res.status(409).json({
        message: "One or more selected seats were just reserved by someone else",
      });
    }

    res.status(400).json({ message: error.message });
  }
};

export const getMyReservations = async (req, res) => {
  try {
    const reservations = await prisma.reservation.findMany({
      where: {
        userId: req.user.id,
      },
      include: {
        seats: true,
        showtime: {
          include: {
            movie: true,
            auditorium: true,
          },
        },
      },
      orderBy: {
        createdAt: "desc",
      },
    });

    res.json({ reservations });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const cancelReservation = async (req, res) => {
  try {
    const { id } = req.params;

    const reservation = await prisma.reservation.findFirst({
      where: {
        id,
        userId: req.user.id,
      },
      include: {
        showtime: true,
      },
    });

    if (!reservation) {
      return res.status(404).json({ message: "Reservation not found" });
    }

    if (reservation.status === "CANCELLED") {
      return res.status(400).json({ message: "Reservation already cancelled" });
    }

    if (reservation.showtime.startsAt <= new Date()) {
      return res.status(400).json({
        message: "Cannot cancel a past or already started showtime",
      });
    }

    await prisma.$transaction(async (tx) => {
      await tx.reservationSeat.deleteMany({
        where: {
          reservationId: reservation.id,
        },
      });

      await tx.reservation.update({
        where: { id: reservation.id },
        data: {
          status: "CANCELLED",
          cancelledAt: new Date(),
        },
      });
    });

    res.json({ message: "Reservation cancelled" });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};
```

### Why This Prevents Overbooking

We use three protections:

1. We check if seats are already reserved.
2. We create reservation and seat locks inside a transaction.
3. We have a unique database constraint on `showtimeId + seatId`.

Even if two users click the same seat at exactly the same time:

- One transaction will succeed.
- The other transaction will fail with unique constraint error.

This is the most important part of the project.

---

### Reservation Routes

Create `backend/src/routes/reservationRoutes.js`:

```js
import express from "express";
import {
  cancelReservation,
  createReservation,
  getMyReservations,
} from "../controllers/reservationController.js";
import { protect } from "../middleware/authMiddleware.js";
import { validate } from "../middleware/validateMiddleware.js";
import { createReservationSchema } from "../schemas/reservationSchemas.js";

const router = express.Router();

router.use(protect);

router.post("/", validate(createReservationSchema), createReservation);
router.get("/my", getMyReservations);
router.patch("/:id/cancel", cancelReservation);

export default router;
```

Route details:

| Method | URL | Purpose | Access |
|---|---|---|---|
| `POST` | `/api/reservations` | Reserve seats | Logged-in user |
| `GET` | `/api/reservations/my` | My reservations | Logged-in user |
| `PATCH` | `/api/reservations/:id/cancel` | Cancel upcoming reservation | Owner |

Reservation request:

```json
{
  "showtimeId": "showtime-id",
  "seatIds": ["seat-id-1", "seat-id-2"]
}
```

---

## Reporting Routes

Admins need reports for:

- All reservations
- Capacity
- Revenue
- Reservation counts by movie

Create `backend/src/controllers/reportController.js`:

```js
import { prisma } from "../db.js";

export const getAllReservations = async (req, res) => {
  try {
    const reservations = await prisma.reservation.findMany({
      include: {
        user: {
          select: {
            id: true,
            name: true,
            email: true,
          },
        },
        seats: true,
        showtime: {
          include: {
            movie: true,
            auditorium: true,
          },
        },
      },
      orderBy: {
        createdAt: "desc",
      },
    });

    res.json({ reservations });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const getRevenueReport = async (req, res) => {
  try {
    const activeReservations = await prisma.reservation.findMany({
      where: {
        status: "ACTIVE",
      },
      include: {
        seats: true,
      },
    });

    const totalRevenueCents = activeReservations.reduce(
      (sum, reservation) => sum + reservation.totalCents,
      0
    );

    res.json({
      totalReservations: activeReservations.length,
      totalTickets: activeReservations.reduce(
        (sum, reservation) => sum + reservation.seats.length,
        0
      ),
      totalRevenueCents,
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

export const getShowtimeCapacity = async (req, res) => {
  try {
    const { showtimeId } = req.params;

    const showtime = await prisma.showtime.findUnique({
      where: { id: showtimeId },
      include: {
        auditorium: {
          include: {
            seats: true,
          },
        },
        reservedSeats: true,
      },
    });

    if (!showtime) {
      return res.status(404).json({ message: "Showtime not found" });
    }

    const totalSeats = showtime.auditorium.seats.length;
    const reservedSeats = showtime.reservedSeats.length;

    res.json({
      showtimeId,
      totalSeats,
      reservedSeats,
      availableSeats: totalSeats - reservedSeats,
      occupancyRate: totalSeats === 0 ? 0 : reservedSeats / totalSeats,
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};
```

Create `backend/src/routes/reportRoutes.js`:

```js
import express from "express";
import {
  getAllReservations,
  getRevenueReport,
  getShowtimeCapacity,
} from "../controllers/reportController.js";
import { protect } from "../middleware/authMiddleware.js";
import { requireAdmin } from "../middleware/roleMiddleware.js";

const router = express.Router();

router.use(protect, requireAdmin);

router.get("/reservations", getAllReservations);
router.get("/revenue", getRevenueReport);
router.get("/showtimes/:showtimeId/capacity", getShowtimeCapacity);

export default router;
```

Route details:

| Method | URL | Purpose |
|---|---|---|
| `GET` | `/api/reports/reservations` | All reservations |
| `GET` | `/api/reports/revenue` | Revenue report |
| `GET` | `/api/reports/showtimes/:id/capacity` | Capacity report |

---

## Server Setup

Create `backend/src/app.js`:

```js
import express from "express";
import cors from "cors";
import authRoutes from "./routes/authRoutes.js";
import movieRoutes from "./routes/movieRoutes.js";
import showtimeRoutes from "./routes/showtimeRoutes.js";
import reservationRoutes from "./routes/reservationRoutes.js";
import reportRoutes from "./routes/reportRoutes.js";
import { errorHandler, notFound } from "./middleware/errorMiddleware.js";

export const app = express();

app.use(
  cors({
    origin: process.env.CLIENT_URL,
    credentials: true,
  })
);

app.use(express.json());

app.get("/", (req, res) => {
  res.json({ message: "CineReserve API is running" });
});

app.use("/api/auth", authRoutes);
app.use("/api/movies", movieRoutes);
app.use("/api/showtimes", showtimeRoutes);
app.use("/api/reservations", reservationRoutes);
app.use("/api/reports", reportRoutes);

app.use(notFound);
app.use(errorHandler);
```

Create `backend/src/server.js`:

```js
import dotenv from "dotenv";
import { app } from "./app.js";

dotenv.config();

const PORT = process.env.PORT || 5000;

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

Run backend:

```bash
npm run dev
```

Expected:

```text
Server running on http://localhost:5000
```

---

## Frontend Setup

The frontend is optional for a backend-focused project, but it helps you test the full user flow.

### STEP 1: Create React App

```bash
cd ../frontend
npm create vite@latest . -- --template react
npm install
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
}
```

---

### STEP 2: Axios Client

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

- Sets backend URL once.
- Automatically adds JWT token to requests.

---

### STEP 3: Protected Route

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

---

### STEP 4: Signup Page

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
      navigate("/movies");
    } catch (err) {
      setError(err.response?.data?.message || "Signup failed");
    } finally {
      setLoading(false);
    }
  };

  return (
    <main className="min-h-screen flex items-center justify-center px-4">
      <form onSubmit={handleSubmit} className="bg-white border rounded-lg p-6 w-full max-w-md space-y-4">
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
          Already have account? <Link to="/login" className="text-blue-600">Login</Link>
        </p>
      </form>
    </main>
  );
}
```

---

### STEP 5: Login Page

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
      navigate("/movies");
    } catch (err) {
      setError(err.response?.data?.message || "Login failed");
    } finally {
      setLoading(false);
    }
  };

  return (
    <main className="min-h-screen flex items-center justify-center px-4">
      <form onSubmit={handleSubmit} className="bg-white border rounded-lg p-6 w-full max-w-md space-y-4">
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
    </main>
  );
}
```

---

### STEP 6: Movies Page

Create `frontend/src/pages/Movies.jsx`:

```jsx
import { useEffect, useState } from "react";
import { Link } from "react-router-dom";
import api from "../api/axios";

export default function Movies() {
  const [movies, setMovies] = useState([]);
  const [date, setDate] = useState(new Date().toISOString().slice(0, 10));
  const [showtimes, setShowtimes] = useState([]);

  const fetchMovies = async () => {
    const res = await api.get("/movies");
    setMovies(res.data.movies);
  };

  const fetchShowtimes = async () => {
    const res = await api.get(`/showtimes?date=${date}`);
    setShowtimes(res.data.showtimes);
  };

  useEffect(() => {
    fetchMovies();
  }, []);

  useEffect(() => {
    fetchShowtimes();
  }, [date]);

  return (
    <main className="p-6 max-w-6xl mx-auto">
      <h1 className="text-3xl font-bold">Movies</h1>

      <div className="mt-4">
        <label className="text-sm font-medium">Showtimes date</label>
        <input
          type="date"
          value={date}
          onChange={(e) => setDate(e.target.value)}
          className="ml-3 border rounded-md px-3 py-2"
        />
      </div>

      <div className="grid md:grid-cols-3 gap-4 mt-6">
        {movies.map((movie) => (
          <div key={movie.id} className="bg-white border rounded-lg p-4">
            <div className="h-48 bg-gray-200 rounded-md mb-4" />
            <h2 className="font-bold text-xl">{movie.title}</h2>
            <p className="text-sm text-gray-600">{movie.genre?.name}</p>
            <p className="mt-2 text-sm">{movie.description}</p>
          </div>
        ))}
      </div>

      <h2 className="text-2xl font-bold mt-10">Showtimes</h2>

      <div className="mt-4 space-y-3">
        {showtimes.map((showtime) => (
          <div key={showtime.id} className="bg-white border rounded-lg p-4 flex justify-between">
            <div>
              <h3 className="font-semibold">{showtime.movie.title}</h3>
              <p className="text-sm text-gray-600">
                {new Date(showtime.startsAt).toLocaleString()} - {showtime.auditorium.name}
              </p>
            </div>

            <Link
              to={`/showtimes/${showtime.id}/seats`}
              className="bg-blue-600 text-white px-4 py-2 rounded-md"
            >
              Select Seats
            </Link>
          </div>
        ))}
      </div>
    </main>
  );
}
```

---

### STEP 7: Seat Grid Component

Create `frontend/src/components/SeatGrid.jsx`:

```jsx
export default function SeatGrid({ seats, selectedSeatIds, onToggleSeat }) {
  return (
    <div>
      <div className="bg-gray-800 text-white text-center py-2 rounded-md mb-6">
        Screen
      </div>

      <div className="grid grid-cols-10 gap-2">
        {seats.map((seat) => {
          const selected = selectedSeatIds.includes(seat.id);

          return (
            <button
              key={seat.id}
              disabled={seat.isReserved}
              onClick={() => onToggleSeat(seat.id)}
              className={[
                "p-2 rounded-md text-sm border",
                seat.isReserved ? "bg-red-200 cursor-not-allowed" : "",
                selected ? "bg-blue-600 text-white" : "",
                !seat.isReserved && !selected ? "bg-white hover:bg-blue-50" : "",
              ].join(" ")}
            >
              {seat.label}
            </button>
          );
        })}
      </div>
    </div>
  );
}
```

What this does:

- Shows seats in a grid.
- Reserved seats are disabled.
- Selected seats are highlighted.

---

### STEP 8: Showtime Seats Page

Create `frontend/src/pages/ShowtimeSeats.jsx`:

```jsx
import { useEffect, useState } from "react";
import { useNavigate, useParams } from "react-router-dom";
import api from "../api/axios";
import SeatGrid from "../components/SeatGrid";

export default function ShowtimeSeats() {
  const { id } = useParams();
  const navigate = useNavigate();
  const [seats, setSeats] = useState([]);
  const [selectedSeatIds, setSelectedSeatIds] = useState([]);
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);

  const fetchSeats = async () => {
    const res = await api.get(`/showtimes/${id}/seats`);
    setSeats(res.data.seats);
  };

  const toggleSeat = (seatId) => {
    setSelectedSeatIds((current) =>
      current.includes(seatId)
        ? current.filter((id) => id !== seatId)
        : [...current, seatId]
    );
  };

  const reserveSeats = async () => {
    setError("");

    if (!selectedSeatIds.length) {
      setError("Please select at least one seat");
      return;
    }

    try {
      setLoading(true);
      await api.post("/reservations", {
        showtimeId: id,
        seatIds: selectedSeatIds,
      });
      navigate("/my-reservations");
    } catch (err) {
      setError(err.response?.data?.message || "Reservation failed");
      fetchSeats();
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchSeats();
  }, [id]);

  return (
    <main className="p-6 max-w-4xl mx-auto">
      <h1 className="text-3xl font-bold mb-6">Select Seats</h1>

      {error && <p className="mb-4 text-red-600">{error}</p>}

      <SeatGrid
        seats={seats}
        selectedSeatIds={selectedSeatIds}
        onToggleSeat={toggleSeat}
      />

      <button
        onClick={reserveSeats}
        disabled={loading}
        className="mt-6 bg-green-600 text-white px-5 py-2 rounded-md disabled:opacity-60"
      >
        {loading ? "Reserving..." : "Reserve Seats"}
      </button>
    </main>
  );
}
```

Important UI logic:

- If reservation fails because another user booked a seat, we refetch seats.
- This gives the user updated availability.

---

### STEP 9: My Reservations Page

Create `frontend/src/pages/MyReservations.jsx`:

```jsx
import { useEffect, useState } from "react";
import api from "../api/axios";

export default function MyReservations() {
  const [reservations, setReservations] = useState([]);

  const fetchReservations = async () => {
    const res = await api.get("/reservations/my");
    setReservations(res.data.reservations);
  };

  const cancelReservation = async (id) => {
    const confirmed = confirm("Cancel this reservation?");

    if (!confirmed) return;

    await api.patch(`/reservations/${id}/cancel`);
    fetchReservations();
  };

  useEffect(() => {
    fetchReservations();
  }, []);

  return (
    <main className="p-6 max-w-5xl mx-auto">
      <h1 className="text-3xl font-bold mb-6">My Reservations</h1>

      <div className="space-y-4">
        {reservations.map((reservation) => (
          <div key={reservation.id} className="bg-white border rounded-lg p-4">
            <h2 className="font-bold">{reservation.showtime.movie.title}</h2>
            <p className="text-sm text-gray-600">
              {new Date(reservation.showtime.startsAt).toLocaleString()}
            </p>
            <p className="mt-2">
              Seats: {reservation.seats.map((seat) => seat.seatLabel).join(", ")}
            </p>
            <p>Status: {reservation.status}</p>

            {reservation.status === "ACTIVE" && (
              <button
                onClick={() => cancelReservation(reservation.id)}
                className="mt-3 text-red-600"
              >
                Cancel
              </button>
            )}
          </div>
        ))}
      </div>
    </main>
  );
}
```

---

### STEP 10: App Routes

Update `frontend/src/App.jsx`:

```jsx
import { BrowserRouter, Link, Route, Routes } from "react-router-dom";
import Login from "./pages/Login";
import Signup from "./pages/Signup";
import Movies from "./pages/Movies";
import ShowtimeSeats from "./pages/ShowtimeSeats";
import MyReservations from "./pages/MyReservations";
import ProtectedRoute from "./components/ProtectedRoute";

function Navbar() {
  const logout = () => {
    localStorage.removeItem("token");
    localStorage.removeItem("user");
    window.location.href = "/login";
  };

  return (
    <nav className="bg-white border-b px-6 py-4 flex justify-between">
      <Link to="/movies" className="font-bold">CineReserve</Link>
      <div className="flex gap-4">
        <Link to="/movies">Movies</Link>
        <Link to="/my-reservations">My Reservations</Link>
        <button onClick={logout}>Logout</button>
      </div>
    </nav>
  );
}

export default function App() {
  return (
    <BrowserRouter>
      <Navbar />
      <Routes>
        <Route path="/login" element={<Login />} />
        <Route path="/signup" element={<Signup />} />
        <Route path="/movies" element={<Movies />} />
        <Route
          path="/showtimes/:id/seats"
          element={
            <ProtectedRoute>
              <ShowtimeSeats />
            </ProtectedRoute>
          }
        />
        <Route
          path="/my-reservations"
          element={
            <ProtectedRoute>
              <MyReservations />
            </ProtectedRoute>
          }
        />
      </Routes>
    </BrowserRouter>
  );
}
```

---

## Authentication, If Needed

This project requires authentication.

### Signup Flow

```text
User submits signup form
Backend validates body
Backend checks duplicate email
Backend hashes password
Backend creates user
Backend returns JWT token
Frontend stores token in localStorage
```

### Login Flow

```text
User submits email/password
Backend finds user
Backend compares password with bcrypt
Backend returns JWT token
Frontend stores token
```

### Protected Routes

Backend protected route:

```js
router.post("/", protect, createReservation);
```

Frontend protected route:

```jsx
<ProtectedRoute>
  <ShowtimeSeats />
</ProtectedRoute>
```

### Logout Flow

```text
Remove token from localStorage
Redirect user to login page
```

---

## Version 2: Add Important Features

### Feature 1: Payment Processing

Add payment status to reservation:

```prisma
enum PaymentStatus {
  PENDING
  PAID
  FAILED
  REFUNDED
}
```

Use Stripe later:

```text
Create reservation as pending
Create Stripe payment intent
Confirm payment
Mark reservation as paid
```

Important:

For real payments, do not mark paid from frontend only. Use payment provider webhooks.

---

### Feature 2: Email Confirmation

When reservation is created:

```text
Create reservation
Send confirmation email
```

Use:

- Nodemailer for local testing
- SendGrid/Mailgun for production

---

### Feature 3: Seat Hold Timer

Real booking systems often hold seats for 5 to 10 minutes before payment.

Add:

- `SeatHold` table
- Expiration time
- Background job to release expired holds

This prevents someone from taking a seat while another user is paying.

---

### Feature 4: Admin Dashboard

Add frontend pages for:

- Create movie
- Create showtime
- View all reservations
- View capacity report
- View revenue report

---

## Running The Application

### Run Backend

```bash
cd backend
npm run dev
```

Expected:

```text
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

### Run Database Migration And Seed

```bash
cd backend
npm run prisma:migrate -- --name init
npm run seed
```

---

## Testing The Application Manually

### Test Auth

1. POST `/api/auth/signup`
2. POST `/api/auth/login`
3. Copy token
4. GET `/api/auth/me` with `Authorization: Bearer TOKEN`

### Test Admin Movie Flow

1. Login as `admin@cine.com`
2. Create a movie using `POST /api/movies`
3. Create a showtime using `POST /api/showtimes`
4. Check `GET /api/showtimes?date=YYYY-MM-DD`

### Test Reservation Flow

1. Login as regular user.
2. Get showtime seats.
3. Select seats.
4. POST `/api/reservations`.
5. Refresh seats.
6. Confirm selected seats are now reserved.
7. Try reserving same seat again.
8. Confirm backend rejects it.

### Test Cancellation

1. GET `/api/reservations/my`.
2. Cancel an upcoming reservation.
3. Refresh showtime seats.
4. Confirm seats become available again.

### Test Reports

1. Login as admin.
2. GET `/api/reports/reservations`.
3. GET `/api/reports/revenue`.
4. GET `/api/reports/showtimes/:id/capacity`.

---

## Key Concepts For Beginners

### REST API

A REST API exposes backend actions through URLs.

Example:

```text
POST /api/reservations
```

means:

Create a reservation.

### HTTP Methods

- `GET`: read data
- `POST`: create data
- `PATCH`: update part of data
- `DELETE`: delete data

### Request And Response

Request:

```json
{
  "showtimeId": "abc",
  "seatIds": ["seat1", "seat2"]
}
```

Response:

```json
{
  "reservation": {
    "id": "reservation-id"
  }
}
```

### ORM

Prisma is an ORM.

ORM means Object Relational Mapper.

It lets JavaScript talk to SQL tables using code like:

```js
await prisma.movie.findMany();
```

instead of manually writing:

```sql
SELECT * FROM movies;
```

### Transactions

A transaction groups multiple database operations.

In reservation:

```text
Create reservation
Create reserved seats
```

Both must succeed together. If one fails, everything rolls back.

### Unique Constraint

This prevents duplicate data.

```prisma
@@unique([showtimeId, seatId])
```

This is how we prevent overbooking.

### Middleware

Middleware runs before the controller.

Example:

```js
protect -> requireAdmin -> createMovie
```

### CORS

CORS allows frontend and backend on different ports to communicate.

Frontend:

```text
http://localhost:5173
```

Backend:

```text
http://localhost:5000
```

### Authentication

Authentication answers:

```text
Who are you?
```

### Authorization

Authorization answers:

```text
What are you allowed to do?
```

---

## Common Errors And Solutions

### Server Not Running

Problem:

Frontend cannot call API.

Fix:

```bash
cd backend
npm run dev
```

### Database Connection Error

Problem:

Prisma cannot connect.

Fix:

- Check PostgreSQL is running.
- Check `DATABASE_URL`.
- Check database exists.

### CORS Error

Problem:

Browser blocks request.

Fix:

Check backend:

```js
app.use(cors({ origin: process.env.CLIENT_URL }));
```

Check `.env`:

```env
CLIENT_URL=http://localhost:5173
```

### Token Error

Problem:

Protected route says unauthorized.

Fix:

- Login again.
- Check localStorage has token.
- Check Axios interceptor adds `Authorization` header.

### Seat Already Reserved

Problem:

Reservation fails.

Reason:

Another user already booked the seat.

Fix:

Refresh seats and choose another seat.

### Migration Error

Problem:

Prisma migration fails.

Fix:

```bash
npm run prisma:generate
npm run prisma:migrate -- --name init
```

If local database is disposable:

```bash
npx prisma migrate reset
```

Warning:

This deletes local data.

---

## Final Summary

You built a complete movie reservation system using JavaScript.

You learned how to:

- Build an Express backend.
- Use PostgreSQL with Prisma.
- Design relational data models.
- Build authentication with JWT and bcrypt.
- Create admin-only routes.
- Manage movies and showtimes.
- Model auditoriums and seats.
- Reserve multiple seats.
- Prevent overbooking with transactions and unique constraints.
- Cancel upcoming reservations.
- Build reports for revenue and capacity.
- Connect a React frontend to the backend.

The most important concept in this project is:

```text
Seat reservation must be protected at the database level.
```

Frontend checks are helpful, but the database constraint is what truly prevents overbooking.

---

## Next Improvements

Useful next features:

1. Online payment with Stripe.
2. Seat hold timer before payment.
3. Email ticket confirmation.
4. QR code ticket generation.
5. Admin dashboard UI.
6. Multiple theaters and locations.
7. Movie search and filters.
8. Refunds.
9. Promo codes.
10. Seat pricing by row.
11. Background jobs for expired holds.
12. Deployment with Vercel, Render, and Neon.

