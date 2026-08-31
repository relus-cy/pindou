# pindou 初始搭建设计

日期:2026-08-31
状态:已批准(用户确认 + 追加访问验证需求)

## 1. 背景与目标

搭建一个拼豆(perler beads)图纸工具:

- **核心功能**:图片生成拼豆图纸 + 手动编辑,两者都要
- **形态**:Web 应用,优先支持 iPad / iPhone Safari
- **颜色体系**:内置品牌色号(MARD / COCO / 漫漫 / 盼盼 / 咪小窝),数据驱动,品牌可后扩
- **数据**:纯前端,图纸存浏览器 localStorage,导出 PNG / CSV,无账号体系
- **部署**:claw(腾讯云 ECS 新加坡 `43.134.43.207`),公网域名访问,带基础访问验证

## 2. 选型结论

基底:**github.com/liangdabiao/perler-beads-ai @ main**(Apache 2.0)。

三个候选的评估:

| 候选 | 结论 | 关键理由 |
|------|------|----------|
| Zippland/perler-beads | 不选(被超集) | 算法核心,但协议 AGPL-3.0,部署需 Node 运行时 |
| liangdabiao/perler-beads-ai | **选用** | Zippland 功能超集(裁剪/橡皮擦/颜色替换/自定义调色板/CSV 导出),静态导出零运行时,Apache 2.0 |
| liangdabiao/perlerBeadsApplet | 不选 | 微信小程序优先,H5 是次要产物,iOS Safari 兼容性验证成本高 |

协议链条说明:Zippland 上游现为 AGPL-3.0,但其仓库内 CLAUDE.md 遗留 "Apache 2.0" 字样,推测 fork 发生于 Apache 2.0 时期,perler-beads-ai 的 Apache 2.0 协议成立;自用部署两种协议下均无影响。

## 3. AI 优化功能:第一版不接

- 该功能本质是图生图(照片 → 像素风重绘),需要图像生成模型;DeepSeek / Kimi 均为纯文本模型,无法接入。
- 基础算法(主导色映射 + BFS 区域合并 + 背景剥离)对动漫图 / 像素画 / Logo 已够用;AI 只对真实照片类输入提升明显。
- 后续如需接入:图像生成 API(即梦 / 豆包 / gpt-image 等)+ 一个轻量后端接口,改动隔离,不阻塞本期。
- 本期处理:隐藏前端 AI 优化入口,避免用户点到不可用的功能。

## 4. 架构

- **纯静态站点**:Next.js 静态导出 `out/`,全部图像处理在浏览器端 Canvas 完成。无后端、无数据库,claw 上零运行时维护。
- **仓库**:
  - `origin`:github.com/relus-cy/pindou(公开 fork,已完成)
  - `upstream`:github.com/liangdabiao/perler-beads-ai(跟踪上游更新,已配置)
- **项目脚手架**:按 `~/workspace/infra/templates/` 规范取最小集——新增 `AGENTS.md`、`.mise.toml`(只留 node)、`docs/status.md`、`docs/setup.md`;上游已有的 `CLAUDE.md` 不覆盖,仅顶部加一行 `@AGENTS.md` 指针;`architecture.md` / `decisions.md` / `CONTEXT.md` 等长出真实内容再建(遵循 infra 升降级规则)。
- **部署链路**:本地 `npm run build` → `rsync out/` → claw `/var/www/pindou/` → Caddy 静态服务。

## 5. 部署设计(claw)

- **站点目录**:`/var/www/pindou/`;首次需 `sudo mkdir -p` 并 `chown ubuntu:ubuntu`,之后 rsync 直传无需 sudo。
- **Caddy**:新增站点块 `pindou.philobscur.com.cn`:
  - `basic_auth`:用户 `freya`;密码哈希用 `caddy hash-password` 生成(bcrypt),只写入 claw 的 `/etc/caddy/Caddyfile`,**不进任何 git 仓库、不落文档**。
  - `root * /var/www/pindou` + `file_server`;TLS 由 Caddy 自动签发续期(443/tcp 已在安全组开放)。
  - 缓存策略:`/_next/static/*` 为内容哈希文件名,加 `Cache-Control: public, max-age=31536000, immutable` 长缓存,避免 basic auth 的 bcrypt 校验拖慢回访。
  - 与既有 Sub-Store 站点按域名分流,互不影响。
- **容量基线**(2026-08-31 实测):load 0.24/0.07/0.02,内存可用 2.5GB,磁盘余 19GB,eth0 日均收 215MB / 发 82MB;pindou 月增量 <1GB,性能与流量均无瓶颈。唯一预期管理:首屏约 3MB 静态资源,按出口 ≥3Mbps 估 5-10 秒,之后走缓存 + PWA。
- **DNS 外部依赖**:`philobscur.com.cn` 需新增 A 记录 `pindou` → `43.134.43.207`,由用户在域名控制台操作。
- **安全组**:维持现状(仅 443 / 22),无需变更。

## 6. 安全与边界

- 纯静态页面无服务端攻击面;唯一的动态层是 Caddy 的 basic auth,走 HTTPS。
- 密码明文只由用户持有;仓库和文档中只记录"认证已配置"这一事实。
- iOS Safari 首次访问会弹认证框;"添加到主屏幕"的 PWA 内首次打开会再弹一次,之后会话内保持。
- 原作者的打赏二维码与品牌信息默认保留,尊重上游。

## 7. 验证清单

本地:

1. `npm install && npm run build` 成功产出 `out/`。
2. 本地 HTTP 服务打开,走通:上传图片 → 生成图纸 → 手动改色 → 导出 PNG / 采购清单 CSV。
3. AI 优化入口已不可见。

claw:

4. `curl -I https://pindou.philobscur.com.cn` 未认证返回 401;带 `-u freya:***` 返回 200。

真机(需用户本人操作):

5. iPad / iPhone Safari 打开站点,通过认证,完整跑一遍生成流程。
6. "添加到主屏幕"后从图标打开,PWA 体验正常。

## 8. 后续方向(不在本期)

- AI 优化接入图像生成模型(需轻量后端或 Cloudflare Pages Function)。
- 编辑器增强:撤销 / 重做、作品管理(参考 perlerBeadsApplet)。
- 品牌色号扩充(Artkal / Perler / Hama)。
