You are simulating the canonicalization process of a Java test equivalence checker.
Your task is to determine whether two Java test method bodies are semantically
equivalent by computing their canonical forms and comparing them.

\---

## CANONICALIZATION RULES

### Rule 1 — Skip Diagnostic Statements

Silently ignore any statement that does not affect program state or assertion
outcomes, for example:

* `System.out.println(...)`, `System.err.printf(...)`
* `logger.info(...)`, `LOG.debug(...)`, `log.warn(...)`
* `logStart()`, `logStep(...)` (bare calls whose name starts with `log` + uppercase)
* `printResults(...)`

\---

### Rule 2 — Variable Handling

* **Numeric / boolean literal init** → inline immediately:
`int x = 3;` → inlineMap\[x] = `3`
* **`final String` constant** → store in constMap; render as `STR:<value>` at use sites
* **Simple expression, used ≤ 2 times** → inline:
`String s = obj.getName();` → inlineMap\[s] = `obj.getName()`
* **Otherwise** → assign stable token: varMap\[x] = `VARn`; emit `ASSIGN(VARn, <RHS>)`

At use sites: check inlineMap → constMap → varMap → render as-is.

\---

### Rule 3 — Method Call Normalization

* Format: `scope.methodName:\[arg1,arg2]` or `methodName:\[arg1,arg2]` (no scope)
* String concat folding: `STR:a + STR:b` → `STR:ab`
* For example, `Arrays.asList(a,b)` / `List.of(a,b)` → `LIST\[a,b]`

\---

### Rule 4 — Assertion Normalization

Apply after normExpr. For example:

* `assertEquals(a, b)` → sort args lexicographically → `assertEquals:\[min,max]`
* `assertEquals(null, x)` / `assertEquals(x, null)` → `assertNull:\[x]`
* `assertEquals(true, x)` / `assertEquals(x, true)` → `assertTrue:\[x]`
* `assertEquals(false, x)` / `assertEquals(x, false)` → `assertFalse:\[x]`
* `assertTrue(!x)` → `assertFalse:\[x]`
* `assertFalse(!x)` → `assertTrue:\[x]`
* `assertTrue(x != null)` / `assertTrue(null != x)` → `assertNotNull:\[x]`
* `assertNull("msg", x)` / `assertNotNull("msg", x)` → strip the leading String message
* `assertThat(x, is(equalTo(true)))` → `assertTrue:\[x]`

\---

### Rule 5 — Output Format per Statement

```
【Statement N】<original source line>
  Type        : <VariableDecl | Assignment | MethodCall>
  Diagnostic? : YES → skip, reason: ...  |  NO
  normExpr    :
    "<raw sub-expression>" → <lookup: inlineMap/constMap/varMap/as-is> → "<normalized>"
    ...
  Canonical   : <CALL(...) | ASSIGN(...)>
  State after :
    varMap    = {...}
    inlineMap = {...}
    result    = \[...]
```

\---

### Rule 6 — Final Comparison

```
---- CANONICAL 1 ----
\[CALL(...), CALL(...), ...]

---- CANONICAL 2 ----
\[CALL(...), CALL(...), ...]

---- Diff Analysis ----
Item : DIFFER
  1: CALL(...)
  2: CALL(...)

---- Verdict ----
isEquivalent → true | false
Reason: <one-sentence explanation>
```

\---

## INPUT

**Method 1:**

```java
<PASTE METHOD 1 HERE>
```

**Method 2:**

```java
<PASTE METHOD 2 HERE>
```

Now execute the canonicalization step by step following the rules above.

