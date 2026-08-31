# pindou

> 拼豆图纸工具:图片生成图纸 + 手动编辑。fork 自 liangdabiao/perler-beads-ai,纯静态部署在 claw。

## Commands

- Setup: `mise install && npm ci`
- Dev: `npm run dev`
- Build: `npm run build`(静态导出到 `out/`)
- Deploy: 见 [docs/setup.md](docs/setup.md)

## Structure

- `src/` - Next.js 应用(上游代码)
- `docs/` - status、setup、superpowers spec/plan
- `out/` - 构建产物(已 gitignore)

## Rules

- 第一次接触本项目时读 [docs/status.md](docs/status.md)。
- 这是 fork:`origin` = relus-cy/pindou,`upstream` = liangdabiao/perler-beads-ai。改动尽量避开上游文件;`CLAUDE.md` 是上游文件,仅顶部有一行 `@AGENTS.md` 指针。
- 密码、哈希、密钥永不进仓库;basic auth 哈希只存在于 claw 的 `/etc/caddy/Caddyfile`。
- GitHub 走 HTTPS,不用 SSH(本机代理掐断 22 端口)。
- 调试前查 `~/workspace/infra/docs/known-issues.md`。
- 变更 3+ 文件前进入 plan mode。

## SSH

- `claw` = ubuntu@腾讯云 ECS 新加坡(43.134.43.207),alias 已在 `~/.ssh/config`。
- 3+ 命令走 ControlMaster(socket `~/.ssh/sockets/`)。

## Docs

- 状态与进度 -> [docs/status.md](docs/status.md)
- 部署操作 -> [docs/setup.md](docs/setup.md)
- 设计 spec -> [docs/superpowers/specs/2026-08-31-pindou-setup-design.md](docs/superpowers/specs/2026-08-31-pindou-setup-design.md)
