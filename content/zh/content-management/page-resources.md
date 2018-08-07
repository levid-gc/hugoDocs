---
title : "页面资源"
description : "页面资源 —— 图像，其他页面，文档，等。 —— 拥有页面相关的 URL 和它们的元数据。"
date: 2018-01-24
categories: ["content management"]
keywords: [bundle,content,resources]
weight: 4003
draft: false
toc: true
linktitle: "页面资源"
menu:
  docs:
    parent: "content-management"
    weight: 31
---

## 属性

ResourceType
: 资源主要类型。举个例子，一个文件的 MIME 类型为 `image/jpg`，那么它的 ResourceType 就是 `image`。

Name
: 默认值为文件名（相对于本身页面）。可以在文件头中设置。

Title
: 默认值为空。可以在文件头中设置。

Permalink
: 资源的绝对 URL。`page` 资源类型没有值。

RelPermalink
: 资源的相对 URL。`page` 资源类型没有值。

## 方法

ByType
: 根据给定类型返回页面资源。

```go
{{ .Resources.ByType "image" }}
```

Match
: 返回所有 `Name` 匹配给定 Glob 模式的所有页面资源（[示例](https://github.com/gobwas/glob/blob/master/readme.md)）。匹配是大小写不敏感的。

```go
{{ .Resources.Match "images/*" }}
```

GetMatch
: 与 `Match` 相同，但是返回的是第一个匹配到的结果。

### 模式匹配

```go
// 使用 Match/GetMatch 来查找 images/sunset.jpg
.Resources.Match "images/sun*" ✅
.Resources.Match "**/Sunset.jpg" ✅
.Resources.Match "images/*.jpg" ✅
.Resources.Match "**.jpg" ✅
.Resources.Match "*" 🚫
.Resources.Match "sunset.jpg" 🚫
.Resources.Match "*sunset.jpg" 🚫

```

## 页面资源元数据

页面资源的元数据是在它们的文件头中管理的，使用的是一个名为 `resources` 的数组/表格参数。您可以使用一个 [通配符](http://tldp.org/LDP/GNU-Linux-Tools-Summary/html/x11655.htm) 来批量设置值。

{{% note %}}
`page` 资源类型的 `Title` 等属性是从它们自己的文件头中获取。
{{% /note %}}

name
: 设置 `Name` 返回的值。

{{% warning %}}
`Match` 和 `GetMatch` 方法使用 `Name` 来匹配资源。
{{%/ warning %}}

title
: 设置 `Title` 返回的值。

params
: 一个自定义的键/值对映射。

### 资源元数据示例

{{< code-toggle copy="false">}}
title: Application
date : 2018-01-25
resources :
- src : "images/sunset.jpg"
  name : "header"
- src : "documents/photo_specs.pdf"
  title : "Photo Specifications"
  params:
    icon : "photo"
- src : "documents/guide.pdf"
  title : "Instruction Guide"
- src : "documents/checklist.pdf"
  title : "Document Checklist"
- src : "documents/payment.docx"
  title : "Proof of Payment"
- src : "**.pdf"
  name : "pdf-file-:counter"
  params :
    icon : "pdf"
- src : "**.docx"
  params :
    icon : "word"
{{</ code-toggle >}}

上面的例子中：

- `sunset.jpg` 将会得到一个新的 `Name` 并且现在可以通过 `.GetMatch "header"` 获取。
- `documents/photo_specs.pdf` 将会获取 `photo` 图标。
- `documents/checklist.pdf`，`documents/guide.pdf` 和 `documents/payment.docx` 都将得到通过 `title` 设置的 `Title`。
- Bundle 中的 `PDF`，除 `documents/photo_specs.pdf` 外都将获取 `pdf` 图标。
- 所有的 `PDF` 文件都将拥有一个新的 `Name`。`name` 参数包含了一个特殊的占位符 [`:counter`](#the-counter-placeholder-in-name-and-title)，所以 `Name` 将会是 `pdf-file-1`，`pdf-file-2`，`pdf-file-3`。
- Bundle 中的每一个 docx 都将获取 `word` 图标。

{{% warning %}}
__顺序很重要__ —— `title`，`name` 和 `params`-**键** 只有 **首次设置** 的值将会被使用。后续的参数只在没有被设置的情况下才会被设置。举个例子，上面的例子中，在 `src = "documents/photo_specs.pdf"` 中 `.Params.icon` 已经被设置为 `"photo"` 了。所以它将不会被 `src = "**.pdf"` 规则中的 `"pdf"` 所覆盖。
{{%/ warning %}}

### `name` 和 `title` 中的 `:counter` 占位符

`:counter` 是 `name` 和 `title` 中的一个特殊占位符。

不管是在 `name` 还是 `title` 中，只要是第一次使用，那么它将从 1 开始计数。

举个例子，如果一个 bundle 有 `photo_specs.pdf`，`other_specs.pdf`，`guide.pdf` 和 `checklist.pdf` 这些资源，并且文件头的 `resources` 如下所示：

{{< code-toggle copy="false">}}
[[resources]]
  src = "*specs.pdf"
  title = "Specification #:counter"
[[resources]]
  src = "**.pdf"
  name = "pdf-file-:counter"
{{</ code-toggle >}}

the `Name` and `Title` will be assigned to the resource files as follows:

| Resource file     | `Name`            | `Title`               |
|-------------------|-------------------|-----------------------|
| checklist.pdf     | `"pdf-file-1.pdf` | `"checklist.pdf"`     |
| guide.pdf         | `"pdf-file-2.pdf` | `"guide.pdf"`         |
| other\_specs.pdf  | `"pdf-file-3.pdf` | `"Specification #1"` |
| photo\_specs.pdf  | `"pdf-file-4.pdf` | `"Specification #2"` |
