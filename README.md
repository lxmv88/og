# 微信 OG 链接卡片 · 测试包

用于预览「密信」分享链接在微信里的卡片效果（**无称呼 / 无手机尾号**）。

## ⚠️ Netlify Drop 免费版不可用

Netlify Drop 部署的站点默认带 **密码保护（401）**，关闭需付费订阅，**微信爬虫进不去**。请改用 **GitHub Pages**（见 `deploy-github-pages.md`）。

## ⚠️ 临时隧道（lhr.life / ngrok）仍可能只有纯链接

即使 OG 标签正确、页面能在微信里打开，**免费穿透域名**也经常被微信降级为纯链接，原因包括：

- 域名无 ICP 备案、信誉低（`*.lhr.life`、`*.loca.lt`、`*.ngrok-free.app` 等）
- 微信对陌生域名的链接卡片展示策略收紧（安全策略，非代码 bug）

**结论：本地隧道适合验证页面能打开，不适合作为「微信 OG 是否生效」的最终依据。**

要稳定看到卡片，请用 **已备案的正式域名**（如 `s.secreit.cn`），或下面「Netlify 一键部署」。

## ⚠️ 为什么 loca.lt 只有纯链接、没有卡片？

**localtunnel 会拦截微信爬虫。** 用微信 UA 访问 `*.loca.lt` 时，返回的是「请先输入 IP 确认」页面（HTTP 511），**不是**你的 OG 页面，所以微信只能显示纯链接。

**不要用 localtunnel 测微信 OG。** 请改用：

- **localhost.run**（本机已验证可用，见下方命令）
- **cloudflared** / **ngrok**（无拦截页）
- 正式域名（如 `s.secreit.cn`）或 GitHub Pages / Vercel 静态部署

## ⚠️ 常见错误：发成「网页文件」

| 错误做法 | 结果 |
|----------|------|
| 把 `h5-landing-og.html` **文件**拖进微信 / 复制本地路径 | 显示为 **网页文件** 附件，**没有**链接卡片 |
| 打开 `wechat-card-mockup.html` 截图发微信 | 只是图片，不是真实 OG |
| 发 `file:///Users/.../h5-landing-og.html` | 微信无法抓取 |

**正确做法：** 在聊天输入框里 **只粘贴 HTTPS 链接文字**，例如：

```
https://ten-hats-slide.loca.lt/h5-landing-og.html
```

然后按发送。微信会抓取该 URL 的 `og:title` / `og:description` / `og:image` 生成卡片。

> `wechat-card-mockup.html` 仅供本机双击预览样式，**不要**发到微信里测 OG。

## 文件

| 文件 | 用途 |
|------|------|
| `wechat-card-mockup.html` | **本地模拟**微信聊天气泡 + 链接卡片（双击即看） |
| `h5-landing-og.html` | 带 OG 标签的 **H-L 落地页**，部署后真机分享 |
| `og-image.jpg` | 链接卡片缩略图（1200×630 JPEG，约 60KB，**微信请用此文件**） |
| `og-image.png` | 设计稿原图（约 1.2MB，过大，微信可能抓取失败） |

## 1. 本地先看模拟（无需部署）

双击打开 `wechat-card-mockup.html`，切换「默认 / 带留言」查看卡片样式。

## 2. 微信真机测试（需公网 HTTPS）

微信爬虫 **只读 HTML 源码**，**不执行 JavaScript**。`og:image` 必须是写在 HTML 里的 **绝对 HTTPS URL**。

### 推荐：GitHub Pages（免费、无密码、适合微信 OG）

Netlify Drop 免费版会强制站点密码保护，微信爬虫无法访问。

请按 **`deploy-github-pages.md`** 操作（全程浏览器，约 5 分钟）：

1. 新建公开仓库 → 上传 `h5-landing-og.html` + `og-image.jpg`
2. Settings → Pages → 从 `main` 分支发布
3. `./set-og-base.sh https://<用户名>.github.io/<仓库名>`
4. 覆盖上传 HTML 后，微信发送：`.../h5-landing-og.html?v=1`

### 备选：Netlify Drop（需付费才能关密码保护，已不推荐）

### 备选：localhost.run / trycloudflare（仅验证页面可访问，微信卡片不稳定）

终端 1 — 本地静态服务：

```bash
cd pm-docs/prototype/og-preview
python3 -m http.server 8765
```

终端 2 — 公网隧道：

```bash
ssh -T -R 80:localhost:8765 nokey@localhost.run
```

复制终端里显示的 `https://xxxx.lhr.life`，然后更新 OG 地址：

```bash
./set-og-base.sh https://xxxx.lhr.life
```

发到微信的链接：

```
https://xxxx.lhr.life/h5-landing-og.html
```

### 当前可测链接（localhost.run，会话内有效）

```
https://3db0688f4c1350.lhr.life/h5-landing-og.html
```

> 隧道断开后 URL 失效，需重新跑 `ssh` 命令并执行 `set-og-base.sh`。

### 其他穿透方式

| 场景 | URL |
|------|-----|
| 默认 OG | `https://xxx/h5-landing-og.html` |
| 带留言 OG | `https://xxx/h5-landing-og.html?note=会议室Wi-Fi密码看完即焚` |
| 无密码 | `https://xxx/h5-landing-og.html?pwd=0` |

### OG 文案（当前方案）

- **og:title**：`密信 · 安全阅读邀请`
- **og:description（默认）**：`一次性加密消息，仅可查看一次，无需安装 App`
- **og:description（有 note 参数）**：留言前 30 字（仅浏览器内 JS 生效；微信默认仍读静态 description，正式环境需 SSR）

## 3. 注意

- 微信会 **缓存** OG，改完 meta 后可能需换 URL 或等缓存过期
- `og:image` 使用 **`og-image.jpg`**（< 300KB），不要用 1.2MB 的 png
- 正式环境域名建议：`s.secreit.cn`
- 若卡片仍不显示：用 [微信分享调试工具](https://developers.weixin.qq.com/doc/offiaccount/enrichment/share/share_debugger.html) 查抓取结果