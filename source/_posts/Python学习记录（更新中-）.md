title: Python学习记录（更新中......）
author: Mario
date: 2024-11-18 15:18:04
tags:
---
python学习中，记录知识点
<!--more-->

### 注释

``` bash
# 这是一个单行注释

"""
这是一个多行注释
"""

'''
这也是一个多行注释
'''
```
### 变量
我们的Python程序，处理的所有数据都必须在变量中。怎么去理解变量呢?官方的定义就是是变化的量叫变量，每一个变量都有变量名和变量值。从字面上看每个字都认识，但是好像不太好理解。所有的变量都存在内存中。你把内存看成是一个大的五星酒店。每一间房就代表数据存放的内存地址范围。在这一个地址范围中存起来的就是我们的数据，对应客房中所住的客人。为了快速的查我使用这个数据，通常我们把存储数据的地址范围和房间号一样，也定一个名称，这个名称就是变量。总结一下，房间就对应变量，房号对应变量名，入住的客人对应变量值，有了变量之后就可以快速的在茫茫(数海)的内存中，找到对应的数。

##### 一、标识符（变量）命名规则

Python中定义各种名字的时候的一规范，具体如下:
- 由数字、字母、下划线组成
- 不能数字开头
- 严格区分大小写
- 不能使用内置关键字(如下)

![upload successful](/images/pasted-38.png)


##### 二、命名习惯
- 见名知义
- 单词之间用下划线隔开，例如：my_name

``` bash
# 变量声明例子
name = 'Mario'
age = 30
# 两个变量共用一个值
name01 = name02 = ‘名字’
# a=100 b=200
a,b = 100,200
```
##### 三、数据类型

Python是强类型语言，检测数据类型的函数：type()

![upload successful](/images/pasted-39.png)
``` bash
# 字符串
name = 'Mario'
name2 ="test"
# 整型
age = 30
# 浮点型
high = 1.78
# 布尔型
a = False
```
### Python的输入与输出格式化
##### 一、Python的输入

input("提示信息")

- 当程序执行到 input，等待用户输入，输入完成之后才继续向下执行。
- 在Python中，input 接收用户输入后，一般存储到变量，方便使用。
- 在Python中，input 会把接收到的任意用户输入的数据都当做字符串处理。

<strong>注意:所有的通过input获取的数据,都是字符串类型。 Python是一门强类型的语言</strong>

##### 二、类型转换

转换数据类型常用的函数
- int()
- float()
- str()
- list()
- tuple()
- eval()

![upload successful](/images/pasted-40.png)
##### 三、Python的输出格式化

一、用%(占位符)进行格式化输出

![upload successful](/images/pasted-41.png)
<strong>技巧</strong>
- %s，可以输出字符串，也可以是数字
- %06d，表示输出的整数显示位数，不足以0补全，超出当前位数则原样输出
- %.2f，表示小数点后显示的小数位数。

<strong>注意:因为计算机只认识和处理和二进制的数值，在实际的计算中，所有的数据都必须为二进制，计算后又转换回来，就是在这个过程中造成数据的截断误差。</strong>

``` bash
# 格式化输出
x = 'mario'
age=26
high = 1.77
print("我是%s 我的年龄是%d 我的身高%fm" % (x,age,high))

# 精确的四舍五入
from decimal import Decimal
num = 1.773242353
# 四舍五入，保留两位小数
resu=Decimal(num).quantize(Decimal("0.00"),rounding="ROUND_HALF_UP")
print(resu)
```
二、用f-string方法格式化输出

格式化字符串除了%s，还可以写为 f'{表达式}
``` bash
# 格式化输出
x = 'mario'
age=26
high = 1.77
print(f"我是{x} 我的年龄是{age} 我的身高{high}m")
```
### Python的运算符
运算符的分类

- 算术运算符
- 赋值运算符
- 复合赋值运算符
- 比较运算符
- 逻辑运算符

##### 一、算数运算符

![upload successful](/images/pasted-42.png)

``` bash
# 2的平方
print(2**2)
pow(2,2)
# 2的立方
print(2**3)
pow(2,3)
# 8的立方根
print(8**(1/3))
pow(8,1/3)
# 4的平方根
print(4**0.5)
pow(4,0.5)
```
##### 二、赋值运算符

![upload successful](/images/pasted-43.png)

##### 三、复合赋值运算符

![upload successful](/images/pasted-44.png)
<strong>注意：先计算后赋值</strong>

##### 四、比较运算符

比较运算符也叫关系运算符，通常用用来判断。

![upload successful](/images/pasted-45.png)

##### 五、逻辑运算符

![upload successful](/images/pasted-46.png)
- 与:and
- 或:or
- 非: not

### Python的流程控制语句
两种流程控制语句:
- 条件语句
- 循环语句

##### 一、条件语句

让程序根据条件有选择性的执行语句。
- if子句必须有
- elif 子句可以有0个或多个
- else 子句可以有0个或1个，且只能放在if语句的最后

![upload successful](/images/pasted-47.png)

``` bash
a = False
print(type(a))
if a:
    print("真")
else:
    print("假")
```

一、if嵌套

语法：
![upload successful](/images/pasted-50.png)
<strong>注意:条件2的if也是处于条件1成立执行的代码的缩进关系内部。</strong>
``` bash
a = False
print(type(a))
if a:
    print("真")
    if a:
       print("真")
    else:
        print("假")
else:
    print("假")
```
二、三目条件运算

三目运算也叫三元运算: 是为了快速给一个变量赋值，采用简单的条件语句。

语法如下：满足条件的值1 if 条件 else 不满足条件的值2

``` bash
num = 2
result = f'当前的数字{num}是偶数' if num %2 == 0 else f'当前的数字{num}是奇数'
print(result)
```
##### 二、循环语句

可以让一段代码，重复执行。
- while循环
- for循环

![upload successful](/images/pasted-51.png)

``` bash
#计算1~100的所有数和
my_sum=0 #定义一个结果
n = 1
while 1<=n<= 100:
  my_sum += n
  n += 1
print(f'结果是:{my_sum}')

#计算1~100的所有数和
my_sum=0 #定义一个结果
for n in range(1,100):
    my_sum += n
print(f'结果是:{my_sum}')
```
一、range函数

用来创建一个生成一系列整数的可迭代对象(也叫整数序列生成器)。

语法是:范围(开始点)，结束点，间隔))

<strong>口诀:包头不包尾</strong>
``` bash
#计算1~100的所有数和
my_sum=0 #定义一个结果
for n in range(1,100):
    my_sum += n
print(f'结果是:{my_sum}')
```

二、break和continue

break和continue都是用来控制循环结构的，主要作用是停止循环。

1、break用于<strong>跳出一个循环体或者完全结束一个循环</strong>，可以结束其所在的循环
- 结束当前整个循环，执行当前循环下边的语句。忽略循环体中任何其它语句
- 只能跳出一层循环，如果你的循环是嵌套循环，那么你需要按照你嵌套的层次，逐步使用break来跳出。【逐层逐步跳出】

2、continue语句的作用是<strong>跳过本次循环体中剩下尚未执行的语句，立即进行下一次的循环条件判定</strong>，可以理解为只是中止(跳过)本次循环，接着开始下一次循环。
- 终止本次循环的执行，即跳过当前这次循环中continue语句后尚未执行的语句，接着进行下一次循环条件的判断。
- 终止当前的循环过程，但他并不跳出循环,而是继续往下判断循环条件执行语句.他只能结束循环中的一次过程,但不能终止循环继续进行。

<strong>注意:break和continue只能用于循环语句中;并且:在嵌套循环中使用时，只对最内层循环有效。</strong>

### Python中的序列

Python中序列:
- 字符串
- 列表
- 元祖

##### 一、字符串
一、定义

字符串是 Python 中最常用的数据类型。我们一般使用引年(单引号、双引号和三引号都可以)来创建字符串。字符串是由一个一个的字符组成的。
<strong>注意:字符串中可能会包含一种特殊字符转义符</strong>
``` bash
var1 = "Hello Word!"
var2 = 'Hello Word!'
var3 = '''Hello Word!'''
```
二、下标

“下标”又叫“索引”，就是编号。比如:学号，座位号，座位号的作用:按照编号快速找到对应的座位。同理，下标的作用即是通过下标快速找到对应的数据。(所有序列都有下标)

<strong>注意:正向索引从0开始，反过来从-1开始</strong>

![upload successful](/images/pasted-52.png)
三、切片

切片是指对操作的对象截取其中一部分的操作。字符串、列表、元组都支持切片操作。
序列对象【开始位置下标:结束位置下标:步长】; 口诀:包头不包尾。
``` bash
name ="abcdefg"
print(name[2:5:1])# cdeprint(name[2:5])# cde
print(name[:5])# abcde
# bcdefgprint(name[1:])
print(name[:])# abcdefg
print(name[::2])# aceg
# abcdef，负1表示倒数第一个数据print(name[:-1])
print(name[-4:-1])# def
print(name[::-1])# gfedcba
```
四、字符串中的函数

1、字符串查询(index，find)

建议使用find，因为如果没有找到匹配的字符串，index方法会报异常

![upload successful](/images/pasted-53.png)

2、字符串大小写转换操作(upper、lower、swapcase、capitalize和title)

![upload successful](/images/pasted-54.png)

3、字符串对齐(center、just和zfill)

![upload successful](/images/pasted-55.png)

4、分割字符串(split、splitlines和partition)

![upload successful](/images/pasted-56.png)

5、合井与替换(join、replace)

![upload successful](/images/pasted-57.png)

6、判断字符串(isidentifier、isspace、isalpha、isdecimal、isnumeric和isanum等)

![upload successful](/images/pasted-58.png)

![upload successful](/images/pasted-59.png)

7、去除两端多余字符操作(strip)

![upload successful](/images/pasted-60.png)

##### 二、列表List
一、定义

由一系列变量组成的可变序列容器。列表可以一次性存储多个数据，且可以为不同数据类型。
``` bash
[数据1、数据2，数据3......]
lit = [1,2,3,4,'a','b','c']
```
二、列表的操作

- <strong>下标和切片：</strong> 查找、修改、截取
- <strong>index函数：</strong>返回指定数据所在位置的下标
- <strong>count函数：</strong>统计指定数据在当前列表中出现的次数。
- <strong>len函数(内置的)：</strong>访问列表长度，即列表中数据的个数。
- <strong>append函数：</strong>列表结尾追加数据。
- <strong>extend函数：</strong>列表结尾追加数据，如果数据是一个序列，则将这个序列的数据逐一添加到列表。
- <strong>insert函数：</strong>指定位置新增数据
- <strong>pop函数：</strong>删除指定下标的数据(默认为最后一个)，并返回该数据。
- <strong>del表达式：</strong>内置的删除
- <strong>remove函数：</strong>移除列表中某个数据的第一个匹配项,
- <strong>reverse函数：</strong>逆序
- <strong>sort函数：</strong>重新排序
- <strong>for循环遍历</strong>

##### 三、元组
一、定义

由一系列变量组成的不可变序列容器。不可变是指一但创建，不可以再添加/删除/修改元素。元组特点:元组使用小括号，且逗号隔开各个数据，数据可以是不同的数据类型，和列表一样都是有顺序的。
``` bash
元组名=(1,2,3)
yz=(1,2,3,4,5,6,7,'a','b')
# 元组中只有一个元素时，最后要加个逗号
元组名=(1,)
yz1=('abc',)
```
二、元组的操作

- <strong>下标和切片：</strong> 查找、截取操作 I
- <strong>index函数：</strong>返回指定数据所在位置的下标
- <strong>count函数：</strong>统计指定数据在当前列表中出现的次数。
- <strong>len函数(内置的)：</strong>访问列表长度，即列表中数据的个数。
- <strong>for循环遍历</strong>

##### 四、序列的通用操作
一、数学运算符

- +：用于拼接两个序列

- +=：用原序列与右侧序列拼接,并重新绑定变量

- *：重复生成序列中的元蒸

- *=：用原序列生成重复元素,并重新绑定变量

- < <= > >= == !=：依次比较两个序列中元素,一但不同则返回比较结果

二、成员判断
- 数据 in 序列
- 数据 not in 序列

三、Python内置函数操作

- len(x)返回序列的长度
- max(x)返回序列的最大值元素
- min(x)返回序列的最小值元素
- sum(x)返回序列中所有元素的和(元素必须是数值类型)

### Python的内置容器

##### 一、容器的概念

Python中，可包含其他对象的对象，称之为“容器”。容器是一种数据结构。常用的容器主要划分为两种:序列(如:列表、元组等)和映射(如:字典)。列中，每个元素都有下标，它们是有序的。映射中，每个元素都有名称(又称“键")，它们是无序的。除了序列和映射之外，还有一种需要注意的容器-"集合"。

![upload successful](/images/pasted-62.png)

##### 二、字典
字典特点:

- 符号为:大括号
- 数据为:键值对 形式出现;<strong>字典中的“键”必须是独一无二的</strong>
- 键和值之间用:冒号 隔开

注意:般称冒号前面的为键(key)，简称k;冒号后面的为值(value)，简称v。<strong>合起来称之为“项”。</strong>

1、定义字典

字典是 Python 中的唯一内置映射，和之前所提到的列表、字符串一样，字典也拥有它的转换函数-- dict。
``` bash
# 创建字典
dict1 = {} #空字典
dict2 = {'name' : 'mario','age' : 30}
dict3 = dict([('name','mario'),('age',30)])
dict4 = dict(name='mario',age=30)
```
2、常见操作
- 增/改/删除

``` bash
#增、删、改字典
dict5 = {'name' : 'mario','age' : 30}
dict5['age'] = 20
del dict5['age']
```
- 查找

  1、<strong>len(dict)：</strong>返回字典 dict 对应的项数。

  2、<strong>d[ k ]：</strong>返回与键k相应的值。

  3、<strong>k in d：</strong>检査键k是否包含于字典 d。

- 字典中函数

  1、-clear：可以清除字典中的所有数据
  
  2、-fromkeys：使用该方法可以创建一个新字典，其中包含指定的键，且每个键对应的值都是 None
  
  3、-get：直接访问字典中的值
  
  4、-items：会返回一个包含所有字典项的列表，其中每个元素都为键值对的形式，且排列顺序不确定。
  
  5、-keys：返回字典中的所有key，组成一个列表
  
  6、-values：返回字典中所有的value.
  
  7、-pop：删除并返回指定key的值
  
##### 三、Set集合
由一系列不重复的不可变类型变量组成的可变散列容器。相当于只有键没有值的字典(键则是集合的数据)。

<strong>创建集合</strong>使用{}或 set(),但是如果要创建空集合只能使用set(),因为 {} 用来创建空字典

<strong>特点:
  
1、集合中不能出现重复的数据，自动去掉重复数据

2、集合数据是无序的，故不支持下标</strong>

``` bash
set1 = set()
set2={1,2,3,'123'}
set1.add(99)
set1.add('hello')
print(set1)
# 删除
set1.remove('hello')
print(set1)
```
##### 四、总结
字符串str：储存字符编码值,不可变

序列列表list：储存变量,可变

序列元组tuple：储存变量,不可变

序列字典dict：储存键值对,可变,散列;键不能重复且不可变

集合set：储存键,可变,散列

![upload successful](/images/pasted-63.png)

- 不可变:数据在内存中本质都是不可变,采用按需分配的存储机制;就更无法改变内存中的值。

- 可变:具有扩容能力,采用预留空间的存储机制

<strong>不可变:基本类型(字符串，int，float等)，tuple</strong>

<strong>可变:list，dict，set</strong>

### Python的函数

##### 一、函数的定义

<strong>函数是组织好的，可重复使用的，用来实现相关功能的代码段。</strong>

函数能提高应用的模块性，和代码的重复利用率。Python提供了许多内建函数，比如print()。我们也可以自己创建函数，这被叫做用户自定义函数。

1、定义函数
``` bash
def test(a,b):
    return a+b
```
2、调用函数
``` bash
print(test(1, 2))
```
<strong>注意:</strong>
- 不同的需求，参数可有可无
- 在Python中，函数必须先定义,后使用

##### 二、函数中的参数

1、必要传参，也叫位置参数

定义函数时，根据需求必需要传递的参数工而且，在调用函数时根据函数定义的参数位置顺序来传递参数。
``` bash
# 位置参数
def test(a,b):
    return a+b

print(test(1, 2))
```

<strong>注意:传递和定义参数的顺序及个数必须一致</strong>

2、关键字传参

函数调用，通过“键=值“形式加以指定。可以让函数更加清晰、容易使用，同时也清除了参数的顺序需求。

<strong>注意:函数调用时，如果有位置参数时，位置参数必须在关键字参数的前面，但关键字参数之间不存在先后顺序。</strong>

``` bash
# 不定长名称参数+默认值参数
def test3(init_sum=1,**kwargs):
    return init_sum+sum(kwargs.values())
```
3、默认传参

用于定义函数，为参数提供默认值，调用函数时可不传该默认参数的值(注意:所有位置参数必须出现在默认参数前，包括函数定义和调用)

``` bash
# 位置参数+默认值参数
def test1(a,b,init_sum=10):
    return a+b+init_sum

print(test1(1, 2))
```
4、不定长传参

不定长参数也叫可变参数。用于不确定调用的时候会传递多少个参数(不传参也可以)的场景。此时，来进行参数传递，会显得非常方便。

- 不定长普通参数
- 不定长关键字参数

``` bash
# 不定长参数+默认参数
def test2(*args,init_sum=10):
    return init_sum+sum(args)


print(test2(1, 2, 3, 4, 5, init_sum=12))
```
<strong>参数顺序 位置参数->默认值参数->不定长普通参数->不定长关键字参数</strong>

##### 三、函数的返回值

return 语句用于返回函数的值，并且退出函数，选择性地使用return 语句， 默认是返回 None
 - return a，b写法，返回多个数据的时候，默认是元组类型。
 - return后面可以连接列表、元组或字典，以返回多个值。
 
##### 四、局部变量和全局变量
1、局部变量

就是在函数内部定义的变量;其作用范围是这个函数内部，即只能在这个函数中使用，在函数的外部是不能使用的;
因为其作用范围只是在自己的函数内部，所以不同的函数可以定义相同名字的局部变量当函数调用时，局部变量被创建，当函数调用完成后这个变量就不能够使用了
``` bash
a = 1
def test2():
    global a # 在函数内部，使用global声明a为全局变量
    a=30 #对全局变量a 进行的修改
    print(a)
test2()
print(a)
```
2、全局变量

全局变量和局部变量的区别在于定义在函数的外面，全局变量在整个py文件中声明，全局范围内可以使用

<strong>注意: 当函数内出现局部变量和全局变量相同名字时，函数内部中的 变量名 = 数据 ，此时理解为定义了-个局部变量，而不是修改全局变量的值。如果要修改全局变量，必须使用global。</strong>

##### 五、总结

- 函数的定义:

  可重复使用的，用来实现某个功能的代码段。
- 函数使用

  定义函数
  ``` bash
   def 函数名():
      代码1
      代码2
      ...
  ```
  调用函数
  ``` bash
     函数名()
  ```
- 函数的参数:

  *必要传参
  
  *默认传参
  
  *关键字传参
  
  *不定长传参
  
- 函数的返回值

  *作用:函数调用后，返回需要的计算结果
  
  *return关键字
  
- 局部变量和全局变量

  *局部变量:在函数内部定义的变量，只能在函数内部使用
  
  *全局变量:是在函数外部定义的变量，所有函数内部都可以使用这个变量
  
##### 六、易错题

``` bash
#定义一个函数
def test(a,lst1=[1,2]):
    # 把a添加到列表中
    if a not in lst1:
      lst1.append(a)
    return lst1
print(f'第一次调用test函数的结果是:{test(10)}')#1、2，10
print(f'第二次调用test函数的结果是:{test(20)}')#1、2、20 正确的是:1，2，10，20
print(f'第三次调用test函数的结果是::{test(30,lst1=[60,70])}') #正确的是:60，70，30
print(f'第四次调用test函数的结果是:{test(40)}')#正确的是:1，2，10，20,40
```
### 递归函数和高阶函数

##### 一、递归函数
递归函数: 在函数的内部自己调用自己，并且有退出函数的出口。

<strong>应用:</strong>

- 如果要遍历一个文件夹下面所有的文件，通常会使用递归来实现，
- 很多算法都离不开递归，例如:快速排序。

![upload successful](/images/pasted-64.png)

``` bash
# 需求:计算一个正整数n的阶乘
def test(n: int)-> int:
  '''
  计算一个数字n的阶乘
  :param n:
  :return:
  '''
  if n == 1:
    return 1 #递归函数退出的出口
  return n* test(n-1)

print(test(2))
```
##### 二、匿名函数:lambda
如果一个函数有一个返回值，并且只有一句代码，可以使用lambda简化。
``` bash
lambda 参数列表: 表达式
```
<strong>注意:</strong>

- lambda表达式的参数可有可无;
- 函数的参数在lambda表达式中完全适用。
- lambda表达式能接收任何数量的参数但只能返回一个表达式的值。
- 直接打印lambda表达式，输出的是此lambda的内存地址
``` bash
lambdaFun = lambda a,b: a+b
print(lambdaFun(1, 2))
```

使用匿名函数做判断:

``` bash
  fn1 = lambda a, b: a if a > b else b
  print(fn1(1000,580))

```

使用匿名函数和sort函数结合完成排序:
``` bash
#需要给某个复杂的列表排序
lst =[
{'name':'张三','age': 34},
{'name':'李四','age': 34},
{'name':'王五','age': 34}]
# 根据年龄来排序(降序)
lst.sort(key=lambda item: item['age'],reverse=True)
print(lst)
```
##### 三、高阶函数
把函数作为参数传入，或者返回值是另外一个函数，这样的函数称为高阶函数，高阶函数是函数式编程的体现。函数式编程就是指这种高度抽象的编程范式。函数式编程大量使用函数，减少了代码的重复，因此程序比较短，开发速度较快。

1、函数的参数是函数
``` bash
#对任意两个数字，整理之后再求和
def sum_num(a, b):
   return abs(a)+ abs(b)

#高阶函数的实现
def sum_num2(a,b,f):
    '''
    :param a:
    :param b:
    :param f:就是对两个数字进行整理的函数
    :return:
    '''
    return f(a)+f(b)
#通过绝对值整理之后再求和
print(sum_num2(2,6,abs))
#通过平方整理之后再求和
print(sum_num2(2,6,lambda n:n**2))
```
2、函数的返回值是函数

高阶函数除了可以接受函数作为参数外，还可以把函数作为结果值返回，一个函数返回值(return)为另外一个函数。
``` bash
def sum_fun_b(*args):
    def sum_a():
        a=0
        for n in args:
            a=a+n
        return a
    return sum_a
```
##### 四、Python中内置的高阶函数
1、map函数

map函数接收的是两个参数，一个是函数名，另外一个是序列，其功能是将序列中的数值作为函数的参数依次传入到函数值中执行，然后再返回到列表中。返回值是一个迭代器对象。

![upload successful](/images/pasted-65.png)
``` bash
# map函数
# 结果为[1,4,9,16,25,36]
print(list(map(lambda n:n **2,[1,2,3,4,5,6])))
```
2、reduce函数

reduce函数也是一个参数为函数，另一个参数为序列对象(比如: list列表)。其返回值为一个值而不是迭代器对象，故其常用与叠加、叠乘等等。

函数详解:

- function:一个有两个参数的函数
- sequence:是一个序列，是一些数据的集合，或者是一组数据，可迭代对象
- initial:可选，初始参数
- 返回值:返回函数计算的结果
- reduce()函数,使用function函数(有两个参数)先对集合中的sequence第 1、2 个元素进行操作，如果5.存在initial参数，则将会以sequence中的第一个元素和initial作为参数，用作调用，得到的结果再与sequence中的下个数据用 function 函数运算，最后得到一个结果

![upload successful](/images/pasted-66.png)
``` bash
#reduce函数
from functools import reduce
print(reduce(lambda x,y:x+y,[2,4,6,8,10],10))
```
``` bash
# 案例:给你很长的字符串,统计字符串中每个单词出现的次数
str1 = 'hello world python hello python java hello python flask'
# 第一步,把单词切开
lst =str1.split('')
#第二部:每个单词只要出现了,那么就代表有一次
#['hello':1,'world':1,...]
new_lst = list(map(lambda item:{item: 1},lst))
#第三步:调用reduce实现相同单词的叠加
def func(dict1,dict2):
    #把dict1作为叠加的返回字典
    key =list(dict2.items())[0][0] # 得到dict2中的key(单词:world)
    value =list(dict2.items())[0][1] # 得到dict2中的value(1)
    dict1[key]= dict1.get(key, 0)+ value
    return dict1
from functools import reduce
print(reduce(func, new_lst))
```
3、filter函数

Python内建的 filter()函数用于过滤序列，和 map()类似，filter()也接收一个函数和一个序列;但是不同的是 filter()把传入的函数依次作用于每个元素，然后根据返回值是 True 还是 False 决定元素的保留与丢弃。
``` bash
lst1 = [1,2,3,4,5,6,7,8,9,10]
# 偶数留下
print(list(filter(lambda n:n%2 ==0,lst1)))
```
4、sorted函数

Python内置的sorted()函数就可以对list进行排序;和序列中本身的sort函数类似。

``` bash
sorted([36,5,-12,9,-21])
#[-21,-12,5,9,36]

sorted([36,5,-12,9,-21],key=abs)
#[5,9,-12,-21,36]

sorted(['bob','about','Zoo','Credit'],key=str.lower)
#['about','bob','Credit','Zoo']
```
默认情况下，对字符串排序，是按照ASCII的大小比较的，由于'Z'<'a'，结果，大写字母Z会排在小写字母a的前面。所以，需要忽略大小写来比较两个字符串，实际上就是先把字符串都变成大写(或者都变成小写)，再比较。

### 文件和目录的操作

##### 一、IO流(Stream)

通过“流”的形式允许计算机程序使用相同的方式来访问不同的输入/输出源。stream是从起源(source)到接收的(sink)的有序数据。我们这里把输入/输出源对比成“水桶”，那么流就是“管道”。

<strong>文件流：就是源或者目标都是文件的流。</strong>

![upload successful](/images/pasted-67.png)
##### 二、文件流的操作

- 打开文件流

  文件对象 = open(目标文件,访问模式(默认读),编码格式)
  ``` bash
	# 创建（打开）文件流
   f = open('test.txt',mode='r',encoding='utf-8')
  ```
- 读操作

  文件对象.read()：默认读取整个文件。 或者可以读取指定大小的数据

  文件对象.readlines()

  文件对象.readline()
   ``` bash
	# 创建（打开）文件流
  f = open('test.txt',mode='r',encoding='utf-8')
  # 读取文件内容  
  # f.read(10) 读取10个字符
  print(f.read())
  # 每次读取一行数据
  print(f.readline())
  # 读取所有行数据，返回一个装载每一行数据的数组
  print(f.readlines())
  # 关闭文件流
  f.close()
   
  ```
- 写操作

  文件对象.write()
   ``` bash
  # 创建（打开）文件流
  wf = open('test1.txt',mode='w+',encoding='utf-8')
  wf.write('测试写文件')
  # 关闭文件流
  wf.close()
   
  ```
- seek()指针操作

  seek(偏移量，起始位置)：起始位置分为：0:起始位置、1:当前位置、2:文件结尾位置
  
  tell()函数返回当前指针位置。
  
- 关闭

  close()
- 访问模式

![upload successful](/images/pasted-69.png)

##### 三、with语句

对于系统资源如文件、数据库连接、socket 而言，应用程序打开这些资源并执行完业务逻辑之后，必须做的一件事就是要关闭(释放)该资源。

比如 Python 程序打开一个文件，往文件中写内容，写完之后，就要关闭该文件，如果不关闭会出现什么情况呢? 极端情况下会出现<strong> Too many open files </strong>的错误，因为系统允许你打开的最大文件数量是有限的。

<strong>使用with语句可以帮助程序员自动关闭资源</strong>
``` bash
with open('test2.txt',"w") as f:
    f.write("hello python")
```
##### 四、文件和文件夹的操作

在Python中文件和文件夹的操作要借助os模块里面的相关功能。OS模块是Python标准库中的一个用于访问操作系统功能的模块。

首先需要导入OS模块: import os

![upload successful](/images/pasted-70.png)

代码演示：
``` bash
import os
# 打印系统名称
print(os.name)
# 打印当前工作目录
print(os.getcwd())
```
### Python面向对象编程

##### 一、类和对象

面向对象是一种抽象化的编程思想，很多编程语言中都有的一种思想。一切皆对象类:是对一系列具有相同 特征 和 行为 的事物的统称。

对象:对象是基于类创建出来的真实存在的事物。

1、定义类

<strong>注意:类名要满足标识符命名规则，同时遵循:大驼峰命名习惯</strong>
``` bash
class 类名():
    属性1
    属性2
    属性3
    函数1
    函数2
```
定义类示例：
``` bash
# 描述一辆汽车
class Car():
     # self:指的是当前对象（实例）本身
    def __init__(self):
        # 汽车的品牌
        self.brand = None
        # 型号
        self.type_name = None
        # 类型 SUV、轿车
        self.category = None

    def run(self): # run函数就是车的行为
        print("Car run")
```
2、创建对象

<strong>创建对象的过程也叫实例化对象。</strong>
``` bash
c1 = Car()
c1.run()
```
3、self是什么意思

self指的是当前对象（实例）本身

4、访问对象的属性和函数

   类外面访问

   -  对象名.属性名称
   -  对象名.函数名称
   
   类里面访问
   
   -  self.属性名称
   -  self.函数名称
   
##### 二、魔术函数
在Python中，__xx__()的函数叫做魔法函数，指的是具有特殊功能 或者 有特殊含义的函数。而且这些函数都是在某种情况下自动调用的。

1、init函数
``` bash

1、__init__():对象的初始化函数
2、__init__()，在创建一个对象时默认被调用，不需要手动调用
3、__init__(se1f)中的se1f参数，不需要开发者传递，python解释器会自动把当前的对象引用传递过去
```