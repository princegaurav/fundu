---
name: next-best-practices
description: Next.js best practices - file conventions, RSC boundaries, data patterns, async APIs, metadata, error handling, route handlers, image/font optimization, bundling
user-invocable: false
---

# Next.js Best Practices

Apply these rules when writing or reviewing Next.js code.

## File Conventions

- Project structure and special files
- Route segments (dynamic, catch-all, groups)
- Parallel and intercepting routes

## RSC Boundaries

Detect invalid React Server Component patterns:
- Async client component detection (invalid)
- Non-serializable props detection
- Server Action exceptions

## Async Patterns

Next.js 15+ async API changes:
- Async `params` and `searchParams`
- Async `cookies()` and `headers()`

## Runtime Selection

- Default to Node.js runtime
- Use Edge runtime only when needed (ultra-low latency, global distribution)

## Directives

- `'use client'`, `'use server'` (React)
- `'use cache'` (Next.js)

## Navigation Hooks

- `useRouter`, `usePathname`, `useSearchParams`, `useParams`

## Error Handling

- `error.tsx`, `global-error.tsx`, `not-found.tsx`
- `redirect`, `permanentRedirect`, `notFound`
- `forbidden`, `unauthorized` (auth errors)

## Data Patterns

- Server Components vs Server Actions vs Route Handlers
- Avoiding data waterfalls (`Promise.all`, Suspense, preload)
- Client component data fetching

## Route Handlers

- `route.ts` basics
- GET handler conflicts with `page.tsx`
- When to use vs Server Actions

## Metadata & OG Images

- Static and dynamic metadata
- `generateMetadata` function
- OG image generation with `next/og`

## Image Optimization

- Always use `next/image` over `<img>`
- Remote images configuration
- Responsive `sizes` attribute
- Priority loading for LCP

## Font Optimization

- `next/font` setup
- Google Fonts, local fonts
- Tailwind CSS integration

## Bundling

- Server-incompatible packages
- CSS imports (not link tags)
- ESM/CommonJS issues
- Bundle analysis

## Hydration Errors

- Common causes (browser APIs, dates, invalid HTML)
- Debugging with error overlay
- Fixes for each cause

## Suspense Boundaries

- CSR bailout with `useSearchParams` and `usePathname`
- Which hooks require Suspense boundaries

## Self-Hosting

- `output: 'standalone'` for Docker
- Cache handlers for multi-instance ISR
