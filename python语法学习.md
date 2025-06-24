## python语法学习

### 基础语法

- 变量声明：`x = 10`

- 常量声明：`PI = 3.14`

- 函数定义：

  ```python
  def add(a,b):
  	return a + b 
  ```

- 条件判断：`if x > 0:`
- 循环结构：`for i in range(5):`
- 数组/切片：`arr = [1,2,3]`
- 输入：`input()` 默认读取字符串，然后类型转换
- 打印输出：`print("Hello")`
- 字符串拼接：`"hello " + name`
- 注释：`# 这是注释`
- 模块导入：`import math`



### 内置函数

- `map(function,iterable,...)`：
  - `function`：一个函数，作用于每个函数
  - `iterable`：一个或多个可迭代对象（列表、元祖等）





### 字符串函数

#### 基本方法

- `s.lower()`：转小写
- `s.upper()`：转大写
- `s.strip()`：去除首尾空白字符
- `s.lstrip()/s.rstrip()`：去除左/右空白字符
- `s.replace(old,new)`：替换子串
- `s.split(sep)`：按分隔符拆分为列表
- `sep.join(list)`：用分隔符连接列表

#### 查找与判断

- `s.startswith(prefix)`：是否以某前缀开头
- `s.endswith(suffix)`：是否以某后缀结尾
- `s.find(sub)`：返回子串首次出现的位置，找不到返回-1
- `s.count(sub)`：统计子串出现次数
- `s.isdigit()`：是否全是数字字符
- `s.isalpha()`：是否全是字母
- `s.isalnum()`：是否全是字母或数字

#### 格式化字符串（插值）

```python
# 1.f-string（格式化字符串字面量）
s1 = f"My name is {name},age {age}"

# 2.format()
s2 = "My name is {},age {}".format(name,age)

# 3.% 格式化
s3 = "My name is %s,age %d" % (name,age)
```

