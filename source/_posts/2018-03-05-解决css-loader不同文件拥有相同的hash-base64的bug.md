---
title: '解决css-loader不同文件拥有相同的hash:base64的bug'
date: 2018-03-05 21:01:27
tags:
- 前端
- Webpack
---

> **前言**
> 解决 css-loader 在启用 cssModules 的时候处理相同文件名的文件的时候会产生相同的 [hash:base64] 的 bug

<!-- more -->

## bug 描述

css-loader 在启动了 `modules` 的时候，默认会将类名转换为 [hash:base64]，出于开发方便的考虑，我们一般会配置转换规则 `localIdentName` 为 `[local]___[hash:base64]`，这样同时显示了类名和哈希串，便利了开发也实现了样式隔离。但是会出现一种情况，有时候相同文件名里面的相同样式所生成的哈希串会是一样的，这是为什么呢？

```js
// webpack 配置
{
  test: /\.less$/,
  include: 'src',
  use: [
    {
      loader: 'style-loader',
    },
    {
      loader: 'css-loader',
      options: {
        modules: true,
        localIdentName: '[local]___[hash:base64]',
      },
    },
    {
      loader: 'less-loader',
    },
  ],
}
```

```js
// /page1 下面的文件以及样式
index.less -> .main {}
// /page2 下面的文件以及样式
index.less -> .main {}
```

```css
/* 生成的哈希值 */
/* /page1/index.less */
.main___3JR3AQ6dNZLxjhld7RKqmr
/* /page2/index.less */
.main___3JR3AQ6dNZLxjhld7RKqmr
```

## 原因

> 相关 issue
> [Same file name but different directory, after build hash is same.(use css module)](https://github.com/webpack-contrib/css-loader/issues/464/)
> 英文好的哥们可以直接进去看

大概的原因是 css-laoder 的实现里面是根据引用文件时的相对路径来作为文件夹的路径，上述两个文件在他们各自的根目录下被引用时文件名（以及路径）是一样的，所以生成的哈希串是一样的（这里的哈希串是以唯一的路径作为唯一的依据而非文件内容）。

解决方法是给 css-loader 添加一个参数 `context` 来作为判断文件路径的上下文（这个参数没有在文档中被列出，可能是 hack 出来的）比如是源代码的根路径 src，这样上面两个文件的路径就分别变为 `/page1/index.less` 和 `/page2/index.less` 了。(我用的是 `path` 模块包装过的绝对路径，而不是下面写的相对路径)

```js
// webpack 配置
{
  test: /\.less$/,
  include: 'src',
  // loader: 'style!css?minimize&-autoprefixer!postcss!less',
  use: [
    {
      loader: 'style-loader',
    },
    {
      loader: 'css-loader',
      options: {
        modules: true,
        context: 'src', // 修复不同文件拥有相同[hash:base64]的bug-https://github.com/webpack-contrib/css-loader/issues/464
        localIdentName: '[path]_[name]_[local]___[hash:base64]',
      },
    },
    {
      loader: 'less-loader',
    },
  ],
}
```

## 思考

这个问题大概耗费了一个上午的事件去解决，考虑过是不是被其他模块影响、模块有没有 bug、也看了下源码、查了下社区，但是都没有深入去钻，导致了时间白白流逝。

后面优化了一下关键字，从 `[hash:base64] no unique`，到 `different file same [hash:base64] localIdentName` 等关键字，才终于找到了相关的讨论，解决这问题的哥们正是从源码中发现了设计者的上述的设计误区。

更加有趣的是，没找到解决问题前，`localIdentName` 配置项中的 `[path]` 在相同相对路径相同文件名里面被输出的都是空（`/`)，在设置了引用上下文之后，`[path]`便能被正确解析出来了。

所以，以后遇到问题，最好能后沉下心来，试着从反思到的这几点着手解决

- 整理模块的使用方式和阅读文档，但有无错误使用
- 试着查看源码，找出出错部分的逻辑设计与文档设计预期是否相同
- 试着调整关键字查找相关讨论，沉下心挑战英文环境的不适应
- 解决完问题记得要记录和整理
