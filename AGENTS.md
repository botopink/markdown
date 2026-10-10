# markdown

> Repository: `botopink/markdown` (`git@github.com:botopink/markdown.git`) · in the meta checkout: `repository/markdown/`

Markdown read into a tree of its own (decision 396): `Heading`, `Paragraph`, `Emphasis`, `Strong`, `Code`, `Link`, … — naming no framework (113). Moved from onze-content's reader by 142 step 3, which maps the tree to jhonstart's `Element`. One consumer today: the maintainer's exception to 115.

A library of its own (decision 396, decision 326's rule). It is filled by
`specs/1.0.12-beta/03-bundled-libs/142-data-formats`; until its step lands this repository holds
only its scaffold.

## Hooks

Every clone runs the shared pre-commit gate:

```sh
git config core.hooksPath scripts/git-hooks
```

`scripts/git-hooks/pre-commit` and `scripts/git-hooks/lib/runner-standalone.sh` are byte-identical
across the library repositories (the meta repository's CI check 4); edit them in one place and copy.
