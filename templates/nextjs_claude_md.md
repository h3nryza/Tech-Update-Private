# Next.js / React / TypeScript — CLAUDE.md Template

> Copy this entire file into `.claude/CLAUDE.md` at your project root.
> Replace all `<!-- CUSTOMIZE -->` sections with your project-specific values.

---

```markdown
# CLAUDE.md

<!-- CUSTOMIZE: Replace with your project name and one-line description -->
## Project: My App
A Next.js application with TypeScript, Tailwind CSS, and Prisma.

---

## Build & Run Commands

<!-- These are the commands Claude will use. Verify they match your package.json scripts. -->

### Development
```bash
npm run dev              # Start dev server (localhost:3000)
npm run build            # Production build
npm run start            # Start production server
npm run lint             # ESLint
npm run lint:fix         # ESLint with auto-fix
npm run type-check       # TypeScript type checking (tsc --noEmit)
```

### Testing
```bash
npm run test             # Run all tests (Vitest)
npm run test:watch       # Watch mode
npm run test:coverage    # Coverage report
npm run test:ui          # Vitest UI
npm run test:e2e         # Playwright end-to-end tests
npm run test:e2e:ui      # Playwright UI mode
```

### Database
```bash
npx prisma generate      # Generate Prisma client
npx prisma db push       # Push schema to database
npx prisma migrate dev   # Create and apply migration
npx prisma studio        # Open Prisma Studio GUI
npx prisma db seed       # Seed database
```

### Single Test Patterns
```bash
# Run a specific test file
npx vitest run src/lib/__tests__/utils.test.ts

# Run tests matching a pattern
npx vitest run --reporter=verbose -t "should calculate total"

# Run Playwright for a specific test
npx playwright test tests/checkout.spec.ts
```

---

## Code Conventions

### TypeScript
- **Strict mode is ON** — never use `any` without a comment explaining why
- Use `interface` for object shapes, `type` for unions/intersections/utility types
- Prefer `const` assertions and `satisfies` over explicit type annotations where inference is sufficient
- Export types from the file where they're defined, re-export from barrel `index.ts` files

```typescript
// GOOD: Interface for object shapes
interface User {
  id: string;
  email: string;
  role: "admin" | "user";
}

// GOOD: Type for unions
type ApiResponse<T> = { data: T; error: null } | { data: null; error: string };

// GOOD: satisfies for type-safe inference
const config = {
  apiUrl: process.env.NEXT_PUBLIC_API_URL,
  timeout: 5000,
} satisfies AppConfig;
```

### React / Next.js
- **App Router** (Next.js 13+) — all routes live in `app/`
- Use Server Components by default. Add `"use client"` only when needed (event handlers, hooks, browser APIs)
- Colocate files: `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, component files all in the route folder
- Use `next/image` for all images, `next/link` for all internal links
- Server Actions for mutations — prefer over API routes for form submissions

```typescript
// GOOD: Server Component (default)
export default async function UsersPage() {
  const users = await getUsers(); // Direct DB call, no API needed
  return <UserList users={users} />;
}

// GOOD: Client component only when needed
"use client";
export function SearchBar() {
  const [query, setQuery] = useState("");
  // ...
}
```

### Component Patterns
- Name components with PascalCase: `UserCard.tsx`
- One component per file (small helpers in the same file are OK)
- Props interface named `{Component}Props`
- Use `children` prop for composition, not render props (unless truly needed)

```typescript
interface UserCardProps {
  user: User;
  onSelect?: (user: User) => void;
}

export function UserCard({ user, onSelect }: UserCardProps) {
  // ...
}
```

### Naming
- Files: `kebab-case.ts` for utilities, `PascalCase.tsx` for components
- Variables/functions: `camelCase`
- Constants: `UPPER_SNAKE_CASE` only for true constants (env vars, config)
- Database models: `PascalCase` singular (`User`, not `Users`)
- API routes: `kebab-case` (`/api/user-sessions`, not `/api/userSessions`)

---

## Styling (Tailwind CSS)

- Use Tailwind utility classes directly in JSX — no separate CSS files unless absolutely necessary
- Extract repeated patterns into components, not `@apply` directives
- Use `cn()` utility (clsx + tailwind-merge) for conditional classes
- Follow mobile-first responsive: `base` -> `sm:` -> `md:` -> `lg:` -> `xl:`
- Design tokens are in `tailwind.config.ts` — extend the theme, don't override defaults

```typescript
// cn() utility — put this in lib/utils.ts
import { type ClassValue, clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

// Usage
<div className={cn(
  "rounded-lg border p-4",
  isActive && "border-blue-500 bg-blue-50",
  className
)} />
```

---

## State Management

<!-- CUSTOMIZE: Pick the pattern that matches your app -->

### Server State (Recommended: TanStack Query or Server Components)
- Use Server Components + `fetch` for read-heavy pages
- Use TanStack Query (`@tanstack/react-query`) for client-side data fetching with caching
- Never store server data in Zustand/Redux — let the data fetching library own it

### Client State
- `useState` for component-local state
- `useReducer` for complex state logic within a component
- Zustand for shared client state (auth, UI preferences, modals)
- URL state (`useSearchParams`) for filterable/shareable UI state

```typescript
// Zustand store example
import { create } from "zustand";

interface UIStore {
  sidebarOpen: boolean;
  toggleSidebar: () => void;
}

export const useUIStore = create<UIStore>((set) => ({
  sidebarOpen: true,
  toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),
}));
```

---

## Project Structure

<!-- CUSTOMIZE: Adjust to match your actual directory layout -->

```
app/
  (auth)/              # Auth-related route group
    login/page.tsx
    register/page.tsx
  (dashboard)/         # Dashboard route group
    layout.tsx
    page.tsx
  api/                 # API routes (when Server Actions won't work)
    webhooks/
  layout.tsx           # Root layout
  page.tsx             # Home page
components/
  ui/                  # Reusable UI primitives (Button, Input, Card)
  forms/               # Form components
  layouts/             # Layout components (Header, Sidebar, Footer)
lib/
  db.ts                # Prisma client singleton
  auth.ts              # Auth configuration
  utils.ts             # cn(), formatters, helpers
  validations/         # Zod schemas
hooks/                 # Custom React hooks
types/                 # Shared TypeScript types
prisma/
  schema.prisma        # Database schema
  migrations/          # Migration files
  seed.ts              # Database seed script
public/                # Static assets
tests/                 # Playwright e2e tests
```

---

## Testing

### Unit Tests (Vitest)
- Test files colocated: `Button.test.tsx` next to `Button.tsx`, or in `__tests__/` folders
- Use `@testing-library/react` for component tests
- Mock external dependencies, not internal modules
- Minimum 80% coverage for `lib/` and `components/`

```typescript
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { UserCard } from "./UserCard";

describe("UserCard", () => {
  it("displays user name and calls onSelect when clicked", async () => {
    const user = { id: "1", name: "Alice", email: "alice@example.com" };
    const onSelect = vi.fn();

    render(<UserCard user={user} onSelect={onSelect} />);

    expect(screen.getByText("Alice")).toBeInTheDocument();

    await userEvent.click(screen.getByRole("button"));
    expect(onSelect).toHaveBeenCalledWith(user);
  });
});
```

### E2E Tests (Playwright)
- Test critical user flows: auth, checkout, CRUD operations
- Use Page Object Model for complex flows
- Tests run against a seeded test database

### What NOT to Test
- Tailwind classes or styling details
- Third-party library internals
- Next.js routing (trust the framework)

---

## Deployment (Vercel)

<!-- CUSTOMIZE: Replace with your deployment target if not Vercel -->

### Environments
| Environment | Branch | URL |
|-------------|--------|-----|
| Production | `main` | `<!-- CUSTOMIZE -->` |
| Preview | PRs | Auto-generated |
| Development | local | `localhost:3000` |

### Environment Variables
- Set in Vercel dashboard, NOT in `.env` files committed to git
- `.env.local` for local development (gitignored)
- `.env.example` committed with dummy values as documentation
- Prefix client-side vars with `NEXT_PUBLIC_`

### Pre-deployment Checks
Before merging to `main`, ensure:
1. `npm run build` succeeds
2. `npm run type-check` passes
3. `npm run lint` passes
4. `npm run test` passes
5. Playwright tests pass on preview deployment

---

## Error Handling

- Use `error.tsx` boundaries for route-level errors
- Wrap async Server Component data fetches in try/catch with user-friendly fallbacks
- Client-side: use `react-error-boundary` for component-level recovery
- API routes: return structured errors with status codes

```typescript
// Standard API error response
interface ApiError {
  error: string;
  code: string;
  details?: Record<string, string[]>;
}

// Usage in API route
return NextResponse.json(
  { error: "Validation failed", code: "VALIDATION_ERROR", details: errors },
  { status: 400 }
);
```

---

## Security

- Never expose server-side env vars to the client (no `NEXT_PUBLIC_` prefix for secrets)
- Validate all inputs with Zod — both client and server side
- Use `next-auth` or `clerk` for authentication — never roll your own
- CSRF protection is built into Server Actions — prefer them over raw API routes
- Rate limit API routes with `@upstash/ratelimit` or similar

---

## Common Tasks

### Add a new page
1. Create `app/your-route/page.tsx`
2. Add loading state: `app/your-route/loading.tsx`
3. Add error boundary: `app/your-route/error.tsx`

### Add a new API route
1. Create `app/api/your-route/route.ts`
2. Export named handlers: `GET`, `POST`, `PUT`, `DELETE`
3. Add Zod validation for request body

### Add a database model
1. Edit `prisma/schema.prisma`
2. Run `npx prisma migrate dev --name descriptive_name`
3. Run `npx prisma generate`
4. Add seed data to `prisma/seed.ts`

### Add a reusable component
1. Create `components/ui/YourComponent.tsx`
2. Add props interface with JSDoc comments
3. Add test: `components/ui/YourComponent.test.tsx`
4. Export from `components/ui/index.ts`
```
