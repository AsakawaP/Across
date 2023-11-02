# Docker + Caddy + X-UI

使用 `Docker` 快速部署 `Caddy` + `X-UI` 并配置 `VLESS` + `WS` + `TLS`

## 前置条件

1. 一枚已经托管在 `CloudFlare` 的域名
2. 安装 `Docker` + `Docker Compose`
3. `root` 用户无需 `sudo`

## 快速部署

### 拉取文件

```sh
git clone --depth 1 --no-checkout https://github.com/AsakawaP/Across
cd Across/docker-vless-ws-tls
```

### 修改 Caddyfile

根据[配置说明](../Caddy-XUI-Example.md)修改 `Caddyfile` 设置域名、TLS 证书、反向代理等。

### 启动容器

```sh
sudo docker compose up -d
```

## X-UI + VLESS + WS + TLS

进入 X-UI 管理面板，根据[配置说明](../Caddy-XUI-Example.md)添加一项 `VLESS` + `WS` 入站规则，监听端口与 Caddy 反向代理一致。

## 安装 Docker

略
