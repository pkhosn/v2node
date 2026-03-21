# v2node
A v2board backend base on moddified xray-core.
一个基于修改版xray内核的V2board节点服务端。

**注意：本项目需要搭配支持 v2node 接口的 v2board 版本。**

## 对接说明（v2node 节点）

v2node 对接分为两段接口：

- 节点配置拉取：`/api/v2/server/config`
- 用户/流量同步：`/api/v1/server/UniProxy/*`

因此，面板必须同时具备 v2 配置接口和 UniProxy 接口。

## 软件安装

### 一键安装

```
wget -N https://raw.githubusercontent.com/pkhosn/v2node/master/script/install.sh && bash install.sh
```

### 一键安装（自动写入节点配置）

```bash
wget -N https://raw.githubusercontent.com/pkhosn/v2node/master/script/install.sh && \
bash install.sh --api-host "https://your-panel-domain" --node-id "1" --api-key "YOUR_SERVER_TOKEN"
```

安装后会自动生成 `/etc/v2node/config.json` 并重启服务。

## 面板节点配置模板

在面板新增节点时，选择 `v2node` 类型，核心字段建议如下：

- `protocol`: `vmess` / `vless` / `trojan` / `shadowsocks` / `tuic` / `hysteria2` / `anytls`
- `server_port`: 节点监听端口
- `host` / `server_name`: 该协议需要的域名
- `tls`: 按需开启（0/1）
- `network`: `tcp` / `ws` / `grpc` 等

`tls_settings` 建议使用 JSON，示例：

```json
{
  "server_name": "node.example.com",
  "cert_mode": "self",
  "cert_file": "/etc/v2node/node.example.com.crt",
  "key_file": "/etc/v2node/node.example.com.key",
  "provider": "cloudflare",
  "dns_env": "CF_DNS_API_TOKEN=YOUR_TOKEN",
  "reject_unknown_sni": "0"
}
```

证书模式说明：

- `self`: 使用本地证书；若证书文件不存在则自动自签
- `http`: 使用 HTTP-01 自动申请（需 80 端口可达）
- `dns`: 使用 DNS-01 自动申请（需 `provider` + `dns_env`）
- `none` 或空：不处理证书

## 节点配置文件模板（/etc/v2node/config.json）

```json
{
  "Log": {
    "Level": "info",
    "Output": "",
    "Access": "none"
  },
  "Nodes": [
    {
      "ApiHost": "https://your-panel-domain",
      "NodeID": 1,
      "ApiKey": "YOUR_SERVER_TOKEN",
      "Timeout": 15
    }
  ]
}
```

## 验证对接是否成功

```bash
systemctl status v2node --no-pager
journalctl -u v2node -n 100 --no-pager
```

日志中出现以下内容即表示基本对接成功：

- `Got nodes info from server`
- `Added ... new users`
- `Nodes started`

也可以直接探测面板接口：

```bash
curl "https://your-panel-domain/api/v2/server/config?node_type=v2node&node_id=1&token=YOUR_SERVER_TOKEN"
curl "https://your-panel-domain/api/v1/server/UniProxy/user?node_type=v2node&node_id=1&token=YOUR_SERVER_TOKEN"
```

返回 `200` 且内容正常即为链路可用。

## 构建
``` bash
GOEXPERIMENT=jsonv2 go build -v -o build_assets/v2node -trimpath -ldflags "-X 'github.com/pkhosn/v2node/cmd.version=$version' -s -w -buildid="
```

## Stars 增长记录

[![Stargazers over time](https://starchart.cc/wyx2685/v2node.svg?variant=adaptive)](https://starchart.cc/wyx2685/v2node)
