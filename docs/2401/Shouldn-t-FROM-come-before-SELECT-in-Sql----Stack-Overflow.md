<!--yml

类别：未分类

日期：2024-05-27 15:09:37

-->

# [Sql 中 FROM 应该在 SELECT 之前吗？ - Stack Overflow](https://stackoverflow.com/questions/5074044/shouldnt-from-come-before-select-in-sql)

> 来源：[`stackoverflow.com/questions/5074044/shouldnt-from-come-before-select-in-sql`](https://stackoverflow.com/questions/5074044/shouldnt-from-come-before-select-in-sql)

是的，这很奇怪，也很不直观。休·达文（Hugh Darwen）对这种现状的产生提出了理论：

> 你是否认为 SELECT-FROM-WHERE 是理所当然的，还是像我一样，觉得 System R 团队竟然拒绝了常规的表达任意复杂性的方式，而选择了一些完全特殊的、可以说是相当专断的方式……？
> 
> 事实上，上世纪六十年代出现了各种脚本语言（正如我们今天倾向于称呼这样的东西），目的是为了报告生成，尤其是临时报告生成。我们在 IBM 从 1969 年到 1977 年工作的预关系型 DBMS 中有一种语言叫做 Terminal Business System（TBS）。我们的语言要求用户按照*规定的顺序*在一系列步骤中指定所需的报告…
> 
> IBM 美国后来开发了一个类似但更复杂的报告生成器，作为名为（像 IBM 那个时代的风格一样不太吸引人）广义信息系统（GIS）的产品的一部分……当我第一次看 SQL 时，我立刻反应道：“哦，不！是 GIS 的后代吗？请不要！”我可能完全错了。我所感受到的相似可能是虚假的，即使不是，我也没有确凿的证据表明 System R 团队中的任何人都熟悉 GIS。事实仍然是，当时一种固定行动顺序的一般风格是当时的潮流。我假设 SQL 的 SELECT-FROM-WHERE 就是由此产生的。

来自 [度过`HAVING`时光](http://www.dcs.warwick.ac.uk/~hugh/TTM/HAVING-A-Blunderful-Time.html)
