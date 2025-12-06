# 个人博客架构设计方案

## 一、技术栈架构

### 1.1 核心技术
- **框架**: Astro 5.x (静态站点生成器)
- **样式**: Tailwind CSS 3.x (原子化CSS)
- **语言**: TypeScript (类型安全)
- **包管理**: pnpm (高效依赖管理)
- **UI组件**: React 19.x (交互组件)
- **国际化**: astro-i18n-aut (多语言支持)

### 1.2 项目结构
```
kankan-blog/
├── public/                    # 静态资源
│   ├── images/               # 图片资源
│   ├── favicon.svg
│   └── robots.txt
├── src/
│   ├── assets/               # 资源文件
│   ├── components/           # 组件库
│   │   ├── common/          # 通用组件
│   │   │   ├── Header.astro
│   │   │   ├── Footer.astro
│   │   │   ├── Navigation.astro
│   │   │   ├── ThemeToggle.tsx
│   │   │   └── LanguageSwitcher.tsx
│   │   ├── posts/           # 文章相关组件
│   │   │   ├── PostCard.astro
│   │   │   ├── PostList.astro
│   │   │   └── PostContent.astro
│   │   ├── notes/           # 笔记相关组件
│   │   ├── inbox/           # 收集箱组件
│   │   │   └── ChatMessage.astro
│   │   ├── projects/        # 项目组件
│   │   ├── timeline/        # 时间轴组件
│   │   └── friends/        # 友链组件
│   ├── content/             # 内容管理
│   │   ├── posts/          # 文章 Markdown
│   │   ├── notes/          # 笔记 Markdown
│   │   ├── projects/       # 项目数据 (从 GitHub Issue 获取)
│   │   ├── timeline/       # 时间轴数据
│   │   ├── friends/        # 友链数据
│   │   └── navigation/     # 导航站数据
│   ├── content/config.ts   # Content Collections 配置
│   ├── layouts/            # 布局组件
│   │   ├── Layout.astro    # 基础布局
│   │   ├── PostLayout.astro
│   │   └── PageLayout.astro
│   ├── pages/              # 路由页面
│   │   ├── index.astro     # 首页
│   │   ├── about.astro     # 关于/简历
│   │   ├── posts/
│   │   │   ├── index.astro # 文章列表
│   │   │   └── [slug].astro # 文章详情
│   │   ├── notes/
│   │   │   ├── index.astro
│   │   │   └── [slug].astro
│   │   ├── inbox.astro     # 收集箱
│   │   ├── navigation.astro # 导航站
│   │   ├── timeline.astro  # 时间轴
│   │   ├── projects.astro  # 项目
│   │   └── friends.astro   # 友链
│   ├── utils/              # 工具函数
│   │   ├── date.ts
│   │   ├── seo.ts
│   │   ├── rss.ts
│   │   └── github.ts       # GitHub API 集成
│   ├── styles/             # 全局样式
│   │   └── global.css
│   ├── i18n/               # 国际化配置
│   │   ├── locales/
│   │   │   ├── zh.json
│   │   │   └── en.json
│   │   └── config.ts
│   └── types/              # TypeScript 类型定义
│       └── content.ts
├── astro.config.mjs        # Astro 配置
├── tailwind.config.mjs     # Tailwind 配置
├── tsconfig.json
└── package.json
```

## 二、内容类型设计

### 2.1 文章 (Posts)
- **存储**: `src/content/posts/*.md` (Markdown 文件)
- **Frontmatter 结构**:
  ```yaml
  title: 文章标题
  description: 文章描述
  pubDate: 2024-01-01
  updatedDate: 2024-01-02
  category: 分类
  tags: [标签1, 标签2]
  featured: true  # 是否置顶
  lang: zh | en
  ```
- **路由**: `/posts` (列表), `/posts/[slug]` (详情)

### 2.2 笔记 (Notes)
- **存储**: `src/content/notes/*.md` (Markdown 文件)
- **Frontmatter**: 类似文章，但更简洁
- **路由**: `/notes` (列表), `/notes/[slug]` (详情)

### 2.3 收集箱 (Inbox)
- **存储**: `src/content/inbox.json` (JSON 格式)
- **数据结构**:
  ```typescript
  interface InboxItem {
    id: string
    date: string
    type: 'text' | 'link' | 'image' | 'quote'
    content: string
    metadata?: {
      url?: string
      image?: string
      author?: string
    }
  }
  ```
- **路由**: `/inbox`
- **UI**: 微信聊天风格，时间分组显示

### 2.4 在线简历 (About)
- **存储**: `src/content/about.json` 或直接写在页面
- **内容**: 教育经历、工作经验、技能、社交链接
- **路由**: `/about`

### 2.5 导航站 (Navigation)
- **存储**: `src/content/navigation.json`
- **数据结构**:
  ```typescript
  interface NavItem {
    category: string
    items: Array<{
      name: string
      url: string
      icon?: string
      description?: string
    }>
  }
  ```
- **路由**: `/navigation`

### 2.6 时间轴 (Timeline)
- **存储**: `src/content/timeline.json`
- **数据结构**:
  ```typescript
  interface TimelineEvent {
    date: string
    title: string
    description: string
    type: 'personal' | 'work' | 'project'
    icon?: string
  }
  ```
- **路由**: `/timeline`

### 2.7 项目 (Projects)
- **存储**: 从 GitHub Issue 获取 (通过 GitHub API)
- **数据来源**: GitHub Issue，内容编写在 Issue 中
- **数据结构**:
  ```typescript
  interface Project {
    name: string
    description: string
    github?: string
    demo?: string
    tags: string[]
    status: 'active' | 'archived' | 'completed'
    stars?: number  # 从 GitHub API 获取
    featured: boolean
    issueNumber?: number  # GitHub Issue 编号
  }
  ```
- **路由**: `/projects`
- **功能**: 展示 GitHub stars、标签、状态，从 Issue 获取详细信息

### 2.8 友链 (Friends)
- **存储**: `src/content/friends.json`
- **数据结构**:
  ```typescript
  interface Friend {
    name: string
    url: string
    avatar: string
    description: string
    tags?: string[]
  }
  ```
- **路由**: `/friends`

## 三、核心功能实现

### 3.1 路由设计
- **文件系统路由**: 利用 Astro 的文件系统路由
- **动态路由**: `[slug].astro` 用于文章/笔记详情
- **路由结构**:
  ```
  /              → 首页
  /about         → 简历
  /posts         → 文章列表
  /posts/[slug]  → 文章详情
  /notes         → 笔记列表
  /notes/[slug]  → 笔记详情
  /inbox         → 收集箱
  /navigation    → 导航站
  /timeline      → 时间轴
  /projects      → 项目
  /friends       → 友链
  ```

### 3.2 国际化 (i18n)
- **集成**: `astro-i18n-aut`
- **配置**: `src/i18n/config.ts`
- **语言文件**: `src/i18n/locales/zh.json`, `en.json`
- **实现**:
  - 自动检测浏览器语言
  - 语言切换组件
  - URL 前缀支持 (`/zh/`, `/en/`)
  - 内容多语言支持

### 3.3 响应式设计
- **Tailwind 断点**:
  - `sm`: 640px
  - `md`: 768px
  - `lg`: 1024px
  - `xl`: 1280px
- **移动优先**: 所有组件采用移动优先设计
- **布局**: Flexbox/Grid 实现自适应
- **左右结构**: 左侧展示导航栏，右侧展示具体内容信息

### 3.4 SEO 优化
- **Meta 标签**: 使用 Astro 的 `<Head>` 组件
- **Open Graph**: 社交媒体分享优化
- **结构化数据**: JSON-LD 格式
- **站点地图**: 自动生成 `sitemap.xml`
- **robots.txt**: 配置搜索引擎爬虫规则
- **工具函数**: `src/utils/seo.ts`

### 3.5 RSS 订阅
- **生成**: `src/utils/rss.ts`
- **路由**: `/rss.xml`
- **内容**: 包含所有文章和笔记
- **格式**: RSS 2.0 标准
- **自动更新**: 构建时生成

### 3.6 深色模式
- **实现方式**: CSS 变量 + Tailwind dark mode
- **检测**: 系统主题检测 (`prefers-color-scheme`)
- **切换**: 手动切换按钮 (ThemeToggle 组件)
- **持久化**: localStorage 存储用户偏好
- **配置**: `tailwind.config.mjs` 启用 `darkMode: 'class'`

## 四、技术实现细节

### 4.1 Content Collections
使用 Astro 的 Content Collections 管理文章和笔记:
```typescript
// src/content/config.ts
import { defineCollection, z } from 'astro:content';

const postsCollection = defineCollection({
  type: 'content',
  schema: z.object({
    title: z.string(),
    description: z.string(),
    pubDate: z.date(),
    updatedDate: z.date().optional(),
    category: z.string(),
    tags: z.array(z.string()),
    featured: z.boolean().default(false),
    lang: z.enum(['zh', 'en']).default('zh'),
  }),
});

const notesCollection = defineCollection({
  type: 'content',
  schema: z.object({
    title: z.string(),
    description: z.string().optional(),
    pubDate: z.date(),
    updatedDate: z.date().optional(),
    tags: z.array(z.string()).optional(),
    lang: z.enum(['zh', 'en']).default('zh'),
  }),
});

export const collections = {
  posts: postsCollection,
  notes: notesCollection,
};
```

### 4.2 GitHub API 集成
项目信息从 GitHub Issue 获取:
```typescript
// src/utils/github.ts
interface GitHubIssue {
  number: number
  title: string
  body: string
  labels: Array<{ name: string }>
  state: 'open' | 'closed'
  created_at: string
  updated_at: string
}

// 从 GitHub API 获取 Issue 列表
// 解析 Issue body 中的项目信息（使用 frontmatter 或特定格式）
// 获取仓库的 stars 数量
```

### 4.3 组件设计原则
- **Astro 组件**: 静态内容、布局
- **React 组件**: 交互功能 (主题切换、语言切换)
- **可复用性**: 通用组件抽离
- **类型安全**: 所有 Props 定义 TypeScript 类型

### 4.4 样式系统
- **Tailwind 配置**: 自定义主题色、字体
- **CSS 变量**: 深色模式颜色变量
- **组件样式**: 使用 Tailwind 工具类
- **响应式**: 移动优先的响应式设计
- **布局**: 左右分栏布局，左侧导航，右侧内容

### 4.5 性能优化
- **图片优化**: Astro Image 组件
- **代码分割**: 自动代码分割
- **预加载**: 关键资源预加载
- **静态生成**: 所有页面静态生成

## 五、开发流程

### 5.1 依赖安装
```bash
pnpm add @astrojs/tailwind tailwindcss astro-i18n-aut @astrojs/sitemap
pnpm add -D @types/node
```

### 5.2 配置文件
- `astro.config.mjs`: 集成 Tailwind、i18n、sitemap
- `tailwind.config.mjs`: 主题配置、深色模式
- `tsconfig.json`: TypeScript 严格模式

### 5.3 开发顺序
1. 项目基础配置 (Tailwind、i18n、TypeScript)
2. 布局组件 (Header、Footer、Navigation)
3. 首页和关于页
4. 文章系统 (列表、详情)
5. 笔记系统
6. 其他内容页面 (收集箱、导航站等)
7. GitHub API 集成 (项目信息)
8. 功能增强 (SEO、RSS、深色模式)
9. 优化和测试

## 六、部署建议

- **平台**: Vercel、Netlify、Cloudflare Pages
- **构建**: `pnpm build`
- **环境变量**: GitHub API token (用于获取项目信息)
- **域名**: 配置自定义域名
- **CDN**: 利用平台 CDN 加速

## 七、扩展功能 (可选)

- **评论系统**: 集成 Giscus 或 Utterances
- **搜索功能**: 客户端搜索或 Algolia
- **分析**: Google Analytics 或 Plausible
- **PWA**: 渐进式 Web 应用支持

## 八、数据存储说明

### 8.1 内容存储
- **文章**: Markdown 文件存储在 `src/content/posts/`
- **笔记**: Markdown 文件存储在 `src/content/notes/`
- **收集箱**: JSON 文件存储在 `src/content/inbox.json`
- **其他数据**: JSON 文件存储在对应的 `src/content/` 子目录

### 8.2 GitHub 集成
- **项目信息**: 从 GitHub Issue 获取
- **Issue 格式**: 在 Issue body 中使用 frontmatter 或特定格式标记项目信息
- **API 调用**: 构建时或运行时通过 GitHub API 获取数据
- **缓存策略**: 考虑使用静态生成时获取数据，避免运行时 API 调用

