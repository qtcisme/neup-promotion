# 推广部招新网站 — 部署说明

## 🟢 正式访问地址（推荐分享这个）

```
https://qtcisme.github.io/neup-promotion/
```

- **HTTPS** 加密，**全球可访问**（不需要 IPv6，手机流量 / 任何网络都能打开）
- 托管在 GitHub Pages，仓库：https://github.com/qtcisme/neup-promotion
- 推送 `main` 分支后自动重新部署（约 1 分钟）

### 备用地址（自有服务器）

```
http://[2a01:111:f100:6000::4134:a761]:8080/
```

托管在自己的 Azure VM 上，仅 IPv6 可达，作为备份/校园网内直连使用。IPv6 地址放进 URL 必须用方括号包起来——这是 URL 规范要求，不是笔误。

---

## 一、更新网站内容

### 方式 A：改 GitHub Pages（正式站）

```bash
# 在 neup-site 目录下
git add -A
git commit -m "更新内容"
git push
```

推送后 GitHub Actions 自动部署，约 1 分钟生效。查看部署状态：

```bash
gh run list --limit 5
gh run watch
```

### 方式 B：改自有服务器（备用站）

```bash
scp index.html changerning@[2a01:111:f100:6000::4134:a761]:/var/www/neup-site/index.html
```

HTML 已配置 `no-cache`，刷新即生效，无需重启服务。

---

## 二、自有服务器的部署信息（备用站）

| 项目 | 值 |
|---|---|
| 服务名 | `neup-site.service`（systemd，已 enable 开机自启） |
| nginx 配置 | `/etc/nginx/neup-site.conf`（**独立实例，与 sing-box 自带的 nginx 完全隔离**） |
| 网站根目录 | `/var/www/neup-site/` |
| 监听端口 | 8080（IPv4 + IPv6 双栈） |
| 日志 | `/var/log/nginx/neup-site-access.log`、`neup-site-error.log` |

**为什么用 8080 而不是默认 80**：sing-box 的订阅服务占用了 52617，而 80 端口未放通。选 8080 是为了和免流服务完全隔离，将来重跑 sing-box 安装脚本不会影响这个网站。

```bash
systemctl status neup-site        # 查看状态
systemctl restart neup-site       # 重启
nginx -t -c /etc/nginx/neup-site.conf   # 校验配置
```

---

一个**零依赖的单文件静态网站**：全部 HTML / CSS / JS 都在 `index.html` 里，没有任何外部依赖（无 CDN、无字体请求、无构建步骤）。

---

## 三、文件

```
neup-site/
├── index.html                     网站全部内容（约 43 KB）
├── README.md                      本文件
└── .github/workflows/pages.yml    GitHub Pages 自动部署工作流
```

## 二、本地预览

直接双击 `index.html` 即可。或起一个本地服务器（推荐，行为更接近线上）：

```bash
# Python 3
python -m http.server 8080

# 或 Node
npx serve .
```

然后访问 http://localhost:8080

## 三、部署到线上

### 方案 A：GitHub Pages（最省事，免服务器）

1. 新建仓库，把 `index.html` 推到根目录
2. 仓库 → Settings → Pages → Source 选 `Deploy from a branch` → 选 `main` / `/ (root)` → Save
3. 等 1 分钟，访问 `https://<用户名>.github.io/<仓库名>/`

### 方案 B：自己的服务器 / Nginx

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name recruit.example.com;
    root /var/www/neup-site;
    index index.html;
    location / { try_files $uri $uri/ /index.html; }
}
```

把 `index.html` 放到 `/var/www/neup-site/` 即可。

### 方案 C：其他静态托管

Cloudflare Pages / Vercel / Netlify 都可以，构建命令留空，发布目录填 `neup-site`（或把 `index.html` 放根目录）。

## 四、需要改的地方

搜索 `index.html` 里的 `mengxin2026.neupioneer.team`，共 6 处，都是跳转到面前题文档站的链接。如果要换成本组织的正式域名，全部替换即可。

其他可改项：

| 想改什么 | 位置 |
|---|---|
| 标题 / 副标题 | `<h1>` 与 `.hero .lead` |
| 首屏四个数字 | `.hero-stats` 里的 `.stat` |
| 部门介绍文案 | `#about` 区块 |
| 适合谁 / 不适合谁 | `#fit` 区块 |
| 10 道题与答案 | `#qa` 区块里的 10 个 `<details class="qa">` |
| 加入流程四步 | `#join` 区块 |
| 主题色 | `:root` 里的 `--brand` / `--brand-2` / `--brand-3` |
| 页脚版权 | `<footer>` |

改主题色一处即可全站生效（渐变、按钮、卡片高亮都会跟着变）。

## 五、已实现的功能

- **响应式**：断点 860px / 400px，实测 390px 手机宽度下无横向溢出
- **暗色 / 亮色主题**：右上角切换，自动跟随系统偏好，并用 localStorage 记住选择
- **面前题手风琴**：点击展开，一次只开一个
- **滚动出现动画**：用 IntersectionObserver，无依赖
- **无障碍**：语义化标签、`aria-label`、`aria-expanded`、键盘可操作
- **省电与晕动症友好**：`prefers-reduced-motion` 下自动关闭动画
- **分享友好**：带 Open Graph 元信息，微信 / QQ 分享有标题和描述
- **无外部请求**：不依赖 CDN，内网 / 校园网环境也能正常打开

## 六、浏览兼容性

使用了 `color-mix()`（Chrome 111+ / Safari 16.2+ / Firefox 113+）做半透明背景。在这些版本以上的浏览器效果最佳；更老的浏览器会退化为不透明背景，**不影响内容可读性和功能**。

---

## 七、注意

网站里的 10 道答案是我基于题目自作的理解，**面试时建议用自己的话重新组织，并补上你真实的经历**。直接背稿容易被追问穿；写出"你的版本"才是这个网站和这份答案的真正价值。
