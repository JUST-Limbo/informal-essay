## 前言

技术文章，尤其是前端技术文章具有时效性。

如文中提到的部分内容出现*break change*或出现内容错误（文字错误/错误的理论描述），为尽可能避免对后面的读者造成困扰，如果可以的话，希望在文章的评论区或代码仓库issues中予以指正，十分感谢。

文章代码所在仓库：<https://github.com/JUST-Limbo/vue3-gpt-practice>

**注意：因为是前端演示项目，所以这个仓库的对话数据是mock的，如果想要接入OpenAI这种大模型，你需要自行调整接口**

## 摘要

本文主要介绍了`GPT`打字机效果的前端实现思路。

在参考完~~抄别人作业~~目前的资料我总结了一下，打字机光标效果的实现思路主要有两种：

*   递归查找当前`DOM`树中的最后一个节点内容不为空的文本叶子节点，然后追加一个闪烁的光标`DOM`
*   依赖`CSS`选择器，通过`:last-child:after`子代选择器和伪类选择器锁定位置，其实也是递归查最后一个非空叶子节点。

**总之就是要找到最后一个非空叶子节点，然后在那个节点位置想办法把光标的效果摇出来。**

我采用的是第1种实现方式，不过本文会对这两种实现方式都略作分析。

## 效果预览

![GIF.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cfb56572126f419e827aed068b39aa31~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1022&h=842&s=374836&e=gif&f=232&b=f5f5f5)

## 队列思路概述

有一道很经典的小学数学题，说一个水池有一进水管一排水管，单开进水管5小时蓄满，单开排水管7小时排空，问从空池开始，同时打开进水管和出水管多久蓄满。

打字机的字符串渲染处理就类似于这种排水问题，它具备在入队的同时也在出队的特征，本质上是先入先出的队列思想。

为了使代码更紧凑，我们采用`ES6 class`的方式通过以下思路组织代码：

1.  声明一个`Pipe`类作为功能体的集合

2.  声明一个静态属性`str`来保存需要渲染的字符串内容

    *   声明一个`write`静态方法来填充`str`，即队列入队
    *   声明一个`pop`静态方法来删除`str`靠前的内容，即队列出队

3.  声明一个`start`静态方法启动渲染行为

    通过定时器尾递归执行渲染和出队行为，即修改`ref`对象来渲染视图层（`consume`）和弹出`str`队头元素（`pop`）

4.  声明一个`consumeAll`，一次性消耗掉队列中剩下的元素

    当`GPT`回答结束，意味着当前队列不会再有入队的元素，但是队列可能没渲染完，这种情况继续用定时器渲染一点一点是没有必要的，应该一次性把剩余所有元素都渲染到视图层上然后清空队列。

代码实现如下：

```typescript
// views/home/cmp/chatInputPanel.vue
class Pipe {
    static str = ''
    static timer = 0
    static target: Nullable<Ref<gptMockNamespace.chatRecord>> = null
    static reset () {
        Pipe.str = ''
        clearInterval(Pipe.timer)
        unLockScroll()
        Pipe.target = null
    }
    static start (data: Ref<gptMockNamespace.chatRecord>) {
        Pipe.target = data
        function recursiveTimeoutFunction () {
            Pipe.timer = setTimeout(() => {
                Pipe.consume(Pipe.getFirstStr())
                Pipe.pop()
                recursiveTimeoutFunction()
            }, 50);
        }
        recursiveTimeoutFunction()
    }
    static write (chunk: string) {
        Pipe.str += chunk
    }
    static getFirstStr () {
        return Pipe.str[0]
    }
    static pop () {
        Pipe.str = Pipe.str.substring(1)
    }
    static consume (message: string = '') {
        const ans = Pipe.target!
        ans.value.message += message
        !chatWindowScollLock.value && scrollToBottom()
    }
    static consumeAll () {
        Pipe.consume(Pipe.str)
        Pipe.str = ''
        delete Pipe.target!.value.chatting
        clearInterval(Pipe.timer)
    }
}
```

## 打字机光标实现

### DOM方式实现光标

这种方式主要是通过找到当前`DOM`树的最后一个文本叶子节点`lastText`，本质上是二叉树倒序遍历

然后用`appendChild`将`textNode`放到`lastText`后面，这个时候我们就能获取到`textNode`的横轴纵轴坐标了。

拿到坐标把坐标数据覆盖到光标`.blink`元素的坐标上就可以了。

```vue
// views/home/cmp/chatItemBot.vue
<script setup lang="ts">
import type { gptMockNamespace } from '@/api/interface/gptmock'

import markdown from '@/utils/markdownIt'
import { findLastNonEmptyTextNode } from '@/utils/DomUtils'

const chatMarkdownBody = ref()
const chatMessageContainer = ref()

const pos = reactive({ x: 0, y: 0 })

let textNode: Nullable<Text> = document.createTextNode('_')
function refreshPosXY () {
    const lastText = findLastNonEmptyTextNode(chatMarkdownBody.value)
    if (lastText) {
        lastText.parentNode!.appendChild(textNode!)
    }
    const range = document.createRange()
    range.setStart(textNode!, 0)
    range.setEnd(textNode!, 0)
    const textNodeRect = range.getBoundingClientRect()
    const containerRect = chatMessageContainer.value.getBoundingClientRect()
    pos.x = textNodeRect.left - containerRect.left
    pos.y = textNodeRect.top - containerRect.top
    textNode!.remove()
}
onMounted(() => {
    refreshPosXY()
})
onUpdated(() => {
    refreshPosXY()
})
onBeforeUnmount(() => {
    textNode = null
})
</script>
<style lang="scss" scoped>
.blink {
    position: absolute;
    width: 10px;
    height: 2px;
    transform: translateY(13px);
    background: black;
    left: calc(v-bind('pos.x') * 1px);
    top: calc(v-bind('pos.y') * 1px);
    animation: blink 1s steps(5, start) infinite;
}
</style>

```

这种实现有一个问题（能解决但是我觉得没必要）在这里也简单说一下：

```html
<div class="chat-markdown-body">
    <p>Lorem ipsum dolor sit amet consectetur, adipisicing elit. Reprehenderit officiis tempora iusto voluptates
        exercitationem vel praesentium dolores sapiente aliquam fugiat, esse eveniet quae quisquam omnis facilis
        deserunt ullam obcaecati? Harum.</p>
    <pre class="hljs highlight-pre">
        <code class="highlight-code"></code>
    </pre>
</div>
```

当`DOM`树结构是上述代码时，`findLastNonEmptyTextNode`匹配到的是`Harum.`的结尾，也就是说光标会落在`p`元素的结尾，然而这其实是有点不符合预期的，虽然`pre>code`的内容是空的，但是这其实意味着`p`元素的渲染其实已经结束了，即将渲染的是`code`的文本内容，所以光标应该落在`code`元素里。

这是一个小瑕疵，问题不是特别大。没有完善的原因是，我不想为了兼容这个场景在`findLastNonEmptyTextNode`函数内做硬编码判断破坏它的紧凑性。

### CSS方式实现光标

`CSS`伪类实现光标的性能要好很多。

这种方式也能应对大多数的场景，有个小问题是子代选择器没有什么特别全面的办法锁定最后一个非空叶子结点（如果某个场景出问题就要接着写样式去覆盖那个场景，纯纯的手动挡懂我意思吧？），而且会被换行符之类的东西影响。

```scss
// styles/highlight.scss
@mixin FlashingCursor {
  animation: blink 1s steps(5, start) infinite;
  content: '▋';
  margin-left: 0.25rem;
  vertical-align: baseline;
}

@keyframes blink {
  0%,
  100% {
    opacity: 0;
  }

  25%,
  50% {
    opacity: 1;
  }
}

.chat-markdown-body-rendering {
  &:empty:after {
    content: '思考中...';
    animation: blink 1s steps(5, start) infinite;
    margin-left: 0.25rem;
    vertical-align: baseline;
  }

  // 打字机光标 锁定位置主要看这里
  > :not(ol):not(ul):not(pre):last-child:after,
  > pre:last-child code:after,
  > ol:last-child li:last-child:after,
  > ul:last-child li:last-child:after {
    @include FlashingCursor;
  }
}
```

## 总结

总体来说，效果其实不难，主要是`markdown-it`和`highlight.js`要仔细看看文档，这2个包好像`vitepress`也在用，就算有坑前面应该都有人帮填上了，所以放心使用。

打字机光标应该还有别的实现方案，可以去`bilibili`上搜`CSS 打字机`，我记得很多年前就刷到过。

## 参考资料·鸣谢

+ [ChatGPTNextWeb](https://github.com/ChatGPTNextWeb/ChatGPT-Next-Web)
+ [markdown-it](https://github.com/markdown-it/markdown-it)
+ [highlight.js (highlightjs.org)](https://highlightjs.org/)
+ [highlight.js中文网 (fenxianglu.cn)](https://fenxianglu.cn/highlight.html)
+ https://juejin.cn/post/7237426124669157433
