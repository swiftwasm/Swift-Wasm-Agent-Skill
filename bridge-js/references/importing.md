# Importing JavaScript into Swift (macros)

Declare JavaScript APIs in Swift with `@JSFunction`, `@JSClass`, `@JSGetter`, and `@JSSetter` so they are callable from Swift. Two ways to supply the implementation: **inject at initialization** via `getImports()`, or **read from globalThis** with `from: .global`.

For full detail, see DocC in the checked-out JavaScriptKit repo: `Sources/JavaScriptKit/Documentation.docc/Articles/BridgeJS/Importing-JavaScript-into-Swift.md`, and Importing-JS-Function, Importing-JS-Class, Importing-JS-Variable.

## Functions: `@JSFunction`

Declare the function in Swift with the same signature; use Swift types that bridge to the JS types (see Supported-Types in DocC).

```swift
@JSFunction func add(_ a: Double, _ b: Double) throws(JSException) -> Double
@JSFunction func setTitle(_ title: String) throws(JSException)
```

- **Inject**: Return the function in the object passed to `getImports()` when calling `init()`.
- **Global**: Add `from: .global` to bind a function on `globalThis` (e.g. `parseInt`); do not pass it in `getImports()`.

All bound functions are `throws(JSException)`; use `try` or `try?`.

## Classes / object shapes: `@JSClass`

Use a Swift struct with `@JSClass`. Mirror the JS class: `@JSFunction init(...)` for the constructor, `@JSFunction func` for methods, `@JSGetter` / `@JSSetter` for properties. Setters are Swift functions (e.g. `setMessage(_:)`) because Swift property setters cannot throw.

```swift
@JSClass struct Greeter {
    @JSFunction init(id: String, name: String) throws(JSException)
    @JSGetter var id: String
    @JSGetter var message: String
    @JSSetter func setMessage(_ newValue: String) throws(JSException)
    @JSFunction func greet() throws(JSException) -> String
}
```

- **Inject**: Pass the JS class (or factory) in `getImports()`.
- **Global**: Add `from: .global` on `@JSClass` for classes on `globalThis`; omit from `getImports()`.

## Globals / variables: `@JSGetter` and `@JSSetter`

Bind a global or any value you provide at init:

```swift
@JSGetter(from: .global) var document: Document
@JSGetter(from: .global) var console: JSConsole
@JSGetter(from: .global) var myConfig: String
@JSSetter(from: .global) func setMyConfig(_ newValue: String) throws(JSException)
```

- **Inject**: Omit `from: .global` and pass the value in `getImports()`.
- **Global**: Use `from: .global`; runtime reads from `globalThis`; do not pass in `getImports()`.

## Summary

| Macro | Use for | From global | From getImports() |
|-------|---------|------------|-------------------|
| `@JSFunction` | Functions | `from: .global` | Return function in object |
| `@JSClass` | Classes / object shapes | `from: .global` on struct | Return class/factory |
| `@JSGetter` / `@JSSetter` | Variables, properties | `from: .global` | Return value |

Use `jsName` when the Swift name differs from the JavaScript name (see API reference for `JSFunction(jsName:from:)`, etc.). Async functions are not supported for imported bindings.
