## 前言

技术文章，尤其是前端技术文章具有时效性。

如文中提到的部分内容出现*break change*或出现内容错误（文字错误/错误的理论描述），为尽可能避免对后面的读者造成困扰，如果可以的话，希望在文章的评论区或代码仓库issues中予以指正，十分感谢。

相关仓库地址：

[文章所在仓库](https://github.com/JUST-Limbo/informal-essay)

## 概述

在业务开发中，选择地址、商品或标签之类的需求很常见。如果直接在页面里维护选中状态，代码虽然能跑，但往往没有充分利用 Vue 2 的组件机制，后续也不容易复用。

本文从一个简单示例出发，讨论两个问题：

- 如何用 `v-model` 简化选择器组件的状态传递
- 如何用 `provide/inject` 拆分选择容器和选项组件

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

这段代码的问题在于，`.item` 的激活状态依赖 `tableData` 中每一项的 `selected` 属性。实际项目里的 `tableData` 通常来自接口，而接口数据未必包含这个字段。

如果只是为了控制选中样式，就没有必要修改每一项数据。当前选中的 ID 已经保存在 `selectId` 中，直接用它判断即可。

代码可以改成这样：

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

现在，`.active` 直接由 `selectId == item.id` 决定，`selectAddress` 只负责更新 `selectId`，不再改动 `tableData`。

## 支持 v-model 的选择器组件

这类组件本质上是在维护一个单选值或一组多选值，可以参考 Vue 处理表单控件的方式，用自定义组件的 `v-model` 对外传递选中结果。

> 如果不熟悉 Vue 2 自定义组件的 `v-model`，可以先看以下资料：
>
> - [API — Vue.js (vuejs.org)](https://v2.cn.vuejs.org/v2/api/#model)
> - [自定义事件 — Vue.js (vuejs.org)](https://v2.cn.vuejs.org/v2/guide/components-custom-events.html#自定义组件的-v-model)

接下来把前面的代码拆成页面组件和 `AddressSelect` 组件。

### `index.vue`

页面只保留数据和选中结果，通过 `v-model` 接收 `AddressSelect` 的值：

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

### `AddressSelect.vue`

组件内部需要处理三件事：声明 `model` 配置、根据 `value` 判断激活状态，以及在点击后触发约定的事件。

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
  // 外层使用 v-model="selectedId" 时，selectedId 会作为 value 传入组件
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
      // 触发 select 事件后，外层的 selectedId 会更新为 item.id
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

## 用 provide/inject 拆分选择容器和选项

如果项目里有多种选择器，继续为每个业务场景单独写一个组件，通常会遇到这些问题：

1. 每个组件都要重复配置 `v-model`。
2. 单选、多选的状态更新逻辑会反复出现。
3. 选项内容相同，但横向排列、纵向排列或弹出层等布局不同。把这些场景都塞进一个业务组件后，维护成本会逐渐增加。

可以参考 `el-select` 的组件结构，把选择逻辑和选项内容拆开：

- `Selector` 负责接收选中值，处理单选和多选逻辑，并通过 `customClass` 开放布局样式。
- `SelectOption` 负责渲染具体内容，并把点击行为交给 `Selector` 处理。

两者通过 `provide/inject` 建立联系。新增业务选择器时，通常只需要实现选项的内容和样式。

下面的代码用于说明整体思路，省略了类型校验、禁用状态等生产环境中可能需要的细节。

### `index.vue`

`SelectOption1` 和 `SelectOption2` 都放在 `Selector` 内部，但可以使用不同的内容和布局：

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

### `Selector.vue`

`Selector` 通过 `provide` 暴露自身。选项组件注入该实例后，就可以读取选中状态并调用统一的选择方法。

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

### `SelectOption1.vue`

`active` 由 `Selector` 统一计算，点击时再把当前选项的值交回 `Selector`：

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

### `SelectOption2.vue`

它与 `SelectOption1.vue` 的交互逻辑相同，只调整展示内容或样式：

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

在低于 2.6 的 Vue 版本中，同时使用 `v-model`、`v-bind="$attrs"` 和 `v-on="$listeners"` 时，可能出现 `value` 属性无法传递的问题。

详见：

1. [Should v-model work on components using both v-bind="$attrs" and v-on="$listeners"? · Issue #6216 · vuejs/vue (github.com)](https://github.com/vuejs/vue/issues/6216)
2. [v-model's value not in $attrs if value not defined as a prop · Issue #9330 · vuejs/vue (github.com)](https://github.com/vuejs/vue/issues/9330)
3. [Allow v-bind="$attrs" with v-on="$listeners" to work with v-model by chrisvfritz · Pull Request #6327 · vuejs/vue (github.com)](https://github.com/vuejs/vue/pull/6327)
4. [$attrs 与 v-model结合使用value未被接收问题 · Issue #6 · bienvenidoY/blog (github.com)](https://github.com/bienvenidoY/blog/issues/6)


## 结语

选择器逻辑直接写在页面里并非一定错误。场景简单、没有复用需求时，这样写反而更直接。但当单选、多选和多种布局反复出现后，就值得把选中状态和交互逻辑收进组件。

通过 `v-model`，父组件只需要关心选中结果；通过 `provide/inject`，容器组件可以统一处理选择逻辑，选项组件则专注于内容和样式。

这种拆分会增加组件之间的隐式依赖，因此不必套用到所有场景。是否采用，还是要看选择逻辑的复用程度和布局复杂度。
