---
title: 博客全功能测试
published: 2025-10-10
image: './120.jpg'
tags: [Markdown, Blogging, Demo, Test]
description: A simple example of a Markdown blog post.
category: Examples
draft: false
---

# 博客全功能测试

这是一篇专门用来测试博客系统 Markdown 功能的文章。内容涵盖基础语法、扩展语法、代码高亮、提示块、视频、公式等。你可以逐一确认渲染是否正确。

---

## 1. Draft 草稿功能

当文章还在草稿状态时，可以在 frontmatter 设置：

```markdown
---
title: Draft Example
published: 2024-01-11
tags: [Markdown, Blogging, Demo]
category: Examples
draft: true
---
```

当 `draft: true` 时，文章不会对外公开显示。准备发布时改成 `false` 即可。

---

## 2. 基础 Markdown 功能

### 段落与强调
普通文字、*斜体*、**粗体**、`行内代码`。  
[这是一个链接](https://example.com)。  

### 列表
- 无序列表项一
- 无序列表项二
  - 嵌套子项

1. 有序列表一
2. 有序列表二
3. 有序列表三

### 引用
> 这是一个引用示例  
> 可以换行

### 表格
| 尺寸 | 材料   | 颜色   |
|------|--------|--------|
| 9    | 皮革   | 棕色   |
| 10   | 麻布   | 自然色 |
| 11   | 玻璃   | 透明   |

---

## 3. 代码与语法高亮 (Expressive Code)

```js
console.log('Hello, Markdown!');
```

带文件名的代码块：

```js title="my-test-file.js"
console.log('带标题的代码块');
```

终端代码块：

```bash
echo "这是一个终端示例"
```

支持 diff 标记：

```diff
+ 新增一行
- 删除一行
```

支持行标记：

```js {2,4}
function test() {
  console.log("这一行会被高亮");
  return true;
}
```

---

## 4. 折叠与行号

```js collapse={1-3} showLineNumbers
// 这部分会折叠并显示行号
function calc(a, b) {
  return a + b;
}
```

---

## 5. 扩展 Markdown 语法

### Admonitions 提示块

:::note
这是一个 Note 提示块。
:::

:::tip
这是一个 Tip 提示块。
:::

:::warning
这是一个 Warning 提示块。
:::

### Spoiler
这是一个剧透 :spoiler[隐藏的内容]。

---

## 6. GitHub 仓库卡片

::github{repo="saicaca/fuwari"}

---

## 7. 数学公式

行内公式：$E = mc^2$  

块级公式：

$$I = \int \rho R^{2} dV$$

---

## 8. 视频嵌入

### YouTube

<iframe width="100%" height="360" src="https://www.youtube.com/embed/5gIf0_xpFPI" frameborder="0" allowfullscreen></iframe>

### Bilibili

<iframe width="100%" height="360" src="//player.bilibili.com/player.html?bvid=BV1fK4y1s7Qf&p=1" frameborder="0" allowfullscreen></iframe>
