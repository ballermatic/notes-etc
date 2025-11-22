# Coding Directives

## 1\. Core Principles

* P-1 (KISS): Prioritize readability and simplicity over cleverness. Code must be "succinct, not clever." If a solution is complex, add a comment explaining the why.

* P-2 (DRY): Adhere to DRY. Identify repeated logic. However, do not abstract prematurely; if a pattern is only used twice, duplication may be acceptable. Use three repetitions as the signal for abstraction.

* P-3 (YAGNI): Do not add functionality, abstractions, or configurations unless explicitly part of the request.

## 2\. TypeScript Directives

* TS-1 (Strict): All generated TypeScript must be compatible with "strict": true in tsconfig.json. Do not use any unless absolutely unavoidable and explicitly requested.

* TS-2 (Exports): Always use named exports for all modules (components, functions, types). Never use export default.

  * Correct: export const MyComponent \= () \=\> ...

  * Incorrect: export default MyComponent;

* TS-3 (Types vs. Interfaces): Use type for defining props, state, and data shapes. Use interface only if building a class or a library object intended for extension.

* TS-4 (Zod Integration): When data validation is required, use Zod.

  1. Define the Zod schema first.

  2. Infer the TypeScript type from the schema.

  3. This schema is the single source of truth for both validation and type definition.

  * Example: export type User \= z.infer\<typeof userSchema\>;

## 3\. File Structure & Imports

* FS-1 (Directory): Organize the project by feature (vertical slices), not by file type.

  * Correct: src/features/auth/LoginForm.tsx, src/features/auth/use-auth.ts

  * Incorrect: src/components/LoginForm.tsx, src/hooks/use-auth.ts

* FS-2 (Colocation): Components, hooks, functions, etc should be colocated with their feature or page.tsx file.

* FS-3 (UI Components): Truly reusable, "dumb" UI elements (Button, Input, Card) belong in src/components/ui/. These are most often shadcn or headless-ui based, but may be custom.

* FS-4 (Imports): Use path aliases for all imports (e.g., @/lib/, @/components/ui/Button). Never use relative paths like ../../../.

## 4\. Framework Directives (PayloadCMS 3.64+)

* F-1 (Next.js Native Architecture): Payload 3 runs inside your Next.js application.

  * Do not treat Payload as a separate external API when working within the same repository.

  * Pattern: In Server Components, Route Handlers, and Server Actions, always use the Local API (payload.find, payload.create) instead of fetch() calls to REST/GraphQL endpoints. This skips the network layer and significantly improves performance.

* F-2 (Local API Initialization):

  * Standard: Use getPayloadHMR (for HMR support during dev) or getPayload with the config promise to instantiate the local API.

  * Context: Pass the config explicitly to ensure type inference works correctly.

  * Example:  
    TypeScript

```ts
import { getPayloadHMR } from '@payloadcms/next/utilities'
import configPromise from '@payload-config'
const payload = await getPayloadHMR({ config: configPromise })
const data = await payload.find({ collection: 'posts' })
```

* F-3 (Generated Types):

  * Strict Adherence: You must generate types (npx payload generate:types) and rely on the output file (usually src/payload-types.ts).

  * Forbidden: Never manually type an interface that mirrors a Collection. Always import the generated Config or specific collection interfaces (e.g., Page, Post) from payload-types.ts.

  * Frontend Usage: Use Pick\<Page, 'title' | 'slug'\> if you need a subset of the generated type for a UI component.

* F-4 (Admin UI Customization):

  * Server by Default: Custom components injected into the Admin UI (e.g., specific field views) are Server Components by default.

  * Client Boundary: If your custom component needs interactivity (using hooks like useFormFields or useDocumentInfo), you must add 'use client' at the top of the file.

  * Imports: When importing UI elements for Admin views, you may import from @payloadcms/ui.

* F-5 (Hooks & Jobs):

  * Validation: Use beforeChange or beforeValidate hooks for business logic and data integrity.

  * Performance: Heavy operations (image processing, third-party syncs) that do not require an immediate response must be offloaded to the Payload Jobs Queue (payload.jobs.queue) rather than blocking the main thread in a hook.

  * Type Safety: Explicitly type hook arguments using the exported types from payload.

  * Example:  
    TypeScript

```ts
import type { CollectionBeforeChangeHook } from 'payload'
export const myHook: CollectionBeforeChangeHook = async ({ data, req }) => { ... }
```

* F-6 (Database & Direct Access):

  * Safety First: Prefer the Local API (payload.update) over direct database calls (payload.db.updateXXX) because the Local API ensures hooks and validations run.

  * Exception: Use direct DB access (payload.db) only for high-performance bulk operations where bypassing hooks is intentional and documented with a comment.

* F-7 (Config Organization):

  * Split Configs: Do not define all Collections and Globals in the root payload.config.ts.

  * Colocation: Define the Collection Config in the feature directory (e.g., src/features/blog/posts.collection.ts) and import it into the root config. This aligns with FS-1.

## 5\. Styling Directives (Tailwind CSS 4.1+)

* S-1 (No @apply): Do not use @apply in CSS files.

* S-2 (Abstraction): Encapsulate common class strings into reusable components (e.g., Button.tsx).

* S-3 (Variants): For components with multiple styles (variants, sizes), use class-variance-authority (cva) and the tailwind-merge \+ clsx (cn) utility function.

* S-4 (Sorting): All Tailwind class strings must be sorted by the official Prettier plugin standard.

## 6\. Forms & Validation Directives

* FV-1 (Stack): All forms must use React Hook Form (RHF).

* FV-2 (Validation): All form validation must use Zod via the @hookform/resolvers/zod package.

* FV-3 (Pattern): The implementation must follow this pattern:

  1. Define Zod schema.

  2. Infer TS type from schema.

  3. Pass the schema to zodResolver.

  4. Pass the resolver to useForm.

  5. Use the inferred TS type as the generic for useForm\<Type\>.

## 7\. Code Formatting & Linting

* ESLint: Use it to enforce coding rules (like no-console, import order, etc.).

* Prettier: Use it to format your code. This is non-negotiable. It ends all debates about style (quotes, spacing, commas).

  * Add the eslint-config-prettier plugin to make ESLint and Prettier work together.

  * Add the Tailwind CSS Prettier plugin (prettier-plugin-tailwindcss) to automatically sort your utility classes.

* husky \+ lint-staged: Set this up to automatically run ESLint and Prettier on your staged files before you're allowed to make a commit.

## T-1: General Testing Principles

* GT-1 (Testability): All generated business logic (hooks, utility functions) should be testable. Favor pure functions. For components, logic should be in hooks (useSomething) that can be tested in isolation.

* GT-2 (Colocation): All unit and integration test files must be colocated with their source file.

  * Example: MyComponent.tsx and MyComponent.test.tsx live in the same folder.

* GT-3 (Data Attributes): For E2E test selectors, add data-testid attributes to key interactive elements. Generated components should include these attributes on elements like buttons, inputs, and navigation links when appropriate.

## T-2: Unit & Integration Testing (Vitest \+ RTL)

* V-1 (Primary Tool): Vitest is the required test runner for all unit and integration tests.

* V-2 (Component Testing): All React component tests must use React Testing Library (RTL).

* V-3 (Querying): Tests must query the DOM using accessible roles and text, following RTL best practices (getByRole, getByText). Avoid querying by class names or data-testid unless no other accessible selector is available.

* V-4 (Mocking): Use vi.mock to mock modules and API requests. Avoid mocking component internals.

* V-5 (Syntax): Use the describe, it, expect BDD-style syntax.

* V-6 (Snapshots): Avoid snapshot tests for components, as they are brittle. Use them only for simple, stable data transformations if necessary.

## T-3: End-to-End Testing (Playwright)

* P-1 (Primary Tool): Playwright is the required tool for all E2E tests.

* P-2 (Selectors): Tests must prioritize selectors in this order:

  1. User-facing roles (e.g., getByRole('button', { name: 'Submit' })).

  2. data-testid attributes (e.g., getByTestId('user-profile-menu')).

  3. Avoid CSS classes or auto-generated IDs.

* P-3 (Test Structure): Use test.describe() to group related tests for a specific feature or page.

* P-4 (Page Object Model): For complex, repeated interactions (like authentication or site navigation), encapsulate logic in a Page Object Model (POM) class, but do not overuse it for simple tests.

* P-5 (Isolation): Each test() must be fully isolated and independent. Do not rely on state from a previous test. Use test.beforeEach to set up common state (e.g., logging in).

## M-1: Model Tool & Resource Utilization

* MTR-1 (Proactive Search Mandate): The model must use its search tool when a query implicitly or explicitly requires external information. This mandate is triggered by, but not limited to, the following conditions:

  * The query contains keywords like "latest," "current," "recent," "new," or references a specific recent version (e.g., "Next.js 15").

  * The query asks for a comparison between two or more external libraries, packages, or frameworks (e.g., "Vitest vs. Jest," "Drizzle vs. Prisma").

  * The query asks for documentation, installation guides, or specific API signatures for a public package.

  * The query references a person, company, or event that is not universally known.

* MTR-2 (Verification Over Confidence): Do not answer from internal training data if the topic is known to be fast-moving. The JavaScript/TypeScript, Next.js, and Vercel ecosystems are to be considered fast-moving. When in doubt, search to verify.

* MTR-3 (Ambiguity Resolution): If the user's query contains a term, acronym, or library name that is not in your core knowledge base (e.g., "Context Seven"), you must first use search to understand the term before formulating an answer.

* MTR-4 (No Nannying): Do not prompt the user "Would you like me to search for that?" Simply perform the search as per the MTR-1 mandate and provide the answer.