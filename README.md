# Code Design Standard (CDS)

A standard for code **inside a file**: how a file reads top to bottom (scenario header, paragraphs, reading depth) and in what order code is placed in it (sections, class members, function module).

The scope of the standard is **one file**. Service folder layout and import direction between files are out of scope.

## Quick Start

1. Read [STANDARD.md](./versions/1.0.0/STANDARD.md) — full text of the current version
2. Use [examples](./versions/1.0.0/STANDARD.md#examples) as a template for your files
3. Check compliance against [criteria](./versions/1.0.0/COMPLIANCE.md)

## Versions

- **Current stable:** [1.0.0](./versions/1.0.0/STANDARD.md)
- **Archive:** [`versions/`](./versions/)

## Contents

- **[versions/1.0.0/STANDARD.md](./versions/1.0.0/STANDARD.md)** — glossary, structure, principles, anti-patterns, examples
- **[versions/1.0.0/COMPLIANCE.md](./versions/1.0.0/COMPLIANCE.md)** — compliance criteria (review checklist)
- **[CHANGELOG.md](./CHANGELOG.md)** — change history

## Applicability

TypeScript backend services. The Readability principles carry over to any imperative language; the Skeleton part is TypeScript-specific.

## Acknowledgements

- John Ousterhout, _A Philosophy of Software Design_ — deep vs shallow modules, conjoined methods, pass-through methods.
- John Carmack, _On inlined code_ (2007) — inline single-call functions.
- Kent Beck, _Tidy First?_ — guard clauses, chunk statements, explaining comments, One Pile.
- Artem Zakirullin, _Cognitive load is what matters_; the htmx essay _Locality of Behaviour_.
- Kent C. Dodds (AHA), Sandi Metz (_The Wrong Abstraction_) — do not abstract in advance.
- G. Ann Campbell, _Cognitive Complexity_ (SonarSource, 2017).

## License

[CC BY 4.0](./LICENSE)
