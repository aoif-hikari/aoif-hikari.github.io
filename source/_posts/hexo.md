---
date: 2021/7/7 19:46:00
tags: hexo 
mathjax: true 
title: hexo配置 
---

> hexo，快速、简洁且高效的博客框架

<!--more-->

## 基本操作

```
hexo clean // 清除缓存
hexo g // 创建静态文件
hexo s // 本地运行
hexo d // 部署到远程
```

## 主题

主题文件夹放在`./theme/` 下，`_config.yml` 中`theme:` 改成主题文件夹名

## 文章

文章存放路径`/source/_posts/*.md`

- 文章头部


```
---
date: 2017/8/3 18:20:00 //发布时间
tags: hexo //分类
mathjax: true //使用mathjax渲染公式
title: hexo博客 //标题
---
```

- 文章折叠

需要折叠的部分之前加

```
<!-- more -->
```

- 公式

卸载原有公式渲染引擎 `npm uninstall hexo-renderer-marked --save` 

安装kramed `npm install hexo-renderer-kramed --save`

> 问题1：kramed对`\,{,}`有转义，无法正常显示
>
> 解决方法：
>
> 定位到博客根目录，找到`/node_modules/kramed/lib/rules/inline.js`文件，进行部分修改：
>
> ```js
> //escape: /^\\([\\`*{}\[\]()#$+\-.!_>])/,      第11行，将其修改为
> escape: /^\\([`*\[\]()#$+\-.!_>])/,
> //em: /^\b_((?:__|[\s\S])+?)_\b|^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,    第20行，将其修改为
> em: /^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
> ```



> 问题2：公式中以下符号会解析错误`{{` `}}` `{%` `%}`
>
> 解决方法：中间加空格，变为`{ {` `} }` `{ %` `% }`

## 多分支管理

同时管理了两个分支：

- master -负责展示静态网页
- hexo -备份本地hexo文件（默认分支）

执行`hexo d`时，自动更新master分支（config文件中deploy配置）

add, commit, pull更新hexo分支需要备份的源文件

## 其他

无法解决的问题尝试清除缓存`hexo clean`后重新生成

