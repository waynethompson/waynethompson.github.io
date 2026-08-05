---
layout: post
title:  "Choosing Your JavaScript Package Manager in 2026: npm vs. Yarn vs. pnpm vs. Bun vs. Lerna"
date:   2026-08-05 12:00:00 +1000
categories: [javascript, tooling]
tags: [javascript, typescript, npm, yarn, pnpm, bun, lerna, monorepo, nodejs]
permalink: /blog/2026/Choosing-Your-JavaScript-Package-Manager/
---

Every JS/TS project starts with the same boring question: `npm install`, `yarn`, `pnpm install`, or `bun install`? It seems trivial until you're six months into a monorepo with a bloated `node_modules` folder, a CI pipeline that takes twelve minutes to install dependencies, and a junior dev who just asked why there are three different lockfiles in the repo.

I went down this rabbit hole again recently while setting up a new monorepo, and it was a good excuse to properly compare where each tool actually stands in 2026 rather than relying on outdated opinions from a few years back.

<!--more-->

One thing worth calling out before we start: **Lerna isn't a like-for-like alternative to the other four**. npm, Yarn, pnpm, and Bun are package managers - they resolve, download, and link your dependencies. Lerna is a monorepo *orchestration* tool - it manages versioning, publishing, and task-running across packages, and it sits on top of one of the other four rather than replacing it. I've included it because "should we use Lerna" is a question that comes up in the same breath as "which package manager should we use," but it's answering a different problem.

---

## 1. npm

npm is the default that ships with Node.js, and it's come a long way since the days of notoriously slow, non-deterministic installs. Modern npm (v10+) has a solid lockfile, built-in workspaces support, and is fast enough for most projects.

```bash
npm install
npm install lodash --save
npm install --workspace=packages/api express
```

**The Pro:** Zero setup - it's already on every machine with Node installed. It has the largest community, the most Stack Overflow answers, and every CI/CD template and tutorial assumes it by default. `package-lock.json` is well understood by every tool in the ecosystem.

**The Con:** It still uses a flat, hoisted `node_modules` structure, which means "phantom dependencies" - your code can `import` a package you never declared, just because some other dependency pulled it in. Install times and disk usage are also noticeably worse than pnpm or Bun on larger projects, even with newer caching improvements.

**Best For:** Small to mid-sized projects, teams who want the path of least resistance, and anywhere you can't guarantee a second tool will be installed on every machine or CI runner.

---

## 2. Yarn

Yarn originally existed to fix npm's early reliability problems, and "Yarn Classic" (v1) is still what most people picture when they hear the name. But the project has moved on - "Yarn Berry" (v2+) is a genuinely different tool, built around Plug'n'Play (PnP), which skips `node_modules` entirely and resolves packages straight from a cache.

```bash
yarn add lodash
yarn workspaces foreach run build
```

**The Pro:** Yarn Berry's Plug'n'Play mode can make installs and cold starts extremely fast, since there's no `node_modules` tree to write to disk. Yarn also pioneered good workspaces UX, and "zero-installs" (committing the cache to the repo) is a compelling option for teams that want fully reproducible CI without a network round-trip.

**The Con:** The Classic-to-Berry migration fractured the community, and a lot of tooling and older tutorials still assume Yarn 1, which is unmaintained. PnP mode, while fast, isn't fully compatible with every package that does non-standard things with the filesystem, so some teams run Yarn Berry in "node-modules linker" mode anyway, which gives up most of the benefit.

**Best For:** Teams already invested in the Yarn ecosystem, or projects that want to commit to Plug'n'Play and are willing to deal with the occasional compatibility issue in exchange for install speed.

---

## 3. pnpm

pnpm has quietly become the default recommendation for most new JS/TS projects, and for good reason. It uses a single global content-addressable store on disk, and links packages into a `node_modules` structure using hard links and symlinks instead of copying files.

```bash
pnpm install
pnpm add lodash --filter api
pnpm -r run build
```

**The Pro:** Massive disk space savings across projects (a dependency is only ever stored once on disk, no matter how many projects use it), fast installs, and - critically - a *strict* `node_modules` structure that prevents phantom dependencies by default. Its monorepo/workspace support (`pnpm-workspace.yaml`, `--filter`) is widely regarded as the best of the bunch, which is why projects like Vue, Vite, and Prisma use it internally.

**The Con:** The strictness that makes it safer can also break packages that quietly rely on npm/Yarn's looser hoisting behaviour, usually surfacing as a confusing "module not found" error the first time you migrate an existing project. It's also still a separate install on top of Node, unlike Bun.

**Best For:** Almost any new project, but especially monorepos. If you're not sure which to pick and don't have a specific reason to choose otherwise, pnpm is the safest default in 2026.

---

## 4. Bun

Bun isn't just a package manager - it's an all-in-one JavaScript runtime, bundler, test runner, *and* package manager, written in Zig for speed. Its package manager is a drop-in-ish replacement for npm, reading `package.json` and producing a binary lockfile (`bun.lock` in text form since Bun 1.1+).

```bash
bun install
bun add lodash
bun run build
```

**The Pro:** Installs are dramatically faster than any of the other three, often by an order of magnitude on a warm cache, because Bun does dependency resolution and linking natively rather than through Node.js. If you also adopt the Bun runtime, you get native TypeScript execution, a built-in test runner, and a bundler, which removes several tools from your stack entirely.

**The Con:** Bun-the-runtime is still catching up on Node.js API and native addon (N-API) compatibility, so complex backend projects can hit edge cases in production. You can absolutely use `bun install` purely as a package manager while still running Node in production, which sidesteps most of that risk - but then you lose some of the appeal of going all-in on the ecosystem.

**Best For:** Greenfield projects, CLIs, scripts, and frontend tooling where install and iteration speed matters most. Increasingly viable for backend/Express-style projects too, but worth a compatibility check before committing for anything with heavy native dependencies.

---

## 5. Lerna

Lerna predates all the modern workspace tooling - it was originally built to manage versioning and publishing for multi-package repos (famously, Babel and Jest used it). It nearly died when Yarn/npm workspaces made its core use case redundant, but it was picked up and revived by the Nx team in 2022, and now happily delegates dependency installation to npm, Yarn, or pnpm workspaces while focusing on what it's actually good at: versioning, changelogs, publishing, and (via optional Nx integration) cached, parallelised task running.

```bash
npx lerna version --conventional-commits
npx lerna publish
npx lerna run build --parallel
```

**The Pro:** Best-in-class support for coordinated versioning and publishing across many packages - independent or fixed/locked versioning, automatic changelog generation, and safe, ordered `npm publish` across a whole package graph. With Nx integration enabled, you also get computation caching and task graph awareness, so `lerna run build` only rebuilds what actually changed.

**The Con:** It solves a specific problem - publishing multiple packages, typically to a public or internal registry - that most application monorepos simply don't have. If you're building one deployable app split across `apps/` and `libs/` rather than publishing a set of npm packages, Lerna is usually more tooling than you need; plain workspaces (or Nx/Turborepo directly) get you most of the task-running benefit without the extra layer.

**Best For:** Monorepos that publish multiple independently-versioned packages to npm - component libraries, SDKs, internal tool suites - not general-purpose application monorepos.

---

## The Comparison at a Glance

| Feature | npm | Yarn (Berry) | pnpm | Bun | Lerna |
|---|---|---|---|---|---|
| **Category** | Package manager | Package manager | Package manager | Runtime + package manager | Monorepo orchestrator |
| **Install speed** | Moderate | Fast (PnP) | Fast | Very fast | N/A (delegates to a PM) |
| **Disk efficiency** | Low (duplicated) | Moderate | Very high (content-addressable store) | Moderate | N/A |
| **Prevents phantom deps** | No | Yes (PnP) | Yes (strict by default) | Partially | N/A |
| **Workspaces/monorepo support** | Basic | Good | Excellent | Improving | Excellent (versioning/publishing) |
| **Ecosystem compatibility** | Highest | High (PnP has edge cases) | High (strictness can surface bugs) | Improving, some Node API gaps | Depends on underlying PM |
| **Ships built-in** | With Node.js | No | No | With Bun | No |
| **Best known for** | Ubiquity | Plug'n'Play | Speed + strictness + disk savings | All-in-one speed | Multi-package publishing |

---

## Picking a Tool for Your Use Case

- **Angular:** The Angular CLI defaults to npm and its schematics are tested most heavily against it, but pnpm works well and is a common choice for larger Angular monorepos (e.g. Nx-based workspaces, which Angular's own tooling has close ties to). Avoid Yarn PnP with Angular unless you've verified your specific Angular version and plugins play nicely with it - some codegen tooling still expects a real `node_modules` tree.

- **React (Vite/Next.js/CRA-successors):** Any of the four package managers work cleanly here since the tooling is largely package-manager agnostic. pnpm is a great default for a single app; Bun is worth trying if you want faster local dev loops and aren't relying on anything with native bindings.

- **Express.js / Node backend:** npm remains the safest choice if you're deploying to a locked-down environment (e.g. certain PaaS or container base images) where you can't guarantee Bun or pnpm are available. If you control your deployment pipeline, pnpm is a strong upgrade for install speed and dependency hygiene; Bun is compelling if you're comfortable running it as your production runtime too, not just for installs.

- **Monorepo with multiple published packages (component library, SDK, internal npm packages):** Use pnpm or Yarn workspaces for dependency management, and layer Lerna on top for versioning and publishing. This is Lerna's actual sweet spot.

- **Monorepo with a single deployable app (frontend + backend + shared libs, nothing published):** Use pnpm workspaces (or Yarn) with Nx or Turborepo for task orchestration and caching. Skip Lerna entirely - you don't have a publishing problem to solve.

---

## Final Verdict: Which Should You Use?

There isn't a single right answer, but there's a sensible default and a set of reasons to deviate from it:

- **Default to pnpm** for new projects. The disk savings, install speed, and strict dependency resolution catch real bugs before they ship, and its workspace support is the strongest of the bunch for monorepos.

- **Stick with npm** if you need maximum compatibility with constrained environments, are working on a small project where the extra speed doesn't matter, or you're onboarding people who've never touched anything else.

- **Consider Yarn Berry** only if you're specifically after Plug'n'Play and zero-installs, and you're willing to validate compatibility with your dependency tree first.

- **Reach for Bun** when install and iteration speed is the priority and you're not deep in Node-specific native dependencies - it's increasingly viable as your full runtime too, not just an installer.

- **Add Lerna** only when you're actually publishing multiple independently-versioned packages. Otherwise, use your package manager's native workspaces plus Nx or Turborepo for task running.

For my own projects, I've settled on **pnpm workspaces**, with **Nx** for task orchestration rather than Lerna, since I'm building applications rather than publishing package suites. I keep an eye on Bun for anything greenfield and speed-sensitive, but for anything going into a shared Angular/Express monorepo, pnpm's strictness has already saved me from a few "works on my machine" phantom dependency bugs that npm would have let slide.
