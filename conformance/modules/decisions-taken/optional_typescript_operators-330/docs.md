# Decision 330 — `?T` is read with TypeScript's operators and has no methods; `result` goes; a type declared in a type is its associated type.

A blog's `Post` may carry a Markdown source (`source: ?string`) and a hook that rewrites a heading's anchor (`anchor: ?fn(string) -> string`); the guide has both, the draft neither. Every read of those optionals is an operator, never a method:

- `Post.firstSection(guideDoc)?.slug ?? "top"` reads a field of an optional `Post.Section` as a `?string`, defaulted by `??` — `install`, the guide's first heading's id;
- `guide.anchor?.(first) ?? first` calls the hook when the post holds one, and `?? "top"` gives the draft its default — the first line is `INSTALL`, the second `top`;
- `notesOf(guide)?.[0]?.label` reads the first footnote through an optional array: the index already answers `?Footnote`, and through `?.` it stays `?Footnote`, never `??Footnote` — the third line is the guide's footnote label, `n`, the fourth `no note`;
- `guide.source!` asserts the guide's source is there; a `null` at that point would abort, located, alike on every target.

`titleOf` answers a `Result`, and `main` reads it by its own methods, `unwrapOr("untitled")` and `isError()` — `Install`, then `true` for the draft —; there is no `result` namespace to call. `Post.Section` is declared in the body of `Post`, so it is `Post`'s associated type: `firstSection`, a static of `Post`, answers `?Post.Section`. Writing `?.` over a value that is never `null` — `guide.title?.length()` — or `??` beside `&&` without parentheses would be refused.

Cells of this repository that prove it: none yet; the cell comes with 158's Conformance step.

[draft/main.bp](draft/main.bp)
