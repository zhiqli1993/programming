# IO操作

Go语言提供了丰富的IO操作接口和实现，主要分布在`io`、`io/ioutil`、`os`和`bufio`等包中。Go的IO系统设计简洁而强大，基于接口组合，使得代码可以更好地复用和扩展。

## 核心接口

### io包中的基本接口

```go
package main

import (
    "fmt"
    "io"
    "strings"
)

func main() {
    // io.Reader接口示例
    r := strings.NewReader("Hello, Go IO!")
    buf := make([]byte, 8)
    
    for {
        n, err := r.Read(buf)
        if err == io.EOF {
            break
        }
        if err != nil {
            fmt.Println("读取错误:", err)
            break
        }
        
        fmt.Printf("读取了 %d 字节: %s\n", n, buf[:n])
    }
    
    // io.Writer接口示例
    var builder strings.Builder
    n, err := builder.Write([]byte("写入数据测试"))
    if err != nil {
        fmt.Println("写入错误:", err)
        return
    }
    
    fmt.Printf("写入了 %d 字节, 结果: %s\n", n, builder.String())
}
```

### io/fs包的文件系统接口 (Go 1.16+)

```go
package main

import (
    "fmt"
    "io/fs"
    "os"
    "path/filepath"
    "time"
)

func main() {
    // 使用os.DirFS创建一个文件系统
    fsys := os.DirFS(".")
    
    // 遍历文件系统
    fs.WalkDir(fsys, ".", func(path string, d fs.DirEntry, err error) error {
        if err != nil {
            return err
        }
        
        info, _ := d.Info()
        fmt.Printf("路径: %-20s 大小: %-8d 修改时间: %s\n", 
            path, 
            info.Size(), 
            info.ModTime().Format(time.RFC3339))
        
        return nil
    })
}
```

## 文件操作

### 基本文件读写

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    // 写入文件
    file, err := os.Create("example.txt")
    if err != nil {
        fmt.Println("创建文件错误:", err)
        return
    }
    
    data := []byte("Hello, Go文件操作!\n这是第二行内容。")
    n, err := file.Write(data)
    if err != nil {
        fmt.Println("写入错误:", err)
        file.Close()
        return
    }
    
    fmt.Printf("写入了 %d 字节到文件\n", n)
    file.Close()
    
    // 读取文件
    readFile, err := os.Open("example.txt")
    if err != nil {
        fmt.Println("打开文件错误:", err)
        return
    }
    defer readFile.Close()
    
    buf := make([]byte, 128)
    for {
        n, err := readFile.Read(buf)
        if err != nil {
            if err != os.ErrClosed && err.Error() != "EOF" {
                fmt.Println("读取错误:", err)
            }
            break
        }
        
        fmt.Printf("读取内容: %s", buf[:n])
    }
}
```

### 使用os包的高级文件操作

```go
package main

import (
    "fmt"
    "os"
    "time"
)

func main() {
    // 文件信息
    fileInfo, err := os.Stat("example.txt")
    if err != nil {
        fmt.Println("获取文件信息错误:", err)
        return
    }
    
    fmt.Printf("文件名: %s\n", fileInfo.Name())
    fmt.Printf("大小: %d 字节\n", fileInfo.Size())
    fmt.Printf("权限: %s\n", fileInfo.Mode())
    fmt.Printf("修改时间: %s\n", fileInfo.ModTime())
    fmt.Printf("是目录: %t\n", fileInfo.IsDir())
    
    // 创建目录
    err = os.Mkdir("testdir", 0755)
    if err != nil && !os.IsExist(err) {
        fmt.Println("创建目录错误:", err)
        return
    }
    
    // 重命名文件
    err = os.Rename("example.txt", "renamed.txt")
    if err != nil {
        fmt.Println("重命名错误:", err)
        return
    }
    
    // 修改文件时间
    newTime := time.Now().Add(24 * time.Hour)
    err = os.Chtimes("renamed.txt", newTime, newTime)
    if err != nil {
        fmt.Println("修改时间错误:", err)
        return
    }
    
    // 删除文件
    err = os.Remove("renamed.txt")
    if err != nil {
        fmt.Println("删除文件错误:", err)
        return
    }
}
```

## 缓冲IO

### 使用bufio进行高效IO

```go
package main

import (
    "bufio"
    "fmt"
    "os"
    "strings"
)

func main() {
    // 缓冲写入
    file, _ := os.Create("buffered.txt")
    defer file.Close()
    
    writer := bufio.NewWriter(file)
    
    // 写入缓冲区
    writer.WriteString("第一行内容\n")
    writer.WriteString("第二行内容\n")
    fmt.Fprintf(writer, "格式化的第%d行内容\n", 3)
    
    // 将缓冲区内容写入文件
    writer.Flush()
    
    // 缓冲读取
    readFile, _ := os.Open("buffered.txt")
    defer readFile.Close()
    
    reader := bufio.NewReader(readFile)
    
    // 按行读取
    for {
        line, err := reader.ReadString('\n')
        if err != nil {
            break
        }
        fmt.Printf("读取行: %s", line)
    }
    
    // 扫描器
    fmt.Println("\n使用Scanner:")
    readFile.Seek(0, 0) // 回到文件开头
    scanner := bufio.NewScanner(readFile)
    
    for scanner.Scan() {
        fmt.Println(scanner.Text())
    }
    
    if err := scanner.Err(); err != nil {
        fmt.Println("扫描错误:", err)
    }
}
```

### 自定义Scanner分割函数

```go
package main

import (
    "bufio"
    "bytes"
    "fmt"
    "strings"
)

func main() {
    // 自定义分割函数 - 按逗号分割
    text := "apple,banana,orange,grape,melon"
    scanner := bufio.NewScanner(strings.NewReader(text))
    
    // 自定义分割函数
    scanner.Split(func(data []byte, atEOF bool) (advance int, token []byte, err error) {
        if atEOF && len(data) == 0 {
            return 0, nil, nil
        }
        
        if i := bytes.IndexByte(data, ','); i >= 0 {
            // 找到分隔符
            return i + 1, data[0:i], nil
        }
        
        // 如果到达EOF, 返回剩余数据
        if atEOF {
            return len(data), data, nil
        }
        
        // 请求更多数据
        return 0, nil, nil
    })
    
    // 使用自定义分割函数扫描
    for scanner.Scan() {
        fmt.Printf("水果: %s\n", scanner.Text())
    }
}
```

## 实用IO操作

### 读取整个文件内容

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    // 读取整个文件内容到内存
    data, err := os.ReadFile("buffered.txt")
    if err != nil {
        fmt.Println("读取文件错误:", err)
        return
    }
    
    fmt.Printf("文件内容:\n%s\n", string(data))
    
    // 写入整个文件
    content := []byte("这是一个新文件的全部内容。\n可以一次性写入。")
    err = os.WriteFile("newfile.txt", content, 0644)
    if err != nil {
        fmt.Println("写入文件错误:", err)
        return
    }
}
```

### IO工具函数

```go
package main

import (
    "fmt"
    "io"
    "os"
    "strings"
)

func main() {
    // 复制数据
    src := strings.NewReader("这是源数据")
    dst := &strings.Builder{}
    
    n, err := io.Copy(dst, src)
    if err != nil {
        fmt.Println("复制错误:", err)
        return
    }
    
    fmt.Printf("复制了 %d 字节: %s\n", n, dst.String())
    
    // 限制读取
    file, _ := os.Open("buffered.txt")
    defer file.Close()
    
    // 只读取前20个字节
    limited := io.LimitReader(file, 20)
    data, _ := io.ReadAll(limited)
    
    fmt.Printf("前20个字节: %s\n", string(data))
    
    // 多Reader串联
    r1 := strings.NewReader("第一部分 ")
    r2 := strings.NewReader("第二部分 ")
    r3 := strings.NewReader("第三部分")
    
    readers := io.MultiReader(r1, r2, r3)
    result, _ := io.ReadAll(readers)
    
    fmt.Printf("合并结果: %s\n", string(result))
}
```

### 临时文件和目录

```go
package main

import (
    "fmt"
    "os"
    "path/filepath"
)

func main() {
    // 创建临时目录
    tempDir, err := os.MkdirTemp("", "gotest-*")
    if err != nil {
        fmt.Println("创建临时目录错误:", err)
        return
    }
    defer os.RemoveAll(tempDir) // 结束时删除临时目录
    
    fmt.Printf("创建的临时目录: %s\n", tempDir)
    
    // 在临时目录中创建临时文件
    tempFile, err := os.CreateTemp(tempDir, "example-*.txt")
    if err != nil {
        fmt.Println("创建临时文件错误:", err)
        return
    }
    
    fmt.Printf("创建的临时文件: %s\n", tempFile.Name())
    
    // 写入临时文件
    tempFile.WriteString("这是临时文件内容")
    tempFile.Close()
    
    // 读取临时文件
    content, _ := os.ReadFile(tempFile.Name())
    fmt.Printf("临时文件内容: %s\n", string(content))
}
```

## IO性能优化

1. **使用缓冲IO**：对于频繁的小数据读写操作，使用`bufio`包提供的缓冲读写器可以显著提高性能。

2. **批量读写**：尽可能使用较大的缓冲区进行批量读写，减少系统调用次数。

3. **避免频繁打开关闭文件**：在需要多次访问同一文件时，保持文件句柄打开状态，减少打开关闭开销。

4. **使用`io.Copy`**：当需要将数据从一个源复制到目标时，使用`io.Copy`比手动循环更高效。

5. **使用`io.ReadAll`**：对于小文件，使用`io.ReadAll`一次性读取所有内容比循环读取更简单高效。

6. **使用适当的IO模式**：对于顺序访问，使用流式处理；对于随机访问，使用`os.File`的`Seek`方法。

7. **注意垃圾回收**：大量IO操作可能产生大量临时对象，合理使用对象池或预分配缓冲区可以减轻GC压力。

8. **并发IO**：利用goroutine可以实现并发IO，但要注意适当控制并发度，避免资源竞争。
