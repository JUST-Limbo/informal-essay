# 关于 Vue2 选择器组件开发思路的若干描述

## 前言

技术文章，尤其是前端技术文章具有时效性。

如文中提到的部分内容出现*break change*或出现内容错误，为尽可能避免对后面的读者造成困扰，请尽快在文章的评论区予以指正，十分感谢。

## 概述

在工作过程中发现，一部分前端开发人员在开发时，对于具有选择器特征的业务功能写出的代码从可读性的角度来讲不是很契合 Vue2 这个框架。

本文通过一段简单的案例代码为切入来简述：

- 如何通过`v-model`来改善选择器组件代码
- 如何通过`provide inject`封装通用选择器组件

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

这段代码最大的问题是`.item`元素视图层的激活状态`.active`类依赖于对象数组`tableData`中对象元素的`selected`属性。在业务开发中，`tableData`的数据是通过调接口请求而来，因此很大概率前端拿到的数据中并没有`selected`属性。

因此这个案例中，仅仅为了维护视图层状态就对源数据随意地新增一个属性是不合理的，且有更简洁的应对策略。

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

注意关于`.active`类的处理、`model`配置项的声明和`selectAddress`方法的实现。

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
  // 外层传入的v-model="selectedId",selectedId的值在组件内会指向prop value
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
      // 执行这行代码,将会修改外层的selectedId,值为传入的item.id
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

随着业务的增长，开发人员会遇到越来越多的选择器功能需求，不可避免的会出现以下问题：

1. 每个选择器自定义组件都要重复写一次支持自定义`v-model`的配置是不合理的
2. 如果一个选择器支持单选和多选，则每次开发组件都要重复实现一次选择的逻辑是不合理的
3. 如果选择器列表项内容相同，但是在不同场景下布局不同（列表的布局可能是横向/纵向排列，弹出层等），则需要根据场景做不同的处理，长期的场景堆积会造成组件难以维护

为了应对以上问题，可以参考`el-select`的思路，**将选择器的容器与具体的列表项内容分离**，即：

给出一个专门容纳列表项内容的**容器组件**`Selector`，其作用是保存最终的选择结果、承载单选多选场景，同时暴露出一个`customClass`属性来从`Selector`容器外对列表布局进行控制。

这样在开发选择器功能时，**只需要关注列表项内容组件的代码开发即可**。

下面给出简明代码（不完善，仅作为描述思路）：

_index.vue_

注意`Selector`和`SelectOption`两个组件的层级

```vue
<template>
  <div>
    <!-- e.g.1 -->
    <div>{{ selectedValue }}</div>
    <Selector v-model="selectedValue" custom-class="horizontal-list">
      <SelectOption1
        v-for="item in tableData"
        :key="item.id"
        :value="item.id"
        v-bind="item"
      ></SelectOption1>
    </Selector>
    <!-- e.g.2 -->
    <div>{{ selectedValue2 }}</div>
    <Selector v-model="selectedValue2" multiple custom-class="vertical-list">
      <SelectOption2
        v-for="item in tableData"
        :key="item.id"
        :value="item.id"
        v-bind="item"
      ></SelectOption2>
    </Selector>
  </div>
</template>

<script>
import Selector from "./Selector.vue";
import SelectOption1 from "./SelectOption1.vue";
import SelectOption2 from "./SelectOption2.vue";

export default {
  components: {
    Selector,
    SelectOption1,
    SelectOption2,
  },
  data() {
    return {
      selectedValue: "",
      selectedValue2: "",
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

<style lang="scss">
.horizontal-list {
  display: flex;
}
.vertical-list {
}
</style>
```

_Selector.vue_

注意`provide`、`onOptionSelect`

```vue
<template>
  <div :class="[customClass]">
    <slot></slot>
  </div>
</template>

<script>
export default {
  name: "Selector",
  inheritAttrs: false,
  props: {
    value: {
      required: true,
    },
    multiple: Boolean,
    customClass: String,
  },
  provide() {
    return {
      $Selector: this,
    };
  },
  created() {
    if (this.multiple && !Array.isArray(this.value)) {
      this.$emit("input", []);
    }
    if (!this.multiple && Array.isArray(this.value)) {
      this.$emit("input", "");
    }
  },
  methods: {
    onOptionSelect(option) {
      if (this.multiple) {
        const targetIndex = this.value.indexOf(option.value);
        const valueClone = this.value.slice();
        if (targetIndex > -1) {
          valueClone.splice(targetIndex, 1);
        } else {
          valueClone.push(option.value);
        }
        this.$emit("input", valueClone);
      } else {
        this.$emit("input", option.value);
      }
    },
    calcItemActive(itemValue) {
      if (this.multiple) {
        return this.value.includes(itemValue);
      } else {
        return this.value == itemValue;
      }
    },
  },
};
</script>
```

_SelectOption1.vue_

注意`active`计算属性

```vue
<template>
  <div class="item" @click="selectAddress" :class="{ active }">
    <div>{{ date }}</div>
    <div>{{ name }}</div>
    <div>{{ address }}</div>
  </div>
</template>

<script>
export default {
  name: "SelectOption1",
  inheritAttrs: false,
  inject: ["$Selector"],
  model: {
    prop: "value",
    event: "select",
  },
  props: {
    value: [String, Number],
    data: Array,
    date: {},
    name: {},
    address: {},
  },
  computed: {
    active() {
      return this.$Selector.calcItemActive(this.value);
    },
  },
  methods: {
    selectAddress() {
      this.$Selector.onOptionSelect({ value: this.value });
    },
  },
};
</script>

<style lang="scss" scoped>
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
</style>
```

_SelectOption2.vue_

大部分代码同`SelectOption1.vue`，样式略有不同

```vue
<template>
  <div class="item" @click="selectAddress" :class="{ active }">
    <div>{{ date }}</div>
    <div>{{ name }}</div>
    <div>{{ address }}</div>
  </div>
</template>

<script>
export default {
  name: "SelectOption2",
  inheritAttrs: false,
  inject: ["$Selector"],
  model: {
    prop: "value",
    event: "select",
  },
  props: {
    value: [String, Number],
    data: Array,
    date: {},
    name: {},
    address: {},
  },
  computed: {
    active() {
      return this.$Selector.calcItemActive(this.value);
    },
  },
  methods: {
    selectAddress() {
      this.$Selector.onOptionSelect({ value: this.value });
    },
  },
};
</script>

<style lang="scss" scoped>
.item {
  border: 1px dashed #d9d9d9;
  border-radius: 2px;
  border-radius: 6px;
  margin-bottom: 12px;
  &.active {
    background-color: #409eff;
  }
}
</style>
```

## 版本差异

在`vue@'<2.6'`的版本中，`v-model`与`v-bind="$attrs" v-on="$listeners"`的写法会导致`value`属性丢失。

详见：

1. [Should v-model work on components using both v-bind="$attrs" and v-on="$listeners"? · Issue #6216 · vuejs/vue (github.com)](https://github.com/vuejs/vue/issues/6216)
2. [v-model's value not in $attrs if value not defined as a prop · Issue #9330 · vuejs/vue (github.com)](https://github.com/vuejs/vue/issues/9330)
3. [Allow v-bind="$attrs" with v-on="$listeners" to work with v-model by chrisvfritz · Pull Request #6327 · vuejs/vue (github.com)](https://github.com/vuejs/vue/pull/6327)
4. [$attrs 与 v-model结合使用value未被接收问题 · Issue #6 · bienvenidoY/blog (github.com)](https://github.com/bienvenidoY/blog/issues/6)


## 结语

综上所述，在开发过程中，应该尽量避免松散地将选择器功能实现在页面这一层代码中。

从代码可读性和可维护性的角度来讲， 此类不直接影响业务的纯功能性代码都应该集中到一个组件里，同时暴露出`v-model`实现与父级组件数据双向通讯。

此外，在面对具有复杂布局的选择器功能时，考虑将选择器的列表容器和列表选项各自独立，或许能改善代码的可维护性。

## 参考资料链接

（待补充）
