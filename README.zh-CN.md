# LuomiNest CxPlugin 注册表

[English](./README.md) | [简体中文](./README.zh-CN.md) | [日本語](./README.ja-JP.md)

LuomiNest 插件/技能注册表 — 通过 `index.json` 聚合所有可分发的插件与技能元数据。

## 仓库职责

| 仓库 | 作用 |
|------|------|
| **LuomiNest-cxp-registry**（本仓库） | 中心索引注册表，仅维护 `index.json` |
| LuomiNest-cxp-skills | 技能集合（每个技能一个子目录） |
| LuomiNest-cxp-plugin-template | 插件开发模板（fork 起始点） |
| LuomiNest-cxp-<plugin-name> | 各插件独立仓库（PDF reader、weather-query 等） |

**本仓库是中心索引，不存放任何插件/技能的实际代码** — 仅维护指向各仓库 Release 资源的元数据。

## index.json 格式

```json
{
  "version": "1.0.0",
  "updatedAt": "2026-07-20T00:00:00Z",
  "plugins": [
    {
      "id": "cxp-pdf-reader",
      "name": "CxPlugin PDF 智能阅读器",
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
      "name": "旅行规划",
      ...
    }
  ]
}
```

详细字段说明见 [docs/development/plugin-system.md](https://github.com/luminous-ChenXi/LuomiNest/blob/main/docs/development/plugin-system.md)。

## 现有条目

### 插件

| ID | 名称 | 平台 | 仓库 |
|----|------|------|------|
| `cxp-pdf-reader` | PDF 智能阅读器 | fullstack | LuomiNest-cxp-PDF_reader |
| `weather-query` | 天气查询 | backend | LuomiNest-cxp-weather-query |

### 技能

| ID | 名称 | 仓库 |
|----|------|------|
| `cooking-assistant` | 烹饪助手 | LuomiNest-cxp-skills |
| `skill-creator` | 技能创建助手 | LuomiNest-cxp-skills |
| `travel-planner` | 旅行规划 | LuomiNest-cxp-skills |

## 用户访问流程

```
LuomiNest 启动 → 后端 GET raw.githubusercontent.com/.../index.json
  → 缓存 6 小时
  → 合并本地静态目录 → 返回给前端「插件市场」
  → 用户点击安装 → 后端从 downloadUrl 下载 zip → 解压到本地
```

## 提交新条目流程

### 插件作者

1. 使用 [LuomiNest-cxp-plugin-template](https://github.com/luminous-ChenXi/LuomiNest-cxp-plugin-template) 创建独立仓库
2. 开发完成 → 创建 GitHub Release（tag: `v<version>`，附 zip 资源）
3. Fork 本仓库 → 在 `index.json` 的 `plugins` 数组添加新条目
4. 提交 PR，标题格式：`feat: register plugin <plugin-id> v<version>`

### 技能作者

1. 在 [LuomiNest-cxp-skills](https://github.com/luminous-ChenXi/LuomiNest-cxp-skills) 创建技能子目录
2. PR 合并后请维护者创建 Release 并附 `skill-<id>.zip` 资源
3. Fork 本仓库 → 在 `index.json` 的 `skills` 数组添加新条目
4. 提交 PR，标题格式：`feat: register skill <skill-id> v<version>`

## GitHub Actions

| Workflow | 触发 | 功能 |
|----------|------|------|
| `validate-index.yml` | push/PR 到 main | 校验 `index.json` 结构（必填字段、命名规范、semver、URL、重复 ID） |
| `auto-sync.yml` | `repository_dispatch` 或手动 | 从 LuomiNest 后端 `publish-local` API 自动更新 `index.json` |

## 自动化

LuomiNest 主仓库的 `backend/app/infrastructure/sync/registry_sync.py` 提供：

- `publish_local_plugins_to_registry()` — 自动扫描本地插件/技能元数据，合并到远程 `index.json` 并 PUT 推送（需 GitHub Token）
- `build_registry_index()` — 生成本地 index.json 快照
- `write_local_index_snapshot()` — 写入 `backend/app/data/cxp-registry-index.json`

调用方式：

```bash
# 后台异步推送（需配置 GITHUB_TOKEN）
curl -X POST http://localhost:18000/api/v1/marketplace/registry/publish-local

# 或生成本地快照手动提交
curl -X POST http://localhost:18000/api/v1/marketplace/registry/build-snapshot
```

## License

MIT — 见 [LICENSE](./LICENSE)
