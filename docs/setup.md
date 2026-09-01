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

站点块(追加在 Caddyfile 末尾;`meow` 后的哈希为 `caddy hash-password` 的运行时输出,只存在于该文件):

```
pd.meowmeowmoon.com {
	encode zstd gzip

	header {
		Referrer-Policy "no-referrer"
		X-Content-Type-Options "nosniff"
	}

	basic_auth {
		meow <bcrypt 哈希,见上行说明>
	}

	root * /var/www/pindou

	@immutable path /_next/static/*
	header @immutable Cache-Control "public, max-age=31536000, immutable"

	try_files {path} {path}.html {path}/index.html

	file_server
}
```

改配置后:

```bash
sudo caddy validate --config /etc/caddy/Caddyfile
sudo systemctl reload caddy
```

## 认证

basic auth 用户 `meow`;哈希用 `caddy hash-password` 在 claw 上生成,只写入 Caddyfile,不进仓库、不落文档。
