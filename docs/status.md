# Status

## 当前阶段

v0 已上线(2026-08-31):fork 自 perler-beads-ai,已隐藏 AI 入口,静态部署在 claw,公网 basic auth 访问。

## 运行配置

| 项目 | 值 |
|------|-----|
| 工具链 | `.mise.toml`(node 24) |
| 目标地址 | https://pd.meowmeowmoon.com(basic auth: meow) |
| 部署目标 | claw `/var/www/pindou/`,Caddy `file_server` |
| 上游 | github.com/liangdabiao/perler-beads-ai @ main |

## 最近变更

- 2026-08-31 — fork + 初始 spec/plan
- 2026-08-31 — claw 上线:pd.meowmeowmoon.com(basic auth + immutable 静态缓存)
- 2026-09-01 — 品牌调整为 meowmeow拼豆:删打赏按钮、头部简化、水印更名,已重新部署

## 已知问题

- 本机代理会掐断 GitHub SSH(22),git 操作走 HTTPS

## 下一步

### P0

1. 隐藏 AI 入口并部署上线
2. 真机验收(iPad/iPhone Safari + 添加到主屏幕)

### P1

1. AI 优化接入图像生成模型(即梦 / gpt-image),需轻量后端
