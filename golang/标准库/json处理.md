# JSON处理

Go语言的`encoding/json`包提供了对JSON数据的编码（序列化）和解码（反序列化）功能，使得在Go程序中处理JSON数据变得简单高效。

## 基本序列化与反序列化

### 结构体与JSON的转换

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Person struct {
    Name    string `json:"name"`
    Age     int    `json:"age"`
    Email   string `json:"email,omitempty"`
    Address string `json:"-"` // 不进行序列化
}

func main() {
    // 结构体转JSON
    person := Person{
        Name:    "张三",
        Age:     30,
        Email:   "zhangsan@example.com",
        Address: "北京市朝阳区", // 不会被序列化
    }
    
    data, err := json.Marshal(person)
    if err != nil {
        fmt.Println("序列化错误:", err)
        return
    }
    
    fmt.Println("JSON:", string(data))
    // 输出: JSON: {"name":"张三","age":30,"email":"zhangsan@example.com"}
    
    // JSON转结构体
    jsonData := []byte(`{"name":"李四","age":25,"email":"lisi@example.com"}`)
    var newPerson Person
    
    err = json.Unmarshal(jsonData, &newPerson)
    if err != nil {
        fmt.Println("反序列化错误:", err)
        return
    }
    
    fmt.Printf("结构体: %+v\n", newPerson)
    // 输出: 结构体: {Name:李四 Age:25 Email:lisi@example.com Address:}
}
```

### 处理嵌套结构

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Address struct {
    Street string `json:"street"`
    City   string `json:"city"`
    ZIP    string `json:"zip"`
}

type Employee struct {
    Name    string   `json:"name"`
    Age     int      `json:"age"`
    Address Address  `json:"address"`
    Skills  []string `json:"skills"`
}

func main() {
    employee := Employee{
        Name: "王五",
        Age:  35,
        Address: Address{
            Street: "中关村大街",
            City:   "北京",
            ZIP:    "100080",
        },
        Skills: []string{"Go", "Python", "Docker"},
    }
    
    data, _ := json.Marshal(employee)
    fmt.Println(string(data))
    
    // 嵌套JSON反序列化
    jsonStr := `{
        "name": "赵六",
        "age": 28,
        "address": {
            "street": "人民路",
            "city": "上海",
            "zip": "200001"
        },
        "skills": ["Java", "Kubernetes", "AWS"]
    }`
    
    var newEmployee Employee
    json.Unmarshal([]byte(jsonStr), &newEmployee)
    
    fmt.Printf("姓名: %s\n", newEmployee.Name)
    fmt.Printf("城市: %s\n", newEmployee.Address.City)
    fmt.Printf("技能: %v\n", newEmployee.Skills)
}
```

## 高级JSON处理

### Map与JSON转换

```go
package main

import (
    "encoding/json"
    "fmt"
)

func main() {
    // Map转JSON
    userMap := map[string]interface{}{
        "name":  "张三",
        "age":   30,
        "admin": true,
        "contacts": map[string]string{
            "email": "zhangsan@example.com",
            "phone": "13800138000",
        },
    }
    
    data, _ := json.Marshal(userMap)
    fmt.Println(string(data))
    
    // JSON转Map
    jsonStr := `{
        "name": "李四",
        "age": 25,
        "skills": ["Go", "MySQL"],
        "address": {
            "city": "北京",
            "district": "海淀区"
        }
    }`
    
    var result map[string]interface{}
    json.Unmarshal([]byte(jsonStr), &result)
    
    fmt.Printf("姓名: %s\n", result["name"])
    fmt.Printf("年龄: %.0f\n", result["age"])
    
    skills := result["skills"].([]interface{})
    fmt.Printf("第一个技能: %s\n", skills[0])
    
    address := result["address"].(map[string]interface{})
    fmt.Printf("城市: %s\n", address["city"])
}
```

### 自定义JSON编码

```go
package main

import (
    "encoding/json"
    "fmt"
    "time"
)

type CustomTime time.Time

func (t CustomTime) MarshalJSON() ([]byte, error) {
    return json.Marshal(time.Time(t).Format("2006-01-02 15:04:05"))
}

func (t *CustomTime) UnmarshalJSON(data []byte) error {
    var timeStr string
    if err := json.Unmarshal(data, &timeStr); err != nil {
        return err
    }
    
    parsedTime, err := time.Parse("2006-01-02 15:04:05", timeStr)
    if err != nil {
        return err
    }
    
    *t = CustomTime(parsedTime)
    return nil
}

type Event struct {
    Title     string     `json:"title"`
    CreatedAt CustomTime `json:"created_at"`
}

func main() {
    event := Event{
        Title:     "Go会议",
        CreatedAt: CustomTime(time.Now()),
    }
    
    data, _ := json.Marshal(event)
    fmt.Println(string(data))
    
    jsonStr := `{"title":"技术分享会","created_at":"2023-06-15 14:30:00"}`
    var newEvent Event
    json.Unmarshal([]byte(jsonStr), &newEvent)
    
    fmt.Println("事件标题:", newEvent.Title)
    fmt.Println("创建时间:", time.Time(newEvent.CreatedAt).Format("2006/01/02"))
}
```

## 性能优化

### 使用json.Decoder和json.Encoder

```go
package main

import (
    "encoding/json"
    "os"
    "strings"
)

type User struct {
    ID   int    `json:"id"`
    Name string `json:"name"`
}

func main() {
    // Decoder示例 - 从reader读取
    jsonStr := `{"id":1,"name":"张三"}
                {"id":2,"name":"李四"}
                {"id":3,"name":"王五"}`
    
    reader := strings.NewReader(jsonStr)
    decoder := json.NewDecoder(reader)
    
    for {
        var user User
        if err := decoder.Decode(&user); err != nil {
            break
        }
        // 处理每个user对象
    }
    
    // Encoder示例 - 写入到writer
    users := []User{
        {ID: 1, Name: "张三"},
        {ID: 2, Name: "李四"},
    }
    
    encoder := json.NewEncoder(os.Stdout)
    for _, user := range users {
        encoder.Encode(user)
    }
}
```

### 使用json.RawMessage

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Message struct {
    Type    string          `json:"type"`
    Content json.RawMessage `json:"content"`
}

type TextContent struct {
    Text string `json:"text"`
}

type ImageContent struct {
    URL    string `json:"url"`
    Width  int    `json:"width"`
    Height int    `json:"height"`
}

func main() {
    // 解析不同类型的消息
    jsonStr := `[
        {"type":"text","content":{"text":"这是一条文本消息"}},
        {"type":"image","content":{"url":"https://example.com/image.jpg","width":800,"height":600}}
    ]`
    
    var messages []Message
    json.Unmarshal([]byte(jsonStr), &messages)
    
    for _, msg := range messages {
        fmt.Printf("消息类型: %s\n", msg.Type)
        
        if msg.Type == "text" {
            var content TextContent
            json.Unmarshal(msg.Content, &content)
            fmt.Printf("文本内容: %s\n", content.Text)
        } else if msg.Type == "image" {
            var content ImageContent
            json.Unmarshal(msg.Content, &content)
            fmt.Printf("图片URL: %s, 尺寸: %dx%d\n", content.URL, content.Width, content.Height)
        }
    }
}
```

## JSON最佳实践

1. 使用适当的标签控制JSON字段名称和行为
2. 对于可选字段，使用指针类型或omitempty标签
3. 处理大型JSON文件时，使用流式处理（Decoder/Encoder）而非一次性加载
4. 对于未知结构的JSON，使用map[string]interface{}或json.RawMessage
5. 进行数值转换时注意JSON中的数字默认为float64
6. 使用泛型简化类型转换和处理
7. 对于特殊的日期时间格式，实现MarshalJSON/UnmarshalJSON接口
8. 考虑使用第三方库提高性能（如：jsoniter、easyjson）
