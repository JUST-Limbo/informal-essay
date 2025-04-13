## 前言

技术文章，尤其是前端技术文章具有时效性。

如文中提到的部分内容出现*break change*或出现内容错误（文字错误/错误的理论描述），为尽可能避免对后面的读者造成困扰，如果可以的话，希望在文章的评论区或代码仓库issues中予以指正，十分感谢。

## 摘要

本文主要介绍了uniapp编译到小程序平台时，页面自定义导航栏适配小程序胶囊布局的应对思路。

**本文给出的实现思路和其他资料中的实现思路大体并无差异。**

本文想要强调的是：相比于在`onLoad、created`回调中初始化数据的常见方式，利用状态管理工具`Vuex、Pinia`来存储胶囊布局信息是更好的方式。

## 效果预览

| ![QQ_1744478401186](.\assets\QQ_1744478401186.png) | ![QQ_1744471479835](C:\code\informal-essay\assets\QQ_1744471479835.png) | ![QQ_1744471489856](.\assets\QQ_1744471489856.png) | ![QQ_1744471499714](.\assets\QQ_1744471499714.png) |
| -------------------------------------------------- | ------------------------------------------------------------ | -------------------------------------------------- | -------------------------------------------------- |

## 概述

事实上，关于适配胶囊布局的问题，很容易就可以在网络资料中找到最优解，即：

**自定义导航栏高度 = 胶囊上边界坐标 + 胶囊高度**

如果想要让导航栏内容和胶囊在垂直方向上对齐，那么你需要：

**自定义导航栏上内边距 = 胶囊上边界坐标**

> 注意，此时自定义导航栏的box-sizing值为border-box

如果想要让导航栏的内容不被右边胶囊遮挡，那么你需要：

**自定义导航栏右内边距 =  屏幕宽度 - 胶囊左边界坐标**

## 需要注意

部分安卓机型调用`uni.getMenuButtonBoundingClientRect`的返回结果有误差，具体表现为`top、right`等值偏大。

## 数据的获取时机

在很多材料中，对于胶囊节点信息数据的读取通常在`created、mounted、onLoad、onReady、onLaunch`中，然后再将整理好的数据在赋值到`data(){}`中的变量上。

实际上我们可以通过直接在`data(){}`中进行初始化使代码更紧凑，又或者借助状态管理工具集中管理以避免代码重复。

### 在data()中直接初始化

```vue
<script>
export default {
	data() {
		const { windowWidth } = uni.getWindowInfo()
		const { top, height, left } = uni.getMenuButtonBoundingClientRect()
		return {
			navBarHeight: top + height,
			menuHeight: height,
			menuTop: top,
			menuLeft: windowWidth - left
		}
	}
}
</script>
```

### 在Vuex中组织初始化代码

注意`main.js`中的`store.commit("menuButtonLayout/init")`

```js
// store/modules/menuButtonLayout.js
export default {
	namespaced: true,
	state: () => ({
		statusBarHeight: 0,
		navBarHeight: 0,
		menuHeight: 0,
		menuTop: 0,
		menuRight: 0,
		menuWidth: 0
	}),
	mutations: {
		init(state) {
			const { statusBarHeight, windowWidth } = uni.getWindowInfo()
			const { top, height, right, width, left } = uni.getMenuButtonBoundingClientRect()
			state.statusBarHeight = statusBarHeight
			state.navBarHeight = top + height // 也可以选择额外+10留出一些底部空间
			state.menuHeight = height // 胶囊高度
			state.menuWidth = width
			state.menuTop = top // 胶囊上坐标
			state.menuRight = windowWidth - right // 胶囊右侧距右方间距（并不是胶囊右边界坐标）
			state.menuLeft = windowWidth - left // 胶囊左侧距右方间距（并不是胶囊左边界坐标）
		}
	}
}

// store/index.js
import Vue from "vue"
import Vuex from "vuex"
import menuButtonLayout from "./modules/menuButtonLayout"
Vue.use(Vuex)
const store = new Vuex.Store({
	state: {},
	modules: {
		menuButtonLayout
	}
})
export default store

// main.js
import App from "./App"
import Vue from "vue"
import store from "./store"
Vue.config.productionTip = false
App.mpType = "app"
store.commit("menuButtonLayout/init") // 初始化节点信息
const app = new Vue({
	...App,
	store
})
app.$mount()

```

### 在Pinia中组织初始化代码

```js
// stores/menuButtonLayout.js
import { defineStore } from "pinia"
import { ref } from "vue"

export const useMenuButtonLayoutStore = defineStore("menuButtonLayout", () => {
	const { statusBarHeight, windowWidth } = uni.getWindowInfo()
	const { top, height, right, width, left } = uni.getMenuButtonBoundingClientRect()
	return {
		statusBarHeight: ref(statusBarHeight),
		navBarHeight: ref(top + height),
		menuHeight: ref(height),
		menuWidth: ref(width),
		menuTop: ref(top),
		menuRight: ref(windowWidth - right),
		menuLeft: ref(windowWidth - left)
	}
})

```

## Vue2用例

```vue
<template>
	<div>
		<div
			v-if="curNavBarType === 1"
			class="nav-bar nav-bar1"
			:style="{
				boxSizing: 'border-box',
				height: navBarHeight + 'px',
				lineHeight: menuHeight + 'px',
				paddingTop: menuTop + 'px',
				textAlign: 'center'
			}"
		>
			无返回居中导航栏怪异盒子
		</div>
		<div
			v-else-if="curNavBarType === 2"
			class="nav-bar nav-bar2"
			:style="{
				height: menuHeight + 'px',
				lineHeight: menuHeight + 'px',
				paddingTop: menuTop + 'px',
				textAlign: 'center'
			}"
		>
			无返回居中导航栏标准盒子
		</div>
		<div
			v-else-if="curNavBarType === 3"
			class="nav-bar nav-bar3"
			:style="{
				boxSizing: 'border-box',
				height: navBarHeight + 'px',
				lineHeight: menuHeight + 'px',
				paddingTop: menuTop + 'px',
				textAlign: 'center'
			}"
		>
			<uni-icons type="back" size="30" style="position: absolute; left: 0"></uni-icons>
			有返回居中导航栏怪异盒子
		</div>
		<div
			v-else-if="curNavBarType === 4"
			class="nav-bar nav-bar4"
			:style="{
				boxSizing: 'border-box',
				height: navBarHeight + 'px',
				paddingTop: menuTop + 'px',
				paddingRight: menuLeft + 'px',
				display: 'flex',
				alignItems: 'center'
			}"
		>
			<uni-icons type="back" size="30"></uni-icons>
			<input type="text" placeholder="搜索" style="flex: 1; background: #fff" />
		</div>
		<div class="nav-bar-type-list">
			<div class="nav-bar-type-item" @click="curNavBarType = 1">无返回居中导航栏</div>
			<div class="nav-bar-type-item" @click="curNavBarType = 2">无返回居中导航栏标准盒子</div>
			<div class="nav-bar-type-item" @click="curNavBarType = 3">有返回居中导航栏怪异盒子</div>
			<div class="nav-bar-type-item" @click="curNavBarType = 4">搜索导航栏</div>
		</div>
		<div class="lorem" v-for="i in 20" :key="i">
			Lorem ipsum dolor sit amet consectetur adipisicing elit. Rerum itaque eius maxime nihil ducimus asperiores nostrum beatae aspernatur nam. Iusto temporibus eaque cupiditate porro! Ipsam repellat dolores tempore eius quasi.
		</div>
	</div>
</template>

<script>
import { mapState } from "vuex"
export default {
	data() {
		return {
			curNavBarType: 1
		}
	},
	computed: {
		...mapState("menuButtonLayout", ["navBarHeight", "menuTop", "menuHeight", "menuLeft"])
	}
}
</script>

<style lang="scss" scoped>
.nav-bar {
	position: sticky;
	top: 0;
	left: 0;
	background-color: lightgreen;
}
.nav-bar-type-list {
	.nav-bar-type-item {
		margin-top: 10px;
		padding: 10px;
		background-color: lightblue;
		text-align: center;
	}
}
</style>

```

## Vue3用例

```vue
<template>
	<div>
		<div
			v-if="curNavBarType === 1"
			class="nav-bar nav-bar1"
			:style="{
				boxSizing: 'border-box',
				height: navBarHeight + 'px',
				lineHeight: menuHeight + 'px',
				paddingTop: menuTop + 'px',
				textAlign: 'center'
			}"
		>
			无返回居中导航栏怪异盒子
		</div>
		<div
			v-else-if="curNavBarType === 2"
			class="nav-bar nav-bar2"
			:style="{
				height: menuHeight + 'px',
				lineHeight: menuHeight + 'px',
				paddingTop: menuTop + 'px',
				textAlign: 'center'
			}"
		>
			无返回居中导航栏标准盒子
		</div>
		<div
			v-else-if="curNavBarType === 3"
			class="nav-bar nav-bar3"
			:style="{
				boxSizing: 'border-box',
				height: navBarHeight + 'px',
				lineHeight: menuHeight + 'px',
				paddingTop: menuTop + 'px',
				textAlign: 'center'
			}"
		>
			<uni-icons type="back" size="30" style="position: absolute; left: 0"></uni-icons>
			有返回居中导航栏怪异盒子
		</div>
		<div
			v-else-if="curNavBarType === 4"
			class="nav-bar nav-bar4"
			:style="{
				boxSizing: 'border-box',
				height: navBarHeight + 'px',
				paddingTop: menuTop + 'px',
				paddingRight: menuLeft + 'px',
				display: 'flex',
				alignItems: 'center'
			}"
		>
			<uni-icons type="back" size="30"></uni-icons>
			<input type="text" placeholder="搜索" style="flex: 1; background: #fff" />
		</div>
		<div class="nav-bar-type-list">
			<div class="nav-bar-type-item" @click="curNavBarType = 1">无返回居中导航栏</div>
			<div class="nav-bar-type-item" @click="curNavBarType = 2">无返回居中导航栏标准盒子</div>
			<div class="nav-bar-type-item" @click="curNavBarType = 3">有返回居中导航栏怪异盒子</div>
			<div class="nav-bar-type-item" @click="curNavBarType = 4">搜索导航栏</div>
		</div>
		<div class="lorem" v-for="i in 20" :key="i">
			Lorem ipsum dolor sit amet consectetur adipisicing elit. Rerum itaque eius maxime nihil ducimus asperiores nostrum beatae aspernatur nam. Iusto temporibus eaque cupiditate porro! Ipsam repellat dolores tempore eius quasi.
		</div>
	</div>
</template>

<script setup>
import { ref } from "vue"
import { storeToRefs } from "pinia"

import { useMenuButtonLayoutStore } from "@/stores/menuButtonLayout"

const menuButtonLayoutStore = useMenuButtonLayoutStore()
const { navBarHeight, menuHeight, menuTop, menuLeft } = storeToRefs(menuButtonLayoutStore)

const curNavBarType = ref(1)
</script>

<style lang="scss" scoped>
.nav-bar {
	position: sticky;
	top: 0;
	left: 0;
	background-color: lightgreen;
}
.nav-bar-type-list {
	.nav-bar-type-item {
		margin-top: 10px;
		padding: 10px;
		background-color: lightblue;
		text-align: center;
	}
}
</style>

```



## 是否要自定义导航栏封装组件

![QQ_1744477287694](C:\code\informal-essay\assets\QQ_1744477287694.png)

需要特别注意的是，`uniapp`编译到小程序平台的组件，其在节点树的表现方式并不像H5平台上那样，我们在开发中应该尽量小心。

由上图我们可以观察到，编译到小程序平台的自定义组件会在外层包裹一个自定义标签，并且这个自定义标签默认是行内元素，**如果你按照图中复现代码，你甚至可以观察到该行内元素上下`margin`居然生效，而左右`margin`居然不生效的诡异情况**。

此外，小程序的自定义组件默认不会内外`class`合并（注意观察图中的`.class-define-in-parentpage`和`.class-define-in-test`）。

为了平滑渡过这些跨小程序端的坑点，你需要仔细查阅`virtualHost、mergeVirtualHostAttributes`的相关资料：

[应用配置 | uni-app官网](https://uniapp.dcloud.net.cn/tutorial/vue3-api.html#其他配置)

[manifest.json 应用配置 | uni-app官网](https://uniapp.dcloud.net.cn/collocation/manifest.html#mp-weixin)

[自定义组件 / 组件模板和样式](https://developers.weixin.qq.com/miniprogram/dev/framework/custom-component/wxml-wxss.html#虚拟化组件节点)



除了上面提到的，考虑到当页面使用了自定义导航栏模式时（`navigationStyle:custom`），你所在公司的产品经理和UI设计一定会整出什么逆天的烂活儿。那么为了以后的开发体验，我的建议还是最好不要上来就封装通用的自定义导航栏组件，除非真的有很多页面确定会复用同一个导航栏。

## 总结

如果你的小程序不用考虑宽屏、横屏的场景，那么一个设备上的胶囊布局数据通常是固定不变的。所以在组织代码的时候没必要反复调用API获取，只需要初始化一次，并且初始化时机越早越好，没有必要放到页面的生命周期回调函数中。

## 参考资料·鸣谢

[优雅解决uniapp微信小程序右上角胶囊菜单覆盖问题- 掘金](https://juejin.cn/post/7309361597556719679)

[uniapp标题水平对齐微信小程序胶囊按钮及适配_uniapp 水平对齐-CSDN博客](https://blog.csdn.net/weixin_42220130/article/details/139981343)
