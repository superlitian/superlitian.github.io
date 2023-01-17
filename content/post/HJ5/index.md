---
title: HJ5 进制转换

description: 写出一个程序，接受一个十六进制的数，输出该数值的十进制表示。

slug: HJ5

date: 2023-01-17

categories:
- Huawei🌼

tags:
- Go
- Huawei🌼
- Easy😄
---

### 数据范围

$$ 1≤n≤2^{31}−1 $$

### 输入描述

输入一个十六进制的数值字符串。

### 输出描述

输出该数值的十进制字符串。不同组的测试用例用\n隔开。

### 示例

#### 输入

**0e**

#### 输出

**14**

#### 说明

最后一个单词为nowcoder，长度为8

### code

```go
package easy

import (
	"strconv"
)

func HJ5(str string) int {
	res, err := strconv.ParseInt(str, 16, 32)
	if err != nil {
		panic(err)
	}
	return int(res)
}
```

