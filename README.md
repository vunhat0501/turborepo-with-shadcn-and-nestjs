# Turborepo Monorepo Template: Next.js & NestJS

This template provides a robust monorepo foundation featuring a **[Next.js](https://nextjs.org/)** frontend, a **[NestJS](https://nestjs.com/)** backend, and a shared **[shadcn/ui](https://ui.shadcn.com/)** component library. It utilizes Turborepo for fast, incremental build pipelines.

- [Turborepo Monorepo Template: Next.js \& NestJS](#turborepo-monorepo-template-nextjs--nestjs)
  - [Few note before diving in](#few-note-before-diving-in)
  - [Creating monorepo with shadcn/ui build in](#creating-monorepo-with-shadcnui-build-in)
  - [🚀 Adding a NestJS App to the Monorepo](#-adding-a-nestjs-app-to-the-monorepo)
    - [Step 1: Scaffold the NestJS App](#step-1-scaffold-the-nestjs-app)
    - [Step 2: Configure Shared Packages](#step-2-configure-shared-packages)
    - [Step 3: Configuring Prettier](#step-3-configuring-prettier)
  - [🧪 Configuring Jest Testing](#-configuring-jest-testing)
    - [🚨 The Problem it solves](#-the-problem-it-solves)
    - [📁 File breakdown](#-file-breakdown)
  - [🧩 UI Components (shadcn/ui)](#-ui-components-shadcnui)
    - [Tailwind Configuration](#tailwind-configuration)
    - [Adding New Components](#adding-new-components)
    - [Using Components in your Apps](#using-components-in-your-apps)

## Few note before diving in

This README is almost all about how to set up a turborepo with nestjs and twitching utilities. If you only interested in using this project, you can completely skip this file.

The project use [pnpm](https://pnpm.io/) as package manager, Turborepo also recommend you to use pnpm when working with monorepo to save more space. Make sure to follow [installation guide](https://pnpm.io/installation) before process.

Also, remember to install nestjs CLI to your machine to save more time when working with it.

```bash
npm install -g @nestjs/cli
```

## Creating monorepo with shadcn/ui build in

The base project was created with shadcn/ui CLI. Find more detail [here](https://ui.shadcn.com/docs/monorepo).

## 🚀 Adding a NestJS App to the Monorepo

Follow these steps to generate and configure a new NestJS application within the workspace.

### Step 1: Scaffold the NestJS App

Run the NestJS CLI command from your `apps` directory:

```bash
nest new api --skip-git --package-manager pnpm
```

This command basically create a nestjs repo inside of apps directory without .git file since turborepo has already set up it own git.

### Step 2: Configure Shared Packages

Here is step by step how to link your new API to the monorepo's shared configuration standards. This is based on an example in [turborepo repository](https://github.com/vercel/turborepo/tree/main/examples/with-nestjs):

1. Add nestjs.json to the @workspace/typescript-config package. And extend from base config. Copy the rule from the current tsconfig.ts to the nestjs.json.

2. Similarly, add a nest.js configuration to the @workspace/eslint-config package and copy the rule form the old file to the new one.

3. In apps/api/package.json, add imports for both @workspace/typescript-config and @workspace/eslint-config.

4. Update your apps/api/tsconfig.json and eslint.config.mjs to extend these shared configurations.

5. Extra step: if you get and error with typescript unable to find those new file when importing, even though you have your path correct. Go to either tsconfig.json or tsconfig.build.json and add the name of the file to the include property.

### Step 3: Configuring Prettier

To ensure consistent code formatting across all applications and packages:

1. Update Shared Config: Add a prettier-base.js file to the @workspace/eslint-config package that import("prettier").Config.

2. Root Config: Add a .prettierrc.mjs file to the root workspace directory and set it to use the shared config.

3. App Config: Update apps/api/.prettierrc.mjs and apps/web/.prettierrc.mjs to point to the shared configuration.

4. Extra step: you will also want to add the apps/api/.prettier.mjs to the include property.

## 🧪 Configuring Jest Testing

This monorepo uses a centralized testing configuration strategy to keep frontend and backend test environments isolated but easy to maintain.

Setup Steps:

1. Create a [package.json](./packages/jest-config/package.json) with typescript, next, and jest a dev dependency. This will be the way you export your jest rule, check more in the package file.

2. Create src folder with [base.ts](./packages/jest-config/src/base.ts) file. Turns on code coverage (collectCoverage), tells Jest to use the faster V8 engine for coverage tracking, and sets the default test environment to a simulated browser (jsdom).

3. Create [nest.ts](./packages/jest-config/src/nest.ts) changes the environment from jsdom to node (since APIs don't run in browsers). Tell Jest to look inside the src folder for files ending in .spec.ts, and wires up ts-jest.

4. Create [next.ts](./packages/jest-config/src/next.ts), extend from base.ts and appends React-specific file extensions (jsx, tsx) to the base configuration.  
   _Note_: As you can see in [set up Jest with Next.js](https://nextjs.org/docs/app/guides/testing/jest) the Config variable has the same name in both jest and jest/types, so you will want to import the second as type and change the name a bit but don't have to use it.

5. Finally, entry.ts is the way to export the config you just create.

6. Update apps/api/package.json and apps/web/package.json to include the shared Jest package as a dependency.

7. Add a jest.config.ts file to apps/api and apps/web that import the shared config.

8. Build the shared package so the types and distributions are available:

```bash
pnpm build --filter @workspace/jest-config
```

### 🚨 The Problem it solves

- Next.js requires tests to run in a simulated browser (jsdom) and needs the SWC compiler for React components.

- NestJS requires tests to run in a pure Node environment (node) and relies heavily on ts-jest for TypeScript decorators.

- Solution: It enforces DRY (Don't Repeat Yourself) principles, strict typing (satisfies Config), and framework isolation.

### 📁 File breakdown

- base.ts (The Foundation): Defines the generic rules. Turns on code coverage, sets the V8 coverage provider, and establishes jsdom as the default environment.

- nest.ts (The Backend Config): Overwrites the base to use a node environment and wires up ts-jest to compile NestJS decorators properly.

- next.ts (The Frontend Config): Wraps the base config in next/jest, allowing Next.js to automatically handle path aliases, SWC compilation, and .env loading before tests run.

- package.json (The Traffic Cop): Uses the exports field to define strict access pathways, ensuring a Next.js app can only import the Next config without leaking backend logic.

## 🧩 UI Components (shadcn/ui)

This template is pre-configured with a shared UI package powered by Tailwind CSS and shadcn/ui.

### Tailwind Configuration

Your tailwind.config.ts and globals.css are already set up to scan and use components directly from the @workspace/ui package.

### Adding New Components

To add a new component (e.g., a Button) to your repository, run the following command from the root of your web app:

```Bash
pnpm dlx shadcn@latest add button -c apps/web
```

_Note:_ This will place the UI components securely into the packages/ui/src/components directory so they can be shared.

### Using Components in your Apps

Simply import them from the shared UI workspace package:

```TypeScript
import { Button } from '@workspace/ui/components/button';

export default function MyPage() {
return <Button>Click Me</Button>;
}
```
