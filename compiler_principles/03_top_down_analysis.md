# 3. Top-down parsing

cf. bottom-up parsing.

## Introduction

### Two causes of non-determinism

... in every derivation step...

- which nonterminal symbol to derive from
- which production rule to use

#### Determine nonterminal symbol

Use a left-most derivation.

### Deterministic top-down parsing

lookahead method: 
- scan once from left to right
- allowed to read ahead at most N tokens ahead while scanning, where N is a fixed value

There are CFGs that cannot be parsed by this method.

- left recursions
  > analyze `ba^n` using CFG G:
  > `S -> Sa | b`
  > must look ahead n+2 alphabets to determine a derivation sequence
  - there are also "indirect left recursions":
  > `S -> Aa`
  > `A -> Sb`
  
- can remove left recursion on certain occasions
  
- left common factor
  > `S -> aAb | aAc`
  > `A -> a | aA`
  - can remove left common factors on certain occasions
  
## LL(1) parser

> 重点：Fi和Fo的概念
>
> 

An LL (left-to-right, leftmost derivation) parser is called an LL(k) parser if it uses k tokens of lookahead.

A grammer is called an LL(k) grammar if an LL(k) parser can be constructed from it. ([LL parser - Wikipedia](https://en.wikipedia.org/wiki/LL_parser))

### First-set (*Fi*) and Follow-set (*Fo*)

#### First-set

for CFG G=(V<sub>T</sub>, V<sub>N</sub>, P, S), the first-set of any &alpha; &isin; (V<sub>T</sub> &cup; V<sub>N</sub>)<sup>\*</sup> is defined as follows:
- **Fi**(&alpha;) = \{ a | &alpha; =>\* a &beta;, a &isin; V<sub>T</sub>, &beta; &isin; (V<sub>T</sub> &cup; V<sub>N</sub>)<sup>\*</sup>, or &alpha; =>\* a where a=	&epsilon; \}

explanation: **Fi**(&alpha;) contains the first token of any possible derivation of &alpha;, or &epsilon; for &epsilon; production rule

> $ A \Rightarrow^* ast\beta: Fi(A)\ni a $
>
> $A\Rightarrow^* \epsilon: Fi(A) \ni \epsilon$
>
> 也就是说，&epsilon;只出现在空产生式

##### Calculation of *Fi*

> 迭代法计算（因为拓扑关系很复杂）
>
> 可以事先确定好非终结符之间的拓扑关系，然后每当某个符号的Fi被改，就改变依赖它的非终结符；不过最后还要靠迭代来检验

```python
def get_Fi(V_T, V_N, P, S):
# setup
    for x in (V_T + V_N + {epsilon} + {any suffix of any production rule}).closure():
        if x in V_T or x == epsilon:
            Fi[x] = {x}
        else:
            Fi[x] = {}

    while Fi_changed(): # do until Fi does not change anymore
        for p in P:
            (A, beta) = p  # p = A->beta
            if beta == epsilon:  
                Fi[A] = Fi[A]+{epsilon}
            for Ys in u.suffix():  # generates [Y1, Y2, ..., Yk] where Yj in V_n.join(V_T)
                
        
```


#### Follow-set

for CFG G=(V<sub>T</sub>, V<sub>N</sub>, P, S), the follow-set of any A &isin; V<sub>N</sub> is defined as follows:

- **Fo**(A) = \{a | S# =><sup>\*</sup> &alpha;A&beta;# and a &isin; Fi(&beta;#), where &alpha;, &beta; &isin; (V<sub>T</sub> &cup; V<sub>N</sub>)<sup>\*</sup>}, where # is the end-of-input token

> Fo就是当一个符号出现以后可能会跟的符号集合。上述定义可能有迷惑性，其实Fo是没有&epsilon;的（Fi(&beta;#)也不可能有&epsilon;）
>
> $Fo(S) \ni \#$，因为S# =>\* S#且# &isin; Fi(#)
>
> Fo有两种情况：
>
> - 如果G->&beta;Ax，那么x&in; Fo(A)，对吧；
>
> - 如果G->&beta;A，那么Fo(G)&subset;Fo(A)，因为G后面可能跟的也是A后面可能跟的
>
> 计算：迭代，初值是Fo(S)={#}，而且这个是不会变的

##### Calculation of *Fo*
```python
def get_Fo(V_T, V_N, P, S):
    for x in V_N:
        Fo[x] = {} 
    Fo[S] = {EOI}
    while Fo_changed():
        for p in P:
            (A, w) = p  # A -> w
            for B in V_N:
                if B in w:  # A -> xBy
                    x, B, y = w
                    Fo[B] += Fi[y] - {epsilon}
                    if epsilon in Fi(y):
                        Fo[B] += Fo[A]
```

### Predictive set

> 这就是LL(1)的具体操作了。一个产生式的预测集的意思是，如果我LL(1)读到下一个字符属于这个预测集，那么我就有可能用到这个产生式。
>
> 所以A->&alpha;有两种情况：
>
> - 如果Fi(&alpha;)不含有&epsilon;，那么Fi里面的每个字符都是预测集的元素
> - 如果含有，那么不仅Fi(&alpha;)这些字符（除了eps外）都在预测集里面，而且Fo(A)的字符也在里面，因为如果Fi是eps那么说明A可能转移到空，那么下一个字符可能就是A后面的那个了。

PS(A -> &alpha;) = 
- Fi(&alpha;) if &epsilon; &notin; Fi(&alpha;)
- (Fi(&alpha;) - \{&epsilon;\}) &cup; Fo(A)

### LL(1) grammar

> 所以什么情况下只预测一个就可以了呢？就是产生式的预测集不打架。我目前的栈上面是A，A可能有几个产生式，我要求它们的预测集不要冲突。

G is LL(1) *iff* PS(A -> &alpha;) &cap; PS(A -> &beta;) = &empty; for any A -> &alpha; | &beta;.

### Recursive descent LL(1) parser

Each nonterminal symbol corresponds to a subroutine which determines the production rule by reading the next input token, *a*.

When scanning through the production rules, scan their right-hand side:

- for terminal symbol *t*, judge if *a* == *t*:
  - if *a* == *t*: get the next token
  - else: error
- for nonterminal symbol *A*, call the corresponding subroutine.

> 比如 F -> int <u>id</u> (P)S的程序：
>
> ```python
> def ParseF():
>     MatchToken("int")
>     MatchToken("id")
>     MatchToken("(")
>     ParseP(P)
>     MatchToken(")")
>     ParseS(S)
> ```
>
> 如果有多个产生式，一般是这样：
>
> ```python
> def ParseA():
>     if lookahead in PS["A->u1"]:
>         pass
>     elif lookahead in PS["A->u2"]:
>         pass
> ```
>
> lookahead初始化为input[0]，并在MatchToken里面被消费和更新

### Table-driven LL(1) parser

> 预测分析表描述了PS，行是非终结符，列是终结符+#，中间内容是产生式。形如
>
> |      | a      | b            | #    |
> | ---- | ------ | ------------ | ---- |
> | S    | S->AaS | S->BbS       |      |
> | A    | A->a   |              |      |
> | B    |        | B->&epsilon; |      |
>
> #这列也可能被填上，全看PS的情况
>
> 维护一个下推栈，初始只包含#。将S入栈，然后开始根据表和当前输入符号进行操作。
>
> - 栈顶为终结符的，判断是否匹配，匹配后出栈并消费当前符号，否则失败
> - 栈顶为非终结符的，查表判断是否存在，并用表中产生式右边替代左边符号，记住要从右往左入栈
> - 栈顶和输入符号都为#时结束
>
> 格式：
>
> | 步骤 | 下推栈 | 余留符号串 | 下一步动作             |
> | ---- | ------ | ---------- | ---------------------- |
> | 1    | #S     | a          | 应用产生式S->a         |
> | 2    | #a     | a          | 匹配栈顶和当前输入符号 |
> | 3    | #      | #          | 返回：分析成功/失败    |
>
> 

## 消除左递归

> 直接左递归：P -> Pa | b
>
> 间接左递归：P -> Qa, Q -> Rb, R -> Pa
>
> 左递归构成一幅依赖图。只要不存在eps产生式，而且没有回路（P=><sup>+</sup>P），那么就可以消除左递归。

1. 直接左递归，我称其为“反正最后都要变，不如早点变，变完了就不再变，剩下的顺序无所谓”

   > P -> P&alpha;|&beta;
   >
   > -------------
   >
   > P -> &beta;Q   反正最后都要变&beta;，不如早点变
   >
   > Q -> &alpha;Q|&epsilon;    变完了就不再变，剩下&alpha;的顺序无所谓

   

   > P -> Pa | Pb | c | d
   >
   > ----------
   >
   > P -> cQ | dQ      反正最后都要变c或d，不如早点变
   >
   > Q -> aQ | bQ | &epsilon;         变完了就不再变，剩下的a, b的顺序无所谓

2. 间接左递归，无回路、无空产生式的情况

   首先将非终结符排序，然后双重循环：对j<i，反复用A<sub>i</sub> -> ar代替A<sub>i</sub>->A<sub>j</sub>r，其中A<sub>j</sub>->a，之后消除A<sub>i</sub>的直接左递归

   >A -> Ba | a
   >
   >B -> Cb | b
   >
   >C -> Cc | Ac | c
   >
   >---
   >
   >做法1：排序ABC
   >
   >| (i, j) | A                                             | B                                                     | C                                                            |
   >| ------ | --------------------------------------------- | ----------------------------------------------------- | ------------------------------------------------------------ |
   >| A      | #1                                            | -                                                     | -                                                            |
   >| B      | #2                                            | #3                                                    | -                                                            |
   >| C      | #4<br />C -> Cc \| ~~Ac~~ \| c *\| Bac \| ac* | #5<br />C-> Cc \| c \| ~~Bac~~ \| ac *\| Cbac \| bac* | #6<br />C-> cQ \| acQ \| bacQ<br />Q-> cQ \| bacQ \| &epsilon; |
   >
   >做法2：排序CBA
   >
   >| (i, j) | C                                                | B                                                 | A                                                            |
   >| ------ | ------------------------------------------------ | ------------------------------------------------- | ------------------------------------------------------------ |
   >| C      | #1<br />C -> AcQ \| cQ<br />Q -> cQ \| &epsilon; | -                                                 | -                                                            |
   >| B      | #2<br />B -> ~~Cb~~ \| b \| *AcQb \| cQb*        | #3                                                | -                                                            |
   >| A      | #4                                               | #5<br />A -> ~~Ba~~ \| a \| *ba \| AcQba \| cQba* | #6<br />A -> aP \| baP \| cQbaP<br />P -> cQbaP \| &epsilon; |
   >
   >我们的依赖链是C -> B -> A，显然，逆着依赖链排序更方便
   >
   >

## 提取左公因子

> P -> abA | abcA
>
> ab是abA和abcA的左公因子，需要提取
>
> P -> abQ
>
> Q -> A | cA