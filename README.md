# Vite

## What makes it special

It's fast. Like, really fast.

- Server starts in < 300ms
- Hot Module Replacement (HMR) updates in < 100ms
- Esbuild-powered TS/JSX transforms
- On-demand: only process modules of current code-split route

## Vite can seperate into two major parts

1. dev server for development
2. Rollup-based build for production

### Dev Server

  Like webpack, browserify, rollup which crawl the whole dependency and build into one single bundle, and then serve it.
  you have to finish that whole process before you can display anything.
  that's why webpack dev server start time is long.

- Traditional "bundlers" require a full build of the entire application before the dev server can display anything.
  - This includes full-blown parsing of every single file for their import/export relationships!
  - Algorithms to sort, rewrite, and concatenate all the modules together

- The bigger your app gets, the slower your dev server starts up
- Code splitting helps prod performance, but does NOT help dev performance

#### Trandition based

1. eagerly work through the entire project from entry points
2. analysis the project and build bundles
3. after entire build process done then serve it

#### Native ES modules based 

- `<script type="module">`
- Browser natively parses and resolves ES modules imports
- The dev server receives HTTP requests for individual modules

1. open dev server, and wait for request
2. the request fetch the html page, which include your actual entry file
3. and then browser fetch the entry file and find dynamic imports

#### Problems: HTTP request waterfall

- Page loads slower vs. bundled when module count is large
- Prominent for dependencies with many internal modules, e.g. lodash-es 

- Optimization 1: pre-bundling dependencies
  - Ensure 1 file / 1 request per dep
  - Heuristics to determine bundled deps:
    - ESM entry w/ multiple imports
    - CommonJS

- Optimization 2: etag & 304 Not Modified headers
  Only the change file will refetch

- Optimization 3: code split
  require the user to actually do it, forces you to think about code splitting during development

  - Native ESM means code splitting improves both dev and prod performance

#### Problems: Native ESM doesn't support bare imports (yet)

- Lightweight module rewriting with es-module-lexer + magic-string
- No full AST parse / transforms, extremely fast (<1 ms for most files)

Source 
```javascript
import { createApp } from 'vue'
```

Rewritten
```javascript
import { createApp } from '/@modules/vue'
```

#### Challenge: HMR over native ESM

- Limitation: no actual way to "swap" a native ESM module
  1. Record module import graph while rewriting imports
  2. import.meta.hot usage marks file as "HMR boundary"
  3. When a file changes, we trace its importer chains to look for the HMR boundaries
  4. Boundaries re-import the changed module and apply updates
  5. If any of the parent chains reach a "dead end", a full reload will be triggered to ensure consistency

### Rollup-based build for production

- Rollup is the best-performing JS-based bundler in terms of build speed,
  tree-shaking and output size
- It's also built around ES modules, which aligns with Vite's premise

- Rollup itself is relatively less often used for bundling apps due to the difficulty of configuring it
  to handle app-specific use cases (e.g. css)
- Vite ships with an opinionated Rollup configuration that works out of the box for the most common cases

- A decent number of module transform logic can be reused
- Vite also provides an abstracted Transform API that works for both dev and prod

## Vite getting started

```bash
npm init vite-app
```

### Vite is also framework-agnostic

```bash
npm init vite-app --template react
```
