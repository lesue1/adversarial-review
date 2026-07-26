# Gotchas

## False positives (things that look like bugs but usually aren't)

### 1. Early return with no error handling
```js
function getConfig(key) {
  return config[key]; // No try-catch
}
```
**Looks like**: missing error handling.
**Usually fine when**: the caller has the try-catch, or config is guaranteed to exist by construction. Only flag if the crash path is reachable from user input.

### 2. `any` type in TypeScript
```ts
function parse(data: any) { ... }
```
**Looks like**: type safety violation.
**Usually fine when**: it's a library boundary (parsing unknown JSON), or inside a generic utility. Only flag if there's a safer alternative available.

### 3. Hardcoded values within the same module
```py
MAX_RETRIES = 3  # At module level
```
**Looks like**: magic number.
**Usually fine when**: it's a named constant at module level, not buried inline in logic. The name documents it.

### 4. Nested callbacks (2 levels)
```js
fetch(url).then(res => res.json()).then(data => process(data))
```
**Looks like**: callback hell.
**Usually fine when**: it's 2 levels max and the flow is linear. Only flag 3+ levels or branching chains.

### 5. SQL queries without parameterization
```py
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")
```
**Always flag this.** Even if `user_id` "comes from a trusted source." Trusted sources become untrusted.

---

## Stealth bugs (things that look fine but hide real problems)

### 1. `||` for default values with falsy inputs
```js
const count = userInput || 0; // Looks fine
```
`0` is falsy. If the user enters `0`, it becomes `0` anyway... but if `userInput` is the empty string `""`, it also becomes `0` — silently swallowing a potential input error. Use `??` instead:
```js
const count = userInput ?? 0;
```

### 2. Floating-point equality
```py
if amount == 0.3:  # Looks fine
```
`0.1 + 0.2 != 0.3` in floating point. Any `==` on floats is suspect. Use a tolerance or decimal type.

### 3. `try` with bare `except`
```py
try:
    do_stuff()
except:
    pass  # Looks like defensive coding
```
This swallows `KeyboardInterrupt`, `SystemExit`, and memory errors. Always specify the exception type.

### 4. Mutable default arguments
```py
def add_item(item, items=[]):  # Looks fine
    items.append(item)
    return items
```
The default `[]` is created once, shared across all calls. Classic Python gotcha — every call appends to the same list.

### 5. Non-atomic check-then-act
```js
if (fileExists(path)) {    // Check
  const content = readFile(path); // Act — file might be gone now
}
```
Between check and act, the file system can change. The read should handle the "not found" case anyway; the check is redundant and misleading.

### 6. `finally` block with `return`
```js
function getValue() {
  try {
    return computeValue();
  } finally {
    return "fallback"; // This overrides the try block's return!
  }
}
```
A `return` in `finally` silently overrides any return or throw in `try`/`catch`. Almost never intentional.
