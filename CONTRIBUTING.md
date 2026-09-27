# Contributing

This repository is the Vizanix Quant Engineering Library: a set of original books on algorithmic trading engineering, written entirely by Vizanix. It is not a curated list of external resources, and it does not accept links to third-party books, courses, blogs, or projects. Contributions here are about improving Vizanix's own material, not adding outside content.

## What you can contribute

- **Corrections** — factual errors, broken internal links, typos, unclear explanations, outdated code/pseudocode in an existing book.
- **Translations** — porting an existing book between English and Russian, or adding a new language, while preserving the original meaning and the Vizanix voice.
- **Diagrams** — new or improved SVG illustrations for a chapter, following the visual style already used in `library/assets/`.
- **New book proposals** — a gap in the library (a topic at a given level that isn't covered yet). Proposals are welcome; the actual writing is done by the Vizanix team to keep the voice, structure, and licensing consistent across the library.
- **Tooling** — improvements to the link-check and lint workflows, or to the catalog generation in the README.

## What you cannot contribute

- Content copied or closely paraphrased from any external book, article, or paper.
- Links to, or summaries of, third-party books, courses, trading tools, or projects, however good they are. This library only contains Vizanix's own writing.
- Trading signals, strategy claims, profit projections, or anything implying guaranteed returns.
- AI-generated filler with no substantive correction — a PR should fix something specific, not pad word count.

## How to submit a correction or translation

1. Open a pull request against the relevant file under `library/en/` or `library/ru/`.
2. Describe what was wrong and what you changed, referencing the section or chapter.
3. Keep the existing file structure: title, byline, abstract, table of contents, numbered chapters, summary, license footer.
4. If translating, place the new file in the mirror location on the other language tree, using the same directory (`beginner`, `intermediate`, or `professional`) and a matching slug.

Use the [pull request template](.github/pull_request_template.md) — it walks through the checklist above.

## How to propose a new book

Open an issue using the **Suggest a book topic** template, describing the gap: the level (beginner, intermediate, professional), the subject, and why it belongs in the library. Vizanix maintainers decide what gets written and when.

## License and attribution

All content is released under [CC BY 4.0](LICENSE). By contributing a correction, translation, or diagram, you agree that your contribution is licensed under the same terms and becomes part of the Vizanix-authored library.

## Code of Conduct

By participating in this project, you agree to abide by the [Code of Conduct](CODE_OF_CONDUCT.md).
