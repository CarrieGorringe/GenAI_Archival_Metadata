# MovieReviewMetaDataExtract

Extract structured **metadata** from free-flow, unstructured text according to a **defined schema**.

The long-term goal is to build structured data from **media (film and audio) preservation notes and archival notes** — the kind of loosely-formatted prose that archivists and curators write by hand. Movie reviews are used here as a convenient, richly-described **test corpus** while the extraction approach and schema are developed; the intended production domain is archival/preservation metadata.

## How it works

The pipeline takes unstructured text in and produces schema-conformant structured records out:

```
unstructured text  ──▶  extractor  ──▶  structured metadata (schema-validated)
   (review prose,         (schema-           (e.g. schema.org-style
    archival notes)        driven)            Movie / Review records)
```

A target schema defines the fields to populate (for the movie corpus: title, year, director, cast, production company, etc.). The extractor reads the prose and emits a record conforming to that schema, which can then be validated against the corpus's known-good values.

## Test corpus: Nitrate Online reviews

Development and evaluation use an archive of ~2,000 film reviews from **[nitrateonline.com](https://nitrateonline.com)**, written by Carrie Gorringe and owned by the project authors.

- Source repository: [`ScottThurlow/nitrateonline`](https://github.com/ScottThurlow/nitrateonline)
- Format: HTML files organized by year (e.g. `1996/rcasino.html`); `r`-prefixed files are reviews, `f`-prefixed files are festival coverage.
- Each review is free-flow prose. A number of files also embed **schema.org `Review` / `Movie` JSON-LD** (title, year, director, actors, production company, poster image), which serves as a useful **ground-truth target** for measuring extraction accuracy.

## Status

Early setup. The repository currently contains project scaffolding (license, code ownership, contribution rules). Schema definitions, the extractor, and evaluation tooling are to follow.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). In short: the code owners (Carrie Gorringe and Scott Thurlow) commit directly; everyone else opens a pull request that must be approved by a code owner.

## License

© Carrie Gorringe and Scott Thurlow. Licensed under
**[Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International (CC BY-NC-ND 4.0)](https://creativecommons.org/licenses/by-nc-nd/4.0/)** — see [LICENSE](LICENSE).

This is the "don't reuse" license: you may view and share the material **with attribution**, but **no commercial use** and **no derivative/modified versions** may be distributed without permission. The Nitrate Online review texts remain the property of their author and are included for research and evaluation only.
