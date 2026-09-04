# Utopia · LazyCat

[Utopia](https://github.com/deeplethe/utopia) 的懒猫微服 LPK 包装项目。Utopia 是全球首个开源企业世界模型，由一个 Rust 服务与带 pgvector 的 PostgreSQL 组成。

## 运行结构

- `app`：`ghcr.io/deeplethe/utopia:0.1.0-rc4`，HTTP 端口 `1516`。
- `db`：`pgvector/pgvector:pg16`，仅在应用内网开放 `5432`。
- PostgreSQL 数据保存在 `/lzcapp/var/postgres`。
- 原始文件、凭据密钥和全文索引保存在 `/lzcapp/var/data`。

数据库 owner 和运行时受限角色使用两个独立的 `stable_secret`。首次初始化脚本通过 LPK `contentdir` 复制到 PostgreSQL 容器，不使用 LazyCat 不支持的单文件 bind。Utopia 自动生成 JWT 密钥与凭据封印密钥，并分别保存在数据库和持久化数据目录。

## 健康检查

- `db` 保留上游 Compose 的 `pg_isready` 检查。
- 应用通过 Utopia 原生端点 `/api/v1/health` 检查，不依赖容器内的 curl/wget。

## 登录与文件选择器

按项目要求，保留 Utopia 原生的 `/login` 注册/登录流程，不配置免密登录。本应用只发布到喵喵商店，因此按项目要求不集成懒猫文件选择器拦截。

## 自动更新与发布

`.github/lazycat-action.yml` 管理两个明确的镜像目标：

- `app` 更新 `services.app.image`，并作为应用版本源。标签规则同时接受 RC 和正式版，并通过 `{version}` 显式保留完整 RC 后缀，如 `0.1.0-rc4 → 0.1.0 → 0.1.1-rc1 → 0.1.1`。
- `db` 更新 `services.db.image`，跟踪 PG16 的 `pg16` 标签，并将该非 SemVer 标签映射为 `16.0.0` 供更新检查排序；它不作为应用版本源。

工作流每日检查，也可手动运行。两个镜像均使用 `mirror` 交付：GHCR 通过 `ghcr.1ms.run`，Docker Hub 通过 `docker.1ms.run`。GitHub Action 要求镜像加速器与上游的 Linux amd64 digest 一致，只做读取校验，不复制到 LazyCat Registry。应用只发布到喵喵商店；懒猫官方商店始终关闭。版本数据库迁移只前滚，升级前请备份 PostgreSQL 与 `/lzcapp/var/data`。

所需 GitHub Actions Secrets：

- `APPSTORE_URL`
- `APPSTORE_TOKEN`
- `APP_ID`（首次发布成功后固定新应用的数字 ID）
