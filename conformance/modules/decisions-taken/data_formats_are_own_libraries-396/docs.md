# Decision 396 — Markdown, YAML and JSON are three libraries of their own — `markdown`, `yaml`, `json` (`botopink/{markdown,yaml,json}` at `repository/{markdown,yaml,json}`, 326's rule) — and std knows nothing of JSON.

A consumer of `markdown` that renders a note into an element type of its own. `markdown.parse` answers `markdown`'s tree — `MdNode`'s `Heading`, `Paragraph`, `Emphasis`, `Strong`, `Code`, `CodeBlock`, `Link`, `Image`, `List`, `Item`, `Quote`, `Text`, … — and nothing in it names a framework, so the program's `element` walks the tree with one `case` and builds its own `Element`, as onze-content builds jhonstart's. Each top-level block is printed as the element it became: `h1("Notes")`; the paragraph with its `em`, `strong`, `code` and `a:https://botopink.dev` children; the quote holding a paragraph with `img:logo.png` and its alt text; the `ul` with its one `li`; the fenced block as `pre:bp` with its text.

`markdown` is a library of its own, a dependency the program declares and imports `from "markdown"`, never a module of std; `json` and `yaml` are the same, and std knows nothing of JSON — `import {json} from "std"` names nothing.

Cells of this repository that prove it: none yet. The library's own tests walk the tree (`test/markdown_test.bp`); the cell comes with 158's Conformance step.

[draft/main.bp](draft/main.bp)
