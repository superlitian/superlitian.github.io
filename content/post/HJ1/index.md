---
title: HJ1 字符串最后一个单词的长度

description: 计算字符串最后一个单词的长度，单词以空格隔开，字符串长度小于5000。（注：字符串末尾不以空格为结尾）

slug: HJ1

date: 2023-01-17

categories:
- Huawei🌼

tags:
- Go
- Huawei🌼
- Easy😄
---

### 输入描述

输入一行，代表要计算的字符串，非空，长度小于5000。

### 输出描述

输出一个整数，表示输入字符串最后一个单词的长度。

### 示例

#### 输入

**hello world**

#### 输出

**5**

#### 说明

最后一个单词为world，长度为5

### code

```go
package easy

import "strings"

func HJ1(str string) int {
	arr := strings.Split(str, " ")
	return len(arr[len(arr)-1])
}
```

