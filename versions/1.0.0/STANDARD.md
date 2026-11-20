# Code Design Standard (CDS)

**Version:** 1.0.0

A standard for code inside a file. Two parts: **Readability** — how a file reads top to bottom; **Skeleton** — in what order code is placed in a file. Project folder layout and import direction between files are out of scope.

Goal: to understand **what** a file and a function do, there is no need to open any other method or file; a method is opened only to learn **how** a detail works. The main metric is cognitive load; line count is secondary: 50 lines of linear code read more easily than 6 methods of 10 lines that cannot be understood without each other.

---

## Glossary

- **Process** — a complete unit of work implemented by a file: a client of an external system, a use case, an event handler.
- **File Header** — the opening doc comment: what Process the file implements and which numbered business steps it goes through — in plain words, without technical details.
- **Scenario** — a public operation written as a single function made of Paragraphs.
- **Paragraph** — a blank line, a heading comment with the step number from the Header, and the statements of the step.
- **Deep Method** — a method whose name, signature and "Step N: …" line describe it completely, while its implementation is much more complex than its interface.
- **Detail** — a method holding a technical minutia the Scenario reader does not need to look into.
- **Reading Depth** — how many levels of calls must be opened to understand what a Scenario does.
- **Flat Flow** — a function body without branching ladders and long-lived mutable variables.
- **Cognitive Load** — how many facts the reader holds in mind at once.
- **Cognitive Complexity** — a formal metric of cognitive load following the Cognitive Complexity model (SonarSource, 2017): it penalizes nesting and breaks in linear flow and does not penalize linear code.
- **Outside → Inside** — what is visible from outside (`public`/`export`) comes before what is internal (`private`/non-exported).
- **Type A File** — a singleton class; **Type B File** — a function module.
- **File Sections** — the top level of order: Header → imports → types → constants → body.
- **Body** — the last section: class members (Type A) or module functions (Type B).
- **File Skeleton** — the "Sections → Body" hierarchy: the mandatory order of code in a file.

---

## Structure

The normative order of code; the Skeleton principles refer here and do not repeat it.

### File Types

| Type                   | Typical location                           | Exports                             |
| ---------------------- | ------------------------------------------ | ----------------------------------- |
| **A. Singleton class** | application, domain, infrastructure layers | `export const x = new X();` + types |
| **B. Function module** | shared code, entity rules                  | functions, constants, types         |

`index` files — re-exports only; the order does not apply to them. A types-only file follows item 3 of File Sections.

### File Hierarchy

| Level        | Contents                                         | Common to both types |
| ------------ | ------------------------------------------------ | -------------------- |
| **Sections** | Header → imports → types → constants → body      | ✓                    |
| **Body**     | Type A — class members; Type B — function module | depends on the type  |

### File Sections (Both Types)

1. **Header** — if the file implements a Process (see Scenario Header).
2. **Imports** — far → near (external packages, then own modules from lower levels to neighbours).
3. **Types**: `export` first, then non-exported.
4. **Module constants**: `export` first, then non-exported. Exception — a load-time dependency: a constant read by the initializer of another one comes first (otherwise a TDZ error).
5. **Body** — the class (Type A) or the functions (Type B).

### Type A Body — Class Members

1. **Fields** — only at the top of the class: `public` → `protected` → `private`. A field initializer that reads another field comes after the field it reads.
2. **`constructor`** — the first method (if any).
3. **Methods** — `public` → `protected` → `private`; private methods at the end of the class. Getters/setters at the start of their visibility group. Arrow fields (used for `this` binding) count as methods. `static` comes before non-static members of the same group; `readonly`/`async` do not affect the order.
4. **Singleton** — the last line of the file: `export const x = new X();`. The class itself is not exported.

### Type B Body — Function Module

1. **Module state** (`let`, caches, `Map`) — non-exported only; it is exposed through functions.
2. **Exported functions** — caller above callee, related functions together.
3. **Non-exported functions** (helpers) — at the end of the file; hoisting of `function` declarations makes a call above the declaration safe.

---

## Principles

### Necessary (Mandatory)

#### Readability

- **Scenario Header** — a file implementing a Process opens with a Header: one sentence on what the module does and who calls it; for each public operation — its business steps; the outcomes of a step when there are several (`a) / b) / c)`) and what each one means for the caller; general rules — at the end. The Header is written in plain words, in the language of the domain, without technical details: endpoints and API methods, parameters, response codes, limit and timeout values, env variables, keys, error texts. These are visible in code, and the non-obvious ones are described at their step (`/** Step N: … */`). The Header is understood by someone who knows the process but has not read the code. A separate document does not replace the Header: the reader of the code will not open it.
- **Header Matches Code** — Paragraphs start with the same numbers; doc comments of methods and constants name their step (`/** Step N: … */`); when behaviour changes, the Header is updated **in the same change**.
- **One Scenario Function** — a public operation is written as a single function made of Paragraphs, read top to bottom.
- **Extract a Method Only in Three Cases**: it is reused (≥ 2 calls), it is a Deep Method, it is a Detail. A single-call step without which the Scenario cannot be understood stays a Paragraph; one-line wrappers are inlined.
- **Reading Depth ≤ 1** — Scenario → Deep Methods / Details; not "step → sub-step → sub-sub-step".
- **Flat Flow** — guard clauses and early `return`/`throw` instead of an `if/else` ladder; nesting ≤ 2; the outcome of a step is a value (`T | Error`, a discriminated union), not a `let` reassigned throughout the function or a set of flags (a `let` under `try` with a single assignment is allowed); cancellation uses the standard mechanism (`signal.throwIfAborted()`), not hand-written checks.
- **Blank Lines** — inside a function body, a blank line after every statement. The only exception: consecutive variable/constant declarations or settings assignments form one block.
- **Decisions Apart From Actions** — branching **on data** (status → result, code → value) goes into a pure function or a table (`Set`, `Record`); the Scenario keeps the chain "call → decide → call" (functional core / imperative shell).
- **Remove Complexity First** — before laying the code out, delete: dead branches (git keeps them); special cases that barely change the result; duplicate checks and repeated lines; types whose outcomes are handled the same way. Every deletion that changes behaviour (an error text, a log line) is listed explicitly in the change description.
- **Comment Explains Intent** — a comment explains **why** and which step this is; what a line does is visible from the code. Non-obvious contracts (text read by another module) — a comment where it is produced.

#### Skeleton

- **Outside → Inside** — the general principle behind the order in Structure: a private class method and a non-exported module function are analogues, both go at the end.
- **Order Is Mandatory** — sections and body follow Structure exactly; groups are not mixed.
- **Type A Purity** — no module functions in a class file: a helper of one class is a `private` method, a helper needed by several classes goes to shared code; one class per file; `public` without a modifier, `private`/`protected` explicit, privacy via `private`, not `#field`. Error classes (`extends Error`) — member order as in Type A, without a singleton.
- **Type B Purity** — only `function` / `async function` declarations, not `const f = () => …` (an arrow function is not hoisted — a helper at the bottom would break a load-time call); `export` at the declaration, not as a list at the end; no classes (except error classes).

### Useful (Recommended)

- **One Process — One File While It Reads** — do not split a Process by layers (transport / protocol / scenario) in advance. Splitting is justified when a part is actually reused or the file no longer reads as a whole; otherwise the reader jumps between files, and intermediaries and adapter types appear between them.
- **Metrics** — the measurable part of the standard, formulated independently of tooling: any analyzer that can compute them will do.
  - **Threshold**: cognitive complexity ≤ 10 per function (the model does not penalize linear code, so a One Pile scenario does not hit the threshold — nesting and flow breaks do); nesting depth ≤ 2; parameter count ≤ 4 (a signature that cannot be held in mind is more complex than the implementation — such a method is not Deep, just an extracted chunk).
  - **Zero** (presence = violation): `else` after a branch ending with `return`/`throw`; a reassigned variable (one assignment under `try` is allowed); a nested condition that can be merged with the outer one; boolean noise (`if (x) return true; else return false`, `!(a === b)`); textually identical functions and branches with the same body; unreachable and redundant `return`/`continue`; an intermediate variable returned immediately (`const x = …; return x;`).
- **Refactoring Procedure**:
  1. describe the Process in plain words (business steps and outcomes — as it works now) and check it with the code owner;
  2. One Pile: gather the Process into one file, inline single-call step methods and intermediate layers;
  3. remove unnecessary complexity, write down behaviour changes;
  4. Header from item 1, Paragraphs, Flat Flow, blank lines;
  5. re-extract only Deep Methods and Details;
  6. decisions on data — into pure functions/tables;
  7. order — per Structure; run the existing tests. Refactoring the order does not change behaviour: moves only.

---

## Anti-patterns (Red Flags)

- No Header, or the Header is a glossary without steps, or a retelling of the implementation (endpoints, parameters, codes, timeouts, env variables, keys); numbers in code do not match the Header.
- **Conjoined methods** — two methods cannot be understood without each other → they are one method.
- **Pass-through** — a method only passes its arguments further (exception — the public facade of a module).
- **Single-call step method** that has to be opened to understand the Scenario.
- **Type for a single hand-off** between two methods or files, which disappears when they are merged.
- A Process scattered across several files without reuse.
- A `let` changing across several Paragraphs; nesting 3+; statements without blank lines.

---

## Examples

### File Header

```ts
/**
 * Payment provider client: issues an invoice for an order.
 * Called by the domain layer.
 *
 * createInvoice({ orderId, amount }) — payment link:
 *  1. Check that the provider is available: after several failures in a row,
 *     stop calling it for a while — fail right away.
 *  2. Create an invoice for the order amount. Outcome:
 *     a) invoice created — return the payment link;
 *     b) the provider refused;
 *     c) no response — the invoice may have been created.
 *  3. On "b" and "c", look for an invoice for this order created earlier:
 *     found — return it, not found — fail.
 *
 * No connection to the provider or a slow response counts as a failure for step 1.
 */
```

### Technical Details — at Their Step, Not in the Header

```ts
/** Step 1: slower than this — provider failure; 5xx and network errors too. */
const REQUEST_TIMEOUT_MS = 10_000;

// …

/** Step 2: invoice created — code 0 and invoiceId present; any other response — refusal. */
private register(params: CreateParams): Promise<ProviderInvoice | Error> { … }
```

### Scenario of Paragraphs (with Step Numbers from the Header)

```ts
async createInvoice(params: CreateParams): Promise<ProviderInvoice> {
  // 1. Provider availability.
  return this.withCircuit(async () => {
    // 2. Invoice creation.
    const registered = await this.register(params);

    // 3. Refusal or unknown outcome — look for an invoice created earlier.
    return registered instanceof Error
      ? await this.findExisting(params.orderId, registered)
      : registered;
  });
}
```

### Blank Lines and Flat Flow

A fragment of `findExisting` — step 3:

```ts
const code = this.errorCode(res);
const message = String(res.errorMessage ?? "").toLowerCase();

if (code === NOT_FOUND || message.includes("not found")) throw registerError;

const invoiceId = this.extractInvoiceId(res);

if (!invoiceId) throw registerError;

this.circuit.recordSuccess();

logger.info("provider.findExisting", invoiceId);

return { invoiceId };
```

### Type A File — Singleton Class

```ts
/** Header — as in the File Header example. */

import …;

export type ProviderInvoice = { … };
type CreateParams = { … };

const API_URL = requireEnv("PROVIDER_API_URL");

/** Step 1: slower than this — provider failure; 5xx and network errors too. */
const REQUEST_TIMEOUT_MS = 10_000;

class ProviderClient {
  private readonly circuit = new CircuitBreaker("provider");

  constructor() { … }

  async createInvoice(params: CreateParams): Promise<ProviderInvoice> { … }

  /** Step 1: circuit open — no request is sent, fail right away. */
  private withCircuit<T>(run: () => Promise<T>): Promise<T> { … }

  /** Step 2: invoice created — code 0 and invoiceId present; any other response — refusal. */
  private register(params: CreateParams): Promise<ProviderInvoice | Error> { … }

  /** Step 3: getOrderStatus method by orderId; not found — throws registerError. */
  private findExisting(orderId: string, registerError: Error): Promise<ProviderInvoice> { … }

  /** Detail of all steps: POST {API_URL}/<method> with the REQUEST_TIMEOUT_MS timeout. */
  private call<Req, Res>(method: string, request: Req): Promise<Res> { … }
}

export const providerClient = new ProviderClient();
```

### Type B File — Function Module

A request rate limiter by plan:

```ts
import …;

export type Plan = "free" | "pro";
export type LimitDecision = { allowed: boolean; retryAfterMs: number };
type Bucket = { tokens: number; refilledAt: number };

export const DEFAULT_PLAN: Plan = "free";
const LIMIT_PER_MINUTE: Record<Plan, number> = { free: 60, pro: 600 };
const REFILL_INTERVAL_MS = 60_000;

const buckets = new Map<string, Bucket>();

export function takeToken(key: string, plan = DEFAULT_PLAN): LimitDecision { … } // calls remainingTokens
export function remainingTokens(key: string, plan = DEFAULT_PLAN): number { … }
export function resetLimit(key: string): void { … }

function bucketFor(key: string, plan: Plan): Bucket { … }
function refill(bucket: Bucket, plan: Plan, now: number): Bucket { … }
```
