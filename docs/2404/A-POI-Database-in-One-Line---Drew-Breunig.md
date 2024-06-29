<!--yml

类别：未分类

日期：2024-05-27 13:20:14

-->

# 一行代码创建 POI 数据库 | Drew Breunig

> 来源：[https://www.dbreunig.com/2024/04/18/a-poi-database-in-one-line.html](https://www.dbreunig.com/2024/04/18/a-poi-database-in-one-line.html)

本周，[Overture Maps Foundation](https://overturemaps.org/) 发布了 [一个方便的命令行工具](https://github.com/OvertureMaps/overturemaps-py) 用于下载 Overture 数据集的子集。使用方法很简单：

```
$ pip install overturemaps
$ overturemaps download --bbox=-71.068,42.353,-71.058,42.363 -f geojson --type=building -o boston.geojson 
```

这个命令下载一个包含波士顿一部分 Overture 建筑物的 GeoJSON 文件。我们将 GeoJSON 输出到名为`boston.geojson`的本地文件。

使用新工具，我想重新审视使用 Overture 创建 POI 数据库的练习。[上次](https://gist.github.com/dbreunig/bd13dd99d6d708560bfcca9f45c9fb29)，我们需要使用`aws-cli`和`duckdb`来整理 162 行代码才能得到可用的结果。这一次我们会用多少行？

只需要一行代码：

```
$ overturemaps download --bbox=-71.068,42.353,-71.058,42.363 -f geojsonseq --type=place | geojson-to-sqlite boston.db places - --nl --pk=id 
```

我们对 `overturemaps` 地图调用进行了三处修改：指定我们需要的是 `place` 数据，输出 `geojsonseq`（或者换行分隔的 GeoJSON），并删除输出参数。如果没有指定输出，地点数据将输出到 `stdout`，我们可以将其管道传输到 Simon Willison 的 [geojson-to-sqlite](https://simonwillison.net/2020/Jan/31/geojson-sqlite/)，确保我们使用的是换行分隔的 GeoJSON，并将主键指定为 `id`。

在我的设备上，这项任务大约花费了 ~9 秒来下载全球 Overture 地点数据集中的 1,648 个地点。

但我们可以做得更好！如果我们想要获取我镇的所有地点，我必须弄清楚边界框。这有点麻烦。幸运的是，LLMs 在生成城镇、城市、州和地区的边界框方面非常擅长。因此，让我们使用 Simon 的 [llm](https://llm.datasette.io/en/stable/) 工具来生成我们的边界框（这里使用 OpenAI 的 `gpt-3.5`），使用命令替换：

```
$ overturemaps download --bbox=$(llm 'Give me a bounding box for Alameda, California expressed as only four numbers delineated by commas, with no spaces, longitude preceding latitude.') -f geojsonseq --type=place | geojson-to-sqlite alameda.db places - --nl --pk=id 
```

`llm` 返回“-122.336,37.572,-122.187,37.804”，非常准确。

这个一行代码下载了 4,005 个位置数据，并在大约 14 秒内将它们加载到 SQLite 数据库中。
