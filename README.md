# StoryWeaver

Empowering anyone to create stunning videos with AI. StoryWeaver is a visual, node-based video creation environment designed to make advanced AI video generation accessible to everyone. By providing an intuitive, interactive canvas, users can easily piece together scenes, craft AI prompts, and orchestrate complex generative workflows to bring their stories to life—no prior video editing experience required.

## Key Features

- **Visual Node Editor:** An intuitive drag-and-drop canvas powered by React Flow.
- **AI-Powered Generation:** Seamless integration with Google GenAI for prompt-to-video capabilities.
- **Cloud Storage:** Native support for Google Cloud Storage to manage and retrieve media assets.
- **Modern Monorepo:** Structured using Turborepo with separate workspaces for the server, studio interface, main client, and shared utilities.

## Tech Stack

- **Language**: TypeScript
- **Monorepo Manager**: Turborepo, Bun Workspaces
- **Backend Framework**: Bun, Hono
- **Database**: PostgreSQL with Drizzle ORM
- **Authentication**: Better Auth
- **Frontend Framework**: React, Vite
- **Styling**: Tailwind CSS, Radix UI
- **State Management**: Zustand, TanStack Query
- **Routing**: TanStack Router
- **Interactive Canvas**: React Flow (@xyflow/react)
- **AI / Cloud Services**: Google GenAI, Google Cloud Storage

## Prerequisites

Before starting, ensure you have the following installed on your machine:
- [Bun](https://bun.sh/) (v1.2+)
- PostgreSQL (or Docker for running a local Postgres container)
- A Google Cloud project configured with GenAI and Cloud Storage credentials

## Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/stevedylandev/bhvr.git storyweaver
cd storyweaver
```

### 2. Install Dependencies

Install the packages for all workspaces via Bun:

```bash
bun install
```

### 3. Environment Setup

Create `.env` files in your workspace (or at the root, depending on your setup) using the following configuration. The core environment variables required are:

| Variable | Description | Example |
| -------- | ----------- | ------- |
| `DATABASE_URL` | PostgreSQL connection string | `postgresql://user:pass@localhost:5432/storyweaver` |
| `BETTER_AUTH_SECRET` | Secret key for Better Auth sessions | `your-secret-string` |
| `BETTER_AUTH_URL` | Base URL for auth callbacks | `http://localhost:5173` |
| `GEMINI_API_KEY` | Your Google Gemini API Key | `AIzaSy...` |
| `GCS_BUCKET_NAME` | Google Cloud Storage Bucket Name | `my-storyweaver-bucket` |
| `GOOGLE_APPLICATION_CREDENTIALS` | Path to your GCP service account JSON | `./gcp-key.json` |

Additional optional variables for OAuth include `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`, `GOOGLE_CLIENT_ID`, and `GOOGLE_CLIENT_SECRET`.

### 4. Database Setup

Ensure PostgreSQL is running. Run the Drizzle ORM migrations (the setup expects your schema in `server/src/db/schemas/schema.ts`):

```bash
cd server
bunx drizzle-kit push
cd ..
```

### 5. Start Development Server

Run the development servers across all workspaces concurrently using Turbo:

```bash
bun run dev
```

This command will automatically spin up:
- The **Server** (Hono API running on its defined port)
- The **Client** (Vite app on `http://localhost:5173` typically)
- The **Studio** (Vite app for the React Flow canvas)

## Architecture

### Directory Structure

```
├── client/              # Main frontend application (Landing, Dashboard, etc.)
│   ├── src/             # React components, TanStack router paths
│   └── package.json
├── server/              # Backend API and Database interactions
│   ├── src/             
│   │   ├── db/          # Drizzle ORM schemas and connections
│   │   ├── lib/         # Middlewares (Auth, CORS) and utility functions
│   │   ├── routes/      # Hono route definitions
│   │   ├── app.ts       # Main Hono app configuration
│   │   └── index.ts     # Entry point
│   ├── drizzle.config.ts# Drizzle configuration
│   └── package.json
├── shared/              # Shared TypeScript definitions, Zod schemas, utilities
│   ├── src/             # Common types and schemas
│   └── package.json
├── studio/              # Specialized video creator environment
│   ├── src/             # React Flow nodes, panels, Zustand stores
│   └── package.json
├── package.json         # Root workspace package.json
└── turbo.json           # Turborepo configuration
```

### Request Lifecycle

1. **User Interaction**: User creates nodes in the `studio` interface.
2. **Frontend State**: Zustand maintains the flowchart state while React Flow renders the canvas.
3. **API Call**: Submitting a generation request hits the Hono `server` API.
4. **Backend Processing**: The Hono server validates the request using Zod (from the `shared` workspace) and triggers the Google GenAI service.
5. **Asset Storage**: The generated media is uploaded to Google Cloud Storage.
6. **Database Persistence**: Video metadata and URLs are saved to PostgreSQL via Drizzle ORM.
7. **Client Update**: The updated asset URL is returned to the frontend and rendered back onto the canvas.

## Environment Variables

### Required for Local Development

| Variable | Description |
| -------- | ----------- |
| `DATABASE_URL` | Used by Drizzle to connect to Postgres. |
| `GEMINI_API_KEY` | Required to communicate with Google's GenAI endpoints. |
| `GCS_BUCKET_NAME` | The bucket used to store user-generated video content. |
| `GOOGLE_APPLICATION_CREDENTIALS` | The local path to the Google service account key file for GCS access. |
| `BETTER_AUTH_SECRET` | Required by Better Auth for signing cookies and tokens. |
| `BETTER_AUTH_URL` | The application base URL for authentication flows. |

## Available Scripts

From the root of the project, you can run:

| Command | Description |
| ------- | ----------- |
| `bun run dev` | Starts all development servers (Client, Server, Studio) using Turborepo. |
| `bun run build` | Builds all packages. |
| `bun run lint` | Runs ESLint across all workspaces. |
| `bun run type-check` | Runs TypeScript compiler checks. |
| `bun run build:single` | Builds the client and studio, copies their static assets to `server/static`, and builds the server for production deployment. |
| `bun run start:single` | Starts the production server which serves the API and the static SPA assets. |

## Testing

Testing is handled via Turborepo:

```bash
bun run test
```

## Deployment

StoryWeaver is designed to be bundled into a single Node/Bun process that serves the Hono API and statically hosts the compiled Vite frontends.

### Docker (Recommended)

1. **Build the production assets**:
   ```bash
   bun run build:single
   ```
2. **Containerize**: Create a Dockerfile that copies the `server/dist` and `server/static` directories.
3. **Run**:
   ```bash
   docker run -p 3000:3000 -e DATABASE_URL=... -e GEMINI_API_KEY=... my-storyweaver-image
   ```

### Standard VPS / PaaS (e.g. Render, Railway)

1. **Build Command**: 
   ```bash
   bun install && bun run build:single
   ```
2. **Start Command**: 
   ```bash
   bun run start:single
   ```
3. **Environment**: Ensure all `DATABASE_URL`, `GEMINI_API_KEY`, and Google Cloud secrets are securely set in the platform's environment variables.

## Troubleshooting

### Database Connection Issues
**Error:** `PostgresError: Connection refused`  
**Solution:** Ensure your PostgreSQL server is running locally on port `5432` or update the `DATABASE_URL` in your `.env` to match your local setup.

### Google GenAI Fails to Authenticate
**Error:** Authentication/Authorization failures from Google APIs.  
**Solution:** Verify that your `GEMINI_API_KEY` is correct. If using Google Cloud Storage, double-check that the `GOOGLE_APPLICATION_CREDENTIALS` variable points to a valid, accessible JSON key file with the proper IAM roles assigned (Storage Object Admin).

### TypeScript/Zod Import Errors
**Error:** Cannot find module `shared` or similar cross-workspace import errors.  
**Solution:** Run `bun install` at the root to ensure Bun properly links the workspace dependencies. You may also need to run `bun run build` in the `shared` package to generate the types initially if your editor complains.
