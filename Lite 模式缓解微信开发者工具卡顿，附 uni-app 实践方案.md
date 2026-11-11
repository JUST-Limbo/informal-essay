# Lite 模式缓解微信开发者工具卡顿，附 uni-app 实践方案

## 前言

用 uni-app 开发微信小程序时，我通常在外部编辑器里写代码，在终端中运行编译命令，再打开微信开发者工具查看页面、调试和预览。

这套流程用得多了，一个问题就变得很明显：微信开发者工具有时会比较卡。明明代码编辑和 uni-app 编译已经放在外部完成，为什么还需要一直开着一套完整的开发工作台？有没有更轻一点的用法？

最初，我想研究一下能不能做一个 Web 版工作台。结果在分析微信开发者工具的安装目录时，偶然发现了官方内置的 Lite 模式。继续追查后，发现官方连对应的启动命令都已经写在 Skill 里了。

这篇文章介绍这条启动方式，以及在 uni-app CLI 工程中如何使用。

## 微信开发者工具为什么会卡顿

先说一个前提：卡顿是一种表现，背后可能有不同原因。没有针对具体项目做分析之前，很难把它归结为某一个功能。

完整的开发者工具需要承担多项工作：代码编辑与分析、项目文件监听、小程序代码编译、模拟器运行，以及调试器中的日志、网络请求和页面检查等。项目文件多、编译频繁、运行逻辑复杂，或者调试信息持续积累，都可能增加资源开销。

对于 uni-app，还要区分两段编译过程：

```text
uni-app 源码
    ↓ uni-app 构建工具
微信小程序产物（通常位于 dist/dev/mp-weixin）
    ↓ 微信开发者工具继续编译和加载
模拟器中的页面
```

uni-app 已经生成小程序代码，并不意味着微信开发者工具可以直接跳过自己的编译和运行环节。它仍然需要处理这些产物，启动小程序运行环境。

不过，如果平时主要用外部编辑器写代码，只在开发者工具中看模拟器，那么确实可以尝试更精简的工作台。Lite 模式正好提供了这样的入口。

至于它能减少多少内存、启动速度能改善多少，本文先不下结论，后面会单独补充对比实测。

## 从安装目录里发现 Lite 模式

这次探索的出发点，其实是想把开发者工具精简一下，甚至研究能不能把工作台搬到浏览器里。

我先去看了本机开发者工具的安装目录。在追踪项目窗口的创建逻辑时，发现代码里出现了 `liteMode`，并且开窗入口能够选择窗口模式。

顺着这条线索继续找，又发现安装包内提供了一套 `wechatide-skill`。其中的 `initializer` Skill 明确写着：

```bash
wechatide -c <clientName> open_project_window --project <project> [--window-mode liteMode|fullMode]
```

也就是说，官方已经提供了启动 Lite 窗口的命令入口。

本次查看的安装包中，这份说明位于以下相对位置，具体安装根目录因电脑而异：

```text
resources/app.asar.unpacked/wechatide-skill/skills/initializer/SKILL.md
```

这里仅把安装文件作为调查和查阅依据。使用 Lite 不需要修改安装目录，也不需要把官方程序文件复制出来。

## Lite 模式是什么

按照安装包内的官方 Skill 说明，`liteMode` 默认只打开模拟器；需要编辑器和调试器时，使用 `fullMode`。

因此，它适合这样的日常分工：

| 工作 | 使用的工具 |
| --- | --- |
| 编辑代码、管理 Git | 自己常用的编辑器 |
| 编译 uni-app 项目 | 项目中的构建命令 |
| 查看小程序页面 | 微信开发者工具 Lite 模式 |
| 使用完整编辑与调试工作台 | 微信开发者工具完整模式 |

Lite 仍然运行在本机微信开发者工具中，模拟器、编译链路和相关后台服务仍然存在。它不能消除小程序自身的运行开销，也不能保证解决所有卡顿。

我注意到，抖音、支付宝的小程序开发者工具中，已经能看到切换 Lite 模式的界面入口。但在本次检查的微信开发者工具版本中，没有找到类似的界面切换开关，最终是在官方 Skill 中找到了这条命令。

HBuilderX 的自动启动也没有直接帮我走这条通道。本次检查本机 HBuilderX 的微信小程序启动代码，Windows 下使用的是传统的 `cli.bat open --project`，macOS 下也是对应 CLI 的 `open --project`，没有调用这里的 `open_project_window --window-mode liteMode`。

因此，使用 HBuilderX 自动拉起微信开发者工具，并不意味着已经启用了本文介绍的 Lite 模式。这里描述的是本次检查到的启动实现，后续版本可能调整。

## 用官方命令启动 Lite 模式

### 先更新微信开发者工具

建议先更新到最新版，再使用下面的命令。旧版本可能没有提供相同的 `wechatide` 入口或参数。

正常安装并配置命令后，可以先在终端查看帮助：

```bash
wechatide --help
```

如果安装后仍提示找不到命令，先重新打开终端；如果终端集成在编辑器中，也需要重启编辑器，让新进程读取更新后的 PATH。Windows 上仍未刷新时，可以再按官方说明重启资源管理器或电脑。

macOS 的命令可用性还与安装方式有关，若入口未配置到 PATH，需要按官方安装指引补齐。单纯反复重启并不能解决没有配置入口的问题。

### 打开项目

以一个 Windows 工程为例：

```bash
wechatide -c uni-app open_project_window --project "C:/projects/my-uni-app/dist/dev/mp-weixin" --window-mode liteMode
```

把路径换成自己项目的小程序产物目录即可，路径中有空格时也要保留引号。

| 参数 | 含义 |
| --- | --- |
| `-c uni-app` | 调用方名称，用于识别和授权；这里使用 `uni-app` 作为名称 |
| `open_project_window` | 打开项目窗口 |
| `--project` | 微信小程序项目目录，uni-app CLI 工程通常指向构建产物 |
| `--window-mode liteMode` | 使用 Lite 模式创建窗口 |

首次以新的调用方名称连接时，可能需要在开发者工具中完成授权。如果返回结果是等待授权或确认，先完成对应交互，不要把命令已经返回当成窗口已经打开。

有些资料或兼容入口会写成 `-t project_open_window`。本次检查的版本仍能识别，但会提示改用正式名称 `open_project_window`，本文统一使用正式写法。

### 注意：项目路径要用绝对路径

下面这种写法，在本次使用的 Windows 版本中会报“项目路径不存在”：

```bash
wechatide -c uni-app open_project_window --project ./dist/dev/mp-weixin --window-mode liteMode
```

调查安装文件后，可以看到 Windows 的 `wechatide.cmd` 在启动 CLI 前会切换到工具安装目录，而参数处理仅对路径格式做规范化，没有先按照调用方的项目目录，把 `--project` 转换为绝对路径。

这使得传入的相对路径不能按预期指向工程产物。官方 Skill 也要求路径参数使用本地绝对路径，因此这里直接传绝对路径即可。

这不是 npm script 的限制，而是本次检查的 CLI 路径处理行为；后续版本如果修复，应以实际行为为准。

### 已有窗口会被复用

如果相同项目已经开着，命令可能返回 `type: reuse`，复用原来的窗口。指定 `liteMode` 并不保证把一个已有的完整模式窗口切换为 Lite。

因此，如果发现执行后仍是原来的完整模式，先手动关闭该项目窗口，再重新执行命令。

## 在 uni-app CLI 工程中使用

下面给出两种方式：直接维护一条开窗命令，或者接入构建工具插件。

### 方案一：在 package.json 中维护命令

在项目已有的 `scripts` 中增加一项，保留原来的开发命令：

```json
{
  "scripts": {
    "wx:lite": "wechatide -c uni-app open_project_window --project \"C:/projects/my-uni-app/dist/dev/mp-weixin\" --window-mode liteMode"
  }
}
```

上面的工程路径只是示例，需要替换成自己的绝对路径。macOS 使用对应的本地绝对路径。

先正常运行开发命令：

```bash
npm run dev:mp-weixin
```

等待第一次编译完成，确认 `dist/dev/mp-weixin` 中已经生成小程序产物和 `project.config.json`，再开一个终端执行：

```bash
npm run wx:lite
```

这是配置最少的方案。如果只是希望方便地打开 Lite，维护这一条命令就足够了。

不足之处也很直接：它需要手动执行，示例中的绝对路径在工程迁移或不同成员使用时需要调整。也可以额外写一个短脚本，用 `path.resolve()` 计算路径，再调用官方命令。

不要直接把启动命令接在持续编译命令的 `&&` 后面：

```text
持续编译命令 && Lite 开窗命令
```

watch 进程会一直运行，后面的命令要等前者成功退出才会执行，不能借此实现“首次编译成功就开窗”。

### 方案二：使用构建工具插件自动启动

如果希望只运行一次 `npm run dev:mp-weixin`，编译完成后就自动打开 Lite，可以接入构建工具插件。

这次探索中，我把这部分逻辑整理成了一个社区插件包，提供 webpack 和 Vite 两个入口：

[wechatide-lite-plugins](https://github.com/JUST-Limbo/wechatide-lite-plugins)

它调用的是本机开发者工具提供的命令，不包含开发者工具安装文件。

目前可以从 GitHub 安装：

```bash
npm install -D github:JUST-Limbo/wechatide-lite-plugins
```

#### webpack 工程

在已有的 `vue.config.js` 中合并插件配置：

```js
const WechatideLitePlugin = require("wechatide-lite-plugins/webpack")

module.exports = {
    configureWebpack: {
        plugins: [new WechatideLitePlugin()],
    },
}
```

这里展示的是最小配置，不要直接覆盖工程已有的 `configureWebpack`。如果本来使用 `chainWebpack`，也可以在已有函数中注册：

```js
config.plugin("wechatide-lite").use(WechatideLitePlugin)
```

#### Vite 工程

在已有的 `vite.config.js` 或 `vite.config.ts` 中引入插件，放在 uni-app 插件后面：

```js
import { defineConfig } from "vite"
import uni from "@dcloudio/vite-plugin-uni"
import wechatideLite from "wechatide-lite-plugins/vite"

export default defineConfig({
    plugins: [uni(), wechatideLite()],
})
```

保留项目原有的 uni-app 插件导入、注册方式，只追加 `wechatideLite()` 即可。不同工程模板对 `@dcloudio/vite-plugin-uni` 的导出处理可能不同，不必为了追加插件改写原有配置。

#### 接入后怎么用

修改构建配置后，停止原来的编译进程，重新运行：

```bash
npm run dev:mp-weixin
```

插件会在第一次成功构建、产物落盘后尝试打开 Lite。首次编译失败不会开窗，修复后第一次成功才触发；同一开发进程后续热更新不会反复开窗，普通生产打包默认也不会触发。

它还会把产物目录转换成绝对路径，再交给 `wechatide`，因此不需要在配置中写死本机工程路径。默认跟随构建输出目录，特殊工程也可以通过 `project` 选项指定目录。

命令入口会按缓存、PATH 和系统安装信息查找，显式配置可以覆盖自动发现；首次客户端授权也会进入相应的等待处理。即便开窗失败，项目的编译监听仍然继续。

插件的作用主要是省掉这些启动衔接工作。如果愿意手动执行开窗命令，前一种方案已经可以满足需求。

## 效果对比

> 待补充：完整模式与 Lite 模式的界面截图、相同项目和操作条件下的内存占用、启动与编译体验，以及实际使用中的功能差异。本节等待后续实测后再填写，暂不提供性能降幅或效果结论。

## 参考资料

- [微信开发者工具 Skills 官方文档](https://developers.weixin.qq.com/miniprogram/dev/devtools/Skills.html)
- 微信开发者工具安装包中的 `wechatide-skill/skills/initializer/SKILL.md`：开窗命令与窗口模式说明。
- 微信开发者工具安装包中的 `wechatide.cmd`、`skill-index.js`：本次 Windows 路径行为的调查依据。
- 本机 HBuilderX 的 `plugins/uniapp-extension/out/index.js`：微信小程序启动链路的调查依据。
- [webpack / Vite 插件及使用说明](https://github.com/JUST-Limbo/wechatide-lite-plugins)

本文涉及的安装目录调查和命令行为截至 2026 年 11 月，后续版本可能发生变化。
