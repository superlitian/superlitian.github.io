---
title: HJ2 计算某字符出现次数

description: 写出一个程序，接受一个由字母、数字和空格组成的字符串，和一个字符，然后输出输入字符串中该字符的出现次数。（不区分大小写字母）

slug: HJ2

date: 2023-01-17

categories:
- Huawei🌼

tags:
- Go
- Huawei🌼
- Easy😄
---
### 数据范围

**1≤n≤1000**

### 输入描述

第一行输入一个由字母、数字和空格组成的字符串，第二行输入一个字符（保证该字符不为空格）。

### 输出描述

输出输入字符串中含有该字符的个数。（不区分大小写字母）

### 示例

#### 输入

**ABCabc**
**A**

#### 输出

**2**

### code

```go
package easy

import "strings"

func HJ2(str, target string) int {
	s := strings.ToLower(str)
	tar := strings.ToLower(target)
	count := 0
	for _, v := range s {
		if string(v) == tar {
			count++
		}
	}
	return count
}
```

