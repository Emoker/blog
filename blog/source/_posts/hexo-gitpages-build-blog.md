---
title: 使用 Hexo 在 GitHub 上搭建博客
date: 2019-06-28 17:24:20
categories:
tags:
- 示例
- 分享
- Hexo
- GitHub
- GitPages
---

这是一篇使用Hexo在GitHub上通过GitPages搭建博客的分享
<!-- more -->
### 太长不看版
##### Hexo 安装 | [官网](https://hexo.io/zh-cn/)
```
npm install hexo-cli -g
```

##### Hexo 常用命令 | [官网指令参考](https://hexo.io/zh-cn/docs/commands)
```
# 初始化一个Hexo项目
hexo init [folder]

# 新建一个文件
hexo new [layout] <filename>
# ex:
hexo new draft <filename>

# 生成页面
hexo generate
hexo g

# 清除生成的页面
hexo clean

# 本地服务预览
hexo server
hexo s

# 发布草稿
hexo publish [layout] <filename>
# ex:
hexo publish draft <filename>

# 安装 deploy 插件
npm install hexo-deployer-git --save

# 部署到GitHub
hexo deploy
hexo d
```

### 目录
- Hexo介绍
    - Hexo 是什么
    - Hexo 能做什么
- 开始一个Hexo项目
    - 前置依赖
    - 安装
    - 配置你的Hexo项目
    - 尝试编写
    - 预览
- 部署到GitHub
    - 前置依赖
    - 插件安装
    - 配置并部署
- 个性化定制
    - 个性化域名
    - 定制文章链接
    - 定制主题
- Hexo 高级功能
    - Hexo 添加插件
- 


参考
 - [hexo史上最全搭建教程](https://blog.csdn.net/sinat_37781304/article/details/82729029)
 - [link](https://note.youdao.com/)