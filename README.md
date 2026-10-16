# SystemVerilog / UVM Style Guide

A coding style guide for SystemVerilog and UVM verification code: testbenches, UVM
environments and VIP. RTL is out of scope.

It is written for two readers:

- **Verification engineers**, who use it for new code, code reviews and onboarding.
- **AI coding assistants**, which can load it as context and apply its rules.

Every rule has a stable ID (for example `NAM-4` or `SEQ-8`) and a severity keyword
(**MUST**, **SHOULD**, **MAY**). Reviews, lint waivers and AI prompts can cite a rule by ID.

The guide comes from years of production ASIC/SoC verification work.

**Read the guide:** [`sv_uvm_style_guide.md`](sv_uvm_style_guide.md)

> Status: **v0.3, public draft.** Rules may still change before v1.0.
> Rule IDs are never renumbered. Retired rules keep their IDs.

## What is covered

Formatting, naming, files and packages, comments, class layout, language usage,
components and phases, factory and macros, configuration, interfaces, transactions,
sequences, objections, TLM, checking and scoreboards, messaging, coverage, the register
layer, tests, and reuse. It also includes a complete minimal agent example, a review
checklist, and a table of the decisions where common industry practice disagrees.

## Using it with AI assistants

Point your assistant at the single file `sv_uvm_style_guide.md`. For example, reference
it from your project's `AGENTS.md`, `CLAUDE.md`, or Copilot instructions file. Section 0
of the guide has specific instructions for AI assistants.

## Contributions

The repository is read-only for now, and pull requests are not accepted.
You are welcome to open an issue to report a mistake or ask a question.

## License

Copyright 2026 HEXK.

- **Guide text:** [Creative Commons Attribution 4.0](LICENSE-docs) (CC BY 4.0).
  You may share and adapt it, including commercially, if you give attribution.
- **Code** (examples in the guide and any code files in this repository):
  [Apache License 2.0](LICENSE).

See [NOTICE](NOTICE) for acknowledgements.
