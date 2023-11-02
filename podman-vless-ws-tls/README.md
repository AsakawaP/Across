# Podman + Caddy + X-UI

使用 `Podman` 快速部署 `Caddy` + `X-UI` 并配置 `VLESS` + `WS` + `TLS`

## 前置条件

1. 一枚已经托管在 `CloudFlare` 的域名
2. 本方法适用于 `Debian 12` / `Ubuntu 22.04`
3. 安装 `Podman` + `Podman Compose`
4. `root` 用户无需 `sudo`

## 快速部署

### 拉取文件

```sh
git clone --depth 1 --no-checkout https://github.com/AsakawaP/Across
cd Across/podman-vless-ws-tls
```

### 修改 Caddyfile

根据[配置说明](../Caddy-XUI-Example.md)修改 `Caddyfile` 设置域名、TLS 证书、反向代理等。

### 启动容器

```sh
sudo podman-compose up -d

# 启动用于伪装站点的容器，示例容器名为 `masker`
sudo podman run -dt --restart always --name masker -p 8080:8080 docker.io/modem7/docker-rickroll
```

如想以 rootless 模式运行容器，参考后文的 systemd 配置方法。

## X-UI + VLESS + WS + TLS

进入 X-UI 管理面板，根据[配置说明](../Caddy-XUI-Example.md)添加一项 `VLESS` + `WS` 入站规则，监听端口与 Caddy 反向代理一致。

## 安装 Podman

```sh
# Debian 12
sudo apt install podman python3-pip
sudo pip3 install podman-compose --break-system-packages

# Ubuntu 22.04
sudo apt install podman python3-pip
sudo pip3 install podman-compose
```

## 配置 Systemd 管理 Podman Rootless 容器

Podman 安装时自动配置 `podman-restart.service`，可以在宿主机重启后自动启动所有重启策略为 `always` 的 Rootful 容器。
如不使用 `sudo` 运行的 Rootless 容器仍可正常映射宿主机端口，但在用户登出时会停止，需要 `Systemd` 管理容器的运行和重启。

```sh
# 生成伪装网站容器的 systemd 单元文件
podman generate systemd --files --container-prefix xui --name masker 

# 为当前用户安装并启用容器的 systemd 单元
cp xui-masker.service $HOME/.config/systemd/user
systemctl --user daemon-reload

systemctl --user enable xui-masker.service
systemctl --user is-enabled xui-masker.service

systemctl --user start xui-masker.service
systemctl --user status xui-masker.service
```

## Why Podman

- `apt install podman` 直装
- `Rootless`
- 与 `ufw` 没有冲突，便于管理
- 在没有技术负债的情况下，并没有比 `Docker` 更难上手
