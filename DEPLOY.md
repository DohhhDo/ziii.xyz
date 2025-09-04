# 🚀 服务器部署指导 - 阶段A成果

> 本指导用于在性能孱弱的服务器上部署已构建的Next.js应用，无需服务器端构建

## 📦 准备部署文件

### 1. 本地构建完成的文件
已在本地完成构建，需要上传到服务器的文件：

**必需文件和目录：**
```
├── .next/              # 构建产物（完整目录）
├── public/             # 静态资源
├── package.json        # 依赖定义
├── next.config.mjs     # Next.js配置
└── .env.local         # 环境变量（需手动创建）
```

**可选文件：**
```
├── node_modules/       # 如果服务器网络差，可以一起上传
└── pnpm-lock.yaml     # 锁定版本文件
```

## 🔧 服务器操作步骤

### 1. 环境准备
```bash
# 确保Node.js版本 >= 18
node --version

# 安装pnpm（如果未安装）
npm install -g pnpm
```

### 2. 上传文件
将本地构建的文件上传到服务器目录

### 3. 安装生产依赖
```bash
# 进入项目目录
cd /path/to/your/project

# 仅安装生产依赖（如果没上传node_modules）
pnpm install --prod
```

### 4. 配置环境变量
创建 `.env.local` 文件：
```env
# 根据你的实际配置填写
NODE_ENV=production
DATABASE_URL=your_database_url
RESEND_API_KEY=your_resend_key
UPSTASH_REDIS_REST_URL=your_redis_url
UPSTASH_REDIS_REST_TOKEN=your_redis_token
NEXT_PUBLIC_SANITY_PROJECT_ID=your_sanity_project
NEXT_PUBLIC_SANITY_DATASET=production
NEXT_PUBLIC_SANITY_USE_CDN=true
NEXT_PUBLIC_SITE_URL=your_site_url
NEXT_PUBLIC_SITE_EMAIL_FROM=your_email
INDEXNOW_KEY=your_indexnow_key
BAIDU_AIP_CLIENT_ID=your_baidu_client_id
BAIDU_AIP_CLIENT_SECRET=your_baidu_client_secret
```

### 5. 启动应用
```bash
# 启动生产服务器
pnpm start

# 或者使用PM2管理进程
pm2 start "pnpm start" --name "ziii-blog"

# 或者使用nohup后台运行
nohup pnpm start > app.log 2>&1 &
```

## ✅ 验证清单

部署后访问以下页面确认功能正常：

- [ ] 主页 `/` - 正常加载
- [ ] 博客页 `/blog` - 文章列表显示
- [ ] 友链页 `/friends` - 项目和友链显示  
- [ ] 留言板 `/guestbook` - 可以查看留言
- [ ] API接口 `/api/activity` - 返回JSON
- [ ] API接口 `/api/songci` - 返回诗词
- [ ] 站点地图 `/sitemap.xml` - XML格式正确
- [ ] RSS订阅 `/feed.xml` - XML格式正确

## 🔍 性能监控

### 服务器资源使用
```bash
# 查看内存使用
free -h

# 查看CPU使用  
top

# 查看Node.js进程
ps aux | grep node
```

### 应用日志
```bash
# 查看应用日志
tail -f app.log

# 或PM2日志
pm2 logs ziii-blog
```

## 🚨 故障排除

### 常见问题

1. **端口占用**
   ```bash
   # 查看端口使用
   netstat -tulpn | grep :3000
   
   # 杀死占用进程
   kill -9 <PID>
   ```

2. **环境变量错误**
   - 检查 `.env.local` 文件是否存在
   - 确认所有必需的环境变量都已配置

3. **数据库连接失败**
   - 检查 `DATABASE_URL` 是否正确
   - 确认数据库服务是否正常运行

4. **Redis连接失败**
   - 检查 `UPSTASH_REDIS_REST_URL` 和 `UPSTASH_REDIS_REST_TOKEN`
   - 确认网络可以访问Redis服务

## 📈 阶段A完成状态

**✅ 已完成：**
- 运行时统一为Node.js（保留2处图像处理为edge）
- 本地开发服务器验证通过
- 本地构建成功完成
- 服务器部署包已准备

**⚠️ 已知问题：**
- LazyTremor组件在构建时有警告（不影响运行）
- favicon API需要绝对URL（已在预期范围内）

---

*生成时间：2025-09-04*
*对应阶段：A - Next本地运行与运行时统一*
