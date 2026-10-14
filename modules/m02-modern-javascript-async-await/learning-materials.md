# M02 Learning Materials — Modern JavaScript & Async/Await

> **Purpose.** A self-contained study guide for the concepts, language features, and asynchronous
> reasoning named in [`brief.md`](./brief.md). It teaches the material; it does **not** replace the
> lab. Work the lab in your own public repository and record what really happened in your own
> `write-up.md` (created from [`write-up-template.md`](./write-up-template.md)).
>
> **Audience.** Developers with foundational programming knowledge who are new to — or rusty on —
> modern JavaScript and asynchronous execution. No prior Node.js or async experience is assumed.
>
> **Scope note.** All code, console output, and timing numbers below are **illustrative teaching
> examples**, not records of a completed lab. Replace every example with your own real state before
> treating anything here as evidence.
>
> **Documentation vs. lab.** This guide is the *learning material* — a documentation deliverable. The
> **Async toolkit** (`sleep`, `retry`, `withTimeout`, `mapLimit`), its `node:assert` self-checks, your
> timing numbers, and your `fs` refactor are a **separate lab deliverable that you build and verify
> yourself**. Nothing here is a ready-to-submit solution; see the two checkpoint sets in
> [§24](#24-git-workflow-write-up-and-definition-of-done-self-review).
>
> **How to use this guide.** Each section follows **Understand → Try → Verify → Continue**. The guided
> path takes roughly **3.5–4.5 hours of reading and study**; the M02 module is allocated **10 hours**
> overall, and the remaining **~5.5–6.5 hours** are hands-on: building the toolkit, experimenting,
> verifying, and writing up. Reading alone does not complete the module.
>
> **Prerequisites.** Complete M01 first: a working toolchain, **Node.js 24 LTS through a version
> manager**, a **public GitHub repository**, and the branch → PR → merge habit. For this module you
> also need a place for your toolkit code — use the public training repo you created in M01.

## Contents

**Part I — Modern JavaScript essentials**

1. [Bindings, scope, and closures](#1-bindings-scope-and-closures)
2. [Functions, `this`, and parameters](#2-functions-this-and-parameters)
3. [Strings, destructuring, and modern operators](#3-strings-destructuring-and-modern-operators)
4. [Truthy/falsy and equality pitfalls](#4-truthyfalsy-and-equality-pitfalls)
5. [Arrays and iteration](#5-arrays-and-iteration)
6. [Grouping and collections](#6-grouping-and-collections)
7. [Classes and encapsulation](#7-classes-and-encapsulation)
8. [Errors and exceptions](#8-errors-and-exceptions)
9. [ES modules](#9-es-modules)

**Part II — Asynchronous JavaScript**

10. [The event loop: stack, tasks, and microtasks](#10-the-event-loop-stack-tasks-and-microtasks)
11. [Callbacks to Promises](#11-callbacks-to-promises)
12. [Async/await and asynchronous errors](#12-asyncawait-and-asynchronous-errors)
13. [Sequential versus parallel, and Promise combinators](#13-sequential-versus-parallel-and-promise-combinators)
14. [Newer Promise APIs and asynchronous iteration](#14-newer-promise-apis-and-asynchronous-iteration)
15. [Timeouts and cancellation](#15-timeouts-and-cancellation)
16. [Fetch and JSON](#16-fetch-and-json)
17. [Common asynchronous pitfalls](#17-common-asynchronous-pitfalls)
18. [Temporal — awareness only](#18-temporal--awareness-only)

**Part III — Lab preparation**

19. [Design considerations for the four async utilities](#19-design-considerations-for-the-four-async-utilities)
20. [Self-checking with `node:assert`](#20-self-checking-with-nodeassert)
21. [Sequential versus parallel JSONPlaceholder timing](#21-sequential-versus-parallel-jsonplaceholder-timing)
22. [Callback-style `node:fs` to Promises/async-await](#22-callback-style-nodefs-to-promisesasync-await)
23. [Predicting asynchronous output order](#23-predicting-asynchronous-output-order)
24. [Git workflow, write-up, and Definition-of-Done self-review](#24-git-workflow-write-up-and-definition-of-done-self-review)
25. [References](#25-references)

---

# Part I — Modern JavaScript essentials

## 1. Bindings, scope, and closures

_Estimated: 25–35 minutes._

**Understand.** JavaScript binds names to values with `const`, `let`, and the legacy `var`. What
separates them is not "reassignable or not" — it is **scope** and **when the binding becomes usable**.

- `const` — block-scoped, cannot be reassigned (the *binding* is fixed; the value it points to can
  still be mutated).
- `let` — block-scoped, reassignable.
- `var` — function-scoped, hoisted and initialised to `undefined`, and legal to use before its
  declaration line (which hides bugs).

A **block** is any `{ }` — an `if`, a `for`, or a bare block. A **closure** is a function that keeps a
live reference to the variables of the scope where it was created, even after that scope has
returned. Closures are why `let` fixes the classic loop bug.

### `var` vs `let` vs `const`

| | Scope | Reassignable | Hoisting behaviour | Use |
| --- | --- | --- | --- | --- |
| `var` | Function | Yes | Hoisted; initialised to `undefined` | Legacy — avoid in new code |
| `let` | Block | Yes | Hoisted but in the **temporal dead zone** until declared | Values that change |
| `const` | Block | No | Same TDZ as `let` | Default choice for values that don't rebind |

**Why the loop bug happens.** A `var` declaration has one shared function-scoped binding. Closures
created inside the loop all read that same binding, so by the time they run the loop has finished.
`let` creates a **fresh binding per iteration**, so each closure captures its own `i`.

**Try.** Run this as an ES module (`node example.mjs`):

```javascript
const fromVar = [];
for (var i = 0; i < 3; i++) fromVar.push(() => i);
console.log("var loop:", fromVar.map((f) => f()).join(","));

const fromLet = [];
for (let j = 0; j < 3; j++) fromLet.push(() => j);
console.log("let loop:", fromLet.map((f) => f()).join(","));
```

**Verify.** The `var` line prints `3,3,3` (shared binding, already finished). The `let` line prints
`0,1,2` (one binding per iteration). If your output differs, confirm you saved the file as `.mjs` or
your project is `"type": "module"`.

**The temporal dead zone.** `let`/`const` bindings exist from the top of their block but cannot be
touched until the declaration executes. Accessing them earlier throws a `ReferenceError`, not
`undefined` — a deliberate guard against use-before-init.

```javascript
try {
  console.log(x); // TDZ: x exists but is not initialised yet
  let x = 1;
} catch (e) {
  console.log("TDZ:", e.constructor.name); // ReferenceError
}
```

**Continue.** Closures appear again in [§11](#11-callbacks-to-promises) (callbacks capture state) and
[§19](#19-design-considerations-for-the-four-async-utilities) (`retry` captures attempt state). Prefer
`const`; reach for `let` only when you genuinely reassign.

---

## 2. Functions, `this`, and parameters

_Estimated: 25–30 minutes._

**Understand.** Functions are values in JavaScript, and `this` is **not** where a function is
defined — it is **how it is called**. Arrow functions are the exception: they do not get their own
`this`; they inherit it lexically from the surrounding scope. That single rule explains most
`this`-related confusion.

| Call form | `this` inside a regular function |
| --- | --- |
| `obj.method()` | `obj` |
| `fn()` (plain call, module scope) | `undefined` in ES modules / strict mode |
| Arrow function | Whatever `this` was where it was **defined** |
| As a callback (`arr.map(fn)`) | Usually `undefined` — do not rely on it |

**Try.** Compare a method written as a regular function with one written as an arrow:

```javascript
const obj = {
  value: 42,
  regular() {
    return this?.value;
  },
  arrow: () => this?.value,
};
console.log("regular this:", obj.regular()); // 42
console.log("arrow this:", obj.arrow);       // undefined
```

**Verify.** `regular()` prints `42` because it is called as `obj.regular()`. The arrow inherits the
module's `this` (which is `undefined`), so `this?.value` is `undefined`. **A method on an object
should be a regular function, not an arrow.**

**Parameters: defaults, rest, and spread.** Default parameters apply only when an argument is
`undefined`. Rest (`...args`) collects remaining arguments into an array. Spread (`...iterable`)
expands an iterable into individual elements.

```javascript
const greet = (name = "world") => `Hello, ${name}`;
console.log(greet());        // Hello, world
console.log(greet("Ada"));   // Hello, Ada

const sum = (...nums) => nums.reduce((total, n) => total + n, 0);
console.log(sum(1, 2, 3));   // 6

console.log([0, ...[1, 2]]); // [0, 1, 2]
```

**Continue.** `this` rarely matters in the M02 toolkit (top-level functions), but it matters the
moment you pass a method as a callback — a common source of "`this` is undefined" bugs.

---

## 3. Strings, destructuring, and modern operators

_Estimated: 30–40 minutes._

**Understand.** Template literals, destructuring, optional chaining, and the nullish operators exist
to make it hard to write accidental `undefined` access or run error-prone manual extraction.

- **Template literals** — backtick strings with `${...}` interpolation.
- **Destructuring** — pull values out of objects/arrays by shape, with defaults.
- **Optional chaining `?.`** — short-circuit to `undefined` instead of throwing when a link is
  `null`/`undefined`.
- **Nullish coalescing `??`** — fall back only when the left side is `null` or `undefined`.
- **Nullish assignment `??=`** — assign only when the current value is `null`/`undefined`.

### `||` vs `??` — the trap

`||` treats **every falsy** value as "missing" (`0`, `""`, `false`, `NaN`). `??` treats only
`null`/`undefined` as missing. When `0` or `""` are valid, `||` silently replaces them.

| Expression | Result | Why |
| --- | --- | --- |
| `0 \|\| "fallback"` | `"fallback"` | `0` is falsy |
| `0 ?? "fallback"` | `0` | `0` is neither `null` nor `undefined` |
| `"" ?? "x"` | `""` | empty string is a real value |
| `null ?? "x"` | `"x"` | `null` is nullish |

**Try.**

```javascript
const { a = 1, b: { c = 2 } = {} } = { a: 10 };
console.log(a, c);                 // 10 2

const [first, , third = "z"] = [1, 2];
console.log(first, third);         // 1 z

const user = { profile: null };
console.log(user.profile?.name);   // undefined (no throw)

const cfg = { retries: null };
cfg.retries ??= 3;
console.log(cfg.retries);          // 3
```

**Verify.** Destructuring `{ a = 1, b: { c = 2 } = {} }` applies defaults only for missing values and
requires the inner default `= {}` so a missing `b` does not throw. `?.` returns `undefined` rather
than throwing. `??=` fills in only the nullish value.

**Continue.** `?.` and `??` are the safe way to read optional config and API payloads throughout the
lab. `structuredClone` and spread-based copying appear in [§5](#5-arrays-and-iteration).

---

## 4. Truthy/falsy and equality pitfalls

_Estimated: 20–25 minutes._

**Understand.** JavaScript has eight falsy values: `false`, `0`, `-0`, `0n`, `""`, `null`,
`undefined`, and `NaN`. **Everything else is truthy** — including `[]`, `{}`, and `"0"`. The loose
equality operator `==` performs type coercion, which produces famous surprises; `===` does not
coerce. `Object.is` is like `===` except it treats `NaN` as equal to itself and distinguishes `+0`
from `-0`.

| Expression | Result | Because |
| --- | --- | --- |
| `0 == "0"` | `true` | `==` coerces the string to a number |
| `0 === "0"` | `false` | different types, no coercion |
| `NaN === NaN` | `false` | `NaN` is not equal to anything |
| `Object.is(NaN, NaN)` | `true` | `Object.is` special-cases `NaN` |
| `[] == ![]` | `true` | coercion chain on both sides |
| `"" == false` | `true` | both coerce to `0` |
| `null == undefined` | `true` | special-cased by `==` |

**Try.**

```javascript
console.log([] == ![]);          // true
console.log(Object.is(NaN, NaN)); // true
console.log(NaN === NaN);         // false
```

**Verify.** `[] == ![]` is `true` only under loose equality — a good demonstration of why **`===` is
the default**. Use `Object.is` specifically when you must compare `NaN` or signed zero. `Array.prototype.includes`
uses the `SameValueZero` algorithm (like `Object.is` but treating `+0`/`-0` as equal), so
`[NaN].includes(NaN)` is `true`.

**Continue.** Prefer `===`. Behaviour-sensitive checks belong in your `node:assert` self-checks — see
[§20](#20-self-checking-with-nodeassert).

---

## 5. Arrays and iteration

_Estimated: 35–45 minutes._

**Understand.** Array methods fall into two families: **mutating** (`sort`, `reverse`, `splice`) and
**non-mutating** (everything below). Reach for the non-mutating modern helpers (`toSorted`,
`toReversed`, `with`) when you want a new array and the original left untouched.

- `map` — transform every element, returns a **new array of the same length**.
- `filter` — keep elements that pass a predicate.
- `reduce` — fold the array into one value.
- `find` — first matching element (or `undefined`).
- `some` / `every` — boolean tests, short-circuiting.
- `flatMap` — `map` then flatten one level.
- `toSorted` / `toReversed` / `with` — non-mutating `sort` / `reverse` / index-replace.
- `structuredClone` — deep copy of structured data (objects, arrays, `Map`, `Set`, dates).

### `map` vs `forEach`

| | Returns | Chainable | Use when |
| --- | --- | --- | --- |
| `map` | A new array | Yes | You need the transformed values |
| `forEach` | `undefined` | No | You only need a side effect |

**The classic bug:** using `forEach` to build an array and then reading the `undefined` return.

**Try.**

```javascript
const nums = [1, 2, 3];
console.log(nums.map((n) => n * 2));        // [2, 4, 6]
console.log(nums.forEach((n) => n * 2));    // undefined  <-- no return value

const arr = [3, 1, 2];
console.log(arr.toSorted());                // [1, 2, 3]
console.log(arr);                           // [3, 1, 2]  <-- unchanged
console.log(arr.with(1, 99));               // [3, 99, 2]

const original = { nested: { list: [1, 2] } };
const copy = structuredClone(original);
copy.nested.list.push(3);
console.log(original.nested.list);          // [1, 2]  <-- deep copy, untouched
```

**Verify.** `map` returns `[2,4,6]`; `forEach` returns `undefined`. `toSorted` and `with` leave `arr`
unchanged. `structuredClone` gives a deep copy, so mutating `copy` does not touch `original`.

**Continue.** `reduce`, `find`, `some`, and `every` are the workhorses of the `mapLimit` discussion in
[§19](#19-design-considerations-for-the-four-async-utilities).

---

## 6. Grouping and collections

_Estimated: 25–35 minutes._

**Understand.** Plain objects are strings-only keys; `Map` accepts **any** key type and remembers
insertion order. `Set` stores **unique** values and now has full set algebra. Grouping helpers turn
an array into a bucketed structure without a manual `reduce`.

- `Object.groupBy(items, fn)` — returns a null-prototype object of arrays.
- `Map.groupBy(items, fn)` — same, but keys may be non-strings.
- `Map` — `set`/`get`/`has`/`delete`/`size`; keys can be objects.
- `Set` — `add`/`has`/`delete`/`size`, plus `union`, `intersection`, `difference`,
  `symmetricDifference`, `isSubsetOf`, `isSupersetOf`, `isDisjointFrom`.

**Try.**

```javascript
const people = [
  { name: "Ana", dept: "eng" },
  { name: "Bo", dept: "ops" },
  { name: "Cy", dept: "eng" },
];
console.log(JSON.stringify(Object.groupBy(people, (p) => p.dept)));
// {"eng":[{"name":"Ana",...},{"name":"Cy",...}],"ops":[{"name":"Bo",...}]}

const a = new Set([1, 2, 3]);
const b = new Set([3, 4]);
console.log([...a.union(b)]);         // [1, 2, 3, 4]
console.log([...a.intersection(b)]);  // [3]
console.log(a.isDisjointFrom(b));     // false
```

**Verify.** `Object.groupBy` returns arrays keyed by the classifier. The Set methods return **new
sets** (they do not mutate `a` or `b`). Dedupe an array with `[...new Set(items)]`.

**Continue.** `Map` shows up in the `retry` design discussion when you want to track per-key state
across calls ([§19](#19-design-considerations-for-the-four-async-utilities)).

---

## 7. Classes and encapsulation

_Estimated: 25–35 minutes._

**Understand.** `class` is syntax over prototypes. The `#name` form declares a **true private** field
or method — inaccessible outside the class body (unlike the `_name` convention, which is advisory
only). Getters/setters let a property hide a computation or validation behind normal access syntax.

**Try.**

```javascript
class Counter {
  #count = 0;
  get value() {
    return this.#count;
  }
  set value(v) {
    if (v < 0) throw new RangeError("negative");
    this.#count = v;
  }
  increment() {
    this.#count++;
    return this; // enables chaining
  }
}

const c = new Counter();
c.increment().increment();
console.log(c.value);       // 2
try { c.value = -1; } catch (e) { console.log(e.constructor.name); } // RangeError
console.log(c.#count);      // SyntaxError: private field is not accessible
```

**Verify.** `c.value` prints `2`. The setter enforces the invariant. Accessing `c.#count` from
outside is a **syntax error**, not `undefined` — that is the point of `#private`. Returning `this`
from a method enables chaining.

**Continue.** Classes are relevant to M02 mainly as a way to package the toolkit, though a plain
module of exported functions is equally valid. Choose one and be consistent.

---

## 8. Errors and exceptions

_Estimated: 20–30 minutes._

**Understand.** `try` runs risky code; `catch` receives a thrown value; `finally` always runs
(cleanup). Throwing an `Error` (not a string) preserves a stack trace. Custom error **classes** let
callers branch on `instanceof`. The `cause` option chains a low-level error to a higher-level one
without losing the original.

```javascript
class ConfigError extends Error {
  constructor(message, options) {
    super(message, options); // forwards { cause }
    this.name = "ConfigError";
  }
}

try {
  try {
    JSON.parse("{bad");
  } catch (e) {
    throw new ConfigError("bad config", { cause: e });
  }
} catch (e) {
  console.log(e.name);              // ConfigError
  console.log(e.cause.constructor.name); // SyntaxError
}
```

**Try.** Run the snippet above.

**Verify.** The outer catch sees a `ConfigError` whose `.cause` is the original `SyntaxError`. This
pattern — wrap and preserve — is exactly how you should propagate failures in the toolkit: never
lose the original error.

**Rules of thumb.**
- Throw `Error` instances (or subclasses), never bare strings.
- Catch only what you can handle; otherwise rethrow.
- Use `finally` for cleanup (timers, files, connections) so it runs on both success and failure.

**Continue.** Error propagation is central to `retry` and `withTimeout` in
[§19](#19-design-considerations-for-the-four-async-utilities).

---

## 9. ES modules

_Estimated: 20–30 minutes._

**Understand.** This module uses **ES modules** (`import`/`export`), not CommonJS. There are two
export styles:

- **Named exports** — `export function foo() {}`; imported with braces: `import { foo } from "./x.mjs"`.
  Names must match.
- **Default export** — `export default function () {}`; imported without braces under any local name:
  `import anything from "./x.mjs"`. One per module.

Named exports are preferred for libraries: they are explicit, tree-shakeable, and rename-safe on
import. Use a default export only when a module genuinely has a single obvious thing.

**Try.** Create two files in the same directory and run the importing one:

```javascript
// toolkit.mjs
export const double = (n) => n * 2;
export function greet(name) {
  return `Hi, ${name}`;
}
export default greet;
```

```javascript
// main.mjs
import greetDefault, { double, greet as hello } from "./toolkit.mjs";

console.log(double(21));       // 42
console.log(hello("Ada"));     // Hi, Ada
console.log(greetDefault("Bo")); // Hi, Bo
```

**Verify.** The named imports resolve to `toolkit.mjs`'s named exports; the default import resolves
to `greet`. `import { greet as hello }` renames on import. If Node complains about module syntax,
either the file extension is `.mjs` or the nearest `package.json` has `"type": "module"`.

**Continue.** Decide now whether your toolkit uses named or default exports, and note the choice in
your write-up. Named exports are the recommended default.

---

# Part II — Asynchronous JavaScript

## 10. The event loop: stack, tasks, and microtasks

_Estimated: 35–45 minutes._

**Understand.** JavaScript runs your code on a single **call stack** — one frame at a time. When the
stack empties, the runtime's **event loop** pulls the next callback and runs it. Those callbacks live
in two queues with different priorities:

- **Task queue** (macrotasks) — `setTimeout`, `setInterval`, I/O callbacks.
- **Microtask queue** — Promise callbacks (`then`/`catch`/`finally`), `queueMicrotask`, and the
  resumption of `await`.

The rule that predicts almost every ordering question: **after the current synchronous code
finishes, the event loop drains the *entire* microtask queue before it runs the next task.**

```
call stack ──► (empty) ──► drain ALL microtasks ──► run ONE task ──► drain ALL microtasks ──► ...
```

**Tasks vs microtasks**

| | Task (macrotask) | Microtask |
| --- | --- | --- |
| Examples | `setTimeout`, `setInterval`, I/O | `.then()`, `.catch()`, `.finally()`, `await` resumption, `queueMicrotask` |
| Priority | Lower — one per loop turn | Higher — all drained before the next task |
| Starvation risk | Low | High — an endless microtask chain blocks tasks forever |

**Try — predict, then run.**

```javascript
console.log("A");
setTimeout(() => console.log("B"), 0);
Promise.resolve().then(() => console.log("C"));
console.log("D");
```

**Verify.** Output is `A`, `D`, `C`, `B`. Synchronous logs (`A`, `D`) run first; then all microtasks
(`C`); then the task (`B`). `setTimeout(…, 0)` does not mean "now" — it means "next task turn". For
this example the microtask-vs-task ordering is predictable, but the wall-clock moment a timer
callback actually fires can vary with the runtime and the event-loop workload.

**`await` is a microtask boundary.** Awaiting a non-promise (like `null`) still suspends the function
and resumes it via the microtask queue, so code after an `await` runs after the surrounding
synchronous code.

```javascript
async function main() {
  console.log("1");
  await null;      // suspend; resume as a microtask
  console.log("2");
}
console.log("3");
setTimeout(() => console.log("4"), 0);
main();
Promise.resolve().then(() => console.log("5"));
console.log("6");
```

**Verify.** Output is `3, 1, 6, 2, 5, 4`. Trace: sync prints `3`, `main()` prints `1` then suspends,
the `.then` is queued, `6` prints. Microtasks run in order: `main` resumes printing `2`, then the
`.then` prints `5`. Finally the task prints `4`. Being able to narrate this trace is a **Definition of
Done** skill; [§23](#23-predicting-asynchronous-output-order) drills it.

**Continue.** Every subsequent async section rests on this ordering rule.

---

## 11. Callbacks to Promises

_Estimated: 35–45 minutes._

**Understand.** A **callback** is a function passed to be run later. Nested callbacks that depend on
each other produce "callback hell" — deep indentation, duplicated error handling, and inverted
control flow. A **Promise** represents a future value in one of three states: *pending*, *fulfilled*,
or *rejected*, and it settles exactly once. Promises flatten nesting into a **chain**.

- `new Promise((resolve, reject) => { … })` — wrap an async operation.
- `.then(onFulfilled)` — run on success; returns a **new** promise.
- `.catch(onRejected)` — run on failure (also catches throws in a preceding `.then`).
- `.finally(onFinally)` — run on either outcome (cleanup); does not change the value.

**Try — the same task, three shapes.**

```javascript
// (1) callback style
function loadUserCallback(id, cb) {
  setTimeout(() => cb(null, { id, name: "Ada" }), 10);
}
loadUserCallback(1, (err, user) => {
  if (err) return console.error(err);
  console.log("callback:", user.name);
});

// (2) promise style
function loadUser(id) {
  return new Promise((resolve, reject) => {
    setTimeout(() => resolve({ id, name: "Ada" }), 10);
  });
}
loadUser(1)
  .then((user) => user.name)
  .then((name) => console.log("promise:", name))
  .catch((err) => console.error(err))
  .finally(() => console.log("promise: done"));
```

**Verify.** Both print the same name; the promise version is flat and has one error path, and
`finally` runs regardless of outcome. Note that a `.then` return value **becomes** the next `.then`'s
input — that is the chaining mechanism.

**Why chains flatten nesting.** Each `.then` returns a new promise, so dependent steps sit at the
same indentation level instead of nesting deeper. One `.catch` at the end handles a failure anywhere
above it.

**Continue.** `async`/`await` ([§12](#12-asyncawait-and-asynchronous-errors)) is the same machinery
with syntax that reads top-to-bottom.

---

## 12. Async/await and asynchronous errors

_Estimated: 30–40 minutes._

**Understand.** `async` functions always return a Promise; `await` pauses the function until a promise
settles, then yields its value (or throws its rejection). This makes async code read like synchronous
code while behaving asynchronously. **A rejection becomes a thrown exception at the `await` point**,
so ordinary `try/catch` works — but only if you actually `await` inside the `try`.

**Try.**

```javascript
async function loadUser(id) {
  if (id <= 0) throw new Error("invalid id");
  return { id, name: "Ada" };
}

async function main() {
  try {
    const user = await loadUser(1);
    console.log("ok:", user.name);
    await loadUser(-1); // rejects -> throws here
  } catch (err) {
    console.log("caught:", err.message);
  } finally {
    console.log("cleanup runs");
  }
}
main();
```

**Verify.** Output is `ok: Ada`, `caught: invalid id`, `cleanup runs`. The rejection surfaced as a
`throw` at the `await`, so the surrounding `try/catch` handled it.

**Key rules.**
- An `async` function that throws returns a **rejected** promise, not a synchronous throw.
- Calling an `async` function **without `await`** returns a promise you may ignore — and any
  rejection becomes an unhandled rejection ([§17](#17-common-asynchronous-pitfalls)).
- `try/catch` only catches `await`ed rejections; it does not catch a promise you forgot to await.

**Continue.** Now that `await` is understood, choose between doing work **one at a time** versus **all
at once** ([§13](#13-sequential-versus-parallel-and-promise-combinators)).

---

## 13. Sequential versus parallel, and Promise combinators

_Estimated: 40–50 minutes._

**Understand.** `await` inside a loop runs **sequentially** — each iteration waits for the previous
one. When the operations are independent, that wastes time. Starting them first and awaiting together
runs them **concurrently**. Crucially, concurrency here is about overlapping *waiting*, not parallel
CPU threads — JavaScript stays single-threaded.

- `Promise.all` — wait for **all** to fulfil; rejects fast on the first rejection.
- `Promise.allSettled` — wait for all to **settle**; never rejects; returns `{status, value|reason}`.
- `Promise.race` — settles with the **first to settle** (fulfil or reject).
- `Promise.any` — fulfils with the **first fulfilment**; rejects only if *all* reject
  (`AggregateError`).

### Combinator comparison

| Combinator | Resolves with | Rejects when | Typical use |
| --- | --- | --- | --- |
| `Promise.all` | Array of all values | Any one rejects (immediately) | Independent tasks where all must succeed |
| `Promise.allSettled` | Array of `{status,…}` | Never | You want every result, including failures |
| `Promise.race` | First to settle | If the first settle is a rejection | Timeout patterns, "first response wins" |
| `Promise.any` | First fulfilment | All reject | Redundant sources; need at least one success |

**Try — sequential vs parallel timing.**

```javascript
const tick = (ms, label) => new Promise((r) => setTimeout(() => r(label), ms));

async function sequential() {
  const start = Date.now();
  const out = [];
  for (const ms of [50, 50, 50]) out.push(await tick(ms, ms));
  return Date.now() - start;
}
async function parallel() {
  const start = Date.now();
  await Promise.all([tick(50, 1), tick(50, 2), tick(50, 3)]);
  return Date.now() - start;
}

console.log("seq ms:", await sequential());  // ~150 (3 x 50)
console.log("par ms:", await parallel());    // ~50  (overlapped)
```

**Verify.** Sequential timing is roughly the sum (~150 ms here); parallel is roughly the maximum
(~50 ms). Exact numbers vary by machine — that is fine; the *shape* is what matters.

**Try — combinators.**

```javascript
const ok = (v, ms) => new Promise((r) => setTimeout(() => r(v), ms));
const fail = (e, ms) => new Promise((_, rej) => setTimeout(() => rej(new Error(e)), ms));

console.log(await Promise.allSettled([ok("a", 10), fail("boom", 5)]).then(
  (rs) => rs.map((r) => r.status).join(","))); // fulfilled,rejected

console.log(await Promise.race([ok("fast", 5), ok("slow", 20)])); // fast
console.log(await Promise.any([fail("bad", 5), ok("good", 10)])); // good
```

**Verify.** `allSettled` reports both outcomes; `race` returns the earlier settle; `any` skips the
rejection and returns the first fulfilment.

**Continue.** `Promise.all` is how `mapLimit` runs its workers, and `allSettled` is the safe choice
when partial failure is acceptable ([§19](#19-design-considerations-for-the-four-async-utilities)).

---

## 14. Newer Promise APIs and asynchronous iteration

_Estimated: 30–40 minutes._

**Understand.** Node 24 ships several ergonomic additions:

- `Promise.withResolvers()` — returns `{ promise, resolve, reject }`, removing the awkward
  "extract resolvers from the executor" pattern.
- `Promise.try(fn, ...args)` — runs `fn` and converts **both** its return value and any synchronous
  throw into a promise outcome.
- `for await…of` — consume an async iterable sequentially.
- `Array.fromAsync(iterable)` — collect an async iterable into an array.

**Try.** (Verified on Node 24.)

```javascript
const { promise, resolve } = Promise.withResolvers();
resolve(42);
console.log(await promise);                 // 42

try {
  await Promise.try(() => { throw new Error("sync!"); });
} catch (e) {
  console.log("Promise.try caught:", e.message); // sync!
}

async function* gen() { yield 1; yield 2; }
const collected = [];
for await (const v of gen()) collected.push(v);
console.log(collected);                      // [1, 2]
console.log(await Array.fromAsync(gen()));   // [1, 2]
```

**Verify.** `withResolvers` resolves as expected. `Promise.try` catches a *synchronous* throw that a
plain `.then` would miss. `for await…of` and `Array.fromAsync` both consume the async generator
sequentially.

**Where `Promise.try` matters.** Combining synchronous and asynchronous steps in one chain without
wrapping the sync part in `Promise.resolve().then(...)` or a manual `try/catch`.

**Continue.** Cancellation and timeouts are next — including the subtle difference between rejecting
and actually stopping work ([§15](#15-timeouts-and-cancellation)).

---

## 15. Timeouts and cancellation

_Estimated: 30–40 minutes._

**Understand.** There are two distinct ideas people conflate:

1. **A timeout** is a wrapper that **rejects** if an operation is too slow.
2. **Cancellation** is **stopping** the underlying operation.

Rejecting a timeout wrapper does **not** stop the underlying work — it may still run to completion
(and still have side effects). True cancellation requires the operation to **cooperate**, normally by
observing an `AbortSignal`. The platform provides `AbortController` (the source of a signal) and
`AbortSignal.timeout(ms)` (a signal that aborts itself after `ms`).

### Timeout rejection vs actual cancellation

| | Timeout wrapper only | Cancellation via `AbortSignal` |
| --- | --- | --- |
| Caller sees | Rejection | Rejection |
| Underlying operation | **Keeps running** | **Stops** (if it checks the signal) |
| Cleanup | You must `clearTimeout` | Signal fires `abort`; you clear timers |
| Requires cooperation? | No | Yes — the operation must observe the signal |

**Try — timeout without cancellation.**

```javascript
function timeoutOnly(promise, ms) {
  return new Promise((resolve, reject) => {
    const t = setTimeout(() => reject(new Error("timeout")), ms);
    promise.then(
      (v) => { clearTimeout(t); resolve(v); },
      (e) => { clearTimeout(t); reject(e); },
    );
  });
}

let finished = false;
const slow = new Promise((r) => setTimeout(() => { finished = true; r("slow"); }, 200));

try {
  await timeoutOnly(slow, 30);
} catch (e) {
  console.log("threw:", e.message); // timeout
}
await new Promise((r) => setTimeout(r, 250));
console.log("underlying finished anyway?", finished); // true
```

**Verify.** The wrapper rejects after 30 ms, yet the underlying timer still fires and sets
`finished = true` at 200 ms. **The timeout did not cancel anything.**

**Try — cooperative cancellation.**

```javascript
function cancellable(delayMs, signal) {
  return new Promise((resolve, reject) => {
    const t = setTimeout(() => resolve("done"), delayMs);
    signal.addEventListener("abort", () => {
      clearTimeout(t);            // clean up the timer
      reject(signal.reason);      // propagate the reason
    });
  });
}

const ctrl = new AbortController();
const work = cancellable(200, ctrl.signal);
setTimeout(() => ctrl.abort(new Error("aborted")), 30);
try {
  await work;
} catch (e) {
  console.log("cancelled:", e.message); // aborted
}
```

**Verify.** Because `cancellable` listens for `abort`, the timer is cleared and the promise rejects
with `signal.reason` — genuine cancellation and cleanup.

**Cleanup rule.** Always `clearTimeout` when the race resolves, or you leak timers and may trigger
spurious rejections. `AbortSignal.timeout(ms)` is a convenient signal source for `fetch` and other
signal-aware APIs.

**Design note (not an official requirement).** Whether `withTimeout` should merely reject or should
also propagate a signal is a **design decision** for your toolkit — the brief does not specify it.
Whatever you choose, document the trade-off. This distinction is intentionally revisited in
[§19](#19-design-considerations-for-the-four-async-utilities).

**Continue.** Real network calls need the same care — see `fetch` and HTTP error handling in
[§16](#16-fetch-and-json).

---

## 16. Fetch and JSON

_Estimated: 25–35 minutes._

**Understand.** Node 24 provides a global `fetch` returning a Promise for a `Response`. A `Response`
is not an error for HTTP 4xx/5xx — **you must check `response.ok` (or `response.status`) yourself**.
Parse the body with `response.json()` (also async) or `response.text()`.

- `response.ok` — `true` for status 200–299.
- `response.status` — the numeric HTTP status.
- `response.json()` — parses the body as JSON (returns a promise).
- Node 24's `fetch` (Undici 7) is **standards-strict**: URLs must be absolute and valid, or `fetch`
  throws a `TypeError` before any network activity.

**Try.**

```javascript
async function getUser(id) {
  const res = await fetch(`https://jsonplaceholder.typicode.com/users/${id}`);
  if (!res.ok) {
    throw new Error(`HTTP ${res.status} for user ${id}`);
  }
  return res.json();
}

const user = await getUser(1);
console.log(user.name); // e.g. "Leanne Graham"

try {
  await fetch("/relative-url"); // throws: invalid URL in Node
} catch (e) {
  console.log("bad url:", e.constructor.name); // TypeError
}
```

**Verify.** `getUser(1)` resolves to a parsed object. A missing resource (e.g. `/users/9999`) returns
status 404, so the `!res.ok` guard throws instead of silently parsing an empty body. A relative URL
throws `TypeError` immediately.

**Always validate status.** A common async bug is awaiting `res.json()` without checking `res.ok`, so
an error page becomes confusing downstream data.

**Continue.** The failure modes here feed directly into [§17](#17-common-asynchronous-pitfalls).

---

## 17. Common asynchronous pitfalls

_Estimated: 35–45 minutes._

**Understand.** Five bugs account for most accidental async failures. Learn to recognise each.

### 17.1 Forgotten `await`

```javascript
async function getCount() {
  return Promise.resolve(41).then((n) => n + 1);
}
console.log(await getCount()); // 42 — correct because the caller awaits

async function buggy() {
  const promise = getCount();  // BUG: the returned Promise is never awaited
  return promise instanceof Promise ? "still a promise" : "a number";
}
console.log(await buggy()); // "still a promise"
```

A Promise was used where a resolved value was expected. If nothing awaits it and it rejects, it
becomes an **unhandled rejection**.

### 17.2 `await` inside loops (when you meant concurrency)

Sequential `await` in a loop is *correct* but slow when iterations are independent. See
[§13](#13-sequential-versus-parallel-and-promise-combinators) — use `Promise.all` for independent
work, or a bounded helper when you must cap concurrency.

### 17.3 `forEach(async …)` — the silent no-op

`forEach` ignores the promises its callback returns, so it neither waits nor propagates errors.

```javascript
const results = [];
[1, 2, 3].forEach(async (n) => {
  await new Promise((r) => setTimeout(r, 10));
  results.push(n);
});
console.log(results);              // []  <-- callback hasn't finished yet
await new Promise((r) => setTimeout(r, 50));
console.log(results);              // [1, 2, 3]  <-- too late to be useful
```

**Fix:** use `for…of` with `await`, or `await Promise.all(array.map(async …))`.

### 17.4 Unhandled rejections

A rejected promise with no `catch` (or no `await` inside a `try`) becomes an **unhandled rejection**.
Node's default is to **crash the process** with a non-zero exit code — this is intentional, so failures
are not silently lost.

```javascript
Promise.reject(new Error("no one is listening"));
// Node prints the error and exits non-zero by default.
```

### 17.5 Swallowed errors

An empty `catch` (or a `catch` that logs and returns `undefined`) hides the failure from callers.

```javascript
async function load() {
  try {
    await Promise.reject(new Error("db down"));
  } catch {
    // swallows the error; caller thinks load() succeeded
  }
}
```

**Fix:** rethrow, or wrap with context and rethrow: `throw new Error("load failed", { cause: err })`.

**Try.** Run the §17.3 snippet and observe the empty first log.

**Verify.** The first log is `[]`; the second is `[1,2,3]`. This proves `forEach` did not wait.

**Continue.** These five pitfalls are the ones the toolkit explicitly designs around
([§19](#19-design-considerations-for-the-four-async-utilities)).

---

## 18. Temporal — awareness only

_Estimated: 5–10 minutes._

**Understand.** `Temporal` is the proposed modern replacement for `Date`, offering immutable
date/time objects, explicit time zones, and clearer arithmetic. **It is not available in the Node 24
runtime used by this module** — on Node v24.19.0 `typeof Temporal` is `"undefined"`. You do **not**
need it for the lab.

**Try.** Confirm for yourself (read-only):

```javascript
console.log(typeof Temporal); // "undefined" on Node 24
```

**Verify.** The output is `"undefined"`, confirming it is not usable here. If you need dates in the
lab, use `Date` (and be aware of its quirks), or a library — but none is required.

**Continue.** No exercise follows. This section exists only so you know Temporal's role and why you
are not using it yet.

---

# Part III — Lab preparation

> **Boundary.** This part prepares you to build the **Async toolkit** yourself. It explains the
> reasoning, semantics, and edge cases behind each utility — it does **not** provide a ready-to-submit
> implementation. The toolkit functions, their self-checks, and your measurements are the **lab
> deliverable**, written by you, in your own public repository. Where `brief.md`/`tasks.md` leave
> behaviour unspecified, the guide marks it as a **design decision**, not a requirement.

## 19. Design considerations for the four async utilities

_Estimated: 45–60 minutes._

**Understand.** Each utility is small but has a deceptively large set of decisions hiding inside it.
The goal of this section is to make you *aware of the decisions* so your implementation is deliberate
and defensible. The shapes below are **pseudo-code for reasoning**, not drop-in code.

### 19.1 `sleep(ms)`

A `sleep` resolves after `ms`. The core technique was already introduced in
[§13](#13-sequential-versus-parallel-and-promise-combinators): a promise whose executor schedules a
timer that calls `resolve`.

**Design questions to answer for your implementation:**
- What does it resolve to — `undefined`, or something else? (Keep it simple.)
- Do you support cancellation (e.g., an optional `AbortSignal`)? If yes, you must `clearTimeout` on
  abort and reject with `signal.reason`.
- What happens for a negative or non-finite `ms`? (`setTimeout` treats non-positive as `0`; document
  your choice.)

### 19.2 `retry(fn, { retries, delayMs })`

```text
lastError = undefined
for attempt in 0 .. retries:
    try:
        return await fn()          # success -> return immediately
    catch err:
        lastError = err
        if attempt < retries:
            await sleep(delayMs)   # wait before the next try
throw lastError                    # all attempts failed
```

**Design decisions (unspecified in the brief — choose and document):**
- **Does `retries` mean additional attempts or total attempts?** The loop above treats it as
  *additional retries*, so total attempts = `retries + 1`. State this in your write-up.
- **Which errors are retryable?** Blindly retrying *all* errors can hammer a down service or repeat a
  permanent failure. Consider retrying only transient/5xx errors.
- **Fixed or backoff delay?** Constant `delayMs` is simplest; exponential backoff (with an upper
  bound) is kinder to servers. The brief only specifies `delayMs`.
- **Side-effect safety.** If `fn` is not idempotent, a retry can duplicate its effect (e.g., two
  charges). Retrying is safe only when `fn` is idempotent or the failure happened before any side
  effect — a genuine design concern, not just theory.
- **Error propagation.** After the final failure, rethrow the **last** error (preserve it). Do not
  swallow it.

### 19.3 `withTimeout(promise, ms)`

This is where [§15](#15-timeouts-and-cancellation) pays off.

**Design decisions:**
- **Timeout rejection vs cancellation.** Rejecting after `ms` does **not** stop the underlying work.
  If you want real cancellation, the operation must accept an `AbortSignal`; a wrapper cannot cancel
  a promise it did not create. Decide whether `withTimeout` merely guards the *caller's wait* or also
  passes a signal through.
- **What error on timeout?** A distinct error (e.g., a custom `TimeoutError`) lets callers branch on
  it; include the timeout duration in the message.
- **Timer cleanup.** You **must** `clearTimeout` in both the fulfilled and rejected paths, or you leak
  timers and risk spurious rejections. (Note this is a *design/cleanup* requirement even though the
  exact behaviour is unspecified.)
- **Interaction with the underlying rejection.** If the promise rejects before the timeout, propagate
  that original rejection unchanged — do not mask it with a timeout error.

```text
timer = setTimeout(() => reject(TimeoutError(ms)), ms)
promise.then(
  v => { clearTimeout(timer); resolve(v) },
  e => { clearTimeout(timer); reject(e) }   # preserve original error
)
```

### 19.4 `mapLimit(items, limit, fn)`

```text
results = new Array(items.length)
next = 0
worker():
    while true:
        i = next++                 # claim the next index (single-threaded: no race)
        if i >= items.length: break
        results[i] = await fn(items[i], i)
run `min(limit, items.length)` workers concurrently via Promise.all
return results
```

**Design decisions:**
- **Concurrency limit vs rate limit.** `limit` bounds how many calls are *in flight at once*; it is
  **not** a rate limiter (calls per second). The brief's `limit` is a concurrency cap. Note the
  distinction; do not over-build.
- **Output ordering.** The array-index assignment above keeps results in **input order** even though
  completion order varies. Decide whether order must be preserved (usually yes) and document it.
- **Partial failure.** If one `fn` call rejects, do you fail the whole batch (`Promise.all`
  semantics) or collect per-item outcomes (`Promise.allSettled`-style)? The brief does not specify —
  decide and justify.
- **`limit` edge cases.** Clamp `limit` to at least `1` and at most `items.length`; an empty `items`
  should resolve to `[]` without starting workers.

### 19.5 Record your decisions

Your write-up's "key decisions" section is where these choices belong. For each utility, note **the
behaviour you chose** and **why**, especially where the brief was silent. That is what turns a
correct implementation into a defensible one.

**Continue.** Now learn to prove each utility works — see `node:assert`
([§20](#20-self-checking-with-nodeassert)).

---

## 20. Self-checking with `node:assert`

_Estimated: 20–30 minutes._

**Understand.** With no test framework until M09, `node:assert` is your verification tool. Put the
checks in your toolkit module and run the file with `node`; a failing assertion throws and exits
non-zero.

Common assertions:

| Call | Passes when |
| --- | --- |
| `assert.ok(value)` | `value` is truthy |
| `assert.strictEqual(a, b)` | `a === b` |
| `assert.deepStrictEqual(a, b)` | Deep, type-aware equality (objects/arrays) |
| `assert.rejects(asyncFn)` | The promise rejects (`await` it) |
| `assert.throws(fn)` | The function throws synchronously |

**Try — an async self-check.**

```javascript
import assert from "node:assert/strict";

// A stand-in for one of your utilities — replace with your own implementation.
async function double(n) {
  return n * 2;
}

assert.strictEqual(await double(21), 42);        // value check
assert.deepStrictEqual([1, 2].map((n) => n * 2), [2, 4]);

await assert.rejects(
  async () => { throw new Error("boom"); },
  /boom/,
);

console.log("self-checks passed");
```

**Verify.** Running the file prints `self-checks passed`. Change an expected value to see the
assertion fail with a diff and a non-zero exit code.

**Practical tips.**
- Use `node:assert/strict` so `assert.equal` behaves like `strictEqual`.
- Always `await assert.rejects(...)` — otherwise the rejection escapes unhandled.
- Keep each utility's self-check **next to that utility** so failures point at the right code.
- Timing-based checks (e.g., that `withTimeout` rejects) should assert on *behaviour* (it rejected)
  with generous margins, not on exact milliseconds.

**Continue.** The timing comparison ([§21](#21-sequential-versus-parallel-jsonplaceholder-timing)) is
the other half of your evidence.

---

## 21. Sequential versus parallel JSONPlaceholder timing

_Estimated: 30–45 minutes._

**Understand.** The brief requires you to fetch from **JSONPlaceholder**
(`https://jsonplaceholder.typicode.com`) sequentially, then in parallel, and compare the timings. The
point is to *see* concurrency and explain *why* it wins. This section teaches a fair method — you
produce the real numbers.

**Why JSONPlaceholder.** It is free, needs no auth or API key, has generous and predictable rate
limits, and is widely used for exactly this exercise — so the timing difference comes from your code,
not from flaky third-party behaviour (`brief.md`, "Resolved decisions").

**A fair method.**

1. Pick a fixed list of independent resources (e.g., `/posts/1` … `/posts/8`).
2. Run the **sequential** version with `for…of` + `await`; time it with `performance.now()`.
3. Run the **parallel** version with `Promise.all`; time it the same way.
4. Repeat a few times and report a representative run — do not cherry-pick the best.
5. Capture the same request count and payloads in both versions so the comparison is apples-to-apples.

```javascript
const BASE = "https://jsonplaceholder.typicode.com";
const ids = [1, 2, 3, 4, 5, 6, 7, 8];

async function sequential() {
  const start = performance.now();
  const out = [];
  for (const id of ids) {
    const res = await fetch(`${BASE}/posts/${id}`);
    out.push(await res.json());
  }
  return { ms: performance.now() - start, count: out.length };
}

async function parallel() {
  const start = performance.now();
  const out = await Promise.all(
    ids.map(async (id) => {
      const res = await fetch(`${BASE}/posts/${id}`);
      return res.json();
    }),
  );
  return { ms: performance.now() - start, count: out.length };
}

console.log("sequential:", await sequential());
console.log("parallel:", await parallel());
```

**Verify.** Illustrative output (your real numbers will differ):

```text
sequential: { ms: 412.7, count: 8 }
parallel:   { ms: 96.4, count: 8 }
```

Parallel is roughly the **slowest single request**, sequential is roughly the **sum**. That shape is
the checkpoint.

**Why numbers vary (and what to do).** Network latency jitters; DNS/TLS warm-up affects the first
request; a small number of requests can make the gap look tiny. To make the difference clearly
visible and repeatable:
- Use **enough independent requests** (8–20) so the sum versus the max is unambiguous.
- Run each version **twice**, discarding the first (cold) run, and report a warm run.
- Optionally add a fixed artificial delay inside your fetch wrapper to model a slower API — but say so
  explicitly; do **not** fake the numbers.

**Do not fabricate.** The checkpoint requires the timing comparison to **actually show parallel
beating sequential**. Report what you measured. If a run does not show the expected shape, explain
why (e.g., too few requests, network hiccup) and retry the method — never invent a result.

**Continue.** The other lab step — refactoring callback `fs` code — is next
([§22](#22-callback-style-nodefs-to-promisesasync-await)).

---

## 22. Callback-style `node:fs` to Promises/async-await

_Estimated: 25–35 minutes._

**Understand.** The classic `node:fs` API uses **error-first callbacks**. `node:fs/promises` exposes
the same operations as promise-returning functions, which compose cleanly with `async`/`await`. The
refactor removes the `(err, data)` boilerplate and flattens nested callbacks.

**Callback style.**

```javascript
import { readFile, writeFile } from "node:fs";

writeFile("out.txt", "hello", "utf8", (err) => {
  if (err) return console.error(err);
  readFile("out.txt", "utf8", (err, data) => {
    if (err) return console.error(err);
    console.log(data); // hello
  });
});
```

**Promises / async-await style.**

```javascript
import { readFile, writeFile } from "node:fs/promises";

async function main() {
  await writeFile("out.txt", "hello", "utf8");
  const data = await readFile("out.txt", "utf8");
  console.log(data); // hello
}
main().catch((err) => console.error(err));
```

**Try.** Run the promises version (adjust the path as needed). This example is **illustrative** —
your lab deliverable is a refactor **you** write and self-check.

**Verify.** Both print `hello`; the async version has one error path instead of one per callback and
no nesting. Confirm you can explain *why* the callback version nests and the async version does not.

**Notes.**
- Import from `node:fs/promises` for the promise API; the callback `node:fs` API still exists.
- `util.promisify` can wrap callback APIs that have no promise variant — awareness only here.
- Handle the error: an unhandled rejection from `main()` crashes the process
  ([§17.4](#174-unhandled-rejections)).

**Continue.** Practise the checkpoint skill of reasoning about ordering before you build
([§23](#23-predicting-asynchronous-output-order)).

---

## 23. Predicting asynchronous output order

_Estimated: 20–30 minutes._

**Understand.** The Definition of Done requires that you can predict and explain, **out loud**, the
output order of a snippet mixing `setTimeout`, Promises, and `await`. Use the rule from
[§10](#10-the-event-loop-stack-tasks-and-microtasks): *synchronous first → all microtasks →
next task*.

**Method for any snippet.**
1. Write down the synchronous lines in order (everything not behind a callback/`await`).
2. List the microtask sources in the order they are queued (`.then`, `await` resumptions).
3. List the task sources (`setTimeout`).
4. Emit sync → microtasks (in queue order) → tasks (in timer order).

**Drill 1.**

```javascript
console.log("A");
setTimeout(() => console.log("B"), 0);
Promise.resolve().then(() => console.log("C"));
console.log("D");
```

<details><summary>Answer</summary>

`A, D, C, B`. Sync `A`, `D`; then microtask `C`; then task `B`.
</details>

**Drill 2.**

```javascript
async function main() {
  console.log("1");
  await null;
  console.log("2");
}
console.log("3");
setTimeout(() => console.log("4"), 0);
main();
Promise.resolve().then(() => console.log("5"));
console.log("6");
```

<details><summary>Answer</summary>

`3, 1, 6, 2, 5, 4`. Sync: `3`, then `main` prints `1` and suspends, `.then` queued, `6`. Microtasks:
`main` resumes (`2`), then `5`. Task: `4`.
</details>

**Drill 3.**

```javascript
setTimeout(() => console.log("timeout"), 0);
queueMicrotask(() => console.log("micro"));
console.log("sync");
```

<details><summary>Answer</summary>

`sync, micro, timeout`. Sync `sync`; then the microtask `micro`; then the task `timeout`.
</details>

**Verify.** For each drill, write your predicted order **before** running it, then run it and compare.
If you were wrong, re-trace using the four-step method. Being able to narrate the trace is the skill
the checkpoint tests — not memorising the answers.

**Continue.** Finish with workflow and the Definition-of-Done self-review
([§24](#24-git-workflow-write-up-and-definition-of-done-self-review)).

---

## 24. Git workflow, write-up, and Definition-of-Done self-review

_Estimated: 15–25 minutes._

**Understand.** M02 continues the M01 habit: work on a **feature branch**, open a **pull request**,
and merge — using **Conventional Commits** (`feat:`, `fix:`, `docs:`, `test:`, `refactor:`). Your
toolkit lives in your **own public training repo**, not in this training-materials repository. Link
the repo from the header of your write-up.

### 24.1 The two checkpoint sets

Keep these distinct so neither is mistaken for the other (see the documentation pattern referenced in
[§25](#25-references)):

| Documentation deliverable (this repo) | Lab checkpoint (your own repo / write-up) |
| --- | --- |
| `learning-materials.md` — this teaching guide | The Async toolkit: `sleep`, `retry`, `withTimeout`, `mapLimit` |
| Illustrative examples only | `node:assert` self-checks that **actually pass** |
| No claimed timings | Your **real** sequential-vs-parallel numbers |
| No claimed PR | Your real branch → PR → merge with Conventional Commits |

Illustrative examples in this guide are **not** evidence of lab completion. Only your own artifacts
are.

### 24.2 Definition-of-Done self-review

Before you call M02 done, check every box from `brief.md` against your own work:

- [ ] **All `node:assert` self-checks pass.** Run your toolkit file; confirm a clean run (exit code 0)
      and that each utility has its own self-check ([§20](#20-self-checking-with-nodeassert)).
- [ ] **You can predict and explain, out loud, the output order** of a snippet mixing `setTimeout`,
      promises, and `await` ([§10](#10-the-event-loop-stack-tasks-and-microtasks),
      [§23](#23-predicting-asynchronous-output-order)).
- [ ] **The timing comparison actually shows parallel beating sequential**, and you can explain why
      ([§13](#13-sequential-versus-parallel-and-promise-combinators),
      [§21](#21-sequential-versus-parallel-jsonplaceholder-timing)).

Also confirm the surrounding requirements:
- [ ] The toolkit is **one ES module** using `node:assert` for self-checks, in plain modern JS.
- [ ] You refactored a piece of callback-style `node:fs` code to Promises/async-await
      ([§22](#22-callback-style-nodefs-to-promisesasync-await)).
- [ ] No TypeScript and no external test framework were introduced.

### 24.3 Capturing evidence in your write-up

Map each heading of [`write-up-template.md`](./write-up-template.md) to what to record:

| Write-up heading | What to capture |
| --- | --- |
| What I built | Log each toolkit function **as you finish it**, not all at the end. |
| Why it's built this way | The design decisions from [§19.5](#195-record-your-decisions) — retry/`mapLimit` choices, sequential-vs-parallel reasoning, the pitfall you designed around. |
| How to build it (teach the next trainee) | A reasoned walkthrough of one utility — your own example, not just syntax. |
| Concepts worth explaining | 1–2 ideas in your own words (event loop/microtasks, closures, or combinator choice). |
| What tripped me up | Forgotten-`await`/race surprises as they happen. |
| Checkpoint evidence | Self-checks passing, your actual timing numbers, your `setTimeout`/promise/`await` order explanation. |
| What I'd do differently | Honest reflection if you restarted the module. |

**Verify.** Your write-up contains **real** outputs and numbers from your own runs — never the
illustrative values from this guide. Inspect `git log --oneline` to confirm your branch carries
Conventional Commits before opening the PR.

**Continue.** You are ready to complete the lab independently. Track the ordered checklist in
[`tasks.md`](./tasks.md) as you go.

---

## 25. References

- [MDN — JavaScript reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference) —
  the canonical reference for every language feature in Part I.
- [MDN — Concurrency model and the event loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Event_loop) —
  call stack, task queue, microtasks (§10).
- [MDN — Using promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises) —
  chaining and combinators (§11–§13).
- [MDN — `Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) —
  `withResolvers`, `try`, `allSettled`, `any`, `race` (§13–§14).
- [Node.js — 24.0.0 release notes](https://nodejs.org/en/blog/release/v24.0.0/) — V8 13.6 and the
  Node 24 feature set used in this module.
- [Node.js — `node:assert`](https://nodejs.org/api/assert.html) — the self-check API (§20).
- [Node.js — `node:fs/promises`](https://nodejs.org/api/fs.html#promises-api) — promise-based file
  system API (§22).
- [Node.js — `AbortController` / `AbortSignal`](https://nodejs.org/api/globals.html#class-abortcontroller) —
  cancellation and `AbortSignal.timeout` (§15).
- [Node.js — Global `fetch`](https://nodejs.org/api/globals.html#fetch) — the built-in HTTP client (§16).
- [JSONPlaceholder](https://jsonplaceholder.typicode.com) — the public API used for the timing
  comparison (§21).
- [Conventional Commits](https://www.conventionalcommits.org/) — the commit grammar for your PR (§24).
- [Temporal proposal](https://tc39.es/proposal-temporal/docs/) — awareness only (§18); not available
  in the Node 24 runtime.

For the module's deliverable and checkpoints, see [`brief.md`](./brief.md); for an ordered checklist,
see [`tasks.md`](./tasks.md); for the write-up shape, see
[`write-up-template.md`](./write-up-template.md).
