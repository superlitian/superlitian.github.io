---
title: HJ4 字符串分隔

description: 输入一个字符串，请按长度为8拆分每个输入字符串并进行输出；长度不是8整数倍的字符串请在后面补数字0，空字符串不处理。

slug: HJ4

date: 2023-01-17

categories:
- Huawei🌼

tags:
- Go
- Huawei🌼
- Easy😄
---

### 输入描述

连续输入字符串(每个字符串长度小于等于100)

### 输出描述

依次输出所有分割后的长度为8的新字符串

### 示例

#### 输入

**abc**

#### 输出

**abc00000**

### code

```go
package easy

import (
	"fmt"
	"math"
	"strings"
)

func HJ4(str string) string {
	if str == "" {
		return ""
	}
	var build strings.Builder
	length := int(math.Ceil(float64(len(str)) / 8.0))
	arr := make([]string, length)
	// len(str) <= 8
	if len(str) <= 8 {
		n := 8 - len(str)
		build.WriteString(str)
		for n > 0 {
			build.WriteString("0")
			n--
		}
		return build.String()
	}
	p1 := 0
	index := 0
	p2 := 7
	for p1 < len(str) {
		build.WriteString(string(str[p1]))
		if p1 == p2 {
			arr[index] = build.String()
			build.Reset()
			index++
			p2 = p1 + 8
		}
		p1++
	}
	if p1 < p2 {
		c := p2 - p1
		for c >= 0 {
			build.WriteString("0")
			c--
		}
		arr[index] = build.String()
	}
	res := ""
	for _, v := range arr {
		res += fmt.Sprintf("%s\n", v)
	}
	fmt.Println(arr)
	return res
}
```

