# Decision 594 — Every library runs at build on the wasm comptime runtime (434), jhonstart first; a function only Erlang or Node can run is a located error there.

The program renders one note twice: `comptime render(…)` runs `markdown`'s `parse` and `toHtml` at build, on the wasm comptime runtime, whatever `--target` the program is built for, and splices the HTML as a literal; `render(source)` runs the same calls at run time on the program's target. The first lines printed are the note as HTML — the heading with its GitHub id, `release-notes`, and the paragraph with `<em>reader</em>` —; the last, `true`, says the two answers are one string.

The reader's few host cells — `md_text`'s `charsOf`, `Chars.at`, `Chars.text` and `isWidePunctHost`, code-point reads and a Unicode class written in Erlang and JavaScript for convenience — are pure, so each has a wasm form, and the build reaches them through it. A function only Erlang or JavaScript can run, reached by that `comptime` line, would be refused at build, located, naming that function — never run on the target's VM.

Cells of this repository that prove it: none yet. The library's suites run `parse` and `toHtml` on erlang and commonJS (`test/markdown_test.bp`, `test/commonmark/`), not at build; the cell comes with 158's Conformance step.

[draft/main.bp](draft/main.bp)
