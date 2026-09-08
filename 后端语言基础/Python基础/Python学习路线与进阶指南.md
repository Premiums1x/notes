
> 从基础语法到实习项目水平的完整学习路径

---

## 目录

1. [Python基础语法](#1-python基础语法)
2. [Python进阶](#2-python进阶)
3. [面向对象编程](#3-面向对象编程)
4. [常用第三方库](#5-常用第三方库)
5. [Web开发技术栈](#6-web开发技术栈)
6. [数据科学与AI技术栈](#7-数据科学与ai技术栈)
7. [自动化与脚本](#8-自动化与脚本)
8. [项目实战建议](#9-项目实战建议)
9. [实习必备技能](#10-实习必备技能)

---

## 1. Python基础语法

### 1.1 变量与数据类型

```python
# 数字类型
age = 25              # int
price = 19.99         # float
complex_num = 3 + 4j  # complex

# 字符串
name = "Python"
message = f"Hello, {name}"  # f-string (Python 3.6+)

# 布尔值
is_active = True
is_done = False

# 类型转换
str(123)      # '123'
int("456")    # 456
float("3.14") # 3.14
```

### 1.2 数据结构

#### 列表 (List)
```python
# 创建与访问
numbers = [1, 2, 3, 4, 5]
numbers[0]      # 1
numbers[-1]     # 5 (倒数第一个)
numbers[1:4]    # [2, 3, 4] (切片)

# 常用方法
numbers.append(6)      # 末尾添加
numbers.insert(0, 0)   # 指定位置插入
numbers.remove(3)       # 删除指定值
numbers.pop()          # 删除并返回末尾
numbers.extend([7, 8]) # 扩展列表
numbers.count(1)       # 统计出现次数
numbers.index(2)       # 查找索引
numbers.sort()         # 排序
numbers.reverse()      # 反转

# 列表推导式
squares = [x**2 for x in range(10)]
evens = [x for x in range(20) if x % 2 == 0]

# 嵌套列表
matrix = [[1, 2, 3], [4, 5, 6]]
matrix[0][1]  # 2
```

#### 元组 (Tuple) - 不可变序列
```python
coordinates = (10, 20)
coordinates[0]  # 10

# 单元素元组
single = (1,)   # 注意逗号

# 元组解包
x, y = coordinates

# 命名元组
from collections import namedtuple
Point = namedtuple('Point', ['x', 'y'])
p = Point(1, 2)
p.x  # 1
```

#### 字典 (Dictionary)
```python
# 创建与访问
person = {
    "name": "Alice",
    "age": 25,
    "city": "Beijing"
}
person["name"]         # "Alice"
person.get("height", 170)  # 安全获取，默认值170

# 常用方法
person["email"] = "alice@example.com"  # 添加/修改
person.update({"age": 26, "job": "Dev"})  # 批量更新
person.pop("city")     # 删除并返回
person.keys()          # dict_keys(['name', 'age', 'email'])
person.values()        # dict_values(['Alice', 26, 'alice@example.com'])
person.items()         # dict_items([('name', 'Alice'), ...])

# 字典推导式
squares = {x: x**2 for x in range(6)}
# {0: 0, 1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
```

#### 集合 (Set) - 无序不重复
```python
# 创建
fruits = {"apple", "banana", "orange"}
numbers = set([1, 2, 3, 3, 2, 1])  # {1, 2, 3}

# 操作
fruits.add("pear")           # 添加
fruits.remove("apple")       # 删除（不存在会报错）
fruits.discard("apple")      # 删除（不存在不报错）
fruits.clear()               # 清空

# 集合运算
a = {1, 2, 3}
b = {3, 4, 5}
a | b          # 并集 {1, 2, 3, 4, 5}
a & b          # 交集 {3}
a - b          # 差集 {1, 2}
a ^ b          # 对称差集 {1, 2, 4, 5}
```

### 1.3 控制流

#### 条件语句
```python
# if-elif-else
score = 85
if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
else:
    grade = "F"

# 三元表达式
grade = "A" if score >= 90 else "B"

# 多条件
if 18 <= age <= 65:
    print("工作年龄")

# 身份运算符
if x is None:
    print("x是None")

if x is not None:
    print("x不是None")
```

#### 循环
```python
# for循环
for i in range(5):        # 0, 1, 2, 3, 4
    print(i)

for i in range(1, 6):     # 1, 2, 3, 4, 5
    print(i)

for i in range(0, 10, 2): # 0, 2, 4, 6, 8
    print(i)

# 遍历列表
for fruit in fruits:
    print(fruit)

# 遍历字典
for key, value in person.items():
    print(f"{key}: {value}")

# 同时获取索引和值
for index, value in enumerate(fruits):
    print(f"{index}: {value}")

# while循环
count = 0
while count < 5:
    print(count)
    count += 1

# 循环控制
for i in range(10):
    if i == 3:
        continue    # 跳过本次
    if i == 7:
        break       # 跳出循环
    print(i)

# else块（循环正常结束时执行）
for i in range(5):
    print(i)
else:
    print("循环正常结束")
```

### 1.4 函数

```python
# 基本函数
def greet(name):
    return f"Hello, {name}!"

# 默认参数
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

# 可变参数
def sum_all(*args):
    return sum(args)

sum_all(1, 2, 3, 4)  # 10

# 关键字参数
def create_profile(**kwargs):
    return kwargs

create_profile(name="Alice", age=25, city="Beijing")

# 混合使用
def func(a, b=2, *args, **kwargs):
    pass

# 类型注解 (Python 3.5+)
def add(x: int, y: int) -> int:
    return x + y

# Lambda函数
square = lambda x: x**2
square(5)  # 25

# 高阶函数
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, numbers))     # [1, 4, 9, 16, 25]
evens = list(filter(lambda x: x % 2 == 0, numbers))  # [2, 4]
```

### 1.5 异常处理

```python
# 基本异常处理
try:
    result = 10 / 0
except ZeroDivisionError:
    print("不能除以零")

# 多个异常
try:
    file = open("notfound.txt", "r")
except FileNotFoundError:
    print("文件不存在")
except PermissionError:
    print("没有权限")
except Exception as e:
    print(f"未知错误: {e}")

# else和finally
try:
    result = risky_operation()
except ValueError as e:
    print(f"错误: {e}")
else:
    print("操作成功")
finally:
    print("无论如何都会执行")

# 抛出异常
def validate_age(age):
    if age < 0:
        raise ValueError("年龄不能为负数")

# 自定义异常
class InvalidAgeError(Exception):
    pass
```

### 1.6 文件操作

```python
# 读取文件
with open("file.txt", "r", encoding="utf-8") as f:
    content = f.read()        # 全部内容
    lines = f.readlines()     # 按行读取列表

# 逐行读取（大文件）
with open("large_file.txt", "r") as f:
    for line in f:
        print(line.strip())

# 写入文件
with open("output.txt", "w", encoding="utf-8") as f:
    f.write("Hello, World!\n")

# 追加写入
with open("log.txt", "a") as f:
    f.write("New log entry\n")

# JSON操作
import json

# 写入JSON
data = {"name": "Alice", "age": 25}
with open("data.json", "w") as f:
    json.dump(data, f, indent=2)

# 读取JSON
with open("data.json", "r") as f:
    data = json.load(f)
```

### 1.7 模块与包

```python
# 导入模块
import math
print(math.sqrt(16))  # 4.0

# 导入特定函数
from math import sqrt, pi
sqrt(16)  # 4.0

# 别名
import numpy as np
import pandas as pd

# 导入包内模块
from package import module

# 相对导入（在包内）
from .sibling_module import function
from ..parent_module import function

# __main__ 用法
if __name__ == "__main__":
    # 只有直接运行此文件时执行
    main()
```

---

## 2. Python进阶

### 2.1 装饰器 (Decorators)

装饰器是Python中非常强大的特性，用于修改或增强函数行为。

```python
# 基本装饰器
def timer(func):
    import time
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} 耗时: {end - start:.2f}秒")
        return result
    return wrapper

@timer
def slow_function():
    import time
    time.sleep(1)

# 带参数的装饰器
def repeat(times):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def say_hello():
    print("Hello!")

# 类装饰器
class CountCalls:
    def __init__(self, func):
        self.func = func
        self.count = 0

    def __call__(self, *args, **kwargs):
        self.count += 1
        print(f"调用次数: {self.count}")
        return self.func(*args, **kwargs)

@CountCalls
def greet():
    print("Hello!")

# functools.wraps 保留原函数信息
from functools import wraps

def logger(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"调用 {func.__name__}")
        return func(*args, **kwargs)
    return wrapper
```

### 2.2 生成器 (Generators)

生成器是一种特殊的迭代器，可以节省内存。

```python
# 生成器函数
def count_up_to(n):
    count = 1
    while count <= n:
        yield count
        count += 1

for num in count_up_to(5):
    print(num)

# 生成器表达式
squares = (x**2 for x in range(10))

# 无限生成器
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

# send() 方法
def accumulator():
    total = 0
    while True:
        value = yield total
        if value is not None:
            total += value

acc = accumulator()
next(acc)        # 启动生成器
acc.send(10)     # 发送值
acc.send(5)
```

### 2.3 上下文管理器

```python
# 自定义上下文管理器
class Timer:
    def __enter__(self):
        import time
        self.start = time.time()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        import time
        self.end = time.time()
        print(f"执行时间: {self.end - self.start:.2f}秒")

with Timer():
    # 一些操作
    pass

# contextlib 简化版
from contextlib import contextmanager

@contextmanager
def timer():
    import time
    start = time.time()
    yield
    print(f"耗时: {time.time() - start:.2f}秒")

with timer():
    # 代码
    pass
```

### 2.4 迭代器协议

```python
class CountIterator:
    def __init__(self, n):
        self.n = n
        self.current = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.current >= self.n:
            raise StopIteration
        self.current += 1
        return self.current

for i in CountIterator(5):
    print(i)  # 1, 2, 3, 4, 5
```

### 2.5 描述器 (Descriptors)

```python
class PositiveNumber:
    def __get__(self, instance, owner):
        return instance.__dict__.get(self.name)

    def __set_name__(self, owner, name):
        self.name = f"_{name}"

    def __set__(self, instance, value):
        if value < 0:
            raise ValueError("值必须为正数")
        instance.__dict__[self.name] = value

class Product:
    price = PositiveNumber()
    quantity = PositiveNumber()
```

### 2.6 元类 (Metaclasses)

```python
# 基础元类
class Singleton(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=Singleton):
    pass

db1 = Database()
db2 = Database()
db1 is db2  # True

# 使用__init_subclass__
class Base:
    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        print(f"创建子类: {cls}")
```

### 2.7 多线程与多进程

```python
# 多线程 (适合IO密集型)
import threading
import time

def worker(name):
    print(f"{name} 开始工作")
    time.sleep(1)
    print(f"{name} 完成工作")

# 创建线程
t1 = threading.Thread(target=worker, args=("线程1",))
t2 = threading.Thread(target=worker, args=("线程2",))

t1.start()
t2.start()

t1.join()  # 等待完成
t2.join()

# 线程池
from concurrent.futures import ThreadPoolExecutor

def process_url(url):
    # 处理URL
    return f"Processed {url}"

urls = ["url1", "url2", "url3"]

with ThreadPoolExecutor(max_workers=3) as executor:
    results = list(executor.map(process_url, urls))

# 多进程 (适合CPU密集型)
from multiprocessing import Process, Pool

def square(x):
    return x * x

if __name__ == "__main__":
    with Pool(4) as p:
        results = p.map(square, range(10))
```

### 2.8 异步编程 (async/await)

```python
import asyncio
import aiohttp

async def fetch_data(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()

async def main():
    urls = ["url1", "url2", "url3"]
    tasks = [fetch_data(url) for url in urls]
    results = await asyncio.gather(*tasks)
    return results

# 运行异步代码
asyncio.run(main())

# 异步上下文管理器
class AsyncTimer:
    async def __aenter__(self):
        self.start = asyncio.get_event_loop().time()
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print(f"耗时: {asyncio.get_event_loop().time() - self.start:.2f}秒")
```

### 2.9 高级函数式编程

```python
from functools import partial, reduce
import operator

# partial - 偏函数
multiply_by_2 = partial(operator.mul, 2)
multiply_by_2(5)  # 10

# reduce
numbers = [1, 2, 3, 4, 5]
product = reduce(operator.mul, numbers)  # 120

# 柯里化
def add(x):
    def inner(y):
        return x + y
    return inner

add_5 = add(5)
add_5(3)  # 8
```

---

## 3. 面向对象编程

### 3.1 类与对象基础

```python
class Person:
    # 类属性
    species = "Homo sapiens"

    def __init__(self, name, age):
        # 实例属性
        self.name = name
        self.age = age

    # 实例方法
    def greet(self):
        return f"Hello, I'm {self.name}"

    # 类方法
    @classmethod
    def from_birth_year(cls, name, birth_year):
        current_year = 2024
        age = current_year - birth_year
        return cls(name, age)

    # 静态方法
    @staticmethod
    def is_adult(age):
        return age >= 18

# 使用
person = Person("Alice", 25)
print(person.greet())
```

### 3.2 继承与多态

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        pass

class Dog(Animal):
    def speak(self):
        return f"{self.name} says: Woof!"

class Cat(Animal):
    def speak(self):
        return f"{self.name} says: Meow!"

# 多态
animals = [Dog("Buddy"), Cat("Whiskers")]
for animal in animals:
    print(animal.speak())

# super() 调用父类方法
class Child(Parent):
    def __init__(self, name, age):
        super().__init__(name)
        self.age = age
```

### 3.3 特殊方法 (魔法方法)

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    # 字符串表示
    def __str__(self):
        return f"Vector({self.x}, {self.y})"

    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

    # 相等比较
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

    # 加法运算
    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    # 长度
    def __len__(self):
        return int((self.x**2 + self.y**2) ** 0.5)

    # 索引
    def __getitem__(self, index):
        return [self.x, self.y][index]

# 使用
v1 = Vector(3, 4)
v2 = Vector(1, 2)
v3 = v1 + v2
print(len(v3))  # 5
```

### 3.4 抽象基类

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

    @abstractmethod
    def perimeter(self):
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

    def perimeter(self):
        return 2 * (self.width + self.height)

# 不能直接实例化抽象类
# shape = Shape()  # TypeError
```

### 3.5 数据类 (Dataclasses)

```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class Person:
    name: str
    age: int
    email: str = "unknown@example.com"
    friends: List[str] = field(default_factory=list)

    def __post_init__(self):
        if self.age < 0:
            raise ValueError("年龄不能为负数")

# 自动生成__init__, __repr__, __eq__等方法
person = Person("Alice", 25, "alice@example.com")
person2 = Person("Alice", 25, "alice@example.com")
person == person2  # True
```

### 3.6 设计模式

#### 单例模式
```python
class Singleton:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
```

#### 工厂模式
```python
class AnimalFactory:
    @staticmethod
    def create_animal(animal_type):
        if animal_type == "dog":
            return Dog()
        elif animal_type == "cat":
            return Cat()
        else:
            raise ValueError("Unknown animal type")
```

#### 观察者模式
```python
class Subject:
    def __init__(self):
        self._observers = []

    def attach(self, observer):
        self._observers.append(observer)

    def notify(self):
        for observer in self._observers:
            observer.update(self)
```

---

## 4. 标准库核心模块

### 4.1 os - 操作系统接口

```python
import os
import pathlib

# 路径操作
path = os.path.join("folder", "file.txt")  # 跨平台路径拼接
os.path.exists(path)                       # 检查路径是否存在
os.path.isfile(path)                       # 是否是文件
os.path.isdir(path)                        # 是否是目录
os.path.abspath(".")                       # 获取绝对路径

# 文件夹操作
os.makedirs("new/folder", exist_ok=True)  # 创建文件夹
os.listdir(".")                            # 列出目录内容
os.remove("file.txt")                      # 删除文件
os.rmdir("folder")                         # 删除空目录
import shutil
shutil.rmtree("folder")                     # 删除目录及内容

# 环境变量
os.environ.get("PATH")                     # 获取环境变量
os.environ["MY_VAR"] = "value"             # 设置环境变量

# pathlib（现代路径操作）
from pathlib import Path

path = Path("folder") / "file.txt"
path.exists()                              # 检查存在
path.is_file()                             # 是否文件
path.read_text()                           # 读取文本
path.write_text("Hello")                   # 写入文本
path.mkdir(parents=True, exist_ok=True)    # 创建目录
```

### 4.2 sys - 系统特定参数

```python
import sys

# 命令行参数
sys.argv[0]    # 脚本名
sys.argv[1:]   # 其他参数

# 退出程序
sys.exit(0)    # 正常退出
sys.exit(1)    # 异常退出

# 路径
sys.path       # Python模块搜索路径

# 平台信息
sys.platform   # 操作系统
sys.version    # Python版本
```

### 4.3 datetime - 日期时间处理

```python
from datetime import datetime, date, time, timedelta

# 当前时间
now = datetime.now()
print(now.strftime("%Y-%m-%d %H:%M:%S"))

# 创建日期
d = date(2024, 3, 15)
t = time(14, 30, 0)

# 字符串解析
dt = datetime.strptime("2024-03-15 14:30", "%Y-%m-%d %H:%M")

# 时间差
delta = timedelta(days=7, hours=2)
next_week = now + delta

# dateutil（需要安装）
from dateutil.relativedelta import relativedelta
next_month = now + relativedelta(months=1)
```

### 4.4 re - 正则表达式

```python
import re

# 匹配
pattern = r"\d+"
text = "有123个苹果"
re.search(pattern, text)       # 搜索
re.findall(pattern, text)       # 查找所有
re.sub(pattern, "X", text)     # 替换

# 编译正则
phone_pattern = re.compile(r"1[3-9]\d{9}")
phone_pattern.match("13812345678")

# 分组
email_pattern = r"(\w+)@(\w+\.\w+)"
match = re.search(email_pattern, "test@example.com")
match.group(1)    # test
match.group(2)    # example.com
```

### 4.5 json - JSON数据处理

```python
import json

# Python转JSON
data = {
    "name": "Alice",
    "age": 25,
    "scores": [90, 85, 95]
}
json_str = json.dumps(data, indent=2, ensure_ascii=False)

# JSON转Python
parsed = json.loads(json_str)

# 文件操作
with open("data.json", "w") as f:
    json.dump(data, f, indent=2)

with open("data.json", "r") as f:
    data = json.load(f)
```

### 4.6 collections - 高级数据结构

```python
from collections import Counter, defaultdict, deque, OrderedDict

# Counter - 计数
counts = Counter(["a", "b", "a", "c", "b", "a"])
counts.most_common(2)  # [('a', 3), ('b', 2)]

# defaultdict - 默认值字典
d = defaultdict(int)
d["key"] += 1  # 不会报错

# deque - 双端队列
dq = deque([1, 2, 3])
dq.append(4)       # 右端添加
dq.appendleft(0)   # 左端添加
dq.pop()           # 右端弹出
dq.popleft()       # 左端弹出

# OrderedDict - 有序字典（Python 3.7+普通dict已有序）
od = OrderedDict([("a", 1), ("b", 2)])
```

### 4.7 itertools - 迭代器工具

```python
from itertools import count, cycle, repeat, chain, combinations, permutations

# 无限计数
for i in count(10, 2):  # 从10开始，每次+2
    if i > 20:
        break

# 循环
for item in cycle(["a", "b", "c"]):
    # 循环访问
    pass

# 重复
for i in repeat(10, 5):  # 重复10次10
    print(i)

# 链接
list(chain([1, 2], [3, 4], [5, 6]))  # [1, 2, 3, 4, 5, 6]

# 组合
list(combinations([1, 2, 3], 2))     # [(1, 2), (1, 3), (2, 3)]
list(permutations([1, 2, 3], 2))    # [(1, 2), (1, 3), (2, 1), (2, 3), (3, 1), (3, 2)]
```

### 4.8 logging - 日志记录

```python
import logging

# 基础配置
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    filename='app.log'
)

logger = logging.getLogger(__name__)

logger.debug("调试信息")
logger.info("普通信息")
logger.warning("警告")
logger.error("错误")
logger.critical("严重错误")

# 同时输出到文件和终端
handler1 = logging.FileHandler('app.log')
handler2 = logging.StreamHandler()
logger.addHandler(handler1)
logger.addHandler(handler2)
```

### 4.9 unittest - 单元测试

```python
import unittest

class TestMath(unittest.TestCase):
    def test_add(self):
        self.assertEqual(2 + 2, 4)

    def test_divide(self):
        with self.assertRaises(ZeroDivisionError):
            1 / 0

    def setUp(self):
        # 每个测试前执行
        pass

    def tearDown(self):
        # 每个测试后执行
        pass

if __name__ == "__main__":
    unittest.main()
```

---

## 5. 常用第三方库

### 5.1 requests - HTTP请求

```python
import requests

# GET请求
response = requests.get("https://api.example.com/data")
data = response.json()
status = response.status_code

# 带参数
params = {"page": 1, "limit": 10}
response = requests.get("https://api.example.com", params=params)

# POST请求
payload = {"name": "Alice", "age": 25}
response = requests.post("https://api.example.com/users", json=payload)

# 带Headers
headers = {"Authorization": "Bearer token123"}
response = requests.get("https://api.example.com/protected", headers=headers)

# Session（保持cookie）
session = requests.Session()
session.get("https://example.com/login")
session.get("https://example.com/dashboard")

# 上传文件
files = {"file": open("report.pdf", "rb")}
response = requests.post("https://example.com/upload", files=files)
```

### 5.2 Pillow - 图像处理

```python
from PIL import Image, ImageFilter, ImageDraw, ImageFont

# 打开图像
img = Image.open("photo.jpg")

# 调整大小
img_resized = img.resize((800, 600))

# 裁剪
img_cropped = img.crop((100, 100, 400, 400))

# 旋转
img_rotated = img.rotate(45)

# 滤镜
img_blurred = img.filter(ImageFilter.BLUR)

# 绘图
draw = ImageDraw.Draw(img)
draw.rectangle([(10, 10), (100, 100)], fill="blue")
draw.text((50, 50), "Hello", fill="white")

# 保存
img.save("output.jpg", quality=95)
```

### 5.3 SQLAlchemy - ORM

```python
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.orm import sessionmaker, declarative_base

Base = declarative_base()

class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    email = Column(String(100))

# 创建连接
engine = create_engine("sqlite:///database.db")
Base.metadata.create_all(engine)

Session = sessionmaker(bind=engine)
session = Session()

# 增
user = User(name="Alice", email="alice@example.com")
session.add(user)
session.commit()

# 查
users = session.query(User).all()
alice = session.query(User).filter_by(name="Alice").first()

# 改
alice.email = "newemail@example.com"
session.commit()

# 删
session.delete(alice)
session.commit()
```

### 5.4 pydantic - 数据验证

```python
from pydantic import BaseModel, EmailStr, field_validator
from typing import Optional

class User(BaseModel):
    name: str
    age: int
    email: EmailStr
    is_active: bool = True

    @field_validator('age')
    @classmethod
    def check_age(cls, v):
        if v < 0 or v > 150:
            raise ValueError('年龄必须在0-150之间')
        return v

# 验证
user_data = {"name": "Alice", "age": 25, "email": "alice@example.com"}
user = User(**user_data)  # 自动验证
```

### 5.5 pytest - 测试框架

```python
import pytest

def test_addition():
    assert 2 + 2 == 4

@pytest.fixture
def sample_data():
    return {"name": "Alice", "age": 25}

def test_with_fixture(sample_data):
    assert sample_data["name"] == "Alice"

@pytest.mark.parametrize("a,b,expected", [
    (1, 2, 3),
    (2, 3, 5),
    (3, 4, 7),
])
def test_add(a, b, expected):
    assert a + b == expected
```

### 5.6 click - 命令行工具

```python
import click

@click.command()
@click.option('--count', default=1, help='Number of greetings.')
@click.option('--name', prompt='Your name', help='The person to greet.')
def hello(count, name):
    """Simple program that greets NAME COUNT times."""
    for _ in range(count):
        click.echo(f"Hello, {name}!")

if __name__ == '__main__':
    hello()
```

---

## 6. Web开发技术栈

### 6.1 FastAPI - 现代Web框架

FastAPI是当前最流行的Python Web框架之一，支持异步、自动文档生成。

```python
from fastapi import FastAPI, HTTPException, Depends
from pydantic import BaseModel
from typing import List, Optional

app = FastAPI()

# 数据模型
class User(BaseModel):
    id: Optional[int] = None
    name: str
    email: str
    age: int

# 内存数据库
fake_db: List[User] = []
next_id = 1

# CRUD 操作
@app.post("/users/", response_model=User)
def create_user(user: User):
    global next_id
    user.id = next_id
    next_id += 1
    fake_db.append(user)
    return user

@app.get("/users/", response_model=List[User])
def get_users(skip: int = 0, limit: int = 10):
    return fake_db[skip:skip+limit]

@app.get("/users/{user_id}", response_model=User)
def get_user(user_id: int):
    for user in fake_db:
        if user.id == user_id:
            return user
    raise HTTPException(status_code=404, detail="User not found")

@app.put("/users/{user_id}", response_model=User)
def update_user(user_id: int, user: User):
    for i, u in enumerate(fake_db):
        if u.id == user_id:
            user.id = user_id
            fake_db[i] = user
            return user
    raise HTTPException(status_code=404, detail="User not found")

@app.delete("/users/{user_id}")
def delete_user(user_id: int):
    for i, user in enumerate(fake_db):
        if user.id == user_id:
            del fake_db[i]
            return {"message": "Deleted"}
    raise HTTPException(status_code=404, detail="User not found")

# 依赖注入
async def get_current_user(token: str):
    # 验证token
    return User(name="test", email="test@test.com", age=25)

@app.get("/protected")
def protected_route(current_user: User = Depends(get_current_user)):
    return {"message": f"Hello {current_user.name}"}

# 运行: uvicorn main:app --reload
```

### 6.2 Django - 全功能Web框架

```python
# models.py
from django.db import models

class User(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField(unique=True)
    age = models.IntegerField()
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.name

# views.py
from django.shortcuts import render, get_object_or_404
from django.http import JsonResponse
from .models import User
from .serializers import UserSerializer
from rest_framework.decorators import api_view

def user_list(request):
    users = User.objects.all()
    return render(request, 'users.html', {'users': users})

@api_view(['GET', 'POST'])
def user_api(request):
    if request.method == 'GET':
        users = User.objects.all()
        serializer = UserSerializer(users, many=True)
        return JsonResponse(serializer.data, safe=False)
    elif request.method == 'POST':
        serializer = UserSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data, status=201)
        return JsonResponse(serializer.errors, status=400)

# urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('users/', views.user_list, name='user_list'),
    path('api/users/', views.user_api, name='user_api'),
]
```

### 6.3 Flask - 轻量级Web框架

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

users = []

@app.route('/')
def home():
    return jsonify({"message": "Hello, Flask!"})

@app.route('/users', methods=['GET'])
def get_users():
    return jsonify(users)

@app.route('/users', methods=['POST'])
def create_user():
    data = request.get_json()
    users.append(data)
    return jsonify(data), 201

if __name__ == '__main__':
    app.run(debug=True)
```

### 6.4 Web相关技术

```python
# aiohttp - 异步HTTP客户端/服务器
import aiohttp
import asyncio

async def fetch(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()

# websockets
from fastapi import WebSocket

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Echo: {data}")

# Jinja2 模板
from jinja2 import Template

template = Template("Hello {{ name }}!")
print(template.render(name="Alice"))

# HTML处理
from bs4 import BeautifulSoup

html = "<html><body><h1>Hello</h1></body></html>"
soup = BeautifulSoup(html, 'html.parser')
print(soup.h1.text)  # Hello
```

### 6.5 认证与安全

```python
# JWT认证
from jose import jwt
from datetime import datetime, timedelta

SECRET_KEY = "your-secret-key"
ALGORITHM = "HS256"

def create_access_token(data: dict):
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=30)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

def verify_token(token: str):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload
    except jwt.JWTError:
        return None

# 密码哈希
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str):
    return pwd_context.hash(password)

def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)
```

---

## 7. 数据科学与AI技术栈

### 7.1 NumPy - 数值计算

```python
import numpy as np

# 创建数组
arr = np.array([1, 2, 3, 4, 5])
matrix = np.array([[1, 2], [3, 4]])

# 特殊数组
zeros = np.zeros((3, 3))
ones = np.ones((2, 4))
random = np.random.rand(3, 3)
range_arr = np.arange(0, 10, 2)  # [0, 2, 4, 6, 8]
linspace = np.linspace(0, 10, 5)  # [0, 2.5, 5, 7.5, 10]

# 数组操作
arr.shape      # 形状
arr.dtype      # 数据类型
arr.reshape(5, 1)  # 重塑
arr.T          # 转置

# 索引与切片
arr[0:3]       # 前3个元素
matrix[:, 0]   # 第一列
matrix[matrix > 2]  # 条件筛选

# 数学运算
np.sum(arr)        # 求和
np.mean(arr)       # 平均值
np.std(arr)        # 标准差
np.max(arr)        # 最大值
np.min(arr)        # 最小值
np.dot(a, b)       # 点积
np.linalg.inv(matrix)  # 矩阵逆
```

### 7.2 Pandas - 数据分析

```python
import pandas as pd

# 创建DataFrame
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 35],
    'city': ['Beijing', 'Shanghai', 'Guangzhou']
})

# 读取数据
df = pd.read_csv('data.csv')
df = pd.read_excel('data.xlsx')
df = pd.read_json('data.json')

# 基本操作
df.head()           # 前5行
df.tail()           # 后5行
df.info()           # 数据信息
df.describe()       # 统计描述
df.shape            # 形状
df.columns          # 列名

# 选择数据
df['name']          # 单列
df[['name', 'age']] # 多列
df.loc[0]           # 按标签
df.iloc[0]          # 按位置
df[df['age'] > 30]  # 条件筛选

# 数据处理
df['age'] = df['age'].fillna(0)  # 填充空值
df.dropna()                       # 删除空值
df.drop_duplicates()             # 删除重复
df.groupby('city')['age'].mean()  # 分组聚合

# 数据转换
df['age_doubled'] = df['age'] * 2
df['age_category'] = pd.cut(df['age'], bins=[0, 25, 35, 100], labels=['young', 'middle', 'old'])

# 写入数据
df.to_csv('output.csv', index=False)
df.to_excel('output.xlsx', index=False)
```

### 7.3 Matplotlib - 数据可视化

```python
import matplotlib.pyplot as plt
import numpy as np

# 线图
x = np.linspace(0, 10, 100)
y = np.sin(x)
plt.plot(x, y, label='sin(x)')
plt.xlabel('x')
plt.ylabel('y')
plt.legend()
plt.show()

# 散点图
plt.scatter(df['age'], df['score'])

# 柱状图
plt.bar(df['name'], df['age'])

# 子图
fig, axes = plt.subplots(2, 2)
axes[0, 0].plot(x, np.sin(x))
axes[0, 1].plot(x, np.cos(x))
axes[1, 0].scatter([1,2,3], [4,5,6])
axes[1, 1].bar(['A','B','C'], [10,20,15])
plt.tight_layout()
```

### 7.4 PyTorch - 深度学习

```python
import torch
import torch.nn as nn
import torch.optim as optim

# 张量操作
x = torch.tensor([1, 2, 3, 4])
y = torch.zeros(2, 3)
z = torch.randn(2, 3)

# GPU支持
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
x = x.to(device)

# 神经网络
class SimpleNet(nn.Module):
    def __init__(self):
        super().__init__()
        self.fc1 = nn.Linear(10, 50)
        self.fc2 = nn.Linear(50, 2)
        self.relu = nn.ReLU()

    def forward(self, x):
        x = self.relu(self.fc1(x))
        x = self.fc2(x)
        return x

model = SimpleNet().to(device)
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=0.001)

# 训练循环
for epoch in range(10):
    outputs = model(inputs)
    loss = criterion(outputs, labels)
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
```

### 7.5 Scikit-learn - 机器学习

```python
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline

# 数据准备
X = [[1, 2], [3, 4], [5, 6]]
y = [0, 1, 0]

# 划分数据集
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# 创建模型
model = Pipeline([
    ('scaler', StandardScaler()),
    ('classifier', RandomForestClassifier())
])

# 训练
model.fit(X_train, y_train)

# 预测
predictions = model.predict(X_test)
accuracy = accuracy_score(y_test, predictions)
```

---

## 8. 自动化与脚本

### 8.1 文件处理自动化

```python
import os
import shutil
from pathlib import Path

# 批量重命名
def batch_rename(folder, old_name, new_name):
    for filename in os.listdir(folder):
        if old_name in filename:
            new_filename = filename.replace(old_name, new_name)
            os.rename(
                os.path.join(folder, filename),
                os.path.join(folder, new_filename)
            )

# 按类型整理文件
def organize_files(source):
    extensions = {
        '.jpg': 'Images',
        '.png': 'Images',
        '.pdf': 'Documents',
        '.docx': 'Documents',
        '.mp3': 'Music',
        '.mp4': 'Videos'
    }

    for file in Path(source).glob('*'):
        if file.is_file():
            ext = file.suffix.lower()
            if ext in extensions:
                dest = Path(source) / extensions[ext] / file.name
                dest.parent.mkdir(exist_ok=True)
                shutil.move(str(file), str(dest))
```

### 8.2 网络爬虫

```python
import requests
from bs4 import BeautifulSoup
from concurrent.futures import ThreadPoolExecutor

def scrape_page(url):
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')
    return soup.title.string

# 多线程爬取
urls = ["url1", "url2", "url3"]
with ThreadPoolExecutor(max_workers=5) as executor:
    results = list(executor.map(scrape_page, urls))

# Scrapy框架
import scrapy

class MySpider(scrapy.Spider):
    name = 'myspider'
    start_urls = ['https://example.com']

    def parse(self, response):
        title = response.css('h1::text').get()
        yield {'title': title}
```

### 8.3 定时任务

```python
import schedule
import time

def job():
    print("执行任务")

schedule.every(10).seconds.do(job)
schedule.every().day.at("09:30").do(job)
schedule.every().monday.do(job)

while True:
    schedule.run_pending()
    time.sleep(1)
```

---

## 9. 项目实战建议

### 9.1 入门级项目

1. **待办事项应用**
   - FastAPI + SQLite
   - CRUD操作
   - 用户认证

2. **天气查询工具**
   - requests调用天气API
   - 命令行界面
   - 数据持久化

3. **文件管理器**
   - GUI (Tkinter/PyQt)
   - 文件操作自动化

### 9.2 中级项目

1. **博客系统**
   - Django/Django REST Framework
   - Markdown支持
   - 用户系统

2. **电商网站**
   - FastAPI + PostgreSQL
   - 购物车功能
   - 支付集成

3. **数据分析平台**
   - Pandas + Matplotlib
   - 数据可视化
   - 报告生成

### 9.3 高级项目

1. **实时聊天应用**
   - FastAPI + WebSocket
   - Redis缓存
   - 消息持久化

2. **机器学习API服务**
   - PyTorch/Scikit-learn
   - 模型训练与部署
   - Docker容器化

3. **微服务架构**
   - 多个FastAPI服务
   - 服务间通信
   - 服务发现

---

## 10. 实习必备技能

### 10.1 技术技能

- **Web框架**: FastAPI/Django/Flask
- **数据库**: PostgreSQL/MySQL + SQLAlchemy
- **异步编程**: asyncio, aiohttp
- **测试**: pytest, unittest
- **API文档**: OpenAPI/Swagger
- **版本控制**: Git
- **容器化**: Docker
- **CI/CD**: GitHub Actions

### 10.2 软技能

- 代码可读性和文档编写
- 代码审查
- 问题调试能力
- 团队协作
- 自主学习能力

### 10.3 项目准备

- 在GitHub上维护活跃项目
- 编写完整的README文档
- 添加测试覆盖
- 遵循代码规范（PEP 8）
- 学习设计模式

### 10.4 学习资源

#### 官方文档
- [Python官方文档](https://docs.python.org/zh_CN/3/)
- [FastAPI文档](https://fastapi.tiangolo.com/zh/)
- [Django文档](https://docs.djangoproject.com/zh-hans/)

#### 在线课程
- Real Python
- Coursera Python专项课程
- edX Python课程

#### 实践平台
- LeetCode (算法)
- HackerRank
- Codewars

#### 社区
- Python中文社区
- Stack Overflow
- GitHub上的Python项目

---

## 学习建议

1. **边学边练**: 每学一个概念都要写代码练习
2. **项目驱动**: 通过完成项目来整合所学知识
3. **阅读源码**: 学习优秀开源项目的代码风格
4. **持续练习**: 保持每天编码的习惯
5. **参与社区**: 提问、回答、贡献代码

**祝学习顺利，早日达到实习水平！**
