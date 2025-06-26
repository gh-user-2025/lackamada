# 開発ガイド

この文書では、工場設備管理アプリの開発環境構築と開発手順を説明します。

## 開発環境の前提条件

### 必要なソフトウェア
- **Python 3.11以上**
- **Node.js 18以上** (Azure Functions Core Tools用)
- **Visual Studio Code**
- **Git**
- **Azure CLI**
- **Azure Functions Core Tools**

## 開発環境のセットアップ

### 1. Python 環境の準備

```bash
# Python バージョンの確認
python --version

# 仮想環境の作成
python -m venv venv

# 仮想環境のアクティベート（Windows）
venv\Scripts\activate

# 仮想環境のアクティベート（macOS/Linux）
source venv/bin/activate

# pip のアップグレード
python -m pip install --upgrade pip
```

### 2. 必要なPythonパッケージのインストール

```bash
# requirements.txt の作成
cat > requirements.txt << EOF
azure-functions==1.18.0
azure-cosmos==4.5.1
azure-identity==1.15.0
azure-storage-blob==12.19.0
azure-iot-device==2.12.0
pyodbc==5.0.1
pandas==2.1.4
numpy==1.26.2
python-dotenv==1.0.0
fastapi==0.104.1
uvicorn[standard]==0.24.0
pytest==7.4.3
pytest-asyncio==0.21.1
requests==2.31.0
EOF

# パッケージのインストール
pip install -r requirements.txt
```

### 3. Azure Functions Core Tools のインストール

```bash
# npm を使用してインストール
npm install -g azure-functions-core-tools@4 --unsafe-perm true
```

### 4. Visual Studio Code の拡張機能

以下の拡張機能をインストールしてください：

- **Azure Functions**
- **Python**
- **Azure Account**
- **Azure Resources**
- **REST Client**
- **Python Docstring Generator**

## プロジェクト構造

```
factory-equipment-app/
├── docs/                          # ドキュメント
│   ├── project-overview.md
│   ├── azure-setup.md
│   └── development-guide.md
├── src/                           # ソースコード
│   ├── functions/                 # Azure Functions
│   │   ├── __init__.py
│   │   ├── data_processor/        # データ処理関数
│   │   ├── anomaly_detector/      # 異常検知関数
│   │   └── alert_handler/         # アラート処理関数
│   ├── models/                    # データモデル
│   │   ├── __init__.py
│   │   ├── equipment.py
│   │   ├── sensor_data.py
│   │   └── maintenance.py
│   ├── services/                  # サービス層
│   │   ├── __init__.py
│   │   ├── cosmos_service.py
│   │   ├── sql_service.py
│   │   └── iot_service.py
│   └── utils/                     # ユーティリティ
│       ├── __init__.py
│       ├── config.py
│       └── logger.py
├── tests/                         # テスト
│   ├── __init__.py
│   ├── test_functions/
│   ├── test_models/
│   └── test_services/
├── scripts/                       # デプロイ・運用スクリプト
│   ├── deploy.sh
│   ├── setup_database.py
│   └── generate_sample_data.py
├── .env.example                   # 環境変数テンプレート
├── .gitignore
├── requirements.txt
├── README.md
└── host.json                      # Azure Functions 設定
```

## 環境変数の設定

### 1. 環境変数ファイルの作成

```bash
# .env.example から .env をコピー
cp .env.example .env

# .env ファイルを編集（実際の接続情報を設定）
cat > .env << EOF
# Azure SQL Database
SQL_CONNECTION_STRING="Server=tcp:your-server.database.windows.net,1433;Initial Catalog=your-database;Persist Security Info=False;User ID=your-username;Password=your-password;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"

# Azure Cosmos DB
COSMOS_ENDPOINT="https://your-cosmos-account.documents.azure.com:443/"
COSMOS_KEY="your-cosmos-key"
COSMOS_DATABASE_NAME="SensorData"
COSMOS_CONTAINER_NAME="Equipment"

# Azure IoT Hub
IOT_HUB_CONNECTION_STRING="HostName=your-iot-hub.azure-devices.net;SharedAccessKeyName=your-key-name;SharedAccessKey=your-key"

# Application Insights
APPINSIGHTS_INSTRUMENTATIONKEY="your-instrumentation-key"

# その他の設定
LOG_LEVEL="INFO"
ENVIRONMENT="development"
EOF
```

## 開発ワークフロー

### 1. 新機能の開発手順

```bash
# 1. 新しいブランチの作成
git checkout -b feature/new-feature-name

# 2. 仮想環境のアクティベート
source venv/bin/activate  # macOS/Linux
# または
venv\Scripts\activate     # Windows

# 3. 依存関係のインストール
pip install -r requirements.txt

# 4. 開発作業
# ... コード編集 ...

# 5. テストの実行
pytest tests/

# 6. コードの品質チェック
flake8 src/
black src/
mypy src/

# 7. 変更のコミット
git add .
git commit -m "Add new feature: description"

# 8. プッシュ
git push origin feature/new-feature-name
```

### 2. Azure Functions のローカル開発

```bash
# Functions プロジェクトの初期化
func init . --python

# 新しい関数の作成
func new --name DataProcessor --template "Timer trigger" --language python

# ローカルでの実行
func start

# 関数のテスト（別ターミナル）
curl -X POST http://localhost:7071/api/DataProcessor
```

### 3. データベースの初期化

```bash
# データベーススキーマの作成
python scripts/setup_database.py

# サンプルデータの生成
python scripts/generate_sample_data.py
```

## テスト戦略

### 1. 単体テスト

```bash
# 特定のテストファイルを実行
pytest tests/test_models/test_equipment.py -v

# カバレッジ付きでテスト実行
pytest --cov=src tests/

# 特定の関数のテスト
pytest tests/ -k "test_equipment_creation"
```

### 2. 統合テスト

```bash
# Azure リソースを使用した統合テスト
pytest tests/integration/ --azure

# モックを使用したテスト
pytest tests/integration/ --mock
```

### 3. 負荷テスト

```bash
# locust を使用した負荷テスト
pip install locust

# 負荷テストの実行
locust -f tests/load_test.py --host=http://localhost:7071
```

## デバッグ手順

### 1. Visual Studio Code でのデバッグ

`.vscode/launch.json` の設定例：

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Attach to Python Functions",
            "type": "python",
            "request": "attach",
            "port": 9091,
            "preLaunchTask": "func: host start"
        }
    ]
}
```

### 2. ログの確認

```bash
# ローカルログの確認
tail -f local.settings.json

# Azure でのログ確認
az functionapp log tail --name your-function-app --resource-group your-rg
```

## デプロイメント

### 1. 開発環境へのデプロイ

```bash
# Azure にログイン
az login

# Function App の作成（既存の場合はスキップ）
az functionapp create \
  --resource-group your-rg \
  --consumption-plan-location japaneast \
  --runtime python \
  --runtime-version 3.11 \
  --functions-version 4 \
  --name your-function-app \
  --storage-account your-storage

# デプロイの実行
func azure functionapp publish your-function-app
```

### 2. 環境変数の設定（Azure）

```bash
# Function App に環境変数を設定
az functionapp config appsettings set \
  --name your-function-app \
  --resource-group your-rg \
  --settings \
    "SQL_CONNECTION_STRING=your-connection-string" \
    "COSMOS_ENDPOINT=your-cosmos-endpoint" \
    "COSMOS_KEY=your-cosmos-key"
```

## 監視とメンテナンス

### 1. Application Insights での監視

```bash
# ログクエリの例
az monitor log-analytics query \
  --workspace your-workspace-id \
  --analytics-query "requests | where timestamp > ago(1h) | summarize count() by resultCode"
```

### 2. パフォーマンス監視

```bash
# メトリクスの確認
az monitor metrics list \
  --resource your-function-app-resource-id \
  --metric FunctionExecutionCount \
  --aggregation Count
```

## コード品質とセキュリティ

### 1. コード品質チェック

```bash
# インストール
pip install flake8 black mypy bandit

# 実行
flake8 src/           # PEP8 チェック
black src/            # コードフォーマット
mypy src/             # 型チェック
bandit -r src/        # セキュリティチェック
```

### 2. 依存関係のセキュリティチェック

```bash
# pip-audit のインストールと実行
pip install pip-audit
pip-audit

# safety を使用した脆弱性チェック
pip install safety
safety check
```

## よくある問題と解決方法

### 1. 接続エラー

**問題**: Azure リソースへの接続ができない
**解決**: 
- 環境変数の確認
- ファイアウォール設定の確認
- 認証情報の確認

### 2. パフォーマンス問題

**問題**: 関数の実行が遅い
**解決**:
- タイムアウト設定の調整
- メモリ使用量の最適化
- 非同期処理の導入

### 3. デプロイエラー

**問題**: デプロイが失敗する
**解決**:
- requirements.txt の確認
- 関数のエントリポイント確認
- Azure CLI のバージョン確認

## 参考資料

- [Azure Functions Python 開発者ガイド](https://docs.microsoft.com/ja-jp/azure/azure-functions/functions-reference-python)
- [Azure Cosmos DB Python SDK](https://docs.microsoft.com/ja-jp/azure/cosmos-db/sql/sql-api-sdk-python)
- [Azure IoT Hub Python SDK](https://docs.microsoft.com/ja-jp/azure/iot-hub/iot-hub-python-python-device-management-get-started)

## 次のステップ

1. データモデルの実装
2. Azure Functions の開発
3. IoT デバイスシミュレーターの作成
4. フロントエンドの開発（Power BI ダッシュボード）
5. テストの自動化
6. CI/CD パイプラインの構築