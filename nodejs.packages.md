# Guide to Creating Reusable Local Node.js Packages

This guide walks you through the process of creating and managing reusable Node.js packages designed for local use only (not published to npm), versioned via a private GitHub repository. You’ll learn how to set up a package, apply software design principles, structure your codebase, write tests, use TypeScript, bundle and distribute your code, and optionally set up CI/CD.

⸻

1. Initializing a Node.js Package

A Node.js package is a reusable module that can be imported into other projects. To create one, start by creating a project directory and running npm init:

```sh
mkdir my-package
cd my-package
npm init
```

This command generates a package.json file where you define properties such as:

```sh
{
  "name": "my-package",
  "version": "1.0.0",
  "main": "src/index.js",
  "scripts": {
    "test": "jest"
  },
  "dependencies": {},
  "devDependencies": {}
}
```

You can also use `npm init -y` to skip the prompts and use default values. If the package is private, you can add "private": true to the package.json.

⸻

2. Linking and Using the Package Locally

You can use a local package in other projects without publishing it to npm. There are a few options:

Option 1: npm link

This creates a global symlink to your package, which you can use in other projects:

```sh
cd ~/projects/my-lib
npm link

cd ~/projects/my-app
npm link my-lib
```

Alternatively:

```sh
npm link ../my-lib
```

Option 2: Local Paths or Git URLs

You can also use a local file path or a Git URL in your `package.json`:

```sh
"dependencies": {
  "my-lib": "file:../my-lib"
}
```

Or for a private GitHub repository:

```sh
"dependencies": {
  "my-lib": "git+ssh://[email protected]/user/my-lib.git#v1.2.3"
}
```

This fetches the package by tag or branch.

Option 3: GitHub Packages (Private npm Registry)

GitHub provides a private npm registry. You can publish your package under a scope (e.g. @your-org/package) and configure .npmrc with an access token.

⸻

3. Software Design Principles

Applying software engineering principles helps make your package clean, testable, and maintainable. Key principles:
	•	SRP (Single Responsibility Principle): Each module should do one thing. Split responsibilities across files.
	•	OCP (Open/Closed Principle): Your code should be open to extension but closed to modification. Use strategy patterns or config-driven behavior.
	•	DRY (Don’t Repeat Yourself): Extract reusable logic into helper modules.
	•	KISS (Keep It Simple, Stupid): Avoid over-engineering. Prefer straightforward solutions.
	•	YAGNI (You Ain’t Gonna Need It): Don’t add features until they’re required.
	•	Convention Over Configuration: Use predictable folder structures and file names.
	•	Composition Over Inheritance: Build features by combining modules rather than deep class hierarchies.
	•	Law of Demeter: A module should only interact with its direct dependencies.

These principles shape your code structure and promote clean, modular, and extendable design.

⸻

4. Recommended Folder Structure

Here’s a typical structure for a well-designed local package:

```sh
my-package/
├── src/           # Source code
├── dist/          # Compiled/bundled output
├── tests/         # Unit tests
├── types/         # TypeScript declaration files (if needed)
├── package.json
├── README.md
└── tsconfig.json  # TypeScript config
```

This organization encourages separation of concerns and makes testing and distribution cleaner.

⸻

5. Unit Testing with Jest

Jest is a widely used testing framework that works great with both JS and TS.

Setup:

```sh
npm install --save-dev jest ts-jest @types/jest
```

Create a basic `jest.config.js`:

```sh
module.exports = {
  preset: "ts-jest",
  testEnvironment: "node"
};
```

Add this to your `package.json`:

```sh
"scripts": {
  "test": "jest"
}
```

Create test files alongside your source code or in a tests/ folder:

```sh
// tests/math.test.ts
import { add } from '../src/math';

test('adds numbers', () => {
  expect(add(2, 3)).toBe(5);
});
```

Run tests with:

```sh
npm test
```

⸻

6. UI Testing (If Frontend Components Are Involved)

If your package contains UI components (e.g., React):
	•	Use React Testing Library + Jest for unit and integration tests.
	•	Use Cypress for end-to-end tests in a real browser environment.

Choose based on your needs:
	•	Unit tests check component behavior.
	•	E2E tests verify user flows.

⸻

7. Using TypeScript

TypeScript is highly recommended for reusable packages. It improves reliability and developer experience.

Install and Configure:

```sh
npm install --save-dev typescript
```

Create a `tsconfig.json`:

```sh
{
  "compilerOptions": {
    "target": "ES2019",
    "module": "ESNext",
    "declaration": true,
    "outDir": "dist",
    "strict": true
  },
  "include": ["src"]
}
```

Update your `package.json`:

```sh
{
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "files": ["dist"]
}
```

When you compile with tsc, TypeScript will output both the compiled JS and the .d.ts type declarations.

⸻

8. Bundling for Consumption

Use a bundler to produce clean output (CommonJS/ESM) and make the package ready for consumption.

Recommended: `Tsup`

Install:

```sh
npm install --save-dev tsup
```

Create `tsup.config.ts`:

```sh
import { defineConfig } from "tsup";

export default defineConfig({
  entry: ["src/index.ts"],
  format: ["cjs", "esm"],
  dts: true,
  clean: true
});
```

Run with:

```sh
npx tsup
```

Update `package.json`:

```sh
{
  "main": "dist/index.js",
  "module": "dist/index.mjs",
  "types": "dist/index.d.ts"
}
```

This gives consumers both CommonJS and ESM versions of your package.

⸻

9. Versioning and GitHub Distribution

Use Semantic Versioning: MAJOR.MINOR.PATCH.

Tag releases in Git:

```sh
git tag v1.2.0
git push origin v1.2.0
```

Consume the package via Git in other projects:

```json
"dependencies": {
  "my-lib": "git+ssh://[email protected]/my-org/my-lib.git#v1.2.0"
}
```

Or set up GitHub Packages as a private npm registry and use an .npmrc file for authentication.

⸻

10. Linting, Formatting, and CI/CD (Optional but Recommended)
	•	ESLint: Lint JavaScript/TypeScript code.
	•	Prettier: Enforce consistent formatting.
	•	Husky: Run pre-commit hooks for linting and tests.
	•	GitHub Actions: Set up CI to run lint and tests on every push/PR.

Example `.github/workflows/ci.yml` for Node.js:

name: CI

```sh
on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm install
      - run: npm run lint
      - run: npm test
```
⸻

Summary

By following this guide, you’ll build a reusable, well-structured, tested, and type-safe local Node.js package that follows modern development practices. Key takeaways:
	•	Use npm init to start.
	•	Organize files cleanly (src/, dist/, tests/).
	•	Apply SOLID, DRY, and other key principles.
	•	Use Jest for testing, TypeScript for types, and a bundler (e.g., Tsup) for distribution.
	•	Version your releases and link or install your package via Git.
	•	Optionally, use CI/CD and code quality tools.
