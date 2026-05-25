---
name: react-expert
description: Use when building React 18+ applications in .jsx or .tsx files, Next.js App Router projects, or create-react-app setups. Creates components, implements custom hooks, debugs rendering issues, migrates class components to functional, and implements state management. Invoke for Server Components, Suspense boundaries, useActionState forms, performance optimization, or React 19 features.
license: MIT
metadata:
  author: https://github.com/Jeffallan
  version: "1.1.0"
  domain: frontend
  triggers: React, JSX, hooks, useState, useEffect, useContext, Server Components, React 19, Suspense, TanStack Query, Redux, Zustand, component, frontend
  role: specialist
  scope: implementation
  output-format: code
  related-skills: fullstack-guardian, playwright-expert, test-master
---

# React Expert

Senior React specialist with deep expertise in React 19, Server Components, and production-grade application architecture.

## When to Use This Skill

- Building new React components or features
- Implementing state management (local, Context, Redux, Zustand)
- Optimizing React performance
- Setting up React project architecture
- Working with React 19 Server Components
- Implementing forms with React 19 actions
- Data fetching patterns with TanStack Query or `use()`

## Core Workflow

1. **Analyze requirements** - Identify component hierarchy, state needs, data flow
2. **Choose patterns** - Select appropriate state management, data fetching approach
3. **Implement** - Write TypeScript components with proper types
4. **Validate** - Run `tsc --noEmit`; if it fails, review reported errors, fix all type issues, and re-run until clean before proceeding
5. **Optimize** - Apply memoization where needed, ensure accessibility; if new type errors are introduced, return to step 4
6. **Test** - Write tests with React Testing Library; if any assertions fail, debug and fix before submitting

## Reference Guide

Load detailed guidance based on context:

| Topic | Reference | Load When |
|-------|-----------|-----------|
| Server Components | `references/server-components.md` | RSC patterns, Next.js App Router |
| React 19 | `references/react-19-features.md` | use() hook, useActionState, forms |
| State Management | `references/state-management.md` | Context, Zustand, Redux, TanStack |
| Hooks | `references/hooks-patterns.md` | Custom hooks, useEffect, useCallback |
| Performance | `references/performance.md` | memo, lazy, virtualization |
| Testing | `references/testing-react.md` | Testing Library, mocking |
| Class Migration | `references/migration-class-to-modern.md` | Converting class components to hooks/RSC |

## Key Patterns

### Server Component (Next.js App Router)
```tsx
// app/users/page.tsx — Server Component, no "use client"
import { db } from '@/lib/db';

interface User {
  id: string;
  name: string;
}

export default async function UsersPage() {
  const users: User[] = await db.user.findMany();

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### React 19 Form with `useActionState`
```tsx
'use client';
import { useActionState } from 'react';

async function submitForm(_prev: string, formData: FormData): Promise<string> {
  const name = formData.get('name') as string;
  // perform server action or fetch
  return `Hello, ${name}!`;
}

export function GreetForm() {
  const [message, action, isPending] = useActionState(submitForm, '');

  return (
    <form action={action}>
      <input name="name" required />
      <button type="submit" disabled={isPending}>
        {isPending ? 'Submitting…' : 'Submit'}
      </button>
      {message && <p>{message}</p>}
    </form>
  );
}
```

### Custom Hook with Cleanup
```tsx
import { useState, useEffect } from 'react';

function useWindowWidth(): number {
  const [width, setWidth] = useState(() => window.innerWidth);

  useEffect(() => {
    const handler = () => setWidth(window.innerWidth);
    window.addEventListener('resize', handler);
    return () => window.removeEventListener('resize', handler); // cleanup
  }, []);

  return width;
}
```

## Clean Code Rules — MANDATORY

Non-negotiable. Any violation = reject and refactor before submitting.

### Structure & Architecture

Use feature-based folder structure:

```
features/user/
├── components/        ← UI only
├── hooks/             ← logic only (use- prefix)
├── services/          ← API / external calls
├── utils/             ← pure helpers
├── types.ts           ← shared types
└── index.tsx          ← entry point
```

- No global dumping ground — every file has a home
- `services/` centralizes all API calls; never fetch ad-hoc inside components
- `utils/` = pure functions only, no React imports

### File Size — Hard Limit

- **Component file max 300 lines.** Approaching → extract immediately.
- **Hook file max 150 lines.** Extract sub-logic to smaller hooks.
- Extract sub-components to `components/` within same entity folder:
  ```
  pages/user/profile/index.tsx
  pages/user/profile/components/avatar-card.tsx
  pages/user/profile/components/edit-form.tsx
  ```
- Shared types used across 2+ files → extract to `types.ts`

### Components — Single Responsibility

One component = one concern. Split when a component:
- Fetches data AND renders UI AND handles form state
- Has more than one reason to change
- Has sections that could stand alone

Separate data-fetching containers from presentational components:
```tsx
// container — fetches
function UserProfileContainer({ id }: { id: string }) {
  const { data, isLoading, error } = useUser(id);
  if (isLoading) return <Skeleton />;
  if (error) return <ErrorState error={error} />;
  return <UserProfile user={data} />;
}

// presentational — pure UI
function UserProfile({ user }: { user: User }) {
  return <div>{user.name}</div>;
}
```

Use functional components only. No class components.

### State Management

| State type | Tool |
|-----------|------|
| Local simple | `useState` |
| Local complex | `useReducer` |
| Global / shared | Context or Zustand |
| Server state | TanStack Query |

- Never store derived state — compute it instead
- Prop drilling max **2 levels**. Beyond → Context or Zustand
- Max **5 props per component** — more = component doing too much

### useEffect — Discipline Required

One effect per concern. Never stack unrelated side effects:

```tsx
// WRONG
useEffect(() => {
  fetchUser();
  setupAnalytics();
  subscribeToSocket();
}, []);

// RIGHT — three effects, each with clear purpose + cleanup
useEffect(() => { fetchUser(); }, [id]);
useEffect(() => { setupAnalytics(); return () => teardown(); }, []);
useEffect(() => {
  const unsub = subscribeToSocket();
  return unsub;
}, []);
```

- If effect count exceeds 3 → extract all to a custom hook
- Always correct dependency array — never lie to React
- No side effects outside hooks

### Logic / JSX Separation

Zero business logic inside JSX return block:

```tsx
// WRONG
return (
  <div>
    {data.filter(x => x.active).sort((a, b) => b.date - a.date).map(item => (
      <Card key={item.id} title={item.name.slice(0, 50) + '...'} />
    ))}
  </div>
);

// RIGHT
const activeItems = useMemo(
  () => data.filter(x => x.active).sort((a, b) => b.date - a.date),
  [data]
);
const truncate = (s: string) => s.length > 50 ? `${s.slice(0, 50)}…` : s;

return (
  <div>
    {activeItems.map(item => (
      <Card key={item.id} title={truncate(item.name)} />
    ))}
  </div>
);
```

- Inline ternary = only simple show/hide
- Nested ternary = always extract to variable or early return
- Use early return over deeply nested conditionals

### Data Fetching

- All API calls live in `services/` — never inline fetch inside component
- Use TanStack Query for server state (loading, error, retry, cache)
- Never fetch same data in multiple components — centralize, share via query key
- Always handle: loading state, error state, empty state

### Naming Convention

| Type | Convention | Example |
|------|-----------|---------|
| Component | PascalCase | `UserCard`, `OrderTable` |
| Hook | camelCase + `use` prefix | `useAuth`, `useOrderList` |
| Service fn | camelCase | `getOrders`, `createUser` |
| Util fn | camelCase, descriptive | `formatCurrency`, `truncateText` |
| File/folder | kebab-case | `user-card.tsx`, `use-auth.ts` |

No abbreviations unless universally known (`id`, `url`, `api`).

### Performance

- `React.memo` — heavy components that receive stable props
- `useMemo` — expensive computations, stable references for children
- `useCallback` — functions passed to memoized children
- No premature optimization — profile first, optimize proven bottlenecks
- `key` = stable unique ID. Array index = never on dynamic lists

### Type Safety

- TypeScript strict mode always
- Define interfaces for all props and data shapes
- No `any` — use `unknown` + type guard if type is truly unknown
- Extract shared interfaces to `types.ts`

### Reusability

- Build base UI components (Button, Input, Modal, Card) without hardcoded domain logic
- Configure via props, not internal branches
- Domain-specific behavior = higher-level wrapper, not inside base component

### Anti-Patterns — Hard Block

These patterns = immediate refactor required:

| Anti-pattern | Fix |
|-------------|-----|
| God Component (>300 lines, handles everything) | Split by responsibility |
| Multiple unrelated `useEffect` in one block | Separate effects or extract hook |
| Prop drilling >2 levels | Context / Zustand |
| Fetch inside component body | Move to `services/` + TanStack Query |
| Logic in JSX return | Extract to variable / hook |
| `any` type | Proper type or `unknown` + guard |
| Derived state stored in `useState` | Compute from source |
| Inline styling (large) | CSS module / Tailwind class |
| Copy-pasted logic | Extract to util or custom hook |

### Quick Self-Audit

Before submitting, answer these. "Yes" = refactor first:

- [ ] File exceeds 300 lines?
- [ ] More than 3 `useEffect` in one component?
- [ ] Props passing through more than 2 levels?
- [ ] Business logic inside JSX return block?
- [ ] Fetching data directly inside component (no service/query)?
- [ ] Hard to understand in 30 seconds?
- [ ] Logic that could be a custom hook?
- [ ] Any `any` type used?
- [ ] Derived state stored unnecessarily?

### Constraints

#### MUST DO
- TypeScript strict mode
- Error boundaries for production components
- Stable unique `key` on list renders
- Cleanup return in every `useEffect` with subscriptions/timers
- Semantic HTML + ARIA attributes
- Memoize callbacks/objects passed to memoized children
- Suspense boundaries for async operations
- Feature-based folder structure
- All API calls in `services/`

#### MUST NOT DO
- Mutate state directly
- Array index as key for dynamic lists
- Functions created inline in JSX
- Missing `useEffect` cleanup
- Ignore React Strict Mode warnings
- Skip error boundaries in production
- File exceeding 300 lines
- Multiple unrelated concerns in one `useEffect`
- Prop drilling beyond 2 levels
- Logic in JSX return block
- `any` type
- God Components

## Output Templates

When implementing React features, provide:
1. Component file with TypeScript types
2. Test file if non-trivial logic
3. Brief explanation of key decisions

## Knowledge Reference

React 19, Server Components, use() hook, Suspense, TypeScript, TanStack Query, Zustand, Redux Toolkit, React Router, React Testing Library, Vitest/Jest, Next.js App Router, accessibility (WCAG)
