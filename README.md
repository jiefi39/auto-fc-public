# auto-fc-public

定时重建面板实例的**运行仓库**。

代码不在这里 —— 在私有仓库 `jyucoeng/fc-private`，运行时检出到 `fc-private/` 子目录再跑。
本仓库只放两样东西：

```
.github/workflows/fc-renew.yml   定时任务
fc/1/time.txt                    保活提交（防 60 天无活动被停用定时任务）
```

## 为什么拆成公开 + 私有

- 公开仓库的 Actions 分钟数不计费，定时任务也不会因为「仓库 60 天没活动」被自动停用；
- 私有仓库只存代码和 Secrets，不暴露运行记录、日志和账号信息。

## 需要的 Secrets

Settings → Secrets and variables → Actions：

| Secret | 必填 | 说明 |
| --- | --- | --- |
| `PRIVATE_SSH_KEY` | ✅ | `fc-private` 只读部署密钥的私钥（Settings → Deploy keys 里那把公钥对应的私钥） |
| `FC_ACCOUNTS` | ✅ 二选一 | `accounts.json` 压成一行的 JSON |
| `FC_USER` / `FC_PASS` | ✅ 二选一 | 单账号时用 |
| `FC_TG_TOKEN` | 可选 | Telegram Bot Token，不配就不发报告 |
| `FC_TG_CHAT` | 可选 | Telegram 会话 id，多个用逗号分隔 |
| `HY2_PROXY_URL` | 可选 | 登录用的节点 URL（`hy2://` `vless://` `trojan://` `ss://` `tuic://` `anytls://`） |

Variables（可选）：`FC_INTERVAL_DAYS`（默认 29）、`FC_DUE_DAYS`（默认 2）。

## 跑法

- **定时**：每天 UTC 19:23（北京时间次日 03:23）。脚本自己判断「距到期不足 2 天」才动手，
  所以不是每天都重建，只是每天都来看一眼。
- **手动**：Actions → 🔄 FC 实例重建 ① → Run workflow，可以选阶段（auto/open/maintain）、
  强制忽略到期检查、或只体检（dry-run）。

跑完会在 Telegram 收到一条报告：

```
🎮 FC 实例重建报告

🕐 运行时间: 2026-10-24 06:20:08
📊 重建结果: ✅ 成功
👤 账号: guo***500

🌐 IP 信息: 185.156.53.*** (UA) [✅ 代理]
```

## 只保活、不干活

不配 `FC_*` 凭据就行：「校验凭据」那步会打印
`没配凭据 → 本次只保活，不跑维护`，跳过安装代理和运行两个步骤，
但**照样提交 `fc/1/time.txt`**，整轮是成功的，不会每天红一次。

## 注意

- 本仓库是**唯一**跑定时任务的地方，私有仓库 `fc-private` 里不放任何 workflow。
- `fc-private` 的代码更新后不用动本仓库 —— 每次运行都是重新检出私有仓库的 `main`。
- 要加第 ② 个账号组：复制 workflow 改文件名，然后改 `name` / `cron` /
  `concurrency.group` / `TIME_DIR` 四处（`TIME_DIR` 不改会并发提交互相覆盖）。

详细文档在私有仓库的 `README.md`。
