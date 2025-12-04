# Tailscale Sidecar

通用的 Tailscale Sidecar 配置模板，让任意 Docker 服务通过 Tailnet 安全访问。

## 文件说明

| 文件 | 说明 |
|------|------|
| `docker-compose.yml` | 主服务定义（示例占位，请替换为你的实际服务） |
| `docker-compose.ts.override.yml` | Tailscale sidecar 配置，仅 Tailnet 访问 |
| `docker-compose.nginx.override.yml` | 扩展配置，同时支持 Nginx 公网反代 |
| `serve.json.template` | Tailscale serve 配置模板，支持环境变量 |
| `.env.example` | 环境变量示例 |

## 食用方式

1. 修改 `docker-compose.ts.override.yml` 里的主服务名称：
   ```yaml
   service:
     app: # 改成你的主服务名称
   ```
> 自行按需同样修改 `docker-compose.nginx.override.yml`

2. 复制`docker-compose.ts.override.yml` 、 `serve.json.template` 和示例环境变量内容到你的项目下
> 自行按需复制 `docker-compose.nginx.override.yml`

3. 编辑你的项目的 `.env` ，填入配置：
   ```env
   TS_AUTHKEY=tskey-auth-xxxxx    # Tailscale Auth Key
   TS_HOSTNAME=my-app             # 设备名
   TS_TAILNET=your-tailnet.ts.net # Tailnet 域名
   PORT=80                        # 主服务端口
   # 若你的项目有override，请自行添加，若无则不需要
   COMPOSE_FILE=docker-compose.yml:docker-compose.ts.override.
   yml:docker-compose.override.yml
   ```

4. 启动服务：
   ```bash
   docker compose up -d
   ```

5. 通过 Tailnet 访问：
   ```
   https://<TS_HOSTNAME>.<TS_TAILNET>
   ```

## 更多的自定义方式

通过环境变量自定义访问方式，详细说明请参考[环境变量示例文件](.env.example)

### 仅 Tailnet 访问（默认）

```env
docker-compose.yml:docker-compose.ts.override.yml:docker-compose.override.yml
```

### Tailnet + Nginx 公网反代

```env
COMPOSE_FILE=docker-compose.yml:docker-compose.ts.override.yml:docker-compose.nginx.override.yml:docker-compose.override.yml
```

需要预先创建外部 Docker 网络 `nginx-proxy-network`。

## 环境变量

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `TS_AUTHKEY` | Tailscale 认证密钥 | 无，必填 |
| `TS_HOSTNAME` | Tailnet 中的设备名 | `ts-sidecar` |
| `TS_TAILNET` | Tailnet 域名 | 无，必填 |
| `PORT` | 主服务监听端口 | `80` |

## 注意事项

- 记得更改主服务名称
- TS_TAILNET变量是你的完整前三级域名，即 `your-tailnet.ts.net`
- 建议使用可复用（Reusable）+ 带标签（Tagged）的 Auth Key
- sidecar 会自动取消主服务的端口映射，如需直接访问请自行修改
- Tailscale 状态数据保存在 `./tailscale-data/` 目录

## 许可证

MIT License © YewFence
