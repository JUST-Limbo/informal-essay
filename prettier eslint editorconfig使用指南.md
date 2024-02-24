## 前言

技术文章，尤其是前端技术文章具有时效性。

如文中提到的部分内容出现*break change*或出现内容错误（文字错误/错误的理论描述），为尽可能避免对后面的读者造成困扰，如果可以的话，希望在文章的评论区或代码仓库issues中予以指正，十分感谢。

相关仓库地址：

[文章所在仓库](https://github.com/JUST-Limbo/informal-essay)

## 摘要

## 背景

之前面试被问到了，从对面的反应来看答得似乎不是很好，**于是有了这篇文章。**



## settings.json

```text
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
  },
  "eslint.validate": ["typescript", "javascript", "vue"]
}
```

`editor.formatOnSave`

在保存时格式化文件

`editor.formatOnSaveMode`

控制在保存时设置格式是设置整个文件格式还是仅设置修改内容的格式。仅当 "#editor.formatOnSave#" 处于启用状态时适用。

`editor.codeActionsOnSave`

`"editor.formatOnSave": true,`这个配置项主要就是用来做样式的格式化，而`"editor.codeActionsOnSave"`不仅局限于样式的格式化，还会做代码检查等其它工作。

[一文了解VsCode、Eslint、Prettier、Husky相关配置 - 掘金 (juejin.cn)](https://juejin.cn/post/7169889743486844965)

`eslint.validate`

An array of language ids which should be validated by ESLint.

## prettier和ESLint的区别

`ESLint`更是一种代码质量检查工具。通常用来检查代码的语法正确性，风格统一性。也可以通过使用`--fix`参数来实现代码修复。

`Prettier`更倾向于代码格式化。通常用来按照配置的规则对代码风格进行格式化，比如缩进，换行符等。它不会对代码的质量进行检查，即：不会检查代码的语法是否正确，风格是否符合要求。

## vscode

`prettier:require config`

prettier配置文件必须存在

## eslint

```bash
npm i eslint
```

初始化eslintrc文件

```
npx eslint --init
```

## eslint-plugin-prettier 和 eslint-config-prettier



## prettier

格式化优先级

prettier.rc > editorconfig > settings.json

[API · Prettier](https://prettier.io/docs/en/api.html#prettierresolveconfigfileurlorpath--options)

[Configuration File · Prettier](https://prettier.io/docs/en/configuration#editorconfig)



bracketSameLine

假设现在有一个Vue项目，并没有配置Eslint, Prettier和EditorConfig，那我们该如何实现代码规范呢？

https://juejin.cn/post/7114162786137358372

```js
// 配置文件 -> .prettierrc.js
module.exports = {
    // ------------------------最重要----------------------------
    // 最大长度80个字符
    printWidth: 80,
    // 使用单引号, 默认false(在jsx中配置无效, 默认都是双引号)
    singleQuote: true,
    // 行末分号, 默认true
    semi: false,
    // 尽可能使用尾随逗号（包括函数参数）,默认none,可选 none|es5|all
    // es5 包括es5中的数组、对象
    // all 包括函数对象等所有可选
    trailingComma: 'none',
    // ------------------------------------------------------------
    // 对象字面量的大括号间使用空格（默认true） { a:1 }
    bracketSpacing: true,
    // 箭头函数参数括号 默认avoid 可选 avoid| always
    // avoid 能省略括号的时候就省略 如x => x
    // always 总是有括号 如(x) => x
    arrowParens: 'always',
    // tab缩进大小,默认为2
    tabWidth: 2,
    // 使用tab缩进还是空格，默认false
    useTabs: false,
    // ------------------------------------------------------------
    // vue缩进脚本和样式
    vueIndentScriptAndStyle: false,
    // 在文件顶部插入一个特殊的 @format 标记，指定文件格式需要被格式化。
    insertPragma: false,
    // 行尾换行格式
    endOfLine: 'auto',
    // html空格敏感度
    htmlWhitespaceSensitivity: 'ignore',
    // jsx 中的 > 标签放在最后一行的末尾，而不是单独放在下一行 默认false
    // jsxBracketSameLine: true,
    // JSX双引号
    // jsxSingleQuote: false,
};

```



## 一些prettier规则

`bracketSameLine`



## vetur

[VS Code 中 Vetur、prettier、ESLint 使用-CSDN博客](https://blog.csdn.net/my_study_everyday/article/details/123542800)

[Vetur+Eslint+Prettier介绍 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/158883214)



参考

https://juejin.cn/post/6971783776221265927

