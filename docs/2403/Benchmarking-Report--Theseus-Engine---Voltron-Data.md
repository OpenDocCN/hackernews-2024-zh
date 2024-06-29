<!--yml

category: 未分类

date: 2024-05-29 12:30:40

-->

# 基准报告：Theseus Engine | Voltron Data

> 来源：[https://voltrondata.com/benchmarks/theseus](https://voltrondata.com/benchmarks/theseus)

1 Pricing Summary Report Query 报告已开票、已发货和已退货的业务量。

SUM, COUNT, AVG, WHERE, GROUP BY, ORDER BY Heavy Aggregation 2 Minimum Cost Supplier Query 查找特定区域内零件的最便宜供应商，考虑到折扣。

Subquery, INNER JOIN（7），WHERE, MIN, ORDER BY Heavy Sort 3 Shipping Priority Query 列出在给定时间段内最有利可图的客户，重点关注已下大量订单的客户。

SUM, INNER JOIN（2），WHERE, GROUP BY, ORDER BY Heavy Join 4 Order Priority Checking Query 标识在特定时间范围内已下但尚未发货的订单，突出潜在的延迟。

COUNT, WHERE, GROUP BY, ORDER BY Heavy Join 5 Local Supplier Volume Query 计算在特定区域内每个国家的销售收入，强调区域促销活动的有效性。

SUM, INNER JOIN（5），GROUP BY, ORDER BY, INTERVAL Heavy Join 6 Forecasting Revenue Change Query 根据对特定项目销售应用的折扣，预测给定年份的收入变化。

SUM, WHERE, INTERVAL Heavy Aggregation 7 Volume Shipping Query 确定在特定时间段内在国家之间运输的货物量，突出国际贸易模式。

SUM, INNER JOIN（5），GROUP BY, ORDER BY Heavy Join 8 National Market Share Query 评估全国市场份额对特定零件类型的影响，关注一个国家的变化如何影响其他国家的销售。

SUM, INNER JOIN（7），WHERE, GROUP BY, ORDER BY Heavy Join 9 Product Type Profit Measure Query 评估从原材料到最终销售的整个供应链中特定零件产生的利润。

SUM, INNER JOIN（5），GROUP BY, ORDER BY Heavy Aggregation 10 Returned Item Reporting Query 根据其订单价值确定特定时间段内最有利可图的客户。

SUM, INNER JOIN（3），GROUP BY, ORDER BY Heavy Sort 11 Important Stock Identification Query 根据供应商提供的零件数量排名供应商，相对于该国家可用总量。

SUM, Subquery, GROUP BY, HAVING, INNER JOIN（2），WHERE, ORDER BY Heavy Aggregation 12 Shipping Modes and Order Priority Query 检查运输模式和订单状态，以确定订单处理系统的效率。

SUM, WHERE, GROUP BY, INNER JOIN（1），ORDER BY, INTERVAL Heavy Join 13 Customer Distribution Query 统计具有特定特征的客户及其订单数量，重点关注可能表明客户忠诚度或问题的客户。

LEFT OUTER JOIN（1），COUNT, GROUP BY, ORDER BY, Heavy Aggregation 14 Promotion Effect Query 计算促销折扣对某一时期内销售的收入影响。

SUM, WHERE, JOIN (1), CASE WHEN, INTERVAL Heavy Aggregation 15 顶级供应商查询根据特定时间范围内的收入贡献确定顶级供应商。

子查询, WHERE, INTERVAL, GROUP BY, MAX, SUM, INNER JOIN (1), ORDER BY Heavy Join 16 零部件/供应商关系查询根据品牌、类型和尺寸等特征分析零部件的需求，确定潜在的库存调整。

SCOUNT, INNER JOIN (1), WHERE, GROUP BY, ORDER BY Heavy Sort 17 小数量订单收入查询检查零部件的平均订购数量，突出显示库存水平或定价策略可能存在的问题。

SUM, INNER JOIN (1), WHERE Heavy Aggregation 18 大量客户查询列出大量客户及其订单，重点关注订单大小与客户重要性之间的关系。

SUM, INNER JOIN (2), GROUP BY, ORDER BY Heavy Join 19 折扣收入查询评估三种不同类型产品的销售盈利能力，考虑各种折扣和运输方式。

SUM, WHERE (MULTIPLE AND’s), JOIN (1) Heavy Join 20 潜在零部件促销查询确定有库存以满足特定大订单的供应商，考虑到需要未备货零部件的交货时间。

INNER JOIN (1), WHERE, ORDER BY, INTERVAL Heavy Aggregation 21 延迟订单的供应商查询检查可能因延迟订单而面临业务流失风险的供应商，重点关注供应商的可靠性。

COUNT, LEFT OUTER JOIN (3), WHERE, GROUP BY, ORDER BY Heavy Join 22 全球销售机会查询通过将客户列表与订单历史匹配，识别潜在的销售机会，重点关注营销的重点领域。

聚合函数 (SUM, COUNT, AVG), WHERE, GROUP BY, ORDER BY Heavy Join
