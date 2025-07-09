# Go语言汇编和CGO

## 📚 学习目标
掌握Go语言的汇编语言基础和CGO的使用方法，理解跨语言调用的实现方式。

---

## 1. 汇编语言基础

### 1.1 Go汇编语言简介
- Go语言的汇编语言用于编写高性能代码。
- 汇编代码以`.s`文件形式存在。

### 1.2 汇编代码示例
```asm
TEXT ·Add(SB), NOSPLIT, $0
    MOVQ a+0(FP), AX
    MOVQ b+8(FP), BX
    ADDQ BX, AX
    MOVQ AX, ret+16(FP)
    RET
```

---

## 2. CGO简介

### 2.1 什么是CGO
- CGO是Go语言用于调用C代码的工具。
- CGO允许Go程序与C库进行交互。

### 2.2 使用CGO调用C代码
```go
package main

/*
#include <stdio.h>
void hello() {
    printf("Hello from C!\n");
}
*/
import "C"

func main() {
    C.hello()
}
```

---

## 3. CGO的使用方法

### 3.1 调用C函数
```go
package main

/*
#include <math.h>
double square(double x) {
    return x * x;
}
*/
import "C"
import "fmt"

func main() {
    result := C.square(3.0)
    fmt.Printf("3.0的平方: %.2f\n", result)
}
```

### 3.2 使用C结构体
```go
package main

/*
#include <stdlib.h>
typedef struct {
    int x;
    int y;
} Point;
*/
import "C"
import "fmt"

func main() {
    p := C.Point{x: 10, y: 20}
    fmt.Printf("Point: (%d, %d)\n", p.x, p.y)
}
```

---

## 4. CGO的注意事项

### 4.1 性能开销
- CGO调用会增加性能开销。
- 尽量减少CGO调用的频率。

### 4.2 内存管理
- C代码中的内存需要手动释放。
- 使用`C.free`释放动态分配的内存。

---

## 5. 综合案例：调用C库
```go
package main

/*
#include <stdlib.h>
#include <string.h>
char* reverse(const char* str) {
    int len = strlen(str);
    char* rev = (char*)malloc(len + 1);
    for (int i = 0; i < len; i++) {
        rev[i] = str[len - i - 1];
    }
    rev[len] = '\0';
    return rev;
}
*/
import "C"
import (
    "fmt"
    "unsafe"
)

func main() {
    input := "Go语言"
    cstr := C.CString(input)
    defer C.free(unsafe.Pointer(cstr))

    reversed := C.reverse(cstr)
    defer C.free(unsafe.Pointer(reversed))

    fmt.Printf("原始字符串: %s\n", input)
    fmt.Printf("反转字符串: %s\n", C.GoString(reversed))
}
```

---

## 6. 学习检查点

- [ ] 理解Go汇编语言的基本概念
- [ ] 掌握CGO的使用方法
- [ ] 能用CGO调用C函数和结构体
- [ ] 理解CGO的性能开销和内存管理
- [ ] 能用CGO实现跨语言调用

---

汇编和CGO是Go语言的高级特性，掌握这些基础将显著提升程序的性能和扩展能力。
