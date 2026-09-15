# Workspace

## Overview

pnpm workspace monorepo using TypeScript. Each package manages its own dependencies.

## Stack

- **Monorepo tool**: pnpm workspaces
- **Node.js version**: 24
- **Package manager**: pnpm
- **TypeScript version**: 5.9
- **API framework**: Express 5
- **Database**: PostgreSQL + Drizzle ORM
- **Validation**: Zod (`zod/v4`), `drizzle-zod`
- **API codegen**: Orval (from OpenAPI spec)
- **Build**: esbuild (CJS bundle)

## Key Commands

- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- `pnpm --filter @workspace/api-server run dev` — run API server locally

See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details.

## Artifacts

### Catering Manager (artifacts/catering-app)
- React + Vite SPA at preview path `/`
- Full catering management app with 4 sections: Ingredients, Dishes, Orders, Materials
- Uses React Query hooks from `@workspace/api-client-react`
- Design: warm orange palette, clean professional UI

## Database Schema (lib/db/src/schema/)

- **categories** — ingredient categories (id, name)
- **ingredients** — ingredient library (id, name, categoryId, unit)
- **dishes** — dish definitions (id, name, basePax)
- **dish_ingredients** — many-to-many dish-ingredient with qty
- **orders** — customer orders (id, firstName, lastName, eventDate)
- **order_dishes** — many-to-many order-dish with pax

## API Routes (artifacts/api-server/src/routes/)

- `/api/categories` — CRUD for categories (delete blocked if ingredients use it)
- `/api/ingredients` — CRUD for ingredients (delete blocked if used in dishes)
- `/api/dishes` — CRUD for dishes with nested ingredients
- `/api/orders` — CRUD for orders with nested dish+pax, sorted by eventDate
- `/api/materials` — POST to generate consolidated materials list (3 modes: single, multiple, dateRange)
  - Auto unit conversion: 1000g → kg, 1000ml → L
  - Grouped by category, sorted alphabetically

## Notes

- After running `pnpm --filter @workspace/api-spec run codegen`, manually set `lib/api-zod/src/index.ts` to only `export * from "./generated/api"` (codegen adds conflicting re-exports)
- Default categories seeded on first setup: Fruit, Vegetable, Meat, Provision, Miscellaneous
