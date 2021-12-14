# esbuild-export-bug

There seems to be a bug in esbuild (or I'm just a dunce when it comes to JS) that causes imports/exports to break depending on the order of the exports. This repo reproduces the issue.

| Package    | Version |
|------------|---------|
| esbuild    | 0.14.3  |
| typescript | 4.5.4   |
| node       | 16.13.1 |

## Reproduction steps

First, create a new file named `run.ts` and add the following code to it:

```ts
import { startApp } from "./src";

startApp();
```

Next, create a folder named `src` and add add three files to it named `say.ts`, `app.ts` and `index.ts` with the following code:

```ts
// In say.ts

export function say(phrase: string) {
    console.log(phrase);
}
```

```ts
// In app.ts

import { say } from "./";

export function startApp() {
    say("hello world from app.ts")
}
```

```ts
// In index.ts

export * from "./app";
export * from "./say";
```

Now compile the four files with esbuild:

```sh
esbuild --outdir=esbuild-bin --platform=node --format=cjs run.ts src/*.ts
```

Finally, run the `run.js` file with node:

```sh
node esbuild-bin/run.js
```

If reproduced successfully, you'll get an error that looks something like this:

```
TypeError: (0 , import__.say) is not a function
    at startApp (/home/nozzlegear/repos/playgrounds/esbuild-bug-repro/esbuild-bin/src/broken/app.js:29:20)
    at Object.<anonymous> (/home/nozzlegear/repos/playgrounds/esbuild-bug-repro/esbuild-bin/run.js:24:28)
    at Module._compile (node:internal/modules/cjs/loader:1101:14)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1153:10)
    at Module.load (node:internal/modules/cjs/loader:981:32)
    at Function.Module._load (node:internal/modules/cjs/loader:822:12)
    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:81:12)
    at node:internal/main/run_main_module:17:47
```

## Workarounds

The bug seems to exist in the order of exports from `src/index.ts`. I've found several workarounds:

**Workaround 1:** Reorder the exports in `src/index.ts` to have `./say` exported before `./app`:

```
// In index.ts

export * from "./say";
export * from "./app";
```

**Workaround 2:** Export the `say` function explicitly:

```
// In index.ts

export * from "./app";
export { say } from "./say";
```

**Workaround 3:** Import the `say` function from its own file instead of from `index.ts`

```
// In app.ts
import { say } from "./say";

export function startApp() {
    say("hello world from app.ts")
}
```

**Workaround 4:** Don't use esbuild, just compile the code with typescript instead:

```sh
npx tsc && node typescript-bin/run.js
```

## Running the code

This repo contains a reproduction of the bug at `src/broken` and two of the workarounds at `src/works-2` and `src/works-3`. You can run the code in all three of these folders using the following commands in your terminal:

1. `npm install`
2. `npm run build`
3. `npm run start`

The two workarounds will log a message successfully, and the `src/broken` folder will throw an error complaining that `import__.say) is not a function`.
