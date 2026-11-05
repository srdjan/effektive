# Effektive

Keep `name`, `version`, and `exports` in `deno.json`. Do not remove them. Deno warns when `name` has no `exports`.

Before you change the site design, read `.design-system/system.md`. When a design rule changes, update that file in the same commit.

In `public/effektive-spec.html`, keep "Status and scope" to the scope of the language and the current edition. Record each new draft or compiler release as a paragraph at the top of the "Change history" section, newest first.

The `docs/solutions/` directory holds documented solutions to past problems. A category directory groups the files, and each file has YAML frontmatter with `module`, `tags`, and `problem_type`. These solutions are relevant when an agent changes the examples, the pages, or the site checks.
