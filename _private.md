# 多项目域名规范（silencewood.cn）

> 定稿：2026-10-21。依据：DNSPod 官方套餐配额页、GitHub Pages 自定义域名与路径继承规则、本机实测。
> 适用：本站（next-blogs）以及后续所有需要线上访问的仓库。

---

## 0. 结论

| 决策 | 取值 |
| --- | --- |
| 正式项目的地址 | `xxx.silencewood.cn`（一条 CNAME） |
| 试验 / 临时项目的地址 | `silent-wood.github.io/<repo>`（零配置，开 Pages 即有） |
| apex `silencewood.cn` | 独立仓库承载门户页（不放跳转） |
| 子路径 `/repo` 形态 | **不采用**（原因见 §4） |
| apex 的 URL 转发 | **不用**（未备案不可用，见 §5） |

一句话：**域名按项目切分（子域），不按路径切分。**

---

## 1. 三层地址，按项目重要性选

| 层级 | 形态 | 新增成本 | 适用 |
| --- | --- | --- | --- |
| L0 | `silent-wood.github.io/<repo>` | 0，开 Pages 即得 | 试验、Demo、临时页 |
| L1 | `<repo>.silencewood.cn` | 1 条 CNAME + 填一次域名 | 正式对外项目 |
| L2 | `silencewood.cn/<repo>` | user site + 每项目 basePath | 不采用 |

L0 始终存在，与是否绑域名无关。所以「能不能线上访问」不依赖域名规划；域名只决定它挂牌在哪。

---

## 2. 新增一个项目（SOP）

1. 新建仓库，推代码，`Settings → Pages` 开通（Actions 或 branch 均可）。
2. `Settings → Pages → Custom domain` 填 `<repo>.silencewood.cn`，先保存（此时会提示 DNS 未配置，正常）。
3. DNSPod 加记录：主机记录 `<repo>`、类型 `CNAME`、记录值 `silent-wood.github.io`、TTL 600。
4. 回 Pages 页面等 DNS 校验变绿 → 勾 **Enforce HTTPS**。
5. 验证：

```bash
dig +short blog.example.com CNAME      # 应返回 silent-wood.github.io.
curl -sI https://<repo>.silencewood.cn | head -5   # 期望 200 或 301，证书无告警
```

**注意**：一个仓库只能绑一个自定义域名；不要加 `public/CNAME` 文件；CNAME 记录值不要带协议。

---

## 3. 命名约定

- 子域全小写、短、优先无连字符：`blogs`、`notes`、`lab`、`app`。
- **子域与仓库名解耦**：仓库可以改名，URL 不变；反之想改 URL 时按 §7 的顺序操作。
- 避开会被邮件或验证记录占用含义的名字（`mail`、`smtp`、`_dmarc` 等）。
- 显示名可以是中文，子域保持 ASCII。

---

## 4. 为什么不用 `silencewood.cn/<repo>`

这条路在 GitHub Pages 上可行，但有四个结构性代价：

1. **硬前提**：路径继承只从 **user site** 发出，仓库名必须精确等于 `silent-wood.github.io`。随便建个 `home` 绑 apex 不会产生任何继承。
2. **每个项目都要背 basePath**：Next `basePath`、Vite `base`、Astro `base`、纯静态手改绝对路径；图片、fetch、OG 图、sitemap、feed 全部要照顾前缀，漏一处就白屏。
3. **强耦合**：URL 前缀写进构建产物。某个项目将来要加后端或迁到别家托管，必须重建前缀并依赖 user site 关系；子域只需改一条 CNAME，其他项目零影响。
4. **互斥**：apex 门户与路径继承占同一个位置（user site），只能二选一。

结论：项目越多、技术栈越杂，路径式的重复成本越高。子域的边际成本恒定（一条记录）。

---

## 5. apex 为什么不用 URL 转发

DNSPod 官方文档（`docs.dnspod.cn/dns/url-before-domian-error`）明确：

> URL 记录功能在进行转发时，将使用 DNSPod 的 URL 转发服务器进行转发，故转发前域名需在腾讯云完成备案或接入备案后才可正常使用。
> 添加 URL 转发记录时，**涉及到的两个域名**已完成备案。未经备案的域名无法添加 URL 转发。

`silencewood.cn` 是境外托管、未做 ICP 备案，所以「显性 URL」记录保存时报「域名未备案」。这与套餐无关，升级付费版也不能绕过。

替代做法：apex 放**真实页面**（门户页），`@` 用 A 记录、`www` 用 CNAME，属于普通解析，不触发备案校验。

---

## 6. 配额与限制（官方数据，2026-10-21 核实）

DNSPod 免费版：

| 项 | 额度 |
| --- | --- |
| 解析记录数 | **无限** |
| 子域名数量 | 10 万个 |
| 子域名级数 | 5 级 |
| URL 转发 | 2 条 |
| 负载均衡（同一主机 A 记录） | 2 条 |
| TTL 最小值 | 600s |

GitHub Pages：

- 一个仓库只能绑定 **1 个**自定义域名。
- 免费版 Pages 要求仓库为 public。
- project site 未绑域名时，`silent-wood.github.io/<repo>` 始终可用。

---

## 7. 迁移、下线与改名

| 场景 | 操作顺序 |
| --- | --- |
| 单项目迁到别家托管 | 先在新主机配好站点 → 改 CNAME 指向新主机 → 观察 24h → 关旧 Pages。**其他项目不受影响。** |
| 项目下线 | 关 Pages（自动释放域名）→ 删 DNSPod CNAME。先关 Pages 再删记录，避免中间态 404 被索引。 |
| 想换子域前缀 | 先加新 CNAME + 新绑定，等证书绿 → 改站内链接与 canonical → 再删旧记录。不要反向操作。 |
| 仓库改名 | 子域不受影响（解耦）。但 Actions 里的 `vars`、Pages 的 artifact 名要复核。 |

---

## 8. 新增项目检查清单

- [ ] 仓库 public，Pages 已开通且首次部署成功
- [ ] Custom domain 填的是 `<repo>.silencewood.cn`，无协议、无斜杠
- [ ] DNSPod 有一条对应 CNAME → `silent-wood.github.io`
- [ ] DNS 校验绿，Enforce HTTPS 已勾
- [ ] `https://<repo>.silencewood.cn` 200，证书无告警
- [ ] 站点内 canonical / RSS / sitemap 的域名是该子域（不是 apex，也不是 github.io）
- [ ] 门户页 `index.html` 加了一张卡片
- [ ] 未添加 `public/CNAME` 文件
