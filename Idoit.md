### Idoit 当前进展与问题记录

> 本文用于同步当下完成情况、已知问题、风险与下一步计划，配合 `Justdoit.md` 执行。更新时间：2025-09-03

---

## 概览
- **总体状态**: 准备阶段（尚未正式执行各阶段任务）
- **范围**: 覆盖 `Justdoit.md` 中阶段 A/B/C/D 的最小闭环执行
- **环境依赖**: Node >= 18、pnpm、`.env` 关键项已配置（待核验）、可访问 Sanity/Upstash/Neon 或本地等价服务（待核验）

---

## 完成情况（按阶段）

### 阶段A：Next 本地运行与运行时统一（最快落地）
- A1. 统一运行时为 node（保留 2 处图像为 edge）: 未开始
- A2. 验证页面与接口（含 next/og、图片白名单、CORS）: 未开始

### 阶段B：数据库本地化（Neon → 本地 PostgreSQL）
- B1. 切换 Drizzle 适配为 node-postgres（仅 `db/index.ts`）: 未开始
- B2. 迁移与冒烟（`pnpm db:push`、留言/评论/简报 CRUD）: 未开始

### 阶段C：Sanity → Strapi（服务适配层最小改造）
- C1. 部署与建模（post/category/project/friend/settings）: 未开始
- C2. 新增“内容服务适配层” `lib/content-service.ts`: 未开始（需审批）
- C3. 替换 8 处 Sanity 导入为适配层导出: 未开始
- C4. 正文渲染策略：建议选方案A（react-markdown）: 未开始
- C5. 图片、SEO、RSS、sitemap 回归验证: 未开始

### 阶段D：图片托管切换至腾讯云 COS（最小变更）
- D1. 准备与域名/CDN/HTTPS: 未开始
- D2. 前端白名单：更新 `next.config.mjs images.remotePatterns`: 未开始（需 1 处 edits）
- D3. 内容侧切换（使用 COS/CDN 绝对 URL）: 未开始
- D4. 冒烟与监控（命中率/403/Referer/HTTPS）: 未开始
- D5. 回退与收尾（白名单/历史迁移）: 未开始

---

## 当前问题
- 暂无可复现问题记录。待完成「构建体检」与「A1/A2 冒烟」后补充。

---

## 风险与阻塞
- 运行时切换兼容性：`next/og` 在 node 环境下的兼容与性能风险。
- 远程图片域名：未放行将导致加载失败（需维护 `images.remotePatterns`）。
- 数据库本地化：本地 PostgreSQL 启动/权限/SSL/连接池参数不当可能阻塞。
- 外部依赖可达性：Sanity/Upstash/Neon 若不可达需立即启用本地等价方案。

---

## 待决与需批准项
- 新增文件：`lib/content-service.ts`（C2）。
- 配置变更：`next.config.mjs` 新增 COS/CDN 域名白名单（D2）。
- 内容源切换策略：通过 `CONTENT_PROVIDER`（sanity/strapi）环境开关实现双栈可回退。

---

## 近期计划（下一最小闭环）
1) 执行 A1：统一运行时为 node（保留 2 处 edge），逐文件修改并可回退。
2) 执行 A2：本地 `pnpm dev` 验证关键页面与 API，完善 `images.remotePatterns` 与 CORS。
3) 记录构建与运行日志，沉淀首轮问题与回退方案。

---

## 构建体检记录
- 命令：`pnpm install && pnpm build`
- 结果：未执行
- 错误/警告：暂无

---

## 回退锚点（便于快速止血）
- 运行时：逐路由 `edge/nodejs` 行级回退。
- 数据库：仅 `db/index.ts` 一处在 Neon/node-postgres 间切换。
- 内容源：`CONTENT_PROVIDER` 环境开关（sanity/strapi）。

---

## 每日记录（滚动追加）

### 2025-09-03
- 完成：整理执行指导单与状态板，待启动 A1/A2。
- 问题：无。
- 计划：按「近期计划」推进 A1→A2，完成后更新本记录。


