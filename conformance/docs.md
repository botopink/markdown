# markdown — conformance

> Path: `repository/markdown/conformance/`
> Sibling (AGENTS): [`./AGENTS.md`](AGENTS.md)

The reference of `markdown`, the library that reads Markdown into a tree of its own and writes it as HTML. It is one of
the three data-format libraries (decision 396, with `json` and `yaml`): CommonMark 0.31.2 and the GitHub-flavoured
extensions parsed to an `MdNode` tree that names no framework, so a consumer — `rakun-ssr`'s content, which maps the tree
to `rakun-bpp`'s `Element` — walks it with one `case` and builds its own nodes. Every example is a cell (decisions 457,
461, 465): the decision cells under `modules/decisions-taken/` are the programs this reference points at, and a table or a
wire format is fenced as `text`, never as code.

| Section | What |
|---|---|
| *The package* | what a program declares and imports to use `markdown` |
| *markdown* | the options, `parse`, the tree, `headings`, `plainText`, `slug`, `footnoteId`, `toHtml` |
| *md_text* | the reader's text layer: code points, character classes, escaping, label and URL normalisation |
| *md_entities* | the HTML5 named character references |
| *Departures* | the spellings `src/` still writes that are retired |
| *Decisions* | the decision cells, one directory per code-bearing row of the ledger |

## The package

A program that reads Markdown declares `markdown` in `dependencies` (a repository of its own, decision 326) and imports
the module of the library by its path, from the dependency: `import {markdown} from "markdown";`, or the names it needs,
`import {markdown.parse, markdown.toHtml} from "markdown";` (decision 206: a name that comes from a package is written
with `from`; a module of the program's own package is imported by its path in braces and has no `from`, decision 337 —
the shorthand `import {parse};` is refused). The library imports std only and compiles for erlang and commonJS; the
reader is botopink all the way down but two host cells in `md_text`, which hold a string's code points. The decision
cell [own_module_import_without_from-206](modules/decisions-taken/own_module_import_without_from-206/draft/main.bp)
shows a program with a module of its own beside the dependency, and
[shorthand_import_refused-337](modules/decisions-taken/shorthand_import_refused-337/draft/main.bp) the refusal of the
bare name.

A function that calls `parse` and `toHtml` under `comptime` runs the library at build, on the wasm comptime runtime,
whatever the target the program is built for (decision 594: every library runs at build on that runtime); the answer is
the same string the run-time call answers —
[libraries_run_at_build_on_wasm-594](modules/decisions-taken/libraries_run_at_build_on_wasm-594/draft/main.bp).

## markdown

The module is the whole public surface; its other two modules are the reader's own.

### Options

`MarkdownOptions` says what a parse does beyond CommonMark. It has three `bool` fields, each on by default:

| Field | What it turns on |
|---|---|
| `smartPunctuation` | straight quotes curled, `--` and `---` to dashes, `...` to an ellipsis |
| `gfm` | tables, strikethrough, task list items, extended autolinks, footnotes and the tag filter |
| `headingIds` | an `id` on every heading, GitHub's slug, unique per document |

`commonmarkOptions()` answers CommonMark alone (all three off) — the options the specification's examples are rendered
with. `gfmOptions()` answers the extensions without smart punctuation and without ids — the options the GFM
specification's examples are rendered with.

### Parsing

`parse(source) -> MdDoc` reads `source` with the default options; `parseWith(source, options) -> MdDoc` reads it with
the options given. Neither fails: a text is always a document, whatever its content, as CommonMark defines it — there is
no error enum. The reader is the reference implementation's two passes (commonmark.js): the blocks line by line, then the
inlines of each paragraph, heading and table cell, with the delimiter stack of the specification's section 6.2. A
document of 200 kB is read within the budget the repository's tests hold it to.

### The tree

`MdDoc(nodes, footnotes)` holds the top-level blocks and the footnotes that are referenced. `Footnote(label, index,
refs, kids)` is one definition: its label, its number (the order of first reference), how many references it has and its
blocks. `MdNode` is one type with a variant per node.

```text
blocks    Heading(depth, id, kids)        Paragraph(kids)           CodeBlock(lang, text)
          Quote(kids)                     List(ordered, start, tight, items)
          Item(task, kids)                Rule                      Html(text)
          Table(aligns, head, rows)       Row(cells)                Cell(kids)
inlines   Text(value)                     SoftBreak                 HardBreak
          Code(text)                      Emphasis(kids)            Strong(kids)
          Strike(kids)                    Link(href, title, kids)   Image(src, alt, title)
          InlineHtml(text)                FootnoteRef(label, index, nth)
```

An `Item`'s `task` is `0` for an ordinary item, `1` for an unchecked task and `2` for a checked one. A `Table`'s `aligns`
holds one of `""`, `left`, `center` or `right` per column. A `Heading`'s `id` is empty when `headingIds` is off. A
`CodeBlock`'s `lang` is the first word of its info string, empty for an indented block. The tree names no framework
(decision 113): the cell [data_formats_are_own_libraries-396](modules/decisions-taken/data_formats_are_own_libraries-396/draft/main.bp)
walks it with a `case` and builds an element type of the program's own, one node per variant.

### Reading the tree

- `headings(doc) -> TocEntry[]` lists every heading of the document in order, the ones inside a quote or a list item
  included. `TocEntry(depth, slug, text)` carries the heading's depth, its id and its text as plain text. A
  table of contents is built from it; the cell
  [own_module_import_without_from-206](modules/decisions-taken/own_module_import_without_from-206/draft/main.bp) prints
  one line per heading, `1 * Guide #guide`.
- `plainText(nodes) -> string` is the text of inlines as an image's `alt` reads it: the markup dropped, the text kept.
- `slug(title) -> string` is GitHub's slug: lower case, spaces to `-`, punctuation and symbols other than `-` and `_`
  dropped. It is the rule heading ids follow, and the one a collection entry's id is made with. Ids are unique per
  document: a second `Conclusion` is `conclusion-1`, a third `conclusion-2`.
- `footnoteId(label) -> string` is the id a footnote's definition and its references are anchored by — the label
  normalised as a URL: `fn-<id>` for the definition, `fnref-<id>` and `fnref-<id>-<nth>` for the references.

An optional the program keeps beside a tree — a source that may be absent, a hook that may be unset — is read with the
operators of `?T`, never with methods (decision 330) —
[optional_typescript_operators-330](modules/decisions-taken/optional_typescript_operators-330/draft/main.bp).

### Writing HTML

`toHtml(doc) -> string` writes the document the way the CommonMark reference renderer does, and the way cmark-gfm does
for the extensions: the specification's 652 examples, the GFM extensions' 24 and the footnotes' 3 are the fixtures of the
repository's tests, one test each. Text and attribute values are escaped (`&`, `<`, `>` and `"`); a destination is
percent-encoded as the reference implementations encode it. A document with referenced footnotes ends with a
`<section class="footnotes" data-footnotes>` holding an ordered list of them. The same call answers the same string at
build and at run time, on both targets.

## md_text

The text layer under the parser. The module is the package's own and is not imported by a consumer; it is described here
because the parser's behaviour is its.

- `Chars(handle, size)` is a string's code points with constant-time reads. `at(i)` answers the code point at `i`, `-1`
  outside `[0, size)`; `text(start, end)` answers the code points `[start, end)` as a string, the bounds clamped.
  `charsOf(s)` builds one. The parser peeks ahead and behind at every character, and a string's own `at` is not constant
  on erlang, so a scan of a 200 kB document would be quadratic without it. These are the module's two host cells.
- `charText(c)` is the one-character string of a code point; a value that is no Unicode scalar value (zero, a surrogate,
  past U+10FFFF) is U+FFFD, as the specification requires of a numeric character reference.
- The character classes of the specification's section 2.1, each over a code point: `isSpaceOrTab`, `isUnicodeSpace`,
  `isAsciiPunct`, `isPunct` (the ASCII punctuation plus the Unicode categories P and S), `isDigit`, `isAsciiAlpha`,
  `isAsciiAlnum` and `isHexDigit`.
- `escapeHtml(s)` escapes `&`, `<`, `>` and `"` — the four the reference renderer escapes.
- `matchEntity(c, pos)` reads an entity or a numeric character reference starting at the `&` at `pos`, and answers the
  tuple of the index after its `;` and the text it stands for, or `-1` when there is none there.
- `unescapeString(s)` replaces backslash escapes and character references by the characters they stand for — a link
  destination, a title, an info string.
- `normalizeLabel(label)` is a link label as references compare it: trimmed, inner white space collapsed to one space,
  Unicode case-folded.
- `normalizeUri(s)` is the URL form of a destination: every character outside the URL-safe set percent-encoded as UTF-8
  with upper-case hex, an existing `%XX` kept.

## md_entities

`entityText(name) -> string` answers the text an HTML5 named character reference stands for, and `""` for a name that is
not one. The table is the 2125 names of the WHATWG `entities.json` that end in `;`, the only form CommonMark recognises;
the module is generated from that file and is the package's own.

## Departures

`src/` still writes some forms that `specs/retired.md` lists; the library's own codemod is pending, and this reference
describes the API in the spelling in force.

- Its types are spelt `Array<MdNode>`, `Array<Footnote>` and `Array<TocEntry>`; the spelling in force is `array<T>`, written
  `T[]` (decisions 641, 617 (1)) — `MdDoc(nodes: MdNode[], footnotes: Footnote[])`, `headings(doc) -> TocEntry[]`.
- `src/root.bp` declares the modules (`pub mod markdown;`, `mod md_text;`, `mod md_entities;`) and the manifest lists
  `files`; every `.bp` under `src/` is a module whose path is its name, with no `mod` / `pub mod` (decision 592 (1)).
- `md_text` calls `String.fromCodepoint`; the static lives on the type, `string.fromCodepoint` (decision 641).

## Decisions

| Directory | What it holds |
|---|---|
| `modules/decisions-taken/` | The ledger's index: one directory `<slug>-<N>/` per decision of [`../decisions-taken.md`](../decisions-taken.md) that fixes program behaviour, each a `docs.md` walking through one example program of the decision (front 166). |
