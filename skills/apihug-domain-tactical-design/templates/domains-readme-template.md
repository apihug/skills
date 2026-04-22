# Domain Tactical Design

This directory stores macro-domain DDD tactical design artifacts.

## Standard Domain Package

Each domain may contain:

- `tactical-design.md`
- `01-events-commands-terms.md`
- `02-domain-rules.md`
- `03-uml.md`
- `04-glossary.md`
- `05-architecture.md`
- `06-physical-data-model.md`

Optional supporting directories:

- `decisions/`

## Usage

- `tactical-design.md` is the primary fact source for a domain.
- `01~06` are derived standardized artifacts.
- When design changes, update `tactical-design.md` first, then sync `01~06`.
- `03` and `06` keep Mermaid source only by default; rendered images are not required by the standard contract.
