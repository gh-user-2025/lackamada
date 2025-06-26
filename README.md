# 工場設備管理アプリ プロトタイプ

AI-Driven Development による工場設備管理アプリのデモプロジェクトです。

## プロジェクト概要

このプロジェクトは、工場内の設備の稼働状況やメンテナンス情報を一元管理し、効率的な運用をサポートする工場設備管理アプリのプロトタイプです。IoTセンサーからのデータをリアルタイムで監視し、予防保全やデータ分析を通じて生産性向上を実現します。

### 主な機能
- 🔄 **リアルタイム監視**: IoTセンサーによる設備稼働状況の監視
- 🔧 **メンテナンス管理**: 予防保全スケジュールと履歴管理
- 📊 **データ分析**: 運用データの分析と改善提案
- 🚨 **異常検知**: AI による異常パターンの検出とアラート

### 使用技術
- **Azure Functions**: サーバーレスデータ処理
- **Azure SQL Database**: 構造化データ保存
- **Azure Cosmos DB**: リアルタイムデータ処理
- **Power BI**: データ可視化
- **Python**: メイン開発言語

## クイックスタート

### 1. リポジトリのクローン
```bash
git clone https://github.com/gh-user-2025/lackamada.git
cd lackamada
```

### 2. ドキュメントの確認
- 📖 [プロジェクト概要](./docs/project-overview.md) - 詳細な機能説明とアーキテクチャ
- ☁️ [Azure セットアップ](./docs/azure-setup.md) - Azure リソースの作成手順
- 🛠️ [開発ガイド](./docs/development-guide.md) - 開発環境構築と開発手順

### 3. Azure リソースの作成
```bash
# Azure CLI でログイン
az login

# リソースグループの作成
az group create --name rg-factory-equipment-demo --location japaneast
```

詳細な手順は [Azure セットアップガイド](./docs/azure-setup.md) をご確認ください。

## プロジェクト構成

```
lackamada/
├── docs/                    # プロジェクトドキュメント
│   ├── project-overview.md  # プロジェクト概要
│   ├── azure-setup.md       # Azure リソース作成手順
│   └── development-guide.md # 開発ガイド
├── src/                     # ソースコード（開発予定）
├── tests/                   # テストコード（開発予定）
└── README.md               # このファイル
```

## 開発手法

このプロジェクトでは **AI-Driven Development** を採用しています：
- AI支援によるコード生成とレビュー
- 自動化されたテストとデプロイメント
- データドリブンな意思決定

## 貢献方法

1. このリポジトリをフォーク
2. 新しいブランチを作成 (`git checkout -b feature/amazing-feature`)
3. 変更をコミット (`git commit -m 'Add amazing feature'`)
4. ブランチにプッシュ (`git push origin feature/amazing-feature`)
5. プルリクエストを作成

## ライセンス

このプロジェクトは MIT ライセンスの下で公開されています。詳細は [LICENSE](LICENSE) ファイルをご確認ください。

## サポート

質問や問題がある場合は、[Issues](https://github.com/gh-user-2025/lackamada/issues) でお知らせください。

---

> **Note**: このプロジェクトはデモ目的で作成されており、実際の工場環境での使用には追加のセキュリティ対策と最適化が必要です。
