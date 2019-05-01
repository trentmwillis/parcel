### REPL Summary

It might look like much, but these are the important changes:

- augment the `src/visitors/process.js` and `src/visitors/process.js` to actually remove the dead branch when replacing `process.(browser|env)` (might become "core" with my suggestion in #2650)
- with the previous change, put `require`s for Watcher, (HMR)Server, WorkerFarm, ... in a conditional to not bundle them (aren't used anyway with the options used by the REPL)
- alias `fs` to a wrapper around `memory-fs` (makes most packages out of the box, though many try to be very clever if the browser is targeted and therefore break: mainly LESS, browserslist)
- don't bundle "native" Assets (Rust, Kotlin, ...) - might work but didn't test them yet
- some workarounds/monkeypatching for LESS, SASS, node-libs-browser
- in the JS packagers use `fs.readFileSync` directly because Parcel can't follow the code flow through `getExisting` to bundle the prelude snippets
- a browser implementation for Logger (only call console.\*)
- parse sourcemapppingURL only at end of file: #2427
- polyfill Buffer
- (merged scope hoisting destructuring: #2742 - but scope hoisting doesn't really work)
- REPL itself powered by preact@10.0.0-alpha.1

Building with scope hoisting or source maps results in an Out of Memory crash

### Scope Hoisting doesn't work / requiring node modules like `path` doesn't work

```
parcel/packages/core/parcel-bundler/src/builtins/index.js:
    node-libs-browser:
        require.resolve()
```

`require.resolve` should return module id that can be required?

### Peculiarities

Connecting with Chrome Devtools to Chrome on Android silently prevents the execution of Web Workers

### Other TODOs

```js
if(process.env.NODE_ENV === "development"){
  require("preact/debug");
}
if(module.hot) ...
if(process.browser) ...

parcel-bundler: (worker)
{
  "browser": {
    "./node.js": "./browser.js",
    "path": false
  }
}
```

#### core issues

- remove sourcemap from libs if --no-source-map (`preact`, `performance-now)
- Fix `existsCache` map for getConfig (parser.js) (file issue: not reset at runtime, not an issue with cli ???)
- non existing css linked by html: undefined line/column by css
- (scope hoisting: don't replace node builtins) -> #2796
- codecov
- `<link rel=preload>` for dynamic imports
- scope hoisting adds wrapper even without any import (with unreferenced css)

### REPL Issues

- onInput for options & header (debounce cache update)
- "Production" ? NODE_ENV ? As cli flag?
- sass illegal invocation: https://github.com/mbullington/node_preamble.dart/issues/14
- builtins/bundle loaders: fs.readFileSync(require.resolve)
- preview doesn't work with typescript asset
- Cmd+S -> export zip

#### Cache

- mtime
  - "polyfill"
  - use `memfs` for mtime (but: `require(...).emitWarning is not a function`)
- use cache (when enabled atm, Markdown isn't updating)
- with cache, watcher-like functionality (on state change with debouncing - debounce also hash change)

##### Performance

- SW for caching

##### Maybe/Longterm

- Preview using blob url in iframe
- Add a "expand" pull tab to options box
- Feedback that bundling was started/finished
- with Parcel 2: display graph
- Lazy load large asset types
-
- Parcel 2: test coverage