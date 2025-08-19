# Hexo 博客项目指南

这是一个基于 Hexo 7.3.0 框架 + Next 8.20.0 主题的个人博客项目。

## 项目结构

- **主配置文件**: `_config.yml` (Hexo 主配置)
- **主题配置文件**: `_config.next.yml` (Next 主题配置)
- **文章源码**: `source/_posts/` (按分类组织)
- **草稿**: `source/_drafts/`
- **静态文件**: `source/images/`
- **生成的网站**: `public/`

## 常用命令

### 基础操作
- **本地预览**: `npm run server` 或 `hexo server`
- **清理缓存**: `npm run clean` 或 `hexo clean`
- **生成静态文件**: `npm run build` 或 `hexo generate --prod`
- **部署到 GitHub**: `npm run deploy` 或 `hexo deploy`

### 内容管理
- **新建文章**: `hexo new post "文章标题"`
- **新建草稿**: `hexo new draft "草稿标题"`
- **发布草稿**: `hexo publish "草稿标题"`
- **新建页面**: `hexo new page "页面名"`

## 部署配置

项目使用 Git 部署到 GitHub Pages:
- 仓库: `git@github.com:samwei12/samwei12.github.io.git`
- 分支: `main`
- 网站地址: `http://blog.samwei12.cn`

## 主题特性

使用 Next 主题的 Gemini 方案，已启用:
- 深色模式支持
- 本地搜索功能
- 文章字数统计和阅读时间
- 相关文章推荐
- 阅读进度条
- 评论系统 (Disqus)
- 访问统计 (LeanCloud + 不蒜子)
- Google Analytics 和百度统计

## 文章分类

项目按以下分类组织文章:
- **Assembly**: 汇编相关
- **Objective-C**: iOS 开发
- **Python**: Python 编程
- **Reading**: 读书笔记
- **Ruby**: Ruby 和 Cocoapods
- **Server**: 服务端技术
- **Thinking**: 思考随笔
- **Utilities**: 工具和效率

## 插件和依赖

已安装的主要插件:
- `hexo-deployer-git`: Git 部署
- `hexo-generator-*`: 各种生成器
- `hexo-renderer-*`: 渲染器
- `hexo-theme-next`: Next 主题
- `hexo-word-counter`: 字数统计
- `hexo-leancloud-counter-security`: 访问量统计安全

## 开发工作流

1. **写作新文章**:
   ```bash
   hexo new post "文章标题"
   # 编辑 source/_posts/文章标题.md
   npm run server  # 本地预览
   ```

2. **发布流程**:
   ```bash
   npm run clean
   npm run build
   npm run deploy
   ```

3. **主题自定义**: 修改 `_config.next.yml` 文件

## 注意事项

- 文章使用 Markdown 格式，支持 GFM 语法
- 图片放在 `source/images/` 目录
- 修改配置后需要重启本地服务器
- 部署前建议先本地预览确认效果
- Next 主题配置文件独立，便于主题升级

## SEO 优化

已配置:
- 站点地图和 RSS 订阅
- Google 和百度站长验证
- 百度主动推送
- 规范链接标签
- Open Graph 协议

## 问题排查

- **构建失败**: 先运行 `npm run clean` 清理缓存
- **主题样式异常**: 检查 `_config.next.yml` 配置
- **部署失败**: 确认 SSH 密钥配置和仓库权限
- **插件冲突**: 查看 `node_modules` 和依赖版本