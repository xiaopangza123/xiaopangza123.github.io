# 小胖砸的 Hexo 博客

个人技术博客源码，使用 Hexo 7 和 AnZhiYu 主题，通过 GitHub Actions 发布到 GitHub Pages。

站点地址：https://xiaopangza.site

## 内容方向

- AI 与 Hermes Agent
- Linux、Docker 与服务器运维
- Java 与开发工具
- Oracle 数据库与 RAC
- 数字生活和个人项目

## 本地开发

要求 Node.js 20.19+，推荐 Node.js 22 LTS。当前核心版本：

- Hexo 8.1.2
- AnZhiYu 1.7.1

```bash
npm ci
npm run server
```

生成并验证静态站点：

```bash
npm run check
```

生成结果位于 `public/`。

## 发布

推送到 `main` 分支后，`.github/workflows/pages.yml` 会执行确定性安装、构建并部署 GitHub Pages。Pull Request 只执行构建验证，不会部署。

自定义域名由 `source/CNAME` 管理。

## 写作规范

新文章应至少补齐：

- `title`
- `date` 和 `updated`
- `description`
- 一个主分类
- 3–6 个标签
- 稳定英文文件名/slug
- 固定封面

文章正文从二级标题开始，页面标题由主题输出，避免重复 H1。

## 站点分工

Hexo 站强调轻量静态归档；Halo 主博客位于 https://blog.xiaopangza.cn/，用于更丰富的内容和交互。两个站点保持统一品牌和内容方向。
