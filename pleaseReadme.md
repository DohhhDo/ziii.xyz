### 项目现状与服务使用方式 //by=>friday

#### 技术栈与目录结构 //by=>friday
- 使用 Next.js 14（App Router）、React 18、TypeScript、Tailwind CSS、Framer Motion、Radix UI 等，见 `package.json` 与 `README.md` 技术栈描述 //by=>friday
- 主要目录：`app/`（页面与 API 路由）、`lib/`（通用库与服务封装）、`db/`（Drizzle ORM 与迁移）、`sanity/`（CMS 查询与 Studio 配置）、`emails/`（React Email 模板）、`config/`（配置项） //by=>friday
- 环境变量集中校验：`env.mjs` 使用 Zod 合并校验服务端与客户端变量 //by=>friday

#### 部署与运行方式 //by=>friday
- 推荐部署在 Vercel/Netlify，生产中主要考虑 Vercel，见 `README.md` 部署说明 //by=>friday
- 本地运行通过 `pnpm dev`，生产构建与启动 `pnpm build && pnpm start` //by=>friday
- 运行时混合：部分路由使用 `edge`，其余为 `nodejs` 或默认服务端 //by=>friday
  - 已显式标注为 `edge` 的文件：`app/robots.ts`、`app/sitemap.ts`、`app/api/favicon/route.tsx`、`app/api/reactions/route.ts`、`app/api/link-preview/route.tsx`、`app/api/activity/route.ts` //by=>friday
  - 显式标注为 `nodejs` 的文件：`app/api/indexnow/route.ts`、`app/api/indexnow-key/route.ts` //by=>friday
  - 图片远程白名单见 `next.config.mjs images.remotePatterns`（`cdn.sanity.io`、`youke1.picui.cn`、`gitee.com`） //by=>friday

#### 内容管理（Sanity） //by=>friday
- Sanity 客户端：`sanity/lib/client.ts` 通过 `next-sanity` 创建，使用 `projectId/dataset/apiVersion/useCdn` 等 //by=>friday
- 内容查询：`sanity/queries.ts` 使用 GROQ 获取 `post`、`project`、`friend`、`settings` 等数据，返回结构被页面直接消费 //by=>friday
- 数据模型：`sanity/schemas/*` 与 `sanity/schema.ts` 汇总类型；图片字段常通过 `asset->url`、`metadata.lqip`、`palette.dominant` 提供低清与主色 //by=>friday
- Studio：`/app/studio/[[...index]]` 与 `sanity.config.ts` 管理内容后台 //by=>friday

#### 数据库（Neon PostgreSQL + Drizzle ORM） //by=>friday
- 连接：`db/index.ts` 使用 `@neondatabase/serverless` 的 `Pool` 并以 `drizzle(neon-serverless)` 建立连接，依赖 `env.DATABASE_URL` //by=>friday
- 表结构：`db/schema.ts` 定义 `subscribers`、`newsletters`、`comments`、`guestbook` 等 //by=>friday
- 迁移：`db/migrations/*`（包含快照与 `0000_shallow_iron_fist.sql`） //by=>friday
- 使用点：如留言板与评论相关 API（示例：`app/api/guestbook/route.ts`）通过 `db` 读写 //by=>friday

#### 缓存与限流（Upstash Redis） //by=>friday
- 封装：`lib/redis.ts` 使用 `@upstash/redis` 与 `@upstash/ratelimit`，依赖 `UPSTASH_REDIS_REST_URL/TOKEN` //by=>friday
- 使用场景： //by=>friday
  - 接口限流与计数：`app/api/activity/route.ts`、`app/api/reactions/route.ts` 等 //by=>friday
  - 第三方令牌与结果缓存：`lib/baidu.ts`（百度 AIP `access_token`）、`lib/alt.ts`（图片 Alt 文本缓存） //by=>friday

#### 认证与中间件 //by=>friday
- 依赖 `@clerk/nextjs`（见 `package.json`），中间件位于 `middleware.ts`，用于请求前置处理与潜在鉴权集成 //by=>friday

#### 邮件与通知 //by=>friday
- 使用 React Email 组件（`emails/`）与 `Resend` 发送（`lib/mail.ts`、`config/email.ts`），需要 `RESEND_API_KEY` 与发件配置 //by=>friday

#### SEO、Feed 与站点索引 //by=>friday
- 站点地图与 robots：`app/sitemap.ts`（edge）、`app/robots.ts`（edge） //by=>friday
- RSS/Feed：`app/(main)/feed.xml/route.ts`、`app/(main)/feed-full.xml/route.ts` //by=>friday
- IndexNow：`app/api/indexnow/route.ts` 与 `app/api/indexnow-key/route.ts`（依赖 `INDEXNOW_KEY`），并在 `app/[indexnowKey].txt/route.ts` 提供校验文件 //by=>friday

#### 外部服务与关键环境变量（摘自 `env.mjs`） //by=>friday
- 数据库：`DATABASE_URL`（Neon PostgreSQL） //by=>friday
- Redis/限流：`UPSTASH_REDIS_REST_URL`、`UPSTASH_REDIS_REST_TOKEN` //by=>friday
- Sanity：`NEXT_PUBLIC_SANITY_PROJECT_ID`、`NEXT_PUBLIC_SANITY_DATASET`、`NEXT_PUBLIC_SANITY_USE_CDN` //by=>friday
- 站点：`NEXT_PUBLIC_SITE_URL`、`NEXT_PUBLIC_SITE_EMAIL_FROM` //by=>friday
- 索引：`INDEXNOW_KEY` //by=>friday
- 第三方：`RESEND_API_KEY`、`BAIDU_AIP_CLIENT_ID`、`BAIDU_AIP_CLIENT_SECRET`、`ADMIN_EMAILS` 等 //by=>friday

以上内容仅描述“当前实现与服务使用方式”，为后续本地化替代（Vercel→自管、Sanity→自建内容源、Neon→本地 PostgreSQL、Upstash→本地 Redis）提供基线参照 //by=>friday


### Sanity 替换执行清单与风险核查 //by=>friday

- 覆盖的内容类型 //by=>friday
  - 文章 post、分类 category、项目 project、友链 friend、站点设置 settings（含 heroPhotos、resume） //by=>friday

- 全量使用点（直接依赖 `~/sanity/queries` 的文件） //by=>friday
  - `app/(main)/page.tsx`：`getSettings()`（首页 heroPhotos、resume） //by=>friday
  - `app/(main)/blog/BlogPosts.tsx`：`getLatestBlogPosts()`（文章列表卡片） //by=>friday
  - `app/(main)/blog/page.tsx`：`getBlogPostsCount()`（分页总数） //by=>friday
  - `app/(main)/blog/[slug]/page.tsx`：`getBlogPost(slug)`（详情页数据与 SEO） //by=>friday
  - `app/sitemap.ts`：`getAllLatestBlogPostSlugs()`（动态站点地图） //by=>friday
  - `app/(main)/friends/Projects.tsx`：`getSettings()`（项目列表来自 settings.projects） //by=>friday
  - `app/(main)/feed.xml/route.ts`：`getLatestBlogPosts()`（RSS 摘要） //by=>friday
  - `app/(main)/feed-full.xml/route.ts`：`getLatestBlogPostsWithBody()`（RSS 全文） //by=>friday

- 前端关键字段依赖（保持 DTO 一致或等价） //by=>friday
  - 列表卡片：`_id`、`title`、`slug`、`description`、`publishedAt`、`readingTime`、`categories[]`、`mainImage.asset.url`（`lqip/dominant` 可为空） //by=>friday
  - 详情页：上述字段 + `body`（正文）、`related[]`、`headings[]` //by=>friday
  - 图片组件：`{ url, dimensions:{ width,height }, lqip?, alt?, label? }` //by=>friday
  - 首页 Photos：字符串 URL 数组 //by=>friday
  - Resume：`{ company,title,start,end?,logo }`（logo 为字符串 URL） //by=>friday

- 执行路线（最小改动、低失误） //by=>friday
  1) 部署 Strapi + PostgreSQL，创建管理员账号 //by=>friday
  2) 在 Strapi 建模：post/category/project/friend/settings，与上文字段对齐；主图走媒体库 //by=>friday
  3) 打开 Public 只读权限与 CORS，确保 `uploads` 可通过 https 访问 //by=>friday
  4) 新增“内容服务适配层”导出函数：`getLatestBlogPosts`、`getBlogPostsCount`、`getBlogPost`、`getAllLatestBlogPostSlugs`、`getSettings` //by=>friday
  5) 适配规则：媒体 `url` 统一转绝对地址；`width/height/alternativeText→dimensions/alt`；`categories→string[]`；`_id=String(id)`；`lqip/dominant` 留空 //by=>friday
  6) 在上述 8 个使用点逐一把导入从 `~/sanity/queries` 替换为适配层导出，页面与 UI 组件不改或最小改 //by=>friday
  7) 验证页面、RSS、sitemap、图片加载与 https/CORS，无报错后再移除 Sanity 依赖 //by=>friday

- 正文渲染两案（任选其一） //by=>friday
  - A 最简：`post.body` 存 Markdown，详情页用 `react-markdown` 渲染；`headings` 由 Markdown 解析生成 //by=>friday
  - B 兼容：适配层将 Markdown 转为“简化 PortableText 块”，继续交给现有 `PostPortableText` 渲染 //by=>friday

- 风险点与规避 //by=>friday
  - 文本格式差异（Portable Text vs Markdown）：先走 A 快速上线，必要时再做 B 转换 //by=>friday
  - `lqip/dominant` 缺失：组件已可降级，后续需要再补占位图或主色增强 //by=>friday
  - `related` 关联：在 Strapi 以共享分类查询 Top N，适配层实现 //by=>friday
  - `slug` 全量获取：Strapi REST 分页循环取齐，适配层封装 //by=>friday
  - `readingTime` 差异：作为字段存储或由适配层按 `body` 动态计算 //by=>friday
  - CORS/权限/协议：Strapi 开放只读、放行域名、全站 https，避免 403 与混合内容 //by=>friday

- 验收标准 //by=>friday
  - 首页、列表、详情、友链/项目、sitemap、RSS 全部可用，无前端报错 //by=>friday
  - 所有图片正常加载，页面无混合内容与 CORS 错误 //by=>friday
  - 列表分页、SEO 元信息与主图正确，切换 `CONTENT_PROVIDER` 可一键回退 //by=>friday

### Sanity 替换方案（采用 Strapi 自托管） //by=>friday

#### 目标与原则 //by=>friday
- 彻底放弃 Sanity 及 Studio，改为自托管 Strapi，提供可用的后台网页管理 //by=>friday
- 保持前端展示一致化：文章列表/详情、友链、项目、首页配置与图片均能正常渲染 //by=>friday
- 技术难度最小：优先使用 Strapi 内置能力（集合类型、媒体库、权限/CORS）与简单 REST API，对前端做“服务适配层”最小改造 //by=>friday

#### 数据模型设计（Strapi 内容类型） //by=>friday
- Collection: `post` //by=>friday
  - 字段：`title (string)`、`slug (UID by title)`、`description (text)`、`publishedAt (datetime)`、`readingTime (integer)`、`mood (enumeration)`、`categories (relation many-to-many category)`、`mainImage (media single)`、`body (richtext/markdown)` //by=>friday
- Collection: `category` //by=>friday
  - 字段：`title (string)`、`slug (UID)` //by=>friday
- Collection: `project` //by=>friday
  - 字段：`name (string)`、`url (url)`、`description (text)`、`icon (media single)` //by=>friday
- Collection: `friend` //by=>friday
  - 字段：`name (string)`、`url (url)`、`description (text)`、`email (email)`、`logo (media single)` //by=>friday
- Single Type: `settings` //by=>friday
  - 字段：`projects (relation many project)`、`friends (relation many friend)`、`heroPhotos (media multiple)`、`resume (component repeatable: { company, title, start, end, logo(media) })` //by=>friday

说明：与 Sanity 的 GROQ 结构对齐，尽量提供同名/等价字段；`body` 统一为 Markdown/RichText，避免复杂便携块格式 //by=>friday

#### 图片与媒体策略 //by=>friday
- 使用 Strapi 媒体库本地存储，返回 `url/width/height/alternativeText/caption` 等字段 //by=>friday
- 生成的 `url` 常为相对路径（如 `/uploads/xxx.jpg`），前端需拼接 `STRAPI_BASE_URL` 形成绝对 URL //by=>friday
- 不做自动尺寸/裁剪；前端 `next/image` 已设置 `unoptimized`，按宽高与 CSS 缩放展示即可 //by=>friday
- 可选优化（不增加复杂度）：列表页用 `formats.thumbnail/small`，详情页用原图；CDN 只需把 `/uploads` 反代即可 //by=>friday

#### 权限与 CORS //by=>friday
- 后台设置 Public 角色的读取权限（Post/Category/Project/Friend/Settings/Upload find/findOne） //by=>friday
- CORS 加入博客站点域名与本地调试来源，避免 403 与跨域错误 //by=>friday
- 全站 https，避免混合内容；Strapi 通过反代/证书或由 CDN 提供 https 回源 //by=>friday

#### API 对照（Strapi v4 REST，示例） //by=>friday
- 获取文章列表（分页+筛选+主图）：`GET /api/posts?publicationState=live&filters[publishedAt][$lte]=<ISO>&fields=title,slug,description,publishedAt,readingTime,mood&populate[mainImage]=*&sort=publishedAt:desc&pagination[page]=1&pagination[pageSize]=5` //by=>friday
- 获取文章详情（按 slug）：`GET /api/posts?filters[slug][$eq]=<slug>&populate[mainImage]=*&populate[categories]=*&fields=...` → 取第一项 //by=>friday
- 获取所有 slug：`GET /api/posts?publicationState=live&filters[slug][$null]=false&fields=slug&pagination[pageSize]=1000` //by=>friday
- 获取设置：`GET /api/setting?populate[projects]=*&populate[friends]=*&populate[heroPhotos]=*&populate[resume.logo]=*` //by=>friday

#### 前端“服务适配层”（最小改造点） //by=>friday
- 在文档层面定义将新增一个 `lib/content-service.ts`（仅文档说明，待审批后实现），对外暴露与 `sanity/queries.ts` 等价的函数： //by=>friday
  - `getLatestBlogPosts({ limit, offset, forDisplay })` → 返回 Post 列表（含 `_id/slug/title/description/publishedAt/readingTime/mood/mainImage{asset.url,lqip?,dominant?}`） //by=>friday
  - `getBlogPost(slug)` → 返回 PostDetail（`body` 字段为 Markdown 字符串；`headings/related` 后续按需补齐或省略） //by=>friday
  - `getAllLatestBlogPostSlugs()` → 返回 string[] //by=>friday
  - `getSettings()` → 返回 `projects/friends/heroPhotos/resume` //by=>friday
- 适配逻辑： //by=>friday
  - 将 Strapi 的媒体对象映射为 `{ url: absoluteUrl, dimensions: { width, height }, alt }`；`lqip/dominant` 置空 //by=>friday
  - `categories` 从关系数组提取 `title` 列表 //by=>friday
  - `slug` 直接使用 Strapi UID；`_id` 可用 Strapi `id` 或组合 //by=>friday
  - 统一 `publishedAt` 为 ISO 字符串 //by=>friday

#### 正文渲染策略（简化为 Markdown） //by=>friday
- 将 `post.body` 存储为 Markdown/RichText，前端以 `react-markdown` 渲染（项目已依赖该库） //by=>friday
- 兼容策略：在页面中用“正文渲染组件”替换 `PostPortableText` 的使用点，或在适配层把 Markdown 转成简化块结构再交给现有组件（前者复杂度更低） //by=>friday
- 图片内嵌：Markdown 直接引用绝对 URL，`next/image` 仍可用 `unoptimized` 渲染 //by=>friday

#### 实施步骤 //by=>friday
1) 部署 Strapi：Docker 或 Node 方式；启用本地 PostgreSQL；创建管理员账号 //by=>friday
2) 建立内容类型与关系：按“数据模型设计”创建集合/单例与组件 //by=>friday
3) 开放 Public 读权限与 CORS；上传若干测试图片与内容 //by=>friday
4) 在博客项目添加环境变量 `STRAPI_BASE_URL` 指向 Strapi 服务 //by=>friday
5) 实现内容服务适配层：封装上文四个函数，完成字段映射与图片绝对 URL 构造 //by=>friday
6) 逐页替换导入：将 `~/sanity/queries` 的调用替换为适配层导出（`page.tsx`、`BlogPosts.tsx`、`sitemap.ts` 等） //by=>friday
7) 验证：主页与列表、详情页、友链/项目、首页配置、站点地图；图片加载与 https；无跨域错误 //by=>friday
8) 下线 Sanity：移除 `sanity/*` 与相关依赖（在完成回归后） //by=>friday

#### 验证清单 //by=>friday
- `/` 首屏 heroPhotos 与最新文章卡片图片展示正常 //by=>friday
- `/blog` 列表分页正确；主图与数据一致 //by=>friday
- `/blog/[slug]` 正文 Markdown 可渲染，文中图片可放大/查看（保留现有交互或最简展示） //by=>friday
- `/friends`、`/projects` 从 `settings` 读取并展示；logo 图片正常 //by=>friday
- `/sitemap.xml` slugs 正确生成；`/robots.txt` 无变化 //by=>friday

#### 风险与回退 //by=>friday
- 若 Markdown 渲染替换范围较大，可先在适配层输出与现组件兼容的简化块结构，逐步替换页面渲染组件 //by=>friday
- 任一关键页面异常时，可临时回退到 `sanity/queries`，两套实现并存由环境变量开关控制 //by=>friday

### 数据库本地化实施方案（Neon → 本地 PostgreSQL） //by=>friday

#### 核心目标 //by=>friday
- 将 `env.DATABASE_URL` 指向本地 PostgreSQL，替换掉 `@neondatabase/serverless` 驱动，保持 Drizzle ORM 接口不变，保证所有用到 `~/db` 的接口与页面无感迁移 //by=>friday

#### 涉及文件与现状 //by=>friday
- `db/index.ts`：当前使用 `@neondatabase/serverless` 与 `drizzle-orm/neon-serverless` 建立连接 //by=>friday
- `drizzle.config.ts`：driver 配置为 `pg`，`dbCredentials.connectionString` 读取 `DATABASE_URL` //by=>friday
- 依赖 `db` 的位置（示例）：`app/api/guestbook/route.ts`、`app/api/comments/[id]/route.ts`、`app/api/newsletter/route.ts`、`app/admin/*` 等 //by=>friday
- 表定义与迁移：`db/schema.ts`、`db/migrations/*` 已存在，结构为标准 PostgreSQL //by=>friday

#### 本地 PostgreSQL 准备 //by=>friday
- 安装 PostgreSQL（建议 14+）；创建数据库与用户 //by=>friday
- 连接信息建议：`postgres://<user>:<password>@localhost:5432/<db>` //by=>friday
- `.env` 示例：`DATABASE_URL=postgres://jcblog_user:your_password@localhost:5432/jcblog`（本地通常无需 SSL） //by=>friday

#### 依赖调整 //by=>friday
- 新增依赖：`pg`（Node-Postgres 客户端） //by=>friday
- 暂不删除：`@neondatabase/serverless`（待迁移验证通过后再移除，方便回退） //by=>friday

#### 代码替换清单（待审批后执行） //by=>friday
- `db/index.ts` 改为基于 Node-Postgres 的 Drizzle 适配： //by=>friday
  - `import { Pool } from 'pg'` //by=>friday
  - `import { drizzle } from 'drizzle-orm/node-postgres'` //by=>friday
  - `const pool = new Pool({ connectionString: env.DATABASE_URL })` //by=>friday
  - `export const db = drizzle(pool)` //by=>friday
- 其余依赖 `db` 的代码无需修改（调用层保持不变） //by=>friday

#### 迁移与初始化数据 //by=>friday
- 迁移执行：`pnpm db:push`（`drizzle-kit push:pg`）或 `pnpm db:generate` 后再 `pnpm db:push` //by=>friday
- 从 Neon 导入历史数据（可选）： //by=>friday
  - 导出：`pg_dump "<NEON_DATABASE_URL>" -Fc -f neon.dump` //by=>friday
  - 恢复：`pg_restore -d jcblog -c neon.dump` //by=>friday

#### 验证清单 //by=>friday
- Guestbook：页面加载与 `POST /api/guestbook` 写入正常 //by=>friday
- Comments：`GET/POST /api/comments/[id]` 正常（至少冒烟） //by=>friday
- Newsletter：订阅者/简报列表页与创建功能正常 //by=>friday
- 管理台：`/app/admin/*` 涉及数据库读取的页面能渲染 //by=>friday
- 时间与时区：`created_at/updated_at` 与预期一致（必要时统一使用 UTC） //by=>friday

#### 运行时与性能参数（单机建议） //by=>friday
- 连接池大小：`max: 10~20` 即可；如使用 PM2 多进程，注意总连接不要超过本地 PG 限制 //by=>friday
- 空闲连接与超时：可设置 `idleTimeoutMillis`/`statement_timeout` 规避长时间阻塞 //by=>friday

#### 回退策略 //by=>friday
- 保留 Neon 与本地两套连接方式的切换能力（通过切换 `DATABASE_URL` 与恢复 `db/index.ts` 的导入实现） //by=>friday
- 待本地验证完成后，再移除 `@neondatabase/serverless` 依赖 //by=>friday

#### 已知风险与处理 //by=>friday
- SQL 兼容：当前表结构为标准 PG，无特定 Neon 扩展，可直接兼容 //by=>friday
- 权限与网络：本地连接无需 SSL；如容器化或跨主机连接再考虑 SSL 与白名单 //by=>friday
- 并发与一致性：单机场景足够；如多进程并发明显提升，可增加连接池或引入 PGBouncer //by=>friday

### Next.js 本地部署实施方案（替代 Vercel） //by=>friday

#### 目标与范围 //by=>friday
- 目标：在单台服务器以 Node 方式运行本项目，替代 Vercel 平台能力，确保现有功能可用，为后续 Sanity/Neon/Upstash 本地化改造打基础 //by=>friday
- 范围：仅涉及 Next.js 的构建与运行方式变更，不在此阶段修改业务代码逻辑；运行时统一策略在此描述，实际代码改动将单独审批后执行 //by=>friday

#### 前置条件 //by=>friday
- 系统：Linux（推荐 Debian/Ubuntu/CentOS 任一），也可 Windows Server //by=>friday
- Node：>= 18（与 Next 14 兼容） //by=>friday
- 包管理器：pnpm 已安装 //by=>friday
- 网络：服务器能访问外网以安装依赖与拉取远程资源 //by=>friday

#### 环境变量准备 //by=>friday
- 按 `env.mjs` 校验准备 `.env` 文件；此阶段暂沿用线上所需变量（Sanity/Upstash/IndexNow 等），待后续逐项本地化替换 //by=>friday
- 最小可运行建议变量（示例，具体值按现网配置）： //by=>friday
  - NODE_ENV=production //by=>friday
  - NEXT_PUBLIC_SITE_URL=https://你的域名 或 http://服务器IP:端口 //by=>friday
  - NEXT_PUBLIC_SANITY_PROJECT_ID、NEXT_PUBLIC_SANITY_DATASET、NEXT_PUBLIC_SANITY_USE_CDN（在 Sanity 替换前临时保留） //by=>friday
  - DATABASE_URL（在 Neon 替换前仍可指向 Neon；将来切本地 Postgres） //by=>friday
  - UPSTASH_REDIS_REST_URL、UPSTASH_REDIS_REST_TOKEN（在 Redis 替换前临时保留） //by=>friday

#### 运行时统一策略（从 edge 迁移到 nodejs） //by=>friday
- 当前声明为 `edge` 的文件：`app/robots.ts`、`app/sitemap.ts`、`app/api/favicon/route.tsx`、`app/api/reactions/route.ts`、`app/api/link-preview/route.tsx`、`app/api/activity/route.ts` //by=>friday
- 策略：统一改为 `nodejs` 以减少平台差异；如 `next/og` 在 Node 下不完全等价，则临时为个别路由保留 `edge` 或做功能降级（详见后文“验证与回退”） //by=>friday
- 注意：本节仅记录策略与影响，具体代码 edits 将在你确认后逐处提交并标注 `//by=>friday` //by=>friday

#### 构建与启动 //by=>friday
1) 安装依赖：`pnpm install` //by=>friday
2) 生产构建：`pnpm build`（首次构建关注是否有 edge-only 报错） //by=>friday
3) 启动服务：`pnpm start`（默认 3000 端口，可用 `PORT=xxxx pnpm start` 指定端口） //by=>friday
4) 进程守护（二选一）： //by=>friday
   - 使用 PM2：`pm2 start npm --name jcblog -- start`，并 `pm2 save`、`pm2 startup` //by=>friday
   - 使用 systemd：创建 unit 文件，设置 `ExecStart="pnpm start"`、`WorkingDirectory` 为项目根目录 //by=>friday

#### 反向代理与 HTTPS（可选但推荐） //by=>friday
- 使用 Nginx 将 80/443 转发至本地 3000： //by=>friday
  - proxy_set_header Host $host; proxy_set_header X-Real-IP $remote_addr; proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; //by=>friday
  - 配置缓存/超时：图片与静态资源可适度缓存；API 保持 no-store //by=>friday
  - 申请与自动续期证书（Let’s Encrypt/Certbot） //by=>friday

#### 验证清单（功能与性能） //by=>friday
- 路由与页面：`/`、`/blog`、`/blog/[slug]`、`/guestbook`、`/friends` //by=>friday
- SEO：`/sitemap.xml`、`/robots.txt` 正常返回 //by=>friday
- API： //by=>friday
  - `/api/activity`、`/api/reactions`（限流与计数依赖 Upstash，先验证可用性） //by=>friday
  - `/api/link-preview`、`/api/favicon`（基于 `next/og` 的图片响应，重点验证在 nodejs 运行时的行为一致性） //by=>friday
  - 其他接口如 Newsletter、IndexNow、Guestbook 读写冒烟测试 //by=>friday
- 构建日志：无 edge-only API 报错；运行日志无致命错误 //by=>friday

#### 常见问题与处理 //by=>friday
- `next/og` 在 nodejs 下不兼容：为该路由局部保留 `edge` 或改为服务端生成二进制图片/降级到外部快照服务 //by=>friday
- 资源跨域/远程图片：确保 `next.config.mjs` 中 `images.remotePatterns` 覆盖所需域名（`cdn.sanity.io`、`youke1.picui.cn`、`gitee.com`） //by=>friday
- 环境变量校验失败：参考 `env.mjs` 的必填项，逐项补齐或在本地化阶段调整依赖 //by=>friday
- 端口与防火墙：放通反代回源端口（如 3000），并在安全组/防火墙开放 80/443 //by=>friday

#### 回退与最小变更策略 //by=>friday
- 若统一改为 `nodejs` 后出现不可接受的问题： //by=>friday
  - 临时仅对 `link-preview`、`favicon` 两处保留 `edge`（最小范围），其余保持 `nodejs` //by=>friday
  - 或对上述两处做功能降级（例如返回占位图或第三方截图服务），完全去除 edge 依赖 //by=>friday
- 保留原有 `package.json` 脚本，回滚到 dev 开发或在本地模拟生产 `pnpm start` 即可 //by=>friday

#### 后续工作衔接（非本步骤执行，但需规划） //by=>friday
- Sanity → 本地内容源（MDX/JSON 过渡或直上 Postgres），提供兼容查询的“内容服务层” //by=>friday
- Neon → 本地 PostgreSQL（切换 `db/index.ts` 驱动为 node-postgres 适配，迁移与数据校验） //by=>friday
- Upstash Redis → 本地 Redis（替换客户端与限流实现，保证 API 兼容） //by=>friday

### 图片与媒体（切换为腾讯云 COS/CDN） //by=>friday

- 目标：统一将站内所有图片（文章主图、友链 Logo、首页 Photos、简历 Logo、富文本内联图）托管到腾讯云 COS，并使用 CDN 加速域名对外访问。页面与组件不做结构性调整，仅更换图片来源与白名单。 //by=>friday

- 准备工作 //by=>friday
  - 创建 COS 存储桶：区域优先选国内近源；访问策略“公有读”。 //by=>friday
  - 绑定 CDN 自定义域名（推荐）：开启 HTTPS，申请证书，命中缓存策略按图片类型设置长缓存（可按需回源刷新）。 //by=>friday
  - 确认最终对外访问域名：如 `img.example.com`（CDN）或 `bucket-xxx.cos.ap-xxx.myqcloud.com`（直链）。 //by=>friday

- Next 配置（仅放行域名，代码层不做其他改造） //by=>friday
  - 在 `next.config.mjs` 的 `images.remotePatterns` 添加 COS/CDN 域名，保留现有白名单以便平滑迁移： //by=>friday
  - 示例（实际 edits 将按你的最终域名填写）： //by=>friday
    - `{ protocol: 'https', hostname: 'img.example.com', port: '', pathname: '/**' }` //by=>friday
    - 或 `{ protocol: 'https', hostname: 'bucket-xxx.cos.ap-xxx.myqcloud.com', port: '', pathname: '/**' }` //by=>friday

- 内容侧约定 //by=>friday
  - 所有新图片统一使用“绝对 URL（以 COS/CDN 域名开头）”；Sanity/Strapi/友链数据中直接存储该绝对地址，避免相对路径导致混合内容。 //by=>friday
  - 历史内容迁移可分期进行：短期同时保留 `cdn.sanity.io/youke1.picui.cn/gitee.com` 白名单；迁移完成再下线旧域名。 //by=>friday
  - 可选环境变量（便于将来扩展）：`COS_PUBLIC_BASE_URL=https://img.example.com`（仅文档约定，暂不在代码中使用）。 //by=>friday

- 影响面与验证 //by=>friday
  - 页面：`/`、`/blog`、`/blog/[slug]`、`/friends`、`/guestbook` 均应以 COS 域名加载图片，无 403/混合内容。 //by=>friday
  - RSS/站点地图：不直接受影响，但主图 URL 将展示为 COS/CDN 域名。 //by=>friday
  - 性能：命中 CDN 后首屏图片应显著提速；日志监测回源率与带宽。 //by=>friday

- 风险与回退 //by=>friday
  - 若发现区域访问异常，可临时切回旧域名（白名单仍在）；或在 CDN 侧切换回源。 //by=>friday
  - 若 HTTPS/证书异常，先停用强制 HTTPS，恢复后再开启 HSTS。 //by=>friday

- 可选（需审批后再做，当前不执行） //by=>friday
  - 新增 `lib/media.ts` 的 `toCosUrl(url)` 工具，统一把相对路径前置 `COS_PUBLIC_BASE_URL`；或做历史 URL 一次性替换脚本。 //by=>friday