# 我的世界 · 网页版（Cloudflare Pages 部署包）

基于 **Eaglercraft**（MC.JS / mc.js.cool 的上游开源项目）的官方编译产物，纯静态，无需构建、无后端。

## 目录结构

```
deploy/
├─ index.html          版本选择启动页
├─ favicon.svg
├─ _headers            Cloudflare Pages 缓存与安全响应头
├─ wrangler.toml
├─ 1.8.8/              EaglercraftX 1.8 u53（JS 版，资源分离，可长缓存）
│  ├─ index.html
│  ├─ classes.js       21 MB
│  ├─ assets.epk       6.9 MB
│  ├─ favicon.png
│  └─ lang/            74 个语言包（含 zh_CN）
└─ 1.12.2/
   └─ index.html      EaglercraftX 1.12.2 u3（WASM-GC 单文件构建，23.9 MiB）
```

## 部署方式一：Dashboard 直传（最省事）

1. 打开 <https://dash.cloudflare.com/> → **Workers & Pages** → **Create** → **Pages** → **Upload assets**
2. 项目名填 `mc-web`（随意），把 **deploy 目录里面的所有内容**（不是 deploy 这个文件夹本身）拖进上传区
3. 点 Deploy，几十秒后拿到 `https://mc-web.pages.dev`

## 部署方式二：wrangler CLI

```bash
npm i -g wrangler
wrangler login
cd deploy
wrangler pages deploy . --project-name mc-web
```

## 部署方式三：接 GitHub 仓库

把 `deploy/` 里的内容作为仓库根目录推送，然后在 Pages 里连 Git：
- Framework preset: `None`
- Build command: 留空
- Build output directory: `/`

## 多线路（防单域名不可达）

同一个 Pages 项目绑定了三条线路，内容完全一致：

| 线路 | 域名 |
|---|---|
| 官方直连 | `mc-web-57e.pages.dev` |
| 线路 A | `minecraft.games.280627.xyz` |
| 线路 B | `mc.games.280627.xyz` |

首页底部有线路选择器，进入时会自动测一次延迟（请求各域名的 `favicon.svg`），
颜色规则：≤400ms 绿、≤1000ms 黄、>1000ms 红、无响应显示“超时”。
当前域能通就保持当前域，不通才自动切到最快的；手动点过的线路会被记住（`localStorage: mc.line`）。

**注意**：存档走浏览器 IndexedDB，按域名隔离。换线路 = 换存档，不会同步。所以默认策略是“尽量留在当前域”。

新增线路只要改 `index.html` 里的 `LINES` 数组：

```js
var LINES = [
  { host: "mc-web-57e.pages.dev", label: "官方直连", short: "官方" },
  { host: "minecraft.games.280627.xyz", label: "minecraft", suffix: ".games" },
  { host: "mc.games.280627.xyz", label: "mc", suffix: ".games" }
];
```

窄屏下 `suffix` 部分会自动隐藏（只显示 `label`）。

### 自定义域需要的 DNS

Cloudflare Pages 绑定自定义域后，若账号的 API 令牌没有 DNS 编辑权限，CNAME 不会被自动创建，
需要手动在 `280627.xyz` 的 DNS 里加两条（**代理状态必须开启 / 橙色云**）：

| 类型 | 名称 | 内容 |
|---|---|---|
| CNAME | `minecraft.games` | `mc-web-57e.pages.dev` |
| CNAME | `mc.games` | `mc-web-57e.pages.dev` |

加完后回到 Pages → 项目 → Custom domains，状态会从 `pending` 变为 `active`（证书签发约 1–5 分钟）。

## 注意事项

| 项目 | 说明 |
|---|---|
| 单文件上限 | Cloudflare Pages 限制 25 MiB/文件。`1.8.8/classes.js` = 21.1 MB ✓，`1.12.2/index.html` = 23.9 MiB ✓（已贴近上限，若后续换更大构建会传不上去） |
| 首次加载 | 1.8.8 约 26 MB、1.12.2 约 24 MB。启动页与 1.8.8 已做加载提示，第二次走浏览器缓存 |
| 浏览器要求 | 1.8.8 全浏览器可用；1.12.2 是 WASM-GC 构建，需要 Chrome/Edge 119+、Firefox 120+、Safari 18.2+ |
| 存档 | 保存在浏览器 IndexedDB，按域名隔离。换域名 = 存档不通用；清浏览器数据会丢档 |
| 联机 | 内置 lax1dude 的三个公共 relay，可直接加入 Eaglercraft 服务器。自建服务器请改各版本 `index.html` 里的 `relays` / `servers` |
| 响应头 | `_headers` 已给 `assets.epk` 设一年 immutable 缓存；不要加 COOP/COEP，会破坏公共 relay |
| 自定义域名 | Pages 项目 → Custom domains → 绑定，Cloudflare 会签证书 |

## 更新版本

直接替换对应目录的 `index.html` / `classes.js` 重新上传即可。若换了构建，记得同步改启动页上的体积与特性文案。

## 版权说明

Eaglercraft 为 lax1dude 及贡献者的开源项目（源码见 <https://github.com/lax1dude/eaglercraft>），其客户端包含 Mojang 的游戏资源与代码，属于非官方衍生作品。仅供个人学习与自用，公开分发存在合规风险，请自行判断。本项目与 Mojang / Microsoft 无任何关联。
