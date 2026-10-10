# markdown

> Repository: `botopink/markdown` (`git@github.com:botopink/markdown.git`) · in the meta checkout: `repository/markdown/`

Markdown read into a tree of its own (decision 396): `Heading`, `Paragraph`, `Emphasis`, `Strong`, `Code`,
`CodeBlock`, `Link`, `Image`, `List`, `Quote`, … — naming no framework (113). Moved from onze-content's
reader by `specs/1.0.12-beta/03-bundled-libs/142-data-formats` step 3; onze-content maps the tree to
jhonstart's `Element`.

A library of its own (decision 396, decision 326's rule). **Its one consumer, onze-content, is the
maintainer's exception to 115** ("a bundled library only when justified, two or more"; 396 (4)) — the
other two data-format libraries, `yaml` and `json`, meet the rule. A second consumer would end the
exception; none is planned. It imports std only, on commonJS and erlang.

## Tree

```text
markdown/
├── AGENTS.md
├── botopink.json     "name": "markdown", "target": "erlang", "targets": ["erlang", "commonJS"], std only
├── src/
│   ├── root.bp          `pub mod markdown;` · private `md_text`, `md_entities`
│   ├── markdown.bp      the public surface and the reader (below)
│   ├── md_text.bp       `Chars` (a string's code points with constant-time reads), character classes,
│   │                    `escapeHtml`, `matchEntity`, `unescapeString`, `normalizeLabel`, `normalizeUri`
│   └── md_entities.bp   `entityText(name)` — the 2 125 HTML5 named references, generated from
│                        <https://html.spec.whatwg.org/entities.json>
├── test/             both rows
└── scripts/git-hooks/   the shared pre-commit gate
```

## Surface (`import {markdown} from "markdown"`)

- `parse(source) -> MdDoc` / `parseWith(source, MarkdownOptions)`; `MarkdownOptions(smartPunctuation, gfm,
  headingIds)`, all on by default; `commonmarkOptions()` CommonMark alone, `gfmOptions()` the extensions
  without smart punctuation or ids.
- `MdDoc(nodes: Array<MdNode>, footnotes: Array<Footnote>)`; `Footnote(label, index, refs, kids)`.
- `MdNode` — blocks: `Heading(depth, id, kids)`, `Paragraph(kids)`, `CodeBlock(lang, text)`, `Quote(kids)`,
  `List(ordered, start, tight, items)`, `Item(task, kids)` (`task` 0 ordinary, 1 unchecked, 2 checked),
  `Rule`, `Html(text)`, `Table(aligns, head, rows)`, `Row(cells)`, `Cell(kids)`; inlines: `Text(value)`,
  `SoftBreak`, `HardBreak`, `Code(text)` (a code span), `Emphasis(kids)`, `Strong(kids)`, `Strike(kids)`,
  `Link(href, title, kids)`, `Image(src, alt, title)`, `InlineHtml(text)`, `FootnoteRef(label, index, nth)`.
- `toHtml(doc)` — the HTML of the CommonMark reference renderer (cmark-gfm's for the extensions);
  `headings(doc) -> Array<TocEntry(depth, slug, text)>`; `plainText(nodes)`; `slug(title)` (GitHub's
  rule); `footnoteId(label)` (the anchor id a footnote's `fn-` / `fnref-` ids are built from).

The reader is commonmark.js's two passes (blocks line by line over a stack of open blocks, then inlines
with the delimiter stack of § 6.2). Nothing here names a framework: a consumer walks `MdNode`.

## Tests

`botopink test --target erlang|commonJS` at the repository root: `test/commonmark/<section>_test.bp` — the
652 examples of <https://spec.commonmark.org/0.31.2/spec.json>, one test per example, rendered with
`commonmarkOptions()`; `gfm_extensions_test.bp` (24, cmark-gfm's GFM 0.29 spec), `gfm_footnotes_test.bp`
(3), `budget_test.bp` (a 200 kB document within 2 000 ms), `markdown_test.bp` (the default options, the
tree, `headings`) — 690 tests, both rows.

## Hooks

Every clone runs the shared pre-commit gate:

```sh
git config core.hooksPath scripts/git-hooks
```

`scripts/git-hooks/pre-commit` and `scripts/git-hooks/lib/runner-standalone.sh` are byte-identical
across the library repositories (the meta repository's CI check 4); edit them in one place and copy.
