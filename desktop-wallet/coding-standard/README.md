# 编码规范

- [编码规范](#%e7%bc%96%e7%a0%81%e8%a7%84%e8%8c%83)
    - [TS / JS](#ts--js)
    - [less](#less)
    - [HTML / Vue](#html--vue)
    - [修改日志](#%e4%bf%ae%e6%94%b9%e6%97%a5%e5%bf%97)

### TS / JS

- 变量名、属性名、方法名使用小驼峰命名法。（首字母小写）。

```js
let networkType;
const formItems;

function getUserList() {
    // ...
}
```
- 文件名使用大驼峰命名法。（首字母大写）。

```bash
FormAccountCreation.vue
FormAccountCreation.less
FormAccountCreation.ts
```

- 文件夹名使用小驼峰命名法。

```bash
forms
modals
```

- 类名使用大驼峰命名法。（首字母大写）。

```js
export class MyClass {
    // ...
}
```

- 字符串使用单引号，避免使用双引号

```js
const test = 'hello'

import { MyClass } from './MyClass'
```

- 行末一律不用冒号

```js
const test = 123
```

- 行级注释在双斜杠后需要一个空格

```js
//这是一行错误的注释 (错误)

// 这是一行正确的注释 (正确)
```

- 尽量避免使用`@Watch`，尽量使用computed（计算）属性处理相关动态数据。

```js
get userName() {
    return this.name
}
```

- 组件的参数声明应该将`@Prop`与参数变量紧挨在相邻两行，且先于其他类型变量进行声明

```js

export class MyComponent extends Vue {
    @Prop()
    param1: string // 与下方保持一个空行

    @Prop()
    param2: number // 与下方保持一个空行

    myList: string[]
}
```

### less

- class名尽量使用小写，单词之间使用短横线`-`连接。

```css
.top-window {
    /* ... */
}
```

- 颜色尽量使用定义了的变量。（变量存储在`/src/views/resources/css/variables.less`中）

```css
.nem-logo {
    background-color: @whiteLight; /* @whiteLight在variables.less中进行定义 */
}
```

### HTML / Vue

- 顶层包裹容器class名应为`*-wrapper`。

```html
<template>
    <div class="user-list-wrapper">
        <!-- ... -->
    </div>
</template>
```

- 一个完整的Vue文件包含下面2部分

```html
<template>
    <!-- template部分 -->
</template>

<!-- script部分 -->
<script lang="ts">
import { MyComponentTs } from './MyComponentTs' // 引入ts文件类
import './MyComponent.less' // 引入less文件
export default class MyComponent extends MyComponentTs{} // 继承ts代码
</script>
```

其中ts部分代码写在同目录下的`MyComponentTs.ts`文件中。尽量避免写在vue文件中。vue文件尽量保证只是对ts文件代码的继承。

### 修改日志

[修改日志](./CHANGELOG.md)

