# 关于 Vue2 选择器组件的开发思路

## 前言

技术文章，尤其是前端技术文章具有时效性。

如文中提到的部分内容出现*break change*或出现内容错误，为尽可能避免对后面的读者造成困扰，请尽快在文章的评论区/仓库中予以指正，十分感谢。

文章仓库地址：[JUST-Limbo/informal-essay (github.com)](https://github.com/JUST-Limbo/informal-essay)

## 概述

在工作过程中发现，很多前端开发人员在开发时，对于具有选择器特征的业务功能写出的代码从可读性的角度来讲不是很契合 Vue2 这个框架。

本文通过一段简单的案例代码为切入来简述：

- 如何通过`v-model`来改善选择器组件代码
- 如何通过`provide inject`封装选择器功能通用组件

## 案例代码和解析

```vue
<template>
  <div>
    <div class="list">
      <div
        v-for="(item, index) in tableData"
        :key="index"
        class="item"
        @click="selectAddress(item)"
        :class="{ active: item.selected }"
      >
        <div>{{ item.date }}</div>
        <div>{{ item.name }}</div>
        <div>{{ item.address }}</div>
      </div>
    </div>
    <div>{{ selectId }}</div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      selectId: "",
      tableData: [
        {
          id: 1,
          date: "2016-05-02",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1518 弄",
        },
        {
          id: 2,
          date: "2016-05-04",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1517 弄",
        },
        {
          id: 3,
          date: "2016-05-01",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1519 弄",
        },
      ],
    };
  },
  methods: {
    selectAddress(item) {
      this.tableData.forEach((tableItem) => {
        this.$set(tableItem, "selected", false);
      });
      this.selectId = item.id;
      this.$set(item, "selected", true);
    },
  },
};
</script>

<style lang="scss">
.list {
  display: flex;
  .item {
    border: 1px dashed #d9d9d9;
    border-radius: 2px;
    width: 178px;
    height: 178px;
    border-radius: 6px;
    margin-left: 6px;
    &.active {
      background-color: #409eff;
    }
  }
}
</style>
```

这段代码最大的问题是`.item`元素视图层的激活状态依赖于对象数组`tableData`中对象元素的`selected`属性。在业务开发中，`tableData`的数据是通过调接口请求而来，因此很大概率前端拿到的数据中并没有`selected`属性。

在这个案例中，仅仅为了维护视图层的状态就新增一个属性是不合理的，因为有更简洁的应对策略。

**可以对案例代码进行以下修改：**

```vue
<template>
  <div>
    <div class="list">
      <div
        v-for="(item, index) in tableData"
        :key="index"
        class="item"
        @click="selectAddress(item)"
        :class="{ active: selectId == item.id }"
      >
        ...略,
      </div>
    </div>
    ...略,
  </div>
</template>

<script>
export default {
  ...略,
  methods: {
    selectAddress(item) {
      this.selectId = item.id;
    },
  },
};
</script>

<style lang="scss">
...略
</style>
```

上述代码中第 9 行，激活态`.active`类的生效条件由原来的`item.selected`改为`selectId == item.id`，同时`selectAddress`函数中也不再对数组元素的`selected`属性进行赋值操作。

## 支持 v-model 的选择器组件

实际上这类选择器的功能，无非是单选多选，也可以参考 Vue 对`input、radio`等表单控件的处理策略，通过使用自定义组件的`v-model`改善选择器组件代码。

> 自定义组件想要使用 v-model 需要进行一些非常简单的配置，如不知道如何配置，那么可以参考以下资料：
>
> - [API — Vue.js (vuejs.org)](https://v2.cn.vuejs.org/v2/api/#model)
> - [自定义事件 — Vue.js (vuejs.org)](https://v2.cn.vuejs.org/v2/guide/components-custom-events.html#自定义组件的-v-model)

**可以对案例代码进行以下修改：**

_index.vue_

注意第 4 行的代码变化

```vue
<template>
  <div>
    <div>{{ selectId }}</div>
    <AddressSelect v-model="selectId" :data="tableData" />
  </div>
</template>

<script>
import AddressSelect from "./AddressSelect.vue";
export default {
  components: {
    AddressSelect,
  },
  data() {
    return {
      selectId: "",
      tableData: [
        {
          id: 1,
          date: "2016-05-02",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1518 弄",
        },
        {
          id: 2,
          date: "2016-05-04",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1517 弄",
        },
        {
          id: 3,
          date: "2016-05-01",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1519 弄",
        },
      ],
    };
  },
};
</script>
```

_AddressSelect.vue_

注意第 8、20-32 行的代码变化

```vue
<template>
  <div class="list">
    <div
      v-for="(item, index) in data"
      :key="index"
      class="item"
      @click="selectAddress(item)"
      :class="{ active: value == item.id }"
    >
      <div>{{ item.date }}</div>
      <div>{{ item.name }}</div>
      <div>{{ item.address }}</div>
    </div>
  </div>
</template>

<script>
export default {
  name: "AddressSelect",
  // 外层传入的v-model="selectedId",selectedId的值在组件内指向prop value
  model: {
    prop: "value",
    event: "select",
  },
  props: {
    value: [String, Number],
    data: Array,
  },
  methods: {
    selectAddress(item) {
      // 执行这段代码,将会修改外层的selectedId,值为传入的item.id
      this.$emit("select", item.id);
    },
  },
};
</script>
<style lang="scss">
.list {
  display: flex;
  .item {
    border: 1px dashed #d9d9d9;
    border-radius: 2px;
    width: 178px;
    height: 178px;
    border-radius: 6px;
    margin-left: 6px;
    &.active {
      background-color: #409eff;
    }
  }
}
</style>
```

## 通过 provide inject 将选择容器与具体的列表项内容分离

随着业务的增长，开发人员会遇到越来越多的选择器功能需求，如果对每个选择器自定义组件都要做一次`v-model`的配置显然是不合理的。

为了应对这一问题，可以参考`el-select`的思路**将选择容器与具体的列表项内容分离**，这样在开发选择器功能时，只需要关注列表项内容的代码开发即可。至于选择器最终的结果值、是否多选、容器的样式等都集中到容器组件中进行开发。

期望的使用案例是这样的：

```vue
<template>
  <!-- e.g.1 -->
  <Selector v-model="selectedValue" custom-class="list">
    <SelectOption1
      v-for="item in tableData"
      :key="item.value"
      :value="item.value"
      v-bind="item"
    ></SelectOption1>
  </Selector>
  <!-- e.g.2 -->
  <Selector v-model="selectedValue2" custom-class="list">
    <SelectOption2
      v-for="item in tableData"
      :key="item.value"
      :value="item.value"
      v-bind="item"
    ></SelectOption2>
  </Selector>
</template>
```

## 结语

综上所述，在开发过程中，应该尽量避免松散地将选择器功能实现在页面这一层代码中。从代码可读性和可维护性的角度来讲， 此类不直接影响业务的纯功能性代码都应该集中到一个组件里，同时暴露出`v-model`同父级实现双向数据的通讯。

> 本着**不抱怨，想办法**的原则写下这篇文章，希望看完以后少生产点文章开头案例中的低质代码，尽量避免给后续接手维护代码的开发人员造成困扰。

## 相关链接

**return 0;**
