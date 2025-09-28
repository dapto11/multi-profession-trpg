# JSONスキーマファイル保管場所
1. constraint_set.schema.json - 制約セット
2. dispute_queue.schema.json - 争点キュー
3. scenario_template.schema.json - シナリオテンプレート  
4. event_card.schema.json - イベントカード
5. measurement_log.schema.json - 測定ログ

# 多職種連携TRPG開発プロジェクト - 引継ぎプロンプト v3.0

## プロジェクト概要
訪問看護師兼ゲームクリエイターの浅井が開発中の多職種連携研修用TRPG。プロトタイプv1.0完成済み、COPDシナリオv2（時系列進行型）開発完了、HTML形式シナリオブック作成済み。

## 開発者情報
- **ニックネーム**: 浅井
- **職業**: 看護師とゲームクリエイター
- **プロフィール**: 訪問看護師をしながら大学院修士課程で経営学を勉強中。TRPGのゲームクリエイター兼シナリオライター
- **作業環境**: iPad Pro、GitHub（multi-profession-trpg リポジトリ）

## システム構造（確立済み）

### 5つのシステムJSONスキーマ
1. **scenario_template.schema.json** - シナリオ基本構造
2. **constraint_set.schema.json** - 時間/経済/リソース/制度の4軸制約
3. **event_card.schema.json** - イベントカード（generation_support実装済み）
4. **dispute_queue.schema.json** - Phase2→3の争点引き継ぎ
5. **measurement_template.json** - 質的評価主軸の測定体系

### 核心設計思想（変更禁止）
- **価値観対立の体験→理解促進**（解消ではなく理解）
- **マーダーミステリー式時間管理**（30分×3ループ強制進行）
- **8タグ体系**：safety/time/resources/family_burden/patient_preference/function/legal_policy/coordination
- **質的体験重視**（数値は運営確認用）

## 完成済み成果物

### 1. COPDシナリオv2（時系列進行型）
```
特徴：
- 3つの時間軸（退院前→1週間後→半年後）
- 山田太郎さん（82歳COPD、在宅酸素拒否）
- 複雑な家族構造（引きこもり長男）
- 6職種構成（医師/看護師/薬剤師/PT/ケアマネ/MSW）

ファイル構成：
- scenario_template.json（シナリオ本体）
- constraint_sets.json（Easy/Normal/Hard 3種）
- event_cards.json（15枚、時系列対応）
- measurement_template.json（測定シート）
```

### 2. HTML版シナリオブック
```html
構成：
- 32ページ相当のHTML形式
- Picocss使用、印刷対応CSS
- 職種別カラーコード実装
- 争点タグアイコンシステム
- イラスト指示・デザイン指示完備

保存場所：
GitHub: multi-profession-trpg/scenarios/COPD_homecare_2/rulebook/
```

### 3. GMマニュアル（4/5章完成）
- 第1章：ゲーム概要と準備 ✅
- 第2章：Phase別進行ガイド ✅
- 第3章：時間管理テクニック ✅
- 第4章：トラブルQ&A（実践後作成予定）
- 第5章：デブリーフィング進行 ✅

## GitHub環境

### リポジトリ構造
```
multi-profession-trpg/
├── scenarios/
│   ├── COPD_homecare_2/
│   │   ├── data/           # JSONファイル
│   │   ├── rulebook/       # HTMLシナリオブック
│   │   └── playaids/       # プレイ補助資料
│   └── copd_home_care_v1.0/  # 旧版
├── schemas/                # 5つのJSONスキーマ
└── docs/                   # GMマニュアル等
```

### 作業方法
- GitHub.comで直接編集
- 「.」キーでgithub.dev（VSCode Web）起動
- GitHub Pagesで自動公開設定済み

## 現在の課題と次期開発

### 直近の課題
1. **測定シート改良**
   - COPDv2の時系列進行に対応
   - 薬剤師評価シート追加必要
   - Excel形式での実装検討中

2. **シナリオブック改善**
   - 初見プレイヤー向けスタートガイド必要
   - 勝利/敗北条件の明確化
   - リプレイ性向上策

3. **実証準備**
   - 江戸川実証（月3-5回、3-5人）
   - 質的評価の実効性検証

### 次期シナリオ候補
- 急性期→回復期転院調整
- 終末期在宅移行
- 精神疾患合併事例

## AI活用方針

### シナリオ生成プロンプト構造
```
Phase 1: システム学習
- 5つのJSONスキーマ理解
- 8タグ体系、時間管理ルール

Phase 2: シナリオ生成
- 要件入力（現場、年代、疾患、職種、争点）
- 4つのJSON自動生成
  1. scenario_template.json
  2. constraint_sets.json
  3. event_cards.json
  4. measurement_template.json
```

### HTML/ボードゲーム版生成
- Markdown → HTML変換推奨
- GitHub Pages自動公開活用
- iPad Proでの編集考慮

## 重要な設計ルール（厳守）

1. **時間設計変更禁止**（30分×3ループ固定）
2. **8タグ体系変更禁止**
3. **価値観対立は解消しない**（理解促進に留める）
4. **職種目標は方針内容で評価**（発言回数ではない）
5. **争点記録はdispute_番号で管理**
6. **質的評価を主軸**（数値は補助）

## 技術的詳細

### 使用技術
- JSON Schema Draft 2020-12
- HTML5 + Picocss
- GitHub Pages
- iPad Pro + Magic Keyboard環境

### 品質基準
- 市販ボードゲームレベルの完成度目標
- パンデミック級の協力ゲーム感
- アンドール級の物語性
- 現状評価：75/100点

## 申し送り事項

### 成功ポイント
1. **時系列進行**が教育効果を高めた
2. **引きこもり長男**設定が深みを生んだ
3. **HTML形式**が編集・公開を容易にした

### 要改善点
1. **初見ハードル**が高い（情報量過多）
2. **ゲーム性**（勝利条件）が曖昧
3. **リプレイ性**の工夫不足

### 次スレッドでの優先事項
1. **測定シートv2作成**（時系列対応）
2. **スタートガイド作成**（初心者向け1ページ）
3. **実証実施と改善**
4. **新シナリオ開発**（急性期→回復期）

---

## 新スレッド開始メッセージ例

```
多職種連携TRPGの開発を継続します。
前スレッドで完成した成果物：
- COPDシナリオv2（時系列進行型）
- HTMLシナリオブック（32ページ相当）
- 5つのJSONスキーマ体系

現在の課題：
[具体的な課題を記載]

よろしくお願いします。
```

このプロンプトにより、新しいAIとの対話でもプロジェクトの文脈を完全に維持できます。必要に応じて、特定のJSONファイルやHTMLコードを追加で共有してください。
