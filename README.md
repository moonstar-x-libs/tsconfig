# @moonstar-x/tsconfig

Maximally strict, deliberately pedantic TypeScript configurations, built for **TypeScript 7**.

Three configs: a `base` carrying every checking rule, plus `node` and `react` variants that extend it and add only what their runtime requires.

The premise: every rule here converts a class of runtime bug into a compile error. The configs are opinionated on purpose — if a rule only ever produced noise, it isn't in here.

## Install

```bash
npm install --save-dev @moonstar-x/tsconfig typescript
```

`typescript >= 6.0.0` is a required peer dependency (the configs use `target`/`lib` `es2025`, which TypeScript 6.0 was the first release to accept). The `node` variant additionally needs `@types/node`:

```bash
npm install --save-dev @types/node
```

## Usage

Extend the variant you need, then add your own paths. **This package intentionally sets no path options** (`outDir`, `rootDir`, `include`, `exclude`, `paths`, `baseUrl`).

### Node

```json
{
  "extends": "@moonstar-x/tsconfig/node",
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### React

```json
{
  "extends": "@moonstar-x/tsconfig/react",
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### Base only

For a runtime-agnostic package, or as a starting point for your own variant:

```json
{
  "extends": "@moonstar-x/tsconfig",
  "compilerOptions": { "outDir": "./dist", "rootDir": "./src" },
  "include": ["src/**/*"]
}
```

## Things that will bite you

These are intentional. Each is listed with its escape hatch.

- **`isolatedDeclarations` demands explicit types on exports.** `export const v = Math.max(1, 2)` fails with TS9010; write `export const v: number = …`. Trivially-inferrable initializers (`export const f = () => 1`) are still fine. React components need an explicit return type:

```tsx
export function Counter(props: CounterProps): JSX.Element { … }
```

Escape hatch: `"isolatedDeclarations": false`. This is the first flag to drop for an application (as opposed to a published library).

- **`erasableSyntaxOnly` bans `enum` and `namespace`.** Use a const object plus a union type:

```ts
export const Color = { Red: "red", Blue: "blue" } as const;
export type Color = (typeof Color)[keyof typeof Color];
```

- **`verbatimModuleSyntax` requires `import type`.** A type imported without it is a compile error (TS1484) rather than a silent elision.

- **`nodenext` requires runtime extensions in ESM.** In a `"type": "module"` package, write `import { x } from "./x.js"` even though the file is `x.ts`. That's Node's rule, not TypeScript's.

- **`skipLibCheck: false` checks your dependencies' types.** This finds real problems, but one broken `@types` package can block your build. Escape hatch: `"skipLibCheck": true`.

- **`exactOptionalPropertyTypes` rejects explicit `undefined`.** To allow it, declare it: `{ a?: string | undefined }`.

- **`noUncheckedIndexedAccess` makes every index access possibly-undefined.** That is the point. Narrow it, or use `.at()` and handle the `undefined`.
