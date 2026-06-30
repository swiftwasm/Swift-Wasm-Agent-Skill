# Exporting Swift to JavaScript

Expose Swift to JavaScript using the `@JS` (and `@JS(namespace:enumStyle:identityMode:)`) macro. For full detail, see DocC in the checked-out JavaScriptKit repo: `Sources/JavaScriptKit/Documentation.docc/Articles/BridgeJS/Exporting-Swift-to-JavaScript.md` and the Exporting-Swift-* topics.

> JS/TS consumes exported `@JS` types through the **generated** TypeScript declarations (`index.d.ts`), which are the source of truth — never hand-author or mirror them. Note that `swift package plugin bridge-js` does **not** refresh `index.d.ts`; only the full `swift package … js` build does. See [Updating Generated Code](project_setup.md#updating-generated-code-aot).

---

## Functions

```swift
import JavaScriptKit

@JS public func calculateTotal(price: Double, quantity: Int) -> Double {
    return price * Double(quantity)
}

@JS public func findUser(id: Int) throws(JSException) -> String {
    if id <= 0 { throw JSException(JSError(message: "Invalid ID").jsValue) }
    return "User_\(id)"
}
```

**JavaScript:**

```javascript
const total = exports.calculateTotal(19.99, 3);

try {
  const name = exports.findUser(42);
} catch (e) {
  console.error("findUser failed:", e);
}
```

Only `throws(JSException)` is supported; `async` is supported (JS gets a Promise).

### Async

Exported `async` functions/methods/closures become Promise-returning JS functions. Call `JavaScriptEventLoop.installGlobalExecutor()` once at startup before invoking any `async` API.

```swift
@JS public func fetchCount(endpoint: String) async -> Int {
    try? await Task.sleep(nanoseconds: 50_000_000)
    return endpoint.count
}

@JS public func loadProfile(userId: Int) async throws(JSException) -> String {
    if userId <= 0 { throw JSException(JSError(message: "Bad userId").jsValue) }
    return "Profile_\(userId)"
}
```

```javascript
const count = await exports.fetchCount("/items");
```

---

## Default parameters

Default values become optional JS parameters; pass `undefined` to skip a middle one.

```swift
@JS public func greet(name: String = "World", enthusiastic: Bool = false) -> String { /* ... */ }
```

```javascript
exports.greet();                 // "Hello, World"
exports.greet("Bob", true);      // "Hello, Bob!"
exports.greet("Bob", undefined); // skip middle default
```

Supported defaults: string/int/float/double/bool literals, `nil`, enum cases (`.north`), no-arg or literal-arg class/struct inits, array literals. Not supported: method calls, closures, dictionary literals, binary ops, ternaries, complex member access.

---

## Classes

Mark the class and the members to expose. Call `release()` from JS when the instance is no longer needed.

```swift
import JavaScriptKit

@JS class Counter {
    private var count = 0

    @JS init() {}
    @JS init(start: Int) throws(JSException) {
        guard start >= 0 else { throw JSException(JSError(message: "Start must be positive").jsValue) }
        self.count = start
    }

    @JS func increment() { count += 1 }
    @JS func getValue() -> Int { return count }
}
```

**JavaScript:**

```javascript
const counter = new exports.Counter();
counter.increment();
console.log(counter.getValue()); // 1
counter.release(); // call when done
```

Pointer-based instance identity (so the same Swift object is the same JS object across crossings) is opt-in via `identityMode` — see [Configuration](project_setup.md#configuration-files).

---

## Structs

Value types; data is copied across the boundary. Often created via an exported `init`-style function or returned from Swift.

```swift
@JS struct Point {
    var x: Double
    var y: Double
    var label: String?
}

@JS func makePoint(x: Double, y: Double, label: String?) -> Point {
    return Point(x: x, y: y, label: label)
}
```

**JavaScript:**

```javascript
const point = exports.makePoint(10, 20, "origin");
console.log(point.x, point.y, point.label);
```

---

## Enums

Case enums and raw-value enums; use `@JS(enumStyle: .tsEnum)` for native TypeScript enums when applicable.

```swift
@JS enum Direction {
    case north, south, east, west
}

@JS enum Theme: String {
    case light = "light"
    case dark = "dark"
}

@JS func setDirection(_ d: Direction) { /* ... */ }
@JS func getDirection() -> Direction { /* ... */ }
```

**JavaScript:**

```javascript
exports.setDirection(exports.DirectionValues.North); // or DirectionValues.South, etc.
const dir = exports.getDirection();

exports.setTheme(exports.Theme.Dark); // string "dark"
```

### Associated values

Associated-value enums become TypeScript discriminated unions (`{ tag; param0; ... }`).

```swift
@JS enum APIResult {
    case success(String)
    case failure(Int)
    case status(Bool, Int, String)
    case info
}

@JS func handle(_ r: APIResult) { /* ... */ }
@JS func getResult() -> APIResult { /* ... */ }
```

```javascript
exports.handle({ tag: exports.APIResultValues.Tag.Success, param0: "ok" });
const r = exports.getResult();
switch (r.tag) {
  case exports.APIResultValues.Tag.Status: /* r.param0/param1/param2 typed */ break;
}
```

Associated values support primitives, `String`, custom classes/structs, other enums, `JSObject`, arrays, and optionals of these.

---

## Arrays

Arrays are copied across the boundary. Element types match what regular `@JS func` params allow: primitives, structs, classes, enums, protocols, `JSObject`, nested/optional arrays. `[T?]` → `(T | null)[]`, `[T]?` → `T[] | null`, `[[T]]` → `T[][]`.

```swift
@JS func processNumbers(_ values: [Int]) -> [Int] { values.map { $0 * 2 } }
```

### JSTypedArray

For native JS TypedArrays (e.g. `fetch` body, WebGPU), use `JSTypedArray<T>` — passed by **reference**, no copy:

```swift
@JS func processData(_ data: JSTypedArray<UInt8>) -> JSTypedArray<UInt8> { data }
@JS func processFloats(_ data: JSFloat32Array) -> JSFloat32Array { data }   // typealias
```

`JSTypedArray<UInt8>`/`JSUint8Array` → `Uint8Array`, `<Int8>` → `Int8Array`, `<Int32>` → `Int32Array`, `<Float>`/`JSFloat32Array` → `Float32Array`, `<Double>`/`JSFloat64Array` → `Float64Array`.

---

## Closures: use JSTypedClosure

When **passing or returning** a Swift closure to JS, prefer **JSTypedClosure** and call `release()` when the closure is no longer needed by JS. Plain Swift closure types (e.g. `(String) -> String`) also work as parameters/returns — the runtime releases them automatically via `FinalizationRegistry`, no `release()` needed. Closure signatures may be `throws(JSException)` and/or `async` in both directions.

```swift
// Return Swift closure to JS from an @JS method
@JS class Processor {
    let multiplier: JSTypedClosure<(Int) -> Int>
    init(factor: Int) {
        multiplier = JSTypedClosure<(Int) -> Int> { $0 * factor }
    }
    deinit { multiplier.release() }

    @JS func getMultiplier() -> JSTypedClosure<(Int) -> Int> {
        return multiplier
    }
    @JS func receiveClosure(_ body: (Int) -> Int) {
        print(body())
    }
}
```

`throws` and `async` signatures are also supported:

```swift
let parse = JSTypedClosure<(String) throws(JSException) -> Int> { Int($0) ?? 0 }
let fetchCount = JSTypedClosure<(String) async -> Int> { $0.count }   // JS sees (s) => Promise<number>
```

**JavaScript:** Call `release()` on any JSTypedClosure (or wrapper holding it) when done. See DocC: Bringing-Swift-Closures-to-JavaScript.

**Receiving closures from JS:** When Swift receives a callback from JavaScript (e.g. JS passes a function into an exported Swift function), use **regular** Swift closure types in the parameter (e.g. `@escaping (String) -> String`, optionally `async throws(JSException)`). Their lifetime is managed automatically.

---

## Namespaces

```swift
@JS(namespace: "MyApp.Utils") func formatDate(timestamp: Double) -> String { /* ... */ }

@JS(namespace: "MyApp.Models") class User {
    @JS var name: String
    @JS init(name: String) { self.name = name }
}
```

**JavaScript:**

```javascript
exports.MyApp.Utils.formatDate(Date.now());
const user = new exports.MyApp.Models.User("Alice");
user.release();
```

---

## DocC topics (JavaScriptKit repo)

When you need details, read DocC from the **checked-out JavaScriptKit repository**: `Sources/JavaScriptKit/Documentation.docc/` when inside the JavaScriptKit repo, or `.build/checkouts/JavaScriptKit/Sources/JavaScriptKit/Documentation.docc/` when in a project that depends on JavaScriptKit.