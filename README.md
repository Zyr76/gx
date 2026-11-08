# gx

囧次元客户端（`Zyr76/jcy`）的远程解析配方仓库。App 启动时拉取 `resolve.json`，缓存到本地；拉取失败时继续使用上一次缓存或 APK 内置默认值。

不需要后端服务器，GitHub raw / jsDelivr 这类静态地址即可。

## App 拉取地址

按顺序尝试：

1. `https://raw.githubusercontent.com/Zyr76/gx/main/resolve.json`
2. `https://cdn.jsdelivr.net/gh/Zyr76/gx@main/resolve.json`

## 更新方式

1. 修改 `resolve.json` 的字段和 `updatedAt`。
2. 提交到 `main`。
3. 重启 App 生效；首次启动会先用本地缓存，再后台拉取新配置。

改路径、adToken 槽位、清晰度策略、headers 解析开关、网关切换开关这类参数时，不需要重新发 APK。

## 字段

### device

| 字段 | 说明 |
| --- | --- |
| `hotVersion` | 设备指纹里的 `hv`。默认 `1.0.11`；服务端明确要求变化时再改，改前先确认主页接口没有被拒。 |

### resolve

| 字段 | 说明 |
| --- | --- |
| `qualitiesPath` | 清晰度列表 + exchangeToken 接口 |
| `exchangePath` | 换取播放地址接口 |
| `signPath` | sign 接口 |
| `adConfigPath` | 广告配置 / nonce 接口 |
| `adClaimPath` | 领取 adToken 接口 |
| `adSlot` | 广告槽位，默认 `player_pre_roll` |
| `getApiUrlAction` | 第一次 sign 的 action |
| `decryptResponseAction` | 上游加密响应解密的 action |
| `preferFreeQuality` | 非会员优先选免费清晰度 |
| `useAdToken` | 是否先领取 adToken |
| `tokenRefreshFallback` | qualities 无 exchangeToken 时强制续期 token |
| `anonymousFallback` | 续期失败后按未登录再请求一次 |
| `gatewayRotate` | 解析失败或主页空数据时切换业务网关 |
| `parseHeaders` | 把上游 `playAddr[].headers` 传给播放器 |
| `retryDelayMs` | qualities 无 token 时的重试间隔，范围 0–10000 |

## 兼容规则

- `schema` 目前为 `1`；字段不兼容地改名时递增，并在客户端加迁移。
- 所有路径必须以 `/api/` 开头，否则客户端会回退到内置默认值。
- 未知字段会被忽略，可以提前加注释性字段；但不要依赖客户端未实现的字段。
- 远程 JSON 目前只承载参数和开关，不承载可执行脚本；流程级变更仍需要客户端支持或后续 Lua 引擎。

## 安全

仓库和 CDN 必须使用 HTTPS。远程配方会影响解析路径和广告槽位，但不会直接执行代码；后续如果加入 Lua 脚本，需要加签名校验后再启用。
