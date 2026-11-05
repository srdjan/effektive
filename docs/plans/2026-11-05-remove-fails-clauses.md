# Remove `fails` clauses from the code examples

Date: 2026-11-05. Status: complete.

## Goal

No consumer function in a code example writes a `fails` clause. The compiler infers each failure set from the function body. A service author still writes the `fails` clause of each service method, because that clause is the service contract.

## Facts that the compiler confirmed (effektive-lang 0.15.1, commit a3504d4)

- Since draft 0.73, the compiler infers an omitted `fails` clause on every function with a body, exported or not.
- Six example files have consumer `fails` clauses: `activation.eff`, `activation-label.eff`, `active-names.eff`, `retry-lookup.eff`, `refined-entry.eff`, and `core1.eff`. With the clauses removed, each file builds with `--tsc` and 0 diagnostics. The emitted TypeScript of each file is byte-identical to the output before the change. So the Effect-TS tabs stay correct without a change.
- `fails` in a service method declaration, in a host interface file (`.effi`), and in prose stays.

## Edit rule for code

In each consumer function signature, delete the `fails` clause. When the clause stands on its own line, move `=>` and the rest of that line to the end of the previous line, after one space.

On decoded source, the rule is the regular expression `/\s+fails\s+[\w\s|]+?=>/g`, replaced with ` =>`. In HTML, apply `/\s+fails\s+[\w\s|]+?=&gt;/g`, replaced with ` =&gt;`, only inside `<code class="language-effektive">` blocks.

Do not change the `spec-diagnostics` figure. Its `broken` function shows the `failure-contract/001` diagnostic of a written `fails` clause (decision D1 below).

## Decisions

The user accepted the recommendation for D1, D2, and D3 on 2026-11-05.

- D1 (diagnostic example): keep the `spec-diagnostics` figure. It shows `failure-contract/001`, and the language still allows a written `fails` clause as an optional guard.
- D2 (advice in the book): remove "A written fails clause guards against this" from `public/index.html:124`. Keep "Editor inlay hints show the inferred failures." The rule prose on both pages that describes `fails` as an optional checked upper bound stays.
- D3 (language rule): the compiler keeps the written `fails` clause as an optional guard. No example uses it. No effektive-lang change.
- D4 (Effect-TS tabs): keep them unchanged, because the emitted types are identical.
- D5 (compiler fixture): no change. No file is added or removed.

## Units

| Unit | Model | Files | Acceptance check | Turn budget |
| --- | --- | --- | --- | --- |
| U1 (example sources and specification page) | Sonnet | six `.eff` files above, `public/effektive-spec.html` (figures `spec-refined-entry` and `spec-core1-effectful-ref`) | Compiler website example test passes 21 of 21. No consumer `fails` clause stays. The `count` panel compiles alone with 0 diagnostics. | 25 |
| U2 (book page) | Sonnet | `public/index.html` | Each changed panel matches its example file. Counts are measured. `deno task typecheck` and `deno task test` pass. | 30 |

U1 and U2 change separate files and run in parallel. Opus reviews each diff, runs every check again, and commits.

## Book prose edits (U2)

- Line 246: replace "including the pure helper and explicit exported contracts" with "including the pure helper and the exported signatures". After "Effektive keeps direct calls and a final expression.", add "The compiler infers the failure set of `saveActive` from the body."
- Line 273: replace "Including the helper and explicit exported contracts" with "Including the helper and the exported signatures".
- Line 308: replace "and explicit exported contracts" with "and the exported signatures".
- Line 320: replace "including explicit exported contracts and excluding" with "including the exported signature and excluding".
- Measure each count again with `/tmp/failsx/scripts/counts.ts`. Before the change, it gives 223, 174, 83, 156, and 109, which match the page.

## Results

- The rule removes 21 clauses from the book page, 2 from the specification page, and the consumer clauses from the six example files. The plan first said 22 for the book page. That was a miscount: the page has 21.
- Each edited file equals the rule applied to the HEAD version. The book page has four more changes: the planned prose edits.
- The compiler website example test in effektive-lang passes 21 of 21 tests. The `spec-core1-effectful-ref` panel builds alone with 0 diagnostics.
- `deno task typecheck` passes 65 strict units. `deno task test` passes 11 of 12 tests. The failing test, "pages load assets with the deno.json version", also fails at HEAD: the pages link `?v=0.15.0`, and `deno.json` is at 0.15.1.
- Book page counts after the change: activation 190 and 304, collection wrapper 62 and 144, collection panel 153 and 266, recovery 135 and 230, retry 88 and 156.
- D2 removed the advice sentence from the book. D1 and D3 need no edit.
