# Remove `with` clauses from the code examples

Date: 2026-11-04. Status: complete.

## Goal

No code example on the site writes a `with` clause. Each example gets its requirements from service headers (`imports X from "..."`). The compiler infers the requirements of every function in a module with service headers.

## Facts that the compiler confirmed (effektive-lang 0.15.0, commit a0916ce)

- A module without service headers closes the requirement set of an exported function. If the `with` clause is removed and the export calls a service, the compiler gives `requirement-contract/001`.
- A module cannot import a service from itself (`module/003`). A service that the same file declares cannot get a header. The service must move to a separate contract module.
- A name in a service header must not also be in an ordinary import of the same module (`declaration/001`). Remove the service name from the ordinary import.
- A local (non-exported) function always infers an omitted `with` clause. It needs no header for a same-file service.
- `provide(StoreLive, ...)` removes `Store`, so `service-workflow` needs headers only for `Catalog` and `Runtime`. `Runtime` comes from `host:store`.
- A provider declaration in a host interface file keeps its `with` clause: `export provider StoreLive provides Store fails StoreFailure with Runtime.database`. Host interface files cannot have service headers. This is not a code example to change.
- Schema type arguments such as `Schema<Optional, Optional, Read.next>` accept a service that a header imports. They are not `with` clauses.

## Decisions

- D1 (same-file services): move each such service into its own contract module and add a header. Inline panels show both modules, each after a `// <file>.eff` comment line.
- D2 (compiler fixture): edit `effektive-lang/fixtures/website-examples.json` so that the compiler test stages the new contract files. Commit in effektive-lang. Do not push.
- D3 (Effect-TS tabs): keep the Effect-TS tabs unchanged. Single-file tabs already use bare tags, such as `ConsoleUnavailable`, not the emitted `log#ConsoleUnavailable`.
- D4 (rule prose): keep the rule prose that describes `with` as an optional upper bound. It is true for compiler 0.15.0. Change only prose that describes the `with` clause of a changed example.

## Panel convention for split modules

```
// console-service.eff
export type Message = string
...

// log.eff
imports Console from "./console-service.eff"
import { Message } from "./console-service.eff"
...
```

In HTML, escape `<` as `&lt;`, `>` as `&gt;`, and `"` as `&quot;`, as the existing panels do.

## Units

| Unit | Model | Files | Acceptance check | Turn budget |
| --- | --- | --- | --- | --- |
| U1 (example sources and compiler fixture) | Sonnet | `public/effektive-examples/**/*.eff`, three new contract files, `effektive-lang/fixtures/website-examples.json` | `deno test --allow-all tests/website_examples_slow_test.ts` in effektive-lang passes all 21 tests: 20 units and the census test. No `.eff` file in the examples has a `with` clause. | 40 |
| U2 (specification page) | Sonnet | `public/effektive-spec.html` | Each changed Effektive panel compiles in a temporary project. `deno task typecheck` and `deno task test` pass. | 50 |
| U3 (book page) | Sonnet | `public/index.html` | Each changed Effektive panel compiles in a temporary project. Character counts are measured. `deno task typecheck` and `deno task test` pass. | 50 |

U1, U2, and U3 change separate files and run in parallel. Opus reviews each diff, runs every check again, and commits.

## New contract files (exact text)

`console-service.eff`:

```
export type Message = string

export error ConsoleUnavailable = { message: string }

export service Console {
  print(message: Message): void fails ConsoleUnavailable
}
```

`payment-services.eff`:

```
export type Cents = int
export type ChargeId = string

export type OrderId = string
export type Order = {
  id: OrderId,
  totalCents: int,
  status: "pending" | "paid"
}

export error OrderNotFound = { id: OrderId }
export error PaymentDeclined = { reason: string }

export service Orders {
  find(id: OrderId): Option<Order>
  save(order: Order): void
}

export service Payments {
  charge(amount: Cents): ChargeId fails PaymentDeclined
}
```

`core1-services.eff`:

```
export type Int = int
export type Text = string

export service Read { next(value: Int): Int }
export service Defaults { make(): Int }
export service Wire {
  parse(value: Text): Int fails SchemaIssue
  print(value: Int): Text fails SchemaIssue
}
```

## Changed consumers (exact text where a page quotes them)

`log.eff`:

```
imports Console from "./console-service.eff"
import { Message } from "./console-service.eff"

export const log = (message: Message): void =>
  Console.print(message)
```

`pay.eff`: headers `imports Orders from "./payment-services.eff"` and `imports Payments from "./payment-services.eff"`, then `import { Order, OrderId, OrderNotFound } from "./payment-services.eff"`. The body from `const findOrder` to the end stays the same, except `export const pay = (id: OrderId): Order =>` on one line.

`core1.eff`: replace line 1 (`type Int = int`) with headers for `Read`, `Defaults`, and `Wire`, and `import { Int, Text } from "./core1-services.eff"`. Remove the local `Read`, `Defaults`, and `Wire` services and the local `type Text = string`. Remove the four `with` suffixes. `Validation` stays local.

Users files (`activation.eff`, `activation-label.eff`, `active-names.eff`, `retry-lookup.eff`): replace the local declarations with `imports Users from "./users-service.eff"` and an ordinary import of only the names that the file uses. Remove each `with` line and put `=>` (and `{`, if present) at the end of the line before it.

Project examples: `scheduled-worker/worker.eff` gets headers for `Jobs` and `Clock`. `service-workflow/workflow.eff` gets headers for `Catalog` (from `./contracts.eff`) and `Runtime` (from `host:store`). `stream-processor/processor.eff` gets headers for `Input`, `Transform`, and `Output`.

## Character counts on the book page

Each count is the number of non-whitespace characters in a panel after HTML decoding. The Recovery Effect-TS count was already wrong: the page says 216, and the panel has 230. The `users-service#` tag prefix adds 14 characters.

## Results

- No `.eff` example and no Effektive panel on either page has a `with` clause. The provider declaration in `service-workflow/host.effi` keeps its `with Runtime.database` (see the facts above).
- The compiler website example test in effektive-lang passes 21 of 21 tests: 20 units, including the three new contract units, and the census test.
- Each split panel on both pages compiles and builds with `--tsc` with 0 diagnostics. The `log.eff`, `pay.eff`, and contract text in the panels is byte-identical to the example files. The four Users panels are verbatim excerpts of their example files.
- The `broken` panel still gives exactly one error, `failure-contract/001`.
- `deno task typecheck` passes 65 strict units. `deno task test` passes 12 tests.
- Book page counts after the change: activation 223 and 304, collection wrapper 83 and 144, collection panel 174 and 266, recovery 156 and 230, retry 109 and 156.
