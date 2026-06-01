---
title: Python 学习笔记（一）：基础数据类型与流程控制
date: 2026-06-01 20:00:00
cover: https://haowallpaper.com/link/common/file/previewFileImg/18698227609095552
tags:
  - Python
  - 编程入门
  - 学习笔记
categories:
  - 编程学习
---

# 前言

作为一个主要写前端（JavaScript/TypeScript/Vue）的开发者，我一直觉得后端语言是技能树上缺失的一块。最近终于下定决心系统学习 Python，原因很简单：

1. **语法简洁**，上手快，能快速把想法变成代码
2. **生态丰富**，从 Web 开发到数据分析到 AI，覆盖面广
3. **与前端互补**，以后能独立搭建全栈项目

这篇文章记录第一天的学习成果——Python 四大核心数据类型、循环控制流、字符串操作和文件读写。全部内容都配有可运行的代码示例，跟着敲一遍基本就能掌握。

# 环境与工具

```bash
# 安装 Python 3.14（当前最新稳定版）
# Windows 用户直接从官网下载安装包，勾选"Add Python to PATH"

# 验证安装
python --version
# Python 3.14.x

# 编辑器：VSCode + Python 插件
```

项目目录结构：

```
pythonLearn/
├── 01_basics/
│   ├── 01_data_types.py    # 数据类型
│   ├── 02_loops.py         # 循环
│   ├── 03_strings.py       # 字符串
│   ├── 04_file_io.py       # 文件读写
│   └── exercise_*.py       # 各课练习
└── demo/
    └── BMI.py              # 入门作：BMI 计算器
```

# 一、四大核心数据类型

每种语言都有自己的核心数据结构，Python 的 `list` / `tuple` / `dict` / `set` 基本覆盖了 90% 的日常场景。

## 1.1 list — 列表

列表是 **有序、可变** 的序列，用 `[]` 表示。类比 JS 的数组，但 Python 的切片语法更灵活：

```python
scores: list[int] = [85, 92, 78, 90, 88]

# 索引和切片（不包含结束位置）
scores[0]       # 85          — 第一个
scores[-1]      # 88          — 倒数第一个
scores[:3]      # [85, 92, 78] — 前三个
scores[1:4]     # [92, 78, 90] — 索引 1~3

# 增删改查
scores.append(95)            # 追加到末尾
scores.insert(1, 100)        # 在索引 1 处插入
scores.remove(78)            # 按值删除
scores.pop()                 # 弹出最后一个

# 列表推导式 — Python 的标志性语法
doubled = [s * 2 for s in scores]         # 每项 ×2
passed = [s for s in scores if s >= 90]   # 只保留 ≥90 的
```

> **感悟**：列表推导式是第一个让我觉得 Python 确实简洁的地方。同样的逻辑在 JS 里需要 `map` + `filter` 链式调用，Python 一个中括号搞定且语义清晰。

## 1.2 tuple — 元组

元组是 **有序、不可变** 的序列，用 `()` 表示。创建后不能修改，适合存常量、配置、坐标等不会变的数据：

```python
point: tuple[int, int] = (3, 4)
rgb: tuple[int, int, int] = (255, 128, 0)

# 解包 — 非常实用
x, y = point        # x=3, y=4
r, g, b = rgb       # R=255, G=128, B=0

# 一行交换两个变量（JS 需要解构 + 临时变量）
a, b = b, a

# NamedTuple — 给字段命名，可读性更好
from typing import NamedTuple

class Student(NamedTuple):
    name: str
    age: int
    grade: float

s = Student(name="小明", age=15, grade=92.5)
print(s.name, s.age)  # 像对象一样用属性访问
```

> **对比 JS**：Python 元组的不可变性类似 `Object.freeze()`，但语法层面就保证了，不用额外处理。

## 1.3 dict — 字典

字典是 **键值对映射**，用 `{}` 表示。相当于 JS 的 `Object` / `Map`，但访问方式更安全：

```python
student: dict[str, object] = {
    "name": "小明",
    "age": 15,
    "scores": [85, 92, 78],
}

# get() 比 [] 更安全 — 不存在时返回 None 而不是报错
student.get("phone")            # None
student.get("phone", "未填写")  # "未填写" — 设默认值

# 遍历
for key, value in student.items():
    print(f"{key} -> {value}")

# 字典推导式
squares = {x: x ** 2 for x in range(1, 6)}
# {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
```

> **特别提醒**：`student["phone"]` 如果 key 不存在会直接抛 `KeyError`，养成用 `get()` 的习惯能少很多 bug。

## 1.4 set — 集合

集合是 **无序、不重复** 元素的集合。最适合做去重和集合运算：

```python
numbers = {1, 2, 3, 2, 1, 4, 5}
print(numbers)  # {1, 2, 3, 4, 5} — 自动去重

math_club = {"小明", "小红", "小刚", "小丽"}
music_club = {"小红", "小丽", "小华", "小雪"}

math_club & music_club   # 交集：两个社团都参加 {'小红', '小丽'}
math_club | music_club   # 并集：所有成员
math_club - music_club   # 差集：只在数学社 {'小明', '小刚'}
math_club ^ music_club   # 对称差：只在其中一个社

# 成员判断 — 比列表快得多（O(1) vs O(n)）
"小明" in math_club  # True
```

## 1.5 综合示例：班级成绩分析

四种类型一起用才真正感受到组合的力量：

```python
subjects: tuple[str, ...] = ("语文", "数学", "英语")  # 科目固定 → tuple

class_scores: dict[str, list[int]] = {                # 学生→成绩 → dict套list
    "小明": [88, 92, 85],
    "小红": [75, 60, 78],
    "小刚": [90, 88, 92],
    "小丽": [55, 70, 65],
    "小华": [80, 45, 72],
}

# 平均分
for name, scores in class_scores.items():
    avg = sum(scores) / len(scores)
    print(f"{name}: 平均 {avg:.1f}")

# 需要补考的学生（有科目 <60）— 用 set 去重
failing = set()
for name, scores in class_scores.items():
    for score in scores:
        if score < 60:
            failing.add(name)
print(f"补考名单: {failing}")  # {'小华', '小丽'}

# 全科优秀（都 ≥80）
excellent = {name for name, scores in class_scores.items()
             if all(s >= 80 for s in scores)}
print(f"全科优秀: {excellent}")  # {'小刚', '小明'}
```

# 二、循环与控制流

## 2.1 for 循环 — Python 的灵魂

Python 的 `for` 和 JS 不太一样：Python 的 `for` 总是**遍历可迭代对象**，而不是传统的三表达式循环：

```python
# 遍历列表 / 字符串
for fruit in ["苹果", "香蕉", "橘子"]:
    print(fruit)

for char in "Python":
    print(char)       # 字符串也是序列！

# range() — 最常用的数字生成器
range(5)       # 0, 1, 2, 3, 4
range(2, 7)    # 2, 3, 4, 5, 6
range(1, 10, 2)  # 1, 3, 5, 7, 9 （步长为2）

# enumerate() — 同时要索引和值
for i, fruit in enumerate(fruits):
    print(f"[{i}] {fruit}")

# zip() — 同时遍历多个列表
names = ["小明", "小红", "小刚"]
scores = [85, 92, 78]
for name, score in zip(names, scores):
    print(f"{name}: {score}分")
```

> **技巧**：忘记索引的存在，优先用 `for item in iterable`。需要索引时才用 `enumerate()`。

## 2.2 while 循环

```python
countdown = 5
while countdown > 0:
    print(f"{countdown}...")
    countdown -= 1     # 必须改变条件，否则死循环！
print("发射！")
```

## 2.3 break 和 continue

```python
# break — 找到就停
for num in [3, 7, 2, 9, 5]:
    if num == 9:
        print("找到了！")
        break

# continue — 跳过本次
for num in range(1, 11):
    if num % 2 == 0:
        continue      # 偶数跳过
    print(num)        # 只打印奇数
```

## 2.4 for...else — Python 独有特性

这是其他主流语言没有的：`else` 跟在循环后面，**循环正常结束（没被 break）才执行**：

```python
def is_prime(n: int) -> bool:
    for i in range(2, int(n ** 0.5) + 1):
        if n % i == 0:
            print(f"{n} 能被 {i} 整除")
            break          # break → 不执行 else
    else:                 # 循环正常走完 → 执行 else
        print(f"{n} 是素数！")
        return True
    return False
```

这个特性完美解决了"在循环中查找某物，没找到时需要执行默认逻辑"的模式——不用额外设 flag 变量。

# 三、字符串操作

## 3.1 切片规则（和列表完全一致）

```python
text = "Hello Python!"
text[0:5]     # "Hello"
text[6:]      # "Python!"
text[::-1]    # "!nohtyP olleH" — 反转字符串
```

## 3.2 常用方法

```python
"  Hello World!  ".strip()       # 去两端空白
"apple banana apple".count("apple")  # 2 — 计数
"apple banana".replace("apple", "grape")  # 全部替换
"hello".upper()                   # "HELLO"
"Hello".startswith("He")          # True
"123".isdigit()                   # True
```

## 3.3 f-string — 最推荐的格式化方式

```python
name, age = "小明", 15
print(f"我叫{name}，明年{age + 1}岁")  # 花括号里能写表达式！

score = 92.567
print(f"{score:.1f}")    # 92.6 — 保留1位小数
print(f"{1234567:,}")    # 1,234,567 — 千分位

# 对齐
print(f"|{'左对齐':<10}|")
print(f"|{'居中':^10}|")
print(f"|{'右对齐':>10}|")

# 调试技巧 (Python 3.8+)
print(f"{name = }")      # name = '小明' — 变量名和值一起输出！
```

## 3.4 split 和 join — 字符串 ↔ 列表互转

```python
"小明,15,上海".split(",")           # ['小明', '15', '上海']
"Python is fun".split()             # ['Python', 'is', 'fun'] — 默认按空白分

", ".join(["Python", "is", "awesome"])  # "Python, is, awesome"
"".join(["a", "b", "c"])                # "abc"
```

# 四、文件读写

## 4.1 with open() — 永远这样用

`with` 语句会自动关闭文件，即使中途报错也不会漏：

```python
# 写文件 — 'w' 覆盖，'a' 追加
with open("notes.txt", "w", encoding="utf-8") as f:
    f.write("Hello Python\n")
    f.writelines(["第一行\n", "第二行\n"])

# 读文件 — 三种方式
with open("notes.txt", "r", encoding="utf-8") as f:
    content = f.read()          # 方式一：全部读入

with open("notes.txt", "r", encoding="utf-8") as f:
    lines = f.readlines()       # 方式二：按行读入列表

with open("notes.txt", "r", encoding="utf-8") as f:
    for line in f:              # 方式三：逐行迭代（推荐！大文件也不怕）
        print(line.strip())
```

## 4.2 json 持久化 — 结构化数据的正确存储方式

这是我第一天学到的**最重要的教训**。一开始我把 dict 拆成文本行写进文件，读回来只剩字符串，结构全丢了：

```python
# ❌ 错误做法：手动拼文本，读回来结构丢失
f.write(f"{note['date']}\n")
f.write(f"  {note['content']}\n")  # 读回来只是一行行字符串

# ✅ 正确做法：一行 json，结构无损
import json

notes = [{"date": "2024-01-15", "content": "学了 Python", "tags": ["Python"]}]

json.dump(notes, f)       # 写入 — list[dict] 一键保存
notes = json.load(f)      # 读取 — 还原成完全相同的 list[dict]
```

> **核心认知**：结构化数据 → 用 json。手动拼文本看似简单，但数据读回来就废了。

# 五、实战练习

每个知识点后面都跟着一个练习，这里挑两个代表性强的展示：

## 5.1 猜数字游戏（循环 + 控制流）

```python
import random

target = random.randint(1, 10)
max_attempts = 5

for attempt in range(1, max_attempts + 1):
    guess = int(input(f"第{attempt}次猜: "))
    if guess < target:
        print("太小了！")
    elif guess > target:
        print("太大了！")
    else:
        print(f"猜中了！用了 {attempt} 次")
        break
else:
    print(f"游戏结束，答案是 {target}")  # for...else！
```

## 5.2 文本统计分析器（字符串 + 字典 + 集合）

```python
article = "Python was created by Guido. Python is easy. Python is powerful."

words = article.lower().replace(".", "").split()  # 分词
unique = set(words)                                 # 去重

word_count = {}
for word in words:
    word_count[word] = word_count.get(word, 0) + 1  # 词频统计

print(f"总词数: {len(words)}, 不重复: {len(unique)}")
```

## 5.3 笔记管理器（文件 + json + 菜单交互）

```python
import json

notes = []
if Path("notes.json").exists():
    with open("notes.json", "r", encoding="utf-8") as f:
        notes = json.load(f)

while True:
    print("1.写笔记  2.查看  3.搜索  4.退出")
    choice = input("请选择: ")

    if choice == "1":
        note = {"date": input("日期:"), "content": input("内容:"),
                "tags": input("标签(逗号分隔):").split(",")}
        notes.append(note)
    elif choice == "2":
        for n in notes:
            print(f"[{n['date']}] {n['content']}")
    elif choice == "3":
        kw = input("搜索:").lower()
        for n in notes:
            if kw in n["content"].lower():
                print(f"[{n['date']}] {n['content']}")
    elif choice == "4":
        with open("notes.json", "w", encoding="utf-8") as f:
            json.dump(notes, f)
        break
```

# 学习心得

第一天总共写了 **4 个教程文件 + 4 个练习文件**，大约 300 行代码。几个比较深的感受：

**1. Python 的简洁是真实的**

列表推导式、元组解包、f-string 调试模式、`for...else`——这些语法糖不是花哨，而是确实减少了日常代码量。特别是列表推导式，工作半年后回头看估计会觉得 JS 的 `filter().map()` 很啰嗦。

**2. 数据类型选对事半功倍**

用 `set` 做去重和成员判断、用 `dict.get()` 替代 `[]` 安全访问、用 `tuple` 承载不可变配置——这些习惯从一开始建立比后期纠正容易得多。

**3. 结构化数据就该用 json**

第一版笔记管理器用纯文本存数据，读回来结构全丢，自己都感觉不对劲。换成 `json.dump/load` 后代码少了三分之一，数据保真度 100%。**千万别为了"少学一个模块"而用错工具。**

**4. 写代码和写文章是两种收获**

把学到的知识写成这篇博客，逼着我重新梳理每个知识点的"为什么"和"怎么用"，比单纯敲代码理解更深。以后每个阶段学完都值得写一篇。

---

> 项目地址：[github.com/halely/python-study](https://github.com/halely/python-study)
>
> 下一篇预告：Python 函数进阶——`*args / **kwargs`、模块与包、异常处理。
>
> 学习时间：2026-06-01
