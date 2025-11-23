# CLAUDE.md - AI Assistant Guide for boilerplateV3

## Project Overview

This is a **full-stack fleet management and blog application** built with modern edge-first architecture. The application serves multiple business domains:

- **Blog/CMS** - Public-facing blog with post management
- **Fleet Management** - Truck tracking and monitoring
- **Timesheet Management** - Driver timesheet tracking with approval workflows
- **Maintenance Tracking** - Comprehensive vehicle maintenance logs and inspections

**Primary Use Case:** Trucking/fleet management company with integrated blog for public communication.

## Technology Stack

### Frontend
- **Framework:** Next.js 15.2.4 with App Router
- **React:** Version 19 (latest)
- **Runtime:** Bun v1.0.0+
- **UI Components:** Shadcn UI (Radix UI primitives)
- **Styling:** Tailwind CSS 4.0.15
- **State Management:** TanStack Query v5 (React Query)
- **Forms:** React Hook Form + Zod validation
- **Theme:** next-themes (dark/light mode)
- **Icons:** Lucide React

### Backend
- **API Framework:** Hono 4.7.5 (Cloudflare Worker)
- **Database:** Cloudflare D1 (SQLite-based edge database)
- **ORM:** Drizzle ORM 0.41.0
- **Authentication:** Better Auth 1.2.4
- **Validation:** Zod schemas

### Development Tools
- **Package Manager:** Bun (NOT npm/yarn)
- **Type Checking:** TypeScript 5
- **Linting:** ESLint 9
- **Formatting:** Prettier with import sorting
- **Database Migrations:** Drizzle Kit
- **Deployment:** Cloudflare (Workers + Pages + D1)

## Architecture Patterns

### 1. Edge-First Architecture
The application is designed to run on Cloudflare's global network:
- Frontend deployed to Cloudflare Pages
- API deployed to Cloudflare Workers
- Database runs on Cloudflare D1 (distributed SQLite)

### 2. Type-Safe RPC Pattern

**Critical Convention:** This application uses Hono's RPC client for type-safe API calls.

#### Server-Side Components (React Server Components)
```typescript
import { getServerRPC } from "@/lib/server-rpc";

// In async server component
const api = getServerRPC();
const posts = await api.posts.$get();
```

#### Client-Side Components
```typescript
import { clientRPC } from "@/lib/client-rpc";

// In client component
const { data } = useQuery({
  queryKey: ['posts'],
  queryFn: async () => {
    const res = await clientRPC.posts.$get();
    return res.json();
  }
});
```

**Key Difference:**
- `getServerRPC()` - Use in server components (requires cookies from Next.js headers)
- `clientRPC` - Use in client components (uses js-cookie)

### 3. Authentication Architecture

**Better Auth** handles authentication with these features:
- Email/password authentication
- Magic link support
- OpenAPI integration
- Admin capabilities
- User impersonation (admin feature)
- Session-based with cookies

**User Roles:**
- `driver` - Can submit timesheets
- `maintenance` - Can manage maintenance logs
- `admin` - Full access including user management

**Auth Helper Functions** (located in `src/lib/auth-utils.ts`):
- `getSessionUser()` - Get current user from session
- `requireAuth()` - Require authentication (throws if not authenticated)
- `requireRole(role)` - Require specific role
- `requireAdmin()` - Require admin role

### 4. Database Schema Organization

**Schema Files** (all in `src/server/db/`):
- `auth-schema.sql.ts` - Users, sessions, accounts, verifications
- `post-schema.sql.ts` - Blog posts
- `timesheet-schema.sql.ts` - Trucks, timesheets, billing rates
- `maintenance-schema.sql.ts` - Maintenance logs and items

**Important Notes:**
- All schema files use `.sql.ts` suffix
- Schemas are imported and merged in `src/server/db/index.ts`
- Migrations are auto-generated in `src/server/db/migrations/`
- Use `bun run db:generate` to create new migrations
- Use `bun run db:migrate` to apply migrations locally
- Database binding name is `DB` in Cloudflare Worker

## Project Structure

```
/home/user/boilerplateV3/
├── src/
│   ├── app/                          # Next.js App Router
│   │   ├── admin/                    # Admin dashboard (requires admin role)
│   │   ├── api/[...route]/           # Hono API proxy route
│   │   ├── create/                   # Create blog post page
│   │   ├── login/                    # Login page
│   │   ├── post/[slug]/              # Dynamic blog post pages
│   │   ├── register/                 # Registration page
│   │   ├── layout.tsx                # Root layout with theme provider
│   │   ├── page.tsx                  # Homepage (blog listing)
│   │   └── globals.css               # Global styles
│   │
│   ├── components/
│   │   ├── ui/                       # Shadcn UI components (don't modify)
│   │   ├── blog-card.tsx             # Blog post card component
│   │   ├── blog-content.tsx          # Blog post content renderer
│   │   ├── create-post-form.tsx      # Post creation form
│   │   ├── header.tsx                # Site header
│   │   ├── footer.tsx                # Site footer
│   │   └── theme-provider.tsx        # Dark mode provider
│   │
│   ├── config/
│   │   └── site.ts                   # Site-wide configuration
│   │
│   ├── env/
│   │   ├── client.ts                 # Client-side env validation
│   │   └── server.ts                 # Server-side env validation
│   │
│   ├── lib/
│   │   ├── auth.ts                   # Better Auth configuration
│   │   ├── auth-client.ts            # Client-side auth helpers
│   │   ├── auth-utils.ts             # Server-side auth utilities
│   │   ├── client-rpc.ts             # Client RPC setup
│   │   ├── server-rpc.ts             # Server RPC setup
│   │   ├── types.ts                  # Shared TypeScript types
│   │   └── utils.ts                  # Utility functions (cn, etc.)
│   │
│   └── server/                       # Hono backend
│       ├── db/
│       │   ├── migrations/           # Drizzle migrations (auto-generated)
│       │   ├── auth-schema.sql.ts    # Auth tables
│       │   ├── post-schema.sql.ts    # Post tables
│       │   ├── timesheet-schema.sql.ts
│       │   ├── maintenance-schema.sql.ts
│       │   └── index.ts              # DB instance export
│       │
│       ├── middlewares/
│       │   ├── cors-middleware.ts    # CORS configuration
│       │   ├── csrf-middleware.ts    # CSRF protection
│       │   └── session-middleware.ts # Session handling
│       │
│       ├── routes/
│       │   ├── auth-route.ts         # Auth endpoints (/api/auth/*)
│       │   ├── posts-route.ts        # Blog post endpoints
│       │   ├── timesheet-route.ts    # Timesheet endpoints
│       │   └── maintenance-route.ts  # Maintenance endpoints
│       │
│       ├── validations/
│       │   ├── post-schema.ts        # Post validation schemas
│       │   ├── timesheet-schema.ts   # Timesheet validation
│       │   └── maintenance-schema.ts # Maintenance validation
│       │
│       ├── hono-factory.ts           # Hono app factory with context
│       └── index.ts                  # Main Hono server entry
│
├── public/                           # Static assets
├── wrangler.jsonc                    # Cloudflare Worker config
├── drizzle.config.ts                 # Drizzle ORM config
├── next.config.ts                    # Next.js config
├── tsconfig.json                     # TypeScript config
└── package.json                      # Dependencies and scripts
```

## File Naming Conventions

**IMPORTANT:** Follow these conventions when creating new files:

- `.sql.ts` - Database schema files (e.g., `user-schema.sql.ts`)
- `.schema.ts` - Zod validation schemas (e.g., `post-schema.ts`)
- `-route.ts` - API route handlers (e.g., `posts-route.ts`)
- `-middleware.ts` - Hono middleware (e.g., `auth-middleware.ts`)
- Component files use kebab-case (e.g., `blog-card.tsx`)
- Utility files use kebab-case (e.g., `auth-utils.ts`)

## Development Workflows

### Starting Development

```bash
# Terminal 1: Start Next.js dev server
bun run dev

# Terminal 2: Start Hono worker
bun run worker:dev
```

**URLs:**
- Frontend: http://localhost:3000
- API: http://localhost:8787

### Creating New Features

#### 1. Adding a New API Endpoint

**Step 1:** Create validation schema in `src/server/validations/`
```typescript
// example-schema.ts
import { z } from "zod";

export const createExampleSchema = z.object({
  name: z.string().min(1),
  description: z.string().optional(),
});

export type CreateExampleInput = z.infer<typeof createExampleSchema>;
```

**Step 2:** Create route handler in `src/server/routes/`
```typescript
// example-route.ts
import { zValidator } from "@hono/zod-validator";
import { createHonoFactory } from "../hono-factory";
import { createExampleSchema } from "../validations/example-schema";

const app = createHonoFactory();

// GET /api/examples
export const exampleRoute = app
  .get("/", async (c) => {
    const db = c.get("db");
    const examples = await db.query.examples.findMany();
    return c.json(examples);
  })
  // POST /api/examples
  .post("/", zValidator("json", createExampleSchema), async (c) => {
    const db = c.get("db");
    const data = c.req.valid("json");

    const [example] = await db
      .insert(examples)
      .values(data)
      .returning();

    return c.json(example);
  });
```

**Step 3:** Register route in `src/server/index.ts`
```typescript
import { exampleRoute } from "./routes/example-route";

// Add to routes
const routes = app
  .route("/api/auth", authRoute)
  .route("/api/posts", postsRoute)
  .route("/api/examples", exampleRoute); // Add this
```

**Step 4:** Use in frontend (auto-typed!)
```typescript
// Client component
import { clientRPC } from "@/lib/client-rpc";

const { data } = useQuery({
  queryKey: ['examples'],
  queryFn: async () => {
    const res = await clientRPC.examples.$get();
    return res.json();
  }
});

// Server component
import { getServerRPC } from "@/lib/server-rpc";

const api = getServerRPC();
const examples = await api.examples.$get();
```

#### 2. Adding a Database Table

**Step 1:** Create schema in `src/server/db/`
```typescript
// example-schema.sql.ts
import { sqliteTable, text, integer } from "drizzle-orm/sqlite-core";

export const examples = sqliteTable("examples", {
  id: integer("id").primaryKey({ autoIncrement: true }),
  name: text("name").notNull(),
  description: text("description"),
  createdAt: integer("created_at", { mode: "timestamp" })
    .$defaultFn(() => new Date()),
});
```

**Step 2:** Export from `src/server/db/index.ts`
```typescript
import * as exampleSchema from "./example-schema.sql";

const schema = {
  ...authSchema,
  ...postSchema,
  ...exampleSchema, // Add this
};
```

**Step 3:** Generate and run migration
```bash
bun run db:generate
bun run db:migrate
```

#### 3. Adding a New Page

**Step 1:** Create page in `src/app/`
```typescript
// src/app/examples/page.tsx
import { getServerRPC } from "@/lib/server-rpc";

export default async function ExamplesPage() {
  const api = getServerRPC();
  const res = await api.examples.$get();
  const examples = await res.json();

  return (
    <div>
      <h1>Examples</h1>
      {examples.map(example => (
        <div key={example.id}>{example.name}</div>
      ))}
    </div>
  );
}
```

**Step 2:** Add navigation link in `src/components/header.tsx`

### Database Operations

#### Common Drizzle Patterns

```typescript
import { db } from "@/server/db";
import { posts, users } from "@/server/db/post-schema.sql";
import { eq, and, desc, like } from "drizzle-orm";

// SELECT * FROM posts
const allPosts = await db.query.posts.findMany();

// SELECT with relations
const postsWithAuthors = await db.query.posts.findMany({
  with: {
    author: true, // Join with users table
  },
});

// SELECT with WHERE
const post = await db.query.posts.findFirst({
  where: eq(posts.id, postId),
});

// INSERT
const [newPost] = await db
  .insert(posts)
  .values({
    title: "New Post",
    content: "Content here",
    authorId: userId,
  })
  .returning();

// UPDATE
await db
  .update(posts)
  .set({ title: "Updated Title" })
  .where(eq(posts.id, postId));

// DELETE
await db
  .delete(posts)
  .where(eq(posts.id, postId));
```

### Authentication Patterns

#### Protecting Server Components
```typescript
import { requireAuth, requireRole } from "@/lib/auth-utils";

export default async function ProtectedPage() {
  // Throws redirect to login if not authenticated
  const user = await requireAuth();

  // Throws error if user doesn't have admin role
  await requireRole("admin");

  return <div>Welcome {user.name}</div>;
}
```

#### Protecting API Routes
```typescript
import { createHonoFactory } from "../hono-factory";

const app = createHonoFactory();

export const protectedRoute = app.post("/", async (c) => {
  const session = c.get("session");

  if (!session?.user) {
    return c.json({ error: "Unauthorized" }, 401);
  }

  // Check role
  if (session.user.role !== "admin") {
    return c.json({ error: "Forbidden" }, 403);
  }

  // Proceed with logic
  return c.json({ success: true });
});
```

#### Client-Side Auth
```typescript
"use client";

import { useSession } from "@/lib/auth-client";

export function ClientComponent() {
  const { data: session, isPending } = useSession();

  if (isPending) return <div>Loading...</div>;
  if (!session) return <div>Please login</div>;

  return <div>Welcome {session.user.name}</div>;
}
```

## Environment Variables

### Required Environment Variables

**`.env`** (Next.js)
```env
NEXT_PUBLIC_API_URL=http://localhost:8787
BETTER_AUTH_URL=http://localhost:8787
```

**`.dev.vars`** (Cloudflare Worker - local dev)
```env
BETTER_AUTH_SECRET=your-secret-key-min-32-chars
GOOGLE_CLIENT_ID=optional-google-oauth-client-id
GOOGLE_CLIENT_SECRET=optional-google-oauth-secret
```

**Environment Validation:**
- Client env vars are validated in `src/env/client.ts`
- Server env vars are validated in `src/env/server.ts`
- Uses `@t3-oss/env-nextjs` with Zod schemas
- Type-safe access via `process.env`

## Code Quality Standards

### Import Organization

Prettier automatically sorts imports in this order:
1. React imports
2. Third-party packages (alphabetically)
3. Internal imports with `@/` alias
4. Relative imports (`./`, `../`)

```typescript
// Good
import { useState } from "react";
import { useQuery } from "@tanstack/react-query";
import { Button } from "@/components/ui/button";
import { clientRPC } from "@/lib/client-rpc";
import { formatDate } from "./utils";

// Bad (will be auto-formatted)
import { formatDate } from "./utils";
import { useState } from "react";
import { Button } from "@/components/ui/button";
```

### TypeScript Conventions

- Use `type` for object types, `interface` for extensible contracts
- Infer types from Zod schemas: `type T = z.infer<typeof schema>`
- Use Drizzle's `InferSelectModel` for table types
- Enable strict mode (already configured)
- Use path aliases: `@/*` for `src/*`

### Component Conventions

```typescript
// Server Component (default)
export default async function ServerPage() {
  const data = await fetchData();
  return <div>{data}</div>;
}

// Client Component (add "use client")
"use client";

export default function ClientPage() {
  const [state, setState] = useState();
  return <div>{state}</div>;
}
```

### Styling Conventions

- Use Tailwind utility classes
- Use `cn()` helper for conditional classes
- Dark mode: use `dark:` prefix
- Responsive: use `sm:`, `md:`, `lg:` prefixes

```typescript
import { cn } from "@/lib/utils";

<div className={cn(
  "bg-white dark:bg-gray-900",
  "p-4 rounded-lg",
  isActive && "border-blue-500"
)} />
```

## Common Pitfalls and Gotchas

### 1. Database Name Mismatch
**Issue:** `package.json` uses `timeconnectv2` for database name, but project is `nextjs-hono-better-auth-d1`

```json
// package.json shows:
"db:migrate": "wrangler d1 migrations apply timeconnectv2"
```

**Fix:** Update to match your actual database name in `wrangler.jsonc`

### 2. RPC Client Confusion
**Common Error:** Using `clientRPC` in server components

```typescript
// ❌ Wrong - will fail
export default async function Page() {
  const res = await clientRPC.posts.$get(); // Error!
}

// ✅ Correct
export default async function Page() {
  const api = getServerRPC();
  const res = await api.posts.$get();
}
```

### 3. Duplicate Auth Schema
**Issue:** There's a `auth-schema.ts` in root directory (duplicate/legacy)
- Use the one in `src/server/db/auth-schema.sql.ts` instead

### 4. Session Middleware Order
**Critical:** Middleware order matters in Hono!

```typescript
// Must be in this order:
app
  .use("*", cors)       // 1. CORS first
  .use("*", csrf)       // 2. CSRF protection
  .use("*", session)    // 3. Session last (depends on cookies)
```

### 5. Database Migrations
- Always generate migration before modifying schema: `bun run db:generate`
- Test locally before remote: `bun run db:migrate` then `bun run db:migrate-remote`
- Migrations are one-way (no rollback in D1)

### 6. Type Exports
**Issue:** Forgetting to export types from API routes breaks RPC typing

```typescript
// ✅ Always export the type
export const postsRoute = app.get("/", ...);
export type PostsRoute = typeof postsRoute;
```

## Testing

**Note:** Testing setup is not currently configured. To add testing:

1. Install dependencies: `bun add -d vitest @testing-library/react`
2. Create `vitest.config.ts`
3. Add test scripts to `package.json`
4. Create test files with `.test.ts` or `.test.tsx` extension

## Deployment

### Deploy to Cloudflare

```bash
# Deploy API (Hono Worker)
bun run worker:deploy

# Deploy Frontend (Cloudflare Pages)
bun run pages:deploy

# Run migrations on production database
bun run db:migrate-remote
```

### Pre-deployment Checklist

- [ ] Environment variables set in Cloudflare dashboard
- [ ] Database created and ID updated in `wrangler.jsonc`
- [ ] Migrations applied to remote database
- [ ] BETTER_AUTH_SECRET set (secure random string)
- [ ] BETTER_AUTH_URL updated to production URL
- [ ] NEXT_PUBLIC_API_URL updated to production API URL

## Useful Commands

```bash
# Development
bun run dev                    # Start Next.js dev server
bun run worker:dev             # Start Hono worker dev server

# Code Quality
bun run lint                   # Run ESLint
bun run format                 # Format code with Prettier

# Database
bun run db:generate            # Generate migration from schema changes
bun run db:migrate             # Apply migrations locally
bun run db:migrate-remote      # Apply migrations to production

# Build & Deploy
bun run build                  # Build Next.js for production
bun run pages:build            # Build for Cloudflare Pages
bun run pages:deploy           # Deploy to Cloudflare Pages
bun run worker:deploy          # Deploy worker to Cloudflare
```

## Key Dependencies Reference

| Package | Purpose | Documentation |
|---------|---------|---------------|
| Next.js 15 | React framework | https://nextjs.org/docs |
| Hono | API framework | https://hono.dev |
| Better Auth | Authentication | https://www.better-auth.com |
| Drizzle ORM | Database ORM | https://orm.drizzle.team |
| TanStack Query | Data fetching | https://tanstack.com/query |
| Shadcn UI | Component library | https://ui.shadcn.com |
| Zod | Schema validation | https://zod.dev |
| Cloudflare D1 | Database | https://developers.cloudflare.com/d1 |

## Business Domain Models

### 1. Blog System
- Posts with title, content, slug, cover image
- Markdown support for content
- Author relationship to users
- Reading time calculation

### 2. Fleet Management
- Trucks (unit number, make, model, odometer)
- Maintenance intervals tracking
- Fuel efficiency metrics

### 3. Timesheet System
- Driver shifts with clock in/out
- Truck assignments
- Approval workflow (pending → approved/rejected)
- Billing rate integration
- Odometer tracking

### 4. Maintenance System
- Comprehensive 60+ point inspection checklist
- Maintenance items/parts tracking
- Staff assignment
- Truck relationship

## AI Assistant Best Practices

When working on this codebase:

1. **Always use Bun** - Not npm or yarn
2. **Check RPC context** - Use correct client (clientRPC vs getServerRPC)
3. **Follow naming conventions** - Especially for schema and route files
4. **Generate migrations** - Before modifying database schema
5. **Validate with Zod** - All API inputs should have Zod schemas
6. **Type safety first** - Leverage RPC types, don't use `any`
7. **Test both servers** - Changes may require restarting both dev servers
8. **Check middleware order** - CORS → CSRF → Session
9. **Use auth helpers** - Don't reinvent requireAuth/requireRole
10. **Read before modifying** - Understand existing patterns before changes

## Getting Help

- Check existing API routes in `src/server/routes/` for patterns
- Review validation schemas in `src/server/validations/` for Zod examples
- Look at `src/app/` pages for Next.js App Router patterns
- Consult `src/lib/` for utility functions and helpers
- Review database schemas in `src/server/db/` for table structures

## Version Information

- **Last Updated:** 2025-11-23
- **Next.js:** 15.2.4
- **React:** 19.0.0
- **Hono:** 4.7.5
- **Better Auth:** 1.2.4
- **Drizzle ORM:** 0.41.0
- **Bun:** 1.0.0+
