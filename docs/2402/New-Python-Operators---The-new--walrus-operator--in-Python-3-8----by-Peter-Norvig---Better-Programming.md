<!--yml

类别：未分类

日期：2024-05-27 14:59:44

-->

# 新 Python 操作符！Python 3.8 中的新“海象操作符”…… | Peter Norvig | Better Programming

> 来源：[https://betterprogramming.pub/new-python-operators-9f31b56ddcc7?gi=7918b2c3ccf0](https://betterprogramming.pub/new-python-operators-9f31b56ddcc7?gi=7918b2c3ccf0)

海象操作符 `:=`

Python 3.8 中的新“海象操作符”写作 `:=`，引发了许多[讨论](https://docs.python.org/3/whatsnew/3.8.html)。本文描述了 Python 中其他富有幽默命名但不那么广为人知的多字符操作符。

这些操作符于2023年4月1日宣布，但就像多萝西和她的红宝石鞋一样，你总是有能力使用它们，只是需要自己学会而已！

# 滑雪帽操作员

“滑雪帽”操作员写作 `*=0`，可以用来清空变量，无论是列表、字符串、元组还是数值。

例如，执行以下代码后，`skiers` 是一个空列表。

```
skiers = ["Lindsey", "Alberto", "Bode"]

skiers *=0
```

滑雪帽操作员 `*=0`

# 哑铃操作员

“哑铃操作员”写作 `[:]=[]`，也可以用来清空列表，但不如滑雪帽操作员那么灵活，因为它对大多数其他类型不起作用。

执行以下代码后，`reps` 将是一个空列表：

```
reps = [1, 2, 3, 4, 5]

reps [:]=[]
```

哑铃操作员 `[:]=[]`

# 落喵操作员

“落喵”操作员写作 `,=`，用于提取可迭代对象的第一个元素。就像一只挑剔的猫一样，如果可迭代对象中还有其他烦人的元素，它就会抱怨。

执行以下代码后，`water` 是 `'HHO'`。

```
water ,= ['H'*2+'O']
```

落喵操作员 `,=`

# 星际飞船操作员

“星际飞船”操作员写作 `, *_=`，描绘了一个双动力舰（如企业号）和一个光子鱼雷。它类似于落喵操作员，用于提取可迭代对象的第一个元素，但允许可迭代对象有多个元素。

执行以下代码后，`NCC` 是 `1`：

```
NCC    , *_=    [1, 7, 0, 1]
```

星际飞船操作员 , *_=

# 飞碟操作员

“飞碟”操作员写作 `--0--`，执行整数除法“上梁”，使其向上舍入而不是向下舍入。（我从[Mark Dickinson](https://www.enthought.com/team/mark-dickinson-2/)处了解到这个。）

以下表达式评估为 5，而不是 4：

```
--0--   42 // 10
```

飞碟操作员 `--0--`

# 强调操作员

“强调操作员”由整数值表达式用星号括起来编写，并用于通过重复强调以下序列。例如，

```
sigh = 3
['oh', 'good', 'grief', *sigh* '!']
```

评估为 `['oh', 'good', 'grief', '!', '!', '!']`。

强调操作员 *叹息*

# 阶乘操作员

数学爱好者将高兴地了解到阶乘操作符 `n!` 已部分整合到 Python 中。

不幸的是，实现还不完整。但这些单元测试都通过了，所以这已经足够好了，对吧？

```
assert 0!=1
assert 3!=6
assert 4!=24
assert 5!=120
assert 6!=720
assert 7!=5040
assert 8!=40,320
assert 9!=362,880
```

# 抽象关键字

Python 还允许您使用关键字 `abstract` 来指示抽象类的方法必须在子类中实现以便对任何实例化对象进行调用。

在以下代码中，方法 `name` 被定义为 `abstract`，因此对该类的对象调用将导致错误消息指出该方法未定义。

```
class AnAbstractClass:
    def name(self): 
        abstract

>>> AnAbstractClass().name()
NameError: name 'abstract' is not defined
```

# 更多

机智的读者会意识到，这些新运算符依赖于空格的放置方式，这违反了 [PEP 8](https://peps.python.org/pep-0008/) 的规定。

于此我说：但是我的方式更有趣！特别是今天，4月1日。有关更多运算符，包括来自C++的 `++` 和 `<<` 运算符的实现，请参阅我的 [old pos](https://norvig.com/python-iaq.html)t。

PS: 本文所有内容属实，除了“那就够了，对吧？”
