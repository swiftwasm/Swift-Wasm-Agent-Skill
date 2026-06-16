# Unsupported features and limitations

BridgeJS does not support every Swift or TypeScript pattern at the bridge. Design APIs within these limits.

For the full list and examples, see DocC in the checked-out JavaScriptKit repo: `Sources/JavaScriptKit/Documentation.docc/Articles/BridgeJS/Unsupported-Features.md`.

## Cross-module types (exporting)

You **cannot** use a non-`@JS` type from another target, or any type from a **separate Swift package**, in an **exported** API. Within the **same package**, a target's `@JS` types *can* be shared with other targets if the defining target has a `bridge-js.config.json` (even empty `{}`); see [project_setup.md](project_setup.md). BridgeJS otherwise generates glue per target and does not bridge types across package boundaries.

## Errors

- Only **`throws(JSException)`** is supported at the bridge. Plain `throws` (arbitrary Swift errors) is not supported for exported or imported callables.

## Imported bindings

- **Async**: Imported standalone `@JSFunction` cannot be `async` (no first-class Promise/async for the macro-generated import path). Async JS *callbacks* (closure parameters) are supported.
- **Generics**: No generic type parameters on bridged function signatures.

## Per-type limits

Each supported kind (functions, classes, structs, enums, closures, etc.) has specific limits (e.g. no async initializers, no subscripts on classes). Check the DocC Exporting-Swift-* and Importing-JS-* topics for the kind you use.
