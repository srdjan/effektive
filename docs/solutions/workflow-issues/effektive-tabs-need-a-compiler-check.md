---
title: The site type check does not check the Effektive tabs, so compile them with effektive-lang
date: 2026-10-17
last_updated: 2026-11-04
category: workflow-issues
module: typecheck harness and Effektive figures
problem_type: workflow_issue
component: documentation
related_components: [tooling]
severity: medium
applies_when:
  - The effektive-lang compiler adds or changes a language rule
  - An Effektive figure, exercise, or example file on the site changes
  - A figure gets a new data-check-requires entry
  - An example file in public/effektive-examples imports another file
tags: [typecheck, effektive-tabs, data-check-requires, data-check-id, effektive-lang, spec-migration, r1, service-headers, website-examples, with-clause]
---

# The site type check does not check the Effektive tabs, so compile them with effektive-lang

## Context

`deno task typecheck` checks only the Effect-TS tabs. The harness reads each panel whose code has `class="language-typescript"` (`typecheck/check.ts:109`). For an Effektive block, it only checks that the figure has an Effect-TS panel (`typecheck/check.ts:156`). So an Effektive tab that the compiler rejects still passes `deno task typecheck` and `deno task test`.

The harness was built on 2026-10-16, when no compiler could check the Effektive side (session history). The compiler now exists in `/Users/srdjans/Code/effektive-lang`.

For draft 0.2, the compiler added three rules. R1 allows primitive types only inside type or error declarations (`declaration/013`). The other rules add nominal records (`Brand<{ ... }>`) and operators on numeric brands. Before the migration, 66 primitives in signatures broke R1. At that time, the pages had 30 Effektive blocks and the examples had 5 files. The site check passed on all of them.

The example files have an automatic compiler check, but it is in the other repository. `tests/website_examples_slow_test.ts` in effektive-lang runs `check` and then `build --tsc` on each unit that `fixtures/website-examples.json` lists (`tests/website_examples_slow_test.ts:141`). The figures on the two pages still have no automatic compiler check.

## Guidance

After a compiler rule change, check the Effektive side with the compiler itself, in three passes.

1. Parse each Effektive block with the compiler parser. Extract each `<code class="language-effektive">` block from `public/index.html` and `public/effektive-spec.html`. Unescape the HTML entities, and include `public/effektive-examples/*.eff`. Call `parse("block.eff", code)` from `/Users/srdjans/Code/effektive-lang/src/core/parse.ts`.

   Walk the tree for the rule. For R1, report each `NamedType` node named `string`, `int`, `number`, or `boolean`, and each `LiteralType` node, that has no `TypeDecl` or `ErrorDecl` ancestor. The fragments in the figures `spec-pipe` and `spec-deferred` are not modules, so they give syntax findings. Those findings are expected.
2. Build each figure with the figures that it requires. For each `data-check` unit, collect the Effektive blocks of its `data-check-requires` chain in dependency order. Add the own blocks of the unit last.

   Write them to one `.eff` file in a copy of `/Users/srdjans/Code/effektive-lang/fixtures/examples`. The copy needs `effektive.json`, the `host` directory, and `users-live.ts`. Run `deno task effektive build <dir> --tsc` from effektive-lang. Expect 0 diagnostics. Only `spec-diagnostics` must give its documented `failure-contract/001`. This was confirmed again on 2026-11-04.

   Some panels show more than one module. Each module starts after a `// <file>.eff` comment line, for example `// console-service.eff` and then `// log.eff` in the `spec-log` figure. Split such a panel at those lines and write each module to its own file. Do not join them into one file. Service headers (`imports Users from "./users-service.eff"`) must come before all ordinary imports and declarations, so a joined file gives a diagnostic.
3. Run the compiler's website example test from effektive-lang: `deno test --allow-all tests/website_examples_slow_test.ts`. On 2026-11-04 it ran 21 tests in 12 to 18 seconds: one test for each of the 20 units, and the census test.

   The test stages a single-file unit alone, with only the files in its `companions` list (`tests/website_examples_slow_test.ts:71`). When an example file starts to import another file, add that file to `companions` in `fixtures/website-examples.json`. A census test requires one unit for each `.eff` file and each project directory in `public/effektive-examples/` (`tests/website_examples_slow_test.ts:114`). So a new contract file also needs its own unit. The fixture is in effektive-lang, so this change is a second commit in that repository.

When an Effektive panel on the book page changes, measure its character count again. Some paragraphs state the number of non-whitespace characters in a panel, for example "This pair uses 223 non-whitespace characters in Effektive and 304 in Effect-TS". No check compares these numbers with the panels. Count every non-whitespace character of the panel after HTML decoding. Do not estimate.

Also look for Effektive code outside `language-effektive` blocks. An exercise answer in a plain `<pre><code>` escapes both the site check and pass 1. On draft 0.2, the answer `const rename = (user: User, name: string): User => ...` in `index.html` broke R1 until a review found it.

## Why This Matters

The pages are the language reference. A stale Effektive example teaches a program that the compiler rejects. The Effect-TS check cannot find this fault, because the Effect-TS translation of a rejected program can still type-check. On draft 0.2, the Effect-TS tabs used `id: string` in signatures, and `tsc` accepted them.

A stale character count also teaches a wrong comparison. The draft 0.75 change added the 14-character prefix `users-service#` to an error tag in the recovery Effect-TS panel. The paragraph kept 216, while the panel had 230, until a recount on 2026-11-04 found it.

## When to Apply

Apply this after each effektive-lang change that adds a diagnostic or changes a type rule. Apply it before each commit that edits an Effektive figure or an example file.

## Examples

Three faults came up in the draft 0.2 migration. Each one fails silently or with an unclear message.

- `data-check-requires` resolves only against a panel `data-check-id`, not against a figure `data-check` name. The harness gives `unit "book-blocks" requires unknown id "book-contracts"` (`typecheck/check.ts:271`) and checks no unit. To fix it, add `data-check-id="book-contracts"` to the Effect-TS panel of the required figure.
- Some units import `"./users.eff"`, for example `spec-main` and `book-modules`. They must compile alone, beside the example `users.eff`. If you prepend their required figures, the unit probably declares `User` twice. This was not tested.
- In draft 0.2, the `users` unit of the book contained the service `Users` and a `provide(UsersLive, ...)` call. The host interface file of the examples imported `Users` from `../users.eff`. When the whole unit was written as `users.eff`, the build gave `module/003: import cycle`. The fix moved the provider block to a second module, `app.eff`, that imports from `./users.eff`. Since draft 0.75, the host interface file imports `Users` from `../users-service.eff` (`public/effektive-examples/host/providers.effi:1`). The same cycle probably occurs when such a unit is written as `users-service.eff`. This was not tested.

The removal of `with` clauses from the examples on 2026-11-04 (compiler 0.15.0) gave three more faults:

- A module without service headers closes the requirement set of an exported function. When the `with` clause is removed and the export calls a service, the compiler gives `requirement-contract/001`. A local function infers its requirements and needs no header.
- A module cannot import a service from itself (`module/003`). An example that declares its service in the same file must move the service to a contract module, such as `console-service.eff`, and add a header.
- A name in a service header must not also be in an ordinary import of the same module (`declaration/001`). Remove the service name from the ordinary import.

A provider declaration in a host interface file keeps its `with` clause, for example `with Runtime.database` in `public/effektive-examples/service-workflow/host.effi`. Host interface files cannot have service headers.

For a new rule, keep the alias style of the migrated examples. Signatures name a declared type, such as `type Name = string`. The Effect-TS tab uses a plain TypeScript alias, `type Name = string`. The construct table in `docs/plans/2026-10-16-effect-ts-tabs.md` lists these mappings.
