# Testing BridgeJS Projects

## Swift tests (XCTest / swift-testing)

Run Swift tests compiled to WebAssembly in a JavaScript environment (Node.js or browser):

```bash
swift package --disable-sandbox --swift-sdk $SWIFT_SDK_ID js test
```

- The test target must depend on your library/app target and on `JavaScriptEventLoopTestSupport` (from JavaScriptKit).
- The `--disable-sandbox` flag is required so the test runner can execute the JS runtime.
- Output is produced under `.build/plugins/PackageToJS/outputs/<PackageTests>/` (e.g. `PackageTests.wasm`).

Default is Node.js; use `--environment node` or see DocC for browser/Playwright. Full details: `Sources/JavaScriptKit/Documentation.docc/Articles/Testing.md` in the checked-out JavaScriptKit repo.

## JS-side tests (e.g. Vitest)

For end-to-end tests that call into your Swift exports from JavaScript, use the generated package (`instantiate`, `defaultNodeSetup`) and a test runner such as Vitest.

### Setup

```bash
mkdir -p tests && cd tests
npm init -y
npm install -D vitest typescript
```

**vitest.config.ts**:

```typescript
import { defineConfig } from "vitest/config";

export default defineConfig({
    test: {
        globals: true,
        environment: "node",
        include: ["**/*.test.ts"],
    },
});
```

**setup.ts**:

```typescript
import { instantiate } from "../.build/plugins/PackageToJS/outputs/Package/instantiate.js";
import { defaultNodeSetup } from "../.build/plugins/PackageToJS/outputs/Package/platforms/node.js";

export async function load() {
    const options = await defaultNodeSetup();
    return await instantiate({ ...options, getImports: () => ({}) });
}
```

## Example Test

```typescript
import { describe, expect, test } from "vitest";
import { load } from "./setup";

describe("MyApp", async () => {
    const { exports } = await load();

    test("greeter works", () => {
        const greeter = new exports.Greeter("World");
        expect(greeter.greet()).toBe("Hello, World!");
        greeter.name = "Swift";
        expect(greeter.greet()).toBe("Hello, Swift!");
        greeter.release();
    });

    test("enums work", () => {
        expect(exports.Direction.North).toBe(0);
        exports.setDirection(exports.Direction.South);
    });

    test("optionals work", () => {
        expect(exports.roundTripOptionalString(null)).toBeNull();
        expect(exports.roundTripOptionalString("test")).toBe("test");
    });

    test("structs are copied", () => {
        const point = exports.Point.init(1.0, 2.0, null);
        expect(point.x).toBe(1.0);
        expect(point.y).toBe(2.0);
    });
});
```

## Running Tests

```bash
# Build Swift first
swift package --swift-sdk $SWIFT_SDK_ID js

# Run tests
npx vitest run
```

