# silencewood-home

`silencewood.cn` 的门户页（apex 根路径）。单文件静态站，无构建步骤、无依赖。

## 它解决什么问题

博客迁到 `blogs.silencewood.cn` 之后，根域名 `silencewood.cn` 需要一个落点。原本计划的 DNSPod「显性 URL 转发」对未备案域名不可用（转发服务在境内，受 ICP 约束），所以改用**真实页面**承载根路径：既保住根域名可访问，也顺手成为一个项目入口。

## 首次部署（网页操作，约 5 分钟）

1. 在 GitHub 新建仓库 `silencewood-home`（public），把本目录推上去。
2. `Settings → Pages`：Source 选 **Deploy from a branch**，分支 `main`、目录 `/ (root)`，保存。
3. 等首次部署完成，确认 `silent-wood.github.io/silencewood-home/` 可访问。
4. 同一页面的 **Custom domain** 填 `silencewood.cn`，保存。
5. DNSPod 加记录（`@` 用 A、`www` 用 CNAME）：

   | 主机记录 | 类型 | 记录值 |
   | --- | --- | --- |
   | `@` | A | `185.199.108.153` |
   | `@` | A | `185.199.109.153` |
   | `www` | CNAME | `silent-wood.github.io` |

   > 免费版同一主机记录的 A 记录上限 2 条，两条即可，GitHub 校验可通过。

6. 回到 Pages 页面等 DNS 校验变绿，勾选 **Enforce HTTPS**。
7. 访问 `https://silencewood.cn` 验证；`https://www.silencewood.cn` 会自动 301 到 apex。

## 不要做的事

- **不要**在本仓库放 `CNAME` 文件。自定义域名在 Pages Settings 里管，放文件是冗余的，还可能和设置冲突。
- **不要**把仓库改名为 `silent-wood.github.io`。那会变成 user site，并且让所有 project site 自动继承 `/仓库名` 路径——与本目录的门户定位冲突（apex 只能有一个归属）。

## 加一个项目入口

在 `index.html` 的 `<ul class="cards">` 里复制一个 `<li>`，改 `href`、标题、描述与 `card-meta` 即可。占位卡（`class="card ghost"`）保留在最后。

新项目本身的部署不在这里做：各自建仓库、各自开 Pages、各自绑 `xxx.silencewood.cn`。完整规范见 `next-blogs/.docs/多项目域名规范.md`。
