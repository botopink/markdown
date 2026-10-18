# Decision 337 — The shorthand import goes; `mod m;` binds the namespace `m` in the module that declares it.

A refused program. The package's module `toc` declares `outline`, which lists a document's heading ids through the dependency `markdown` (imported with `from "markdown"`). `main.bp` writes `import {outline};` — a name with no module before it and no `from`, the shorthand — and the program is refused at that item, `shorthand-import`, the fix written: `toc` is the one module of the package that declares `outline` `pub`, so the import to write is `import {toc.outline};`. No bare name reaches a dependency either: `import {parse};` would not find `markdown`'s `parse`.

The row's second half, `mod m;` binding a namespace, is overtaken by decision 592: every file under `src/` is a module and no `mod` / `pub mod` is written, so this example declares none — `toc.bp` is a module because it is a file.

Cells of this repository that prove it: none yet; the cell comes with 158's Conformance step.

[draft/main.bp](draft/main.bp)

[draft/toc.bp](draft/toc.bp)
