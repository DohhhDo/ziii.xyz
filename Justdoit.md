### Justdoit 执行指导单 //by=>friday

#### 约束与原则 //by=>friday
- 目标：最低失误率，小步快跑，可回退，可验证 //by=>friday
- 禁止大改页面与 UI；优先通过“服务适配层”接入新后端 //by=>friday
- 每次 edits 在相关行或段尾添加 `//by=>friday` 注释 //by=>friday
- 任何不可行或高风险点，先记录并采用替代方案，确保路径通畅 //by=>friday

#### 前置检查 //by=>friday
- Node >= 18，pnpm 已安装；`.env` 按 `env.mjs` 必填项配置 //by=>friday
- 可访问现有线上依赖（Sanity/Upstash/Neon）或已准备本地等价服务 //by=>friday
- 构建快速体检：`pnpm install && pnpm build`，记录报错与警告 //by=>friday

---

## 阶段A：Next 本地运行与运行时统一（最快落地） //by=>friday

### A1. 统一运行时为 node（保留图像两处为 edge） //by=>friday
- 要做的内容 //by=>friday
  - 将以下文件中的 `export const runtime = 'edge'` 改为 `'nodejs'`： //by=>friday
    - `app/robots.ts`、`app/sitemap.ts`、`app/api/activity/route.ts`、`app/api/reactions/route.ts` //by=>friday
  - 暂保留：`app/api/link-preview/route.tsx`、`app/api/favicon/route.tsx` 为 edge（因 `next/og`） //by=>friday
- 好做的方法 //by=>friday
  - 全局检索 `runtime = 'edge'`，逐文件修改并在行尾追加 `//by=>friday` //by=>friday
- 可能的问题 //by=>friday
  - 将图像接口改为 node 后 `next/og` 可能不兼容（渲染异常或性能退化） //by=>friday
- 替代方案 //by=>friday
  - 保留图像两处为 edge；若必须统一 node，临时降级为占位图或改为其他图像生成 //by=>friday
- 验收 //by=>friday
  - 构建通过，无 edge-only 报错；Node 启动后无相关运行时错误 //by=>friday
- 回退 //by=>friday
  - 行级回退为 `'edge'`，不影响其他路由 //by=>friday

### A2. 验证页面与接口 //by=>friday
- 要做的内容 //by=>friday
  - 页面：`/`、`/blog`、`/blog/[slug]`、`/guestbook`、`/friends` //by=>friday
  - SEO：`/sitemap.xml`、`/robots.txt` //by=>friday
  - API：`/api/activity`、`/api/reactions`、`/api/link-preview`、`/api/favicon` //by=>friday
- 好做的方法 //by=>friday
  - 浏览器/终端调用并观察日志；重点关注 `next/og`、图片远程白名单、CORS //by=>friday
- 可能的问题 //by=>friday
  - 远程图片域名未在 `next.config.mjs images.remotePatterns` 中放行 //by=>friday
- 替代方案 //by=>friday
  - 先放行域名；若仍跨域，临时反代资源或禁用有问题的模块后分阶段上线 //by=>friday
- 验收 //by=>friday
  - 页面与 API 响应正常，无致命错误；日志无异常堆栈 //by=>friday

---

## 阶段B：数据库本地化（Neon → 本地 PostgreSQL） //by=>friday

### B1. 切换 Drizzle 适配为 node-postgres（仅一处文件） //by=>friday
- 要做的内容 //by=>friday
  - 安装依赖：`pnpm add pg` //by=>friday
  - 修改 `db/index.ts`： //by=>friday
    - `import { Pool } from 'pg'` //by=>friday
    - `import { drizzle } from 'drizzle-orm/node-postgres'` //by=>friday
    - `const pool = new Pool({ connectionString: env.DATABASE_URL })` //by=>friday
    - `export const db = drizzle(pool)` //by=>friday
  - 保留 Neon 依赖，便于回退 //by=>friday
- 好做的方法 //by=>friday
  - 仅改一处文件，调用层无感；在行尾注明 `//by=>friday` //by=>friday
- 可能的问题 //by=>friday
  - 本地 PostgreSQL 未启动或无权限；连接参数（SSL/池大小）不当 //by=>friday
- 替代方案 //by=>friday
  - 先回退至 Neon；或用 Docker 快速起 PG 并开放本地连接 //by=>friday
- 验收 //by=>friday
  - `pnpm db:push` 成功；读写 `guestbook/comments/newsletter/admin` 冒烟通过 //by=>friday
- 回退 //by=>friday
  - 恢复 `db/index.ts` 至 Neon 版本，立即生效 //by=>friday

### B2. 迁移与冒烟 //by=>friday
- 要做的内容 //by=>friday
  - `.env` 设置 `DATABASE_URL=postgres://user:pass@localhost:5432/jcblog` //by=>friday
  - 迁移：`pnpm db:generate && pnpm db:push`（或直接 `pnpm db:push`） //by=>friday
  - 冒烟：留言、评论、简报创建与查询 //by=>friday
- 可能的问题与替代方案 //by=>friday
  - 时区不一致 → 统一使用 UTC；连接占满 → 下调池大小或引入 PGBouncer //by=>friday

---

## 阶段C：Sanity → Strapi（服务适配层最小改造） //by=>friday

### C1. 部署与建模 //by=>friday
- 要做的内容 //by=>friday
  - 部署 Strapi + PostgreSQL，创建管理员 //by=>friday
  - 建内容类型：post/category/project/friend/settings，与前端消费字段对齐 //by=>friday
  - 开放 Public 只读与 CORS；上传少量样本数据与图片 //by=>friday
- 可能的问题 //by=>friday
  - 图片 URL 为相对路径、CORS 拒绝、HTTP/HTTPS 混合内容 //by=>friday
- 替代方案 //by=>friday
  - 适配层拼绝对 URL；`next.config.mjs` 放行 Strapi 域名；反代 `/uploads` 至 HTTPS //by=>friday

### C2. 新增“内容服务适配层”（待审批后创建文件） //by=>friday
- 要做的内容 //by=>friday
  - 新建 `lib/content-service.ts`（获批后）：导出与 `~/sanity/queries` 等价的函数： //by=>friday
    - `getLatestBlogPosts({ limit, offset, forDisplay })` //by=>friday
    - `getBlogPostsCount()` //by=>friday
    - `getBlogPost(slug)` //by=>friday
    - `getAllLatestBlogPostSlugs()` //by=>friday
    - `getSettings()` //by=>friday
  - 字段映射：媒体 `{ url:absolute, dimensions, alt }`；`categories`→`string[]`；`_id=String(id)`；`publishedAt` 为 ISO；`lqip/dominant` 为空 //by=>friday
- 好做的方法 //by=>friday
  - 写一个 `toAbsoluteMedia()` 工具函数集中做 URL 与尺寸映射 //by=>friday
- 可能的问题 //by=>friday
  - 分页总数与 slug 全量获取 //by=>friday
- 替代方案 //by=>friday
  - 使用 `meta.pagination.total`；循环分页拉取 slug //by=>friday

### C3. 替换 8 处 Sanity 导入为适配层导出 //by=>friday
- 要做的内容 //by=>friday
  - 保持页面/组件不动，仅替换 import 来源 //by=>friday
  - 逐处验证渲染，无错再进行下一处 //by=>friday
- 可能的问题与替代方案 //by=>friday
  - 字段缺失 → 适配层补齐或返回空值（前端已有降级逻辑） //by=>friday

### C4. 正文渲染策略（二选一） //by=>friday
- 方案A（推荐，最小改动） //by=>friday
  - 在 `app/(main)/blog/[slug]/page.tsx` 用 `react-markdown` 渲染 `post.body` //by=>friday
  - `app/(main)/feed-full.xml/route.ts` 将 Markdown 转纯文本；`headings` 在页面侧解析 //by=>friday
  - 改动文件少（2~3 处），上线快 //by=>friday
- 方案B（保持现有 UI，开发量大） //by=>friday
  - 适配层将 Markdown 转为“简化 PortableText 块”，继续喂给 `PostPortableText` //by=>friday
  - 测试成本高，非优先路径 //by=>friday

### C5. 图片、SEO、RSS、sitemap 回归 //by=>friday
- 要做的内容 //by=>friday
  - `next.config.mjs images.remotePatterns` 加入 Strapi 域名；适配层输出绝对 URL //by=>friday
  - 验证 `app/(main)/feed*.xml` 与 `app/sitemap.ts` 字段完整与计数正确 //by=>friday
- 可能的问题与替代方案 //by=>friday
  - `lqip/dominant` 缺失 → 前端已可降级，后续再补占位或主色 //by=>friday

---

## 后续：Upstash Redis 本地化替代（不在本阶段强推） //by=>friday
- 现状：`@upstash/redis` 基于 HTTP REST，非本地 Redis 客户端 //by=>friday
- 方向：后续评估改为 `ioredis` + 自研滑动窗口限流；或长期保留 Upstash //by=>friday

---

## 回退矩阵 //by=>friday
- 运行时：逐路由切换 `edge/nodejs`，行级回退 //by=>friday
- 数据库：仅 `db/index.ts` 一处切换 Neon/node-postgres，立即回退 //by=>friday
- 内容源：通过 `CONTENT_PROVIDER`（sanity/strapi）环境开关双栈共存 //by=>friday

---

## 每日汇报模板 //by=>friday
- 今日完成：A1/A2/B1…（文件与提交点） //by=>friday
- 遇到问题：描述、影响范围、当前临时方案 //by=>friday
- 明日计划：下一最小闭环事项 //by=>friday
- 需批准：新增文件/运行时变更/环境变量/第三方服务 //by=>friday

---

## 速查清单 //by=>friday
- Sanity 使用点（8 处）：`app/(main)/page.tsx`、`app/(main)/blog/BlogPosts.tsx`、`app/(main)/blog/page.tsx`、`app/(main)/blog/[slug]/page.tsx`、`app/sitemap.ts`、`app/(main)/friends/Projects.tsx`、`app/(main)/feed.xml/route.ts`、`app/(main)/feed-full.xml/route.ts` //by=>friday
- `next/og` 使用点：`app/api/link-preview/route.tsx`、`app/api/favicon/route.tsx` //by=>friday
- 数据库切换点：`db/index.ts` //by=>friday
- 远程图片白名单：`next.config.mjs images.remotePatterns` //by=>friday

---

## 执行顺序建议 //by=>friday
1) A1→A2：先统一运行时并验证 //by=>friday
2) B1→B2：数据库最小切换并冒烟 //by=>friday
3) C1→C5：Strapi 接入与适配层替换（正文优先选方案A） //by=>friday



## 阶段D：图片托管切换至腾讯云 COS（最小变更） //by=>friday

### D1. 准备与域名 //by=>friday
- 新建 COS 桶（公有读），绑定 CDN 自定义域名并开启 HTTPS；记下最终访问域名（如 `img.example.com`）。 //by=>friday

### D2. 放行前端白名单（仅一处 edits） //by=>friday
- 修改 `next.config.mjs images.remotePatterns`，新增： //by=>friday
  - `{ protocol: 'https', hostname: 'img.example.com', port: '', pathname: '/**' }`（或 COS 桶域名） //by=>friday
- 保留现有：`cdn.sanity.io/youke1.picui.cn/gitee.com`，迁移完成后再按计划移除。 //by=>friday
- 验收：`pnpm build` 不报错；本地 `pnpm dev` 访问时 COS 图片正常显示。 //by=>friday

### D3. 内容侧切换（不改代码，仅数据规范） //by=>friday
- 新增或更新的图片字段一律填写“COS/CDN 绝对 URL”。 //by=>friday
- 历史内容先不强制迁移；页面同时兼容旧域名。 //by=>friday

### D4. 冒烟与监控 //by=>friday
- 页面与组件：检查首页 Photos、文章主图、友链/项目 Logo、简历 Logo、富文本内联图。 //by=>friday
- 网络：CDN 命中率、是否出现 403、防盗链/Referer 是否正确放行。 //by=>friday
- 协议：全站 HTTPS，无混合内容；浏览器控制台无跨域错误。 //by=>friday

### D5. 回退与收尾 //by=>friday
- 回退：仅需在内容中恢复旧域名或临时移除 COS 域名白名单即可。 //by=>friday
- 收尾：历史图片逐步迁移至 COS，确认无引用后再移除旧域名白名单。 //by=>friday

### 需批准项 //by=>friday
- 允许对 `next.config.mjs` 进行 1 处 edits（新增 COS/CDN 域名条目，并在行尾追加 `//by=>friday`）。 //by=>friday
- 如需新增工具或迁移脚本，将在文档中提出并二次申请。 //by=>friday