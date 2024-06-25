<!--yml

category: 未分类

date: 2024-05-27 15:02:25

-->

# WaterwayMap.org - OSM 河流流域

> 来源：[`waterwaymap.org`](https://waterwaymap.org)

## 过滤数据

根据类型显示水路。

<template x-if="!$store.tilesets_loaded"></template><template x-if="$store.tilesets_loaded"></template>

## 过滤长度

根据长度排除水路。

## 颜色

分组的水路颜色是随机分配的。

## 框架

显示穿过大型水路组的路线。 用于查找组的连接方式。

<template id="data_age" x-data="" x-if="$store.tilesets_loaded"></template>
