# Status

## 当前阶段

搭建中:fork 已完成,待隐藏 AI 入口并部署到 claw。

## 运行配置

| 项目 | 值 |
|------|-----|
| 工具链 | `.mise.toml`(node 24) |
| 目标地址 | https://pindou.philobscur.com.cn(basic auth: freya) |
| 部署目标 | claw `/var/www/pindou/`,Caddy `file_server` |
| 上游 | github.com/liangdabiao/perler-beads-ai @ main |

## 最近变更

- 2026-08-31 — fork + 初始 spec/plan

## 已知问题

- 本机代理会掐断 GitHub SSH(22),git 操作走 HTTPS

## 下一步

### P0

1. 隐藏 AI 入口并部署上线
2. 真机验收(iPad/iPhone Safari + 添加到主屏幕)

### P1

1. AI 优化接入图像生成模型(即梦 / gpt-image),需轻量后端
