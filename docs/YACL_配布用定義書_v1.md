# YACL 配布用定義書（ユーザー向け）
最終更新: 2025-08-12

> **歴史資料としてアーカイブされています。** この文書は2025年当時の利用手順を記録したものです。YACLは現在保守・サポートされておらず、現在のChatGPTモデルや画面での互換性を保証するものではありません。

この文書は YACL（Yet Another Control Language）を**これから試す人が迷わず使えるように**まとめた配布用の定義書です。  
内部実装名や開発者向けの専門用語（例: `extended_markdown`, `compile`）は**極力使わず**、必要な場合はユーザー用の呼び名に言い換えて説明します。

---

## 1. まず何ができる？（概要）
- YACLは **フラグ駆動 × YAMLゲート** でLLMの出力を安定化する軽量な制御言語です。
- 1ファイル（.txt）を **ChatGPTのチャットにアップロード** → 「テンプレIDを適用」と伝えるだけで、表形式・段落構成・用語表記などを自動で整えます。
- 配布物には **はじめての人向けテンプレ** と **英文強化テンプレ（strict）** を同梱しています。

---

## 2. 用語（ユーザー向け表現）
| ユーザー向けの呼び方 | 役割の説明 | 実装での識別子（参考） |
|---|---|---|
| **作成モード** | 文章の生成と整形を行う段階 | `extended_markdown` |
| **ビルドモード** | 出力の構成チェックや整形を厳密に適用する段階 | `compile` / 仕様上の検査層 |
| **テンプレID** | 目的に合わせてフラグを束ねたプリセット | `extended_markdown/keyword_map_v1` など |

> ※ 実装名は設定ファイル内部でのみ使用されます。ユーザーは上の **呼び方** を覚えておけば十分です。

---

## 3. 単一ファイル配布の中身
配布パッケージには「core」と「templates」を**1ファイルに統合**したものが含まれます。  
このファイルを **.txt のまま** ChatGPT にアップロードしてください。

- `yacl.version`: バージョン（例: `1.1`）
- `params`: 共有パラメータ（例: `min_keywords_per_set: 5`）
- `flags`: 互換用の旧フラグ（徐々に廃止予定）
- `modes.logic_flags`: 主要ロジック（**作成モード**で有効）
- `contract` / `assertions`: 出力構造の契約と検証
- `templates`: テンプレIDの定義（**標準** / **strict**）

> 参考として、配布ファイルの実体（抜粋プレビュー）を下に示します（編集不要・読み物としての確認用）。

---

<details>
<summary>配布ファイルのプレビュー（抜粋）</summary>

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
...（中略）...
```
</details>

---

## 4. 命名＝挙動（YACLの命名規範）
YACLでは、**名前がそのまま挙動を表す**ように設計します（曖昧語禁止）。

- 推奨形式: `<動作動詞>_<主語>_<対象>_<条件句>`
  - 例: `force_gpt_to_render_vertical_keyword_table`  
        `require_gpt_to_open_with_experiential_scene_element`
- 推奨プレフィックス: `force_gpt_to_`, `require_gpt_to_`, `enforce_`
- NG例: `beautify_text`, `more_seo`, `make_it_better` など **意味が曖昧**な名称

この規範により、**設定を読めば振る舞いが理解できる**状態を保ちます。

---

## 5. 実際に効く主なルール（抜粋）
- **ロケール推定＆翻訳**: 文章の言語を自動判断し、日本語を優先（混在時のタイブレークあり）。
- **導入段落の体験描写**: 最初の段落に**情景や行動の手触り**を1文以上入れる。
- **キーワード自然挿入**: 指定語を**リスト羅列ではなく自然文**として組み込む。
- **キーワード表の縦持ち**: 3行×各5語以上の表を作る（失敗時は自動で再試行）。
- **用語ロック（glossary lock）**: 例：**価値提案（EVP）** のように対表記＆太字化（英語モードはテンプレで変更可能）。
- **段落整形**: 言語区切り（。/ .）に基づく段落分割。

> これらは `logic_flags` として実装されています。**作成モード**で適用され、旧 `flags` より優先されます。

---

## 6. 契約（contract）と検証（assertions）
出力は自動で**契約に合っているか**検査されます。

- **契約の要点**
  - 見出しは **H1が1つ**、その後に**キーワード表**、続いて**導入段落**の順
  - 表は **3行 × 各5語以上**
  - 導入段落に**全キーワードを自然挿入**

- **検証の例**
  - `h1_count == 1` / `table_count == 1` / `row_count_in_first_table == 3`
  - `all_rows_have_at_least__{min_keywords_per_set}_terms`
  - `no_html_tags_in_output` / `lang == inferred_locale`

---

## 7. 失敗時のリカバリ（自動再試行）
- 表レンダリング: `vertical-strict → vertical-relaxed → horizontal` の順で再試行
- 言語不一致: 一致するまで再試行し、最終的に推定ロケールへ強制
- 英文時の対表記: **標準**は対表記、**strict**は抑止（テンプレで切替）

---

## 8. テンプレIDの使い分け
- **extended_markdown/keyword_map_v1**（標準）  
  体験導入 + 縦型キーワード表 + 対表記（EVPなど）
- **extended_markdown/keyword_map_strict_v1**（英語強化）  
  英語出力時は対表記を**抑止**し、表レンダリングの回復手順を強化

> 使い方（チャットでコピペ）  
> 「このセッションの全回答に **extended_markdown/keyword_map_v1** を適用してください。」

---

## 9. 単一ファイルのアップロード手順（ChatGPT）
1. `yacl_single-2025-08-12.txt` をチャットにドラッグ＆ドロップ  
2. 上のテンプレIDを伝える（標準 or strict）

> `.yaml` はUIで読めない場合があるため、**.txt の使用を推奨**します。

---

## 10. ライセンス
- **MIT License** — リポジトリの `LICENSE` を参照してください。

---

## 付録A: 開発者向けルール（配布用に簡約）
- **命名＝挙動**。曖昧語は禁止。  
- 新挙動は**構造リファクタを優先**し、フラグ追加は例外。  
- 追加フラグは**時限性とスコープ**を明記し、将来的な吸収/削除を前提に。  
- 変更は**完全統合出力**で提示（差分ではなく全体定義）。  
- モード依存の命令は**該当モードの節にのみ**記述し、混在させない。  
- 配布物は**同名ファイル**で更新（ユーザーが安全に上書きできるように）。

---

以上は、2025年当時の利用手順として保存されています。
