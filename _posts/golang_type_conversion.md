---
title: Go类型转换
toc: true
comments: true
date: 2017-07-11 11:28:39
tags: 类型转换
categories: golang
---

golang中string与其他类型的转换

<!--more-->

### 1. 转换方式

```go
func main() {
	var i32Val int = 1
	var i64Val int64 = 1
	var f32Val float32 = 3.14
	var f64Val float64 = 3.14
	var strI32Val string = "1"
	var strI64Val string = "1"
	var strF32Val string = "3.14"
	var strF64Val string = "3.14"

	//int转string
	fmt.Println(strconv.Itoa(i32Val))
	//int64转string
	fmt.Println(strconv.FormatInt(i64Val, 10)) //第二个参数为进制
	//float32转string
	fmt.Println(strconv.FormatFloat(float64(f32Val), 'f', 6, 32)) //详见strconv包
	//float64转string
	fmt.Println(strconv.FormatFloat(f64Val, 'f', 6, 64))

	//string转int
	fmt.Println(strconv.Atoi(strI32Val))
	//string转int64
	fmt.Println(strconv.ParseInt(strI64Val, 10, 64))
	//string转float32
	fmt.Println(strconv.ParseFloat(strF32Val, 32))
	fmt.Println(strconv.ParseFloat(strF64Val, 64))
}
```

### 2. strconv包详解

strconv包主要有两组重要函数：**strconv.ParseXXX** 和**strconv.FormatXXX**

#### 2.1 strconv.FormatXXX函数(XXX转string)

- strconv.FormatInt函数

  函数原型：`func FormatInt(i int64, base int) string`

  - i：需要转换的数字
  - base：进制[2, 36]

- strconv.FormatInt

  函数原型：`func FormatFloat(f float64, fmt byte, prec, bitSize int) string`

  - f：需要转换的数字
  - byte：格式标志
    - 'b'：-ddddp±ddd，二进制指数
    - 'e'：-d.dddde±dd，十进制指数
    - 'E'：-d.ddddE±dd，十进制指数
    - 'f'：-ddd.dddd，没有指数
  - prec：精度
  - bitSize：32或者64

#### 2.2 strconv.ParseXXX函数(string转XXX)

- strconv.FormatInt

  函数原型：`func ParseInt(s string, base int, bitSize int) (i int64, err error)`

  - s：需要转换的string
  - base：进制
  - bitSize：32或者64
  - err：err.Err = ErrSyntax或者err.Err = ErrRange

- strconv.FormatFloat

  函数原型：`func ParseFloat(s string, bitSize int) (float64, error)`

  - s：需要转换的string
  - bitSize：32或者64
  - err：err.Err = ErrSyntax或者err.Err = ErrRange

  ​