# Supported types at the BridgeJS boundary

Swift types and their JavaScript/TypeScript equivalents. For full detail and per-direction rules, see DocC in the checked-out JavaScriptKit repo: `Sources/JavaScriptKit/Documentation.docc/Articles/BridgeJS/Supported-Types.md` and the type-mapping sections in Generating-from-TypeScript.

| Swift type | JavaScript | TypeScript |
|------------|------------|------------|
| `Int`, `UInt`, `Double`, `Float` | number | `number` |
| `String` | string | `string` |
| `Bool` | boolean | `boolean` |
| `Void` | - | `void` |
| `[T]` | array | `T[]` |
| `[String: T]` | object | `Record<string, T>` |
| `Optional<T>` | `null` or `T` | `T \| null` |
| `JSUndefinedOr<T>` | `undefined` or `T` | `T \| undefined` |
| `JSObject` | object | `object` |
| `JSValue` | any | `any` |
| `JSTypedArray<T>` | TypedArray (by reference) | `Uint8Array`, `Float32Array`, … |

Unbridged or unsupported TypeScript types (e.g. `any`, some generics) are emitted as `JSValue` or `JSObject` in generated Swift.

## TypedArray mapping (reference semantics, no copy)

| Swift | TypeScript |
|-------|-----------|
| `JSTypedArray<UInt8>` / `JSUint8Array` | `Uint8Array` |
| `JSTypedArray<Int8>` / `JSInt8Array` | `Int8Array` |
| `JSTypedArray<Int32>` / `JSInt32Array` | `Int32Array` |
| `JSTypedArray<Float>` / `JSFloat32Array` | `Float32Array` |
| `JSTypedArray<Double>` / `JSFloat64Array` | `Float64Array` |

## `@JS` user types

| Swift | TypeScript | Semantics |
|-------|-----------|-----------|
| `@JS class` | interface (`new`) | reference (call `release()`) |
| `@JS struct` | plain object interface | copy (init exported as static `init`) |
| `@JS enum` (case) | `EnumValues` const + `EnumTag` union (or native `enum` with `.tsEnum`) | copy |
| `@JS enum` (raw value) | const/union or native `enum` keyed by raw value | copy |
| `@JS enum` (associated value) | discriminated union `{ tag; param0; … }` | copy |
| `@JS protocol` | interface | reference |
| Closure `(A) -> B` / `JSTypedClosure<(A) -> B>` | `(arg0: A) => B` | reference |
| `async` closure / func | `(args) => Promise<R>` | reference |

## Notes

- Nested optionals (`T??`) are not supported. Use `JSUndefinedOr<T>` for `undefined` semantics.
- Arrays support optional elements (`[T?]` → `(T \| null)[]`), optional arrays (`[T]?` → `T[] \| null`), and nesting (`[[T]]` → `T[][]`).
- Imported function signatures additionally accept optional `@JSClass`/`@JS struct` and case/associated-value enums as parameters.
