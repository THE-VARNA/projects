# Scalable E-Commerce Microservices Platform - Complete Project Guide

Project Name: ScaleShop  
Project Type: E-commerce platform using microservices and Docker  
Skill Level: Beginner to production-ready path  

Tech Stack:

- Frontend: React, TypeScript, Vite, Tailwind CSS, React Router, TanStack Query
- Backend: Node.js, Express, TypeScript, REST APIs, Zod validation
- Databases: PostgreSQL per service, Redis for cart/cache/session-like data
- ORM: Prisma
- Auth: JWT access tokens, refresh tokens, bcrypt password hashing, role-based access control
- Messaging: RabbitMQ for async service communication
- API Gateway: NGINX for local gateway and reverse proxy
- File/media storage: local storage for beginner version, AWS S3 or Cloudinary for production
- Payments: mock payment provider first, Stripe-style integration later
- Notifications: mock email logger first, SendGrid/Twilio later
- Observability: centralized logs, Prometheus/Grafana concepts, health checks
- Deployment: Docker, Docker Compose, GitHub Actions, Kubernetes or Docker Swarm for production

---

## Important Learning Approach

Microservices are powerful, but they add complexity. A beginner should not start by building every production feature at once.

We will build in versions:

Version 1: Local MVP with Docker Compose

- API Gateway
- User Service
- Product Service
- Cart Service
- Order Service
- Payment Service
- Notification Service
- PostgreSQL databases
- Redis
- RabbitMQ
- React frontend

Version 2: Reliability and production patterns

- Async order events
- Idempotent payment handling
- Better error handling
- Centralized logging
- Health checks
- Service discovery concepts
- Monitoring

Version 3: Production deployment

- CI/CD
- Image registry
- Kubernetes or Docker Swarm
- Managed PostgreSQL
- Managed Redis
- Cloud object storage
- Real payment provider
- Real email/SMS provider

---

## 1. Project Overview

### What The App Does

ScaleShop is a scalable e-commerce platform. Customers can browse products, register/login, add items to cart, place orders, make payments, and receive notifications.

Admins can manage products, categories, and inventory.

Each main business capability is separated into its own microservice.

### Main Features

Customer features:

- Register and login
- Browse products
- Search/filter products
- View product details
- Add products to cart
- Update cart quantities
- Checkout
- Place order
- Pay for order
- View order history
- Receive order confirmation notification

Admin features:

- Create products
- Update products
- Manage inventory
- View orders

System features:

- API Gateway routing
- Independent microservice databases
- Service-to-service REST calls
- Async events using RabbitMQ
- Dockerized services
- Health checks
- Centralized environment configuration
- Production deployment plan

### What The User Will Learn

You will learn:

- What microservices are and when to use them
- How to split an e-commerce app into services
- How Docker Compose connects multiple containers
- Why each service should own its own database
- How REST APIs work between services
- How async events work with RabbitMQ
- How JWT auth works across microservices
- How Redis can store shopping carts
- How order/payment workflows are coordinated
- How frontend communicates through an API gateway
- How to prepare services for production

---

## Microservices Architecture Explained

### Monolith vs Microservices

A monolith puts all features in one backend:

```text
Single backend
|-- auth
|-- products
|-- cart
|-- orders
|-- payments
|-- notifications
```

A microservices architecture splits features:

```text
Frontend
   |
API Gateway
   |
   |-- User Service
   |-- Product Service
   |-- Cart Service
   |-- Order Service
   |-- Payment Service
   |-- Notification Service
```

### Why Use Microservices

Benefits:

- Services can be developed independently.
- Services can be deployed independently.
- Product service can scale separately from payment service.
- Teams can own separate services.
- Failures can be isolated better.

Tradeoffs:

- More moving parts.
- Harder debugging.
- Network calls can fail.
- Data consistency is more difficult.
- Requires better monitoring and automation.

Beginner rule:

Use microservices when you want to learn distributed systems or when your app/team is large enough to justify the complexity.

---

## 2. Folder Structure

Recommended structure:

```text
scaleshop/
|-- apps/
|   |-- web/
|   |   |-- src/
|   |   |   |-- api/
|   |   |   |-- components/
|   |   |   |-- pages/
|   |   |   |-- routes/
|   |   |   |-- stores/
|   |   |   |-- types/
|   |   |   |-- App.tsx
|   |   |   |-- main.tsx
|   |   |-- Dockerfile
|   |   |-- package.json
|
|-- services/
|   |-- user-service/
|   |   |-- prisma/
|   |   |   |-- schema.prisma
|   |   |-- src/
|   |   |   |-- controllers/
|   |   |   |-- middleware/
|   |   |   |-- routes/
|   |   |   |-- schemas/
|   |   |   |-- lib/
|   |   |   |-- app.ts
|   |   |   |-- server.ts
|   |   |-- Dockerfile
|   |   |-- package.json
|   |
|   |-- product-service/
|   |   |-- prisma/
|   |   |-- src/
|   |   |-- Dockerfile
|   |   |-- package.json
|   |
|   |-- cart-service/
|   |   |-- src/
|   |   |-- Dockerfile
|   |   |-- package.json
|   |
|   |-- order-service/
|   |   |-- prisma/
|   |   |-- src/
|   |   |-- Dockerfile
|   |   |-- package.json
|   |
|   |-- payment-service/
|   |   |-- prisma/
|   |   |-- src/
|   |   |-- Dockerfile
|   |   |-- package.json
|   |
|   |-- notification-service/
|   |   |-- src/
|   |   |-- Dockerfile
|   |   |-- package.json
|
|-- gateway/
|   |-- nginx.conf
|   |-- Dockerfile
|
|-- packages/
|   |-- common/
|   |   |-- src/
|   |   |   |-- errors.ts
|   |   |   |-- logger.ts
|   |   |   |-- events.ts
|   |   |-- package.json
|
|-- docker-compose.yml
|-- .env.example
|-- README.md
```

### What Each Main Folder Does

- `apps/web`: React frontend.
- `services/user-service`: registration, login, profile, auth token validation.
- `services/product-service`: products, categories, inventory.
- `services/cart-service`: shopping cart data stored in Redis.
- `services/order-service`: order creation and order history.
- `services/payment-service`: payment creation and payment status.
- `services/notification-service`: sends emails/SMS or logs notifications.
- `gateway`: NGINX API gateway.
- `packages/common`: optional shared TypeScript utilities.

### Why Use Separate Service Folders

Each service should be independently buildable and deployable.

That means each service has its own:

- `package.json`
- `Dockerfile`
- source code
- environment variables
- database connection if needed

---

## 3. Setup Instructions

### Prerequisites

Install:

- Node.js 20 or newer
- npm
- Docker Desktop
- Git
- VS Code
- Postman or Thunder Client

Check versions:

```bash
node -v
npm -v
docker -v
docker compose version
```

Create project:

```bash
mkdir scaleshop
cd scaleshop
mkdir apps services gateway packages
```

---

## Docker Compose Setup

Docker Compose will run:

- API gateway
- frontend
- all microservices
- PostgreSQL databases
- Redis
- RabbitMQ

Create `docker-compose.yml` in the root:

```yaml
services:
  gateway:
    build: ./gateway
    ports:
      - "8080:80"
    depends_on:
      - user-service
      - product-service
      - cart-service
      - order-service
      - payment-service
      - notification-service

  web:
    build: ./apps/web
    ports:
      - "5173:5173"
    environment:
      - VITE_API_URL=http://localhost:8080
    volumes:
      - ./apps/web:/app
      - /app/node_modules
    command: npm run dev -- --host 0.0.0.0

  user-service:
    build: ./services/user-service
    ports:
      - "3001:3001"
    environment:
      - PORT=3001
      - DATABASE_URL=postgresql://postgres:postgres@user-db:5432/userdb
      - JWT_ACCESS_SECRET=change_this_access_secret
      - JWT_REFRESH_SECRET=change_this_refresh_secret
      - RABBITMQ_URL=amqp://rabbitmq:5672
    depends_on:
      - user-db
      - rabbitmq

  product-service:
    build: ./services/product-service
    ports:
      - "3002:3002"
    environment:
      - PORT=3002
      - DATABASE_URL=postgresql://postgres:postgres@product-db:5432/productdb
      - RABBITMQ_URL=amqp://rabbitmq:5672
    depends_on:
      - product-db
      - rabbitmq

  cart-service:
    build: ./services/cart-service
    ports:
      - "3003:3003"
    environment:
      - PORT=3003
      - REDIS_URL=redis://redis:6379
      - PRODUCT_SERVICE_URL=http://product-service:3002
    depends_on:
      - redis
      - product-service

  order-service:
    build: ./services/order-service
    ports:
      - "3004:3004"
    environment:
      - PORT=3004
      - DATABASE_URL=postgresql://postgres:postgres@order-db:5432/orderdb
      - CART_SERVICE_URL=http://cart-service:3003
      - PRODUCT_SERVICE_URL=http://product-service:3002
      - PAYMENT_SERVICE_URL=http://payment-service:3005
      - RABBITMQ_URL=amqp://rabbitmq:5672
    depends_on:
      - order-db
      - cart-service
      - payment-service
      - rabbitmq

  payment-service:
    build: ./services/payment-service
    ports:
      - "3005:3005"
    environment:
      - PORT=3005
      - DATABASE_URL=postgresql://postgres:postgres@payment-db:5432/paymentdb
      - RABBITMQ_URL=amqp://rabbitmq:5672
      - STRIPE_SECRET_KEY=mock_key_for_local_dev
    depends_on:
      - payment-db
      - rabbitmq

  notification-service:
    build: ./services/notification-service
    ports:
      - "3006:3006"
    environment:
      - PORT=3006
      - RABBITMQ_URL=amqp://rabbitmq:5672
    depends_on:
      - rabbitmq

  user-db:
    image: postgres:16
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: userdb
    ports:
      - "5433:5432"
    volumes:
      - user_db_data:/var/lib/postgresql/data

  product-db:
    image: postgres:16
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: productdb
    ports:
      - "5434:5432"
    volumes:
      - product_db_data:/var/lib/postgresql/data

  order-db:
    image: postgres:16
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: orderdb
    ports:
      - "5435:5432"
    volumes:
      - order_db_data:/var/lib/postgresql/data

  payment-db:
    image: postgres:16
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: paymentdb
    ports:
      - "5436:5432"
    volumes:
      - payment_db_data:/var/lib/postgresql/data

  redis:
    image: redis:7
    ports:
      - "6379:6379"

  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"
      - "15672:15672"

volumes:
  user_db_data:
  product_db_data:
  order_db_data:
  payment_db_data:
```

### What This Docker Compose File Does

- Builds every app/service from its folder.
- Creates one PostgreSQL database per service.
- Creates Redis for cart data.
- Creates RabbitMQ for async events.
- Exposes NGINX gateway at `http://localhost:8080`.
- Exposes RabbitMQ dashboard at `http://localhost:15672`.

Important concept:

Inside Docker Compose, services call each other using service names.

Example:

```text
http://product-service:3002
```

This works inside the Docker network. From your browser, you use:

```text
http://localhost:8080
```

---

## API Gateway Setup

Create `gateway/nginx.conf`:

```nginx
events {}

http {
  upstream user_service {
    server user-service:3001;
  }

  upstream product_service {
    server product-service:3002;
  }

  upstream cart_service {
    server cart-service:3003;
  }

  upstream order_service {
    server order-service:3004;
  }

  upstream payment_service {
    server payment-service:3005;
  }

  server {
    listen 80;

    location /api/users/ {
      proxy_pass http://user_service/;
    }

    location /api/products/ {
      proxy_pass http://product_service/;
    }

    location /api/cart/ {
      proxy_pass http://cart_service/;
    }

    location /api/orders/ {
      proxy_pass http://order_service/;
    }

    location /api/payments/ {
      proxy_pass http://payment_service/;
    }

    location /health {
      return 200 "gateway ok";
    }
  }
}
```

Create `gateway/Dockerfile`:

```dockerfile
FROM nginx:1.27-alpine
COPY nginx.conf /etc/nginx/nginx.conf
```

What this does:

- NGINX listens on port `80` inside the container.
- Docker maps it to `localhost:8080`.
- Requests are routed by path.

Example:

```text
GET http://localhost:8080/api/products
```

goes to:

```text
product-service:3002
```

---

## 4. Database Design

### Database Per Service

In microservices, each service should own its own data.

Bad design:

```text
All services share one database
```

Better microservice design:

```text
User Service      -> user-db
Product Service   -> product-db
Order Service     -> order-db
Payment Service   -> payment-db
Cart Service      -> Redis
Notification      -> RabbitMQ events, optional notification-db
```

Why:

- Services are independent.
- Database schema changes in one service do not break every other service.
- Each service controls its own data rules.

Tradeoff:

You cannot easily do SQL joins across services. Services communicate through APIs or events instead.

---

### User Service Database

Tables:

- `User`
- `RefreshToken`

`services/user-service/prisma/schema.prisma`:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

enum UserRole {
  CUSTOMER
  ADMIN
}

model User {
  id           String         @id @default(uuid())
  name         String
  email        String         @unique
  passwordHash String
  role         UserRole       @default(CUSTOMER)
  createdAt    DateTime       @default(now())
  updatedAt    DateTime       @updatedAt

  refreshTokens RefreshToken[]
}

model RefreshToken {
  id        String   @id @default(uuid())
  tokenHash String
  userId    String
  expiresAt DateTime
  createdAt DateTime @default(now())

  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}
```

Why each field exists:

- `email`: login identity.
- `passwordHash`: secure password storage.
- `role`: separates customer/admin actions.
- `RefreshToken`: keeps users logged in without long-lived access tokens.

---

### Product Service Database

Tables:

- `Category`
- `Product`
- `InventoryReservation`

`services/product-service/prisma/schema.prisma`:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Category {
  id        String    @id @default(uuid())
  name      String    @unique
  slug      String    @unique
  products  Product[]
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
}

model Product {
  id          String   @id @default(uuid())
  name        String
  slug        String   @unique
  description String?
  priceCents  Int
  imageUrl    String?
  stock       Int      @default(0)
  categoryId  String?
  isActive    Boolean  @default(true)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  category    Category? @relation(fields: [categoryId], references: [id], onDelete: SetNull)
  reservations InventoryReservation[]
}

model InventoryReservation {
  id        String   @id @default(uuid())
  productId String
  orderId   String
  quantity  Int
  expiresAt DateTime
  createdAt DateTime @default(now())

  product   Product  @relation(fields: [productId], references: [id], onDelete: Cascade)

  @@unique([productId, orderId])
}
```

Why inventory reservations exist:

When a customer checks out, we should reserve inventory before payment completes. This prevents overselling when multiple users buy the same product at the same time.

---

### Cart Service Data

Cart data can live in Redis because:

- Cart data changes often.
- Cart data is temporary.
- Redis is fast.

Redis key design:

```text
cart:user:{userId}
```

Value example:

```json
[
  {
    "productId": "product-id",
    "quantity": 2,
    "priceCents": 1999,
    "name": "Keyboard"
  }
]
```

Production note:

For guest carts, use:

```text
cart:guest:{sessionId}
```

---

### Order Service Database

Tables:

- `Order`
- `OrderItem`

`services/order-service/prisma/schema.prisma`:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

enum OrderStatus {
  PENDING_PAYMENT
  PAID
  FAILED
  CANCELLED
  SHIPPED
  DELIVERED
}

model Order {
  id          String      @id @default(uuid())
  userId      String
  status      OrderStatus @default(PENDING_PAYMENT)
  totalCents  Int
  createdAt   DateTime    @default(now())
  updatedAt   DateTime    @updatedAt

  items       OrderItem[]
}

model OrderItem {
  id         String @id @default(uuid())
  orderId    String
  productId  String
  name       String
  priceCents Int
  quantity   Int

  order      Order  @relation(fields: [orderId], references: [id], onDelete: Cascade)
}
```

Why copy product name and price into order item:

Product prices can change later. An order should preserve what the customer actually bought at that time.

---

### Payment Service Database

Tables:

- `Payment`

`services/payment-service/prisma/schema.prisma`:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

enum PaymentStatus {
  PENDING
  SUCCEEDED
  FAILED
  REFUNDED
}

model Payment {
  id                String        @id @default(uuid())
  orderId           String        @unique
  userId            String
  amountCents       Int
  provider          String        @default("mock")
  providerPaymentId String?
  status            PaymentStatus @default(PENDING)
  idempotencyKey    String        @unique
  createdAt         DateTime      @default(now())
  updatedAt         DateTime      @updatedAt
}
```

Why `idempotencyKey` matters:

Payment APIs can be retried. Without idempotency, the same order might be charged twice.

Idempotency means:

```text
Same request key -> same result, no duplicate charge
```

---

## 5. Backend Development

We will use the same basic service structure for each Node service.

### Common Service Pattern

Each service has:

```text
src/
|-- app.ts
|-- server.ts
|-- routes/
|-- controllers/
|-- schemas/
|-- middleware/
|-- lib/
```

### Service Package Setup

Inside each service:

```bash
npm init -y
npm install express cors dotenv zod morgan
npm install -D typescript tsx @types/node @types/express @types/cors @types/morgan
```

For Prisma services:

```bash
npm install prisma @prisma/client
npx prisma init
```

For user service:

```bash
npm install bcrypt jsonwebtoken date-fns
npm install -D @types/bcrypt @types/jsonwebtoken
```

For Redis cart service:

```bash
npm install ioredis axios
```

For RabbitMQ:

```bash
npm install amqplib
npm install -D @types/amqplib
```

---

### Standard Dockerfile For Node Services

Create this in each service folder:

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "run", "dev"]
```

What this does:

- Uses Node 20 Alpine as lightweight base image.
- Sets `/app` as working folder.
- Installs dependencies.
- Copies source code.
- Runs the service.

Production improvement:

Use multi-stage builds and run compiled JavaScript instead of dev mode.

---

### User Service

The User Service owns:

- Registration
- Login
- Refresh tokens
- User profile
- Admin/customer roles

#### User Service Environment

```env
PORT=3001
DATABASE_URL=postgresql://postgres:postgres@user-db:5432/userdb
JWT_ACCESS_SECRET=change_this_access_secret
JWT_REFRESH_SECRET=change_this_refresh_secret
```

#### User Service Prisma Client

Create `services/user-service/src/lib/prisma.ts`:

```ts
import { PrismaClient } from "@prisma/client";

export const prisma = new PrismaClient();
```

What it does:

This creates a database client for the User Service database.

#### Password Helpers

Create `services/user-service/src/lib/password.ts`:

```ts
import bcrypt from "bcrypt";

export const hashPassword = (password: string) => {
  return bcrypt.hash(password, 12);
};

export const verifyPassword = (password: string, hash: string) => {
  return bcrypt.compare(password, hash);
};
```

Explanation:

- `hashPassword` securely stores passwords.
- `verifyPassword` checks login attempts.
- `12` is the salt rounds count. Higher is more secure but slower.

#### Token Helpers

Create `services/user-service/src/lib/tokens.ts`:

```ts
import crypto from "crypto";
import jwt from "jsonwebtoken";

const ACCESS_SECRET = process.env.JWT_ACCESS_SECRET!;
const REFRESH_SECRET = process.env.JWT_REFRESH_SECRET!;

type TokenPayload = {
  userId: string;
  role: "CUSTOMER" | "ADMIN";
};

export const signAccessToken = (payload: TokenPayload) => {
  return jwt.sign(payload, ACCESS_SECRET, { expiresIn: "15m" });
};

export const signRefreshToken = (payload: TokenPayload) => {
  return jwt.sign(payload, REFRESH_SECRET, { expiresIn: "7d" });
};

export const verifyAccessToken = (token: string) => {
  return jwt.verify(token, ACCESS_SECRET) as TokenPayload;
};

export const verifyRefreshToken = (token: string) => {
  return jwt.verify(token, REFRESH_SECRET) as TokenPayload;
};

export const hashToken = (token: string) => {
  return crypto.createHash("sha256").update(token).digest("hex");
};
```

What this code does:

- Creates access tokens for API calls.
- Creates refresh tokens for new access tokens.
- Verifies tokens.
- Hashes refresh tokens before storing them.

#### User Validation Schemas

Create `services/user-service/src/schemas/auth.schema.ts`:

```ts
import { z } from "zod";

export const registerSchema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
  password: z.string().min(8),
});

export const loginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(1),
});
```

Why use Zod:

Zod checks that incoming request data is valid before business logic runs.

#### User Controller

Create `services/user-service/src/controllers/auth.controller.ts`:

```ts
import type { Request, Response } from "express";
import { addDays } from "date-fns";
import { prisma } from "../lib/prisma";
import { hashPassword, verifyPassword } from "../lib/password";
import {
  hashToken,
  signAccessToken,
  signRefreshToken,
  verifyRefreshToken,
} from "../lib/tokens";

const createTokenPair = async (user: {
  id: string;
  role: "CUSTOMER" | "ADMIN";
}) => {
  const payload = { userId: user.id, role: user.role };
  const accessToken = signAccessToken(payload);
  const refreshToken = signRefreshToken(payload);

  await prisma.refreshToken.create({
    data: {
      userId: user.id,
      tokenHash: hashToken(refreshToken),
      expiresAt: addDays(new Date(), 7),
    },
  });

  return { accessToken, refreshToken };
};

export const register = async (req: Request, res: Response) => {
  const parsed = req.body;

  const existingUser = await prisma.user.findUnique({
    where: { email: parsed.email },
  });

  if (existingUser) {
    return res.status(400).json({ message: "Email already registered" });
  }

  const passwordHash = await hashPassword(parsed.password);

  const user = await prisma.user.create({
    data: {
      name: parsed.name,
      email: parsed.email,
      passwordHash,
    },
  });

  const tokens = await createTokenPair(user);

  res.status(201).json({
    user: {
      id: user.id,
      name: user.name,
      email: user.email,
      role: user.role,
    },
    ...tokens,
  });
};

export const login = async (req: Request, res: Response) => {
  const { email, password } = req.body;

  const user = await prisma.user.findUnique({ where: { email } });

  if (!user) {
    return res.status(401).json({ message: "Invalid email or password" });
  }

  const valid = await verifyPassword(password, user.passwordHash);

  if (!valid) {
    return res.status(401).json({ message: "Invalid email or password" });
  }

  const tokens = await createTokenPair(user);

  res.json({
    user: {
      id: user.id,
      name: user.name,
      email: user.email,
      role: user.role,
    },
    ...tokens,
  });
};

export const refresh = async (req: Request, res: Response) => {
  const { refreshToken } = req.body;
  const payload = verifyRefreshToken(refreshToken);

  const stored = await prisma.refreshToken.findFirst({
    where: {
      userId: payload.userId,
      tokenHash: hashToken(refreshToken),
      expiresAt: { gt: new Date() },
    },
  });

  if (!stored) {
    return res.status(401).json({ message: "Invalid refresh token" });
  }

  const accessToken = signAccessToken(payload);

  res.json({ accessToken });
};
```

Important explanation:

- `register` creates user and returns tokens.
- `login` verifies password and returns tokens.
- `refresh` checks refresh token and returns a new access token.
- The database stores only a hash of the refresh token.

#### User Routes

Create `services/user-service/src/routes/auth.routes.ts`:

```ts
import { Router } from "express";
import { login, refresh, register } from "../controllers/auth.controller";
import { loginSchema, registerSchema } from "../schemas/auth.schema";

export const authRoutes = Router();

const validate =
  (schema: any) =>
  (req: any, res: any, next: any) => {
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

authRoutes.post("/register", validate(registerSchema), register);
authRoutes.post("/login", validate(loginSchema), login);
authRoutes.post("/refresh", refresh);
```

#### User Service App

Create `services/user-service/src/app.ts`:

```ts
import express from "express";
import cors from "cors";
import morgan from "morgan";
import { authRoutes } from "./routes/auth.routes";

export const app = express();

app.use(cors());
app.use(express.json());
app.use(morgan("dev"));

app.get("/health", (req, res) => {
  res.json({ service: "user-service", status: "ok" });
});

app.use("/", authRoutes);
```

Create `services/user-service/src/server.ts`:

```ts
import { app } from "./app";

const PORT = process.env.PORT || 3001;

app.listen(PORT, () => {
  console.log(`User service running on port ${PORT}`);
});
```

---

### Product Catalog Service

The Product Service owns:

- Products
- Categories
- Inventory
- Inventory reservation

#### Product Routes

API design:

| Method | Route | Purpose |
|---|---|---|
| `GET` | `/` | List products |
| `GET` | `/:id` | Get one product |
| `POST` | `/` | Create product, admin only |
| `PATCH` | `/:id` | Update product, admin only |
| `POST` | `/reserve` | Reserve inventory for order |
| `POST` | `/release` | Release reserved inventory |

#### Product Validation

Create `services/product-service/src/schemas/product.schema.ts`:

```ts
import { z } from "zod";

export const createProductSchema = z.object({
  name: z.string().min(2),
  description: z.string().optional(),
  priceCents: z.number().int().positive(),
  stock: z.number().int().min(0),
  categoryId: z.string().uuid().optional(),
  imageUrl: z.string().url().optional(),
});

export const reserveInventorySchema = z.object({
  orderId: z.string().uuid(),
  items: z.array(
    z.object({
      productId: z.string().uuid(),
      quantity: z.number().int().positive(),
    })
  ),
});
```

#### Product Controller

Create `services/product-service/src/controllers/product.controller.ts`:

```ts
import type { Request, Response } from "express";
import { prisma } from "../lib/prisma";

export const listProducts = async (req: Request, res: Response) => {
  const products = await prisma.product.findMany({
    where: { isActive: true },
    include: { category: true },
    orderBy: { createdAt: "desc" },
  });

  res.json({ products });
};

export const getProduct = async (req: Request, res: Response) => {
  const product = await prisma.product.findUnique({
    where: { id: req.params.id },
    include: { category: true },
  });

  if (!product || !product.isActive) {
    return res.status(404).json({ message: "Product not found" });
  }

  res.json({ product });
};

export const createProduct = async (req: Request, res: Response) => {
  const slug = req.body.name
    .toLowerCase()
    .replace(/[^a-z0-9]+/g, "-")
    .replace(/(^-|-$)/g, "");

  const product = await prisma.product.create({
    data: {
      ...req.body,
      slug: `${slug}-${Date.now()}`,
    },
  });

  res.status(201).json({ product });
};

export const reserveInventory = async (req: Request, res: Response) => {
  const { orderId, items } = req.body;

  try {
    await prisma.$transaction(async (tx) => {
      for (const item of items) {
        const product = await tx.product.findUnique({
          where: { id: item.productId },
        });

        if (!product || product.stock < item.quantity) {
          throw new Error(`Insufficient stock for product ${item.productId}`);
        }

        await tx.product.update({
          where: { id: item.productId },
          data: { stock: { decrement: item.quantity } },
        });

        await tx.inventoryReservation.upsert({
          where: {
            productId_orderId: {
              productId: item.productId,
              orderId,
            },
          },
          update: { quantity: item.quantity },
          create: {
            productId: item.productId,
            orderId,
            quantity: item.quantity,
            expiresAt: new Date(Date.now() + 15 * 60 * 1000),
          },
        });
      }
    });

    res.json({ message: "Inventory reserved" });
  } catch (error: any) {
    res.status(400).json({ message: error.message });
  }
};
```

What `prisma.$transaction` does:

It makes multiple database operations succeed or fail together.

Why this matters:

If stock is reduced but reservation creation fails, inventory becomes incorrect. A transaction prevents that.

---

### Cart Service

The Cart Service owns:

- Add item to cart
- Remove item
- Update quantity
- Get cart
- Clear cart

It uses Redis because carts are temporary and need fast reads/writes.

#### Cart Item Type

```ts
type CartItem = {
  productId: string;
  name: string;
  priceCents: number;
  quantity: number;
  imageUrl?: string;
};
```

#### Redis Client

Create `services/cart-service/src/lib/redis.ts`:

```ts
import Redis from "ioredis";

export const redis = new Redis(process.env.REDIS_URL!);
```

#### Cart Controller

Create `services/cart-service/src/controllers/cart.controller.ts`:

```ts
import type { Request, Response } from "express";
import axios from "axios";
import { redis } from "../lib/redis";

type CartItem = {
  productId: string;
  name: string;
  priceCents: number;
  quantity: number;
  imageUrl?: string;
};

const cartKey = (userId: string) => `cart:user:${userId}`;

const getCartItems = async (userId: string): Promise<CartItem[]> => {
  const raw = await redis.get(cartKey(userId));
  return raw ? JSON.parse(raw) : [];
};

const saveCartItems = async (userId: string, items: CartItem[]) => {
  await redis.set(cartKey(userId), JSON.stringify(items), "EX", 60 * 60 * 24 * 30);
};

export const getCart = async (req: Request, res: Response) => {
  const userId = req.headers["x-user-id"] as string;
  const items = await getCartItems(userId);
  const totalCents = items.reduce((sum, item) => sum + item.priceCents * item.quantity, 0);

  res.json({ items, totalCents });
};

export const addToCart = async (req: Request, res: Response) => {
  const userId = req.headers["x-user-id"] as string;
  const { productId, quantity } = req.body;

  const productResponse = await axios.get(
    `${process.env.PRODUCT_SERVICE_URL}/${productId}`
  );

  const product = productResponse.data.product;
  const items = await getCartItems(userId);
  const existing = items.find((item) => item.productId === productId);

  if (existing) {
    existing.quantity += quantity;
  } else {
    items.push({
      productId,
      quantity,
      name: product.name,
      priceCents: product.priceCents,
      imageUrl: product.imageUrl,
    });
  }

  await saveCartItems(userId, items);
  res.status(201).json({ items });
};

export const clearCart = async (req: Request, res: Response) => {
  const userId = req.headers["x-user-id"] as string;
  await redis.del(cartKey(userId));
  res.json({ message: "Cart cleared" });
};
```

Important explanation:

- Cart Service asks Product Service for product details.
- Cart stores a snapshot of name and price.
- Redis key expires after 30 days.
- `x-user-id` is passed by gateway or frontend after auth.

Production improvement:

The API Gateway should validate JWT and pass trusted user info to services. In this beginner version, we keep it simpler.

---

### Order Service

The Order Service owns:

- Creating orders
- Order history
- Order status updates

Checkout flow:

```text
Frontend calls Order Service
Order Service gets cart
Order Service reserves inventory
Order Service creates order
Order Service asks Payment Service to create payment
Payment succeeds or fails
Order Service updates order status
Order Service publishes order event
Notification Service sends message
```

#### Order Controller

Create `services/order-service/src/controllers/order.controller.ts`:

```ts
import type { Request, Response } from "express";
import axios from "axios";
import { prisma } from "../lib/prisma";
import { publishEvent } from "../lib/rabbitmq";

export const createOrder = async (req: Request, res: Response) => {
  const userId = req.headers["x-user-id"] as string;

  const cartResponse = await axios.get(`${process.env.CART_SERVICE_URL}`, {
    headers: { "x-user-id": userId },
  });

  const cart = cartResponse.data;

  if (!cart.items.length) {
    return res.status(400).json({ message: "Cart is empty" });
  }

  const order = await prisma.order.create({
    data: {
      userId,
      totalCents: cart.totalCents,
      items: {
        create: cart.items.map((item: any) => ({
          productId: item.productId,
          name: item.name,
          priceCents: item.priceCents,
          quantity: item.quantity,
        })),
      },
    },
    include: { items: true },
  });

  await axios.post(`${process.env.PRODUCT_SERVICE_URL}/reserve`, {
    orderId: order.id,
    items: order.items.map((item) => ({
      productId: item.productId,
      quantity: item.quantity,
    })),
  });

  const paymentResponse = await axios.post(`${process.env.PAYMENT_SERVICE_URL}/payments`, {
    orderId: order.id,
    userId,
    amountCents: order.totalCents,
    idempotencyKey: `order:${order.id}`,
  });

  await publishEvent("order.created", {
    orderId: order.id,
    userId,
    totalCents: order.totalCents,
  });

  res.status(201).json({
    order,
    payment: paymentResponse.data.payment,
  });
};

export const getMyOrders = async (req: Request, res: Response) => {
  const userId = req.headers["x-user-id"] as string;

  const orders = await prisma.order.findMany({
    where: { userId },
    include: { items: true },
    orderBy: { createdAt: "desc" },
  });

  res.json({ orders });
};
```

Important explanation:

- Order Service coordinates multiple services.
- It does not directly read Product Service database.
- It calls Product Service API.
- It publishes `order.created` event for Notification Service.

Production warning:

This simple flow can fail halfway. A production system should use Saga pattern, retries, compensation, and idempotency.

---

### RabbitMQ Event Helper

Create `services/order-service/src/lib/rabbitmq.ts`:

```ts
import amqp from "amqplib";

let channel: amqp.Channel | null = null;

export const getChannel = async () => {
  if (channel) return channel;

  const connection = await amqp.connect(process.env.RABBITMQ_URL!);
  channel = await connection.createChannel();
  await channel.assertExchange("ecommerce.events", "topic", { durable: true });

  return channel;
};

export const publishEvent = async (routingKey: string, payload: unknown) => {
  const ch = await getChannel();
  ch.publish(
    "ecommerce.events",
    routingKey,
    Buffer.from(JSON.stringify(payload)),
    { persistent: true }
  );
};
```

What this does:

- Connects to RabbitMQ.
- Creates a topic exchange.
- Publishes events such as `order.created`.

Why events:

Order Service should not directly know how email sending works. It just announces that something happened.

---

### Payment Service

Payment Service owns:

- Creating payments
- Tracking payment status
- Integrating payment provider

Beginner version: mock payment success.

#### Payment Controller

Create `services/payment-service/src/controllers/payment.controller.ts`:

```ts
import type { Request, Response } from "express";
import { prisma } from "../lib/prisma";
import { publishEvent } from "../lib/rabbitmq";

export const createPayment = async (req: Request, res: Response) => {
  const { orderId, userId, amountCents, idempotencyKey } = req.body;

  const existing = await prisma.payment.findUnique({
    where: { idempotencyKey },
  });

  if (existing) {
    return res.json({ payment: existing });
  }

  const payment = await prisma.payment.create({
    data: {
      orderId,
      userId,
      amountCents,
      idempotencyKey,
      status: "SUCCEEDED",
      provider: "mock",
      providerPaymentId: `mock_${Date.now()}`,
    },
  });

  await publishEvent("payment.succeeded", {
    paymentId: payment.id,
    orderId,
    userId,
    amountCents,
  });

  res.status(201).json({ payment });
};
```

What this code does:

- Checks idempotency key first.
- Creates payment only once.
- Publishes payment success event.

Production Stripe version:

- Create Stripe PaymentIntent.
- Store Stripe payment ID.
- Listen to Stripe webhook.
- Update payment status from webhook.

---

### Notification Service

Notification Service listens to events:

- `order.created`
- `payment.succeeded`
- `order.shipped`

Beginner version logs to console.

Create `services/notification-service/src/server.ts`:

```ts
import amqp from "amqplib";

const start = async () => {
  const connection = await amqp.connect(process.env.RABBITMQ_URL!);
  const channel = await connection.createChannel();

  await channel.assertExchange("ecommerce.events", "topic", { durable: true });

  const queue = await channel.assertQueue("notification-service", {
    durable: true,
  });

  await channel.bindQueue(queue.queue, "ecommerce.events", "order.*");
  await channel.bindQueue(queue.queue, "ecommerce.events", "payment.*");

  channel.consume(queue.queue, async (message) => {
    if (!message) return;

    const routingKey = message.fields.routingKey;
    const payload = JSON.parse(message.content.toString());

    console.log("Notification event received:", routingKey, payload);

    channel.ack(message);
  });

  console.log("Notification service listening for events");
};

start();
```

What this does:

- Connects to RabbitMQ.
- Subscribes to order and payment events.
- Processes messages one by one.
- Acknowledges messages after processing.

Production improvement:

Replace console log with SendGrid email or Twilio SMS.

---

## 6. Frontend Development

### Initialize React App

```bash
cd apps
npm create vite@latest web -- --template react-ts
cd web
npm install
```

Install dependencies:

```bash
npm install axios react-router-dom @tanstack/react-query zustand zod react-hook-form @hookform/resolvers lucide-react
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

### Frontend API Client

Create `apps/web/src/api/client.ts`:

```ts
import axios from "axios";
import { useAuthStore } from "../stores/auth.store";

export const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL || "http://localhost:8080",
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

- Sets API gateway URL.
- Adds JWT token to requests.

### Auth Store

Create `apps/web/src/stores/auth.store.ts`:

```ts
import { create } from "zustand";
import { persist } from "zustand/middleware";

type User = {
  id: string;
  name: string;
  email: string;
  role: "CUSTOMER" | "ADMIN";
};

type AuthState = {
  user: User | null;
  accessToken: string | null;
  refreshToken: string | null;
  setAuth: (data: {
    user: User;
    accessToken: string;
    refreshToken: string;
  }) => void;
  logout: () => void;
};

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      accessToken: null,
      refreshToken: null,
      setAuth: (data) => set(data),
      logout: () => set({ user: null, accessToken: null, refreshToken: null }),
    }),
    { name: "scaleshop-auth" }
  )
);
```

What this does:

- Stores logged-in user.
- Stores tokens.
- Persists auth data in localStorage.

Production note:

For better security, store refresh tokens in HTTP-only cookies.

### Auth API

Create `apps/web/src/api/auth.api.ts`:

```ts
import { api } from "./client";

export const registerRequest = async (data: {
  name: string;
  email: string;
  password: string;
}) => {
  const res = await api.post("/api/users/register", data);
  return res.data;
};

export const loginRequest = async (data: {
  email: string;
  password: string;
}) => {
  const res = await api.post("/api/users/login", data);
  return res.data;
};
```

### Product API

Create `apps/web/src/api/products.api.ts`:

```ts
import { api } from "./client";

export type Product = {
  id: string;
  name: string;
  description?: string;
  priceCents: number;
  imageUrl?: string;
  stock: number;
};

export const getProducts = async () => {
  const res = await api.get("/api/products");
  return res.data.products as Product[];
};
```

### Cart API

Create `apps/web/src/api/cart.api.ts`:

```ts
import { api } from "./client";

export const getCart = async () => {
  const res = await api.get("/api/cart");
  return res.data;
};

export const addToCart = async (productId: string, quantity: number) => {
  const res = await api.post("/api/cart/items", { productId, quantity });
  return res.data;
};
```

### Product Listing Page

Create `apps/web/src/pages/ProductListPage.tsx`:

```tsx
import { useMutation, useQuery, useQueryClient } from "@tanstack/react-query";
import { getProducts } from "../api/products.api";
import { addToCart } from "../api/cart.api";

const formatMoney = (cents: number) => {
  return `$${(cents / 100).toFixed(2)}`;
};

export const ProductListPage = () => {
  const queryClient = useQueryClient();

  const productsQuery = useQuery({
    queryKey: ["products"],
    queryFn: getProducts,
  });

  const addMutation = useMutation({
    mutationFn: (productId: string) => addToCart(productId, 1),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["cart"] });
    },
  });

  if (productsQuery.isLoading) {
    return <div className="p-6">Loading products...</div>;
  }

  return (
    <main className="p-6">
      <h1 className="text-2xl font-bold">Products</h1>

      <div className="mt-6 grid gap-4 md:grid-cols-3">
        {productsQuery.data?.map((product) => (
          <div key={product.id} className="rounded-lg border bg-white p-4">
            <div className="h-40 rounded-md bg-slate-100" />
            <h2 className="mt-4 font-semibold">{product.name}</h2>
            <p className="text-sm text-slate-600">{product.description}</p>
            <p className="mt-2 font-bold">{formatMoney(product.priceCents)}</p>
            <button
              onClick={() => addMutation.mutate(product.id)}
              className="mt-4 rounded-md bg-blue-600 px-4 py-2 text-white"
            >
              Add to cart
            </button>
          </div>
        ))}
      </div>
    </main>
  );
};
```

What this component does:

- Fetches products using TanStack Query.
- Displays product cards.
- Adds product to cart using mutation.
- Refreshes cart cache after success.

### Cart Page

Create `apps/web/src/pages/CartPage.tsx`:

```tsx
import { useMutation, useQuery } from "@tanstack/react-query";
import { getCart } from "../api/cart.api";
import { api } from "../api/client";

const formatMoney = (cents: number) => `$${(cents / 100).toFixed(2)}`;

export const CartPage = () => {
  const cartQuery = useQuery({
    queryKey: ["cart"],
    queryFn: getCart,
  });

  const checkoutMutation = useMutation({
    mutationFn: async () => {
      const res = await api.post("/api/orders");
      return res.data;
    },
  });

  if (cartQuery.isLoading) {
    return <div className="p-6">Loading cart...</div>;
  }

  const cart = cartQuery.data;

  return (
    <main className="p-6 max-w-3xl mx-auto">
      <h1 className="text-2xl font-bold">Shopping Cart</h1>

      <div className="mt-6 rounded-lg border bg-white">
        {cart.items.map((item: any) => (
          <div key={item.productId} className="flex justify-between border-b p-4">
            <div>
              <h2 className="font-medium">{item.name}</h2>
              <p className="text-sm text-slate-600">Qty: {item.quantity}</p>
            </div>
            <p>{formatMoney(item.priceCents * item.quantity)}</p>
          </div>
        ))}

        <div className="flex justify-between p-4 font-bold">
          <span>Total</span>
          <span>{formatMoney(cart.totalCents)}</span>
        </div>
      </div>

      <button
        onClick={() => checkoutMutation.mutate()}
        className="mt-6 rounded-md bg-green-600 px-4 py-2 text-white"
      >
        {checkoutMutation.isPending ? "Placing order..." : "Checkout"}
      </button>

      {checkoutMutation.isSuccess && (
        <p className="mt-4 text-green-700">Order placed successfully.</p>
      )}
    </main>
  );
};
```

---

## 7. Core Features

### Feature 1: User Registration And Login

What it does:

Creates accounts and authenticates users.

How it works:

```text
Frontend form
User Service validates input
Password is hashed
User row is created
JWT access and refresh tokens are returned
Frontend stores tokens
```

Backend code:

```ts
authRoutes.post("/register", validate(registerSchema), register);
authRoutes.post("/login", validate(loginSchema), login);
```

Frontend code:

```ts
const res = await api.post("/api/users/login", data);
setAuth(res.data);
```

Data flow:

```text
React -> API Gateway -> User Service -> user-db
```

---

### Feature 2: Product Catalog

What it does:

Stores and displays products.

How it works:

```text
Admin creates product
Product Service stores product
Customer fetches product list
Frontend renders products
```

Backend:

```ts
const products = await prisma.product.findMany({
  where: { isActive: true },
});
```

Frontend:

```ts
useQuery({
  queryKey: ["products"],
  queryFn: getProducts,
});
```

---

### Feature 3: Shopping Cart

What it does:

Stores temporary cart items.

How it works:

```text
Customer clicks Add to cart
Cart Service fetches product details
Cart Service stores cart in Redis
Frontend displays cart
```

Why Redis:

Cart data changes frequently and is temporary. Redis is fast and fits this use case.

---

### Feature 4: Checkout And Orders

What it does:

Turns a cart into an order.

How it works:

```text
Frontend calls Order Service
Order Service reads cart
Order Service creates order
Order Service reserves inventory
Order Service creates payment
Order Service publishes event
Notification Service sends confirmation
```

Important concept:

Checkout is a distributed workflow. Multiple services are involved, so failures must be handled carefully.

---

### Feature 5: Payments

What it does:

Processes payment for an order.

Beginner version:

- Mock success payment.

Production version:

- Stripe PaymentIntent
- Webhook verification
- Idempotency
- Refund handling

Why idempotency:

If the network retries the same payment request, the user should not be charged twice.

---

### Feature 6: Notifications

What it does:

Sends email/SMS after important events.

How it works:

```text
Order Service publishes order.created
RabbitMQ stores event
Notification Service consumes event
Notification Service sends email
```

Why async events:

Order creation should not fail just because email sending is slow.

---

## 8. Authentication And Authorization

### Signup

```text
User enters name/email/password
User Service validates data
Password is hashed
User is saved
Access token and refresh token are returned
```

### Login

```text
User enters email/password
User Service finds user
bcrypt compares password
Tokens are returned
```

### Token Handling

Access token:

- Short-lived
- Sent with API requests
- Contains user ID and role

Refresh token:

- Longer-lived
- Used to get new access token
- Stored hashed in database

### Protected Routes

In a production setup, the API Gateway should verify JWT and pass user headers:

```text
x-user-id
x-user-role
```

Then services can trust those headers because only the gateway can call them internally.

Beginner simplification:

Frontend sends token and services verify it directly, or gateway injects headers after token validation.

### Admin Authorization

Admin-only product creation:

```text
If role is not ADMIN -> return 403 Forbidden
```

Example middleware:

```ts
export const requireAdmin = (req: any, res: any, next: any) => {
  if (req.user?.role !== "ADMIN") {
    return res.status(403).json({ message: "Admin only" });
  }

  next();
};
```

### Logout

Logout should:

- Delete refresh token from database.
- Clear frontend auth store.
- Redirect user to login page.

---

## 9. Validation And Error Handling

### Input Validation

Use Zod in every service:

```ts
const createProductSchema = z.object({
  name: z.string().min(2),
  priceCents: z.number().int().positive(),
});
```

Why:

Frontend validation is not enough. Attackers can call backend APIs directly.

### API Error Format

Use consistent errors:

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

### Common Edge Cases

- Product out of stock
- Cart is empty
- Payment request retried
- Payment succeeds but order update fails
- Notification fails
- RabbitMQ unavailable
- Redis unavailable
- Product deleted while in cart
- User token expired

### Error Handling Strategy

For simple request errors:

- Return `400` for validation
- Return `401` for unauthenticated
- Return `403` for forbidden
- Return `404` for missing resource
- Return `500` for unexpected errors

For distributed workflow errors:

- Use retries
- Use idempotency keys
- Use compensation actions
- Use event logs
- Use dead-letter queues

---

## 10. Testing And Debugging

### Test Gateway

```text
GET http://localhost:8080/health
```

Expected:

```text
gateway ok
```

### Test User Service Through Gateway

Register:

```text
POST http://localhost:8080/api/users/register
```

Body:

```json
{
  "name": "Alex",
  "email": "alex@example.com",
  "password": "password123"
}
```

Login:

```text
POST http://localhost:8080/api/users/login
```

Body:

```json
{
  "email": "alex@example.com",
  "password": "password123"
}
```

### Test Product Service

```text
GET http://localhost:8080/api/products
```

### Test Cart Service

```text
POST http://localhost:8080/api/cart/items
```

Headers:

```text
x-user-id: USER_ID
```

Body:

```json
{
  "productId": "PRODUCT_ID",
  "quantity": 1
}
```

### Test Order Service

```text
POST http://localhost:8080/api/orders
```

Headers:

```text
x-user-id: USER_ID
```

Expected:

- Order is created.
- Payment is created.
- Notification event is logged.

### Debugging Docker

View running containers:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs user-service
docker compose logs product-service
docker compose logs rabbitmq
```

Restart one service:

```bash
docker compose restart product-service
```

Rebuild:

```bash
docker compose up --build
```

### Common Bugs And Fixes

#### Service Cannot Connect To Database

Check:

- Database container is running.
- `DATABASE_URL` uses Docker service name, not localhost.

Inside Docker:

```text
postgresql://postgres:postgres@product-db:5432/productdb
```

Not:

```text
localhost:5432
```

#### Gateway Returns 502

Cause:

- Target service is down.
- Wrong service name or port in NGINX config.

Fix:

```bash
docker compose logs gateway
docker compose logs product-service
```

#### RabbitMQ Connection Fails

Cause:

- RabbitMQ starts slower than services.

Fix:

- Add retry logic in RabbitMQ client.
- Restart service after RabbitMQ is ready.

#### Cart Is Empty After Restart

Cause:

- Redis data is memory-based unless persisted.

Fix:

- Add Redis persistence or accept cart loss in local dev.
- For production, configure Redis persistence or store carts in database.

---

## 11. Running The Project

### Start Everything

From project root:

```bash
docker compose up --build
```

### Run Prisma Migrations

For each Prisma service:

```bash
docker compose exec user-service npx prisma migrate dev --name init
docker compose exec product-service npx prisma migrate dev --name init
docker compose exec order-service npx prisma migrate dev --name init
docker compose exec payment-service npx prisma migrate dev --name init
```

### Verify Services

Gateway:

```text
http://localhost:8080/health
```

RabbitMQ dashboard:

```text
http://localhost:15672
```

Default RabbitMQ credentials:

```text
guest / guest
```

Frontend:

```text
http://localhost:5173
```

### Stop Everything

```bash
docker compose down
```

Stop and delete volumes:

```bash
docker compose down -v
```

Warning:

`down -v` deletes database data.

---

## 12. Next Improvements

### Feature Upgrades

1. Product search with Elasticsearch or PostgreSQL full-text search.
2. Product image upload to S3 or Cloudinary.
3. Admin dashboard.
4. Coupon/discount service.
5. Review/rating service.
6. Shipping service.
7. Wishlist service.
8. Refund flow.
9. Stripe payment integration.
10. Order tracking.
11. Email templates.
12. SMS notifications.

### Performance Improvements

1. Add Redis caching for product lists.
2. Add pagination to product APIs.
3. Add database indexes.
4. Add read replicas for product database.
5. Use CDN for images.
6. Add rate limiting at API gateway.
7. Add circuit breakers for service calls.
8. Add retries with exponential backoff.
9. Add message dead-letter queues.

### Reliability Improvements

1. Use Saga pattern for checkout.
2. Store outbox events in database before publishing.
3. Add idempotency to order creation.
4. Add payment webhooks.
5. Add inventory reservation expiration job.
6. Add health checks to every service.
7. Add structured logging with request IDs.

### Monitoring And Logging

Recommended local stack:

- Prometheus for metrics
- Grafana for dashboards
- Loki or ELK for logs

Important metrics:

- Request count
- Error rate
- Response time
- Database query latency
- RabbitMQ queue depth
- Redis memory usage
- Payment failure rate

### CI/CD Pipeline

GitHub Actions flow:

```text
Push code
Install dependencies
Run lint
Run tests
Build Docker image
Push image to registry
Deploy to server or Kubernetes
```

Example `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm install
      - run: npm test
```

For a real monorepo, use workspaces or run commands per service.

### Production Deployment Suggestions

Small production setup:

- Frontend on Vercel
- Backend services on Render/Railway/Fly.io
- PostgreSQL on Neon/Supabase/Railway
- Redis on Upstash/Redis Cloud
- RabbitMQ on CloudAMQP
- Images on S3/Cloudinary

Larger production setup:

- Kubernetes cluster
- NGINX Ingress or Traefik
- Managed PostgreSQL
- Managed Redis
- Managed RabbitMQ or Kafka
- Prometheus/Grafana
- Centralized logging
- Horizontal pod autoscaling

### Service Discovery

In Docker Compose:

```text
Service names act as DNS names.
```

In Kubernetes:

```text
Kubernetes Services provide service discovery.
```

Tools like Consul or Eureka can also provide service discovery, but beginners can start with Docker/Kubernetes built-in DNS.

---

## Production Architecture Summary

```text
Client
  |
CDN / Frontend Hosting
  |
API Gateway / Load Balancer
  |
  |-- User Service -------- user-db
  |-- Product Service ----- product-db
  |-- Cart Service -------- Redis
  |-- Order Service ------- order-db
  |-- Payment Service ----- payment-db
  |-- Notification Service
  |
RabbitMQ Event Bus
  |
Email/SMS Providers

Observability:
  Logs -> ELK/Loki
  Metrics -> Prometheus/Grafana
  Errors -> Sentry
```

---

## Final Summary

You built the blueprint for a scalable e-commerce platform using microservices and Docker.

You learned:

- How to split an e-commerce app into services.
- How Docker Compose runs multi-container systems.
- How NGINX works as an API gateway.
- Why each microservice owns its database.
- How Redis fits shopping carts.
- How PostgreSQL stores core business data.
- How RabbitMQ enables async communication.
- How JWT auth works.
- How order and payment workflows coordinate across services.
- How notifications listen to events.
- How to test, debug, and run the full platform.
- How to grow the MVP into a production-ready distributed system.

The key idea is this:

Build the simple working system first. Then add reliability, observability, scaling, and automation step by step.

