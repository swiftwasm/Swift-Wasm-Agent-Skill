# Unsupported features and limitations

BridgeJS does not support every Swift or TypeScript pattern at the bridge. Design APIs within these limits.

For the full list and examples, see DocC in the checked-out JavaScriptKit repo: `Sources/JavaScriptKit/Documentation.docc/Articles/BridgeJS/Unsupported-Features.md`.

## Cross-module types (exporting)

You **cannot** use a type defined in one Swift target in an **exported** API of another target. For example, if module `Lib` defines `@JS struct LibPoint`, module `App` (which depends on `Lib`) must not export a function that takes or returns `LibPoint`. BridgeJS generates glue per target and does not bridge types across module boundaries.

## Errors

- Only **`throws(JSException)`** is supported at the bridge. Plain `throws` (arbitrary Swift errors) is not supported for exported or imported callables.

## Imported bindings

- **Async**: Imported JavaScript functions are not exposed as async in Swift; no first-class Promise/async support for the macro-generated import path.
- **Generics**: No generic type parameters on bridged function signatures.

## Per-type limits

Each supported kind (functions, classes, structs, enums, closures, etc.) has specific limits (e.g. no async initializers, no subscripts on classes). Check the DocC Exporting-Swift-* and Importing-JS-* topics for the kind you use.
