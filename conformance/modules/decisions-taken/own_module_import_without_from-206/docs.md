# Decision 206 — `import {…} from "<name>"` resolves only to a package — std, a bundled package or a declared dependency.

A consumer of `markdown` with a module of its own, `toc`. The program's imports name where each name comes from: `io.terminal` from std and the module `markdown` from the dependency `markdown`, both with `from`; the function `outline` and the value `bullet` from the package's own module `toc`, by its path in braces and with no `from` — `bullet` bound under another name, `mark`, with `as`. `toc` itself imports `markdown` and `markdown.MdDoc` from the dependency. The program parses a guide with three headings and prints its outline — one line per heading, its depth, the bullet, its text and its id (`1 * Guide #guide`, `2 * Install #install`, `2 * Use #use`) — then the bullet alone.

Writing `from "toc"` for the package's own module would be refused at the source string, with the brace form as its fix; and were a bundled package named `toc` to appear later, `import {toc.outline};` would keep meaning this package's module.

Cells of this repository that prove it: none yet. The library's own tests import its module by its path (`test/markdown_test.bp`, `import {markdown, markdown.MdNode, markdown.MarkdownOptions};`); the cell comes with 158's Conformance step.

[draft/main.bp](draft/main.bp)

[draft/toc.bp](draft/toc.bp)
