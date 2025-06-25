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

- `lamda 参数：表达式`：

  匿名函数表达式

  ```python
  # 普通函数
  def add(x, y):
      return x + y
  
  # lambda 函数（匿名函数）
  add2 = lambda x, y: x + y
  print(add2(2, 3))  # 输出 5
  ```

  - 使用场景（常配合`sorted`、`map`、`filter`、`reduce`）：

    ```python
    # 对列表中的元组，按第二个元素排序：
    items = [('a', 3), ('b', 1), ('c', 2)]
    sorted_items = sorted(items, key=lambda item: item[1])
    print(sorted_items)  # [('b', 1), ('c', 2), ('a', 3)]
    ```

    



### 数组函数

列表（`list`）

#### 创建列表

`arr = [1,2,3,4]`

#### 常用函数与方法

- 添加元素

  ```python
  arr.append(5) # 末尾追加元素
  arr.insert(2,10) # 在索引2插入10
  arr.extend([6,7]) # 扩展多个元素
  ```

- 删除元素

  ```python
  arr.remove(2) # 删除值为2的第一个元素
  arr.pop() # 删除并返回最后一个元素
  arr.pop(1) # 删除并返回索引1的元素
  del arr[0] # 删除索引0的元素
  arr.clear() # 清空列表
  ```

- 查找元素

  ```python
  arr.index(3) # 返回值为3的第一个索引,没有会报错
  arr.count(3) # 返回值为3的出现次数
  ```

- 排序与反转

  ```python
  arr.sort() # 升序排序（原地修改）
  arr.sort(reverse=True) # 降序
  arr.reverse() # 反转列表顺序
  ```

- 拷贝与复制

  ```python
  new_arr = arr.copy() # 浅拷贝
  new_arr = arr[:] # 切片复制
  ```

- 长度、最大/最小/和

  ```python
  len(arr) # 长度
  max(arr) # 最大值
  min(arr) # 最小值
  sum(arr) # 元素总和
  ```

- 判断元素是否存在

  ```python
  if 3 in arr:
      print("存在")
  ```

- 遍历列表

  ```python
  for x in arr:
      print(x)
  
  for i, x in enumerate(arr):
      print(f"第{i}个元素是{x}")
  ```

- 列表推导式（生成新列表）

  ```python
  squares = [x**2 for x in arr]
  evens = [x for x in arr if x % 2 == 0]
  ```

  

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





### `pandas`常用函数和方法

- 基础导入和创建：

  ```
  import pandas as pd
  ```

  创建`DataFrame`：

  ```python
  df = pd.DataFrame({'name':['Tom','Jerry'],'age':[18,20]})
  ```

- 数据读取与写入

  ```python
  pd.read_csv('file.csv') # 读取CSV文件
  pd.read_excel('file.xlsx') # 读取Excel文件
  ```

- 数据查看与基本操作

  ```python
  df.head(5) # 查看前5行
  df.tail(3) # 查看后3行
  df.shape # 查看维度（行数，列数）
  df.info() # 查看基本信息
  df.describe() # 查看数值列统计信息
  df.columns # 列名
  df.index # 索引
  ```

- 数据选择

  ```python
  df['name'] # 选择单列
  df[['name','age']] # 选择多列
  ```

- 数据处理常用

  ```python
  df.drop(columns = ['age']) # 删除列
  df.drop(index = [0]) # 删除行
  df.rename(columns = {'name':'Name'}) # 重命名
  df.sort_values(by = 'age') # 按列排序   第二个可选参数ascending = False为降序
  df.fillna(0) # 缺失值填充
  df.dropna() # 删除缺失值
  df.duplicated() # 查重复
  df.drop_duplicates() # 删除重复
  df.round({"销售占比":6}) # 将"销售占比"这一列的数值四舍五入到小数点后六位
  ```

- 分组和聚合

  ```python
  df.groupby('gender').mean() # 分组求平均  第二个可选参数as_index = False：结果中保留gender为普通列，而不是
  df.groupby('class')['score'].sum() # 分组求和
  df['score'].agg(['mean','max']) # 多个聚合
  ```

- 条件筛选

  ```python
  df[df['age'] > 18] # 筛选
  df[(df['age'] > 18) & (df['score'] > 80)] # 多条件筛选
  ```

- 新列与运算

  ```python
  df['new_col'] = df['score'] * 2 # 添加新列
  df['pass'] = df['score'] >= 60 # 判断列
  ```

  