# Generating bindings from TypeScript

Put a file named `bridge-js.d.ts` in your target source directory (e.g. `Sources/<target-name>/bridge-js.d.ts`). The BridgeJS plugin reads it and generates Swift code that uses the same macros as hand-written bindings (`@JSFunction`, `@JSClass`, `@JSGetter`, `@JSSetter`). Use this when you have existing TypeScript definitions or many APIs to bind.

For full detail and all declaration/type mappings, see DocC in the checked-out JavaScriptKit repo: `Sources/JavaScriptKit/Documentation.docc/Articles/BridgeJS/Generating-from-TypeScript.md`.

## Build

After adding `bridge-js.d.ts`, run:

```bash
swift package --swift-sdk $SWIFT_SDK_ID js
```

The plugin generates Swift (and, when using AOT, you run `swift package plugin bridge-js`). Generated macro-annotated Swift can be inspected under the build plugin outputs (e.g. `BridgeJS.Macros.swift` in the target’s generated output).

## Declaration mappings (summary)

| TypeScript | Swift |
|------------|-------|
| `export function f(...): T` | `@JSFunction func f(...) throws(JSException) -> T` |
| `export const x: T` | `@JSGetter var x: T` |
| `export class C { ... }` | `@JSClass struct C` with `@JSFunction init`, `@JSGetter`/`@JSSetter`, `@JSFunction func` |
| `export interface I { ... }` | `@JSClass struct I` (no constructor) |
| Object-shaped `export type T = { ... }` | Named struct with `@JSClass` / getters / setters |
| String enum | Swift enum with `String` raw value and BridgeJS protocol conformances |

All generated callable bindings throw `JSException`. Primitives map to Swift (`number` → `Double`, `string` → `String`, etc.); `T | null` → `Optional<T>`; `T | undefined` → `JSUndefinedOr<T>`; `Record<string, V>` → `[String: V]`; unbridged types → `JSValue`/`JSObject`. Invalid or keyword names get `jsName` in the generated Swift.

## Previewing d.ts → Swift locally (ts2swift)

To preview the Swift that would be generated from a `.d.ts` file without a full build, run the **ts2swift** script from the checked-out JavaScriptKit:

- **From a project that depends on JavaScriptKit** (e.g. after `swift package resolve`):
  ```bash
  node .build/checkouts/JavaScriptKit/Plugins/BridgeJS/Sources/TS2Swift/JavaScript/bin/ts2swift.js <path-to.d.ts> [options]
  ```
- **From inside the JavaScriptKit repo**:
  ```bash
  node Plugins/BridgeJS/Sources/TS2Swift/JavaScript/bin/ts2swift.js <path-to.d.ts> [options]
  ```

Run with `--help` for options (e.g. `-o` output path, `--global` to add global declaration files). Input can be a file path or `-` for stdin.

## Limitations (from DocC)

No first-class async/Promise-returning functions; no generic type parameters on bridged function signatures.
