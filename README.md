# 要件定義ジェネレータ PoC

当リポジトリは、Dify と n8n を組み合わせた「要件定義ジェネレータ」の簡易 PoC (Proof of Concept) です。ユーザーが自社 Web サイト上のチャットウィジェットを通じてプロジェクト概要や主要機能を入力すると、自動的に要件定義ドラフトが生成され、返却されます。

---
## 目的

- 要件定義からプロトタイプ作成までのリードタイム短縮
- プロトタイプ品質の向上
- 最小限の開発工数で機能検証し、将来的な拡張性（自前フォーム、Typeform 連携、LangChain カスタムノード化など）を担保

---
## コンポーネント構成

1. **フロントエンド**: Dify チャットウィジェット（Static Site でホスティング）
2. **ワークフローエンジン**: n8n (Docker)
3. **自律エージェント (外部)**: LangChain API（PoCでは n8n の HTTP Request ノード経由で呼び出し）

---
## フォルダ構成

以下のようにインデントしたテキスト（4スペース）で記述すると、GitHub でもフォーマットが崩れず表示されます。

    project-root/
    ├── n8n/
    │   ├── data/               # n8n 永続化データ
    │   └── docker-compose.yml  # n8n 用 Docker Compose
    ├── frontend/               # (任意) Static Site フォルダ
    │   └── index.html          # Dify チャットウィジェット埋め込み
    └── README.md               # 本ドキュメント

---
## 前提条件

- Docker, Docker Compose がインストール済み
- Dify アカウントおよび該当プロジェクトの Project ID / API Key を取得済み
- HTTPS でアクセス可能なドメイン、もしくは ngrok/localtunnel のトンネル URL
- GitHub リポジトリを作成済み（初回 Push 用）

---
## セットアップ手順

1. **リポジトリのクローン**
    ```bash
    git clone <リポジトリURL>
    cd project-root/n8n
    ```
2. **n8n コンテナ起動**
    ```bash
    docker-compose up -d
    ```
3. **フロントエンドのホスティング (任意)**
    - `frontend/index.html` に Dify チャットウィジェットのスクリプトを記載
    - Netlify などでデプロイし、HTTPS を有効化
4. **Dify Flow の Webhook 設定**
    - Webhook URL: `https://<あなたのドメイン>/webhook/dify`
5. **n8n ワークフロー作成**
    1. Webhook ノード: `/webhook/dify` 受信設定
    2. HTTP Request ノード: Dify API 呼び出し（chat/completions）
    3. Function ノード: 応答本文の抽出・整形
    4. Respond to Webhook ノード: 整形結果を返却
6. **GitHub 初回 Push**
    ```bash
    cd ../
    git init
    git add .
    git commit -m "Initial PoC setup: Dify + n8n"
    git remote add origin git@github.com:<your-org>/<repo>.git
    git branch -M main
    git push -u origin main
    ```

---
## テスト手順

1. 自社サイトに埋め込んだ Dify チャットウィジェットを開く
2. 「要件定義ジェネレータへようこそ！」のプロンプトに従い、プロジェクト概要と主要機能を入力
3. 送信後、n8n が Dify API を呼び出し、要件定義ドラフトを生成
4. ウィジェット上で生成されたドラフトを確認

---
## 今後の拡張ロードマップ

1. **Fast Path**: Dify → LangChain を直結し、即時応答を実現（n8n は非同期処理専用化）
2. **メモリ／ストリーミング対応**: ConversationSummaryMemory の導入、Dify API のストリーミングレスポンス利用
3. **UI刷新**: 自前フォームまたは Typeform への移行検討
4. **外部ツール連携強化**: ClickUp/GitHub Issue 自動登録、CRM 連携、Analytics トラッキング

---
## パフォーマンス最適化

- **キャッシュ利用**: Redis などでよくあるプロンプト結果をキャッシュし、再利用時に高速返却
- **コネクション最適化**: n8n ホストとエージェントサーバーを同一リージョンに配置し、HTTP Keep-Alive を有効化
- **ストリーミング**: Dify API の `stream` パラメータを true にして部分出力を実装

---
## メモリ対応

- **短期メモリ (ConversationBufferMemory)**: 直近の対話履歴を保持
- **要約メモリ (ConversationSummaryMemory)**: 長期対話の要約と履歴管理
- **ベクトルメモリ (VectorStoreMemory)**: Chroma/Weaviate による履歴検索と参照
