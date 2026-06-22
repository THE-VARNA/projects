# NewTube YouTube Clone - Complete Full-Stack Guide

Source repository: https://github.com/code-with-antonio/next15-youtube-clone

Reference inspected: `master` commit `7a447fe120c4e223cf4d15306c73f8e10a03464f`

Important note: this project is not built with a separate Express backend. It is a full-stack Next.js 15 application. The frontend, backend routes, API logic, authentication, database logic, and webhooks all live inside one Next.js app.

---

## What We Will Build

We will build a modern YouTube-style video platform called `new-tube`.

The final app includes:

- Public video feed
- Category filtering
- Video search
- Trending feed
- Subscribed feed
- Video watch page
- Mux video player
- Video upload from Creator Studio
- Video processing through Mux
- Auto thumbnail and preview generation
- Clerk authentication
- User channel pages
- Channel banner upload
- Subscribe/unsubscribe system
- Like/dislike system for videos
- Comments and replies
- Like/dislike system for comments
- Watch history
- Liked videos page
- Custom playlists
- Add/remove videos from playlists
- AI title generation
- AI description generation
- AI thumbnail generation
- Rate limiting with Upstash Redis
- Type-safe APIs with tRPC
- PostgreSQL database with Drizzle ORM

Think of this as a beginner-friendly clone of YouTube with creator tools and a viewer experience.

---

## Tech Stack Used

### Frontend

- `Next.js 15`: main React framework. It handles pages, layouts, server components, API routes, and routing.
- `React 19`: used to build interactive UI components.
- `TypeScript`: adds type safety so mistakes are caught earlier.
- `Tailwind CSS`: utility-first CSS framework for styling.
- `shadcn/ui` and `Radix UI`: reusable accessible UI components such as dialogs, dropdowns, buttons, inputs, and sidebars.
- `lucide-react`: icon library.
- `Mux Player React`: video player UI for Mux-hosted videos.

### Backend

- `Next.js Route Handlers`: backend endpoints inside `src/app/api`.
- `tRPC`: type-safe API layer. The client can call backend procedures without manually writing REST fetch code.
- `TanStack React Query`: caching and async state management for tRPC calls.
- `Zod`: validates inputs for API procedures.
- `Clerk`: authentication provider for sign in, sign up, and user sessions.
- `Svix`: verifies Clerk webhook signatures.

### Database

- `PostgreSQL`: relational database.
- `Neon`: serverless PostgreSQL provider used by the repo through `@neondatabase/serverless`.
- `Drizzle ORM`: maps TypeScript schema code to database tables.
- `Drizzle Kit`: database schema push/migration tooling.

### Video, Uploads, AI, and Background Jobs

- `Mux`: video uploads, processing, playback IDs, thumbnails, previews, and generated subtitles.
- `UploadThing`: image/file uploads for user banners and video thumbnails.
- `Upstash Redis`: used for rate limiting.
- `Upstash Workflow`: used for long-running AI workflows.
- `OpenAI`: used for generated video titles, descriptions, and thumbnails.
- `ngrok`: exposes local dev server to the internet so Clerk, Mux, and Upstash can call local webhooks.

---

## Final Project Structure

```text
new-tube/
|-- public/
|-- src/
|   |-- app/
|   |   |-- (auth)/
|   |   |   |-- sign-in/[[...sign-in]]/page.tsx
|   |   |   |-- sign-up/[[...sign-up]]/page.tsx
|   |   |-- (home)/
|   |   |   |-- page.tsx
|   |   |   |-- layout.tsx
|   |   |   |-- videos/[videoId]/page.tsx
|   |   |   |-- search/page.tsx
|   |   |   |-- subscriptions/page.tsx
|   |   |   |-- feed/trending/page.tsx
|   |   |   |-- feed/subscribed/page.tsx
|   |   |   |-- playlists/page.tsx
|   |   |   |-- playlists/[playlistId]/page.tsx
|   |   |   |-- playlists/history/page.tsx
|   |   |   |-- playlists/liked/page.tsx
|   |   |   |-- users/[userId]/page.tsx
|   |   |   |-- users/current/route.ts
|   |   |-- (studio)/
|   |   |   |-- studio/page.tsx
|   |   |   |-- studio/videos/[videoId]/page.tsx
|   |   |-- api/
|   |   |   |-- trpc/[trpc]/route.ts
|   |   |   |-- uploadthing/core.ts
|   |   |   |-- uploadthing/route.ts
|   |   |   |-- users/webhook/route.ts
|   |   |   |-- videos/webhook/route.ts
|   |   |   |-- videos/workflows/title/route.ts
|   |   |   |-- videos/workflows/description/route.ts
|   |   |   |-- videos/workflows/thumbnail/route.ts
|   |   |-- layout.tsx
|   |   |-- globals.css
|   |-- components/
|   |   |-- ui/
|   |   |-- user-avatar.tsx
|   |   |-- infinite-scroll.tsx
|   |   |-- filter-carousel.tsx
|   |-- db/
|   |   |-- index.ts
|   |   |-- schema.ts
|   |-- lib/
|   |   |-- mux.ts
|   |   |-- uploadthing.ts
|   |   |-- redis.ts
|   |   |-- ratelimit.ts
|   |   |-- workflow.ts
|   |   |-- utils.ts
|   |-- modules/
|   |   |-- auth/
|   |   |-- categories/
|   |   |-- comments/
|   |   |-- comment-reactions/
|   |   |-- home/
|   |   |-- playlists/
|   |   |-- search/
|   |   |-- studio/
|   |   |-- subscriptions/
|   |   |-- suggestions/
|   |   |-- users/
|   |   |-- video-reactions/
|   |   |-- video-views/
|   |   |-- videos/
|   |-- scripts/
|   |   |-- seed-categories.ts
|   |-- trpc/
|   |   |-- init.ts
|   |   |-- client.tsx
|   |   |-- server.tsx
|   |   |-- query-client.ts
|   |   |-- routers/_app.ts
|   |-- constants.ts
|   |-- middleware.ts
|-- .env.example
|-- drizzle.config.ts
|-- next.config.ts
|-- tailwind.config.ts
|-- package.json
|-- tsconfig.json
```

### How to understand this structure

- `src/app` contains routes, layouts, API endpoints, and route groups.
- `src/modules` contains feature-based code. Each feature has UI files and/or server procedures.
- `src/db` contains the database connection and table definitions.
- `src/trpc` connects the frontend to backend procedures.
- `src/lib` contains reusable service clients such as Mux, Redis, UploadThing, and Workflow.
- `src/components/ui` contains generated shadcn/Radix UI components.

---

## Prerequisites

Install these before starting:

- Bun 1.0 or newer
- Node.js 18 or newer
- Git
- A Neon or PostgreSQL database
- A Clerk account
- A Mux account
- An UploadThing account
- An Upstash Redis account
- An Upstash Workflow/QStash setup
- An OpenAI API key
- ngrok account or another public tunnel provider

Check versions:

```bash
node -v
bun -v
git --version
```

---

## Version 1: Basic MVP

The first goal is to make the existing codebase run locally with authentication, database tables, and the home feed.

### STEP 1: Clone the Project

What we are doing: downloading the exact GitHub repo.

Why this is needed: the guide follows this codebase only, so we start from the same source.

Command:

```bash
git clone https://github.com/code-with-antonio/next15-youtube-clone.git
cd next15-youtube-clone
```

How to check:

```bash
ls
```

You should see files like:

- `package.json`
- `src`
- `.env.example`
- `drizzle.config.ts`
- `tailwind.config.ts`

---

### STEP 2: Install Dependencies

What we are doing: installing all packages from `package.json`.

Why this is needed: Next.js, Clerk, Mux, Drizzle, tRPC, UploadThing, Upstash, and UI packages must exist in `node_modules`.

Using Bun:

```bash
bun install
```

Using npm:

```bash
npm install --legacy-peer-deps
```

The repo uses Bun in its scripts, so Bun is the smoother path.

---

### STEP 3: Create Environment File

What we are doing: creating local secrets/config values.

Why this is needed: the app cannot connect to database, Clerk, Mux, Redis, or OpenAI without environment variables.

Copy the example:

```bash
cp .env.example .env.local
```

If you are on Windows PowerShell:

```powershell
Copy-Item .env.example .env.local
```

Fill `.env.local`:

```env
DATABASE_URL=your_postgres_or_neon_url
NEXT_PUBLIC_APP_URL=http://localhost:3000

MUX_TOKEN_ID=your_mux_token_id
MUX_TOKEN_SECRET=your_mux_token_secret
MUX_WEBHOOK_SECRET=your_mux_webhook_secret

OPENAI_API_KEY=your_openai_api_key

UPSTASH_REDIS_REST_URL=your_upstash_redis_url
UPSTASH_REDIS_REST_TOKEN=your_upstash_redis_token
UPSTASH_WORKFLOW_URL=your_public_app_url_for_workflows
QSTASH_TOKEN=your_qstash_token

CLERK_SIGNING_SECRET=your_clerk_webhook_signing_secret
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=your_clerk_publishable_key
CLERK_SECRET_KEY=your_clerk_secret_key
```

Important: the repo's `drizzle.config.ts` loads `.env.local`, so use `.env.local` for database commands.

---

### STEP 4: Understand `package.json` Scripts

The important scripts are:

```json
{
  "dev": "next dev",
  "dev:webhook": "ngrok http --url=evolved-humbly-gopher.ngrok-free.app 3000",
  "dev:all": "concurrently \"bun run dev:webhook\" \"bun run dev\"",
  "build": "next build",
  "start": "next start",
  "lint": "next lint"
}
```

What each script does:

- `dev`: starts the Next.js app at `http://localhost:3000`.
- `dev:webhook`: starts an ngrok tunnel for local webhooks.
- `dev:all`: starts both Next.js and ngrok together.
- `build`: creates a production build.
- `start`: runs the production build.

Important: the hardcoded ngrok URL belongs to the original repo author. Replace it with your own ngrok domain or use a normal ngrok command:

```bash
ngrok http 3000
```

Then use the generated public URL in Clerk, Mux, and Upstash webhook/workflow settings.

---

### STEP 5: Setup Database Tables

What we are doing: creating PostgreSQL tables from `src/db/schema.ts`.

Why this is needed: the app stores users, videos, comments, likes, views, subscriptions, and playlists in PostgreSQL.

Push schema:

```bash
bunx drizzle-kit push
```

Then seed categories:

```bash
bun run src/scripts/seed-categories.ts
```

How to check:

- Open Neon/PostgreSQL dashboard.
- You should see tables like `users`, `videos`, `categories`, `comments`, `subscriptions`, and `playlists`.
- The `categories` table should contain default category rows such as Education, Gaming, Music, Sports, and more.

---

### STEP 6: Run the App

Start only the app:

```bash
bun run dev
```

Start app plus webhook tunnel:

```bash
bun run dev:all
```

Expected output:

```text
Local: http://localhost:3000
```

Open:

```text
http://localhost:3000
```

At this point you should see the home layout. If the database has no public videos, the feed may be empty.

---

## Backend Setup

In this project, "backend" means:

- Database schema in `src/db/schema.ts`
- Database client in `src/db/index.ts`
- tRPC procedures in `src/modules/*/server/procedures.ts`
- API route handlers in `src/app/api`
- Auth protection in `src/middleware.ts`
- Webhooks for Clerk and Mux
- Background workflows for AI features

---

### STEP 1: Database Connection

File:

```text
src/db/index.ts
```

Purpose:

- Creates the Drizzle database client.
- Uses `DATABASE_URL` from environment variables.
- Allows server code to query PostgreSQL.

Concept:

```ts
db.select().from(table)
db.insert(table).values(data)
db.update(table).set(data)
db.delete(table)
```

Drizzle lets us write SQL-like queries in TypeScript.

---

### STEP 2: Database Schema

File:

```text
src/db/schema.ts
```

Main tables:

| Table | Purpose |
|---|---|
| `users` | Stores app user profiles synced from Clerk |
| `videos` | Stores uploaded video metadata |
| `categories` | Stores video categories |
| `comments` | Stores comments and replies |
| `video_reactions` | Stores video likes/dislikes |
| `comment_reactions` | Stores comment likes/dislikes |
| `video_views` | Stores watch history and view records |
| `subscriptions` | Stores viewer-to-creator subscriptions |
| `playlists` | Stores user-created playlists |
| `playlist_videos` | Join table connecting playlists and videos |

Important relationships:

- One user can upload many videos.
- One user can write many comments.
- One video can have many comments.
- One video can have many views.
- One video can have many reactions.
- One playlist can contain many videos.
- One user can subscribe to many creators.

Important enums:

- `reaction_type`: `like` or `dislike`
- `video_visibility`: `private` or `public`

---

### STEP 3: tRPC Setup

Files:

```text
src/trpc/init.ts
src/trpc/client.tsx
src/trpc/server.tsx
src/trpc/routers/_app.ts
src/app/api/trpc/[trpc]/route.ts
```

What tRPC does:

- Lets frontend call backend procedures with full TypeScript safety.
- Avoids manually writing many REST endpoints.
- Uses React Query for caching/loading states.

Two procedure types:

- `baseProcedure`: public procedure. Anyone can call it.
- `protectedProcedure`: private procedure. User must be signed in.

Protected procedure flow:

```text
Client calls protected procedure
Clerk checks session
App finds matching user row in database
Upstash checks rate limit
Procedure runs if everything is valid
```

Main app router:

```text
src/trpc/routers/_app.ts
```

It combines:

- `users`
- `studio`
- `videos`
- `search`
- `comments`
- `playlists`
- `categories`
- `videoViews`
- `suggestions`
- `subscriptions`
- `videoReactions`
- `commentReactions`

---

### STEP 4: Backend Route Handlers

#### Route: `GET/POST /api/trpc/[trpc]`

File:

```text
src/app/api/trpc/[trpc]/route.ts
```

Purpose:

- Main tRPC HTTP endpoint.
- All tRPC client calls go through this route.

Example request concept:

```text
Frontend calls trpc.videos.getMany.useSuspenseInfiniteQuery(...)
Request goes to /api/trpc/videos.getMany
Server runs videosRouter.getMany
Response returns typed video data
```

Example response shape:

```json
{
  "items": [],
  "nextCursor": null
}
```

---

#### Route: `POST /api/users/webhook`

File:

```text
src/app/api/users/webhook/route.ts
```

Purpose:

- Receives Clerk user events.
- Creates a row in `users` when a Clerk user signs up.
- Updates local user profile when Clerk user changes.
- Deletes local user when Clerk user is deleted.

Why it matters:

Clerk owns authentication, but your database needs a local `users` table so videos, comments, subscriptions, and playlists can reference a user ID.

Webhook events handled:

- `user.created`
- `user.updated`
- `user.deleted`

Testing:

1. Create a Clerk account in the app.
2. Check your database `users` table.
3. A new user row should appear.

---

#### Route: `POST /api/videos/webhook`

File:

```text
src/app/api/videos/webhook/route.ts
```

Purpose:

- Receives Mux video processing events.
- Updates the `videos` table after Mux creates or processes a video.
- Saves Mux asset ID, playback ID, status, duration, thumbnail, and preview.

Mux events handled:

- `video.asset.created`
- `video.asset.ready`
- `video.asset.errored`
- `video.asset.deleted`
- `video.asset.track.ready`

Important flow:

```text
User starts upload
App creates Mux direct upload URL
Mux processes video
Mux sends webhook to /api/videos/webhook
App updates the video row
Video becomes playable after playback ID exists
```

Testing:

1. Upload a video from Studio.
2. Wait for Mux processing.
3. Check the video row in the database.
4. Confirm `muxPlaybackId`, `muxStatus`, `thumbnailUrl`, and `duration` are updated.

---

#### Route: `GET/POST /api/uploadthing`

Files:

```text
src/app/api/uploadthing/core.ts
src/app/api/uploadthing/route.ts
```

Purpose:

- Handles image uploads through UploadThing.
- Supports user banner upload.
- Supports custom video thumbnail upload.

Upload routes:

- `bannerUploader`
- `thumbnailUploader`

Security:

- The middleware checks Clerk auth.
- It finds the logged-in user from the database.
- It only allows updating the current user's banner or current user's video thumbnail.

Testing:

1. Open your user channel page.
2. Upload a banner.
3. Check `users.bannerUrl` and `users.bannerKey`.
4. Open Studio video edit page.
5. Upload a custom thumbnail.
6. Check `videos.thumbnailUrl` and `videos.thumbnailKey`.

---

#### Route: `GET /users/current`

File:

```text
src/app/(home)/users/current/route.ts
```

Purpose:

- Redirects the logged-in user to their own channel page.

Flow:

```text
Check Clerk session
Find local user by clerkId
Redirect to /users/[userId]
```

If not logged in, it redirects to sign in.

---

#### Route: `POST /api/videos/workflows/title`

File:

```text
src/app/api/videos/workflows/title/route.ts
```

Purpose:

- Runs an Upstash Workflow job.
- Fetches the Mux transcript.
- Sends transcript to OpenAI.
- Updates the video title in the database.

Input concept:

```json
{
  "userId": "database-user-id",
  "videoId": "video-id"
}
```

---

#### Route: `POST /api/videos/workflows/description`

File:

```text
src/app/api/videos/workflows/description/route.ts
```

Purpose:

- Gets video transcript from Mux.
- Uses OpenAI to summarize it.
- Saves generated description in `videos.description`.

---

#### Route: `POST /api/videos/workflows/thumbnail`

File:

```text
src/app/api/videos/workflows/thumbnail/route.ts
```

Purpose:

- Uses OpenAI image generation to create a thumbnail from a prompt.
- Uploads generated image into UploadThing.
- Saves thumbnail URL and key in the video row.

Input concept:

```json
{
  "userId": "database-user-id",
  "videoId": "video-id",
  "prompt": "A bright thumbnail for a tutorial about Next.js"
}
```

---

## tRPC Procedure Map

These are the main backend procedures exposed by the repo.

### `categories`

| Procedure | Type | Purpose |
|---|---|---|
| `getMany` | public query | Fetch all categories |

### `videos`

| Procedure | Type | Purpose |
|---|---|---|
| `getMany` | public query | Home feed videos |
| `getManyTrending` | public query | Trending videos ordered by views |
| `getManySubscribed` | protected query | Videos from subscribed creators |
| `getOne` | public query | Single watch-page video |
| `create` | protected mutation | Create Mux upload and video row |
| `update` | protected mutation | Edit title, description, category, visibility |
| `remove` | protected mutation | Delete own video |
| `revalidate` | protected mutation | Recheck Mux asset status |
| `restoreThumbnail` | protected mutation | Restore Mux generated thumbnail |
| `generateTitle` | protected mutation | Trigger AI title workflow |
| `generateDescription` | protected mutation | Trigger AI description workflow |
| `generateThumbnail` | protected mutation | Trigger AI thumbnail workflow |

### `studio`

| Procedure | Type | Purpose |
|---|---|---|
| `getMany` | protected query | Creator's own videos |
| `getOne` | protected query | One owned video for editing |

### `comments`

| Procedure | Type | Purpose |
|---|---|---|
| `getMany` | public query | Fetch comments or replies |
| `create` | protected mutation | Add comment or reply |
| `remove` | protected mutation | Delete own comment |

### `videoReactions`

| Procedure | Type | Purpose |
|---|---|---|
| `like` | protected mutation | Like or toggle video like |
| `dislike` | protected mutation | Dislike or toggle video dislike |

### `commentReactions`

| Procedure | Type | Purpose |
|---|---|---|
| `like` | protected mutation | Like or toggle comment like |
| `dislike` | protected mutation | Dislike or toggle comment dislike |

### `videoViews`

| Procedure | Type | Purpose |
|---|---|---|
| `create` | protected mutation | Record that user watched video |

### `subscriptions`

| Procedure | Type | Purpose |
|---|---|---|
| `getMany` | protected query | Current user's subscriptions |
| `create` | protected mutation | Subscribe to creator |
| `remove` | protected mutation | Unsubscribe from creator |

### `playlists`

| Procedure | Type | Purpose |
|---|---|---|
| `create` | protected mutation | Create playlist |
| `getMany` | protected query | Current user's playlists |
| `getOne` | protected query | One playlist |
| `remove` | protected mutation | Delete playlist |
| `addVideo` | protected mutation | Add video to playlist |
| `removeVideo` | protected mutation | Remove video from playlist |
| `getVideos` | protected query | Videos inside playlist |
| `getManyForVideo` | protected query | Playlists with contains-video state |
| `getLiked` | protected query | Current user's liked videos |
| `getHistory` | protected query | Current user's watch history |

### `search`

| Procedure | Type | Purpose |
|---|---|---|
| `getMany` | public query | Search public videos by title and category |

### `users`

| Procedure | Type | Purpose |
|---|---|---|
| `getOne` | public query | Fetch user channel data |

### `suggestions`

| Procedure | Type | Purpose |
|---|---|---|
| `getMany` | public query | Suggested videos for watch page |

---

## Database Setup Explained

### `users`

Stores app-level user profile data.

Important fields:

- `id`: internal database user ID.
- `clerkId`: ID from Clerk.
- `name`: display name.
- `imageUrl`: profile picture.
- `bannerUrl`: channel banner image.

Why both Clerk ID and database ID exist:

- Clerk handles authentication.
- Your database uses its own UUIDs for relationships.

---

### `videos`

Stores video metadata.

Important fields:

- `title`
- `description`
- `muxUploadId`
- `muxAssetId`
- `muxPlaybackId`
- `muxTrackId`
- `thumbnailUrl`
- `previewUrl`
- `duration`
- `visibility`
- `userId`
- `categoryId`

Important concept:

The actual video file is not stored in PostgreSQL. Mux stores and streams the video. PostgreSQL stores references to Mux and metadata for your app.

---

### `comments`

Stores both top-level comments and replies.

Important fields:

- `id`
- `parentId`
- `userId`
- `videoId`
- `value`

How replies work:

- A normal comment has `parentId = null`.
- A reply has `parentId = id of another comment`.

---

### `video_reactions` and `comment_reactions`

These store likes and dislikes.

The primary key uses both user ID and video/comment ID. That means:

- One user can react once to a video.
- One user can react once to a comment.
- The reaction can be changed or removed.

---

### `subscriptions`

Stores a relationship between a viewer and a creator.

Important fields:

- `viewerId`: the person subscribing.
- `creatorId`: the channel being subscribed to.

The primary key is `(viewerId, creatorId)`, which prevents duplicate subscriptions.

---

### `playlists` and `playlist_videos`

`playlists` stores playlist information.

`playlist_videos` connects playlists to videos.

Why use a join table:

- One playlist can have many videos.
- One video can be added to many playlists.

This is a many-to-many relationship.

---

## Frontend Setup

The frontend uses Next.js App Router route groups.

### Route Groups

| Route group | Purpose |
|---|---|
| `(auth)` | Sign in and sign up pages |
| `(home)` | Viewer-facing YouTube experience |
| `(studio)` | Creator Studio experience |

Route groups do not appear in the URL. For example:

```text
src/app/(home)/page.tsx
```

becomes:

```text
/
```

---

### Root Layout

File:

```text
src/app/layout.tsx
```

Purpose:

- Loads the Inter font.
- Wraps the app in `ClerkProvider`.
- Wraps the app in `TRPCProvider`.
- Adds the global toaster.

Concept:

```text
ClerkProvider gives authentication to the whole app.
TRPCProvider gives API access and React Query caching to the whole app.
```

---

### Home Layout

Files:

```text
src/app/(home)/layout.tsx
src/modules/home/ui/layouts/home-layout.tsx
```

Purpose:

- Gives the viewer-facing app a navbar and sidebar.
- Used by home feed, search, subscriptions, playlists, user pages, and watch pages.

Important UI files:

```text
src/modules/home/ui/components/home-navbar/index.tsx
src/modules/home/ui/components/home-navbar/search-input.tsx
src/modules/home/ui/components/home-sidebar/index.tsx
src/modules/home/ui/components/home-sidebar/main-section.tsx
src/modules/home/ui/components/home-sidebar/personal-section.tsx
src/modules/home/ui/components/home-sidebar/subscriptions-section.tsx
```

---

### Studio Layout

Files:

```text
src/app/(studio)/layout.tsx
src/modules/studio/ui/layouts/studio-layout.tsx
```

Purpose:

- Gives creators their own dashboard layout.
- Includes studio navbar and sidebar.

Important UI files:

```text
src/modules/studio/ui/components/studio-navbar/index.tsx
src/modules/studio/ui/components/studio-sidebar/index.tsx
src/modules/studio/ui/components/studio-upload-modal.tsx
src/modules/studio/ui/components/studio-uploader.tsx
```

---

### Home Page

File:

```text
src/app/(home)/page.tsx
```

What it does:

- Reads optional `categoryId` from search params.
- Prefetches categories.
- Prefetches public videos.
- Renders `HomeView`.

Data flow:

```text
page.tsx prefetches data on server
HydrateClient passes data to client
HomeView renders sections
HomeVideosSection uses infinite query
VideoGridCard displays each video
```

---

### Search Page

File:

```text
src/app/(home)/search/page.tsx
```

What it does:

- Reads search query from URL.
- Fetches matching videos.
- Supports category filtering.

Backend procedure:

```text
trpc.search.getMany
```

---

### Video Watch Page

File:

```text
src/app/(home)/videos/[videoId]/page.tsx
```

Main view:

```text
src/modules/videos/ui/views/video-view.tsx
```

Important components:

```text
video-player.tsx
video-top-row.tsx
video-owner.tsx
video-reactions.tsx
video-description.tsx
comments-section.tsx
suggestions-section.tsx
```

What happens:

1. App loads video by ID.
2. Mux player plays the video using `muxPlaybackId`.
3. App shows owner, views, likes, dislikes, comments, and suggestions.
4. If user is logged in, app can record a view and allow reactions/comments.

---

### Studio Page

File:

```text
src/app/(studio)/studio/page.tsx
```

Main view:

```text
src/modules/studio/ui/views/studio-view.tsx
```

What it does:

- Fetches videos uploaded by the logged-in creator.
- Displays creator video list.
- Allows opening upload modal.

Backend procedure:

```text
trpc.studio.getMany
```

---

### Studio Video Edit Page

File:

```text
src/app/(studio)/studio/videos/[videoId]/page.tsx
```

Main view:

```text
src/modules/studio/ui/views/video-view.tsx
```

What it does:

- Lets creator edit title, description, category, and visibility.
- Lets creator upload or restore thumbnail.
- Lets creator generate AI title, description, and thumbnail.
- Lets creator delete video.

Backend procedures:

```text
trpc.studio.getOne
trpc.videos.update
trpc.videos.remove
trpc.videos.generateTitle
trpc.videos.generateDescription
trpc.videos.generateThumbnail
trpc.videos.restoreThumbnail
trpc.videos.revalidate
```

---

## Authentication

This repo uses Clerk, so you do not manually build password hashing, JWT signing, or login forms.

### Sign Up Flow

```text
User opens /sign-up
Clerk SignUp component handles registration
Clerk creates the auth user
Clerk sends user.created webhook
/api/users/webhook verifies the event
App inserts user into local users table
User can now use protected features
```

### Sign In Flow

```text
User opens /sign-in
Clerk SignIn component handles login
Clerk creates session cookie
Next middleware can protect routes
tRPC protectedProcedure can read current Clerk user
```

### Protected Routes

File:

```text
src/middleware.ts
```

Protected paths:

```text
/studio(.*)
/subscriptions
/feed/subscribed
/playlists(.*)
```

Meaning:

- Visitors can browse public pages.
- Logged-out users cannot access Studio, subscriptions, subscribed feed, or playlists.

### Protected tRPC Procedures

Protected procedures check:

1. Is there a Clerk session?
2. Does this Clerk user exist in local `users` table?
3. Has the user exceeded rate limit?

Only then does the procedure run.

---

## Version 2: Add Important Features

The repo already contains these features. This section teaches how each feature works and which files control it.

### Feature 1: Video Upload

Backend files:

```text
src/modules/videos/server/procedures.ts
src/lib/mux.ts
src/app/api/videos/webhook/route.ts
```

Frontend files:

```text
src/modules/studio/ui/components/studio-upload-modal.tsx
src/modules/studio/ui/components/studio-uploader.tsx
```

Flow:

```text
Creator clicks upload
Frontend calls trpc.videos.create
Server creates Mux direct upload
Server inserts video row with status waiting
Frontend uploads file to Mux
Mux processes video
Mux webhook updates video row
```

How to test:

1. Sign in.
2. Go to `/studio`.
3. Upload a video.
4. Wait for processing.
5. Open the edit page.
6. Change visibility to public.
7. Visit home feed and watch page.

---

### Feature 2: Public Video Feed

Backend:

```text
trpc.videos.getMany
```

Frontend:

```text
src/app/(home)/page.tsx
src/modules/home/ui/views/home-view.tsx
src/modules/home/ui/sections/home-videos-section.tsx
```

Logic:

- Only public videos are shown.
- Optional `categoryId` filters videos.
- Infinite scrolling uses a cursor with `updatedAt` and `id`.

Why cursor pagination:

Cursor pagination is better than page numbers for infinite feeds because new rows do not easily break the order.

---

### Feature 3: Trending Feed

Backend:

```text
trpc.videos.getManyTrending
```

Frontend:

```text
src/app/(home)/feed/trending/page.tsx
src/modules/home/ui/views/trending-view.tsx
src/modules/home/ui/sections/trending-videos-section.tsx
```

Logic:

- Videos are ordered by view count.
- Public videos only.
- Infinite scrolling uses view count plus video ID as cursor.

---

### Feature 4: Search

Backend:

```text
trpc.search.getMany
```

Frontend:

```text
src/modules/home/ui/components/home-navbar/search-input.tsx
src/app/(home)/search/page.tsx
src/modules/search/ui/views/search-view.tsx
```

Logic:

- User types a query.
- Query is stored in URL search params.
- Backend uses case-insensitive title matching.
- Category filter can also be applied.

---

### Feature 5: Video Watch Page

Backend:

```text
trpc.videos.getOne
trpc.videoViews.create
trpc.suggestions.getMany
```

Frontend:

```text
src/modules/videos/ui/views/video-view.tsx
src/modules/videos/ui/components/video-player.tsx
src/modules/videos/ui/sections/suggestions-section.tsx
```

Logic:

- Gets video details.
- Includes creator data.
- Includes view count, like count, dislike count, and viewer reaction.
- Shows suggested videos.
- Records a watch event for signed-in users.

---

### Feature 6: Comments and Replies

Backend:

```text
trpc.comments.getMany
trpc.comments.create
trpc.comments.remove
trpc.commentReactions.like
trpc.commentReactions.dislike
```

Frontend:

```text
src/modules/comments/ui/components/comment-form.tsx
src/modules/comments/ui/components/comment-item.tsx
src/modules/comments/ui/components/comment-replies.tsx
src/modules/videos/ui/sections/comments-section.tsx
```

Logic:

- Top-level comments have no `parentId`.
- Replies store the parent comment ID.
- The backend prevents replying to a reply.
- Users can delete only their own comments.

---

### Feature 7: Likes and Dislikes

Backend:

```text
trpc.videoReactions.like
trpc.videoReactions.dislike
trpc.commentReactions.like
trpc.commentReactions.dislike
```

Frontend:

```text
src/modules/videos/ui/components/video-reactions.tsx
src/modules/comments/ui/components/comment-item.tsx
```

Logic:

- If user clicks like for first time, create a like row.
- If user clicks like again, remove the like.
- If user previously disliked and then likes, update reaction type.
- Same idea works for dislikes.

---

### Feature 8: Subscriptions

Backend:

```text
trpc.subscriptions.getMany
trpc.subscriptions.create
trpc.subscriptions.remove
trpc.videos.getManySubscribed
```

Frontend:

```text
src/modules/subscriptions/ui/components/subscription-button.tsx
src/modules/subscriptions/ui/views/subscriptions-view.tsx
src/app/(home)/feed/subscribed/page.tsx
```

Logic:

- A viewer can subscribe to another user.
- A user cannot subscribe to themselves.
- Subscribed feed shows videos from creators the user follows.

---

### Feature 9: Playlists

Backend:

```text
trpc.playlists.create
trpc.playlists.getMany
trpc.playlists.getOne
trpc.playlists.addVideo
trpc.playlists.removeVideo
trpc.playlists.getVideos
trpc.playlists.getLiked
trpc.playlists.getHistory
```

Frontend:

```text
src/modules/playlists/ui/components/playlist-create-modal.tsx
src/modules/playlists/ui/components/playlist-add-modal.tsx
src/modules/playlists/ui/views/playlists-view.tsx
src/modules/playlists/ui/views/videos-view.tsx
src/modules/playlists/ui/views/liked-view.tsx
src/modules/playlists/ui/views/history-view.tsx
```

Logic:

- User creates playlists.
- User adds public videos to playlists.
- Liked videos are calculated from `video_reactions`.
- Watch history is calculated from `video_views`.

---

### Feature 10: User Channel Pages

Backend:

```text
trpc.users.getOne
trpc.videos.getMany
```

Frontend:

```text
src/app/(home)/users/[userId]/page.tsx
src/modules/users/ui/views/user-view.tsx
src/modules/users/ui/components/user-page-banner.tsx
src/modules/users/ui/components/user-page-info.tsx
src/modules/users/ui/sections/videos-section.tsx
```

Logic:

- Shows creator profile.
- Shows creator's public videos.
- Shows subscriber count.
- Allows subscription if visitor is logged in.
- Allows banner upload for current user's own channel.

---

### Feature 11: AI Title, Description, and Thumbnail

Backend files:

```text
src/modules/videos/server/procedures.ts
src/app/api/videos/workflows/title/route.ts
src/app/api/videos/workflows/description/route.ts
src/app/api/videos/workflows/thumbnail/route.ts
src/lib/workflow.ts
```

Frontend files:

```text
src/modules/studio/ui/sections/form-section.tsx
src/modules/studio/ui/components/thumbnail-generate-modal.tsx
```

Flow:

```text
Creator clicks generate
Frontend calls protected tRPC mutation
Mutation triggers Upstash Workflow
Workflow calls OpenAI
Workflow updates video row
Studio page shows updated data
```

Why use workflow:

AI calls can be slow. A workflow lets the job run outside the normal user request and makes the app more reliable.

---

## Running the Application

### Terminal 1: Start Next.js

```bash
bun run dev
```

Open:

```text
http://localhost:3000
```

### Terminal 2: Start ngrok

```bash
ngrok http 3000
```

Use the ngrok public URL for:

- Clerk webhook URL: `https://your-ngrok-url/api/users/webhook`
- Mux webhook URL: `https://your-ngrok-url/api/videos/webhook`
- Upstash workflow URL: `https://your-ngrok-url`

Or update the repo script:

```json
"dev:webhook": "ngrok http 3000"
```

Then:

```bash
bun run dev:all
```

---

## Testing the Application Manually

### Test Auth

1. Open `/sign-up`.
2. Create an account.
3. Check Clerk dashboard.
4. Check database `users` table.
5. Open `/studio`.
6. Confirm you are allowed in.

### Test Upload

1. Go to `/studio`.
2. Click upload.
3. Upload a video.
4. Wait for Mux processing.
5. Confirm database video row has Mux fields.
6. Edit title/category/visibility.
7. Set visibility to public.

### Test Feed

1. Open `/`.
2. Confirm public videos appear.
3. Click category filters.
4. Open trending feed.
5. Open search page and search by title.

### Test Watch Page

1. Click a video.
2. Confirm video player loads.
3. Like and dislike video.
4. Add comment.
5. Reply to comment.
6. Like/dislike comment.

### Test Subscriptions

1. Open another user's channel page.
2. Click subscribe.
3. Open `/subscriptions`.
4. Open `/feed/subscribed`.

### Test Playlists

1. Create a playlist.
2. Add a video to playlist.
3. Open playlist page.
4. Remove video from playlist.
5. Check liked videos page.
6. Check history page.

### Test AI

1. Upload a video with captions/transcript.
2. Open Studio edit page.
3. Generate title.
4. Generate description.
5. Generate thumbnail from prompt.
6. Confirm updates save in database.

---

## Key Concepts for Beginners

### Full-Stack Next.js

In traditional apps, frontend and backend may be separate projects. In this repo, both live in one Next.js app.

Frontend:

```text
src/app pages
src/modules/*/ui
src/components
```

Backend:

```text
src/app/api
src/modules/*/server
src/db
src/trpc
```

---

### Server Components vs Client Components

Server components run on the server. They can prefetch data and access server-only code.

Client components run in the browser. They can use state, click handlers, forms, modals, and interactive hooks.

In this repo:

- Pages often prefetch data on the server.
- UI components use client-side hooks for interaction.

---

### tRPC

tRPC replaces many manual REST endpoints.

Instead of:

```text
fetch("/api/videos")
```

The app uses:

```text
trpc.videos.getMany
```

Benefit:

- The frontend knows the input and output types automatically.
- Fewer duplicated API types.
- Better developer experience.

---

### React Query

React Query handles:

- Loading states
- Error states
- Caching
- Refetching
- Infinite scroll

tRPC integrates with React Query in this app.

---

### Drizzle ORM

Drizzle lets you define tables in TypeScript and query them safely.

Example concept:

```text
select videos
join users
where visibility is public
order by updatedAt desc
limit result
```

This is how feeds, search, trending, comments, and playlists are built.

---

### Webhooks

A webhook is an HTTP request sent by an external service to your app.

This project uses:

- Clerk webhook for user sync.
- Mux webhook for video processing status.

Why webhooks are needed:

The app cannot know when Clerk creates a user or when Mux finishes processing unless those services notify the app.

---

### Mux Direct Upload

The app does not upload large video files through your Next.js server.

Instead:

```text
App asks Mux for upload URL
Browser uploads video directly to Mux
Mux processes video
Mux tells app when ready
```

This is better because large uploads do not overload your app server.

---

### Cursor Pagination

The app uses cursor pagination for infinite scroll.

Instead of:

```text
page=1
page=2
```

It uses:

```text
last item id
last item updatedAt
```

This helps keep feeds stable when new videos are added.

---

### Visibility

Videos can be:

- `private`
- `public`

New videos start private. They should only appear in public feeds when visibility is set to public.

---

### Rate Limiting

File:

```text
src/lib/ratelimit.ts
```

The app uses Upstash Redis sliding window rate limiting.

Purpose:

- Prevent users from spamming protected actions.
- Protect backend resources.

---

## Common Errors and Solutions

### Error: Database connection failed

Cause:

- Wrong `DATABASE_URL`
- Database is paused
- `.env.local` missing

Solution:

1. Check `.env.local`.
2. Confirm Neon/PostgreSQL database is active.
3. Run `bunx drizzle-kit push`.

---

### Error: Clerk user exists but database user missing

Cause:

- Clerk webhook not configured.
- Webhook secret wrong.
- Local app not exposed through ngrok.

Solution:

1. Start ngrok.
2. Set Clerk webhook URL to `/api/users/webhook`.
3. Add correct `CLERK_SIGNING_SECRET`.
4. Create a new test user.

---

### Error: Studio route redirects to sign in

Cause:

- User is not signed in.
- Clerk keys missing.
- Session cookie not active.

Solution:

1. Add Clerk publishable and secret keys.
2. Sign in again.
3. Check `/sign-in`.

---

### Error: Mux video never becomes playable

Cause:

- Mux webhook not configured.
- `MUX_WEBHOOK_SECRET` wrong.
- ngrok URL changed.
- Mux still processing.

Solution:

1. Check Mux dashboard event logs.
2. Confirm webhook URL is `/api/videos/webhook`.
3. Confirm webhook secret.
4. Run `trpc.videos.revalidate` from Studio UI if needed.

---

### Error: AI title/description does not generate

Cause:

- Missing `OPENAI_API_KEY`.
- Missing Mux transcript.
- Missing Upstash workflow URL.
- Workflow public URL points to wrong app.

Solution:

1. Check `OPENAI_API_KEY`.
2. Check Mux track status.
3. Check `UPSTASH_WORKFLOW_URL`.
4. Check Upstash Workflow logs.

---

### Error: UploadThing upload fails

Cause:

- UploadThing env variables missing.
- User not authenticated.
- Video does not belong to current user.

Solution:

1. Add UploadThing credentials required by your UploadThing project.
2. Sign in.
3. Try uploading banner or thumbnail again.

---

### Error: Categories not showing

Cause:

- Seed script not run.

Solution:

```bash
bun run src/scripts/seed-categories.ts
```

---

### Error: `next lint` does not work

Cause:

- Next.js 15 changed lint behavior in some setups.
- The repo script is `next lint`.

Solution:

Use the repo script first:

```bash
bun run lint
```

If it fails due Next.js lint command changes, use ESLint directly:

```bash
bunx eslint .
```

---

## Final Summary

You built and studied a complete full-stack YouTube clone using the exact `code-with-antonio/next15-youtube-clone` architecture.

You learned:

- How a full-stack Next.js app is structured.
- How route groups organize auth, home, and studio areas.
- How Clerk handles authentication.
- How Clerk webhooks sync users into PostgreSQL.
- How Drizzle defines relational database tables.
- How tRPC exposes type-safe backend procedures.
- How React Query powers loading, caching, and infinite scroll.
- How Mux handles video upload, processing, playback, thumbnails, and transcripts.
- How UploadThing handles image uploads.
- How Upstash Redis rate limits protected actions.
- How Upstash Workflow runs AI jobs.
- How playlists, comments, reactions, views, subscriptions, and feeds are modeled.

---

## Next Improvements

After completing the repo, useful improvements are:

1. Add real analytics dashboard in Studio.
2. Add comment editing.
3. Add playlist privacy.
4. Add channel search.
5. Add notification system.
6. Add video chapters.
7. Add live streaming with Mux.
8. Add moderation tools.
9. Add admin dashboard.
10. Add tests for tRPC procedures.
11. Add production deployment guide for Vercel.
12. Add database migrations instead of schema push only.

