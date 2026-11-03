# Heuristic audit fixes

Date: 2026-11-03. Source: the H1 to H10 audit of the same day. The user approved decision 1 (b), fix U1 to U7; decision 2 (a), a "Change history" section at the end of the specification; and decision 3 (a), link the compiler repository and add the commands.

## Ownership

Two Sonnet implementers run in parallel in the same working tree on local `main`. Each owns its files and edits no other file.

| Implementer | Owns | Units | Turn budget |
| --- | --- | --- | --- |
| A (code) | `public/site.css`, `public/site.js`, `.design-system/system.md` | U3, U6, U7, U5, U4-code | 45 tool calls |
| B (content) | `public/index.html`, `public/effektive-spec.html`, `AGENTS.md` | U4-content, U1, U2 | 40 tool calls |

Opus reviews the diff, runs the browser checks on both pages, raises the version, runs `deno task stamp`, and runs `deno task test` and `deno task typecheck`.

## Units

**U3 (anchor jumps land too low).** `html { scroll-padding-top }` and `.section { scroll-margin-top }` add together. Remove `scroll-margin-top` from both `.section` rules. Check: a desktop contents jump puts the section top at 104px, and the mobile menu label names the target section.

**U6 (no visible loading state on a source link).** Dim `a[aria-busy="true"]` to opacity .55 after a 150ms delay. Check: under a slow network the link dims, and it returns to full opacity when the drawer opens.

**U7 (two names for each explorer function).** Accept the button labels (Lookup, Rename, Recover, Collect) as aliases. Mark a rejected name with `aria-invalid="true"` and the muted color. Clear both the mark and the message on the next input.

**U5 (Back leaves the page while the drawer is open).** Add one history entry when the drawer opens. Close the drawer on `popstate`. Remove the entry when the reader closes the drawer in another way.

**U4 (subsections cannot be linked).** Content: give the eight `h3` headings of `#core1` an `id`, and add a nested contents level for them in both contents lists. Code: make each section `h2` and each `h3` with an `id` a link to its own target. Style the nested contents level.

**U1 (the reference starts with a change log).** Move the 20 draft and release paragraphs from `#status` into a new last section, `#history` ("Change history"), newest first. Keep each paragraph byte-exact, including `id="compiler-reset"`. Link the new section from `#status`, the contents lists, and the section navigation. Add the rule for future drafts to `AGENTS.md`.

**U2 (no path to the compiler).** Link "effektive-lang repository" to `https://github.com/srdjan/effektive-lang` on both pages. Add the clone, check, and build commands to `index.html#orientation`, with links to the tutorial and the installation guide.

## Verification

- Browser: each unit's check at 1280 by 800 and 390 by 844, on both pages.
- `deno task test`, `deno task typecheck`, `deno lint public/site.js`, and `deno fmt --check public/site.js`.
- Every in-page link resolves to an existing `id`.
- The compiler commands: `check` ran in the local clone. `build --tsc` runs in a temporary clone.
