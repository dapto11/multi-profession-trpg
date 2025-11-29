# SYSTEM_OVERVIEW.md

# 多職種連携TRPG システム概要

**Version 3.2 - 他者評価方式対応版**  
**最終更新: 2025年11月29日**

---

## 1. はじめに

### 1.1 このシステムの目的

多職種連携TRPGシステムは、医療・介護現場における多職種連携の質を向上させるための研修用ゲームシステムです。

**主な目的:**
- 職種間の価値観対立を安全に体験する
- 他職種の視点や専門性への理解を深める
- 現実的な制約の中での意思決定プロセスを学ぶ
- 連携の質を可視化し、振り返りを促進する

**特徴:**
- テーブルトークRPG（TRPG）の形式を採用
- 1セッション約2時間（3ループ × 35分）
- 6-8名のプレイヤー + 1名のGM
- ステータスシステムによる可視化
- プレイヤー主導の他者評価方式

---

### 1.2 対象読者

このドキュメントは以下の方を対象としています:

- **システム開発者**: JSONスキーマの理解、実装、拡張
- **研究者**: 測定方法の理解、データ分析
- **シナリオ作成者**: 新しいシナリオの設計
- **教育プログラム設計者**: 研修プログラムへの組み込み

---

### 1.3 ドキュメント構成

本システムには以下のドキュメントがあります:

- **SYSTEM_OVERVIEW.md**（本書）: システム全体の概要
- **GM_GUIDE.md**: ゲームマスター向けの運用ガイド
- **PLAYER_GUIDE.md**: プレイヤー向けの参加ガイド
- **シナリオ生成プロンプト v3.2**: AIによるシナリオ自動生成

---

## 2. システムアーキテクチャ

### 2.1 7つのJSONスキーマ

本システムは以下の7つのJSONスキーマで構成されています:

#### **2.1.1 scenario_template.schema.json**
- **役割**: シナリオの全体設計を定義
- **内容**: 患者情報、家族構成、職種構成、学習目標、ステータス設定
- **参照**: event_pool（イベントカードID）、constraint_sets（制約セットID）

#### **2.1.2 profession_status_catalog.json**
- **役割**: 職種用ステータスタイプの詳細定義（全シナリオ共通）
- **内容**: 12種類のステータスタイプ、典型的な行動例、自己評価ガイド
- **特徴**: 他者評価方式に対応、具体的な上昇条件は記載しない

#### **2.1.3 event_card.schema.json**
- **役割**: イベントカードの定義
- **内容**: 通常イベント（ID: 1-100）、閾値イベント（ID: 101以降）
- **機能**: ステータスへの影響（status_effects）、職種バランス、方向性バランス

#### **2.1.4 constraint_set.schema.json**
- **役割**: Phase 4（方針決定）時の制約を定義
- **内容**: 時間制約、経済制約、リソース制約、法的・制度的制約
- **レベル**: Easy/Normal/Hardの3段階

#### **2.1.5 dispute_queue.schema.json**
- **役割**: Phase 2（争点マーキング）の結果を保持
- **内容**: 争点リスト、関与職種、対立の強度
- **連携**: Phase 3（価値観表明）への引き継ぎ

#### **2.1.6 measurement_log.schema.json**
- **役割**: セッション全体の記録と測定
- **内容**: 参加者情報、ループごとの記録、ステータス推移、他者評価の記録
- **機能**: 質的評価と量的指標の両立

#### **2.1.7 team_evaluation_rules.json**
- **役割**: チーム評価の計算ルール（全シナリオ共通）
- **内容**: 4つの評価軸の計算式、Relevance指標
- **特徴**: 職種構成に依存しない汎用設計

---

### 2.2 ファイル間の関係性

```
scenario_template.json
  ├─ 参照 → constraint_sets.json（制約セットID）
  ├─ 参照 → event_cards.json（イベントカードID）
  ├─ 参照 → profession_status_catalog.json（ステータスタイプ定義）
  └─ 参照 → team_evaluation_rules.json（チーム評価ルール）

measurement_log.json
  ├─ 記録 ← scenario_template.json（参加者、ステータス初期値）
  ├─ 記録 ← event_cards.json（イベント発生、status_effects）
  ├─ 記録 ← dispute_queue.json（争点の記録）
  └─ 計算 ← team_evaluation_rules.json（チーム評価の計算）

dispute_queue.json
  ├─ 生成 ← Phase 2（争点マーキング）
  └─ 使用 → Phase 3（価値観表明）
```

---

### 2.3 データフロー

#### **セッション開始前:**
1. シナリオ生成プロンプト v3.2 を使用してシナリオを生成
2. scenario_template.json、event_cards.json、constraint_sets.json を準備
3. measurement_template.json を初期化

#### **セッション中:**
1. **Phase 1（情報共有）**: 役職カードの配布、初期状況の説明
2. **Phase 2（争点マーキング）**: dispute_queue.json に争点を記録
3. **Phase 3（価値観表明）**: dispute_queue.json を参照して議論
4. **Phase 4（方針決定）**: constraint_sets.json を参照して制約を提示
5. **Phase 5（イベント・ステータス調整）**:
   - event_cards.json からイベントを引く
   - status_effects を自動適用
   - プレイヤー同士で他者評価（peer_evaluation）
   - measurement_log.json に記録

#### **セッション終了後:**
1. measurement_log.json の完成
2. team_evaluation_rules.json に基づいてチーム評価を計算
3. 振り返り（デブリーフィング）

---

## 3. ゲームシステムの概要

### 3.1 5つのPhaseの流れ

#### **Phase 1: 情報共有（6分）**
- **目的**: 各職種が持つ情報を共有
- **進行**: 各プレイヤーが役職カードの情報を読み上げ
- **ルール**: 質問は可、議論・提案は不可

#### **Phase 2: 争点マーキング（6分）**
- **目的**: 意見の対立点を明確化
- **進行**: 「〇〇について意見が分かれそう」と指摘
- **記録**: dispute_queue.json に記録

#### **Phase 3: 価値観表明（8分）**
- **目的**: 各職種の価値観を表明
- **進行**: 争点ごとに「自分はこう考える」を表明
- **ルール**: 他者の価値観を否定しない

#### **Phase 4: 方針決定（7分）**
- **目的**: チームとしての方針を決定
- **進行**: 制約を踏まえて具体的な方針を決定
- **制約**: constraint_sets.json を参照

#### **Phase 5: イベント・ステータス調整（8分）**
- **目的**: イベント発生とステータス調整
- **進行**:
  1. イベントカードを引く（30秒）
  2. status_effects を自動適用（30秒）
  3. 各プレイヤーが自己申告し、他者評価を受ける（6分）
  4. 患者・家族役も評価を受ける（1分）
  5. ステータスシートに記録（30秒）

---

### 3.2 ループ構造

- **1ループ = 30分**（Phase 1-5の合計）
- **1セッション = 3ループ + 振り返り5分 = 約2時間**
- ループ2以降は Phase 5 の評価が効率化（40秒/人）

---

### 3.3 8つの争点タグ

すべてのイベントカード、争点、制約は以下の8つのタグで分類されます:

1. **safety**: 安全性（急変リスク、事故防止）
2. **time**: 時間的制約（緊急性、期限）
3. **resources**: 資源・費用（予算、人員、機器）
4. **family_burden**: 家族負担（介護負担、経済的負担）
5. **patient_preference**: 本人の希望（自己決定、価値観）
6. **function**: 機能・ADL（リハビリ、自立度）
7. **legal_policy**: 法律・制度（保険制度、法的制約）
8. **coordination**: 連携・調整（情報共有、役割分担）

---

## 4. ステータスシステム

### 4.1 ステータスの種類

#### **4.1.1 患者用ステータス（10種類）**
- physical_condition（身体状態）
- trust_and_acceptance（信頼と受容）
- autonomy_satisfaction（自己決定の満足）
- psychological_wellbeing（心理的安寧）
- pain_control（疼痛コントロール）
- medication_adherence（服薬遵守）
- functional_independence（機能的自立度）
- social_connection（社会的つながり）
- hope_and_meaning（希望と生きがい）
- symptom_burden（症状負担）

#### **4.1.2 家族用ステータス（8種類）**
- caregiver_burden（介護負担）
- hope_for_support（支援への期待）
- psychological_distress（心理的苦痛）
- family_cohesion（家族の結束）
- economic_anxiety（経済的不安）
- information_needs（情報ニーズ）
- coping_capacity（対処能力）
- relationship_quality（関係性の質）

#### **4.1.3 職種用ステータス（12種類）**
- goal_achievement（目標達成度）
- professional_contribution（専門的貢献）
- **collaboration_effectiveness（連携効果）** ← 全職種必須
- patient_safety_assurance（患者安全確保）
- symptom_management（症状管理）
- resource_coordination（資源調整）
- family_advocacy（家族代弁）
- information_provision（情報提供）
- rehabilitation_progress（リハビリ進捗）
- medication_optimization（服薬最適化）
- psychosocial_support（心理社会的支援）
- system_navigation（制度活用支援）

---

### 4.2 1-10段階の評価

すべてのステータスは **1-10の整数値** で表現されます:

- **1-2**: 危機的状態（閾値イベント発生の可能性）
- **3-4**: 問題あり、改善が必要
- **5-6**: 基本的には問題なし
- **7-8**: 良好、適切に機能している
- **9-10**: 非常に良好、目標を大きく超えている

---

### 4.3 ステータス変化の範囲

- **±1**: 基本的な変化（通常イベント、他者評価）
- **±2**: 大きな影響（通常イベント、他者評価）
- **±3**: 劇的な変化（閾値イベントのみ）
- **±0**: 変化なし（迷ったら±0）

---

### 4.4 閾値イベント

患者・家族のステータスが危機的状態（通常は2以下、または8以上）に達すると、**閾値イベント**が自動発生します。

**特徴:**
- 自動発生（auto_occurrence: true）
- 強制的な議論時間（forced_discussion_minutes: 5）
- 必ず回復経路が設定されている（recovery_condition, recovery_event_id）
- ネガティブ・スパイラル防止（cooldown_loops, max_occurrences_per_session）

**例:**
```json
{
  "event_id": 101,
  "title_ja": "患者の治療拒否",
  "threshold_trigger": {
    "condition": "patient.trust_and_acceptance <= 2",
    "auto_occurrence": true,
    "recovery_condition": "patient.trust_and_acceptance >= 4",
    "recovery_event_id": 102
  }
}
```

---

### 4.5 scenario_specific_description

各ステータスには、**シナリオ固有の具体例**（scenario_specific_description）が必ず記述されます。

**例:**
```json
{
  "type": "symptom_management",
  "label": "症状管理",
  "scenario_specific_description": "COPD患者の呼吸困難・SpO2管理、増悪予防のための処方調整、在宅酸素導入の判断"
}
```

これにより、プレイヤーは「このシナリオでは何を評価すべきか」を明確に理解できます。

---

## 5. 他者評価方式

### 5.1 Phase 5 の進行フロー（8分）

#### **ステップ1: イベントカードを引く（30秒）**
- GMがイベントカードを1枚引き、読み上げる

#### **ステップ2: status_effects を自動適用（30秒）**
- イベントカードの status_effects に記載された変化を自動的に適用
- 例: `patient.trust_and_acceptance: -1`

#### **ステップ3: 各プレイヤーが自己申告し、他者評価を受ける（6分）**
- **1人あたり1分 × 6名 = 6分**
- プレイヤーAが自己申告:
  - 「私の『患者安全確保』について評価をお願いします」
  - 「今回のループで、急変リスクを予測し、チームに報告しました」
- 他のプレイヤーが評価:
  - 「+1に賛成」「現状維持」「-1に賛成」など
- 多数決または合意形成で決定

#### **ステップ4: 患者・家族役も評価を受ける（1分）**
- 患者・家族のステータスについても同様に評価

#### **ステップ5: ステータスシートに記録（30秒）**
- 決定した変化を team_evaluation_sheet.csv に記録

---

### 5.2 peer_evaluation_guide の構造

各職種の役職カードには、**peer_evaluation_guide** が含まれています:

```json
"peer_evaluation_guide": {
  "self_declaration_template": "私の『症状管理』について評価をお願いします。今回のループで、〇〇を行いました。",
  "evaluation_examples": [
    "症状を適切に評価し、処方調整を行った → +1に賛成",
    "症状の変化に迅速に対応した → +1に賛成",
    "症状を大きく改善させた → +2に賛成",
    "症状の変化を見逃した → -1に賛成"
  ],
  "evaluation_focus_points": [
    "症状評価の適切性と処方調整のタイミング",
    "他職種との情報共有と連携",
    "患者・家族への説明のわかりやすさ"
  ]
}
```

---

### 5.3 評価方法

#### **5.3.1 consensus（合意形成）**
- 全員が同意した場合のみ変化
- 最も慎重な方法

#### **5.3.2 majority（多数決）**
- 過半数が賛成した場合に変化
- 最も一般的な方法

#### **5.3.3 no_objection（異議なし方式）**
- 誰も反対しなければ変化
- ループ2以降で効率化のために使用

---

### 5.4 評価の原則

- **±1が基本**: 通常の貢献は±1
- **±2は大きな影響**: 顕著な貢献または失敗
- **±3は閾値イベントのみ**: 通常の他者評価では使用しない
- **迷ったら±0**: 判断に迷う場合は変化させない
- **発言回数ではなく貢献の質**: 声の大きさや発言回数で評価しない

---

### 5.5 二重カウント防止

イベントカードの status_effects と他者評価で、**同じ根拠で重複して加点しない**:

- **OK**: イベントで `patient.trust_and_acceptance: -2`、他者評価で `profession.physician.symptom_management: +1`
- **NG**: イベントで `patient.trust_and_acceptance: -2`、他者評価でも `patient.trust_and_acceptance: -1`（同じ理由で）

---

## 6. チーム評価システム

### 6.1 4つの評価軸

チーム評価は以下の4つの軸で計算されます（team_evaluation_rules.json に定義）:

#### **6.1.1 patient_centeredness（患者中心性）**
- **構成**: 患者の体験（信頼、自己決定）60% + 専門職の連携効果 40%
- **意味**: 患者の意思が尊重され、満足度が高いか

#### **6.1.2 safety_assurance（安全保障）**
- **構成**: 患者の身体状態 40% + 専門職の安全確保・症状管理 60%
- **意味**: 患者の安全が確保され、症状が適切に管理されているか

#### **6.1.3 care_sustainability（ケアの持続可能性）**
- **構成**: (10-家族の介護負担) 30% + 家族の支援への期待 20% + 専門職の資源調整 50%
- **意味**: ケアが長期的に持続可能か、家族が疲弊していないか

#### **6.1.4 family_support（家族支援）**
- **構成**: (10-家族の介護負担) 30% + 家族の支援への期待 20% + 専門職の家族支援 50%
- **意味**: 家族が適切に支援され、負担が軽減されているか

---

### 6.2 計算ルール

#### **6.2.1 汎用設計**
- 特定の職種名を参照しない（ステータスタイプのみ）
- 職種が欠けても自動対応（重み再配分）
- 人数・職種構成に依存しない

#### **6.2.2 計算例**

```
患者中心性 = 
  (患者.trust_and_acceptance + 患者.autonomy_satisfaction) / 2 × 0.6 +
  (医師.collaboration_effectiveness + 看護師.collaboration_effectiveness + ...) / 職種数 × 0.4
```

---

### 6.3 Relevance指標

各評価軸には **Relevance指標** が表示されます:

- **計算式**: 参照職種数 ÷ 該当ステータスを持つ職種数
- **表示**:
  - ✓ 完全（3/3職種）: 信頼できる
  - ⚠️ 部分的（1/2職種）: 参考程度
  - ⚠️⚠️ 該当職種なし（0/0職種）: 計算不能

**意味**: 計算結果の信頼性を可視化

---

## 7. 設計思想

### 7.1 価値観対立の体験 → 理解促進（解消ではない）

本システムの核心は、**価値観対立を解消することではなく、体験し理解すること**です。

- 各職種には明確な利害関係と専門性がある
- 対立は「悪」ではなく、多様性の表れ
- 対立を経験することで、他職種への理解が深まる

---

### 7.2 現実的制約の重視（理想論回避）

- Phase 4 で constraint_sets.json に基づいて制約を提示
- 「理想的にはこうすべき」ではなく、「現実的にどうするか」を議論
- 制約の中での最善策を模索する体験

---

### 7.3 測定とゲームプレイの一体化

- プレイヤーのステータス調整が測定を兼ねる
- 測定のための特別な作業は不要
- ゲームの進行そのものがデータになる

---

### 7.4 部分最適 vs 全体最適の可視化

- **職種別ステータス**: 各職種の目標達成度（部分最適）
- **チーム評価**: チーム全体の成果（全体最適）
- ギャップを可視化することで、「自分の職種の目標を達成しても、チーム全体では最適でない」ことを体験

---

### 7.5 プロセス重視測定（結果より体験過程の質）

- 発言回数や時間ではなく、**体験の質**を重視
- 質的評価（振り返り、気づき）を主軸に据える
- 量的指標（ステータス推移、他者評価回数）は運営品質確認用

---

## 8. 拡張性

### 8.1 新しいシナリオの追加方法

#### **ステップ1: シナリオ生成プロンプト v3.2 を使用**
- 事例情報を日本語で入力
- AIが自動的に5つのファイルを生成

#### **ステップ2: 生成されたファイルを検証**
- チェックリストで品質確認
- 必要に応じて手動修正

#### **ステップ3: テストプレイ**
- 実際にプレイして動作確認
- フィードバックを反映

---

### 8.2 カスタマイズのポイント

#### **8.2.1 ステータスの選択**
- 患者: 3-5種類
- 家族: 2-3種類
- 職種: 3-4種類（4種類を推奨、collaboration_effectiveness は必須）

#### **8.2.2 scenario_specific_description の記述**
- シナリオ固有の具体例を必ず記述
- プレイヤーが「何を評価すべきか」を明確にする

#### **8.2.3 peer_evaluation_guide の生成**
- self_declaration_template: ステータスタイプごとに具体的に
- evaluation_examples: ±1と±2の両方を含む
- evaluation_focus_points: 発言回数ではなく貢献の質

#### **8.2.4 イベントカードのバランス**
- 方向性: ネガティブ10枚、ミックス3枚、ポジティブ2枚
- 職種バランス: 各職種が主要関与者として最低3回登場

---

### 8.3 新しいステータスタイプの追加

profession_status_catalog.json に新しいステータスタイプを追加する場合:

1. `label`, `description`, `recommended_use` を定義
2. `typical_actions` を5-7個記述
3. `scale_anchors` で1-10段階の意味を定義
4. `self_assessment_guide` を記述
   - plus_2_description, plus_1_description
   - zero_description
   - minus_1_description, minus_2_description
   - evaluation_prompt
   - adjustment_principle

**重要**: `increase_triggers`（具体的な上昇条件）は記載しない

---

## 9. 技術仕様

### 9.1 JSON Schema バージョン

- **使用バージョン**: JSON Schema Draft 2020-12
- **$schema**: `https://json-schema.org/draft/2020-12/schema`

---

### 9.2 必須フィールド

#### **scenario_template.json**
```json
{
  "required": [
    "scenario_id",
    "title_ja",
    "patient",
    "setting",
    "initial_situation",
    "learning_objectives",
    "phase_flow"
  ]
}
```

#### **event_card.schema.json**
```json
{
  "required": [
    "event_id",
    "category",
    "subcategory",
    "title_ja",
    "body_ja",
    "tags"
  ]
}
```

#### **measurement_log.schema.json**
```json
{
  "required": [
    "session_id",
    "scenario_id",
    "started_at",
    "participants",
    "loops"
  ]
}
```

---

### 9.3 バリデーション

#### **9.3.1 enum値の一貫性**

以下のenum値は全ファイルで一致している必要があります:

- **role**: `physician`, `nurse`, `pharmacist`, `pt`, `ot`, `st`, `care_manager`, `home_helper`, `msw`, `social_worker`, `dietitian`, `assistive_device_specialist`, `public_health_nurse`, `patient`, `family`, `gm`, `other`
- **setting**: `home_care`, `acute_hospital`, `chronic_ward`, `rehab_hospital`, `long_term_care_facility`, `outpatient`, `day_care_rehab`, `hospice`, `school`, `community`, `other`
- **tag**: `safety`, `time`, `resources`, `family_burden`, `patient_preference`, `function`, `legal_policy`, `coordination`

#### **9.3.2 ステータスタイプの一致**

- scenario_template.schema.json の `$defs.profession_status_type` enum
- profession_status_catalog.json の `status_types` キー

この2つは完全に一致している必要があります。

---

### 9.4 ファイル命名規則

- **scenario_template.json**: シナリオ固有
- **constraint_sets.json**: シナリオ固有
- **event_cards.json**: シナリオ固有
- **measurement_template.json**: シナリオ固有（セッション開始前）
- **measurement_log.json**: セッション固有（セッション終了後）
- **team_evaluation_sheet.csv**: シナリオ固有

---

## 10. バージョン履歴

### v3.2（2025年11月29日）- 他者評価方式対応版

**主な変更:**
- **他者評価方式の導入**: Phase 5 でプレイヤー同士が評価
- **peer_evaluation_guide の追加**: 自己申告テンプレート、評価例、評価ポイント
- **ステータス数の変更**: 3種類 → 3-4種類（4種類を推奨）
- **scenario_specific_description の追加**: シナリオ固有の具体例を記述
- **イベントバランス要件の追加**: 職種バランス、方向性バランス（ポジティブ2枚）
- **profession_status_catalog.json の追加**: 職種用ステータスの詳細定義
- **measurement_log への追加**: peer_evaluation_sessions の記録

---

### v3.1（2025年11月9日）- ステータスシステム対応版

**主な変更:**
- ステータスシステムの導入（患者・家族・職種）
- 閾値イベントの実装
- チーム評価システムの実装
- 患者・家族の役職化

---

### v3.0（初版）

**基本機能:**
- 5つのPhaseの実装
- 8つの争点タグ
- イベントカードシステム
- 制約セットシステム

---

## 11. 参考資料

### 11.1 関連ドキュメント

- **GM_GUIDE.md**: ゲームマスター向けの運用ガイド
- **PLAYER_GUIDE.md**: プレイヤー向けの参加ガイド
- **シナリオ生成プロンプト v3.2**: AIによるシナリオ自動生成

---

### 11.2 問い合わせ

システムに関する質問や提案は、開発チームまでお問い合わせください。

---

**以上、多職種連携TRPG システム概要（Version 3.2）**
```

---



