[TOC]

# python

## 核心

### 内存管理

[python内存管理参考博客](https://wxnacy.com/2019/06/16/python-memory-management/)

变量定义：不需要对变量进行显示定义，第一次赋值时自动声明，变量只有被创建或者赋值后才能被使用

动态类型：变量没有类型，对象才有类型，变量引用了什么类型的对象，该变量就是什么类型，在程序运行时动态确定的

对象引用：内置函数id()可以返回对象的内存地址,变量实现对对象的引用。

```python
k = 1
print(id(k))
# out:4497728928

a = [1,2,3,4]
b = a      			# 对象[1, 2, 3 4]			a只是对其的引用。
a[0] = 4				# 修改对象[1, 2, 3, 4]
print(b)
# out:[4, 2, 3, 4]
```

引用计数：每个对象都包含一个头部信息，内容为类型标识符和引用计数器，对象被创建时，计数器加一，每当对象出现新的引用时，计数自动加一。`sys.getrefcount()`方法可以获取对象的引用计数，当对象传入该函数时，引用计数也会加一。

引用计数增加：

1. 对象创建
2. 被其他变量引用
3. 成为容器对象的一个元素

引用计数减少：

1. 对象的引用变量引用了另一个对象
2. 引用变量被显示的销毁，`del k`
3. 对象被一个容器对象移除
4. 容器对象本身被销毁
5. 离开其作用域

`当对象的最后一个引用被移除时，引用计数减少为0，该对象无法被访问，会成为垃圾回收机制的回收对象，但是任何对该对象的追踪或者调试都会增加对他的引用，这将推迟该对象被回收的时间`

#### 缓存池

```python
print(sys.getrefcount(1))
# out:135
```

Python 内部存在一个缓冲池，池内对象在内存中只有一份，所有符合缓存规则的对象，如果不存在则创建后进入缓存池，再次调用时只是增加引用计数；如果已经存在，则所有引用都只是增加引用计数，不会增加新的内存地址，好处是在一定程度上减少了内存的消耗。

缓存池包含:`布尔值(True, False),小整数(范围[-5, 256],可用is判断),守规矩字符串`

### 垃圾回收

python解释器承担内存分配回收的复杂任务， 当对象的引用计数为0的时候，成为一个待回收的垃圾内存。垃圾回收时python无法进行其他的任务，会降低工作效率，所以垃圾内存少的时候没必要频繁的执行垃圾回收。

`垃圾回收启动时机：python运行时，会记录其中的分配对象和取消分配对象的次数，当两者差值高于某个阈值，垃圾回收启动`

查看阈值：

```python
import gc

print(gc.get_threshold())
# out: (700, 10, 10)				# 700:垃圾回收启动的阈值, 10：每十次0代垃圾回收，会配合一次一代垃圾回收
														# 10:每10次1代垃圾回收，会配合1次2代垃圾回收
  
# 手动启动垃圾回收
print(gc.collect())
# out:0											# 

```

分代回收：所有的对象分为0， 1，2代，新创建的对象为0代，每经过一次垃圾回收，依然存活的对象，会归入下一代对象，，，该策略基于的假设：程序运行过程中，存活时间越久的对象，越不容易在后边变成垃圾，对于经历了几次垃圾回收依然存活的对象，出于信任和效率，垃圾回收器会减少对其的扫描频率。

循环引用：形成引用环

### 闭包

[闭包参考博客](https://www.cnblogs.com/Lin-Yi/p/7305364.html)

又称词法闭包或者函数闭包，是引用了自由变量的函数，这个被引用的自由变量和函数一同存在，即使已经离开了创造它的环境也不例外。。。。(听听，这说的是人话吗?)

其实就是内部函数使用了外部函数的变量，并且外函数返回值内函数的引用，就形成了闭包。

一般情况下，如果一个函数结束，函数内部的所有内存都会释放掉，但是闭包如果外函数在结束的时候发现有自己的临时变量将会在内部函数中用到，就会把临时变量绑定给内部函数，然后再自己结束。

应用：装饰器， 单例模式， 偏函数

![image-20190713121442572](https://oajua4pqj.qnssl.com/image-20190713121442572.png)



![image-20190713121423974](https://oajua4pqj.qnssl.com/image-20190713121423974.png)

### 装饰器

装饰器是将一个函数镶嵌在另一个函数中进行重复使用的目的， 增加函数的使用方式，但是减少了冗余代码，类似于java 当中的注解，同样能够实现面向切面编程

调用方式:

  1. 装饰器(需要装饰的方法)()		例如：`fun = log(test)   fun()`
  2. @装饰器名		`@log`

带参数的装饰器：

```python
def log_with_param(text):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            print('call %s():' % func.__name__)
            print('args = {}'.format(args))
            print('log_param = {}'.format(text))
            return func(*args, **kwargs)

        return wrapper

    return decorator

@log_with_param("param")
def test_with_param(p):
    print(test_with_param.__name__)
    
# 携带了参数的装饰器将有三层参数，不带参数的是两层
# test_with_param(1)

# 传入装饰器的参数，并接收返回的decorator函数，一层一层实例化并调用
decorator = log_with_param("param")
# 传入test_with_param函数
wrapper = decorator(test_with_param)
# 调用装饰器函数
wrapper("I'm a param")

# out:
# call test_with_param():
# args = (1,)
# log_param = param
# test_with_param
```

### WRAPS包裹

类似于common_api的实现方法，将目标函数传入了装饰器decorator，装饰器将内部函数的引用返回给了目标函数名，当再次调用目标函数的时候实际上调用了装饰器返回的内部函数的实力对象

```python
import functools
def decorator(func):
  
  	@functools.wraps(func)  # 再次查看属性的时候，调用的是dest，wraps帮助我们把目标函数的属性迁移了
    def inner( *args ,**kwargs ):
        print("装饰器的前置业务逻辑")
        res = func( *args ,**kwargs ) #在这里执行目标函数
        print("装饰器的后置业务逻辑")
        return res
    return inner

@decorator  #这里相当于发生了 dest = decorator( dest )
            # 把目标函数dest传进去给了func，返回来的inner给了dest
def dest(): #之后再执行dest 其实执行了inner
    print("这里是目标函数dest")
    
# 此时调用dest其实调用的是inner，因为内函数返回给了dest保存
```



### 函数

1. 可变参数 args,
2. 关键字参数kwargs
3. 命名关键字参数， `person(name, age, *, city, job)`,命名关键字参数需要特殊分隔符*，*后面的参数被视为命名关键字参数。

#### 高阶函数

map()			将传入的函数依次作用到序列的每个元素，并把结果作为新的`Iterator`返回

Sorted()		可以设置key函数来自定义排序，也可以反转排序结果，使用的算法是Timsort

##### Timsort算法

[知乎算法参考](https://www.zhihu.com/question/36280272)

[国内参考博客](https://blog.csdn.net/yangzhongblog/article/details/8184707)

该算法是结合了合并排序和插入排序而得出的排序算法有很好的效率 ，该算法找到数据中已经排好序的快，进行分区，每一个分区叫一个 run，每个run最少两个元素，然后按规则合并这些run，python和java都采用该排序算法。

核心过程：

为了减少对升序部分的回溯和对降序部分的性能倒退，将输入按其升序和降序特点进行了分区。排序的输入单位不是一个单独数字，而是一个个的块的分区，其中每一个分区叫一个run，针对这些run序列，每次拿出一个run 出来按照规则进行合并，每次合并 会将两个run合并成一个run，合并的结果保存到栈中，直到消耗所有的run，这时将栈剩余的run合并到只剩一个run为止，这时，这个run便是排好序的结果。

1. 如果数组长度小于某个值，直接用二分插入算法排序
2. 找到各个run，入栈
3. 按规则合并run

针对每一个run进行归并排序，算法按照升序和降序划分出各个run，如果是升序的，即后一元素要大于等于前一元素；如果run是严格降序的，即run中的前一元素大于后一元素，需要将run中的元素进行反转(必须是严格降序，这么做是为了保证稳定性)

run是一个已经排好序的分区，算法根据run的长度来选择排序的策略，run的长度小于某一个值的时候会选择插入排序算法来排序，run 的最小长度取决于数组的长度，当数组元素小于64时，run的最小长度便是数组的长度，此时使用插入排序算法进行排序，当数组元素大于等于63时，对于较大的数组，从32到65之间选择一个数字作为minrun，使数组的大小等于或者略小于2的幂，最后一个算法数组大小中最重要的6位，如果设置了其余的任何一位，则添加一位，并使用该结果作为minrun，该算法适用于所有情况。

优化run的长度：

优化run的长度是指当run的长度小于minrun时，为了使这样的run 的长度达到minrun的长度，会从数组中选择合适的元素插入run 中，是大部分的run 的长度达到均衡，有助于后面的合并操作

合并run：

划分run和优化run长度以后，就是对run的合并，合并原则是：run 合并的技术要保证有最高的效率，当算法找到一个run时，会将该run在数组中的起始位置和run的长度放入栈中，然后根据先前放入栈中的run决定是否合并该run，算法不会合并在栈中不连续的run，因为会造成什么鬼。。。

算法会合并2个在栈中连续的run，X，Y，Z代表栈最上方的3个run 的长度，当同时不满足下面2个条件时，X，Y这两个run会被合并，知道同时满足下面2个条件，则合并结束：

1. X > Y + Z
2. Y > Z

### -迭代器

#### 可迭代对象

但凡可以返回一个迭代器的对象都可以称之为迭代对象，迭代器就是实现了工厂模式的对象，在每次询问要下一个值的时候返回。比如`itertools`模块的函数返回的都是迭代器对象

```python
x = [1, 2, 3, 4]
y = iter(x)
print(y)
# out: <list_iterator object at 0x10be510f0>
```

#### 生成器

一种特殊的迭代器，更加优雅， `yield`关键字， 生成器一定是迭代器，任何一种生成器也是一种懒加载的模式生成值。

```python
from itertools import islice

# 生成器方式生成斐波那契额数列
def fib():
    prev, curr = 0, 1
    while True:
        yield curr
        prev, curr = curr, curr + prev


f = fib()
print(list(islice(f, 0, 10)))
```



#### 生成器表达式

列表推导式的生成器版本，返回的是一个生成器对象，而不是列表对象。`(x*x for x in range(10))`

### 魔术方法

[详细说明](https://www.cnblogs.com/pyxiaomangshe/p/7927540.html)

在python中，使用`__`双下划线包起来的方法称为魔术方法，可以用来控制类的属性访问。

#### 例

| 方法               | 作用                                                         | 说明         |
| ------------------ | ------------------------------------------------------------ | ------------ |
| `__new__()`        | 类当中，实例化对象，在`__init__`之前执行                     |              |
| `__init__()`       | 类当中初始化对象                                             |              |
| `__name__`         | 标识模块名字的系统变量：主模块为`__main__`,如果是import的模块，则为文件名 |              |
| `__file__`         | 当前文件名，pycharm里面会加上路径                            |              |
| `__str__()`        | 对象的输出                                                   | 返回str      |
| `__getattr__()`    | 定义当用户获取一个不存在的属性的行为                         |              |
| `__setattr__()`    | 无论属性是否存在允许定义属性的行为                           | 避免无限递归 |
| `__getattribute__` | 属性被访问时的行为                                           |              |
| `__call__()`       | 允许实例像函数一样被调用，实例化以后自动调用                 |              |
| `__main__`         | 程序的入口函数的名称                                         |              |
| `__unicode__`      | python3中和`__str__()`区别被废除。                           | 返回unicode  |
| ``                 |                                                              |              |
| ``                 |                                                              |              |



## 容器实现及其复杂度

### 列表List

#### 实现

列表采用的是数据结构中的顺序表，是一种采用分离技术实现的动态顺序表，列表的实现可以是数组和链表

列表实现是基于数组或者链表结构的，当使用列表迭代器的时候，双链表的结构比单链表结构更快

实现细节：列表容易与其它语言的链表混淆，事实上CPython的列表被实现为长度可变的数组。python 的列表是由对其他对象的引用组成的连续数组，指向该数组的指针及其长度被保存在一个列表头结构中，每当操作(增，删)一个元素时，由引用组成的数组需要重新分配，python在创建数组时，采用了指数分配，所以不是每次操作都需要改变数组的大小，因此。添加或取出元素的时候平均复杂度较低

#### 内存

[元组和列表的内存分配机制](https://www.jianshu.com/p/24090fb63968)

创建N个元素的List时：python动态内存分配长N+1个元素的内存，第一个元素存储列表长度和列表的元信息

APPEND一个元素时：python创建一个足够大的列表，来容纳N个元素和将要被追加的元素，重新创建的列表长度大于N+1，实际上为了之后的append操作，M个元素长度，即未被使用到的列表内存会被额外分配，然后旧列表的数据被copy到新列表中， 旧列表销毁。额外分配内存同时也减少了内存分配和copy的次数。

额外内存的分配，只会发生在第一次append 操作时，创建普通列表时，不会额外分配内存。

当一次Append操作发生时，新列表要分配的内存大小：`M = (N >> 3) + (N <9 ? 3 : 6) + 1 `



#### 复杂度

| 方法                 | 作用             | 复杂度      | 说明                                                   |
| :------------------- | ---------------- | ----------- | ------------------------------------------------------ |
| list[i]              | 获取第i 个元素   | O(1)        |                                                        |
| List[i:j]            | 截取             | O(k)k=j - i |                                                        |
| list[i:k] = list()   |                  | O(n + k)    |                                                        |
| List.index(item)     | 某个元素的索引   | O(n)        |                                                        |
| List.append(item)    | 添加元素         | O(1)        |                                                        |
| List.insert(i, item) | 插入item到i位置  | O(n)        |                                                        |
| List.pop()           | 删除最后一个元素 | O(1)        | 返回删除的值                                           |
| list.pop(i)          | 删除第i 个元素   | O(n)        | 如果推出第一个元素，那么数组的所有元素都要重新计算坐标 |
| Del list[i]          | 同上             | O(n)        |                                                        |
| Item in list         | In判断           | O(n)        |                                                        |
| List.reverse()       | 列表反转         | O(n)        |                                                        |
| List.sort()          | 列表排序         | O(n logn)   |                                                        |
| list.remove(item)    | 删除元素item     |             | 返回None，若无该元素。。则报错                         |
|                      |                  |             |                                                        |

### 元组Tuple

内存利用：不支持改变大小(resize),但是可以粘贴两个元组组成一个新的元组，该操作类似于List的append的操作，但是不会额外分配内存，但是我们不能把它当成append，因为append会进行内存分配和内存的copy操作。

tuple的静态本质带来的好处是资源缓存，python的GC是当一个变量不用了，内存会被回收交给OS，但是对于元组，当它不再被用时，内存不会立即返回给OS，而是为了以后应用而暂缓保留，当创建一个20个元素的tuple时(源码宏定义`PyTuple_MAXSAVESIZE=20`)，不需要向OS重新申请分配内存，而是用现有的保存的游离存储。

Tuple的创建很简单，并且比 避免频繁的与OS申请内存，创建一个具有10个元素的Tuple比创建一个List要快不少。

元组与列表相似，但区别在于列表是动态的，大小可以重新分配，元组是不可变的，一旦创建就不能修改,但元组中如果有引用其他对象，修改该对象可引起元组变化。

```python
a = [0, 9, 8]
li = (1, 2, 3, 4, 5, a)
print(li)
# out: (1, 2, 3, 4, 5, [0, 9, 8])
a.pop()
print(li)
# out: (1, 2, 3, 4, 5, [0, 9])
```

| 方法                 | 作用                                         | 说明 |
| -------------------- | -------------------------------------------- | ---- |
| count(item)          | 获取该元素出现的次数                         |      |
| index(x, start, end) | 获取指定元素的下标， start/end：查询起止下标 |      |
| li[i]                | 使用下标获取元素                             |      |



### 字典

[python字典原理](http://foofish.net/python_dict_implements.html)

#### 哈希表

`python内部实现PyDictObject对象`

也叫三列表，根据关键值对而直接进行访问的数据结构，通过把key和value映射到表中一个位置类访问记录，查询速度非常快，更新也快，映射的函数叫做哈希函数，存放值的数组叫做哈希表，哈希函数的实现方式决定了哈希表的搜索效率，具体操作过程：

1. 数据添加：把key通过哈希函数转换成一个整形数字，然后将该数字对数组长度进行取余，取余结果就当作数组的下标，将value存储在以该数字为下标的数组空间里
2. 数组查询：在此使用哈希函数将key转换为对应的数组下标，并定位到数组的位置获取value

对key进行哈希的过程中，不同的key可能出现一样的结果，尤其是数据量增多的时候，这时候会出现`哈希冲突`，解决方法：

1. 连接法			将发生冲突的元素放到同一位置，然后通过指针来串联。
2. 开放寻址法       (python采用该方法)，即所有的元素存放在散列表里，当产生哈希冲突时，通过探测函数计算出下一个候选位置，如果下一个获选位置还有冲突，那么久不断通过探测函数往下找，直到找到一个空槽来存放待插入元素。

#### 实现

字典中的一个key-value键值对元素称为entry(也叫做slots),，对应到python内部时`PyDictEntry`，`PyDictObject`是一个entry的集合，源码当中对其的定义如下：

```c
typedef struct {
    /* Cached hash code of me_key.  Note that hash codes are C longs.
     * We have to use Py_ssize_t instead because dict_popitem() abuses
     * me_hash to hold a search finger.
     */
    Py_ssize_t me_hash;					// 缓存me_key的哈希值，防止每次查询时都要计算，
    PyObject *me_key;						// 键
    PyObject *me_value;					// 值
} PyDictEntry;
/* 该对象有三种状态：
*Unused：me_key==me_value==NULL																初始状态，插入元素后变为Active
*Active：me_key!=NUll and me_key!=dummy and me_value!=NULL			插入元素且元素不为NULL
*Dummy: me_key==dummy and me_value==NULL											 删除元素时，Active转换为Dummy，此时实际上是一个PyStringObject对象，仅作为标志，标志该位置上的元素已经删除，如果插入元素-->Active，不可能再变成Unused
*/
```

PyDictObject就是PyDictEntry对象的集合，其源码结构如下：

```c
typedef struct _dictobject PyDictObject;
struct _dictobject {
    PyObject_HEAD
    Py_ssize_t ma_fill;  /* # Active + # Dummy  处于Active 和Dummy状态的元素的个数*/
    Py_ssize_t ma_used;  /* # Active 所有处于Active状态的元素的个数*/

    /* The table contains ma_mask + 1 slots, and that's a power of 2.
     * We store the mask instead of the size because the mask is more
     * frequently needed.
     */
    Py_ssize_t ma_mask;					// 所有entry元素的个数

    /* ma_table points to ma_smalltable for small tables, else to
     * additional malloc'ed memory.  ma_table is never NULL!  This rule
     * saves repeated runtime null-tests in the workhorse getitem and
     * setitem calls.
     * 当entry数量小于PyDict_MINSIZE，ma_table指向ma_smalltable的首地址，当entry数量大于8时，python        			* 将其作为一个大字典处理，此刻会申请额外的内存空间，同时将ma_table指向这块空间
     */
    PyDictEntry *ma_table;
  
  	//  字典元素的搜索策略
    PyDictEntry *(*ma_lookup)(PyDictObject *mp, PyObject *key, long hash);
    PyDictEntry ma_smalltable[PyDict_MINSIZE];		//创建字典时创建一个大小为PyDict_MINSIZE的PyDictEntry数组
};
```

字典创建过程：

1. 初始化dummy对象
2. 如果缓冲池还有可用的对象，则从缓冲池中读取，否则，执行步骤3
3. 分配内存空间，创建PyDictObject对象，初始化对象
4. 指定添加字典元素时的探测函数，元素的搜索策略

#### 复杂度

| 方法                   | 作用 | 复杂度 |
| ---------------------- | ---- | ------ |
| copy()                 | 复制 | O(n)   |
| get(key)               | 读取 | O(1)   |
| Set(key, value)        | 添加 | O(1)   |
| del dict[key]/pop(key) | 删除 | O(1)   |
|                        |      |        |
|                        |      |        |



#### 疑问

为什么entry会有Dummy状态？

采用开放寻址法时，遇到哈希冲突，会一直找到一个合适的位置，在寻找过程中形成探测链，当在查找元素的时候也是根据探测链进行寻找，当删除探测链上的某元素变为Unused状态，那么会出现探测链断裂的现象，最终导致值丢失，所以需要Dummy状态作为一种伪删除的方式，保证探测链的连续性。

### 集合Set

#### 实现

集合的实现类似于字典，不同之处在于哈希函数操作的对象，字典操作的是key， 而set是直接操作它的元素，set的元素经过哈希函数之后获得下标，该下标位置用来存放元素本身，字典创建了两个list，一个用于存放key。。一个用于存放value，将这种实现方式称之为Hash Set，实现dict的方式称之为HashMap。

## GIL锁

`http://zhuoqiang.me/python-thread-gil-and-ctypes.html`

`https://www.jianshu.com/p/573aaa001b35`

全程Global Interpreter Lock，全局解释器锁

多个python进程有各自独立的GIL锁

python的执行由python的解释器来控制的，在任一时刻，只有一个线程在解释器中运行，对python解释器访问的控制由全局解释器锁GIL控制，本意是为了保护python的全局解释器和环境状态变量的，如果去掉GIL就需要更细粒度的锁对众多全局状态进行保护，或者采用Lock-Free算法，无论是哪一种，要做到多线程安全都会比维系一个GIL要难的多，另外改动的还是Cpython的代码树以及各种第三方扩展也在依赖GIL。

python中使用的线程都是操作系统级别的，linux中使用的是pthread，windows使用的是其原生线程

劣势：python同一时刻只能跑一个线程，这样跑多线程的情况下，只有当线程获取到全局解释器锁后才能运行，而GIL只有一个，因此即使是在多核的情况下，也只能发挥出单核的功能，直接导致python不能利用物理多核的性能加速运行，只要原因是python在创造的时候，多核心CPU还是不可想象的

优势：简化了Cpython的实现，使得对象模型，包括关键的内建类型如字典，都隐式的可以并发访问，锁住全局解释器使得其比较容易实现对多线程的支持。但折损了多处理器主机的并行计算能力

对于标准模块和第三方模块，被设计成在进行密集计算任务和IO操作时，GIL总是被释放的，对所有面对内建的操作系统C代码程序来说，GIL会在这个调用之前被释放，一允许其他线程在等待这个IO的时候运行，如果是纯粹的计算型，没有IO操作，解释器会每隔100次或者15ms去释放GIL，IO密集型比计算密集型的程序更能利用多线程环境带来的便利

python解释器多线程执行顺序：

1. 设置GIL
2. 切换到一个线程去执行
3. 运行代码
   - 指定数量的字节码指令(100个)
   - 固定时间15ms后线程主动让出控制权
4. 把线程设置为睡眠状态
5. 解锁GIL，重复以上步骤

有人试过去掉GIL，但是实践检测对单线程来说性能更低，只有利用的无力CPU到达一定数目后，性能才会比GIL版本好。现在绝大部分的python程序都是单线程的

- python引入多进程标准库，让多进程的python编写简化到类似多线程的程度，大大减轻GIL带来的诸多不利
- 利用ctypes绕过GIL：ctypes可以使python直接调用任意的C动态导出函数，所要做的只是用ctypes写python代码即可，ctypes会在调用C函数前释放GIL。

## 进程

同一时刻每一个CPU只能执行一个进程，如果存在大于核心数的进程，就会产生进程之间的切换，进程之间的切换需要保存当前进程的寄存器状态和内存状态，将另一个进程之前保存的数据恢复，进程之间的切换也是一个耗时的过程。

### fork

Unix或者linux系统提供了一个fork调用，该函数与普通函数不同的是，调用一次会返回两次，操作系统会自动把当前进程(父进程)复制一份(成为子进程)，然后分别在父进程和子进程里返回，子进程永远返回0，父进程返回子进程的id。一个父进程可以fork出很多子进程，所以父进程要记下每个子进程的id，子进程只需要调用getppid()就可以拿到父进程的id，python os模块封装了常见的系统调用

常见的apache服务器就是由父进程监听端口，每当有新的http 请求的时候，就fork出子进程处理新的http请求。

```python
import os
import time

pid = os.fork()
if pid == 0:
    time.sleep(3)
    print(".....")
else:
    print("here is fu")
time.sleep(5)     # 延迟父进程的结束
```



### multiprocessing

```python
import multiprocessing
import os


def run():
    print("current pid:" + str(os.getpid()) + " ")


if __name__ == '__main__':
    pro = multiprocessing.Process(target=run, name="run")
    pro.start()
    run()
    pro.join()				# 等待子进程结束再继续向下，用于子进程间的同步和父进程的守护
```

### 进程池Pool

```python
import multiprocessing
import os
import time
import random


def run(name):
    time.sleep(2)
    print("current pid:" + str(os.getpid()) + " " + str(name))


if __name__ == '__main__':
    pool = multiprocessing.Pool(3)
    for i in range(4):
        pool.apply_async(run, args=(i, ))
    pool.apply(run, args=("three", ))
    pool.close()
    run("main")
    pool.join()					# 等待所有子进程执行完毕
```

### subprocess

可以非常方便的启动子进程，然后控制其输入输出

### 进程之间的通信

进程之间需要通信，操作系统提供了很多机制来实现进程间的通信，python的multiprocessing模块包装了底层的机制，提供了Queue， Pipes等多种方式来交换数据。



```python
from multiprocessing import Process, Queue
import time


def write(queue):
    li = [1, 2, 3, 4, 5, 6, 7, 8, 9, "END"]
    for i in li:
        queue.put(i)
        time.sleep(1)


def read(queue):
    while True:
        value = queue.get(True)
        print(value)
        if value == "END":
            break


if __name__ == '__main__':
    queue = Queue()
    read_p = Process(target=read, args=(queue, ))
    write_p = Process(target=write, args=(queue, ))

    read_p.start()
    write_p.start()
    read_p.join()
```



## 线程

受GIL锁影响，每个python 进程中只能有一个线程，无法发挥多核CPU的优势

线程与进程不同的地方在于，多进程中，同一个变量，各自有一份拷贝存在于每一个进程中，互不影响，而多线程中，所有变量都由所有线程共享，所以任何一个变量，任何一个线程都可以对其进行修改。 因此线程之间共享数据最大的危险在于多个线程同时修改一个变量，所以在修改同一变量时，需要加锁

```python
import threading
import time


def run():
    time.sleep(3)
    print("current thrad name:" + threading.current_thread().name)


if __name__ == '__main__':
    th = threading.Thread(target=run, name="one")
    th.start()
    run()
    th.join()
```



### Lock

```python
import threading

lock = threading.Lock()

def plus(stand):
    global res
    lock.acquire()  #  加锁
    try:
        print("current thread name:" + threading.current_thread().name)
        res += stand
    finally:
        lock.release()		# 释放锁


def sub(stand):
    global res
    lock.acquire()
    try:
        print("current thread name:" + threading.current_thread().name)
        res -= stand
    finally:
        lock.release()


if __name__ == '__main__':
    res = 0
    th1 = threading.Thread(target=plus, name="one", args=(3, ))
    th2 = threading.Thread(target=sub, name="two", args=(5, ))
    th1.start()
    th2.start()
    th1.join()
    th2.join()
    print(res)

```



## 协程

微线程，效率高，看上去是子程序，但是在子程序内部可中断，然后转而执行别的子程序，适当的时候再回来执行，这不是函数调用，有点类似于CPU中断

执行效率高，子程序切换不是线程切换，由程序自身控制(线程切换由操作系统来决定)，因此没有线程切换的开销，和多线程比，多线程数量越多，协程性能优势越明显。

不需要多线程的锁机制，因为只有一个线程，不存在同时写变量的冲突，控制共享资源的时候只需要判断就可以

### yield

[yield及上下文管理器](https://www.jianshu.com/p/bf887cae4d8e)

python 的协程基于yield关键字，yield本身用作生成器，当对产生的值进行消费的时候，就是协程了。

```python
def grep(pattern):
    print("Looking for {}".format(pattern))
    while True:
        line = yield
        if pattern in line:
            print('{} : grep success '.format(line))


g = grep("python")      # 生成器,

next(g)                 # 激活协程，也可以使用g.send(None)
g.send("python")        # 发送数据，相互协作，进行消费
g.close()								# 关闭协程
```

### yield from

替代内层for 循环：如果生成器函数需要产出另一个生成器生成的值，传统的解决方式是使用嵌套的for循环

`yield from x`表达式对对象x做的第一件事就是调用iter(),将其变成一个可迭代对象，从中获取迭代器，因此x可以是任何可迭代对象

### CPU中断和函数(子程序)调用的区别

函数调用：主程序调用子程序，子程序是一组可以共用的指令序列，只需要给出入口地址就可以转入子程序，在功能上具有独立性，一般微机首先执行主程序，碰到调用指令就转去子程序执行，结束后返回主程序断点，优点是：子程序结构可简化程序，防止重复书写错误，节省内存空间

中断：中断是计算机CPU与外设I/O交换数据的一种方式，有多种方式，其中 中断效率高，被大量采用；

中断即计算机在执行某一主程序时，收到中断请求，如果中断响应条件成立，计算机就把正在执行的程序暂停，去处理这一请求，执行中断服务程序，完成后返回原中断点。

联系：两者都需要保护断点，即下一条指令地址，保护现场，恢复现场，恢复断点，可实现嵌套

区别：两者的服务时间和服务对象不一样，函数调用是已知和固定的，中断过程发生的时间是随机的，CPU在执行某一主程序时，收到中断源提出的中断申请，就发生中断过程，中断申请一般由硬件电路发生，申请时间随机，可以说是由系统工作环境随机决定的。。

## 套接字socket









# 协议

## TCP/IP

`自己的csdn博客有`

`https://blog.csdn.net/weixin_40389301/article/details/80310483`

## HTTP

介绍：超文本传输协议，应用层上的一种c/s模型的通信协议，由请求和响应构成，且无状态的，默认端口号80，Https默认端口号443

特点：

1. 协议：规定了通信双方必须遵守的数据传输格式，双方按照约定的格式才能准确的通信
2. 无状态：指两次连接通信之间是没有任何关系的。每次都是一个新的连接，服务端无法记录前后的请求信息
3. 客户端/服务端模型
4. 处于五层网络模型当中的应用层。通常承载于TLS或SSl协议层之上，这时候就成了常说的Https

工作流程：

1. 客户端与服务器建立http连接
2. 客户端发送请求给服务端，请求格式为：统一资源标识符。协议版本号等信息
3. 服务器

# 算法

## 排序

### 交换排序

好处是不需要开辟新的空间

#### 冒泡排序

1. 对所有记录从左到右每相邻的两个记录的排序码进行比较，如果不符合排序要求，则进行交换，一趟将排序码最大者放在最后一个位置，即该排序码对应的记录最终应该放的位置
2. 最剩下的n-1个待排序的记录重复步骤1到位置n-1即可

如果某一趟没有发生任何交换，则说明此时所有的记录已经按照要求排序完毕。复杂度：O(n2)

#### 快速排序

又叫分布交换排序

基本思路：

从n个待排序的记录当中任意取一个记录(不妨取第一个记录)，设法将该记录放置于排序后他最终应该放的位置，使它前面的元素都小于它，后面的元素大于它。

然后对前后两部分待排序记录重复上述过程，可以将所有记录放置于排序成功后的位置，排序即完成。

算法过程：

1. 将排序码(第一个记录)暂存，这时第一个位置空了出来此时排序码位置为i
2. 从后向前找到一个比排序码小的记录，将该记录(位置为j)迁移至第一个位置，迁移后位置为i,即a[i]=a[j]
3. 从前向后找一个比排序码大的记录，该记录位置为i， 将a[j]=a[i]
4. 将排序码赋值给a[i]，完成一次排序，
5. 对前后两部分数据递归实用上述操作，排序便可以完成

### 插入排序

基本思路：

将待排序文件中的记录，逐个的按其排序码值的大小插入到目前已经排好序的若干个记录组成的文件中的适当位置，并保持新文件有序，有直接插入排序算法， 二分法插入排序，表插入排序，shell插入排序算法

#### 直接插入排序

基本思路：

基于有序的记录，初始认为第一个记录已经排好序，然后将第二个到第n个记录，插入到已经排好序的记录当中

#### 二分法插入排序

当插入第n个元素时，认为前n-1个元素已经排好序，则在查找元素的插入位置的时候采用二分查找法，查找待插入元素的插入位置

### 选择排序

其中包含直接选择排序，堆排序，

基本思路：选取排序码(一般选择第一个), 遍历剩余的记录，找到最小的与排序码进行比较，并交换

堆排序思路：使用记录构建一个堆，然后从n/2长度的位置开始进行调整，按照从n/2到1的顺序，将左右儿子节点中的较小的值交换到该子树的根节点，堆当中的根结点则为最小的排序码。将该排序码从堆中删除，重新建堆，重复操作。直至剩余一个元素

重新建堆不需要从头再来，只需要将当前堆中的最后一个元素与根结点交换位置，同时让堆中元素个数减1，因为此时根结点的左右子树都还满足堆的条件

### 归并排序



### 算法对比

| 排序法       | 最差时间分析 | 平均时间复杂度 | 稳定度 | 空间复杂度     |
| ------------ | ------------ | -------------- | ------ | -------------- |
| 冒泡排序     | O(n2)        | O(n2)          | 稳定   | O(1)           |
| 快速排序     | O(n2)        | O(n*log2n)     | 不稳定 | O(log2n)～O(n) |
| 选择排序     | O(n2)        | O(n2)          | 不稳定 | O(1)           |
| 插入排序     | O(n2)        | O(n2)          | 稳定   | O(1)           |
| 二分插入排序 | O(n2)        | O(n2)          | 稳定   | O(1)           |
| 堆排序       | O(n*log2n)   | O(n*log2n)     | 不稳定 | O(1)           |

# 数据结构

## 树

