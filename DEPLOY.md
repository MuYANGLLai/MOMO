# PWA 部署说明 · v1.1.0

这个包是纯静态 PWA，包含 5 本词书（5,335 个去重词条）。上传后即可访问和安装。压缩包内文件位于根目录，直接解压到站点根目录即可。

## 包内容

```
index.html
manifest.webmanifest
sw.js
version.json
assets/
  index-*.css
  index-*.js
icons/
  icon-192.png
  icon-512.png
  icon-maskable-192.png
  icon-maskable-512.png
  apple-touch-icon.png
  favicon-32.png
_headers
vercel.json
DEPLOY.md
```

## 版本与更新检查

- `version.json` 和 `index.html` 的 `<meta name="app-version">` 都写着当前版本。
- App 内「我的 → 检查更新」会拉取 `https://muyangllai.github.io/MOMO/` 的版本标记；发现更高版本时打开在线最新版。
- 以后发布新版本，请同时修改 `version.json` 的 `version` 与 `index.html` 的 `app-version`，并更新 `sw.js` 的缓存名（例如 `momo-word-v5`），避免旧壳缓存。

## 部署要求

- **必须使用 HTTPS**（或 localhost）。PWA 的 Service Worker 和「安装到主屏幕」只在安全上下文可用。
- 静态托管即可，不需要 Node 运行时、数据库或后端。
- 不要修改 `assets/` 下的文件名；它们带内容哈希，`index.html` 和 `sw.js` 会按这些名字加载。
- 站点根目录必须是本包内容。若部署在子目录（例如 `https://example.com/momo/`），当前相对路径也可以工作，但请确认 `/momo/manifest.webmanifest`、`/momo/sw.js` 均可访问。

## 最快部署

### Netlify Drop

1. 打开 https://app.netlify.com/drop
2. 把解压后的文件夹（或本 zip）拖进去
3. 得到 HTTPS 地址，用 Chrome 打开后点击地址栏安装图标

### Vercel

```bash
npx vercel --prod
```

在解压后的目录中执行，按提示选择静态项目即可。

### Cloudflare Pages

Dashboard → Workers & Pages → Create → Pages → Upload assets，上传解压后的文件夹。

### GitHub Pages

把包内文件推到仓库的 Pages 分支/目录，开启 Pages。相对路径已适配子路径部署。

### Nginx

```nginx
server {
  listen 443 ssl;
  server_name example.com;
  root /var/www/momo-word;
  index index.html;

  location / {
    try_files $uri $uri/ /index.html;
  }

  location = /sw.js {
    add_header Cache-Control "no-cache";
  }
}
```

## 缓存策略

包内已经附带 Netlify 的 `_headers` 和 Vercel 的 `vercel.json`：

- `assets/*`：一年强缓存，文件名带哈希，可安全长期缓存
- `sw.js`、`manifest.webmanifest`、`version.json`、`index.html`：不缓存，保证发布更新后能拿到新版本
- `icons/*`：一周缓存

Service Worker 采用：
- 导航请求：网络优先，离线时回退到缓存的 `index.html`
- 静态资源：缓存优先，未命中时请求并写入缓存

## 部署后验证

1. 打开 HTTPS 地址，确认首页能进入引导。
2. Chrome DevTools → Application：
   - Manifest 状态正常，图标 192/512 均能加载
   - Service Workers 状态为 activated and running
3. 刷新页面，地址栏出现安装图标；安装后从桌面图标启动。
4. 断网后刷新，页面仍能打开（首次缓存完成后）。
5. 学习几个单词，刷新确认进度仍在；再测试「我的 → 备份与还原」。

## 数据说明

- 所有学习数据保存在浏览器的 `localStorage`，按「协议 + 域名 + 端口」隔离。
- 换域名、换端口、清除站点数据都会导致本地进度不可见；请先在「备份与还原」导出 JSON。
- 没有账号、没有云端同步、没有埋点。

## Windows 提示

这个 zip 是给静态托管上传用的，**不需要双击打开**。如果 Windows 因“来自 Internet”的标记弹出安全提示：

- 右键 zip → 属性 → 勾选「解除锁定」→ 确定
- 或 PowerShell：

```powershell
Unblock-File .\momo-word-pwa-v1.1.0.zip
```

这不是文件损坏，也不影响部署。
