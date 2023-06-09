## 前言

技术文章，尤其是前端技术文章具有时效性。

如文中提到的部分内容出现*break change*或出现内容错误（文字错误/错误的理论描述），为尽可能避免对后面的读者造成困扰，如果可以的话，希望在文章的评论区或代码仓库issues中予以指正，十分感谢。

相关仓库地址：

[代码仓库](https://github.com/JUST-Limbo/vue2-ssr-practice)

[文章所在仓库](https://github.com/JUST-Limbo/informal-essay)

**另外**，如果你找到这篇文章时的目的是在为公司调研关于`Vue`的`SSR`技术方案，我的观点是直接上`Nuxt3`。

## 摘要

目前常见的技术论坛中关于`Vue2`的`SSR`文章，主要是以`vue/cli@3`和`vue/cli@4`为基础创建的工程，并且很多示例工程在业务开发阶段实际使用的是`SPA`的模式，仅在部署时才会启动`SSR`模式，而开发和部署渲染方式不一致会导致很多问题只有到部署时才会暴露。

[示例仓库](https://github.com/JUST-Limbo/vue2-ssr-practice)主要是在[HackerNews Demo](https://github.com/vuejs/vue-hackernews-2.0)的基础上，将其`webpack3`的依赖升级到`webpack5`，消除一些旧版本依赖包存在的问题（比如`node-sass`），给出`SPA`和`SSR`的`webpack`配置。

本文的主要内容是介绍示例仓库中的webpack配置，侧重方向是`SSR`开发模式的构建配置。

## 正文

### main requirements

| name    | version | note                                                         |
| ------- | ------- | ------------------------------------------------------------ |
| nvm     | 1.1.9   | node版本控制器，用来切换node版本，不多说                     |
| node.js | 16.20.0 | webpack5最低支持node10.13，pnpm最低支持node16.14，这里选择16.20.0，不多说 |
| pnpm    | 8.6.1   | 包管理工具，用过都说好，不多说                               |
| vue     | 2.6     | vue的版本直接影响`vue-loader vue-template-compiler vue-server-renderer`</br>2.7以后支持组合式 API<br/>这里保守起见，采用2.6下最后一个patch版本（当前是2.6.14） |
| webpack | 5.85.0  | 相比wp3和wp4，编译性能十分强劲，心智负担大幅度降低，文档易读性更高 |
| express | 4       | 传世经典，花个把小时看下文档就够用了                         |
| chalk   | 4       | chalk@5不支持require导入，这里选@4                           |

### 快速创建一个webpack项目

空白目录下，依次输入指令：

```bash
pnpm init
pnpm i webpack@5 webpack-cli@5 -D
npx webpack init
```



### histroyApi



## 参考资料