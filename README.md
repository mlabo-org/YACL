# YACL — Yet Another Control Language

> **Archived research artifact (2025).** YACL preserves a GPT-4o-era experiment in flag-driven, YAML-gated control of LLM output. It is no longer maintained, is not a current implementation, and carries no compatibility guarantee for current models. The author's current systems have moved to natural-language contracts.
>
> No support, issue handling, pull requests, modification requests, or compatibility fixes are accepted. The quickstarts, distribution guides, and files under `dist/` are retained as historical documentation, not as a currently supported installation route.

YACL was designed as a lightweight control language for stabilizing LLM output through **Flag-Driven, YAML-Gated** rules.

## Historical features

- **Flag-Driven, YAML-Gated** instructions and validation
- **Mode separation** for plan, write, and compile concepts
- **Contracts and assertions** for checking output
- **Templates** for reusable presets

## Historical repository set

These repositories form one historical record:

- **[structci-core](https://github.com/mlabo-org/structci-core)** — StructCI v8.5 reference implementation
- **[YACL](https://github.com/mlabo-org/YACL)** — control-language, contract-format, and distribution artifacts (this repository)
- **[structci-proof](https://github.com/mlabo-org/structci-proof)** — authorship, provenance, and prior-art record

## Repository layout

- `yacl_core.yaml`, `yacl_templates.yaml`, `yacl_flag_naming_rules.yaml`
- `philosophy.md`
- `dist/` — original single-file distribution artifacts
- `docs/` — original Japanese and English distribution guides
- `CONTRIBUTING.md`, `ROADMAP.md` — archival status records

## License

MIT License. See [LICENSE](LICENSE).
