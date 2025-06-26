# Azure リソース セットアップ ガイド

この文書では、工場設備管理アプリのプロトタイプ開発に必要なAzureリソースの作成手順を説明します。

## 前提条件

### 必要なツール
- Azure CLI のインストール
- Azureサブスクリプション（無料版でも可）
- PowerShell または Bash 環境

### Azure CLI のインストール

#### Windows の場合
```bash
# PowerShell で実行
Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'
```

#### macOS の場合
```bash
# Homebrew を使用
brew update && brew install azure-cli
```

#### Linux の場合
```bash
# Ubuntu/Debian
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

## セットアップ手順

### 1. Azure へのログイン

```bash
# Azure CLI でログイン
az login

# サブスクリプション一覧の確認
az account list --output table

# 使用するサブスクリプションの設定（必要に応じて）
az account set --subscription "<サブスクリプション名またはID>"
```

### 2. リソースグループの作成

```bash
# 変数の設定（お好みの値に変更してください）
RESOURCE_GROUP_NAME="rg-factory-equipment-demo"
LOCATION="japaneast"
PROJECT_NAME="factory-equipment"

# リソースグループの作成
az group create \
  --name $RESOURCE_GROUP_NAME \
  --location $LOCATION

# 作成確認
az group show --name $RESOURCE_GROUP_NAME --output table
```

### 3. Azure SQL Database の作成

```bash
# SQL Server とデータベースの変数設定
SQL_SERVER_NAME="${PROJECT_NAME}-sql-server"
SQL_DATABASE_NAME="${PROJECT_NAME}-db"
SQL_ADMIN_USER="sqladmin"
SQL_ADMIN_PASSWORD="P@ssw0rd123!"  # 実際の運用では安全なパスワードを使用

# SQL Server の作成
az sql server create \
  --name $SQL_SERVER_NAME \
  --resource-group $RESOURCE_GROUP_NAME \
  --location $LOCATION \
  --admin-user $SQL_ADMIN_USER \
  --admin-password $SQL_ADMIN_PASSWORD

# ファイアウォール規則の作成（Azure サービスからのアクセス許可）
az sql server firewall-rule create \
  --resource-group $RESOURCE_GROUP_NAME \
  --server $SQL_SERVER_NAME \
  --name "AllowAzureServices" \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0

# 開発用ファイアウォール規則（お使いのIPアドレスを確認して設定）
# YOUR_IP_ADDRESS=$(curl -s https://ifconfig.me)
# az sql server firewall-rule create \
#   --resource-group $RESOURCE_GROUP_NAME \
#   --server $SQL_SERVER_NAME \
#   --name "AllowMyIP" \
#   --start-ip-address $YOUR_IP_ADDRESS \
#   --end-ip-address $YOUR_IP_ADDRESS

# SQL Database の作成
az sql db create \
  --resource-group $RESOURCE_GROUP_NAME \
  --server $SQL_SERVER_NAME \
  --name $SQL_DATABASE_NAME \
  --service-objective Basic
```

### 4. Azure Cosmos DB の作成

```bash
# Cosmos DB アカウントの変数設定
COSMOS_ACCOUNT_NAME="${PROJECT_NAME}-cosmos"
COSMOS_DATABASE_NAME="SensorData"
COSMOS_CONTAINER_NAME="Equipment"

# Cosmos DB アカウントの作成
az cosmosdb create \
  --name $COSMOS_ACCOUNT_NAME \
  --resource-group $RESOURCE_GROUP_NAME \
  --locations regionName=$LOCATION \
  --default-consistency-level Session

# データベースの作成
az cosmosdb sql database create \
  --account-name $COSMOS_ACCOUNT_NAME \
  --resource-group $RESOURCE_GROUP_NAME \
  --name $COSMOS_DATABASE_NAME

# コンテナーの作成
az cosmosdb sql container create \
  --account-name $COSMOS_ACCOUNT_NAME \
  --database-name $COSMOS_DATABASE_NAME \
  --resource-group $RESOURCE_GROUP_NAME \
  --name $COSMOS_CONTAINER_NAME \
  --partition-key-path "/equipmentId" \
  --throughput 400
```

### 5. Azure Functions の作成

```bash
# Storage Account の変数設定
STORAGE_ACCOUNT_NAME="${PROJECT_NAME}storage$(date +%s)"  # ユニーク名を生成
FUNCTION_APP_NAME="${PROJECT_NAME}-functions"

# Storage Account の作成
az storage account create \
  --name $STORAGE_ACCOUNT_NAME \
  --resource-group $RESOURCE_GROUP_NAME \
  --location $LOCATION \
  --sku Standard_LRS

# Function App の作成
az functionapp create \
  --resource-group $RESOURCE_GROUP_NAME \
  --consumption-plan-location $LOCATION \
  --runtime python \
  --runtime-version 3.11 \
  --functions-version 4 \
  --name $FUNCTION_APP_NAME \
  --storage-account $STORAGE_ACCOUNT_NAME
```

### 6. Azure IoT Hub の作成

```bash
# IoT Hub の変数設定
IOT_HUB_NAME="${PROJECT_NAME}-iot-hub"

# IoT Hub の作成
az iot hub create \
  --resource-group $RESOURCE_GROUP_NAME \
  --name $IOT_HUB_NAME \
  --location $LOCATION \
  --sku F1

# IoT デバイスの作成（サンプル用）
az iot hub device-identity create \
  --device-id "equipment-001" \
  --hub-name $IOT_HUB_NAME \
  --resource-group $RESOURCE_GROUP_NAME
```

### 7. Application Insights の作成

```bash
# Application Insights の変数設定
APP_INSIGHTS_NAME="${PROJECT_NAME}-insights"

# Application Insights の作成
az monitor app-insights component create \
  --app $APP_INSIGHTS_NAME \
  --location $LOCATION \
  --resource-group $RESOURCE_GROUP_NAME
```

### 8. 接続文字列の取得

```bash
echo "=== 接続文字列とキー ==="

# SQL Database の接続文字列
echo "SQL Database 接続文字列:"
az sql db show-connection-string \
  --server $SQL_SERVER_NAME \
  --name $SQL_DATABASE_NAME \
  --client ado.net

# Cosmos DB の接続文字列
echo "Cosmos DB 接続文字列:"
az cosmosdb keys list \
  --name $COSMOS_ACCOUNT_NAME \
  --resource-group $RESOURCE_GROUP_NAME \
  --type connection-strings \
  --query "connectionStrings[0].connectionString" \
  --output tsv

# IoT Hub の接続文字列
echo "IoT Hub 接続文字列:"
az iot hub connection-string show \
  --hub-name $IOT_HUB_NAME \
  --output tsv

# Application Insights のインストルメンテーションキー
echo "Application Insights キー:"
az monitor app-insights component show \
  --app $APP_INSIGHTS_NAME \
  --resource-group $RESOURCE_GROUP_NAME \
  --query "instrumentationKey" \
  --output tsv
```

## 環境変数の設定

作成したリソースの接続情報を環境変数として設定します：

```bash
# 環境変数ファイルの作成（.envファイル）
cat > .env << EOF
# Azure SQL Database
SQL_SERVER_NAME=${SQL_SERVER_NAME}.database.windows.net
SQL_DATABASE_NAME=${SQL_DATABASE_NAME}
SQL_ADMIN_USER=${SQL_ADMIN_USER}
SQL_ADMIN_PASSWORD=${SQL_ADMIN_PASSWORD}

# Azure Cosmos DB
COSMOS_ACCOUNT_NAME=${COSMOS_ACCOUNT_NAME}
COSMOS_DATABASE_NAME=${COSMOS_DATABASE_NAME}
COSMOS_CONTAINER_NAME=${COSMOS_CONTAINER_NAME}

# Azure IoT Hub
IOT_HUB_NAME=${IOT_HUB_NAME}

# Azure Functions
FUNCTION_APP_NAME=${FUNCTION_APP_NAME}

# Application Insights
APP_INSIGHTS_NAME=${APP_INSIGHTS_NAME}

# Resource Group
RESOURCE_GROUP_NAME=${RESOURCE_GROUP_NAME}
LOCATION=${LOCATION}
EOF

echo ".env ファイルが作成されました。接続文字列を追加してください。"
```

## リソースの確認

```bash
# 作成されたリソースの一覧表示
az resource list \
  --resource-group $RESOURCE_GROUP_NAME \
  --output table

# コスト見積もりの確認
az consumption usage list \
  --start-date $(date -d '1 day ago' '+%Y-%m-%d') \
  --end-date $(date '+%Y-%m-%d') \
  --output table
```

## セキュリティの設定

### 1. ネットワークセキュリティの強化

```bash
# SQL Server のファイアウォール設定を確認
az sql server firewall-rule list \
  --resource-group $RESOURCE_GROUP_NAME \
  --server $SQL_SERVER_NAME \
  --output table

# Cosmos DB のファイアウォール設定
az cosmosdb update \
  --name $COSMOS_ACCOUNT_NAME \
  --resource-group $RESOURCE_GROUP_NAME \
  --enable-public-network true \
  --ip-range-filter "0.0.0.0"  # 実際の環境では適切なIPレンジを設定
```

### 2. アクセス制御の設定

```bash
# マネージド ID の有効化（Function App）
az functionapp identity assign \
  --name $FUNCTION_APP_NAME \
  --resource-group $RESOURCE_GROUP_NAME

# Cosmos DB へのアクセス権限の付与
FUNCTION_PRINCIPAL_ID=$(az functionapp identity show \
  --name $FUNCTION_APP_NAME \
  --resource-group $RESOURCE_GROUP_NAME \
  --query principalId \
  --output tsv)

az cosmosdb sql role assignment create \
  --account-name $COSMOS_ACCOUNT_NAME \
  --resource-group $RESOURCE_GROUP_NAME \
  --role-definition-name "Cosmos DB Built-in Data Contributor" \
  --principal-id $FUNCTION_PRINCIPAL_ID \
  --scope "/"
```

## リソースの削除（クリーンアップ）

**注意: この操作は元に戻せません。**

```bash
# リソースグループごと削除（すべてのリソースが削除されます）
az group delete \
  --name $RESOURCE_GROUP_NAME \
  --yes \
  --no-wait

# 削除の確認
az group exists --name $RESOURCE_GROUP_NAME
```

## トラブルシューティング

### よくある問題と解決方法

1. **リソース名の重複エラー**
   - グローバルにユニークな名前が必要なリソース（Storage Account、Cosmos DBなど）でエラーが発生
   - 解決: 名前にタイムスタンプや乱数を追加

2. **権限エラー**
   - Azure サブスクリプションの権限不足
   - 解決: サブスクリプションの所有者または共同作成者権限を確認

3. **クォータ制限**
   - 無料サブスクリプションでのリソース制限
   - 解決: 不要なリソースを削除するか、有料プランに移行

### サポートリソース

- [Azure CLI ドキュメント](https://docs.microsoft.com/ja-jp/cli/azure/)
- [Azure アーキテクチャセンター](https://docs.microsoft.com/ja-jp/azure/architecture/)
- [Azure 料金計算ツール](https://azure.microsoft.com/ja-jp/pricing/calculator/)

## 次のステップ

1. [開発環境の構築](./development-guide.md)
2. データモデルの設計
3. Azure Functions の開発
4. IoT デバイスシミュレーターの作成