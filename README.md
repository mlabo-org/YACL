# YACL — Yet Another Control Language

YACL is a lightweight control language to stabilize LLM output via **Flag-Driven, YAML-Gated** rules.

## Features
- **Flag-Driven, YAML-Gated** instruction + validation
- **Mode separation** for clarity (plan / write / compile concepts)
- **Contract + assertions** to auto-check output
- **Template system** for quick presets

## Quickstart (ChatGPT)
1) Upload `/dist/yacl_single-2025-08-12.txt` to the chat  
2) Say: “Apply **extended_markdown/keyword_map_v1** to all responses in this session.”

## Repository Layout
- `yacl_core.yaml`, `yacl_templates.yaml`, `yacl_flag_naming_rules.yaml`
- `LICENSE`, `README.md`, `philosophy.md`, `CONTRIBUTING.md`, `ROADMAP.md`
- `/dist` single-file distribution (txt + yaml)
- `/docs` distribution guides (JA/EN)

## License
MIT License with YACL Custom Clauses (see LICENSE)