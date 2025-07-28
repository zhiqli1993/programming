# Python JSON处理

JSON(JavaScript Object Notation)是一种轻量级的数据交换格式，在Python中通过内置的`json`模块可以轻松实现JSON数据的编码和解码。本文详细介绍Python中JSON处理的相关知识。

## JSON基础概念

JSON是一种文本格式，语法源自JavaScript，但已成为独立于语言的数据格式。JSON的基本数据类型包括：

- 数字(整数或浮点数)
- 字符串(双引号包围)
- 布尔值(true或false)
- 数组(有序值的集合，用方括号表示)
- 对象(键值对的集合，用花括号表示)
- null

JSON的优势在于：
- 易于人阅读和编写
- 易于机器解析和生成
- 基于文本，与语言无关
- 轻量级，传输效率高

## Python中的JSON模块

Python标准库提供了`json`模块用于处理JSON数据。

```python
import json
```

### Python与JSON类型对应关系

Python数据类型在转换为JSON时的映射关系：

| Python类型 | JSON类型 |
|------------|----------|
| dict | object |
| list, tuple | array |
| str | string |
| int, float | number |
| True | true |
| False | false |
| None | null |

## JSON编码(序列化)

将Python对象转换为JSON字符串的过程称为编码或序列化。

### 基本编码方法

```python
# 将Python字典转换为JSON字符串
person = {
    "name": "张三",
    "age": 30,
    "city": "北京",
    "languages": ["Python", "JavaScript", "Go"],
    "is_programmer": True,
    "height": 175.5,
    "spouse": None
}

# 转换为JSON字符串
json_str = json.dumps(person)
print(json_str)
# 输出: {"name": "张三", "age": 30, "city": "北京", "languages": ["Python", "JavaScript", "Go"], "is_programmer": true, "height": 175.5, "spouse": null}

# 直接写入文件
with open("person.json", "w", encoding="utf-8") as f:
    json.dump(person, f)
```

### 格式化输出

```python
# 美化输出，使用缩进
pretty_json = json.dumps(person, indent=4)
print(pretty_json)
"""
{
    "name": "张三",
    "age": 30,
    "city": "北京",
    "languages": [
        "Python",
        "JavaScript",
        "Go"
    ],
    "is_programmer": true,
    "height": 175.5,
    "spouse": null
}
"""

# 排序键
sorted_json = json.dumps(person, sort_keys=True)

# 自定义分隔符
custom_json = json.dumps(person, separators=(',', ':'))  # 压缩
```

### 处理中文和特殊字符

```python
# 确保中文字符不被转义
json_str = json.dumps(person, ensure_ascii=False)
print(json_str)
# 输出中文字符而非Unicode编码

# 写入文件时指定编码
with open("person.json", "w", encoding="utf-8") as f:
    json.dump(person, f, ensure_ascii=False, indent=4)
```

## JSON解码(反序列化)

将JSON字符串转换回Python对象的过程称为解码或反序列化。

### 基本解码方法

```python
# 从字符串解码
json_str = '{"name": "张三", "age": 30, "city": "北京"}'
data = json.loads(json_str)
print(data["name"])  # 输出: 张三

# 从文件读取并解码
with open("person.json", "r", encoding="utf-8") as f:
    data = json.load(f)
    print(data)
```

### 自定义解码器

默认情况下，JSON解码会将所有对象转换为Python字典。可以通过指定`object_hook`参数来自定义解码行为。

```python
# 定义一个Person类
class Person:
    def __init__(self, name, age, city=None):
        self.name = name
        self.age = age
        self.city = city
    
    def __repr__(self):
        return f"Person(name={self.name}, age={self.age}, city={self.city})"

# 自定义解码函数
def as_person(dct):
    if "name" in dct and "age" in dct:
        return Person(dct["name"], dct["age"], dct.get("city"))
    return dct

# 使用自定义解码器
json_str = '{"name": "张三", "age": 30, "city": "北京"}'
person = json.loads(json_str, object_hook=as_person)
print(person)  # 输出: Person(name=张三, age=30, city=北京)
```

## 处理复杂对象

JSON不直接支持Python中的一些复杂类型，如日期时间、集合、自定义类等。需要进行特殊处理。

### 日期时间处理

```python
import datetime

# 自定义编码器
class DateTimeEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, (datetime.datetime, datetime.date)):
            return obj.isoformat()
        return super().default(obj)

# 使用自定义编码器
data = {
    "name": "张三",
    "birthday": datetime.date(1990, 5, 15),
    "last_login": datetime.datetime.now()
}

json_str = json.dumps(data, cls=DateTimeEncoder)
print(json_str)

# 解码日期时间
def decode_datetime(dct):
    for key, value in dct.items():
        if isinstance(value, str):
            try:
                dct[key] = datetime.datetime.fromisoformat(value)
            except ValueError:
                try:
                    dct[key] = datetime.date.fromisoformat(value)
                except ValueError:
                    pass
    return dct

decoded_data = json.loads(json_str, object_hook=decode_datetime)
print(decoded_data)
```

### 自定义类的序列化

```python
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary
    
    def to_json(self):
        return {
            "name": self.name,
            "salary": self.salary
        }

# 使用自定义方法序列化
emp = Employee("张三", 10000)
json_str = json.dumps(emp.to_json())
print(json_str)

# 使用JSONEncoder子类
class CustomEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, Employee):
            return {
                "name": obj.name,
                "salary": obj.salary
            }
        return super().default(obj)

json_str = json.dumps(emp, cls=CustomEncoder)
print(json_str)
```

### 处理集合类型

```python
# 集合不能直接转换为JSON
my_set = {1, 2, 3}

# 方法1：转换为列表
json_str = json.dumps(list(my_set))

# 方法2：自定义编码器
class SetEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, set):
            return list(obj)
        return super().default(obj)

data = {"numbers": {1, 2, 3, 4, 5}}
json_str = json.dumps(data, cls=SetEncoder)
print(json_str)
```

## 实际应用案例

### 配置文件处理

```python
# 读取配置
def load_config(config_file="config.json"):
    try:
        with open(config_file, "r", encoding="utf-8") as f:
            return json.load(f)
    except (FileNotFoundError, json.JSONDecodeError) as e:
        print(f"配置文件错误: {e}")
        return {}

# 保存配置
def save_config(config, config_file="config.json"):
    try:
        with open(config_file, "w", encoding="utf-8") as f:
            json.dump(config, f, indent=4, ensure_ascii=False)
        return True
    except Exception as e:
        print(f"保存配置失败: {e}")
        return False

# 使用
config = load_config()
config["theme"] = "dark"
save_config(config)
```

### 处理API数据

```python
import requests

# 获取API数据
def get_api_data(url):
    try:
        response = requests.get(url)
        response.raise_for_status()  # 检查HTTP响应状态
        return response.json()  # 自动解析JSON响应
    except requests.RequestException as e:
        print(f"API请求失败: {e}")
        return None
    except json.JSONDecodeError:
        print("无效的JSON响应")
        return None

# 使用示例 - 获取GitHub用户信息
user_data = get_api_data("https://api.github.com/users/username")
if user_data:
    print(f"用户名: {user_data.get('name')}")
    print(f"仓库数: {user_data.get('public_repos')}")
```

### 数据缓存

```python
def cache_data(data, cache_file="cache.json"):
    """将数据缓存到JSON文件"""
    with open(cache_file, "w", encoding="utf-8") as f:
        json.dump({
            "timestamp": datetime.datetime.now().isoformat(),
            "data": data
        }, f, ensure_ascii=False)

def load_cached_data(cache_file="cache.json", max_age_seconds=3600):
    """从缓存加载数据，如果缓存过期则返回None"""
    try:
        with open(cache_file, "r", encoding="utf-8") as f:
            cache = json.load(f)
        
        cached_time = datetime.datetime.fromisoformat(cache["timestamp"])
        age = (datetime.datetime.now() - cached_time).total_seconds()
        
        if age <= max_age_seconds:
            return cache["data"]
        return None
    except (FileNotFoundError, json.JSONDecodeError, KeyError):
        return None
```

## JSON Schema验证

JSON Schema是一种用于验证JSON数据结构的规范。Python可以通过第三方库如`jsonschema`来实现。

```python
# 安装: pip install jsonschema
from jsonschema import validate, ValidationError

# 定义schema
person_schema = {
    "type": "object",
    "required": ["name", "age"],
    "properties": {
        "name": {"type": "string"},
        "age": {"type": "integer", "minimum": 0},
        "email": {"type": "string", "format": "email"},
        "phone": {"type": "string", "pattern": "^\\d{11}$"}
    }
}

# 验证数据
person = {
    "name": "张三",
    "age": 30,
    "email": "zhangsan@example.com",
    "phone": "13812345678"
}

try:
    validate(instance=person, schema=person_schema)
    print("验证通过")
except ValidationError as e:
    print(f"验证失败: {e}")
```

## JSON vs. 其他序列化方式

JSON与Python其他序列化方式的比较：

| 特性 | JSON | Pickle | MessagePack | YAML |
|------|------|--------|------------|------|
| 可读性 | 高 | 低(二进制) | 低(二进制) | 最高 |
| 性能 | 中等 | 高 | 最高 | 低 |
| 跨语言 | 是 | 否(Python专用) | 是 | 是 |
| 安全性 | 高 | 低(可能执行代码) | 高 | 中等 |
| 支持的数据类型 | 有限 | 几乎所有Python对象 | 有限但比JSON多 | 丰富 |
| 文件大小 | 中等 | 大 | 小 | 中等 |

## 性能优化技巧

处理大型JSON数据时的优化建议：

1. **使用streaming解析**：处理大型JSON文件时，可以使用`ijson`库实现流式解析，避免一次性加载整个文件。

```python
# 安装: pip install ijson
import ijson

# 逐项处理大型JSON文件
with open("large_file.json", "rb") as f:
    for item in ijson.items(f, "item"):
        process_item(item)
```

2. **压缩JSON**：减少空白字符

```python
compact_json = json.dumps(data, separators=(',', ':'))
```

3. **使用orjson或ujson**：更快的JSON解析库

```python
# 安装: pip install ujson
import ujson

# 比标准json更快
json_str = ujson.dumps(data)
data = ujson.loads(json_str)
```

4. **避免反复解析**：缓存解析结果

```python
# 缓存解析结果
parsed_data = {}

def get_parsed_data(json_file):
    if json_file not in parsed_data:
        with open(json_file) as f:
            parsed_data[json_file] = json.load(f)
    return parsed_data[json_file]
```

## 常见问题和解决方案

### 1. 循环引用问题

```python
# 循环引用会导致序列化失败
a = {}
b = {}
a["b"] = b
b["a"] = a

try:
    json.dumps(a)  # 这会引发错误
except RecursionError as e:
    print(f"错误: {e}")

# 解决方案：自定义编码器处理循环引用
class CyclicEncoder(json.JSONEncoder):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.seen = set()
        
    def default(self, obj):
        obj_id = id(obj)
        if obj_id in self.seen:
            return "<circular reference>"
        self.seen.add(obj_id)
        if isinstance(obj, dict):
            return {key: self.default(value) for key, value in obj.items()}
        return super().default(obj)

json_str = json.dumps(a, cls=CyclicEncoder)
print(json_str)
```

### 2. NaN和Infinity处理

```python
# JSON不支持NaN和Infinity
data = {"value": float("nan"), "limit": float("inf")}

try:
    json.dumps(data)  # 这会引发错误
except ValueError as e:
    print(f"错误: {e}")

# 解决方案
json_str = json.dumps(data, allow_nan=False, default=lambda x: str(x) if isinstance(x, float) and (math.isnan(x) or math.isinf(x)) else x)
print(json_str)
```

### 3. 大整数精度问题

```python
# JavaScript中大整数可能丢失精度
big_num = {"id": 9007199254740993}  # 超过JS的Number.MAX_SAFE_INTEGER

# 解决方案：将大整数转为字符串
json_str = json.dumps(big_num, default=lambda x: str(x) if isinstance(x, int) and x > 9007199254740991 else x)
print(json_str)
```

## 总结

Python的`json`模块提供了一套完整的工具来处理JSON数据：

1. **基础操作**：`dumps()`/`loads()`用于字符串转换，`dump()`/`load()`用于文件操作
2. **格式化**：可以控制缩进、排序和分隔符
3. **自定义处理**：通过扩展`JSONEncoder`和使用`object_hook`可以处理复杂对象
4. **高级应用**：支持配置文件、API交互、数据缓存等场景

掌握JSON处理技术对于现代Python应用程序开发至关重要，特别是在Web服务、API开发和数据交换方面。通过选择合适的序列化策略和优化技巧，可以在性能和可用性之间取得良好的平衡。
