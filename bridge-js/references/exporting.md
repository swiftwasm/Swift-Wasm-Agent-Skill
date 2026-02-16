# Exporting Swift to JavaScript

Expose Swift to JavaScript using the `@JS` (and `@JS(namespace:enumStyle:)`) macro. For full detail, see DocC in the checked-out JavaScriptKit repo: `Sources/JavaScriptKit/Documentation.docc/Articles/BridgeJS/Exporting-Swift-to-JavaScript.md` and the Exporting-Swift-* topics.

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

---

## Closures: use JSTypedClosure

Do **not** use plain Swift closure types (e.g. `(String) -> String`) for bridging Swift closures to JavaScript. Use **JSTypedClosure** for both passing and returning Swift closures to JS, and call `release()` when the closure is no longer needed by JS.

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

**JavaScript:** Call `release()` on any JSTypedClosure (or wrapper holding it) when done. See DocC: Bringing-Swift-Closures-to-JavaScript.

**Receiving closures from JS:** When Swift receives a callback from JavaScript (e.g. JS passes a function into an exported Swift function), use **regular** Swift closure types in the parameter (e.g. `@escaping (String) -> String`). No JSTypedClosure or `release()` on the Swift side for those parameters.

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