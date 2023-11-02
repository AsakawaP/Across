# Caddy + X-UI 示例

## 域名示例说明

* 假设 `example.com` 已托管在 CloudFlare
* CloudFlare `SSL/TLS` 设置为 `Flexible（灵活）`
* 配置如下 DNS 记录
  * `vless-ws.example.com`
    `VLESS` + `WS` + `TLS` 直连（仅 DNS），灰色云朵
  * `cdn.example.com`
    `VLESS` + `WS` + `TLS` + `CDN` 启用 CloudFlare CDN（已代理），黄色云朵
  * `xui.example.com`
    `X-UI` 管理面板，本示例中启用 CloudFlare CDN
* :memo: 部署前更改为自定义域名

## Caddy 示例说明

* Caddy 部署后自动配置 HTTPS 申请证书
* 不希望 Caddy 申请证书，可启用 `tls` 行配置为自定义证书，默认已注释
* :warning: 部署前反向代理示例转发路径 `/ChangeMe` 更改为自定义路径（例如：随机 UUID）
* 本示例中用于提供伪装网页的容器映射至宿主机 `8080` 端口，无须更改
* 可自定义反向代理，指向任意网页用于伪装

## X-UI 示例说明

* 假设在 X-UI 管理面板中添加 `VLESS` + `WS` 入站规则，监听宿主机端口 `55555`，路径 `/ChangeMe`
* Xray 客户端添加 VLESS 服务器，配置域名、端口 `443`、路径与 Caddy 反向代理转发路径一致
* X-UI 面板默认端口 `54321`
* :memo: 建议在部署后通过 X-UI 面板更改面板端口为自定义端口，或添加 `ufw` 防火墙保护规则

## Caddyfile 示例

```caddy
# # VLESS + WS + TLS
vless-ws.example.com {
  # tls /data/ssl/example.com_cert.pem /data/ssl/example.com_key.pem  # 自定义 TLS 证书
  
  encode gzip

  reverse_proxy /ChangeMe localhost:55555  # 将到达 /ChangeMe 路径的流量转发至 Xray 监听端口

  # 伪装网页
  reverse_proxy localhost:8080  # 反向代理本机 8080 端口的网页服务
  # reverse_proxy https://example-i-guess-no-one-would-read-this.com  # 反向代理任意网页
}

# # VLESS + WS + TLS + CDN
# # 方法一： CloudFlare SSL/TLS 设置为 Flexible（灵活）， Caddy 使用 HTTP 不配置 TLS
http://cdn.example.com {

# # 方法二： CloudFlare SSL/TLS 设置为 Full（完全）
# cdn.example.com {
  # tls /data/ssl/example.com_cert.pem /data/ssl/example.com_key.pem  # 同直连示例

  encode gzip

  reverse_proxy /ChangeMe localhost:55555

  reverse_proxy localhost:8080
}

# # X-UI
# # 通过域名访问 X-UI，原理同 CloudFlare CDN 示例
http://xui.example.com {
  encode gzip

  reverse_proxy localhost:54321  # 反向代理 X-UI 面板监听端口
}
```
