# YACL Distribution Guide (For End Users)
Last updated: 2025-08-12

> **Archived historical guide.** This document records the original 2025 usage instructions. YACL is no longer maintained or supported, and these instructions are not a compatibility guarantee for current ChatGPT models or interfaces.

This document explains **how to use YACL (Yet Another Control Language)** for first-time users without requiring deep technical knowledge.  
Internal implementation terms intended for developers (e.g., `extended_markdown`, `compile`) are **replaced with user-friendly names** where possible.

---

## 1. What Can YACL Do?
- YACL is a lightweight control language for LLMs using **flag-driven + YAML-gated** rules to stabilize output.
- Simply upload a **single file (.txt)** to ChatGPT's chat window → tell it which **Template ID** to apply, and YACL will automatically format output such as tables, paragraph structure, and term styling.
- The package includes a **beginner-friendly template** and an **English-focused strict template**.

---

## 2. Terminology (User-Friendly Terms)
| User-Friendly Term | Purpose | Internal Identifier (Reference) |
|---|---|---|
| **Creation Mode** | The stage for generating and formatting text | `extended_markdown` |
| **Build Mode** | The stage for enforcing structure checks and strict formatting | `compile` / validation layer |
| **Template ID** | A preset that bundles specific flags for a purpose | `extended_markdown/keyword_map_v1` etc. |

> *Note:* Internal identifiers only appear in configuration files. End users only need to know the **User-Friendly Term**.

---

## 3. What's Inside the Single-File Distribution?
The distributed package contains **core** and **templates** merged into a single file.  
Upload the `.txt` version directly to ChatGPT.

- `yacl.version`: Version number (e.g., `1.1`)
- `params`: Shared parameters (e.g., `min_keywords_per_set: 5`)
- `flags`: Legacy flags (to be phased out)
- `modes.logic_flags`: Main logic rules (**Creation Mode**)
- `contract` / `assertions`: Output contract and validation rules
- `templates`: Template IDs (**standard** / **strict**)

> For reference, here is a preview (excerpt only, no editing required):

---

<details>
<summary>Preview of Distributed File (Excerpt)</summary>

```
yacl:
  version: 1.1
  lang: auto   # 実行時推定

  params:
    min_keywords_per_set: 5
    glossary_lock_ja:
      "EVP": "価値提案（EVP）"
      "KPI": "成果指標（KPI）"

  # 互換用の従来フラグ（将来的に構造へ吸収予定）
  flags:
    translate_to_inferred_locale: true
    prohibit_html_tags: true
    render_keyword_table_vertical: true
    boldify__all_keywords__strict: true
    enforce_parabreak_every__2_sentences: true
    opening_style__experiential__required: true
    require_all_keywords_in__introduction: true

  # --- モード別ロジック：命名＝挙動（優先適用） ---
  modes:
    extended_markdown:
      logic_flags:
        precedence: logic_flags_override_generic_flags

        # ロケール判定と翻訳
        force_gpt_to_translate_to_inferred_locale_when_lang_auto:
          strategy: majority_charcount
          cultural_term_override: true
          tiebreaker: prefer_japanese_when_mixed

        # 体験的オープニングの定義
        require_gpt_to_open_with_experiential_scene_element:
          min_experiential_sentences: 1
          must_include_sensory_or_action_cue: true

        # キーワードの自然挿入ルール
        require_gpt_to_include_all_keywords_naturally_with_semantics_preserved:
          allow_inflection_and_compounds: true
          forbid_semantic_shift: true
          forbid_plain_list_dump_in_intro: true

        # カテゴリ命名の拘束
        require_gpt_to_define_keyword_categories_as_logical_facets:
          examples_allowed: ["メインキーワード","共起語","関連ワード"]
          min_categories: 3
          min_terms_per_category: ${params.min_keywords_per_set}

        # 段落改行のカウント方式
        enforce_parabreak_every_two_sentences_by_lang_delimiters:
          ja_sentence_delimiter: "。"
          en_sentence_delimiter: "."
          treat_semicolon_as_inline: true

        # glossary_lock の出力規範（英語出力時の挙動を追加）
        require_gpt_to_pair_glossary_lock_terms_with_original_a
... (truncated) ...
```
</details>

---

## 4. Naming = Behavior (YACL Naming Rules)
In YACL, **the flag name must directly describe the behavior** (no vague terms).

- Recommended format: `<action_verb>_<subject>_<object>_<condition>`
  - Examples: `force_gpt_to_render_vertical_keyword_table`  
               `require_gpt_to_open_with_experiential_scene_element`
- Recommended prefixes: `force_gpt_to_`, `require_gpt_to_`, `enforce_`
- Avoid vague names: `beautify_text`, `more_seo`, `make_it_better`, etc.

This ensures that **reading the name reveals exactly what it does**.

---

## 5. Key Active Rules (Examples)
- **Locale detection & translation**: Detects language, prefers Japanese when mixed.
- **Experiential opening paragraph**: First paragraph must include at least one **sensory or action-based description**.
- **Natural keyword insertion**: Keywords must appear naturally, not as a plain list.
- **Vertical keyword table**: 3 rows × at least 5 terms each (auto-retry if it fails).
- **Glossary lock**: Terms like **Value Proposition (EVP)** are bold and paired (English behavior configurable per template).
- **Paragraph formatting**: Break paragraphs based on language-specific sentence delimiters.

These are implemented as `logic_flags` and take precedence over legacy `flags`.

---

## 6. Contract & Validation
Every output is automatically checked against the contract.

- **Contract essentials**
  - H1 title → keyword table → introduction paragraph in that order
  - Keyword table: **3 rows × ≥5 terms each**
  - Introduction must include **all keywords naturally**

- **Validation examples**
  - `h1_count == 1` / `table_count == 1` / `row_count_in_first_table == 3`
  - `all_rows_have_at_least__{min_keywords_per_set}_terms`
  - `no_html_tags_in_output` / `lang == inferred_locale`

---

## 7. Failure Recovery (Auto-Retry)
- Table rendering: Retry in order `vertical-strict → vertical-relaxed → horizontal`
- Language mismatch: Retry until matched; final force to inferred locale
- English output glossary: **standard** = paired, **strict** = suppressed

---

## 8. Template ID Options
- **extended_markdown/keyword_map_v1** (Standard)  
  Experiential intro + vertical keyword table + paired glossary (EVP, etc.)
- **extended_markdown/keyword_map_strict_v1** (English Strict)  
  Suppresses glossary pairing in English + stronger table recovery

> Usage (copy & paste in ChatGPT):  
> "Apply **extended_markdown/keyword_map_v1** to all responses in this session."

---

## 9. Uploading to ChatGPT
1. Drag & drop `yacl_single-2025-08-12.txt` into the chat window  
2. Tell ChatGPT the Template ID (standard or strict)

> `.yaml` may not be accepted by ChatGPT UI, so **use `.txt`**.

---

## 10. License
- **MIT License** — see the repository's `LICENSE` file.

---

## Appendix A: Developer Rules (Simplified for Distribution)
- **Name = Behavior**; vague terms are prohibited.  
- Prefer structural refactor before adding new flags.  
- Added flags must have **scope & temporality** documented and be designed for future absorption/removal.  
- Always output a **fully merged definition** (not just diffs).  
- Mode-specific instructions must stay in their corresponding mode section.  
- Keep distributed filenames identical for safe overwrite by users.

---

The steps above are preserved as the original 2025 usage record.
