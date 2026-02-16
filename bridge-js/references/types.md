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

Unbridged or unsupported TypeScript types (e.g. `any`, some generics) are emitted as `JSValue` or `JSObject` in generated Swift.
