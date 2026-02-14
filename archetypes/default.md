---
title: "{{ replace .File.ContentBaseName "-" " " | title }}"
date: {{ .Date }}
draft: true
tags: []
categories: []
summary: ""
authors: []
---

<!-- 文章摘要（可选） -->
<!-- 在正文前用 <!--more--> 分隔摘要和正文 -->

## 引言

<!-- 文章开头内容 -->

## 主要内容

<!-- 正文内容 -->

### 代码示例

```c
// 嵌入式C语言示例
#include <stdio.h>

int main() {
    printf("Hello, Embedded World!\n");
    return 0;
}
```

## 总结

<!-- 文章总结 -->

---

**相关标签**：{{ range $index, $tag := .Params.tags }}{{ if $index }}, {{ end }}`{{ $tag }}`{{ end }}

**最后更新**：{{ .Lastmod.Format "2006-01-02 15:04" }}