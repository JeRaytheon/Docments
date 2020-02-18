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

### 修改日志

[修改日志](./CHANGELOG.md)

