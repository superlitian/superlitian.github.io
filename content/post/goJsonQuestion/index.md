---
title: go json 序列化问题

description: go json 序列化特殊字符处理方案

slug: goJsonQuestion

date: 2022-09-29

categories:
- Go

tags:
- Go
- Json
---

### 两种方法解决

- 将value 转为 16进制或base64，服务端单独做一层解码

  ```go
  	bt := []byte{230,136}
  	hexStr := hex.EncodeToString(bt)
  	args := map[string]string{}
  	args["a"] = hexStr
  
  	fmt.Printf("a %v\n",bt)
  	println("len a", len(bt))
  
  	input, err := json.Marshal(args)
  	if err != nil {
  		fmt.Println(err)
  	}
  
  	var decodeArgs map[string]string
  
  	err =json.Unmarshal(input, &decodeArgs)
  	if err != nil {
  		return
  	}
  
  	b, err := hex.DecodeString(decodeArgs["a"])
  
  	if err != nil {
  		return
  	}
  	fmt.Println("b", b)
  	fmt.Println("len b", len(b))
  
  // output
  a [230 136]
  len a 2
  b [230 136]
  len b 2
  ```



- 服务端声明一个对应的接收struct

  ```go
  type  tb struct {
  	Bytes []byte
  }
  
  
  func test() {
  	bt := []byte{230,136}
  	args := tb{Bytes: bt}
  	fmt.Printf("a %v\n",bt)
  	println("len a", len(bt))
  
  	input, err := json.Marshal(args)
  	if err != nil {
  		fmt.Println(err)
  	}
  
  	var decodeArgs tb
  
  	err =json.Unmarshal(input, &decodeArgs)
  	if err != nil {
  		return
  	}
  
  	if err != nil {
  		return
  	}
  	fmt.Println("b", decodeArgs.Bytes)
  	fmt.Println("len b", len(decodeArgs.Bytes))
  }
  
  // output
  a [230 136]
  len a 2
  b [230 136]
  len b 2
  ```

  