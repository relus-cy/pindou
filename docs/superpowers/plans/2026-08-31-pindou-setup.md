# pindou 初始搭建 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 把 fork 的拼豆图纸工具(隐藏 AI 入口)以纯静态形式部署到 claw,通过 `https://pindou.meowmeowmoon.com`(basic auth)对外提供服务。

**Architecture:** Next.js 15 静态导出(`output: "export"` 已在 next.config.ts 确认)→ 本地构建 `out/` → rsync 到 claw `/var/www/pindou/` → Caddy `file_server` + `basic_auth` + 自动 TLS。无后端、无数据库,图像处理全部在浏览器端。

**Tech Stack:** Next.js 15.3.6 / React 19 / Tailwind 4;Node 24(mise 管理);Caddy(已运行于 claw);SSH alias `claw`。

**Spec:** `docs/superpowers/specs/2026-08-31-pindou-setup-design.md`

## Global Constraints

- 仓库是**公开** fork:密码、bcrypt 哈希、任何密钥永不进 git;plan 和文档也不含明文。唯一的秘密材料是 freya 的 basic auth 密码(用户在对话中提供)及其哈希(只存在于 claw 的 `/etc/caddy/Caddyfile`)。
- 对上游代码零改动,仅两处例外:`src/app/page.tsx` 隐藏 AI 入口(Task 2)、`CLAUDE.md` 顶部加一行指针(Task 1)。
- 上游已弃用的 `AIOptimizeModal.tsx` / `src/utils/aiOptimize.ts` **保留不删**(减少 upstream 合并分歧;静态导出会被 tree-shake)。
- Node 24 由 mise 管理;安装依赖一律 `npm ci`(尊重 lockfile)。
- GitHub 走 HTTPS(本机代理会掐断 SSH 22 端口,已验证)。
- claw = SSH alias `claw`(ubuntu@43.134.43.207);Caddy 已有 `waga.philobscur.com.cn` 站点,新站点块风格对齐(tab 缩进、`encode zstd gzip`)。
- 上游无测试套件(package.json 无 test script),验证方式为构建成功 + grep 断言 + 手动冒烟清单。

---

### Task 1: 项目脚手架最小集

**Files:**
- Create: `AGENTS.md`
- Create: `.mise.toml`
- Create: `docs/status.md`
- Create: `docs/setup.md`
- Modify: `CLAUDE.md`(顶部加一行指针,不覆盖上游内容)

**Interfaces:**
- Consumes: 无
- Produces: `docs/setup.md` 中的部署 runbook(Task 4 照它执行);`.mise.toml` 的 node 24(Task 2 构建依赖)

- [ ] **Step 1: 创建 `.mise.toml`**

```toml
[tools]
node = "24"
```

- [ ] **Step 2: 创建 `AGENTS.md`**

```markdown
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
```

- [ ] **Step 3: 创建 `docs/status.md`**(当前阶段写"搭建中",上线后在 Task 4 更新)

```markdown
# Status

## 当前阶段

搭建中:fork 已完成,待隐藏 AI 入口并部署到 claw。

## 运行配置

| 项目 | 值 |
|------|-----|
| 工具链 | `.mise.toml`(node 24) |
| 目标地址 | https://pindou.meowmeowmoon.com(basic auth: freya) |
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
```

- [ ] **Step 4: 创建 `docs/setup.md`**(完整部署 runbook,Task 4 按此执行)

````markdown
# Setup / Deploy

## 本地开发

```bash
mise install
npm ci
npm run dev        # http://localhost:3000
```

## 构建

```bash
npm run build      # 静态导出到 out/
```

## 部署到 claw

```bash
# 首次:创建站点目录(之后不用)
ssh claw 'sudo mkdir -p /var/www/pindou && sudo chown ubuntu:ubuntu /var/www/pindou'

# 日常发布
npm run build
rsync -az --delete out/ claw:/var/www/pindou/
```

## Caddy(claw 上 `/etc/caddy/Caddyfile`)

站点块(追加在 Caddyfile 末尾;`freya` 后的哈希为 `caddy hash-password` 的运行时输出,只存在于该文件):

```
pindou.meowmeowmoon.com {
	encode zstd gzip

	header {
		Referrer-Policy "no-referrer"
		X-Content-Type-Options "nosniff"
	}

	basic_auth {
		freya <bcrypt 哈希,见上行说明>
	}

	root * /var/www/pindou

	@immutable path /_next/static/*
	header @immutable Cache-Control "public, max-age=31536000, immutable"

	file_server
}
```

改配置后:

```bash
sudo caddy validate --config /etc/caddy/Caddyfile
sudo systemctl reload caddy
```

## 认证

basic auth 用户 `freya`;哈希用 `caddy hash-password` 在 claw 上生成,只写入 Caddyfile,不进仓库、不落文档。
````

- [ ] **Step 5: `CLAUDE.md` 顶部加指针行**

文件当前第一行是 `# 拼豆底稿生成器 (Perler Beads Generator)`,在其前面插入两行:

```
@AGENTS.md

```

即文件开头变为:

```
@AGENTS.md

# 拼豆底稿生成器 (Perler Beads Generator)
```

- [ ] **Step 6: 验证工具链**

Run: `mise install && node --version && npm --version`
Expected: 输出版本号(node v24.x);若 `mise: command not found`,先 `cd ~/workspace/infra && just bootstrap`

- [ ] **Step 7: Commit**

```bash
git add AGENTS.md .mise.toml docs/status.md docs/setup.md CLAUDE.md
git commit -m "chore: infra 最小脚手架(AGENTS/mise/status/setup + CLAUDE 指针)"
git push origin main
```

---

### Task 2: 隐藏 AI 优化入口

**Files:**
- Modify: `src/app/page.tsx`(6 处,全部为删除或一句文案修改)

**Interfaces:**
- Consumes: Task 1 的 node 24 工具链
- Produces: 无 AI 入口的 `src/app/page.tsx`;`npm run build` 产出的 `out/` 中不含 AI 相关字样

- [ ] **Step 1: 安装依赖**

Run: `npm ci`
Expected: 成功,无 ERR

- [ ] **Step 2: 删除 import(第 100 行)**

删除整行:

```tsx
import AIOptimizeModal from '../components/AIOptimizeModal';
```

- [ ] **Step 3: 删除弹窗状态(第 197-198 行)**

```tsx
  // 新增：AI优化弹窗状态
  const [isAIOptimizeOpen, setIsAIOptimizeOpen] = useState<boolean>(false);
```

- [ ] **Step 4: 删除三个处理函数(第 668-697 行)**

```tsx
  // 处理AI优化打开
  const handleAIOptimizeOpen = () => {
    if (!originalImageSrc) {
      alert('请先上传图片');
      return;
    }
    setIsAIOptimizeOpen(true);
  };

  // 处理AI优化关闭
  const handleAIOptimizeClose = () => {
    setIsAIOptimizeOpen(false);
  };

  // 处理AI优化完成
  const handleAIOptimized = (optimizedImageSrc: string) => {
    // 使用优化后的图片替换原图，并重新处理
    setOriginalImageSrc(optimizedImageSrc);
    setMappedPixelData(null);
    setGridDimensions(null);
    setColorCounts(null);
    setTotalBeadCount(0);
    setInitialGridColorKeys(new Set());
    setRemapTrigger(prev => prev + 1);

    // 重置手动上色模式
    setIsManualColoringMode(false);
    setSelectedColor(null);
    setIsEraseMode(false);
  };
```

- [ ] **Step 5: 删除"AI优化"按钮(第 2174-2183 行)**

```tsx
                  <button
                    onClick={handleAIOptimizeOpen}
                    disabled={!originalImageSrc}
                    className="inline-flex items-center justify-center h-9 px-3 text-sm rounded-md border border-purple-200 dark:border-purple-700 bg-purple-50 dark:bg-purple-900/30 text-purple-700 dark:text-purple-200 hover:bg-purple-100 dark:hover:bg-purple-800/40 transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed whitespace-nowrap"
                  >
                    <svg className="w-4 h-4 mr-1.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M13 10V3L4 14h7v7l9-11h-7z" />
                    </svg>
                    AI优化
                  </button>
```

- [ ] **Step 6: 修改使用流程文案(第 2712 行)**

把:

```tsx
                      上传图片 → AI优化 → 去掉背景 → 下载图纸 → 专心拼豆
```

改为:

```tsx
                      上传图片 → 去掉背景 → 下载图纸 → 专心拼豆
```

- [ ] **Step 7: 删除弹窗渲染(第 2788-2794 行)**

```tsx
      {/* AI优化弹窗 */}
      <AIOptimizeModal
        imageSrc={originalImageSrc || ''}
        isOpen={isAIOptimizeOpen}
        onClose={handleAIOptimizeClose}
        onOptimized={handleAIOptimized}
      />
```

- [ ] **Step 8: 静态断言 + 构建**

Run: `grep -n "AIOptimize\|isAIOptimizeOpen\|handleAIOptim\|AI优化" src/app/page.tsx; echo "grep-exit=$?"`
Expected: 无任何匹配输出,`grep-exit=1`

Run: `npm run build`
Expected: 构建成功,产出 `out/`(`Export successful` / `✓`)

Run: `grep -rn "AI优化\|AIOptimize" out/ ; echo "grep-exit=$?"`
Expected: 无任何匹配,`grep-exit=1`(产物中不含 AI 入口字样)

- [ ] **Step 9: Commit**

```bash
git add src/app/page.tsx
git commit -m "feat: 隐藏 AI 优化入口(无后端可用,避免用户点到不可用功能)"
git push origin main
```

---

### Task 3: 本地构建与功能冒烟

**Files:**
- 无文件变更;纯验证任务,gate 通过才进 Task 4

**Interfaces:**
- Consumes: Task 2 产出的 `out/`
- Produces: 确认可部署的 `out/`

- [ ] **Step 1: 本地起静态服务**

Run: `python3 -m http.server 4173 -d out`
Expected: `Serving HTTP on :: port 4173`(保持运行,冒烟后 Ctrl-C)

- [ ] **Step 2: 手动冒烟清单(浏览器打开 `http://localhost:4173`)**

逐项确认:

- [ ] 首页正常打开,开发者工具 Console 无红色报错,Network 无 404
- [ ] 上传一张图片(JPG/PNG)→ 生成像素图纸
- [ ] 切换色号系统(如 MARD → 盼盼),图纸色号随之变化
- [ ] 手动改一个格子的颜色
- [ ] 导出带色号图纸 PNG、导出采购清单
- [ ] 页面上看不到"AI优化"按钮,使用流程文案中无"AI优化"

任何一项不通过:停下来排查,不要带问题上 claw。

---

### Task 4: claw 部署上线

**Files:**
- Modify(远端,不进 git): claw `/etc/caddy/Caddyfile` 追加站点块
- Create(远端,不进 git): claw `/var/www/pindou/`
- Modify: `docs/status.md`(阶段更新为已上线)

**Interfaces:**
- Consumes: Task 3 验证过的 `out/`;Task 1 的 `docs/setup.md` runbook
- Produces: 线上服务 `https://pindou.meowmeowmoon.com`

- [ ] **Step 1(GATE,用户操作): DNS A 记录**

用户在域名控制台为 `meowmeowmoon.com` 添加:`pindou` A 记录 → `43.134.43.207`。

验证:`dig +short pindou.meowmeowmoon.com`
Expected: `43.134.43.207`。**未生效前不继续**(Caddy 签证书需要域名解析已指向 claw)。

- [ ] **Step 2: 创建站点目录**

Run: `ssh claw 'sudo mkdir -p /var/www/pindou && sudo chown ubuntu:ubuntu /var/www/pindou'`
Expected: 无输出,exit 0

- [ ] **Step 3: 上传静态产物**

Run: `rsync -az --delete out/ claw:/var/www/pindou/ && ssh claw 'ls /var/www/pindou/index.html /var/www/pindou/manifest.json'`
Expected: rsync 成功;两个文件路径列出

- [ ] **Step 4: 生成 freya 的 bcrypt 哈希(不落盘)**

Run: `ssh claw "caddy hash-password --plaintext '<执行时由执行者从对话上下文取用户提供的密码>'" `
Expected: 输出一行 `$2a$14$...`。**此哈希只进入下一步的 Caddyfile,不写入任何仓库文件;命令随 ssh 非交互执行结束,不进远端 shell history。**

- [ ] **Step 5: 追加 Caddy 站点块**

把下面内容追加到 claw 的 `/etc/caddy/Caddyfile` 末尾(`<FREYA_BCRYPT_HASH>` 替换为 Step 4 的输出——这是密钥材料替换点,不是 placeholder):

```
pindou.meowmeowmoon.com {
	encode zstd gzip

	header {
		Referrer-Policy "no-referrer"
		X-Content-Type-Options "nosniff"
	}

	basic_auth {
		freya <FREYA_BCRYPT_HASH>
	}

	root * /var/www/pindou

	@immutable path /_next/static/*
	header @immutable Cache-Control "public, max-age=31536000, immutable"

	file_server
}
```

Run:

```bash
cat <<'CADDYEOF' | ssh claw 'sudo tee -a /etc/caddy/Caddyfile'
<上面的站点块,哈希已代入>
CADDYEOF
```

- [ ] **Step 6: 校验并重载 Caddy**

Run: `ssh claw 'sudo caddy validate --config /etc/caddy/Caddyfile && sudo systemctl reload caddy'`
Expected: `Valid configuration` / `Config adapted successfully`,reload exit 0

若证书签发未及时完成(TLS 报错),查日志:`ssh claw 'sudo journalctl -u caddy --since -5m | grep -i pindou'`,多数是 DNS 未生效,Caddy 会自动重试。

- [ ] **Step 7: 线上验证(本机 Mac 即国内网络视角)**

密码注入方式:执行环境非 TTY,curl 不会交互提示;用环境变量前缀注入,用完即焚:

```bash
PINDOU_PASS='<用户提供的密码>'; # 仅当前 shell 有效,不写入任何文件
```

Run: `curl -s -o /dev/null -w '%{http_code}\n' https://pindou.meowmeowmoon.com/`
Expected: `401`(未认证)

Run: `curl -u "freya:$PINDOU_PASS" -s -o /dev/null -w '%{http_code} %{time_total}s %{size_download}B\n' https://pindou.meowmeowmoon.com/`
Expected: `200`,记录首屏耗时(size 约几十 KB 的 HTML;首屏总耗时含 JS 需浏览器实测,此值为下限参考)

Run: `ls out/_next/static/ | head` 任取一个实际文件路径代入:`curl -u "freya:$PINDOU_PASS" -sI "https://pindou.meowmeowmoon.com/_next/static/<实际路径>" | grep -i cache-control`
Expected: `cache-control: public, max-age=31536000, immutable`

Run: `unset PINDOU_PASS`

- [ ] **Step 8: 更新 status 并 commit**

`docs/status.md` 的"当前阶段"改为:

```markdown
v0 已上线(2026-08-31):fork 自 perler-beads-ai,已隐藏 AI 入口,静态部署在 claw,公网 basic auth 访问。
```

"最近变更"追加一行:`- 2026-08-31 — claw 上线:pindou.meowmeowmoon.com(basic auth + immutable 静态缓存)`

```bash
git add docs/status.md
git commit -m "docs: status 更新为已上线"
git push origin main
```

---

### Task 5: 真机验收(用户操作)

**Files:**
- 无;验收 gate

**Interfaces:**
- Consumes: Task 4 的线上服务
- Produces: 用户确认的验收结果

- [ ] **Step 1: 用户真机清单**

在 iPad / iPhone Safari 上:

- [ ] 打开 `https://pindou.meowmeowmoon.com`,弹出认证框,输入 freya / 密码,页面正常加载
- [ ] 上传一张照片,走通:生成图纸 → 切换色号系统 → 手动改色 → 导出 PNG / 采购清单
- [ ] "添加到主屏幕",从主屏图标打开,PWA 内再认证一次后正常使用
- [ ] 记录首屏主观速度,反馈是否符合预期(参考:新加坡机房,首次约 5-10 秒,之后走缓存)

- [ ] **Step 2: 验收通过后,把结果记进 `docs/status.md` 的"最近变更"并 commit**

---

## Self-Review 记录

- **Spec 覆盖**:§3 AI 隐藏 → Task 2;§4 脚手架/静态导出 → Task 1/2;§5 部署(目录/Caddy/basic auth/长缓存/DNS/安全组)→ Task 4;§7 验证清单 → Task 2(grep)、Task 3(本地冒烟)、Task 4(curl 401/200/缓存头)、Task 5(真机)。§6 安全 → Global Constraints + Task 4 Step 4。§8 后续方向 → 不在本 plan,已留在 spec。
- **Placeholder 扫描**:唯一替换点是 Task 4 Step 5 的 `<FREYA_BCRYPT_HASH>`,为运行时生成的密钥材料(生成命令在 Step 4),属刻意设计。
- **一致性**:行号基于 2026-08-31 克隆的 upstream main(573006f);若上游变动导致行号漂移,以代码块内容为准搜索定位。
