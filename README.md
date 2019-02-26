# blog
个人博客源码

## 命令

基本命令：
- `hexo init` - 初始化博客
- `hexo n [new blog]` - 新建文章
- `hexo g` - 生成网页
- `hexo s` - 启动服务器预览
- `hexo d` - 部署
- `hexo clean ` - 清除缓存

常用命令：

- `npm run dev` - 服务器预览（localhost:4000）
- `hexo n [blog_name]` - 新建文章
- `hexo n [page/draft/post] [page_name] ` - 新建页面/草稿/文章
- `npm run deploy` - 清除缓存并

## 主题

主题 [Next](https://github.com/theme-next/hexo-theme-next)

```
git clone https://github.com/theme-next/hexo-theme-next themes/next
```

## [主题文件无法推送到自己的仓库的解决方法](https://swibinchter.github.io/2017/01/11/Hexo%E5%8D%9A%E5%AE%A2%E4%B8%BB%E9%A2%98%E6%96%87%E4%BB%B6%E5%A4%B9%E6%97%A0%E6%B3%95%E5%AE%8C%E6%95%B4push%E5%8F%8Aclone/)

1. 将主题文件中的`.git`文件夹及相关文件夹删除
2. 重新add，并推送

## 主页显示文章摘要

在文章内部添加`<!-- more -->`，前面的部分即为摘要

## 评论系统

[Valine](https://valine.js.org/)
