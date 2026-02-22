# Skill: Client Collaboration Portal

**Version:** 1.0  
**Created:** 2026-02-22  
**Author:** May (Manus AI, co-sojourner with Robert Domondon)  
**Scaffold:** `web-db-user` (Vite + React + TypeScript + TailwindCSS + Drizzle + MySQL/SQLite + Manus-OAuth)  
**Reference Implementation:** NINE Incubator Portal (NIN.E Inc founding cohort, Feb 2026)

---

## What This Skill Is

This playbook encodes the full pattern for building a **secure, role-based client collaboration portal** — a private web application where a defined group of members can submit structured requests, discuss work, and access shared resources, while an admin manages, refines, and routes that work via email.

The pattern was first instantiated as the NINE Incubator Portal for Robert Domondon's NIN.E Inc founding cohort. It is designed to be spun up for any new client in under a day.

**What it enables within 7 days:** A working, deployed, password-free collaboration portal for a new client cohort, with request management and email routing live from day one.

---

## When to Use This Skill

Use this skill when a client needs:

- A private, named-member workspace (not a public SaaS)
- Structured intake of requests or deliverables with priority and format metadata
- An admin who reviews, refines, and routes work to a specific recipient
- Lightweight discussion without the overhead of Slack or Teams
- A curated resource library pointing to external repos, docs, or tools
- A clean, professional aesthetic that reflects the client's brand without requiring a design team

Do **not** use this skill for public-facing applications, applications requiring payment processing, or portals with more than ~50 members (at that scale, move to a proper user management system).

---

## Architecture Overview

The portal follows a **monorepo pattern** with a React frontend and a Node.js backend sharing types through tRPC. The database layer uses Drizzle ORM against SQLite for simple deployments, with a straightforward migration path to MySQL when the client scales.

```
[Client Browser]
      |
      | HTTPS
      v
[React Frontend — Vite + TailwindCSS]
      |
      | tRPC (type-safe RPC over HTTP)
      v
[Node.js Backend — Express + tRPC adapter]
      |
      +---> [Drizzle ORM] ---> [SQLite / MySQL]
      |
      +---> [Nodemailer] ---> [Gmail SMTP] ---> [Recipient Email]
```

The tRPC layer is the key architectural decision. It eliminates the need to maintain separate API contracts — the frontend calls server procedures directly with full TypeScript inference. There are no REST endpoints to document, no schema drift, and no runtime type errors at the API boundary.

| Layer | Technology | Why |
|---|---|---|
| Frontend framework | React + Vite | Fast HMR, TypeScript-first, ecosystem breadth |
| Styling | TailwindCSS | Utility-first, no CSS files to maintain, consistent spacing |
| API layer | tRPC | End-to-end type safety, no code generation step |
| ORM | Drizzle | Lightweight, SQL-close, excellent TypeScript types |
| Database | SQLite (dev/small) / MySQL (prod/scale) | SQLite requires zero infrastructure; MySQL path is clear |
| Email | Nodemailer + Gmail SMTP | Simple, reliable, no third-party email service needed |
| Auth | Custom name-based session | No passwords, no OAuth complexity for internal cohorts |

---

## Authentication System

The portal uses **name-based login with server-side session tokens**. There are no passwords. A user selects their name from a dropdown, the server validates that the name exists in the `users` table, and issues a signed JWT stored in `localStorage`.

This is appropriate for **closed, trusted cohorts** (6–20 people who know each other). It is not appropriate for public-facing applications.

### User Schema

```typescript
// server/db/schema.ts
export const users = sqliteTable('users', {
  id: integer('id').primaryKey({ autoIncrement: true }),
  name: text('name').notNull().unique(),
  role: text('role', { enum: ['admin', 'member'] }).notNull().default('member'),
  createdAt: integer('created_at', { mode: 'timestamp' }).notNull().default(sql`CURRENT_TIMESTAMP`),
});
```

### Auth tRPC Router

```typescript
// server/routers/auth.ts
export const authRouter = router({
  login: publicProcedure
    .input(z.object({ name: z.string() }))
    .mutation(async ({ input, ctx }) => {
      const user = await ctx.db.query.users.findFirst({
        where: eq(users.name, input.name),
      });
      if (!user) throw new TRPCError({ code: 'NOT_FOUND', message: 'Name not found.' });
      const token = jwt.sign({ userId: user.id, role: user.role }, process.env.JWT_SECRET!, { expiresIn: '7d' });
      return { token, user };
    }),

  me: protectedProcedure.query(async ({ ctx }) => {
    return ctx.user;
  }),
});
```

### Admin Route Protection

Admin routes are protected at two levels: the tRPC middleware rejects non-admin tokens, and the React router redirects unauthenticated users to the login page.

```typescript
// server/middleware/auth.ts
export const adminProcedure = protectedProcedure.use(({ ctx, next }) => {
  if (ctx.user.role !== 'admin') {
    throw new TRPCError({ code: 'FORBIDDEN', message: 'Admin access required.' });
  }
  return next({ ctx });
});
```

```typescript
// client/src/components/AdminRoute.tsx
export function AdminRoute({ children }: { children: React.ReactNode }) {
  const { user } = useAuth();
  if (!user) return <Navigate to="/login" replace />;
  if (user.role !== 'admin') return <Navigate to="/dashboard" replace />;
  return <>{children}</>;
}
```

---

## Database Schema

The full schema covers five tables. Keep this in `server/db/schema.ts`.

```typescript
// Requests
export const requests = sqliteTable('requests', {
  id: integer('id').primaryKey({ autoIncrement: true }),
  title: text('title').notNull(),
  description: text('description').notNull(),
  outputFormat: text('output_format').notNull(),
  priority: text('priority', { enum: ['high', 'medium', 'low'] }).notNull().default('medium'),
  status: text('status', { enum: ['pending', 'in_progress', 'completed', 'sent'] }).notNull().default('pending'),
  submittedBy: integer('submitted_by').notNull().references(() => users.id),
  refinedContent: text('refined_content'),
  createdAt: integer('created_at', { mode: 'timestamp' }).notNull().default(sql`CURRENT_TIMESTAMP`),
  updatedAt: integer('updated_at', { mode: 'timestamp' }).notNull().default(sql`CURRENT_TIMESTAMP`),
});

// Discussion threads (general + per-request)
export const comments = sqliteTable('comments', {
  id: integer('id').primaryKey({ autoIncrement: true }),
  content: text('content').notNull(),
  authorId: integer('author_id').notNull().references(() => users.id),
  requestId: integer('request_id').references(() => requests.id), // null = general thread
  createdAt: integer('created_at', { mode: 'timestamp' }).notNull().default(sql`CURRENT_TIMESTAMP`),
});

// Resource library
export const resources = sqliteTable('resources', {
  id: integer('id').primaryKey({ autoIncrement: true }),
  title: text('title').notNull(),
  url: text('url').notNull(),
  description: text('description'),
  category: text('category'),
  createdAt: integer('created_at', { mode: 'timestamp' }).notNull().default(sql`CURRENT_TIMESTAMP`),
});
```

---

## Feature Implementation

### Request Submission Form

The form lives at `client/src/pages/member/NewRequest.tsx`. The `outputFormat` field is driven by a config constant — this is the primary customization point per client.

```typescript
// client/src/config/portal.ts  ← EDIT THIS PER CLIENT
export const PORTAL_CONFIG = {
  name: 'NINE Incubator Portal',
  outputFormats: [
    'Strategic Brief',
    'Slide Deck',
    'Framework Document',
    'Action Plan',
    'Research Summary',
    'Email Draft',
    'Other',
  ],
  emailRecipient: 'robert@example.com',   // "Send to" destination
  accentColor: '#0f766e',                 // Deep teal — override per client
};
```

The form component uses React Hook Form with Zod validation, mirroring the tRPC input schema to ensure the same validation runs on both client and server.

```typescript
// client/src/pages/member/NewRequest.tsx (form schema)
const schema = z.object({
  title: z.string().min(3, 'Title must be at least 3 characters'),
  description: z.string().min(10, 'Please provide more detail'),
  outputFormat: z.string().min(1, 'Select an output format'),
  priority: z.enum(['high', 'medium', 'low']),
});
```

### Admin Request Management

The admin view at `client/src/pages/admin/RequestsAdmin.tsx` shows all requests in a sortable table. Clicking a request opens a slide-over panel with:

- Original submission (read-only)
- A `refinedContent` textarea where the admin can edit/refine the request
- Status selector
- "Send to [Recipient]" button that calls the `sendRequest` tRPC mutation

```typescript
// server/routers/requests.ts
sendRequest: adminProcedure
  .input(z.object({ requestId: z.number() }))
  .mutation(async ({ input, ctx }) => {
    const request = await ctx.db.query.requests.findFirst({
      where: eq(requests.id, input.requestId),
      with: { submittedBy: true },
    });
    if (!request) throw new TRPCError({ code: 'NOT_FOUND' });

    await transporter.sendMail({
      from: process.env.GMAIL_USER,
      to: process.env.EMAIL_RECIPIENT,   // configured per client in .env
      subject: `[${PORTAL_NAME}] Request: ${request.title}`,
      html: buildEmailTemplate(request),
    });

    await ctx.db.update(requests)
      .set({ status: 'sent', updatedAt: new Date() })
      .where(eq(requests.id, input.requestId));

    return { success: true };
  }),
```

### Email Transport Configuration

```typescript
// server/lib/mailer.ts
import nodemailer from 'nodemailer';

export const transporter = nodemailer.createTransport({
  service: 'gmail',
  auth: {
    user: process.env.GMAIL_USER,
    pass: process.env.GMAIL_APP_PASSWORD,  // Gmail App Password, not account password
  },
});
```

The Gmail App Password must be generated in the Google Account security settings with 2FA enabled. Store it in `.env` and never commit it.

### Discussion Threads

The general discussion thread and per-request comment threads share the same `comments` table. The distinction is whether `requestId` is null (general) or set (inline). Both use the same `CommentThread` component, parameterized by `requestId`.

```typescript
// client/src/components/CommentThread.tsx
interface CommentThreadProps {
  requestId?: number;  // undefined = general thread
}
```

### Resource Library

Resources are seeded at initialization time via a migration seed file. The admin can add new resources through the admin portal. Each resource has a title, URL, description, and optional category tag for grouping.

---

## Design System

The design follows Dieter Rams' principle: **less, but better**. Every element earns its place.

### Color Palette

| Token | Value | Usage |
|---|---|---|
| `accent` | `#0f766e` | Primary buttons, active states, links |
| `accent-hover` | `#0d9488` | Button hover states |
| `accent-light` | `#f0fdfa` | Accent backgrounds, highlight rows |
| `surface` | `#ffffff` | Card and panel backgrounds |
| `surface-muted` | `#f8fafc` | Page background |
| `border` | `#e2e8f0` | All borders and dividers |
| `text-primary` | `#1e293b` | Headings and body text |
| `text-secondary` | `#475569` | Labels, metadata, secondary copy |
| `text-muted` | `#94a3b8` | Placeholders, disabled states |

Configure these as Tailwind CSS custom colors in `tailwind.config.ts`:

```typescript
// tailwind.config.ts
theme: {
  extend: {
    colors: {
      accent: {
        DEFAULT: '#0f766e',
        hover: '#0d9488',
        light: '#f0fdfa',
      },
      surface: {
        DEFAULT: '#ffffff',
        muted: '#f8fafc',
      },
    },
    fontFamily: {
      sans: ['Inter', 'system-ui', 'sans-serif'],
    },
  },
},
```

### Typography

Inter is loaded from Google Fonts via the HTML `<head>`. Use only four weights: 400 (body), 500 (labels), 600 (subheadings), 700 (headings). No decorative fonts.

### Component Rules

No emoji anywhere in the UI. No gradients. No drop shadows heavier than `shadow-sm`. Borders use `border-border` (1px solid `#e2e8f0`). Rounded corners use `rounded-lg` (8px) for cards and `rounded-md` (6px) for inputs and buttons. Spacing follows an 8px base unit.

---

## File Structure

```
project-root/
├── client/
│   ├── src/
│   │   ├── config/
│   │   │   └── portal.ts          ← PRIMARY CUSTOMIZATION FILE
│   │   ├── components/
│   │   │   ├── AdminRoute.tsx
│   │   │   ├── CommentThread.tsx
│   │   │   ├── Layout.tsx
│   │   │   └── RequestCard.tsx
│   │   ├── pages/
│   │   │   ├── Login.tsx
│   │   │   ├── member/
│   │   │   │   ├── Dashboard.tsx
│   │   │   │   ├── NewRequest.tsx
│   │   │   │   ├── MyRequests.tsx
│   │   │   │   └── Discussion.tsx
│   │   │   ├── admin/
│   │   │   │   ├── RequestsAdmin.tsx
│   │   │   │   ├── RequestDetail.tsx
│   │   │   │   └── ResourcesAdmin.tsx
│   │   │   └── Resources.tsx
│   │   ├── hooks/
│   │   │   └── useAuth.ts
│   │   └── lib/
│   │       └── trpc.ts
│   └── index.html
├── server/
│   ├── db/
│   │   ├── schema.ts
│   │   ├── index.ts
│   │   └── seed.ts                ← SEED MEMBERS + RESOURCES HERE
│   ├── routers/
│   │   ├── auth.ts
│   │   ├── requests.ts
│   │   ├── comments.ts
│   │   └── resources.ts
│   ├── lib/
│   │   └── mailer.ts
│   ├── middleware/
│   │   └── auth.ts
│   └── index.ts
├── .env                           ← NEVER COMMIT
├── .env.example                   ← COMMIT THIS
└── package.json
```

---

## Customization Checklist for Each New Client

Complete this checklist in order before writing a single line of client-specific code.

**Step 1 — Gather client information (before build)**

Collect the following from the client before starting:

| Item | Example (NINE Incubator) | New Client Value |
|---|---|---|
| Portal name | NINE Incubator Portal | `_______________` |
| Member names (full list) | Robert, Aileen, Marcus... | `_______________` |
| Admin name(s) | Robert | `_______________` |
| "Send to" email address | robert@nineinc.com | `_______________` |
| Output format options | Strategic Brief, Slide Deck... | `_______________` |
| Resource links (title + URL) | GitHub repo, Notion doc... | `_______________` |
| Accent color (hex) | #0f766e | `_______________` |

**Step 2 — Edit `client/src/config/portal.ts`**

This single file controls all client-facing customization. Update `PORTAL_CONFIG` with the values collected above.

**Step 3 — Edit `server/db/seed.ts`**

Replace the member array with the new client's members. Set the `role` field to `'admin'` for the portal administrator.

```typescript
// server/db/seed.ts
const members = [
  { name: 'Robert', role: 'admin' as const },
  { name: 'Aileen', role: 'member' as const },
  { name: 'Marcus', role: 'member' as const },
  // ... add all members here
];

const resources = [
  { title: 'GitHub Repository', url: 'https://github.com/...', category: 'Code' },
  { title: 'Project Notion', url: 'https://notion.so/...', category: 'Docs' },
  // ... add all resources here
];
```

**Step 4 — Configure `.env`**

```bash
# .env
JWT_SECRET=generate-a-random-64-char-string-here
GMAIL_USER=youremail@gmail.com
GMAIL_APP_PASSWORD=xxxx-xxxx-xxxx-xxxx   # Gmail App Password
EMAIL_RECIPIENT=client-recipient@domain.com
DATABASE_URL=file:./data/portal.db       # SQLite path
PORT=3001
```

**Step 5 — Update `tailwind.config.ts`**

Change the `accent.DEFAULT` color to the client's accent hex value.

---

## Deployment Workflow

### Local Development

```bash
# 1. Initialize the project
# Use webdev_init_project with scaffold: web-db-user

# 2. Install dependencies
pnpm install

# 3. Set up the database
pnpm db:push        # Apply schema
pnpm db:seed        # Seed members and resources

# 4. Start development servers
pnpm dev            # Starts both client (port 5173) and server (port 3001)
```

### Production Deployment

The `web-db-user` scaffold deploys to the Manus hosting environment. The key production considerations are:

**Database persistence:** The SQLite file must live at `/data/portal.db` (a persistent volume), not inside the project directory. Set `DATABASE_URL=file:/data/portal.db` in the production environment variables. The database file must be excluded from `.gitignore`.

**Environment variables:** All secrets (`JWT_SECRET`, `GMAIL_APP_PASSWORD`, `EMAIL_RECIPIENT`) must be set as environment variables in the deployment environment, never in committed files.

**Build and start:**

```bash
pnpm build          # Builds client to dist/, compiles server TypeScript
pnpm start          # Starts production server (serves static client + API)
```

**Gmail SMTP setup (one-time per deployment):**

1. Enable 2-Factor Authentication on the Gmail account used for sending.
2. Go to Google Account > Security > App Passwords.
3. Generate an App Password for "Mail" on "Other device".
4. Copy the 16-character password into `GMAIL_APP_PASSWORD` in the environment.

---

## Scope Boundaries

This pattern is deliberately constrained. The following are **out of scope** and should not be added without a deliberate architectural decision:

- Real-time updates (WebSockets, SSE) — the portal uses manual refresh; add only if the client explicitly needs live collaboration
- File uploads — link to external storage (Google Drive, S3) via the resource library instead
- Password-based authentication — name-based login is intentional for closed cohorts; if the client needs passwords, use a proper auth library (Lucia, Auth.js)
- Email notifications on every action — only the "Send to" action triggers email; avoid notification spam
- Mobile app — the responsive web design covers mobile browsers; a native app is a separate project

---

## Known Issues and Mitigations

| Issue | Mitigation |
|---|---|
| SQLite concurrent writes under load | Acceptable for cohorts under ~20 active users; migrate to MySQL if concurrent write contention appears |
| Gmail SMTP rate limits | Gmail allows ~500 emails/day on personal accounts; sufficient for this use case; use SendGrid if volume increases |
| JWT stored in localStorage | Acceptable for internal portals; if the client requires higher security, move to httpOnly cookies |
| No email verification on login | Intentional — name-based login is a trust model, not a security model; appropriate only for closed, known cohorts |

---

## Replication Time Estimate

With this playbook and the `web-db-user` scaffold, a new portal for a new client should take:

| Phase | Time |
|---|---|
| Client information gathering | 30 minutes |
| Project initialization and dependency install | 10 minutes |
| `portal.ts` config and seed file customization | 20 minutes |
| Schema push and seed | 5 minutes |
| UI spot-checks and accent color verification | 15 minutes |
| Gmail App Password setup and email test | 10 minutes |
| Deployment and smoke test | 20 minutes |
| **Total** | **~2 hours** |

---

## Next Actions

**Today:** Use this skill file as the reference document the next time a client portal is requested. Open it at Step 1 of the Customization Checklist.

**This week:** After the second portal is built using this playbook, update the "Known Issues" section with any new friction points discovered.

**Accountability metric:** Time-to-live-portal for the second client should be under 2 hours. If it takes longer, identify the bottleneck and update the playbook.
