<!--yml
category: 未分类
date: 2024-05-27 13:20:14
-->

# A POI Database in One Line | Drew Breunig

> 来源：[https://www.dbreunig.com/2024/04/18/a-poi-database-in-one-line.html](https://www.dbreunig.com/2024/04/18/a-poi-database-in-one-line.html)

This week, the [Overture Maps Foundation](https://overturemaps.org/) released [a handy command-line tool](https://github.com/OvertureMaps/overturemaps-py) for downloading a subset of the Overture dataset. Usage is simple:

```
$ pip install overturemaps
$ overturemaps download --bbox=-71.068,42.353,-71.058,42.363 -f geojson --type=building -o boston.geojson 
```

This command downloads a GeoJSON file containing all the Overture buildings in a chunk of Boston. We’re outputting GeoJSON to a local file named `boston.geojson`.

With the new tool, I wanted to revisit the exercise of creating a POI database with Overture. [Last time](https://gist.github.com/dbreunig/bd13dd99d6d708560bfcca9f45c9fb29), it took us 162 lines of wrangling with the `aws-cli` and `duckdb` to get us something usable. How many lines will it take us this time?

It’s just one line:

```
$ overturemaps download --bbox=-71.068,42.353,-71.058,42.363 -f geojsonseq --type=place | geojson-to-sqlite boston.db places - --nl --pk=id 
```

We made three tweaks to the `overturemaps` maps call: specifying we want `place` data, outputting `geojsonseq` (or new-line delimited GeoJSON), and removing the output arguments. With no output specified, place data is output to `stdout`, which we can pipe to Simon Willison’s [geojson-to-sqlite](https://simonwillison.net/2020/Jan/31/geojson-sqlite/), taking care to specify we’re using new-line delimited GeoJSON and that the primary key is `id`.

On my machine, this task took ~9 seconds to pull down 1,648 places from the global Overture places dataset.

But we can do better! If we wanted to grab all the places in my town, I’d have to figure out the bounding box. Which is kind of a pain. Thankfully, LLMs are spookily good at producing bounding boxes for towns, cities, states, and regions. So lets incorporate Simon’s [llm](https://llm.datasette.io/en/stable/) tool to generate our bounding box (here, with OpenAI’s `gpt-3.5`), using command substitution:

```
$ overturemaps download --bbox=$(llm 'Give me a bounding box for Alameda, California expressed as only four numbers delineated by commas, with no spaces, longitude preceding latitude.') -f geojsonseq --type=place | geojson-to-sqlite alameda.db places - --nl --pk=id 
```

`llm` returns “-122.336,37.572,-122.187,37.804”, which is dead on.

This one-liner downloads 4,005 places and loads them into a sqlite database in about 14 seconds.