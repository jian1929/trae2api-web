# 本地部署定制说明

本仓是 [connectedGraph/trae2api-web](https://github.com/connectedGraph/trae2api-web) 的 fork，
额外保留**本地部署定制**，避免下次拉上游时丢失。

## 相对上游的改动

仅 `docker-compose.yml` 一处（其余 41 个文件与上游逐字节一致，差异只是 CRLF/LF 行尾）：

| 项 | 上游 | 本地 | 原因 |
|---|---|---|---|
| 端口 | `7864:7864` | **`7865:7865`** | 7864 被 `wb2api-admin` 占用 |
| 镜像名 | 无（每次 build） | `trae2api-web:latest` | 固定镜像，便于复用 |
| 代理 | 无 | `NO_PROXY=192.168.0.0/16,127.0.0.1,localhost` | 绕开 VM260 注入的 `HTTP(S)_PROXY=7897` |
| 配置挂载 | 无 | `./config.json:/app/config.json:ro` | 外置配置 |
| **healthcheck** | 无 | **指向实际端口 7865** | 🔴 上游 Dockerfile 的 healthcheck 硬编码 `127.0.0.1:7864`，改端口后健康检查永远失败 → 容器每 ~90 秒自我重启。必须在 compose 里覆盖 |

## 部署位置

- 宿主机：`debian-docker-260`（192.168.10.60）
- 栈目录：`/srv/docker/stacks/trae2api-web/`
- 容器名：`trae2api-web`，宿主端口 `:7865`
- 域名：`https://trae2api.jian1929.store:1669`（Lucky 反代）

## 不入库的文件（.gitignore 已覆盖）

- `.env` —— 含 `TW2A_API_KEY`
- `config.json` —— 部署实参（listen `:7865`、checkin_hour 9 等）
- `auths/` —— 账号凭证（trae-*.json）
- `data/` —— 运行态 state.json

## 配置要点

```json
{
  "listen": ":7865",
  "default_model": "deepseek-v4.1-flash",
  "schedule": { "checkin_hour": 9, "refresh_hours": [3] }
}
```

签到：每天 09:00 自动执行，每账号 **+100 credits**（实测）。

## 上游同步

```bash
git remote add upstream https://github.com/connectedGraph/trae2api-web.git
git fetch upstream
git merge upstream/main     # 或 git rebase
```
