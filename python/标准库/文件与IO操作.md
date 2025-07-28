# Python文件与IO操作

文件操作是几乎所有编程语言中的基础功能，Python提供了丰富的文件和IO处理能力，使得读写文件、处理路径和操作文件系统变得简单而强大。本文详细介绍Python中的文件与IO操作，包括基本文件操作、路径处理、目录操作以及高级IO功能。

## 基础文件操作

### 打开和关闭文件

Python使用内置的`open()`函数打开文件，返回一个文件对象：

```python
# 基本语法
# open(file, mode='r', buffering=-1, encoding=None, errors=None, newline=None, closefd=True, opener=None)

# 打开文件进行读取（默认模式）
file = open('example.txt', 'r')
# 使用文件...
file.close()  # 完成后关闭文件
```

更好的做法是使用`with`语句（上下文管理器），它会自动处理文件的关闭：

```python
# 使用with语句（推荐）
with open('example.txt', 'r') as file:
    # 使用文件...
    pass  # 当代码块结束时，文件会自动关闭
```

### 文件打开模式

Python支持多种文件打开模式：

| 模式 | 描述 |
|------|------|
| `'r'` | 只读模式（默认） |
| `'w'` | 写入模式（会覆盖已有内容） |
| `'a'` | 追加模式（在文件末尾添加内容） |
| `'x'` | 创建模式（如果文件已存在则失败） |
| `'b'` | 二进制模式（与上述模式结合使用，如'rb'或'wb'） |
| `'t'` | 文本模式（默认，与上述模式结合使用，如'rt'或'wt'） |
| `'+'` | 更新模式（可读可写，与其他模式结合使用，如'r+'或'w+'） |

示例：

```python
# 只读模式
with open('example.txt', 'r') as file:
    content = file.read()

# 写入模式（覆盖已有内容）
with open('example.txt', 'w') as file:
    file.write('Hello, World!')

# 追加模式
with open('example.txt', 'a') as file:
    file.write('\nAppended line.')

# 二进制读取模式
with open('image.jpg', 'rb') as file:
    image_data = file.read()

# 二进制写入模式
with open('output.bin', 'wb') as file:
    file.write(b'\x00\x01\x02\x03')

# 读写模式
with open('example.txt', 'r+') as file:
    content = file.read()
    file.write('\nNew content')
```

### 文件编码

处理文本文件时，通常需要指定编码：

```python
# 指定UTF-8编码打开文件
with open('example.txt', 'r', encoding='utf-8') as file:
    content = file.read()

# 处理编码错误
with open('example.txt', 'r', encoding='utf-8', errors='ignore') as file:
    content = file.read()  # 忽略无法解码的字符

# 其他常见编码错误处理选项：
# 'strict'：默认值，编码错误时引发UnicodeError异常
# 'replace'：用替换字符（通常是）替换无法解码的字符
# 'surrogateescape'：使用Unicode代理对表示无法解码的字节
# 'xmlcharrefreplace'：使用XML字符引用替换无法解码的字符
```

### 读取文件内容

读取文件内容有多种方法：

```python
# 读取整个文件内容
with open('example.txt', 'r') as file:
    content = file.read()  # 读取所有内容到一个字符串
    print(content)

# 读取特定字节数
with open('example.txt', 'r') as file:
    chunk = file.read(10)  # 读取前10个字符
    print(chunk)

# 按行读取
with open('example.txt', 'r') as file:
    first_line = file.readline()  # 读取一行
    print(first_line)

# 读取所有行到列表
with open('example.txt', 'r') as file:
    lines = file.readlines()  # 返回所有行的列表
    for line in lines:
        print(line.strip())  # strip()移除行尾的换行符

# 迭代文件对象（内存效率高）
with open('example.txt', 'r') as file:
    for line in file:  # 文件对象是可迭代的，每次产生一行
        print(line.strip())
```

### 写入文件内容

写入文件内容的基本方法：

```python
# 写入字符串
with open('output.txt', 'w') as file:
    file.write('Hello, World!\n')
    file.write('This is a second line.')

# 写入多行
with open('output.txt', 'w') as file:
    lines = ['Line 1\n', 'Line 2\n', 'Line 3\n']
    file.writelines(lines)  # writelines()不会自动添加换行符

# 格式化写入
with open('output.txt', 'w') as file:
    name = 'Alice'
    age = 30
    file.write(f'Name: {name}, Age: {age}\n')

# 追加内容
with open('output.txt', 'a') as file:
    file.write('This line is appended.\n')
```

### 文件位置

可以使用`tell()`和`seek()`方法控制文件读写位置：

```python
with open('example.txt', 'r+') as file:
    # 获取当前位置
    position = file.tell()
    print(f"当前位置: {position}")
    
    # 读取一些内容
    content = file.read(5)
    print(f"读取内容: {content}")
    print(f"读取后位置: {file.tell()}")
    
    # 移动到文件开头
    file.seek(0)
    print(f"移动到开头后位置: {file.tell()}")
    
    # 移动到第10个字节
    file.seek(10)
    print(f"移动到第10个字节后位置: {file.tell()}")
    
    # 从当前位置向后移动5个字节
    file.seek(5, 1)  # 第二个参数表示参考点：0=开头，1=当前位置，2=结尾
    print(f"相对移动后位置: {file.tell()}")
    
    # 移动到文件末尾
    file.seek(0, 2)
    print(f"文件末尾位置: {file.tell()}")
```

## 路径处理

Python提供了多种处理文件路径的方式，包括传统的`os.path`模块和更现代的`pathlib`模块。

### 使用os.path模块

`os.path`模块提供了与操作系统无关的路径处理函数：

```python
import os.path

# 拼接路径
path = os.path.join('folder', 'subfolder', 'file.txt')
print(path)  # 在Windows上输出：folder\subfolder\file.txt
             # 在Linux/macOS上输出：folder/subfolder/file.txt

# 获取路径组件
dirname = os.path.dirname(path)  # 获取路径的目录部分
basename = os.path.basename(path)  # 获取路径的文件名部分
print(f"目录: {dirname}, 文件名: {basename}")

# 分离文件名和扩展名
filename, extension = os.path.splitext(basename)
print(f"文件名: {filename}, 扩展名: {extension}")

# 获取绝对路径
abs_path = os.path.abspath(path)
print(f"绝对路径: {abs_path}")

# 判断路径类型
is_file = os.path.isfile(path)  # 是否是文件
is_dir = os.path.isdir(dirname)  # 是否是目录
is_link = os.path.islink(path)  # 是否是符号链接
exists = os.path.exists(path)  # 路径是否存在
print(f"是文件: {is_file}, 是目录: {is_dir}, 是链接: {is_link}, 存在: {exists}")

# 获取文件信息
if os.path.exists(path):
    size = os.path.getsize(path)  # 文件大小（字节）
    mtime = os.path.getmtime(path)  # 最后修改时间（Unix时间戳）
    atime = os.path.getatime(path)  # 最后访问时间
    ctime = os.path.getctime(path)  # 创建时间（Windows）或元数据更改时间（Unix）
    print(f"大小: {size}字节, 修改时间: {mtime}")

# 规范化路径
norm_path = os.path.normpath('/path/to/../file.txt')
print(f"规范化路径: {norm_path}")  # 输出: /path/file.txt

# 获取公共路径前缀
prefix = os.path.commonpath(['/path/to/file1.txt', '/path/to/file2.txt'])
print(f"公共前缀: {prefix}")  # 输出: /path/to

# 获取相对路径
rel_path = os.path.relpath('/path/to/file.txt', '/path')
print(f"相对路径: {rel_path}")  # 输出: to/file.txt
```

### 使用pathlib模块

`pathlib`模块（Python 3.4+）提供了面向对象的路径处理方式，更加直观和强大：

```python
from pathlib import Path

# 创建路径对象
path = Path('folder') / 'subfolder' / 'file.txt'
print(path)  # PosixPath('folder/subfolder/file.txt') 或 WindowsPath('folder\\subfolder\\file.txt')

# 获取当前目录
current_dir = Path.cwd()
print(f"当前目录: {current_dir}")

# 获取用户主目录
home_dir = Path.home()
print(f"用户主目录: {home_dir}")

# 获取路径组件
print(f"父目录: {path.parent}")
print(f"文件名: {path.name}")
print(f"文件名不含扩展名: {path.stem}")
print(f"扩展名: {path.suffix}")
print(f"全部扩展名: {path.suffixes}")  # 处理多重扩展名，如 .tar.gz

# 拼接路径
new_path = path.parent / 'another_file.txt'
print(f"新路径: {new_path}")

# 获取绝对路径
abs_path = path.absolute()
print(f"绝对路径: {abs_path}")

# 获取相对路径
rel_path = path.relative_to(Path('folder'))
print(f"相对路径: {rel_path}")  # 输出: subfolder/file.txt

# 解析路径
resolved_path = path.resolve()  # 解析符号链接并规范化路径
print(f"解析后的路径: {resolved_path}")

# 路径判断
print(f"是文件: {path.is_file()}")
print(f"是目录: {path.is_dir()}")
print(f"是符号链接: {path.is_symlink()}")
print(f"是绝对路径: {path.is_absolute()}")
print(f"路径存在: {path.exists()}")

# 修改路径
print(f"修改扩展名: {path.with_suffix('.log')}")
print(f"修改文件名: {path.with_name('newname.txt')}")
print(f"修改父目录: {path.with_stem('newname')}")  # Python 3.9+

# 路径迭代
for part in path.parts:
    print(part)  # 依次输出: 'folder', 'subfolder', 'file.txt'

# 文件操作（无需使用open函数）
if path.exists():
    # 读取文本文件
    content = path.read_text(encoding='utf-8')
    
    # 读取二进制文件
    binary_data = path.read_bytes()
    
    # 写入文本文件
    path.write_text('Hello, World!', encoding='utf-8')
    
    # 写入二进制文件
    path.write_bytes(b'\x00\x01\x02\x03')
```

## 目录操作

Python提供了多种创建、遍历和操作目录的方法。

### 使用os模块操作目录

```python
import os
import shutil

# 获取当前工作目录
current_dir = os.getcwd()
print(f"当前工作目录: {current_dir}")

# 更改当前工作目录
os.chdir('/path/to/directory')
print(f"新的工作目录: {os.getcwd()}")

# 列出目录内容
entries = os.listdir('/path/to/directory')  # 不包括 . 和 ..
print(f"目录内容: {entries}")

# 创建目录
os.mkdir('new_directory')  # 创建单个目录
os.makedirs('parent/child/grandchild', exist_ok=True)  # 创建多级目录，exist_ok=True防止目录已存在时抛出错误

# 删除目录
os.rmdir('empty_directory')  # 只能删除空目录
shutil.rmtree('directory_tree')  # 递归删除目录树

# 重命名文件或目录
os.rename('old_name.txt', 'new_name.txt')
os.rename('old_directory', 'new_directory')

# 移动文件或目录
shutil.move('file.txt', '/path/to/destination')
shutil.move('source_directory', '/path/to/destination')

# 复制文件
shutil.copy('source.txt', 'destination.txt')  # 复制文件内容和权限
shutil.copy2('source.txt', 'destination.txt')  # 同时复制元数据（修改时间等）

# 复制目录
shutil.copytree('source_dir', 'destination_dir')  # 递归复制整个目录树

# 获取文件状态
stat_info = os.stat('file.txt')
print(f"文件大小: {stat_info.st_size} 字节")
print(f"最后修改时间: {stat_info.st_mtime}")
print(f"权限模式: {stat_info.st_mode}")

# 修改文件权限（类Unix系统）
os.chmod('file.txt', 0o755)  # 设置为可执行文件
```

### 使用pathlib操作目录

`pathlib`模块也提供了丰富的目录操作功能：

```python
from pathlib import Path
import shutil

# 创建目录
path = Path('new_directory')
path.mkdir(exist_ok=True)

# 创建多级目录
nested_path = Path('parent/child/grandchild')
nested_path.mkdir(parents=True, exist_ok=True)

# 获取当前目录
current_dir = Path.cwd()
print(f"当前目录: {current_dir}")

# 列出目录内容
dir_path = Path('/path/to/directory')
for entry in dir_path.iterdir():
    print(entry)

# 列出所有Python文件
python_files = list(dir_path.glob('*.py'))
print(f"Python文件: {python_files}")

# 递归列出所有Python文件
all_python_files = list(dir_path.rglob('*.py'))
print(f"所有Python文件: {all_python_files}")

# 检查目录是否存在
if dir_path.exists() and dir_path.is_dir():
    print(f"{dir_path} 是一个存在的目录")

# 删除文件
file_path = Path('file_to_delete.txt')
if file_path.exists():
    file_path.unlink()

# 删除空目录
empty_dir = Path('empty_directory')
if empty_dir.exists():
    empty_dir.rmdir()

# 删除目录树（需要使用shutil）
dir_to_remove = Path('directory_tree')
if dir_to_remove.exists():
    shutil.rmtree(dir_to_remove)

# 重命名或移动
old_path = Path('old_name.txt')
new_path = Path('new_name.txt')
if old_path.exists():
    old_path.rename(new_path)
```

### 遍历目录树

遍历目录树有多种方式：

```python
import os
from pathlib import Path

# 使用os.walk()
def walk_with_os(root_dir):
    for dirpath, dirnames, filenames in os.walk(root_dir):
        print(f"目录: {dirpath}")
        for dirname in dirnames:
            print(f"  子目录: {dirname}")
        for filename in filenames:
            print(f"  文件: {filename}")
            
# 使用pathlib递归遍历
def walk_with_pathlib(root_dir):
    root_path = Path(root_dir)
    for path in root_path.rglob('*'):
        if path.is_dir():
            print(f"目录: {path}")
        else:
            print(f"文件: {path}")

# 示例
walk_with_os('/path/to/directory')
walk_with_pathlib('/path/to/directory')
```

## 高级IO操作

### 文件对象方法

文件对象提供了多种方法来控制IO操作：

```python
with open('example.txt', 'r+') as file:
    # 获取文件描述符号
    fd = file.fileno()
    print(f"文件描述符: {fd}")
    
    # 刷新内部缓冲区
    file.flush()
    
    # 检查文件是否已关闭
    print(f"文件是否已关闭: {file.closed}")
    
    # 检查文件是否可读/可写
    print(f"可读: {file.readable()}")
    print(f"可写: {file.writable()}")
    
    # 截断文件到指定大小
    file.truncate(100)  # 截断到100字节
```

### 使用StringIO和BytesIO进行内存中的IO操作

`io`模块提供了在内存中进行IO操作的功能，无需创建临时文件：

```python
from io import StringIO, BytesIO

# 文本IO
text_io = StringIO()
text_io.write("Hello, World!\n")
text_io.write("This is a test.")
text_io.seek(0)  # 回到开头
content = text_io.read()
print(content)
text_io.close()

# 二进制IO
binary_io = BytesIO()
binary_io.write(b'\x00\x01\x02\x03')
binary_io.seek(0)
binary_data = binary_io.read()
print(binary_data)
binary_io.close()
```

### 使用tempfile模块创建临时文件和目录

```python
import tempfile
import os

# 创建临时文件
with tempfile.TemporaryFile(mode='w+t') as temp_file:
    # 写入数据
    temp_file.write('Hello, temporary file!')
    
    # 回到文件开头
    temp_file.seek(0)
    
    # 读取数据
    content = temp_file.read()
    print(content)
    
    # 文件会在关闭时自动删除

# 创建命名临时文件
with tempfile.NamedTemporaryFile(delete=False) as named_temp:
    print(f"临时文件名: {named_temp.name}")
    named_temp.write(b'This is a named temporary file.')

# 文件会保留，需要手动删除
os.unlink(named_temp.name)

# 创建临时目录
with tempfile.TemporaryDirectory() as temp_dir:
    print(f"临时目录: {temp_dir}")
    # 使用临时目录...
    # 目录会在退出with块时自动删除
```

### 使用gzip, bz2和zipfile处理压缩文件

Python内置了多种压缩文件格式的支持：

```python
import gzip
import bz2
import zipfile
import tarfile
import os
from pathlib import Path

# gzip文件操作
with gzip.open('file.txt.gz', 'wt') as f:
    f.write('Hello, gzip file!')

with gzip.open('file.txt.gz', 'rt') as f:
    content = f.read()
    print(f"gzip内容: {content}")

# bz2文件操作
with bz2.open('file.txt.bz2', 'wt') as f:
    f.write('Hello, bz2 file!')

with bz2.open('file.txt.bz2', 'rt') as f:
    content = f.read()
    print(f"bz2内容: {content}")

# zip文件操作
with zipfile.ZipFile('archive.zip', 'w') as zip_file:
    # 添加文件
    zip_file.write('file1.txt')
    zip_file.write('file2.txt')
    
    # 添加字符串作为文件
    zip_file.writestr('string_file.txt', 'This content is added directly as a string.')

# 列出zip文件内容
with zipfile.ZipFile('archive.zip', 'r') as zip_file:
    print(f"ZIP文件内容: {zip_file.namelist()}")
    
    # 解压单个文件
    zip_file.extract('file1.txt', path='extracted')
    
    # 解压所有文件
    zip_file.extractall(path='extracted')
    
    # 读取zip中的文件
    with zip_file.open('string_file.txt') as f:
        content = f.read().decode('utf-8')
        print(f"ZIP中的文件内容: {content}")

# tar文件操作
with tarfile.open('archive.tar.gz', 'w:gz') as tar:
    # 添加文件
    tar.add('file1.txt')
    tar.add('file2.txt')
    
    # 添加整个目录
    tar.add('directory')

# 列出tar文件内容
with tarfile.open('archive.tar.gz', 'r:gz') as tar:
    print("TAR文件内容:")
    for member in tar.getmembers():
        print(f"  {member.name} - {member.size} bytes")
    
    # 解压所有文件
    tar.extractall(path='extracted')
    
    # 读取tar中的文件
    member = tar.getmember('file1.txt')
    f = tar.extractfile(member)
    if f:
        content = f.read().decode('utf-8')
        print(f"TAR中的文件内容: {content}")
```

### 使用pickle进行对象序列化

`pickle`模块可以将Python对象序列化为字节流，以便保存到文件或通过网络传输：

```python
import pickle

# 要序列化的数据
data = {
    'name': 'Alice',
    'age': 30,
    'grades': [85, 90, 78],
    'info': {'address': '123 Main St', 'email': 'alice@example.com'}
}

# 将对象序列化到文件
with open('data.pickle', 'wb') as f:
    pickle.dump(data, f)

# 从文件反序列化对象
with open('data.pickle', 'rb') as f:
    loaded_data = pickle.load(f)
    print(f"加载的数据: {loaded_data}")

# 序列化到字节字符串
serialized = pickle.dumps(data)
print(f"序列化的数据: {serialized[:20]}... ({len(serialized)} bytes)")

# 从字节字符串反序列化
deserialized = pickle.loads(serialized)
print(f"反序列化的数据: {deserialized}")

# 注意：pickle不安全，不要反序列化不信任的数据
```

### 使用JSON处理数据

JSON是一种常用的数据交换格式，Python的`json`模块提供了JSON数据的编码和解码功能：

```python
import json

# Python数据
data = {
    'name': 'Alice',
    'age': 30,
    'is_student': False,
    'courses': ['Python', 'Web Development', 'Database'],
    'grades': {'Python': 'A', 'Web Development': 'B+'},
    'address': None
}

# 将Python对象转换为JSON字符串
json_string = json.dumps(data)
print(f"JSON字符串: {json_string}")

# 格式化JSON输出
pretty_json = json.dumps(data, indent=4, sort_keys=True)
print(f"格式化JSON:\n{pretty_json}")

# 将JSON字符串转换回Python对象
parsed_data = json.loads(json_string)
print(f"解析后的数据: {parsed_data}")

# 将Python对象写入JSON文件
with open('data.json', 'w') as f:
    json.dump(data, f, indent=4)

# 从JSON文件读取数据
with open('data.json', 'r') as f:
    file_data = json.load(f)
    print(f"从文件加载的数据: {file_data}")

# 自定义JSON编码
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

def person_encoder(obj):
    if isinstance(obj, Person):
        return {'name': obj.name, 'age': obj.age}
    raise TypeError(f"Object of type {type(obj)} is not JSON serializable")

person = Person('Bob', 35)
json_string = json.dumps(person, default=person_encoder)
print(f"自定义编码的JSON: {json_string}")

# 自定义JSON解码
def person_decoder(json_dict):
    if 'name' in json_dict and 'age' in json_dict:
        return Person(json_dict['name'], json_dict['age'])
    return json_dict

decoded_person = json.loads(json_string, object_hook=person_decoder)
print(f"解码后的对象: {decoded_person.name}, {decoded_person.age}")
```

## 文件和目录的常见操作示例

### 搜索和过滤文件

```python
import os
from pathlib import Path
import fnmatch
import re

# 使用os.walk查找所有Python文件
def find_python_files_with_os(root_dir):
    python_files = []
    for dirpath, dirnames, filenames in os.walk(root_dir):
        for filename in filenames:
            if filename.endswith('.py'):
                full_path = os.path.join(dirpath, filename)
                python_files.append(full_path)
    return python_files

# 使用pathlib查找所有Python文件
def find_python_files_with_pathlib(root_dir):
    return list(Path(root_dir).rglob('*.py'))

# 使用fnmatch进行模式匹配
def find_files_with_pattern(root_dir, pattern):
    matches = []
    for dirpath, dirnames, filenames in os.walk(root_dir):
        for filename in fnmatch.filter(filenames, pattern):
            full_path = os.path.join(dirpath, filename)
            matches.append(full_path)
    return matches

# 使用正则表达式查找文件
def find_files_with_regex(root_dir, regex_pattern):
    pattern = re.compile(regex_pattern)
    matches = []
    for dirpath, dirnames, filenames in os.walk(root_dir):
        for filename in filenames:
            if pattern.match(filename):
                full_path = os.path.join(dirpath, filename)
                matches.append(full_path)
    return matches

# 示例使用
print(find_python_files_with_os('/path/to/project'))
print(find_python_files_with_pathlib('/path/to/project'))
print(find_files_with_pattern('/path/to/project', '*.txt'))
print(find_files_with_regex('/path/to/project', r'.*\.py$'))
```

### 批量文件处理

```python
from pathlib import Path
import shutil
import os

# 批量重命名文件
def batch_rename(directory, old_ext, new_ext):
    for item in Path(directory).iterdir():
        if item.is_file() and item.suffix == old_ext:
            new_name = item.with_suffix(new_ext)
            item.rename(new_name)
            print(f"重命名: {item} -> {new_name}")

# 按文件类型分类整理文件
def organize_by_extension(directory):
    dir_path = Path(directory)
    
    # 获取所有文件
    files = [f for f in dir_path.iterdir() if f.is_file()]
    
    # 按扩展名分组
    for file in files:
        ext = file.suffix.lower().lstrip('.')
        if not ext:  # 没有扩展名
            ext = 'no_extension'
            
        # 创建扩展名对应的目录
        ext_dir = dir_path / ext
        ext_dir.mkdir(exist_ok=True)
        
        # 移动文件
        shutil.move(str(file), str(ext_dir / file.name))
        print(f"移动: {file} -> {ext_dir / file.name}")

# 查找并删除空目录
def remove_empty_dirs(directory):
    for dirpath, dirnames, filenames in os.walk(directory, topdown=False):
        if not dirnames and not filenames:
            print(f"删除空目录: {dirpath}")
            os.rmdir(dirpath)

# 查找重复文件
def find_duplicate_files(directory):
    file_dict = {}  # {file_size: [path1, path2, ...]}
    duplicates = []
    
    for dirpath, dirnames, filenames in os.walk(directory):
        for filename in filenames:
            path = os.path.join(dirpath, filename)
            try:
                file_size = os.path.getsize(path)
                if file_size in file_dict:
                    file_dict[file_size].append(path)
                else:
                    file_dict[file_size] = [path]
            except OSError:
                continue
    
    # 检查每组相同大小的文件
    for size, paths in file_dict.items():
        if len(paths) > 1:
            # 可以进一步比较文件内容确认是否真的相同
            duplicates.append(paths)
    
    return duplicates

# 示例使用
batch_rename('/path/to/files', '.txt', '.md')
organize_by_extension('/path/to/files')
remove_empty_dirs('/path/to/directory')
dups = find_duplicate_files('/path/to/files')
for dup_group in dups:
    print(f"可能的重复文件: {dup_group}")
```

### 文件监控

```python
import time
import os
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

class MyHandler(FileSystemEventHandler):
    def on_created(self, event):
        if not event.is_directory:
            print(f"创建文件: {event.src_path}")
    
    def on_deleted(self, event):
        if not event.is_directory:
            print(f"删除文件: {event.src_path}")
    
    def on_modified(self, event):
        if not event.is_directory:
            print(f"修改文件: {event.src_path}")
    
    def on_moved(self, event):
        if not event.is_directory:
            print(f"移动文件: {event.src_path} -> {event.dest_path}")

def monitor_directory(path):
    event_handler = MyHandler()
    observer = Observer()
    observer.schedule(event_handler, path, recursive=True)
    observer.start()
    
    try:
        print(f"开始监控目录: {path}")
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        observer.stop()
    observer.join()

# 示例使用（需要安装watchdog: pip install watchdog）
if __name__ == "__main__":
    monitor_directory('/path/to/watch')
```

## 最佳实践

### 安全地处理文件路径

```python
from pathlib import Path
import os

def safe_join_paths(base, *paths):
    """安全地拼接路径，防止路径遍历攻击"""
    base_path = Path(base).resolve()
    joined_path = base_path.joinpath(*paths).resolve()
    
    # 确保结果路径是base_path的子路径
    if not str(joined_path).startswith(str(base_path)):
        raise ValueError(f"路径遍历攻击: {joined_path} 不在 {base_path} 内")
    
    return joined_path

# 示例
try:
    # 正常情况
    path = safe_join_paths('/var/www', 'html', 'index.html')
    print(f"安全路径: {path}")
    
    # 尝试路径遍历攻击
    unsafe_path = safe_join_paths('/var/www', '../../../etc/passwd')
    print(f"不安全路径: {unsafe_path}")  # 这行不会执行，会抛出异常
except ValueError as e:
    print(f"错误: {e}")
```

### 正确处理文件编码

```python
import sys
import locale

def get_system_encoding():
    """获取系统默认编码"""
    print(f"默认编码: {sys.getdefaultencoding()}")
    print(f"文件系统编码: {sys.getfilesystemencoding()}")
    print(f"本地化编码: {locale.getpreferredencoding()}")

def read_file_safely(filename):
    """安全地读取可能包含多种编码的文件"""
    encodings = ['utf-8', 'latin-1', 'windows-1252', 'gbk', 'shift-jis']
    
    for encoding in encodings:
        try:
            with open(filename, 'r', encoding=encoding) as f:
                content = f.read()
            print(f"成功使用 {encoding} 编码读取文件")
            return content
        except UnicodeDecodeError:
            continue
    
    # 如果所有编码都失败，使用二进制模式读取
    print("所有编码尝试失败，以二进制模式读取")
    with open(filename, 'rb') as f:
        return f.read()

# 写入文件时指定正确的编码
def write_file_with_encoding(filename, content, encoding='utf-8'):
    with open(filename, 'w', encoding=encoding) as f:
        f.write(content)
    print(f"使用 {encoding} 编码写入文件")

# 示例
get_system_encoding()
content = read_file_safely('example.txt')
write_file_with_encoding('output.txt', 'Hello, 世界!', 'utf-8')
```

### 异常处理

```python
def robust_file_operations():
    """演示文件操作中的异常处理"""
    try:
        with open('non_existent.txt', 'r') as f:
            content = f.read()
    except FileNotFoundError:
        print("文件不存在")
    except PermissionError:
        print("没有权限读取文件")
    except IsADirectoryError:
        print("尝试读取的是一个目录，不是文件")
    except (IOError, OSError) as e:
        print(f"IO错误: {e}")
    
    # 尝试创建文件
    try:
        with open('/root/test.txt', 'w') as f:
            f.write("测试")
    except PermissionError:
        print("没有权限写入文件")
    except FileNotFoundError:
        print("目录不存在")
    
    # 尝试删除文件
    try:
        import os
        os.remove('non_existent.txt')
    except FileNotFoundError:
        print("要删除的文件不存在")
    except PermissionError:
        print("没有权限删除文件")

# 示例
robust_file_operations()
```

### 使用上下文管理器

上下文管理器不仅可以用于文件对象，还可以用于创建自定义的资源管理：

```python
import os
import contextlib
from pathlib import Path

# 使用上下文管理器处理目录切换
@contextlib.contextmanager
def change_directory(path):
    """临时切换工作目录"""
    old_dir = os.getcwd()
    try:
        os.chdir(path)
        yield
    finally:
        os.chdir(old_dir)

# 使用上下文管理器创建临时文件
@contextlib.contextmanager
def temporary_file(content):
    """创建临时文件并在退出时删除"""
    temp_path = Path('temp_file.txt')
    try:
        temp_path.write_text(content)
        yield temp_path
    finally:
        if temp_path.exists():
            temp_path.unlink()

# 使用上下文管理器批量打开文件
@contextlib.contextmanager
def open_files(*filenames):
    """同时打开多个文件"""
    files = []
    try:
        files = [open(filename, 'r') for filename in filenames]
        yield files
    finally:
        for file in files:
            file.close()

# 示例
with change_directory('/tmp'):
    print(f"当前目录: {os.getcwd()}")

with temporary_file("这是临时内容") as temp:
    print(f"临时文件: {temp}")
    print(f"内容: {temp.read_text()}")

with open_files('file1.txt', 'file2.txt') as (f1, f2):
    print(f"文件1内容: {f1.read()[:100]}")
    print(f"文件2内容: {f2.read()[:100]}")
```

### 性能优化

```python
import time
import io
import os
from pathlib import Path

def benchmark(func, *args, **kwargs):
    """简单的函数性能测试"""
    start_time = time.time()
    result = func(*args, **kwargs)
    end_time = time.time()
    print(f"{func.__name__} 耗时: {end_time - start_time:.6f} 秒")
    return result

# 1. 使用缓冲区提高IO效率
def read_file_line_by_line(filename):
    """逐行读取文件（低效）"""
    lines = []
    with open(filename, 'r') as f:
        line = f.readline()
        while line:
            lines.append(line)
            line = f.readline()
    return lines

def read_file_with_readlines(filename):
    """使用readlines读取文件（中等效率）"""
    with open(filename, 'r') as f:
        lines = f.readlines()
    return lines

def read_file_with_iteration(filename):
    """使用迭代器读取文件（高效）"""
    lines = []
    with open(filename, 'r') as f:
        for line in f:
            lines.append(line)
    return lines

# 2. 优化写入操作
def write_inefficient(filename, n_lines):
    """多次小批量写入（低效）"""
    with open(filename, 'w') as f:
        for i in range(n_lines):
            f.write(f"Line {i}\n")

def write_with_buffer(filename, n_lines):
    """使用缓冲区批量写入（高效）"""
    buffer = []
    for i in range(n_lines):
        buffer.append(f"Line {i}\n")
    
    with open(filename, 'w') as f:
        f.writelines(buffer)

def write_with_stringio(filename, n_lines):
    """使用StringIO作为缓冲区（高效）"""
    buffer = io.StringIO()
    for i in range(n_lines):
        buffer.write(f"Line {i}\n")
    
    with open(filename, 'w') as f:
        f.write(buffer.getvalue())

# 3. 使用二进制模式进行大文件处理
def copy_file_text_mode(src, dest, chunk_size=4096):
    """以文本模式复制文件"""
    with open(src, 'r') as sf, open(dest, 'w') as df:
        while True:
            chunk = sf.read(chunk_size)
            if not chunk:
                break
            df.write(chunk)

def copy_file_binary_mode(src, dest, chunk_size=4096):
    """以二进制模式复制文件（更高效）"""
    with open(src, 'rb') as sf, open(dest, 'wb') as df:
        while True:
            chunk = sf.read(chunk_size)
            if not chunk:
                break
            df.write(chunk)

# 4. 使用mmap进行大文件处理
def process_large_file_with_mmap(filename):
    """使用内存映射处理大文件"""
    import mmap
    with open(filename, 'r+b') as f:
        # 创建内存映射
        mm = mmap.mmap(f.fileno(), 0)
        
        # 使用内存映射对象处理文件内容
        line_count = 0
        for line in iter(mm.readline, b''):
            line_count += 1
        
        mm.close()
        return line_count

# 示例：比较不同方法的性能
def compare_performance():
    # 创建测试文件
    test_file = 'test_performance.txt'
    with open(test_file, 'w') as f:
        for i in range(10000):
            f.write(f"This is line {i} of the test file.\n")
    
    # 比较读取性能
    benchmark(read_file_line_by_line, test_file)
    benchmark(read_file_with_readlines, test_file)
    benchmark(read_file_with_iteration, test_file)
    
    # 比较写入性能
    benchmark(write_inefficient, 'write_test1.txt', 10000)
    benchmark(write_with_buffer, 'write_test2.txt', 10000)
    benchmark(write_with_stringio, 'write_test3.txt', 10000)
    
    # 比较复制性能
    benchmark(copy_file_text_mode, test_file, 'copy_text.txt')
    benchmark(copy_file_binary_mode, test_file, 'copy_binary.txt')
    
    # 清理测试文件
    for file in [test_file, 'write_test1.txt', 'write_test2.txt', 
                'write_test3.txt', 'copy_text.txt', 'copy_binary.txt']:
        if os.path.exists(file):
            os.remove(file)

# 运行性能比较
compare_performance()
```

## 总结

Python提供了丰富而强大的文件和IO操作功能，从基本的文件读写到高级的序列化、压缩和目录操作。随着Python的发展，`pathlib`模块提供了更现代、更面向对象的路径处理方式，推荐在新代码中使用。

文件操作是几乎所有程序的基础部分，掌握这些技能对于开发高效、健壮的Python应用至关重要。记住始终使用适当的异常处理、正确处理文件编码，并注意性能优化，特别是在处理大文件时。

此外，使用上下文管理器（with语句）可以确保文件资源被正确关闭，是Python中处理文件的最佳实践。在处理路径时，优先考虑使用`pathlib`而不是`os.path`，因为它提供了更直观、更安全的API。
