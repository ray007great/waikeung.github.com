---
layout: post
title: "语法制导翻译"
description: ""
category: 编译原理
tags: [编译原理,语义分析,语法制导翻译]
---

语法制导翻译技术是语义分析的主流技术。

根据语法制导和翻译模式，可以对输入的符号串进行语法分析，建立分析树，通过分析树，根据需要在树的结点处使用语义规则进行计算，这样的计算可能生成代码、在符号表中存放信息、给出错误信息或执行其他动作。

使用语义规则进行计算所得到的结果就是输入符号串的翻译。

在语法知道翻译中，每一个文法符号都有一个与之相联系的属性集合。属性可以分为两类，综合属性和继承属性。

*   综合属性
    
    属性值是分析树中该结点的子结点的属性值的函数

*   继承熟悉
    
    属性值是分析树中该结点的父结点和/或兄弟结点的属性值的函数

某个结点的综合属性需要通过该结点的子结点的属性值才能计算出来;某结点的继承属性需要通过该结点的父节点和/或兄弟结点才能计算出来。

由此可以看出，综合属性是用于“自下而上”传递信息的，而继承属性是“自上而下”传递信息的。

注意几点：

*   非终结符号(开始符号除外)既可以有综合属性也可以有继承属性。 (这样的结点在分析树中就是既有父结点，又有子结点的)
*   文法开始符号没有继承属性 (开始符号结点一没父结点，二没兄弟结点，从何处继承)
*   终结符号只有综合属性，一般油词法分析器提供。 (终结符号的属性通常都是要用来计算其他结点属性的)

### 语法制导定义

> 为每一条文法产生式 A → α 引入一套语义规则，规则的形式: b = f(c<sub>1</sub>, c<sub>2</sub>, ..., c<sub>k</sub>), f是一个函数。
> 
> *   b 是 A 的综合属性， 则 c<sub>1</sub>, c<sub>2</sub>, ..., c<sub>k</sub>是产生式右边文法符号的属性。
> *   b 是产生式右部某文法符号的继承属性， 则c<sub>1</sub>, c<sub>2</sub>, ..., c<sub>k</sub>是产生式文法符号的属性。

这两种情况，刚好分别对应了综合属性和继承属性的计算。只有在已知c<sub>1</sub>, c<sub>2</sub>, ..., c<sub>k</sub>值的基础上，才能计算得到属性值b。所以称属性b依赖与属性c<sub>1</sub>, c<sub>2</sub>, ..., c<sub>k</sub>。

关于语义规则，可以计算属性值，也可以是语义动作(在符号表中登录信息，输出错误信息，进行类型检查，产生中间代码等)。

#### S-属性定义

S-属性定义是只含有综合属性的语法制导定义。

句子 → 分析树 → 最左归约序列(产生式) → 语义规则 → 执行

依赖图：用来表示属性之间依赖关系的有向图。 有向边：a → b，表示属性b依赖于属性a。

属性的计算顺序：按照有向非循环图的拓扑排序。依赖图的任一拓扑排序是一个合理的属性计算顺序。

![依赖图][1]

这就是一个依赖图。可以看出依赖图中有向边的走向和自低向上分析时建立分析树的顺序是一致的，因此可以在进行语法分析(自低向上)的同时进行翻译(执行语义规则)。

具体实现：扩充LR分析器，为栈中的每一个文法符号增加一个属性域，存放分析过程中该文法符号的综合属性值，当用产生式进行归约时，产生式左边文法符号入栈，其属性值由栈中正在归约的产生式右边符号的属性值计算。

#### L-属性定义

对于每一个产生式 A → X<sub>1</sub>X<sub>2</sub>...X<sub>n</sub>，其中每一个语义规则中的每一个属性都是综合属性(S-属性定义)，或者是X<sub<j</sub>的一个继承属性，它依赖于：

1.  产生式中X<sub>j</sub>的**左**边符号X<sub>1</sub>,X<sub>2</sub>,...,X<sub>j-1</sub>的属性;
2.  A的继承属性。

每一个S-属性定义都是L-属性定义。

句子 → 分析树 → 带语义动作 → 深度优先遍历 → 语义动作的执行顺序 → 执行

构造翻译模式 → 带语义动作的分析树

翻译模式：

> 将语义规则放到{}中，并插入到产生式右部的适当位置，以反映语义规则的计算顺序，这种书写形式称为翻译模式。

翻译模式与语法制导定义的区别在于翻译模式中指明了语义规则的执行顺序。

##### 为L-属性定义构造翻译模式

适合以深度优先顺序计算属性的翻译模式需满足一下条件：

1.  产生式右部文法符号的继承属性必须在这个符号以前的动作中计算出来;
2.  一个动作不能引用该动作右边符号的综合属性;
3.  产生式左部非终结符号的综合属性只有在其引用的所有属性都计算出来后才能计算。计算该属性的动作通常放在产生式的末尾。

构造方法：

1.  将计算产生式右边某文法符号的继承属性的语义规则插入到此符号之前
2.  将计算产生式左边非终结符号综合属性的语义规则放在产生式右端的末尾。

 [1]: /assets/images/DependencyGraph.png
