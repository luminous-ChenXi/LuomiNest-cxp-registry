# LuomiNest CxPlugin Registry

[English](./README.md) | [简体中文](./README.zh-CN.md) | [日本語](./README.ja-JP.md)

LuomiNest plugin/skill registry — aggregates metadata of all distributable plugins and skills via `index.json`.

## Repository Roles

| Repository | Purpose |
|------------|---------|
| **LuomiNest-cxp-registry** (this repo) | Central index, only maintains `index.json` |
| LuomiNest-cxp-skills | Skill collection (each skill is a subdirectory) |
| LuomiNest-cxp-plugin-template | Plugin development template (fork starting point) |
| LuomiNest-cxp-<plugin-name> | Individual plugin repositories (PDF reader, weather-query, etc.) |

**This repository is the central index — it does NOT contain any actual plugin or skill code.** It only maintains metadata pointing to each repository's Release assets.

## index.json Format

```json
{
  "version": "1.0.0",
  "updatedAt": "2026-07-20T00:00:00Z",
  "plugins": [
    {
      "id": "cxp-pdf-reader",
      "name": "CxPlugin PDF Reader",
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
      "name": "Travel Planner",
      ...
    }
  ]
}
```

See full field spec at [docs/development/plugin-system.md](https://github.com/luminous-ChenXi/LuomiNest/blob/main/docs/development/plugin-system.md).

## Available Entries

### Plugins

| ID | Name | Platform | Repository |
|----|------|----------|------------|
| `cxp-pdf-reader` | PDF Smart Reader | fullstack | LuomiNest-cxp-PDF_reader |
| `weather-query` | Weather Query | backend | LuomiNest-cxp-weather-query |

### Skills

| ID | Name | Repository |
|----|------|------------|
| `cooking-assistant` | Cooking Assistant | LuomiNest-cxp-skills |
| `skill-creator` | Skill Creator | LuomiNest-cxp-skills |
| `travel-planner` | Travel Planner | LuomiNest-cxp-skills |

## User Access Flow

```
LuomiNest startup -> Backend GET raw.githubusercontent.com/.../index.json
  -> Cache for 6 hours
  -> Merge with local static catalog -> Return to frontend "Marketplace"
  -> User clicks install -> Backend downloads zip from downloadUrl -> Extracts locally
```

## Submitting New Entries

### For Plugin Authors

1. Create a standalone repository using [LuomiNest-cxp-plugin-template](https://github.com/luminous-ChenXi/LuomiNest-cxp-plugin-template)
2. After development, create a GitHub Release (tag: `v<version>`, with zip asset)
3. Fork this repository -> Add a new entry to the `plugins` array in `index.json`
4. Submit a PR with title: `feat: register plugin <plugin-id> v<version>`

### For Skill Authors

1. Create a skill subdirectory in [LuomiNest-cxp-skills](https://github.com/luminous-ChenXi/LuomiNest-cxp-skills)
2. After PR merge, ask maintainers to create a Release with `skill-<id>.zip` asset
3. Fork this repository -> Add a new entry to the `skills` array in `index.json`
4. Submit a PR with title: `feat: register skill <skill-id> v<version>`

## GitHub Actions

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `validate-index.yml` | push/PR to main | Validate `index.json` structure (required fields, naming convention, semver, URLs, duplicate IDs) |
| `auto-sync.yml` | `repository_dispatch` or manual | Auto-update `index.json` from LuomiNest backend `publish-local` API |

## Automation

LuomiNest main repository's `backend/app/infrastructure/sync/registry_sync.py` provides:

- `publish_local_plugins_to_registry()` — scans local plugin/skill metadata, merges into remote `index.json`, PUT pushes via GitHub Contents API (requires GitHub Token)
- `build_registry_index()` — generates local index.json snapshot
- `write_local_index_snapshot()` — writes to `backend/app/data/cxp-registry-index.json`

Invocation:

```bash
# Async push to GitHub (requires GITHUB_TOKEN configured)
curl -X POST http://localhost:18000/api/v1/marketplace/registry/publish-local

# Or generate local snapshot for manual commit
curl -X POST http://localhost:18000/api/v1/marketplace/registry/build-snapshot
```

## License

MIT — see [LICENSE](./LICENSE)
