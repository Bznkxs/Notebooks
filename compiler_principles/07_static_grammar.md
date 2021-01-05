# 静态语义分析

## 静态语义分析

静态语义：静态一致性/完整性的特征

动态语义：程序执行时的行为

### 类型检查

类型表达式：基本类型、类型名字、类型变量、类型构造子归纳定义得到

bool; array(1..10, int); pointer(int); proc(int); type_error(有类型错误的程序单元); ok(无类型错误的程序单元)

类型系统：将类型表达式赋给程序组成成分的规则的集合

$\vdash$

(跳过)

### 语法制导的类型检查

这里主要描述一些常用的属性

- S.type := 类型表达式

  语句/过程等程序单元的type是type_error或ok

  表达式/变量等程序单元的type是int等

- id.name := 标识符名字

- id.entry：符号表中表示这个标识符的表项(的指针)，比如第3项是id，那么entry就是table[3]

- lookup_type(id.name)：从符号表查找名字对应的type域，若无定义返回nil

- S.break: 继承属性，当前作用域内是否可以使用break语句

## 语法制导的中间代码生成

中间表示与机器无关

多级中间表示：由源程序到AST再到TAC，最后变成目标代码

- AST或DAG（DAG是AST的简化）
- TAC, 三地址码/四元式

### AST

高级中间表示

利用翻译模式，一些方法和属性：

- S.ptr：S对应的结点
- <u>int</u>.val：int要带下划线，表示一个整型字面值如1234，.val是取出这个值
- mkleaf(val)：创造叶子节点，结点属性只有一个值，要么是常数值，要么是一个符号表表项id.entry
- mknode(op, *children)：创造子节点，结点属性是此处的运算/操作，还要告知其孩子列表
- V.v-list：V是程序的“声明部分”，v-list代表变量列表
- F.f-list：F是“过程定义部分”，f-list代表内部过程列表
- A.a-list: A是程序调用部分，a-list是参数列表
- link_list(list1, list2)或merge_list：连接两个list
- make_empty_list()
- make_list(x): 产生[x]
- insert_list(list, x)：在list插入x

### TAC

顺序执行的语句序列，低级中间表示

(op y z x)表示x:=y op z

一些TAC语句：

- x:=y op z,
-  x:=op y, 
- x:=y, 
- goto L, 
- if x rop y goto L, 
- L:, 
- param x1, param x2, ..., call p, n
- return
- x:=y[i], x[i]:=y,
- x:=*y, *x:=y
- 

一些属性和语义函数：

- S.code，某个块的TAC序列
- ||，两个TAC序列连接
- gen(id.place':='A.place)，生成一个TAC语句（也是一个序列）
- A.place: A的值的存放位置
  - id.place: 相应的名字的值的存放位置，比如变量n是栈帧基址为4，偏移为0，这些信息就是id.place
  - A.place可能就会得到id.place
  - :=的含义：把右边的存放位置对应的值拿给左边
- newtemp：一个新的存放位置
  - 比如遇到一个常量3611，那么就开一个新位置存这个值：
    - A -> 3611 {A.place:=newtemp; A.code:=gen(A.place':='3611)}
    - 所以字面值是可以放在:=右边的

TAC里面没有类型、偏移地址等信息，这些是翻译过程中存在符号表里的

- num.lexval：就是字面值
- T.width：类型的字节数
- L.width：变量列表中所有变量的字节数
- L.num：变量个数
- enter(id.name, t, o)：将对应表项的type置为t，offset置为o

#### 控制语句的L-翻译模式

L翻译模式下，是采用code块合并的方式生成TAC序列的。

可以采用求值法：

- E.place：表示布尔语句的存储位置

还可以采用控制流法：

- newlabel：新的语句标号
- E.true, E.false：E为真和假时需要转移到的程序位置，是继承属性

根据if, while等语句，确定最上层的表达式的E.true和E.false，继承给下层的子表达式

#### 控制语句的S-翻译模式

S翻译模式下，没有code块，而是一条条在已有序列之后附加TAC；

利用拉链和代码回填的方法，使用综合属性来得到控制流。

简单地说，把子表达式的代码里面要跳转的部分留空，记录好TAC标号，并且记录这是父表达式为true还是为false该跳转到的地方

- E.truelist：真链，记录的是那些应该填“为真跳转”的TAC标号
- E.falselist：假链
- S.nextlist：next链，记录了要跨过程序体执行之后语句的那些TAC
  - 比如，如果if语句不执行，就要跳到if后面的语句。这个goto的TAC就会保存在nextlist里面
- S.breaklist：break链，记录了跳出循环语句的那些TAC
- M.gotostm：M是空结点，用来保存下一条TAC的标号
- nextstm：下一条TAC的标号
- emit(...)：输出一条TAC，使nextstm++
- backpatch(p, i)：p是一个链，将该链中的跳转语句的目标置为i

代码回填中，emit()往往在最底层关系表达式出现，比如：

E->true{ ... emit("goto _")}

##### 拉链的关系

记住：任何一个表达式都可能短路；任何一个语句块都可能跳到语句块结尾(next)，或者直接跳出去(break)，所以任何一个语句块都要维护next和break。

###### 真假链

在关系表达式中，只会出现truelist和falselist。

###### 放M（跳转目的地）的艺术

```
S -> if E then M S1 {backpatch(E.truelist, M.gotostm);
                     S.nextlist := merge(E.falselist, S1.nextlist)}
```

 当中，要放一个M来记录if体的标号，这样E为真就会跳到M。

凡是可能不会顺序访问到的地方，都需要放一个M。比如if里面，E可能会有短路，所以要放一个M。再比如S->while M1 E then M2 S1。

**放M一定意味着有backpatch，你是想让M前面的块跳转到M来。**

再如S->S1; S2，因为S1可能跳到末尾，所以需要在S2前面加一个M

###### next链

每个大一点的语句块都必须有next链（谁能保证你不会跳到后面去？）来处理那些跳过该语句块的行为。

上面的if中，S的next链包含了E的假链（失败了就要出去）和S1的next链。因为，S1的next链对应的语句也想跨出S1，而跨出了S1就跨出了S。没毛病。

- S的next来自if，或者来自自己最末一个块S1的next
  - 比如S -> S1; M S2，那么S.next=S2.next

###### 放N（空next）的艺术

有的时候需要放一个空的next语句块：

N->epsilon {N.nextlist:=makelist(nextstm); emit('goto _')}

这个语句块的作用是放在一个子块的末尾，强制跳转。因为，一个子块结束以后，如果是顺下来的，那么就会去执行这个子块后面的下一条语句。但是如if E then S1 else S2这种，S1结束以后不能去执行S2。所以S1结尾必须放一个N：

S -> if E then M1 S1 N else M2 S2

**你想让中间的块跳过后面的块，就要用N。**

###### break链

每个块都有break链（你不知道它的上层会不会有循环）。break链的元素在break语句处产生。简单地继承break链、在while块里面合并到next链，就可以了。

#### 过程调用

从左到右/从右到左计算每个参数

从左到右生成实参地址TAC

调用

比如：t:=a; u:=b; param t; param u; call p,2;

