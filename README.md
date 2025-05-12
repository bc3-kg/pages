# 詳細実装計画: TOGAF・C4・クリーンアーキテクチャ統合フレームワーク

## 1. はじめに

本文書は、先に提示された「Mastra.AI Dynamic AgentのRuntime Contextと役割詳細および実装仕様」を基盤とし、ユーザーからの追加要件（2025年5月12日提供の添付ファイル）を反映して、TOGAF、C4モデル、クリーンアーキテクチャを統合した大規模組織向けIT戦略・実装フレームワークの**詳細な実装計画**を記述するものです。特に、Mastra.AI（TypeScriptベース）とDSPy（Pythonベース）の連携を強化し、各階層でのAIエージェントの能力を最大限に引き出すための具体的な実装方針、技術的詳細、ワークフロー、連携プロトコル、コードレベルの統合例に焦点を当てます。

前回の設計案で概説した「Enterprise-to-Code Architecture (E2CA)」のコンセプトを継承しつつ、より実践的で具体的な実装ステップと技術的選択肢を提示します。

## 2. 改訂版 Runtime Contextの設計方針

前回のRuntime Contextの設計をベースに、DSPyとの連携強化と詳細な実装ニーズに対応するため、以下の項目を強化・追加します。

*   **Agent ID**: 変更なし。
*   **Agent Role**: 変更なし。
*   **Current Goal**: 変更なし。
*   **Task Queue**: 変更なし。
*   **State**: 変更なし。
*   **Knowledge Base Access**: 変更なし。
*   **Input Data**: 変更なし。
*   **Output Data**: 変更なし。
*   **DSPy Integration Config**: より詳細化。
    *   `dspy_bridge_endpoint`: REST APIまたはgRPCブリッジサービスのエンドポイントURL。
    *   `dspy_auth_token`: ブリッジサービス認証用トークン。
    *   `available_dspy_optimizers`: 利用可能なDSPyオプティマイザリスト（例: `["BootstrapFewShot", "BayesianSignatureOptimizer", "MIPRO"]`）。
    *   `available_dspy_modules`: 利用可能なDSPyモジュールリスト（例: `["ChainOfThought", "Planner", "Retrieve"]`）。
    *   `dspy_telemetry_config`: DSPy実行ログをMastra.AIのOpenTelemetryに連携するための設定。
*   **Collaboration Context**: 変更なし。
*   **Historical Context**: 変更なし。
*   **Current DSPy Optimization Strategy**: 現在のタスクやコンテキストに応じて選択されているDSPy最適化戦略（例: `BootstrapFewShot`のパラメータ設定、`MIPRO`の試行回数など）。
*   **DSPy Invocation History**: 当該AgentによるDSPyブリッジAPIの呼び出し履歴（リクエスト、レスポンス、実行時間、成功/失敗ステータス）。これはデバッグとパフォーマンス分析に役立ちます。

## 3. Dynamic Agentの生成とライフサイクル管理（変更なし）

前回の設計内容を維持します。

## 4. 各階層におけるDynamic Agentの役割と連携詳細（詳細実装版）

ここでは、前回の設計を基に、各Agentの役割、振る舞い、そして特にMastra.AIとDSPyの連携について、より具体的な実装レベルで詳細化します。

### 4.1. 戦略層 (TOGAFベース)

#### 4.1.1. エンタープライズアーキテクトAgent (EnterpriseArchitectAgent)

*   **Runtime Contextの主要項目（改訂）**:
    *   `Agent Role`: "EnterpriseArchitectAgent"
    *   `Current Goal`: 例: 「フェーズA: アーキテクチャビジョンの策定支援（DSPyによる質問最適化と戦略オプション生成を含む）」
    *   `Knowledge Base Access`: TOGAF標準、ArchiMateメタモデル、業界リファレンスアーキテクチャ、組織ビジネス戦略文書、過去戦略事例、**DSPy最適化済みプロンプトライブラリ**。
    *   `DSPy Integration Config`: プロンプト最適化（`BootstrapFewShot`, `BayesianSignatureOptimizer`）、戦略的洞察生成（`ChainOfThought`, RAG）、意思決定支援（`MultiChainComparison`）モジュールへのアクセス設定。
*   **主な役割と振る舞い（詳細化）**:
    1.  **ビジネス要件分析とDSPyによる質問最適化**:
        *   ユーザーからの曖昧なビジネス目標や課題に対し、Mastra.AI AgentはDSPyブリッジAPIを呼び出します。
        *   **DSPy連携**: `BootstrapFewShot`を利用し、少数の高品質な質問応答ペア（事前に用意または過去の対話から抽出）に基づいて、ユーザーの意図を深掘りするための最適な質問プロンプト群を生成します。例えば、「DX推進」という曖昧な指示に対し、「具体的なDXの目標KPIは何ですか？」「現在のDX推進における最大のボトルネックは何ですか？」といった具体的な質問を生成します。
        *   生成された質問をユーザーに提示し、回答を得ることで要件を明確化します。
    2.  **TOGAF ADMプロセス支援とDSPyによる文書生成補助**:
        *   ADMの各フェーズにおける成果物作成時、Mastra.AI Agentは関連情報を収集し、DSPyブリッジAPIに文書生成支援を要求します。
        *   **DSPy連携**: `ChainOfThought`とRAG（Retrieval-Augmented Generation）を活用。まずRAGで知識ベースから関連性の高いTOGAFテンプレート、過去の類似プロジェクトの成果物、業界標準ドキュメントを検索・取得。次に`ChainOfThought`がこれらの情報をステップバイステップで処理し、成果物の構成案、主要セクションのドラフト、図表の提案などを行います。例えば、「アーキテクチャビジョン文書」作成時、関連するビジネス戦略とITトレンドをRAGで取得し、CoTでビジョンステートメントの候補を複数生成します。
    3.  **ArchiMateモデリング支援とDSPyによる整合性チェック**:
        *   ユーザー記述や既存資料からモデル要素を抽出する際、DSPyの`Predict`（情報抽出タスク用）を利用して要素候補と関係性を提案します。
        *   **DSPy連携**: 作成されたArchiMateモデル（例えばXML形式やDSLで表現）に対し、DSPyの`Predict`を用いて定義済みの整合性ルール（例: 「全てのアプリケーションコンポーネントはビジネスプロセスに紐づくべき」）に基づいてチェックを行い、違反箇所や改善提案をフィードバックします。
    4.  **戦略的代替案の生成と評価（DSPy強化）**:
        *   複数のIT戦略オプションやアーキテクチャ方針を生成する際、DSPyのRAG機能で広範な知識（業界レポート、技術トレンド、競合分析など）を収集します。
        *   **DSPy連携**: `MultiChainComparison`を利用し、各オプションについて、メリット・デメリット、リスク、コスト、ビジネス価値、実現可能性などを多角的に分析・評価する複数の思考チェーンを並行して実行します。結果を構造化された比較表やレーダーチャートとしてMastra.AI Agentに返し、ユーザーに提示します。`BayesianSignatureOptimizer`を用いて、この比較評価プロセスのプロンプトや思考チェーン構造を継続的に改善します。
    5.  **ステークホルダーコミュニケーションとDSPyによるフィードバック分析**:
        *   生成された戦略文書や分析レポートに対するステークホルダーからのフィードバック（テキスト形式）を収集します。
        *   **DSPy連携**: DSPyの`Predict`（感情分析、トピック抽出、要約タスク用）を用いてフィードバック内容を分析し、主要な懸念点、賛同点、改善要望などを抽出・整理します。Mastra.AI Agentはこれに基づき、文書改訂の優先順位付けや具体的な修正案作成を支援します。
*   **DSPyとの連携プロトコル（例: REST API）**:
    *   **プロンプト最適化リクエスト**:
        `POST /dspy/optimize_prompt`
        `Body: { "task_description": "ユーザーのビジネス目標を深掘りする質問生成", "few_shot_examples": [{"input": "DX推進", "output": "具体的なKPIは？"}, ...], "optimizer_type": "BootstrapFewShot" }`
    *   **戦略オプション評価リクエスト**:
        `POST /dspy/evaluate_options`
        `Body: { "options": [{"name": "クラウド全面移行", "details": "..."}, {"name": "ハイブリッド戦略", "details": "..."}], "evaluation_criteria": ["コスト", "リスク", "ビジネス価値"], "module_type": "MultiChainComparison" }`
*   **他Agentとの連携**: 変更なし。

#### 4.1.2. 知識ベース管理Agent (KnowledgeBaseManagerAgent)

*   **Runtime Contextの主要項目（改訂）**:
    *   `DSPy Integration Config`: 情報抽出（`Predict`）、構造化データ変換（`Program`）、**ナレッジグラフ構築支援モジュール**へのアクセス設定。
*   **主な役割と振る舞い（詳細化）**:
    1.  成果物の収集と登録: 変更なし。
    2.  メタデータ管理とDSPyによる自動抽出:
        *   **DSPy連携**: 登録される文書（PDF, Word, テキストなど）の内容から、DSPyの`Predict`を用いてキーワード、概要、主要エンティティ（プロジェクト名、技術要素、関係者など）を自動抽出し、メタデータとして付与します。
    3.  バージョン管理: 変更なし。
    4.  知識の構造化とリンク（DSPy強化）:
        *   **DSPy連携**: DSPyの`Program`を用いて、複数の`Predict`モジュール（エンティティ抽出、関係性抽出）を組み合わせたパイプラインを定義します。これにより、異なる成果物間の暗黙的な関連性（例: ある戦略文書で言及された技術が、別の設計書で採用されている）を推論し、ナレッジグラフのノードとエッジとして構造化します。例えば、Neo4jなどのグラフデータベースへの登録用Cypherクエリを生成します。
    5.  アクセス制御と検索インターフェース提供: 変更なし。
*   **DSPyとの連携プロトコル（例: REST API）**:
    *   **メタデータ抽出リクエスト**:
        `POST /dspy/extract_metadata`
        `Body: { "document_content": "...", "extraction_fields": ["keywords", "summary", "entities"] }`
*   **他Agentとの連携**: 変更なし。

### 4.2. システム設計層 (C4モデル拡張)

#### 4.2.1. ソリューションアーキテクトAgent (SolutionArchitectAgent)

*   **Runtime Contextの主要項目（改訂）**:
    *   `DSPy Integration Config`: C4モデル図作成支援（`Predict`, RAG）、設計パターン提案（RAG, `ChainOfThought`）、クリーンアーキテクチャマッピング支援（`Predict`）、**ADR生成・検索支援モジュール**。
*   **主な役割と振る舞い（詳細化）**:
    1.  戦略的インプットの解釈: 変更なし。
    2.  C4モデル設計とDSPyによるドラフト生成・記述標準化:
        *   **DSPy連携**: ユーザーからの自然言語によるシステム概要説明や、戦略層からの要件を入力として、DSPyの`Predict`とRAG（C4モデルのベストプラクティスや過去事例を検索）を用いて、C4モデルの各レベル（特にコンテキスト図、コンテナ図）の初期ドラフト（要素リスト、関係性、説明文案）を生成します。Structurizr DSL形式での出力も、DSPyの`Predict`（構造化データ生成タスク用）で支援します。
    3.  設計パターンの適用とDSPyによる推薦・評価:
        *   **DSPy連携**: システム要件（例: 「秒間1000リクエスト処理可能な決済システム」）と制約条件を入力とし、DSPyのRAG機能で関連する設計パターン（マイクロサービス、イベントソーシング、CQRSなど）や技術スタックを知識ベースから検索します。その後、`ChainOfThought`を用いて各パターンの適用可能性、メリット・デメリットを分析し、Mastra.AI Agentに推薦理由と共に提示します。
    4.  クリーンアーキテクチャとのマッピングとDSPyによる妥当性チェック:
        *   **DSPy連携**: C4コンポーネントとクリーンアーキテクチャのレイヤーマッピング案に対し、DSPyの`Predict`を用いて定義済みの依存関係ルール（例: 「Use CasesレイヤーはEntitiesレイヤーにのみ依存可能」）に基づいて検証し、違反箇所や曖昧な点を指摘します。
    5.  インターフェース設計: OpenAPI Specificationの記述支援にDSPyの`Predict`（スキーマ生成、ドキュメント記述補助）を活用。
    6.  ADR作成支援とDSPyによるドラフト生成:
        *   **DSPy連携**: 設計会議の議事録や関連ドキュメントから、DSPyの`Predict`（情報抽出・要約タスク用）を用いてADRの主要項目（課題、コンテキスト、決定事項、理由など）を抽出し、ADRテンプレートに沿ったドラフトを自動生成します。
*   **DSPyとの連携プロトコル（例: REST API）**:
    *   **C4モデルドラフト生成リクエスト**:
        `POST /dspy/generate_c4_draft`
        `Body: { "system_description": "...", "requirements": "...", "c4_level": "container", "output_format": "structurizr_dsl" }`
    *   **設計パターン推薦リクエスト**:
        `POST /dspy/recommend_design_pattern`
        `Body: { "system_requirements": "...", "constraints": "..." }`
*   **他Agentとの連携**: 変更なし。

#### 4.2.2. 設計整合性チェックAgent (DesignConsistencyCheckerAgent)

*   **Runtime Contextの主要項目（改訂）**:
    *   `DSPy Integration Config`: 矛盾検出（`Predict`, `ChainOfThought`）、ルールベース推論（`Predict`）、**差異分析レポート生成モジュール**。
*   **主な役割と振る舞い（詳細化）**:
    1.  多層間整合性検証とDSPyによる意味的矛盾検出:
        *   **DSPy連携**: 複数の設計ドキュメント（TOGAF成果物、C4モデル図のメタデータ、クリーンアーキテクチャ定義書など）を入力とし、DSPyの`Predict`や`ChainOfThought`を用いて、異なる文書間で記述されている同名エンティティの属性値の矛盾、論理的な不整合（例: ある機能が戦略では必須とされているが、システム設計では省略されている）を検出します。
    2.  標準・ガイドライン遵守チェック: 変更なし。
    3.  依存関係検証: 変更なし。
    4.  不整合・違反レポートとDSPyによる修正案提案:
        *   **DSPy連携**: 検出された不整合や違反に対し、DSPyの`Predict`を用いて、知識ベース内の類似事例や設計標準に基づいて具体的な修正案や代替案を提案します。
*   **他Agentとの連携**: 変更なし。

#### 4.2.3. ADR管理Agent (ADRManagerAgent)

*   **Runtime Contextの主要項目（改訂）**:
    *   `DSPy Integration Config`: ADRドラフト生成（`Predict`）、情報抽出（`Predict`）、**ADR関連性分析モジュール**。
*   **主な役割と振る瑁い（詳細化）**:
    1.  ADR作成支援: 前述のソリューションアーキテクトAgentとの連携を強化。
    2.  ADR登録と管理: 変更なし。
    3.  ADR検索と参照、DSPyによる関連ADR提示:
        *   **DSPy連携**: ユーザーが特定の設計課題やコンポーネントについて問い合わせた際、DSPyのRAG機能を用いて関連性の高い既存ADRを検索し、その要約や背景、影響範囲などを提示します。また、複数のADR間の関連性（例: あるADRが別のADRの前提となっている）を分析し、可視化するのを支援します。
*   **他Agentとの連携**: 変更なし。

### 4.3. コード実装層 (クリーンアーキテクチャ強化版)

#### 4.3.1. ソフトウェアエンジニアAgent (SoftwareEngineerAgent)

*   **Runtime Contextの主要項目（改訂）**:
    *   `DSPy Integration Config`: コード生成（`Predict`, `ChainOfThought` with RAG for libraries/frameworks）、リファクタリング提案（`Predict`）、テストケース生成（`Predict`）、**コードコメント自動生成モジュール**、**脆弱性パターン検出モジュール**。
*   **主な役割と振る舞い（詳細化）**:
    1.  設計仕様の理解: 変更なし。
    2.  コードスケルトン生成とDSPyによる具体的実装提案:
        *   **DSPy連携**: C4コンポーネント仕様、インターフェース定義、クリーンアーキテクチャのレイヤー情報、選択されたプログラミング言語・フレームワークを入力とし、DSPyの`Predict`とRAG（対象フレームワークのベストプラクティスやAPIドキュメントを検索）を組み合わせて、より具体的なコードスケルトン（クラス定義、メソッド実装の雛形、DI設定、エラーハンドリング構造など）を生成します。例えば、Spring Bootでのコントローラ、サービス、リポジトリのスケルトン生成など。
    3.  ビジネスロジック実装支援とDSPyによるアルゴリズム提案:
        *   **DSPy連携**: 複雑なビジネスロジック（例: 推薦アルゴリズム、価格計算エンジン）について、その仕様や制約をDSPyに伝えると、`ChainOfThought`やRAG（学術論文や専門ブログ記事を検索）を用いて、適切なアルゴリズムの選択肢、擬似コード、または特定ライブラリの利用方法を提案します。
    4.  リファクタリング提案とDSPyによる高度な分析:
        *   **DSPy連携**: 既存コードの静的解析結果（例: SonarQubeのレポート）やコードメトリクス（循環的複雑度など）をDSPyに入力し、`Predict`を用いて潜在的な「コードの臭い」や設計原則違反箇所を特定します。さらに、`ChainOfThought`で具体的なリファクタリング手順（例: 「メソッド抽出」「クラス分割」）や、リファクタリング後の期待効果を提示します。
    5.  ユニットテスト/結合テスト作成支援とDSPyによる多様なケース生成:
        *   **DSPy連携**: メソッドのシグネチャ、事前条件・事後条件、関連するビジネスルールを入力とし、DSPyの`Predict`を用いて、テストケース（正常系、異常系、境界値、セキュリティテスト観点）の入力データと期待される出力、必要なモック設定などを網羅的に生成します。テストフレームワーク（JUnit, Jestなど）用のテストコードテンプレートも生成します。
    6.  コードレビュー支援とDSPyによる潜在的バグ指摘:
        *   **DSPy連携**: プルリクエストの差分コードに対し、DSPyの`Predict`（学習済み脆弱性パターンやアンチパターンに基づいて訓練）を用いて、潜在的なセキュリティ脆弱性、パフォーマンスボトルネック、バグの可能性を指摘し、修正案を提示します。
    7.  **コードコメント自動生成**:
        *   **DSPy連携**: 実装されたメソッドやクラスのコードを入力とし、DSPyの`Predict`を用いて、その機能、パラメータ、返り値などを説明する適切なコードコメント（例: Javadoc, TSDoc形式）を自動生成します。
*   **DSPyとの連携プロトコル（例: REST API）**:
    *   **コード生成リクエスト**:
        `POST /dspy/generate_code`
        `Body: { "component_spec": "...", "interface_def": "...", "language": "typescript", "framework": "nestjs", "target_layer": "interface_adapter" }`
    *   **テストケース生成リクエスト**:
        `POST /dspy/generate_test_cases`
        `Body: { "method_signature": "...", "business_rules": "...", "test_framework": "jest" }`
*   **他Agentとの連携**: 変更なし。

#### 4.3.2. DevOpsエンジニアAgent (DevOpsEngineerAgent)

*   **Runtime Contextの主要項目（改訂）**:
    *   `DSPy Integration Config`: CI/CDスクリプト生成（`Predict`, RAG for CI/CD tools）、デプロイ戦略提案（`ChainOfThought`）、監視設定支援（`Predict`）、**IaCセキュリティスキャンルール生成モジュール**。
*   **主な役割と振る舞い（詳細化）**:
    1.  CI/CDパイプライン構築支援とDSPyによるベストプラクティス適用:
        *   **DSPy連携**: アプリケーションの技術スタック、デプロイターゲット（Kubernetes, AWS EC2など）、組織のセキュリティポリシーを入力とし、DSPyの`Predict`とRAG（CI/CDツールのドキュメントやセキュリティベストプラクティスを検索）を用いて、CI/CDパイプラインスクリプト（GitHub Actions workflow, Jenkinsfileなど）のテンプレートや設定スニペットを生成します。特にセキュリティスキャン（SAST, DAST, 依存関係チェック）の組み込みを推奨します。
    2.  デプロイ戦略策定支援とDSPyによるリスク評価:
        *   **DSPy連携**: アプリケーションの特性、SLA要件、インフラ環境を入力とし、DSPyの`ChainOfThought`を用いて複数のデプロイ戦略（ブルー/グリーン、カナリア、ローリングアップデート）を比較検討します。各戦略のメリット・デメリットに加え、潜在的なリスク（例: カナリアリリース時の影響範囲、ロールバック手順の複雑さ）を評価し、最適な戦略と具体的な設定パラメータを提案します。
    3.  Infrastructure as Code (IaC) 生成支援とDSPyによるセキュリティチェック:
        *   **DSPy連携**: Terraform, AnsibleなどのIaCスクリプト生成を支援する際、DSPyの`Predict`を用いて、生成されたスクリプトに対してセキュリティベストプラクティス（例: 最小権限の原則、ハードコーディングされたシークレットの検出）に基づいてチェックを行い、警告や修正案を提示します。
    4.  監視・ロギング設定支援とDSPyによる異常検知ルール提案:
        *   **DSPy連携**: アプリケーションの主要メトリクスやログパターンを入力とし、DSPyの`Predict`を用いて、PrometheusのアラートルールやGrafanaのダッシュボード構成案を生成します。さらに、過去の障害事例やシステムの特性に基づいて、異常検知のための高度なアラートルール（例: 通常とは異なるログパターンの出現、メトリクスの相関変化）を提案します。
    5.  セキュリティプラクティス適用支援: 変更なし。
*   **他Agentとの連携**: 変更なし。

## 5. Agent間連携のプロトコルとデータフロー（詳細化）

前回の設計に加え、以下の点を詳細化します。

*   **非同期メッセージング**: Mastra.AIのメッセージング基盤として、**NATS**または**Redis Streams**の利用を推奨します。これらは軽量かつ高性能で、TypeScript/Python両方からのアクセスが容易です。メッセージ形式はJSONとし、**JSON Schema**で厳密に定義します。各メッセージには、相関ID、発行元Agent ID、ターゲットAgent ID、タイムスタンプ、ペイロードタイプなどのヘッダー情報を含めます。
*   **共有知識ベース経由**: 知識ベースの実体としては、ドキュメント管理に**SharePoint**や**Confluence**、構造化データ（ArchiMateモデル、C4モデル、ナレッジグラフ）に**Neo4j**や**Amazon Neptune**、ADR管理に特化したツール（例: **Log4brains**）やGitベースのリポジトリの利用を検討します。知識ベース管理Agentはこれらのリポジトリへのアダプタとして機能します。
*   **ワークフローエンジンによるオーケストレーション**: Mastra.AIのワークフローエンジンは、BPMNやカスタムDSLで定義されたプロセスフローに基づいてAgentを制御します。DSPyブリッジAPI呼び出しもワークフローの一環として組み込み、非同期処理やタイムアウト、リトライ処理を管理します。
*   **データフローとトレーサビリティ**: 各成果物や意思決定には一意のIDを付与し、関連性を知識ベースに記録することで、戦略からコードまでのトレーサビリティを確保します。OpenTelemetryを利用して、Agent間のメッセージフローやDSPy呼び出しを含む処理全体のトレース情報を収集し、GrafanaやJaegerで可視化します。

## 6. 各レイヤーの実装仕様指針（詳細化）

前回の指針に加え、以下の点を詳細化・具体化します。

### 6.1. Agentの実装

*   **言語・フレームワーク**: Mastra.AI AgentはTypeScriptと**NestJS**フレームワークを推奨。NestJSはモジュール性、DI、テスト容易性に優れ、大規模アプリケーションに適しています。
*   **DSPyブリッジサービス**: Pythonと**FastAPI**または**Flask**で実装。非同期処理に対応し、スケーラビリティを考慮します。Dockerコンテナとしてデプロイします。
*   **状態管理**: Agentの状態はRuntime Context内で管理し、必要に応じてRedisなどの外部ストアに永続化することも検討します。
*   **ツール利用**: Mastra.AIの標準ツールに加え、DSPyブリッジAPIクライアントライブラリ（TypeScriptで生成または手動作成）を利用します。

### 6.2. データモデルとAPIインターフェース

*   **データモデル**: JSON Schemaに加え、**Protocol Buffers**や**Apache Avro**の利用も検討し、型安全なデータ交換とスキーマ進化に対応します。
*   **DSPyブリッジAPI**: OpenAPI Specification (OAS 3.0) で厳密に定義し、クライアントライブラリとサーバスタブを自動生成します。認証にはAPIキーまたはOAuth2 Client Credentials Grantフローを推奨。

### 6.3. エラーハンドリングとロギング

*   **エラーハンドリング**: DSPyブリッジAPI呼び出し時のエラー（ネットワークエラー、DSPy内部エラー、タイムアウトなど）を考慮し、Mastra.AI Agent側で適切なリトライ戦略（指数バックオフ付き）やフォールバック処理（例: DSPyが利用できない場合はデフォルトのプロンプトを使用）を実装します。サーキットブレーカーパターンの導入も検討します。
*   **ロギング**: 構造化ロギング（例: JSON形式でPinoライブラリを使用）を徹底し、OpenTelemetryと連携して分散トレーシングを実現します。ログレベル（DEBUG, INFO, WARN, ERROR）を適切に設定し、ログ集約基盤（ELK Stack, Grafana Lokiなど）に転送します。

### 6.4. セキュリティ考慮事項

*   **DSPyブリッジAPIの保護**: 入力値のサニタイズと検証を徹底し、プロンプトインジェクション攻撃のリスクを低減します。レート制限やアクセス制御を実装します。
*   **知識ベースのアクセス制御**: ロールベースアクセス制御（RBAC）を厳格に適用し、Agentが必要最小限の権限で知識ベースにアクセスできるようにします。
*   **依存関係セキュリティ**: `npm audit`や`pip-audit`、SnykなどのツールをCI/CDパイプラインに組み込み、依存ライブラリの脆弱性を継続的にスキャンします。

### 6.5. テスト戦略

*   **DSPyモジュールのテスト**: DSPyの`Evaluate`機能とカスタムメトリクスを用いて、各DSPyモジュール（プロンプト最適化、RAG、コード生成など）の品質を定量的に評価します。Mastra.AIの`Evals`機能と連携し、評価プロセスを自動化します。
*   **E2Eテスト**: 主要なユースケースシナリオ（例: 新規ビジネス要件からアプリケーションのコードスケルトン生成まで）を対象に、実際のAgent間連携とDSPyブリッジAPI呼び出しを含むE2Eテストを自動化します。PlaywrightやCypressなどのツールを利用してUI操作を含むテストも検討します。
*   **カオスエンジニアリング**: 意図的にDSPyブリッジサービスの障害や知識ベースの遅延などを発生させ、システム全体の耐障害性やリカバリ能力を検証します。

### 6.6. 技術スタックと言語間連携の課題と解決策

*   **言語間の互換性**: TypeScript (Mastra.AI) と Python (DSPy) の連携は、主に**REST API**または**gRPC**を介したマイクロサービスアーキテクチャで実現します。これにより、各コンポーネントを独立して開発・デプロイできます。
    *   `deno-python`やWebAssembly (Wasm) の利用は、現時点では実験的な要素が強く、大規模システムでの安定性やパフォーマンス、デバッグの観点から、主要な連携手段としてはリスクが高いと判断します。ただし、特定のCPUバウンドなDSPyコアロジックをWasm化してMastra.AIから直接呼び出すといった限定的な利用は将来的に検討可能です。
*   **パフォーマンスオーバーヘッド**: API呼び出しによるオーバーヘッドを最小限に抑えるため、以下の対策を講じます。
    *   DSPyブリッジサービスは非同期処理（例: FastAPIのasync/await）をフル活用し、リクエストを効率的に処理します。
    *   Mastra.AI AgentからのDSPy呼び出しも非同期で行い、ブロッキングを避けます。
    *   頻繁に利用されるDSPyの最適化結果や生成物は、Mastra.AI側またはDSPyブリッジサービス側でキャッシュすることを検討します（例: Redisを利用）。
    *   ペイロードサイズが大きい場合は、gRPC + Protocol Buffersの利用がREST + JSONよりも効率的な場合があります。
*   **ライセンス整合性**: Mastra.AIのELv2ライセンスとDSPyのMITライセンスは互換性があり、統合フレームワーク全体としての配布や利用に問題はありません。ただし、各コンポーネントのライセンス表記は適切に行う必要があります。

## 7. Mastra.AIとDSPyのコードレベル統合例（詳細化）

添付ファイルで示された例を参考に、より具体的なコードレベルの統合イメージを示します。

### 7.1. Mastra.AI Agent (TypeScript) からDSPyブリッジAPIを呼び出す例

```typescript
// src/agents/enterprise-architect-agent.ts
import { BaseAgent, tool } from "@mastra/core";
import { DspyBridgeService, DspyOptimizePromptRequest, DspyEvaluateOptionsRequest } from "../services/dspy-bridge.service";

interface EnterpriseArchitectAgentContext {
  dspyBridge: DspyBridgeService;
  // ... other context properties
}

export class EnterpriseArchitectAgent extends BaseAgent<EnterpriseArchitectAgentContext> {
  constructor(context: EnterpriseArchitectAgentContext) {
    super(context);
  }

  @tool({ description: "ビジネス要件を分析し、戦略的質問を生成します" })
  async analyzeBusinessRequirements(userInput: string): Promise<string[]> {
    const optimizeRequest: DspyOptimizePromptRequest = {
      task_description: "ユーザーのビジネス目標を深掘りする質問生成",
      few_shot_examples: [
        { input: "我が社のDXを推進したい", output: "DX推進の具体的な数値目標（KPI）は何ですか？" },
        { input: "コスト削減が課題だ", output: "どの業務領域でのコスト削減を最優先と考えていますか？" },
      ],
      optimizer_type: "BootstrapFewShot",
      user_query: userInput, // ユーザーの初期入力もコンテキストとして渡す
    };
    try {
      const optimizedQuestions = await this.context.dspyBridge.optimizePrompt(optimizeRequest);
      // optimizedQuestions は string[] 型と想定
      return optimizedQuestions.prompts;
    } catch (error) {
      this.logger.error("DSPyプロンプト最適化エラー:", error);
      // フォールバック処理: デフォルトの質問セットを返すなど
      return ["具体的な目標は何ですか？", "現状の課題は何ですか？"];
    }
  }

  @tool({ description: "複数の戦略オプションを比較評価します" })
  async evaluateStrategicOptions(options: Array<{name: string, details: string}>): Promise<any> {
    const evaluateRequest: DspyEvaluateOptionsRequest = {
      options: options,
      evaluation_criteria: ["コスト", "リスク", "実現可能性", "ビジネスインパクト"],
      module_type: "MultiChainComparison",
    };
    try {
      const evaluationResult = await this.context.dspyBridge.evaluateOptions(evaluateRequest);
      // evaluationResult は構造化された比較結果オブジェクトと想定
      return evaluationResult;
    } catch (error) {
      this.logger.error("DSPy戦略オプション評価エラー:", error);
      throw error; // または適切なエラーハンドリング
    }
  }
}

// src/services/dspy-bridge.service.ts
// (axiosやfetchを用いたHTTPクライアント実装)
export interface DspyOptimizePromptRequest { /* ... */ prompts?: string[] }
export interface DspyEvaluateOptionsRequest { /* ... */ }

export class DspyBridgeService {
  private apiClient: any; // e.g., AxiosInstance
  constructor(baseUrl: string, apiKey: string) {
    // Initialize HTTP client with baseUrl and auth headers
  }
  async optimizePrompt(request: DspyOptimizePromptRequest): Promise<DspyOptimizePromptRequest> { /* ... */ }
  async evaluateOptions(request: DspyEvaluateOptionsRequest): Promise<any> { /* ... */ }
  // ... other DSPy module calls
}
```

### 7.2. DSPyブリッジサービス (Python/FastAPI) の実装例

```python
# dspy_bridge/main.py
from fastapi import FastAPI, HTTPException, Body
from pydantic import BaseModel, Field
from typing import List, Dict, Any
import dspy

# --- DSPyモジュールの設定 (例) ---
lm = dspy.OpenAI(model='gpt-3.5-turbo', api_key='YOUR_OPENAI_API_KEY') # 環境変数から読み込むのが望ましい
dspy.settings.configure(lm=lm)

# --- Pydanticモデル定義 (リクエスト/レスポンス) ---
class FewShotExample(BaseModel):
    input: str
    output: str

class OptimizePromptRequest(BaseModel):
    task_description: str
    few_shot_examples: List[FewShotExample]
    optimizer_type: str = "BootstrapFewShot"
    user_query: str | None = None
    prompts: List[str] | None = None # レスポンス用

class StrategicOption(BaseModel):
    name: str
    details: str

class EvaluateOptionsRequest(BaseModel):
    options: List[StrategicOption]
    evaluation_criteria: List[str]
    module_type: str = "MultiChainComparison"

app = FastAPI()

# --- DSPyモジュールラッパー (例) ---
class OptimizedQuestionGenerator(dspy.Signature):
    """ユーザーのビジネス目標を深掘りするための最適な質問を3つ生成します。"""
    task_description: str = dspy.InputField(desc="質問生成のタスク説明")
    user_query: str = dspy.InputField(desc="ユーザーからの初期クエリ")
    optimized_question_1: str = dspy.OutputField(desc="最適化された質問1")
    optimized_question_2: str = dspy.OutputField(desc="最適化された質問2")
    optimized_question_3: str = dspy.OutputField(desc="最適化された質問3")

# --- APIエンドポイント定義 ---
@app.post("/dspy/optimize_prompt", response_model=OptimizePromptRequest)
async def optimize_prompt_endpoint(request: OptimizePromptRequest = Body(...)):
    try:
        if request.optimizer_type == "BootstrapFewShot":
            # DSPyのBootstrapFewShotの具体的な使い方に合わせる
            # ここでは例としてシグネチャベースの予測器を仮定
            predictor = dspy.Predict(OptimizedQuestionGenerator)
            # few_shot_examples を使って predictor を訓練または設定するロジックが必要
            # (DSPyの実際のBootstrapFewShotの使い方とは異なる単純化された例)
            # trainset = [dspy.Example(input=ex.input, output=ex.output) for ex in request.few_shot_examples]
            # teleprompter = BootstrapFewShot(metric=some_metric)
            # compiled_predictor = teleprompter.compile(predictor, trainset=trainset)
            
            # 実際のDSPyの利用方法に応じて実装。以下はダミーの応答。
            # この部分はDSPyの具体的なAPIに基づいて詳細化が必要。
            result = predictor(task_description=request.task_description, user_query=request.user_query or "")
            
            response_prompts = [
                result.optimized_question_1,
                result.optimized_question_2,
                result.optimized_question_3
            ]
            request.prompts = [p for p in response_prompts if p] # Noneや空文字を除外
            return request
        else:
            raise HTTPException(status_code=400, detail=f"Unsupported optimizer type: {request.optimizer_type}")
    except Exception as e:
        # エラーロギングをここで行う
        raise HTTPException(status_code=500, detail=str(e))

@app.post("/dspy/evaluate_options")
async def evaluate_options_endpoint(request: EvaluateOptionsRequest = Body(...)):
    # MultiChainComparisonなどのDSPyモジュールを利用した評価ロジックを実装
    # ここではダミーの応答を返す
    # この部分はDSPyの具体的なAPIに基づいて詳細化が必要。
    results = []
    for option in request.options:
        evaluation = {crit: "評価結果..." for crit in request.evaluation_criteria}
        results.append({"option_name": option.name, "evaluation": evaluation})
    return {"evaluation_results": results}

# ... 他のDSPyモジュールに対応するエンドポイント

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

**注記**: 上記のDSPyブリッジサービスのコード例は、DSPyの具体的なモジュールの使い方を簡略化しています。実際の`BootstrapFewShot`や`MultiChainComparison`の利用には、適切な`Signature`定義、`Metric`関数、`Teleprompter`の設定、`compile`メソッドの呼び出しなど、より詳細なDSPyの作法に従った実装が必要です。

## 8. 評価・モニタリングの強化（詳細化）

*   **自動評価の統合**: Mastra.AIの`Evals`機能からDSPyブリッジAPIを介してDSPyの`Evaluate`機能を呼び出します。評価用データセット（devset）はMastra.AI側で管理し、評価メトリクス（例: `answer_accuracy`, `faithfulness`, `latency`）も定義します。DSPy側で評価を実行し、結果（スコア、失敗ケースなど）をMastra.AIに返却します。この評価結果を基に、DSPyのオプティマイザ（例: `BayesianSignatureOptimizer`）を起動し、プロンプトやモジュール設定を自動的に改善するループを構築します。
*   **トレーシングの可視化**: Mastra.AI Agentの処理フロー、Agent間メッセージ、DSPyブリッジAPI呼び出し（リクエスト、レスポンス、レイテンシ）、DSPy内部のモジュール実行ステップ（`ChainOfThought`の各思考ステップなど）をOpenTelemetryでトレースします。収集されたトレースデータはGrafana TempoやJaegerで可視化し、パフォーマンスボトルネックの特定やエラー分析に活用します。

## 9. まとめと今後のステップ

本詳細実装計画では、TOGAF、C4モデル、クリーンアーキテクチャを統合したフレームワークにおいて、Mastra.AIとDSPyを連携させるための具体的なアーキテクチャ、Agentの役割、連携プロトコル、技術スタック、コードレベルの統合例を提示しました。

今後のステップとしては以下が考えられます。

1.  **プロトタイプ開発**: 特定のユースケース（例: 戦略層のエンタープライズアーキテクトAgentによるビジネス要件分析と戦略オプション評価）に絞り、Mastra.AI AgentとDSPyブリッジサービスのプロトタイプを開発します。
2.  **DSPyモジュールの詳細実装**: DSPyブリッジサービス側で、`BootstrapFewShot`, `ChainOfThought`, RAG, `MultiChainComparison`などの主要なDSPyモジュールを具体的なタスクに合わせて実装・最適化します。
3.  **知識ベースの構築**: 各階層で利用する知識ベース（TOGAF成果物、設計パターン、ADRなど）の具体的なリポジトリ選定とデータ投入方法を確立します。
4.  **評価基盤の構築**: Mastra.AI `Evals`とDSPy `Evaluate`を連携させた自動評価基盤を構築し、継続的な改善サイクルを確立します。
5.  **段階的導入**: プロトタイプでの検証後、他のAgentやユースケースへ段階的に展開し、フレームワーク全体を構築していきます。

この詳細実装計画が、貴組織におけるAI駆動型IT戦略策定・アプリケーション実装の実現に向けた具体的な一歩となることを期待します。




## 8. 詳細なAgentワークフローと運用プロトコル

本セクションでは、主要なAgentの代表的なワークフロー、連携プロトコルの詳細、エラーハンドリング、トレーサビリティ、セキュリティ管理、そして障害からの回復戦略について、より具体的に記述します。

### 8.1. 代表的なAgentワークフロー例

#### 8.1.1. EnterpriseArchitectAgent: 新規ビジネス戦略に基づくITロードマップ策定支援ワークフロー

1.  **入力受付**: ユーザー（経営層）から「新市場参入のためのIT戦略とロードマップ案作成」の指示と関連資料（市場分析レポート、競合情報など）をMastra.AIプラットフォーム経由で受け取る。
2.  **初期分析と質問生成 (DSPy連携)**:
    *   EnterpriseArchitectAgentは、受け取った資料と指示を基に、Runtime Contextを更新。
    *   DSPyブリッジAPI (`/dspy/optimize_prompt`) を呼び出し、`BootstrapFewShot`を利用してビジネス目標、制約条件、期待成果を明確化するための深掘り質問群を生成。
    *   生成された質問をユーザーに提示し、回答を収集。Runtime Contextの`Input Data`を更新。
3.  **現状分析 (AS-IS Architecture)**:
    *   知識ベース管理Agent経由で、既存のビジネスプロセスモデル、アプリケーションポートフォリオ、技術インフラ情報を取得。
    *   必要に応じて、関連部署の担当者へのヒアリングタスクを生成し、ユーザーに通知。
4.  **ターゲットアーキテクチャ (TO-BE) 方針策定 (DSPy連携)**:
    *   収集した情報とユーザーの回答に基づき、複数のターゲットアーキテクチャ方針案（例: クラウドネイティブ全面移行、段階的モダナイゼーション、特定業務SaaS導入など）を定義。
    *   DSPyブリッジAPI (`/dspy/evaluate_options`) を呼び出し、`MultiChainComparison`を利用して各方針案をコスト、リスク、実現可能性、ビジネス整合性の観点から評価。知識ベースから業界リファレンスアーキテクチャや過去事例をRAGで参照。
    *   評価結果（比較レポート、推奨方針）を生成し、ユーザーに提示。フィードバックを要求。
5.  **ロードマップ策定とTOGAF成果物作成 (DSPy連携)**:
    *   ユーザー承認を得たターゲットアーキテクチャ方針に基づき、具体的な移行プロジェクト、タイムライン、依存関係を含むITロードマップ案を作成。
    *   TOGAFの関連成果物（アーキテクチャビジョン、ビジネスアーキテクチャ、情報システムアーキテクチャ、技術アーキテクチャ、機会と解決策、移行計画など）のドラフト作成をDSPyブリッジAPI (`/dspy/generate_document_draft` - 新規APIエンドポイント想定) に依頼。RAGと`ChainOfThought`を活用。
6.  **成果物レビューと承認**: 作成したロードマップ案とTOGAF成果物ドラフトを関連ステークホルダーに提示し、レビューと承認を依頼。Mastra.AIのワークフロー機能でレビューステータスを管理。
7.  **最終化と知識ベース登録**: 承認された成果物を最終化し、知識ベース管理Agentに登録を依頼。
8.  **ワークフロー完了**: ユーザーに完了を通知。

#### 8.1.2. SolutionArchitectAgent: 新規マイクロサービス設計ワークフロー

1.  **入力受付**: EnterpriseArchitectAgentから承認済みのIT戦略・ターゲットアーキテクチャ、またはビジネス部門から新規サービス開発要件を受け取る。
2.  **要件詳細化とC4コンテキスト定義**: 受け取った要件に基づき、対象システムの境界、主要な外部システムとの関連性をC4コンテキスト図として定義。必要に応じてDSPyの質問生成機能で曖昧点を明確化。
3.  **コンテナ設計 (DSPy連携)**:
    *   システムを構成する主要なコンテナ（例: APIゲートウェイ、商品サービス、注文サービス、顧客DB）を特定。
    *   各コンテナの責務、技術選択肢（言語、フレームワーク、データストア）、コンテナ間インターフェースの概要を定義。
    *   DSPyブリッジAPI (`/dspy/generate_c4_draft`) を利用し、コンテナ図のドラフトと説明文案を生成。
    *   技術選択肢について、DSPyのRAG機能で知識ベースから関連ADRやベストプラクティスを検索し、`ChainOfThought`で比較評価。
4.  **コンポーネント設計とクリーンアーキテクチャ適用 (DSPy連携)**:
    *   各コンテナ内部の主要コンポーネントをC4コンポーネント図として詳細化。
    *   クリーンアーキテクチャの原則に基づき、各コンポーネントをレイヤー（Entities, Use Cases, Interface Adapters, Frameworks & Drivers）にマッピング。
    *   DSPyブリッジAPI (`/dspy/check_clean_architecture_mapping`) を利用し、マッピングの妥当性と依存関係ルールの遵守を検証。
5.  **インターフェース詳細設計 (OpenAPI)**: コンテナ間、コンポーネント間のAPIをOpenAPI Specification (OAS 3.0) で詳細に定義。DSPyの`Predict`でOASのスキーマ定義や記述補助。
6.  **ADR作成 (DSPy連携)**: 主要な設計判断（技術選定、パターン適用理由など）をADRとして記録。ADR管理Agentと連携し、DSPyの`Predict`でドラフト生成支援。
7.  **設計レビューと承認**: 設計整合性チェックAgentに設計全体の整合性検証を依頼。結果を基に設計を修正し、関連ステークホルダー（開発チームリード、セキュリティ担当など）にレビューと承認を依頼。
8.  **実装チームへの引き渡し**: 承認された設計ドキュメント（C4モデル図、OAS、ADR）を知識ベースに登録し、SoftwareEngineerAgentに実装タスクとして引き渡す。

### 8.2. 連携プロトコルの詳細

#### 8.2.1. 非同期メッセージング (NATS / Redis Streams)

*   **主要トピック/ストリーム例**:
    *   `ea.strategy.new_request`: EA Agentへの新規戦略策定リクエスト。
    *   `ea.strategy.approved`: EA Agentが策定した戦略の承認通知。
    *   `sa.design.new_request.{project_id}`: SA Agentへの新規設計リクエスト。
    *   `sa.design.review_request.{component_id}`: SA Agentからの設計レビュー依頼。
    *   `sa.design.approved.{component_id}`: SA Agentの設計承認通知。
    *   `swe.code.new_task.{component_id}`: SWE Agentへの新規実装タスク。
    *   `swe.code.review_request.{pull_request_id}`: SWE Agentからのコードレビュー依頼。
    *   `devops.build.request.{commit_id}`: DevOps Agentへのビルド要求。
    *   `devops.deploy.success.{environment}.{app_version}`: デプロイ成功通知。
    *   `kb.artefact.new`: 知識ベースへの新規成果物登録通知。
    *   `dspy.request.{module_type}`: DSPyブリッジサービスへの処理要求。
    *   `dspy.response.{correlation_id}`: DSPyブリッジサービスからの処理結果。
*   **メッセージ構造 (JSON Schema)**: 各トピックで交換されるメッセージの構造をJSON Schemaで厳密に定義。必須フィールド、データ型、フォーマットを規定。
    ```json
    // 例: sa.design.new_request.{project_id} メッセージスキーマ
    {
      "$schema": "http://json-schema.org/draft-07/schema#",
      "title": "NewDesignRequest",
      "type": "object",
      "properties": {
        "message_id": { "type": "string", "format": "uuid" },
        "correlation_id": { "type": "string", "format": "uuid" },
        "timestamp": { "type": "string", "format": "date-time" },
        "source_agent_id": { "type": "string" },
        "project_id": { "type": "string" },
        "requirements_summary": { "type": "string" },
        "related_artefact_ids": { "type": "array", "items": { "type": "string" } }
      },
      "required": ["message_id", "timestamp", "source_agent_id", "project_id", "requirements_summary"]
    }
    ```
*   **メッセージ配信保証**: NATSでは`At-Least-Once`配信を基本とし、重要なメッセージにはJetStreamを利用して永続化と確認応答（ACK）を導入。Redis StreamsではコンシューマーグループとACKを利用。
*   **デッドレターキュー (DLQ)**: 処理に複数回失敗したメッセージ（例: 3回リトライ後）はDLQに転送。DLQのメッセージは定期的に監視され、手動での分析・再処理、または破棄の判断を行う。

#### 8.2.2. API設計 (DSPyブリッジAPI & 内部サービスAPI)

*   **バージョニング**: URLパスにバージョン番号を含める (例: `/api/v1/dspy/...`)。破壊的変更時にはバージョンを上げる。
*   **認証・認可**: DSPyブリッジAPIや内部サービスAPIには、OAuth2 Client Credentials GrantフローまたはAPIキー認証を必須とする。APIキーは環境変数やシークレット管理サービス（例: HashiCorp Vault）で安全に管理。
*   **リクエスト/レスポンス形式**: JSONを基本とする。リクエストボディとレスポンスボディの構造はOpenAPI Specificationで明確に定義。
*   **エラーレスポンス**: HTTPステータスコードを適切に使用（4xx: クライアントエラー, 5xx: サーバーエラー）。レスポンスボディにはエラーコード、エラーメッセージ、詳細情報（デバッグ用）を含むJSONオブジェクトを返す。
    ```json
    // エラーレスポンス例
    {
      "error_code": "DSPY_MODULE_FAILED",
      "message": "DSPy ChainOfThought module execution failed.",
      "details": "Underlying LLM API returned 503 Service Unavailable after 3 retries.",
      "request_id": "xyz-123-abc"
    }
    ```
*   **冪等性**: 作成・更新系のAPIエンドポイント（特に支払い処理やリソース作成など、副作用が大きいもの）は、クライアントがリクエストヘッダーに冪等キー（`Idempotency-Key`）を付与することで冪等性を保証できるように設計する。サーバー側で冪等キーとリクエスト/レスポンスを一定期間保存し、重複リクエストを検知・処理する。
*   **レート制限**: APIゲートウェイやサービス自体にレート制限を導入し、不正な大量アクセスやDoS攻撃から保護する。

### 8.3. 包括的なエラーハンドリング戦略

*   **Agentレベル**: 各Mastra.AI Agentは、自身の処理ロジック内で発生しうるエラー（外部API呼び出し失敗、データ処理エラー、ビジネスルール違反など）をtry-catchで捕捉。
    *   **リトライ**: ネットワークエラーや一時的なサービス利用不可などの**過渡的エラー**に対しては、指数バックオフとジッターを用いたリトライ処理を実装（例: 1秒、2秒、4秒、8秒間隔で最大5回リトライ）。
    *   **フォールバック**: DSPyブリッジAPIが利用できない場合や、DSPyモジュールが期待通りに動作しない場合、事前に定義されたフォールバックロジック（例: デフォルトのプロンプトを使用、単純なルールベースの処理に切り替え、ユーザーに手動介入を促す）を実行。
    *   **サーキットブレーカー**: 外部サービス（DSPyブリッジAPI、知識ベースAPIなど）への呼び出しにはサーキットブレーカーパターンを適用。連続してエラーが発生する場合、一定期間サービス呼び出しを停止し、システム全体の負荷増大や連鎖的障害を防ぐ。
    *   **状態管理**: エラー発生時、Agentの状態を`Error`または`Degraded`に更新し、Runtime Contextにエラー情報を記録。Mastra.AIのワークフローエンジンや監視システムに通知。
*   **DSPyブリッジサービスレベル**: Python (FastAPI/Flask) 側でも、LLM API呼び出しエラー、DSPy内部モジュールエラー、リソース不足などを適切にハンドリング。
    *   LLM API呼び出し時のリトライ、タイムアウト設定。
    *   詳細なエラー情報をログに出力し、Mastra.AI Agentに適切なエラーコードとメッセージを返す。
*   **ワークフローレベル**: Mastra.AIのワークフローエンジンは、Agentのタスク失敗を検知し、定義された補償トランザクション（例: 作成したリソースのロールバック）、代替パスの実行、またはユーザーへのエスカレーションを行う。

### 8.4. トレーサビリティと監視の強化

*   **分散トレーシング (OpenTelemetry)**:
    *   Mastra.AI Agent、DSPyブリッジサービス、知識ベースアクセス、メッセージキュー間のすべてのリクエストにトレースIDとスパンIDを伝播。
    *   各処理ステップの開始・終了、所要時間、主要なパラメータ、エラー情報をトレース属性として記録。
    *   収集したトレースデータはJaegerやGrafana Tempoに送信し、リクエスト全体の流れを可視化・分析可能にする。
*   **構造化ロギング**: 全てのコンポーネントでJSON形式の構造化ログを出力。ログにはタイムスタンプ、ログレベル、サービス名、トレースID、スパンID、メッセージ、関連コンテキスト情報を含める。ログ集約基盤（ELK Stack, Grafana Loki）に転送。
*   **主要メトリクス監視 (Prometheus & Grafana)**:
    *   **Mastra.AI Agent**: タスク処理時間、タスクキュー長、エラーレート、リソース使用量（CPU、メモリ）。
    *   **DSPyブリッジサービス**: APIリクエストレート、レスポンスタイム（平均、パーセンタイル）、エラーレート、DSPyモジュール別実行時間、LLM API呼び出しレイテンシとエラーレート。
    *   **メッセージキュー**: キュー長、メッセージ処理レート、DLQメッセージ数。
    *   **知識ベース**: クエリ応答時間、エラーレート、接続数。
*   **ダッシュボード**: Grafanaで上記のメトリクスを可視化するダッシュボードを作成。システム全体の健全性、パフォーマンスボトルネック、異常傾向を早期に把握。
*   **アラート (Alertmanager)**: 主要メトリクスにしきい値を設定し、異常発生時（エラーレート急増、レイテンシ増大、キュー滞留など）に関係者に通知（Slack, PagerDutyなど）。

### 8.5. セキュリティ管理の強化

*   **認証・認可**: 
    *   **Agent間通信**: Mastra.AI内部のAgent間通信やメッセージキューアクセスには、mTLSやトークンベース認証を検討。
    *   **APIキー管理**: DSPyブリッジAPIキーや外部サービスAPIキーはHashiCorp Vaultやクラウドプロバイダーのシークレット管理サービスに保存し、Agentが必要に応じて安全に取得。定期的なキーローテーションを実施。
*   **データ保護**:
    *   **通信の暗号化**: すべてのAPI通信（Mastra.AI Agent - DSPyブリッジ、外部サービス）、メッセージキュー通信、データベース接続はTLS 1.2以上で暗号化。
    *   **データの暗号化**: 知識ベースに保存される機密情報（個人情報、ビジネス上の機密データ）は、アプリケーションレベルまたはデータベースレベルで暗号化 (AES-256など)。
*   **入力検証とサニタイズ**: Agentへの入力データ、APIリクエストペイロードは厳密に検証（JSON Schema、Pydanticモデルなど）。特にDSPyブリッジAPIへのプロンプト入力は、プロンプトインジェクション攻撃のリスクを低減するため、サニタイズ処理やコンテキスト分離を検討。
*   **脆弱性管理**: 定期的な脆弱性スキャン（コンテナイメージ、依存ライブラリ）をCI/CDパイプラインに組み込む。セキュリティパッチの迅速な適用プロセスを確立。
*   **監査ログ**: セキュリティ関連イベント（認証試行、認可変更、機密データアクセス、Agentの重要な状態変更など）を詳細な監査ログとして記録。改ざん防止機能のあるログストレージに保存。

### 8.6. 障害からの回復とレジリエンス

*   **Agentの自己修復**: Mastra.AIプラットフォーム（またはKubernetesなどのオーケストレーション基盤）は、ヘルスチェックに基づいて異常なAgentインスタンスを検出し、自動的に再起動する機能を持つ。
*   **ワークフローのチェックポイントと再開**: 長時間実行されるワークフローや複数のステップからなる処理では、重要な中間状態を永続化ストア（例: Redis, PostgreSQL）に保存（チェックポイント）。Agentやシステムがクラッシュした場合でも、最後のチェックポイントから処理を再開できるようにする。
*   **データ整合性**: 分散トランザクションが避けられない場合は、SagaパターンやTwo-Phase Commit (2PC) のようなパターンを検討するが、複雑性が増すため、可能な限り冪等な操作と結果整合性で設計する。重要なデータ操作では、処理前後の状態を記録し、矛盾が発生した場合の補正処理や手動介入プロセスを定義。
*   **DSPyブリッジサービスの冗長化**: DSPyブリッジサービスは複数のインスタンスをデプロイし、ロードバランサー経由でアクセスすることで可用性を高める。ステートレスな設計を心がけ、スケールアウトを容易にする。
*   **知識ベースのバックアップとリストア**: 知識ベース（データベース、ドキュメントストア）は定期的にバックアップを取得し、迅速なリストア手順を確立・テストする。

### 8.7. 監視・評価・改善サイクル (運用への組み込み)

1.  **継続的監視**: 8.4で定義したメトリクス、ログ、トレースをリアルタイムで監視。
2.  **定期的評価 (Mastra.AI Evals + DSPy Evaluate)**:
    *   週次またはリリースごとに、主要なAgentタスクとDSPyモジュールのパフォーマンスを評価データセットで自動評価。
    *   評価結果（精度、頑健性、レイテンシ、コストなど）を記録し、前回評価からの変化を追跡。
3.  **課題特定と分析**: 監視データや評価結果から、パフォーマンスの低いAgent、効果の薄いDSPyプロンプト/モジュール、頻発するエラーなどを特定。トレース情報やログを深掘りして原因を分析。
4.  **改善アクションの計画と実施**: 分析結果に基づき、改善策を計画。
    *   DSPyプロンプトの修正、Few-shotサンプルの更新、DSPyオプティマイザの再実行。
    *   Agentロジックの修正、エラーハンドリングの強化。
    *   知識ベースのデータ拡充や品質向上。
    *   インフラ設定の調整（リソース割り当て、スケーリングポリシーなど）。
5.  **変更のデプロイと効果測定**: 改善策をデプロイし、A/Bテストや段階的ロールアウトを通じて効果を測定。再度、監視と評価を行い、改善サイクルを継続。

この運用プロトコルにより、フレームワーク全体の品質、パフォーマンス、信頼性を継続的に向上させることが可能になります。




### 7.1. Mastra.AI Agent (TypeScript) からDSPyブリッジAPIを呼び出す詳細例

前回の例を拡張し、エラーハンドリング、認証、具体的なリクエスト/レスポンス処理を含めます。

```typescript
// src/services/dspy-bridge.service.ts
import axios, { AxiosInstance, AxiosError } from 'axios';
import { backOff } from 'exponential-backoff';

// DSPyブリッジAPIからの標準的な成功レスポンス型
export interface DspySuccessResponse<T> {
  data: T;
  request_id: string;
  execution_time_ms: number;
}

// DSPyブリッジAPIからの標準的なエラーレスポンス型
export interface DspyErrorResponse {
  error_code: string;
  message: string;
  details?: any;
  request_id: string;
}

// プロンプト最適化リクエスト型
export interface DspyOptimizePromptRequest {
  task_description: string;
  few_shot_examples: Array<{ input: string; output: string }>;
  optimizer_type: 'BootstrapFewShot' | 'BayesianSignatureOptimizer' | 'MIPRO';
  // optimizer_typeに応じて追加のパラメータが必要になる場合がある
  [key: string]: any; 
}

// プロンプト最適化レスポンス型（成功時）
export interface DspyOptimizePromptResponseData {
  optimized_prompt: string;
  evaluation_metrics?: Record<string, any>;
}

// 戦略オプション評価リクエスト型
export interface DspyEvaluateOptionsRequest {
  options: Array<{ name: string; details: string; [key: string]: any }>;
  evaluation_criteria: string[];
  module_type: 'MultiChainComparison'; // 他の評価モジュールも想定可能
  knowledge_base_query?: string; // RAGで利用するクエリ
}

// 戦略オプション評価レスポンス型（成功時）
export interface DspyEvaluateOptionsResponseData {
  evaluation_summary: string; // または構造化された評価結果
  detailed_comparison: Array<{
    option_name: string;
    scores: Record<string, number | string>;
    pros: string[];
    cons: string[];
    reasoning: string;
  }>;
}

export class DspyBridgeService {
  private client: AxiosInstance;
  private apiKey: string;

  constructor(baseURL: string, apiKey: string, timeout: number = 30000) {
    this.client = axios.create({
      baseURL,
      timeout,
      headers: {
        'Content-Type': 'application/json',
      },
    });
    this.apiKey = apiKey;
  }

  private async requestWithRetry<T_Req, T_ResData>(method: 'post', endpoint: string, data: T_Req): Promise<DspySuccessResponse<T_ResData>> {
    const operation = async () => {
      try {
        const response = await this.client.request<DspySuccessResponse<T_ResData>>({
          method,
          url: endpoint,
          data,
          headers: {
            'X-API-Key': this.apiKey, // APIキーをヘッダーに付与
          },
        });
        return response.data;
      } catch (error) {
        const axiosError = error as AxiosError<DspyErrorResponse>;
        if (axiosError.response) {
          // DSPyブリッジサービスがエラーレスポンスを返した場合
          console.error(`DSPy API Error (${axiosError.response.status}) for ${endpoint}:`, axiosError.response.data);
          // 特定のエラーコードやステータスコードでリトライを制御可能
          if (axiosError.response.status === 503 || axiosError.response.status === 504) { // Service Unavailable or Gateway Timeout
            throw axiosError; // リトライ対象のエラーとしてスロー
          }
        } else if (axiosError.request) {
          // リクエストは行われたがレスポンスがない場合 (ネットワークエラーなど)
          console.error(`DSPy API No Response for ${endpoint}:`, axiosError.message);
          throw axiosError; // リトライ対象
        } else {
          // リクエスト設定時のエラー
          console.error(`DSPy API Request Setup Error for ${endpoint}:`, axiosError.message);
        }
        // リトライ対象外のエラーはここで処理または再スロー
        throw new Error(`Failed to call DSPy service at ${endpoint}: ${axiosError.message}`);
      }
    };

    try {
      return await backOff(operation, {
        numOfAttempts: 3, // 最大リトライ回数
        startingDelay: 1000, // 初期遅延 (ms)
        timeMultiple: 2, // 遅延時間の乗数
        jitter: 'full', // ジッター
        retry: (e: any, attemptNumber: number) => {
          console.warn(`Retrying DSPy API call to ${endpoint}, attempt ${attemptNumber}...`);
          return true; // すべてのエラーでリトライを試みる (より詳細な制御も可能)
        },
      });
    } catch (finalError: any) {
      console.error(`DSPy API call to ${endpoint} failed after multiple retries:`, finalError.message);
      // フォールバック処理やエラーの再スロー
      // 例: デフォルトのプロンプトを返す、エラーを上位に伝播するなど
      throw finalError; 
    }
  }

  async optimizePrompt(payload: DspyOptimizePromptRequest): Promise<DspySuccessResponse<DspyOptimizePromptResponseData>> {
    return this.requestWithRetry<'post', DspyOptimizePromptRequest, DspyOptimizePromptResponseData>('post', '/dspy/optimize_prompt', payload);
  }

  async evaluateOptions(payload: DspyEvaluateOptionsRequest): Promise<DspySuccessResponse<DspyEvaluateOptionsResponseData>> {
    return this.requestWithRetry<'post', DspyEvaluateOptionsRequest, DspyEvaluateOptionsResponseData>('post', '/dspy/evaluate_options', payload);
  }
  
  // 他のDSPyモジュール呼び出しメソッドも同様に実装
  // async generateCode(...): Promise<...>
  // async generateDocumentDraft(...): Promise<...>
}

// src/agents/enterprise-architect-agent.ts (一部抜粋)
import { BaseAgent, tool, AgentContext } from "@mastra/core"; // Mastraの型を仮定
import { DspyBridgeService, DspyOptimizePromptRequest, DspyEvaluateOptionsRequest, DspySuccessResponse, DspyOptimizePromptResponseData, DspyEvaluateOptionsResponseData } from "../services/dspy-bridge.service";

interface EnterpriseArchitectAgentRuntimeContext extends AgentContext { // AgentContextはMastraの基本型と仮定
  dspyBridge: DspyBridgeService;
  // ... 他のプロパティ
}

export class EnterpriseArchitectAgent extends BaseAgent<EnterpriseArchitectAgentRuntimeContext> {
  constructor(context: EnterpriseArchitectAgentRuntimeContext) {
    super(context);
  }

  @tool({ description: "ビジネス要件を分析し、戦略的質問を生成します" })
  async analyzeRequirementsAndGenerateQuestions(businessGoal: string, existingInfo: string[]): Promise<string[]> {
    const taskDescription = `ユーザーのビジネス目標「${businessGoal}」を深掘りし、戦略策定に必要な情報を引き出すための質問を生成する。既存情報: ${existingInfo.join(', ')}`;
    const fewShotExamples = [
      { input: "新市場への参入戦略", output: "ターゲット顧客セグメントは？主要な競合は？規制上の課題は？" },
      { input: "既存システムのモダナイゼーション", output: "現在のシステムの課題は？移行期間のビジネス継続性はどう担保する？予算上限は？" },
    ];

    try {
      const response: DspySuccessResponse<DspyOptimizePromptResponseData> = await this.context.dspyBridge.optimizePrompt({
        task_description: taskDescription,
        few_shot_examples: fewShotExamples,
        optimizer_type: 'BootstrapFewShot',
        // 必要なら追加パラメータ: num_candidate_programs, num_threads など
      });
      
      // ここでは最適化されたプロンプト自体が質問群であると仮定
      // 実際には、最適化されたプロンプトを使って再度LLMに質問生成を依頼するステップが必要な場合もある
      const generatedQuestions = response.data.optimized_prompt.split('?').map(q => q.trim() + '?').filter(q => q.length > 1);
      this.logger.info(`Generated strategic questions using DSPy: ${generatedQuestions.join('; ')}`);
      return generatedQuestions;
    } catch (error: any) {
      this.logger.error(`Failed to generate strategic questions via DSPy: ${error.message}`);
      // フォールバック: 事前に定義された汎用的な質問リストを返す
      return [
        "この戦略の具体的な成功指標(KPI)は何ですか？",
        "想定される主要なリスクと、その対策はありますか？",
        "この戦略を実行するための主要なステークホルダーは誰ですか？"
      ];
    }
  }

  @tool({ description: "複数のIT戦略オプションを評価します" })
  async evaluateStrategicOptions(options: Array<{ name: string; details: string }>): Promise<DspyEvaluateOptionsResponseData | null> {
    const evaluationCriteria = ["コスト効率", "市場投入までの時間", "技術的実現可能性", "将来の拡張性", "セキュリティリスク"];
    try {
      const response = await this.context.dspyBridge.evaluateOptions({
        options,
        evaluation_criteria: evaluationCriteria,
        module_type: 'MultiChainComparison',
        knowledge_base_query: "関連する業界のIT戦略事例と成功要因"
      });
      this.logger.info(`Strategic options evaluated by DSPy. Summary: ${response.data.evaluation_summary}`);
      return response.data;
    } catch (error: any) {
      this.logger.error(`Failed to evaluate strategic options via DSPy: ${error.message}`);
      return null; // またはエラー情報を構造化して返す
    }
  }
}
```

### 7.2. DSPyブリッジサービス (Python/FastAPI) の詳細実装例

```python
# dspy_bridge/main.py
from fastapi import FastAPI, HTTPException, Depends, Security
from fastapi.security.api_key import APIKeyHeader
from pydantic import BaseModel, Field
from typing import List, Dict, Any, Literal
import dspy
import os
import time
import uuid
import logging

# --- ロギング設定 ---
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# --- APIキー認証 ---
API_KEY = os.environ.get("DSPY_BRIDGE_API_KEY", "your-secret-api-key") # 環境変数から取得推奨
API_KEY_NAME = "X-API-Key"
api_key_header = APIKeyHeader(name=API_KEY_NAME, auto_error=True)

async def get_api_key(key: str = Security(api_key_header)):
    if key == API_KEY:
        return key
    else:
        raise HTTPException(
            status_code=403,
            detail="Could not validate credentials",
        )

app = FastAPI(
    title="DSPy Bridge Service",
    description="Provides access to DSPy modules and optimizers via REST API for Mastra.AI Agents.",
    version="1.0.0"
)

# --- DSPy設定 (環境変数からLLM APIキーなどを設定することを想定) ---
try:
    # 例: OpenAI GPT-3.5 Turbo をデフォルトLLMとして設定
    # 実際のプロジェクトでは、より柔軟なLLM選択メカニズムが必要になる場合がある
    llm = dspy.OpenAI(model='gpt-3.5-turbo-instruct', api_key=os.environ.get("OPENAI_API_KEY"))
    dspy.settings.configure(lm=llm)
    logger.info("DSPy configured with default LLM.")
except Exception as e:
    logger.error(f"Failed to configure DSPy: {e}")
    # DSPyが設定できない場合はアプリケーションを起動させないか、エラーを返すなどの処理が必要

# --- リクエスト・レスポンスモデル (Pydantic) ---
class FewShotExample(BaseModel):
    input: str
    output: str

class OptimizePromptRequest(BaseModel):
    task_description: str = Field(..., description="Optimization target task description")
    few_shot_examples: List[FewShotExample] = Field(..., description="Few-shot examples for optimization")
    optimizer_type: Literal['BootstrapFewShot', 'BayesianSignatureOptimizer', 'MIPRO'] = Field(..., description="DSPy optimizer to use")
    # オプティマイザごとの追加パラメータ (例)
    num_candidate_programs: int = Field(default=5, description="For BootstrapFewShot: Number of candidate programs to generate")
    num_threads: int = Field(default=4, description="For BootstrapFewShot: Number of threads for generation")

class OptimizePromptResponseData(BaseModel):
    optimized_prompt: str
    evaluation_metrics: Dict[str, Any] | None = None

class EvaluateOptionItem(BaseModel):
    name: str
    details: str
    # 他の属性も許容
    class Config:
        extra = "allow"

class EvaluateOptionsRequest(BaseModel):
    options: List[EvaluateOptionItem]
    evaluation_criteria: List[str]
    module_type: Literal['MultiChainComparison']
    knowledge_base_query: str | None = None

class EvaluatedOptionDetail(BaseModel):
    option_name: str
    scores: Dict[str, Any] # criteriaごとのスコアやテキスト評価
    pros: List[str]
    cons: List[str]
    reasoning: str

class EvaluateOptionsResponseData(BaseModel):
    evaluation_summary: str
    detailed_comparison: List[EvaluatedOptionDetail]

class StandardSuccessResponse(BaseModel):
    data: Any
    request_id: str
    execution_time_ms: float

# --- DSPyシグネチャとモジュールの定義例 ---

# プロンプト最適化のための汎用シグネチャ
class GenerateInstruction(dspy.Signature):
    """Given a task description and a few examples, generate an optimized instruction (prompt) for an LLM to perform that task."""
    task_description = dspy.InputField(desc="Description of the task the LLM should perform.")
    few_shot_examples = dspy.InputField(desc="Examples of input/output for the task.")
    optimized_instruction = dspy.OutputField(desc="A clear, concise, and effective instruction for the LLM.")

# 戦略オプション評価のためのシグネチャ (MultiChainComparison用)
class EvaluateStrategyOption(dspy.Signature):
    """Evaluate a given IT strategy option based on multiple criteria, providing pros, cons, and a reasoning. Refer to the knowledge base if provided."""
    option_name = dspy.InputField(desc="Name of the strategy option.")
    option_details = dspy.InputField(desc="Detailed description of the strategy option.")
    evaluation_criteria = dspy.InputField(desc="Comma-separated list of criteria to evaluate against.")
    knowledge_base_context = dspy.InputField(desc="Relevant information from knowledge base, if any.", optional=True)
    
    score_summary = dspy.OutputField(desc="A summary of scores for each criterion.")
    pros = dspy.OutputField(desc="List of advantages of this option.")
    cons = dspy.OutputField(desc="List of disadvantages of this option.")
    reasoning = dspy.OutputField(desc="Overall reasoning for the evaluation.")

# --- APIエンドポイント ---
@app.post("/dspy/optimize_prompt", response_model=StandardSuccessResponse, dependencies=[Depends(get_api_key)])
async def optimize_prompt_endpoint(request: OptimizePromptRequest):
    request_id = str(uuid.uuid4())
    start_time = time.time()
    logger.info(f"Request ID {request_id}: Received optimize_prompt request with optimizer {request.optimizer_type}")

    try:
        # DSPyオプティマイザの選択と実行
        # ここではBootstrapFewShotの例を示す。実際にはリクエストに応じて分岐する。
        if request.optimizer_type == 'BootstrapFewShot':
            # DSPyの`dspy.Example`形式に変換
            trainset = [dspy.Example(input=ex.input, output=ex.output).with_inputs('input') for ex in request.few_shot_examples]
            
            # 最適化の実行 (ここでは単純なPredictを最適化対象とする例)
            # 実際には、より複雑なDSPyプログラム (ChainOfThoughtなど) を最適化対象にできる
            optimizer = dspy.BootstrapFewShot(metric=None, max_bootstrapped_demos=request.num_candidate_programs, num_threads=request.num_threads) # metricはタスクに応じて定義
            # 最適化対象のプログラム (ここでは単純なPredict)
            optimized_program = optimizer.compile(student=dspy.Predict(GenerateInstruction), trainset=trainset, valset=None) # valsetも用意するのが望ましい
            
            # 最適化されたプロンプト (instruction) を取得する方法は、最適化対象のプログラム構造による
            # この例ではGenerateInstructionシグネチャの`optimized_instruction`がそれにあたると仮定し、
            # 実際には最適化されたプログラム (optimized_program) を使って再度推論を実行し、その結果のプロンプトを得るか、
            # または、optimizerが直接最適化されたプロンプト文字列を返すような仕組みをDSPy側で用意する必要があるかもしれない。
            # ここでは仮に、最適化されたプログラムの最初のデモンストレーションの出力をプロンプトとする
            # (これはBootstrapFewShotの動作とは異なる可能性が高い。実際のDSPyのAPIに合わせる必要あり)
            if hasattr(optimized_program, 'demos') and optimized_program.demos:
                 # 非常に単純化された例。実際にはoptimizer.compileの結果の構造を確認し、適切に抽出する。
                 # GenerateInstructionの例では、studentがPredict(GenerateInstruction)なので、
                 # student.forward(task_description=..., few_shot_examples=...) のようにして、optimized_instructionを得る。
                 # ここでは仮の値を返す。
                optimized_prompt_text = f"Optimized for: {request.task_description} (Example from BootstrapFewShot output)"
                # TODO: 実際の最適化済みプロンプト取得ロジックに置き換える
                # 例: optimized_predictor = optimized_program.predictor
                #    response = optimized_predictor(task_description=request.task_description, few_shot_examples=str(request.few_shot_examples))
                #    optimized_prompt_text = response.optimized_instruction
            else:
                optimized_prompt_text = f"Default prompt for: {request.task_description} (Optimization failed or no demos)"

            response_data = OptimizePromptResponseData(optimized_prompt=optimized_prompt_text)
        else:
            raise HTTPException(status_code=400, detail=f"Optimizer type '{request.optimizer_type}' not yet implemented.")

        execution_time_ms = (time.time() - start_time) * 1000
        logger.info(f"Request ID {request_id}: optimize_prompt completed in {execution_time_ms:.2f} ms")
        return StandardSuccessResponse(data=response_data, request_id=request_id, execution_time_ms=execution_time_ms)
    
    except dspy.dsp.errors.DSPyError as e:
        logger.error(f"Request ID {request_id}: DSPy specific error during optimize_prompt: {e}")
        raise HTTPException(status_code=500, detail=f"DSPy module execution error: {str(e)}")
    except Exception as e:
        logger.error(f"Request ID {request_id}: Unexpected error during optimize_prompt: {e}", exc_info=True)
        raise HTTPException(status_code=500, detail=f"Internal server error: {str(e)}")

@app.post("/dspy/evaluate_options", response_model=StandardSuccessResponse, dependencies=[Depends(get_api_key)])
async def evaluate_options_endpoint(request: EvaluateOptionsRequest):
    request_id = str(uuid.uuid4())
    start_time = time.time()
    logger.info(f"Request ID {request_id}: Received evaluate_options request.")

    try:
        if request.module_type == 'MultiChainComparison':
            # MultiChainComparisonはDSPyの標準モジュールではないため、カスタムで実装するか、
            # 複数のChainOfThoughtを並行実行し結果を統合するロジックをここに記述する。
            # ここでは、各オプションに対してEvaluateStrategyOptionシグネチャを持つChainOfThoughtを実行する例を示す。
            
            evaluator = dspy.ChainOfThought(EvaluateStrategyOption)
            detailed_results: List[EvaluatedOptionDetail] = []
            knowledge_context = ""
            if request.knowledge_base_query:
                # 仮のRAGモジュール呼び出し (実際にはdspy.Retrieveなどを利用)
                retriever = dspy.Retrieve(k=1) # 簡単のためk=1
                # retrieved_docs = retriever(request.knowledge_base_query).passages # dspy 0.1.x
                # dspy 1.0.x以降では retriever(request.knowledge_base_query).passages[0] など
                # ここでは仮の知識をセット
                # knowledge_context = f"Knowledge base context for query '{request.knowledge_base_query}': [Retrieved info placeholder]"
                # dspy.Retrieveの結果を文字列にする必要がある
                # results = retriever(request.knowledge_base_query)
                # knowledge_context = "\n".join(results.passages) if results.passages else "No relevant knowledge found."
                # dspy.settings.rm = ... (retrieval modelの設定が必要)
                pass # RAGの具体的な実装は省略

            for option in request.options:
                try:
                    response = evaluator(option_name=option.name, 
                                         option_details=option.details, 
                                         evaluation_criteria=", ".join(request.evaluation_criteria),
                                         knowledge_base_context=knowledge_context if knowledge_context else None)
                    
                    # responseから各フィールドを抽出 (OutputFieldで定義した名前でアクセス)
                    detailed_results.append(EvaluatedOptionDetail(
                        option_name=option.name,
                        scores=response.score_summary, # これがDict[str, Any]であることを期待
                        pros=response.pros.split('\n') if isinstance(response.pros, str) else response.pros, # LLMの出力形式による
                        cons=response.cons.split('\n') if isinstance(response.cons, str) else response.cons,
                        reasoning=response.reasoning
                    ))
                except Exception as e:
                    logger.error(f"Error evaluating option {option.name}: {e}")
                    detailed_results.append(EvaluatedOptionDetail(
                        option_name=option.name,
                        scores={"error": str(e)},
                        pros=["Evaluation failed"],
                        cons=["Evaluation failed"],
                        reasoning=f"Failed to evaluate due to: {e}"
                    ))
            
            summary = f"Evaluated {len(request.options)} options. See detailed comparison."
            response_data = EvaluateOptionsResponseData(evaluation_summary=summary, detailed_comparison=detailed_results)
        else:
            raise HTTPException(status_code=400, detail=f"Module type '{request.module_type}' not yet implemented.")

        execution_time_ms = (time.time() - start_time) * 1000
        logger.info(f"Request ID {request_id}: evaluate_options completed in {execution_time_ms:.2f} ms")
        return StandardSuccessResponse(data=response_data, request_id=request_id, execution_time_ms=execution_time_ms)

    except dspy.dsp.errors.DSPyError as e:
        logger.error(f"Request ID {request_id}: DSPy specific error during evaluate_options: {e}")
        raise HTTPException(status_code=500, detail=f"DSPy module execution error: {str(e)}")
    except Exception as e:
        logger.error(f"Request ID {request_id}: Unexpected error during evaluate_options: {e}", exc_info=True)
        raise HTTPException(status_code=500, detail=f"Internal server error: {str(e)}")

# 他のエンドポイント (コード生成、ドキュメント生成など) も同様に定義
# 例: /dspy/generate_code, /dspy/generate_document_draft

if __name__ == "__main__":
    import uvicorn
    # 環境変数からAPIキーを設定して実行:
    # OPENAI_API_KEY="sk-..." DSPY_BRIDGE_API_KEY="mysecretkey" python dspy_bridge/main.py
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### 7.3. DSPyブリッジAPIのOpenAPI Specification (OAS 3.0) 例

以下は、上記FastAPIコードから自動生成されるOpenAPIドキュメントの主要部分のイメージです。(FastAPIは`/docs`エンドポイントでSwagger UI、`/redoc`でReDocを自動提供します)

```yaml
openapi: 3.0.0
info:
  title: DSPy Bridge Service
  description: Provides access to DSPy modules and optimizers via REST API for Mastra.AI Agents.
  version: 1.0.0

components:
  securitySchemes:
    APIKeyAuth:
      type: apiKey
      in: header
      name: X-API-Key
  schemas:
    FewShotExample:
      type: object
      properties:
        input: { type: string }
        output: { type: string }
      required: [input, output]

    OptimizePromptRequest:
      type: object
      properties:
        task_description: { type: string, description: "Optimization target task description" }
        few_shot_examples:
          type: array
          items: { $ref: "#/components/schemas/FewShotExample" }
          description: "Few-shot examples for optimization"
        optimizer_type:
          type: string
          enum: ["BootstrapFewShot", "BayesianSignatureOptimizer", "MIPRO"]
          description: "DSPy optimizer to use"
        num_candidate_programs: { type: integer, default: 5, description: "For BootstrapFewShot..." }
        num_threads: { type: integer, default: 4, description: "For BootstrapFewShot..." }
      required: [task_description, few_shot_examples, optimizer_type]

    OptimizePromptResponseData:
      type: object
      properties:
        optimized_prompt: { type: string }
        evaluation_metrics: { type: object, additionalProperties: true, nullable: true }
    
    # ... (EvaluateOptionsRequest, EvaluateOptionsResponseDataなどのスキーマも同様に定義)

    StandardSuccessResponse_OptimizePromptResponseData_:
      type: object
      properties:
        data: { $ref: "#/components/schemas/OptimizePromptResponseData" }
        request_id: { type: string, format: uuid }
        execution_time_ms: { type: number }
      required: [data, request_id, execution_time_ms]

    HTTPValidationError:
      type: object
      properties:
        detail:
          type: array
          items:
            type: object
            properties:
              loc: { type: array, items: { type: string } }
              msg: { type: string }
              type: { type: string }

security:
  - APIKeyAuth: []

paths:
  /dspy/optimize_prompt:
    post:
      summary: Optimize Prompt Endpoint
      operationId: optimize_prompt_endpoint_dspy_optimize_prompt_post
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/OptimizePromptRequest"
        required: true
      responses:
        '200':
          description: Successful Response
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/StandardSuccessResponse_OptimizePromptResponseData_"
        '403':
          description: Forbidden (Could not validate credentials)
        '422':
          description: Validation Error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/HTTPValidationError"
        '500':
          description: Internal Server Error (DSPy module execution error or unexpected error)
          content:
            application/json:
              schema:
                type: object
                properties:
                  detail: { type: string }
  
  # ... (/dspy/evaluate_options エンドポイントも同様に定義)

```

**注記:**

*   上記のPython/FastAPIコードおよびOASの例は、主要な概念を示すためのものであり、実際の運用にはさらなる詳細化（より堅牢なエラーハンドリング、ロギング、DSPyモジュールの網羅的な実装、具体的なLLMやRetrieverの設定、非同期タスクキューの利用検討など）が必要です。
*   DSPyのAPIやベストプラクティスは進化する可能性があるため、最新のDSPyドキュメントを参照することが重要です。
*   `MultiChainComparison`のような複雑なモジュールは、DSPyの基本コンポーネントを組み合わせてカスタム実装するか、DSPyコミュニティで提供される拡張機能を探す必要があります。
*   Mastra.AI Agent側の`DspyBridgeService`は、生成されたOASからクライアントコードを自動生成するツール（例: `openapi-typescript-codegen`）を利用して作成することも可能です。

これらの詳細なコード例とAPI仕様は、Mastra.AIとDSPyを連携させる際の具体的な実装イメージを提供し、開発者が迅速に作業を開始できるようにすることを目的としています。

