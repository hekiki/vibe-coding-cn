# 实战案例：个人博客系统

> 难度：⭐⭐ 中等 | 预计时间：2-4 小时 | 技术栈：Next.js + MDX

## 🎯 项目目标

构建一个支持 Markdown 的个人博客系统，具备：
- 文章列表和详情页
- 标签分类
- 代码高亮
- 响应式设计

## 📋 开始前的准备

### 环境要求
- Node.js 18+
- 包管理器（npm/pnpm/yarn）

### 第一步：需求澄清

复制以下提示词给 AI：

```
我想用 Vibe Coding 的方式开发一个个人博客系统。

技术要求：
- 框架：Next.js 14 (App Router)
- 内容：MDX 格式的 Markdown 文章
- 样式：Tailwind CSS
- 部署：Vercel

功能需求：
1. 首页显示文章列表（标题、日期、摘要）
2. 文章详情页（支持代码高亮）
3. 标签页面（按标签筛选文章）
4. 关于页面

请帮我：
1. 确认技术栈是否合适
2. 生成项目结构
3. 一步步指导我完成开发

要求：每完成一步问我是否成功，再继续下一步。
```

## 🏗️ 项目结构

```
my-blog/
├── app/
│   ├── layout.tsx          # 根布局
│   ├── page.tsx            # 首页
│   ├── posts/
│   │   └── [slug]/
│   │       └── page.tsx    # 文章详情
│   ├── tags/
│   │   └── [tag]/
│   │       └── page.tsx    # 标签页
│   └── about/
│       └── page.tsx        # 关于页
├── components/
│   ├── Header.tsx
│   ├── Footer.tsx
│   ├── PostCard.tsx
│   └── MDXContent.tsx
├── content/
│   └── posts/              # MDX 文章
│       ├── hello-world.mdx
│       └── vibe-coding.mdx
├── lib/
│   └── posts.ts            # 文章处理逻辑
└── tailwind.config.js
```

## 🔧 核心代码片段

### 文章处理逻辑 (lib/posts.ts)

```typescript
import fs from 'fs'
import path from 'path'
import matter from 'gray-matter'

const postsDirectory = path.join(process.cwd(), 'content/posts')

export interface Post {
  slug: string
  title: string
  date: string
  tags: string[]
  excerpt: string
  content: string
}

export function getAllPosts(): Post[] {
  const fileNames = fs.readdirSync(postsDirectory)
  const posts = fileNames
    .filter(name => name.endsWith('.mdx'))
    .map(fileName => {
      const slug = fileName.replace(/\.mdx$/, '')
      const fullPath = path.join(postsDirectory, fileName)
      const fileContents = fs.readFileSync(fullPath, 'utf8')
      const { data, content } = matter(fileContents)
      
      return {
        slug,
        title: data.title,
        date: data.date,
        tags: data.tags || [],
        excerpt: data.excerpt || content.slice(0, 150),
        content
      }
    })
    .sort((a, b) => (a.date < b.date ? 1 : -1))
  
  return posts
}

export function getPostBySlug(slug: string): Post | undefined {
  return getAllPosts().find(post => post.slug === slug)
}
```

### 文章卡片组件 (components/PostCard.tsx)

```tsx
import Link from 'next/link'
import { Post } from '@/lib/posts'

export function PostCard({ post }: { post: Post }) {
  return (
    <article className="border-b border-gray-200 py-6">
      <Link href={`/posts/${post.slug}`}>
        <h2 className="text-xl font-bold hover:text-blue-600">
          {post.title}
        </h2>
      </Link>
      <time className="text-sm text-gray-500">{post.date}</time>
      <p className="mt-2 text-gray-600">{post.excerpt}</p>
      <div className="mt-2 flex gap-2">
        {post.tags.map(tag => (
          <Link
            key={tag}
            href={`/tags/${tag}`}
            className="text-xs bg-gray-100 px-2 py-1 rounded hover:bg-gray-200"
          >
            #{tag}
          </Link>
        ))}
      </div>
    </article>
  )
}
```

## 📝 示例文章格式

```mdx
---
title: "Hello World"
date: "2024-01-01"
tags: ["入门", "博客"]
excerpt: "这是我的第一篇博客文章"
---

# Hello World

欢迎来到我的博客！

## 代码示例

```javascript
console.log('Hello, Vibe Coding!')
```

## 总结

这就是我的第一篇文章。
```

## 🚀 部署步骤

1. 推送代码到 GitHub
2. 在 Vercel 导入项目
3. 自动部署完成

## ✅ 验收清单

- [ ] 首页正确显示文章列表
- [ ] 点击文章可以进入详情页
- [ ] 代码块有语法高亮
- [ ] 标签页面可以筛选文章
- [ ] 移动端显示正常
- [ ] 部署到 Vercel 成功

## 🔗 相关资源

- [Next.js 文档](https://nextjs.org/docs)
- [MDX 文档](https://mdxjs.com/)
- [Tailwind CSS](https://tailwindcss.com/)

## 💡 进阶挑战

完成基础版本后，可以尝试：
- 添加搜索功能
- 添加评论系统（Giscus）
- 添加 RSS 订阅
- 添加暗色模式
- 添加阅读时间估算
