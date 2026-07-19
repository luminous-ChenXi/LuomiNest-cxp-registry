# LuomiNest CxPlugin レジストリ

[English](./README.md) | [简体中文](./README.zh-CN.md) | [日本語](./README.ja-JP.md)

LuomiNest プラグイン/スキルレジストリ — `index.json` で配布可能なすべてのプラグインとスキルのメタデータを集約します。

## リポジトリの役割

| リポジトリ | 役割 |
|------------|------|
| **LuomiNest-cxp-registry**（本リポジトリ） | 中心インデックス、`index.json` のみ管理 |
| LuomiNest-cxp-skills | スキル集約（各スキルはサブディレクトリ） |
| LuomiNest-cxp-plugin-template | プラグイン開発テンプレート（フォーク出発点） |
| LuomiNest-cxp-<plugin-name> | 個別プラグインリポジトリ（PDF reader、weather-query など） |

**本リポジトリは中心インデックスであり、プラグインやスキルの実際のコードは含みません。** 各リポジトリの Release アセットを指すメタデータのみを管理します。

## index.json 形式

```json
{
  "version": "1.0.0",
  "updatedAt": "2026-07-20T00:00:00Z",
  "plugins": [
    {
      "id": "cxp-pdf-reader",
      "name": "CxPlugin PDF リーダー",
      "version": "1.0.0",
      "description": "...",
      "author": { "name": "LuminousCX", "url": "..." },
      "category": "tool",
      "tags": ["pdf", "document"],
      "icon": "FileText",
      "platform": "fullstack",
      "license": "MIT",
      "minAppVersion": "0.7.6",
      "repo": "https://github.com/luminous-ChenXi/LuomiNest-cxp-PDF_reader",
      "downloadUrl": "https://github.com/.../releases/latest/download/cxp-pdf-reader.zip",
      "createdAt": "2026-07-19",
      "updatedAt": "2026-07-19"
    }
  ],
  "skills": [
    {
      "id": "travel-planner",
      "name": "旅行計画",
      ...
    }
  ]
}
```

完全なフィールド仕様は [docs/development/plugin-system.md](https://github.com/luminous-ChenXi/LuomiNest/blob/main/docs/development/plugin-system.md) を参照。

## 既存エントリ

### プラグイン

| ID | 名前 | プラットフォーム | リポジトリ |
|----|------|------------------|------------|
| `cxp-pdf-reader` | PDF スマートリーダー | fullstack | LuomiNest-cxp-PDF_reader |
| `weather-query` | 天気照会 | backend | LuomiNest-cxp-weather-query |

### スキル

| ID | 名前 | リポジトリ |
|----|------|------------|
| `cooking-assistant` | 料理アシスタント | LuomiNest-cxp-skills |
| `skill-creator` | スキル作成アシスタント | LuomiNest-cxp-skills |
| `travel-planner` | 旅行計画 | LuomiNest-cxp-skills |

## ユーザーアクセスフロー

```
LuomiNest 起動 -> バックエンドが GET raw.githubusercontent.com/.../index.json
  -> 6 時間キャッシュ
  -> ローカル静的カタログとマージ -> フロントエンドの「市場」に返却
  -> ユーザーがインストールクリック -> バックエンドが downloadUrl から zip ダウンロード -> ローカルに解凍
```

## 新規エントリの提出

### プラグイン作者向け

1. [LuomiNest-cxp-plugin-template](https://github.com/luminous-ChenXi/LuomiNest-cxp-plugin-template) で個別リポジトリ作成
2. 開発完了後、GitHub Release 作成（tag: `v<version>`、zip アセット添付）
3. 本リポジトリをフォーク -> `index.json` の `plugins` 配列に新エントリ追加
4. PR 提出、タイトル：`feat: register plugin <plugin-id> v<version>`

### スキル作者向け

1. [LuomiNest-cxp-skills](https://github.com/luminous-ChenXi/LuomiNest-cxp-skills) にスキルサブディレクトリ作成
2. PR マージ後、メンテナーに Release 作成と `skill-<id>.zip` アセット添付を依頼
3. 本リポジトリをフォーク -> `index.json` の `skills` 配列に新エントリ追加
4. PR 提出、タイトル：`feat: register skill <skill-id> v<version>`

## GitHub Actions

| Workflow | トリガー | 機能 |
|----------|----------|------|
| `validate-index.yml` | main への push/PR | `index.json` 構造バリデーション（必須フィールド、命名規則、semver、URL、重複 ID） |
| `auto-sync.yml` | `repository_dispatch` または手動 | LuomiNest バックエンドの `publish-local` API から `index.json` を自動更新 |

## 自動化

LuomiNest 本体の `backend/app/infrastructure/sync/registry_sync.py` が提供：

- `publish_local_plugins_to_registry()` — ローカルプラグイン/スキルメタデータをスキャン、リモート `index.json` にマージして GitHub Contents API で PUT プッシュ（GitHub Token 必要）
- `build_registry_index()` — ローカル index.json スナップショット生成
- `write_local_index_snapshot()` — `backend/app/data/cxp-registry-index.json` に書き込み

呼び出し方法：

```bash
# 非同期プッシュ（GITHUB_TOKEN 設定必要）
curl -X POST http://localhost:18000/api/v1/marketplace/registry/publish-local

# またはローカルスナップショットを手動コミット用に生成
curl -X POST http://localhost:18000/api/v1/marketplace/registry/build-snapshot
```

## License

MIT — [LICENSE](./LICENSE) を参照
