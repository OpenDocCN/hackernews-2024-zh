<!--yml
category: 未分类
date: 2024-05-27 13:16:22
-->

# Embeddings are a good starting point for the AI curious app developer

> 来源：[https://bawolf.substack.com/p/embeddings-are-a-good-starting-point](https://bawolf.substack.com/p/embeddings-are-a-good-starting-point)

Vector embeddings have been an Overton window shifting experience for me, not because they’re sufficiently advanced technology indistinguishable from magic, but the opposite. Once I started using them, it felt obvious that this was what the search experience was always supposed to be: less “How did you do that?” and more mundanely, “Why isn’t this everywhere?”

This feels like the right place to start if you’re an app developer looking for an excuse to dip your toes into this new AI world. Embeddings are just arrays of numbers, but they contain a compressed form of a considerable amount of human knowledge and shrink features that used to be substantial specialized projects into ones that individual product engineers can take on.

There are a ton of tooling options available to use embeddings. I’ll highlight our choices and note where you might want to make different ones for your situation. Here are some points I hope you take away:

*   Vector embeddings work for search and recommendations because they’re good at measuring similarity to arbitrary input. This even works for different spoken languages like French or Japanese.

*   [Pgvector](https://github.com/pgvector/pgvector/) is a Postgres extension that stores and queries embeddings without adding a new service. It’s powerful because it can combine standard SQL logic with embedding operations.

*   Unlike LLMs, working with embeddings feels like regular deterministic code.

My friend Charlie Yuan and I built this mini [icon app](https://www.v0.app) to help people discover icons. It’s pretty short and sweet. We have icon sets you can query, bookmark, and add to your project.

There are a bunch of [specialized vector databases](https://lakefs.io/blog/12-vector-databases-2023/) to choose from. Instead, we chose Postgres with [pgvector](https://github.com/pgvector/pgvector/) to blend embedding search with business logic like filtering and scoring. While it’s not the fastest vector database, we didn’t want to have to compare results across multiple data sources. Pgvector probably already has [a library for your favorite database client](https://github.com/pgvector/pgvector). Our project was Typescript through and through, and we used [drizzle-orm](https://github.com/pgvector/pgvector-node?tab=readme-ov-file#drizzle-orm). The docs will be a more robust place for setup documentation, so I’m leaving out that part to focus on the features you can build.

Once set up with pgvector, we created a strategy for encoding our icon data into vector embeddings. Embeddings are points in many-dimensional space, up to thousands of dimensions. Unfortunately, the axes of that grid are not humanly grokable ideas like “size” or “brightness.” They’re a bit of a black box. Luckily, like any good abstraction, they’re a black box with a good API.

The best practice seems to be finding the details that best represent what people want to search for and creating a function that outputs that as a string. Our icons can be a part of many use-case-based ‘categories’ and have many descriptive ‘tags’ associated with them. We encoded that information along with the icon name because that best represents what the icon is. Whereas the name of the icon set the icon belongs to, or its dimensions aren’t relevant. The strings we generated looked basically like this:

```
const createIconEmbeddingsString = (icon) => `icon: "${icon.name}", categories: [${categories}] tags: [${tags}]`;
```

Next, we chose an embedding model. [OpenAI’s embedding models](https://platform.openai.com/docs/guides/embeddings) will probably work just fine. We’re using their `text-embedding-3-small`. If you want to dive in, check out the [leaderboard](https://huggingface.co/spaces/mteb/leaderboard) and pick the model that best meets your needs. Whether you use an embeddings API or self-host an open source option, the interface should be text in and embeddings out.

Many sites implement search, but most icon sites implement search by [text](https://icon-sets.iconify.design/?query=woof)  [matching](https://react-icons.github.io/react-icons/search/#q=woof) or full-text search. If you’re looking for a dog icon, they search over the icon metadata for icons that have ‘dog’ in them. If they want to get [craftier](https://fontawesome.com/search?q=woof&o=r), they come up with a bag of words related to ‘dog,’ like maybe ‘k9’, ‘puppy,’ and ‘woof’ to catch near misses. That’s pretty fragile. Someone has to choose tags for each icon; if they miss an important one, users won’t find what they’re looking for.

Our app gets relevant results when searching for ‘[dog](https://www.v0.app/search?g=popularity&q=dog&qid=nmezx8efde3pvydym2f2sk9n).’ We also get solid results for ‘[puppy](https://www.v0.app/search?g=popularity&q=puppy&qid=v27kjdebk8ex7n30qe4b5tfs)’ without a bag of words by measuring the cosine similarity between the embeddings of your search query and each icon. There are multiple ways to measure how similar embeddings are to each other, but OpenAI’s embeddings are designed to work well with cosine distance. Cosine similarity is just the opposite of cosine distance. Order by cosine distance wherever you can to take advantage of [indexes](https://github.com/pgvector/pgvector?tab=readme-ov-file#why-isnt-a-query-using-an-index).

```
cosine_similarity(x,y) = 1 - cosine distance(x,y)
```

You can even try dog breeds like ‘[hound](https://www.v0.app/search?q=hound),’ ‘[poodle](https://www.v0.app/search?q=poodle),’ or my favorite ‘[samoyed](https://www.v0.app/search?q=samoyed).’ It pretty much just works. But that’s not all; it also works for other languages. Try ‘[chien](https://www.v0.app/search?g=popularity&q=chien&qid=mcbp6rk9z3y8lb1emt1zblev)’ and even ‘[犬](https://www.v0.app/search?g=popularity&q=%E7%8A%AC&qid=hd8whx733f3rljhadasgfzb9)’! With pgvector, we can get these results with simple SQL queries.

```
SELECT
  1 - cosine_distance (search_query.embedding,
    icon.embedding) as similarity,
  *
FROM
  icon
  join search_query on search_query.text = 'dog'
ORDER BY
  cosine_distance (search_query.embedding, icon.embedding) ASC
LIMIT 50;
```

Since we’re ordering by distance, every search returns every result in the table in order of closeness. We cut off results by a fixed number to make this manageable, limiting the query to the top 50\. Using an arbitrary distance cut-off is tempting, like querying for results with a cosine similarity of less than 0.8\. Unfortunately, the absolute distance for one query to produce correct-feeling results might differ drastically from another. We limit by the number of results, not by a minimum value, whenever possible.

If we only wanted similarity search, any vector database would be fine, but Postgres allowed us to layer more features on top. The initial search looks across *all icons* in *all icon sets*, but our user’s style might only match some of the icon sets. We can filter by values like in any other Postgres query.

```
SELECT
  1 - cosine_distance (search_query.embedding,
    icon.embedding) AS similarity,
  *
FROM
  icon
  JOIN search_query ON search_query.text = 'dog'
  JOIN icon_set ON icon_set.slug = icon.icon_set_slug
WHERE
  icon_set.slug in('lucide', 'mdi')
ORDER BY
  cosine_distance (search_query.embedding,
    icon.embedding) ASC
LIMIT 50;
```

This is deterministic; everyone searching for a ‘dog’ will get the same results. However, the inputs are still unbounded, so embedding search doesn’t guarantee that the best results will be produced for every input. We can try different embedding models or ways of encoding icons into embeddings to improve the system.

The embedding search usually puts the correct icon on the page, but the correct icon isn’t always the first result. We could make a simple algorithm that adjusts to user feedback and improves over time. To do this, we’d count every time a user clicks on an icon for a particular search query. When ranking search results, we’d create a score for each icon that combines the embedding search with the click data. 

Here’s a simple ranking algorithm. The details look messy because there’s some null checking and unit conversions, but here are the basics.

1.  Get the cosine similarity of the icon for the search query. It will be a number between 0 and 1\. Multiply it by 0.5.

2.  Divide the number of clicks for each icon by the icon with the most clicks for that query. This normalizes the most clicked icon to 1, and the least clicked to 0\. Multiply by 0.5.

3.  The final score is these two values added together for a range between 0-1.

```
SELECT
  (
    0.5 * COALESCE(      -- so nulls are turned into 0
     1 - cosine_distance (search_query.embedding, icon.embedding),
     0
    ) + 0.5 *     -- so clicks matter less than embeddings.
    COALESCE( 
      search_query_selection. "count"::decimal / 
        max(search_query_selection. "count") OVER (),
    0)
  ) AS score,
  icon.*
FROM
  icon
  LEFT JOIN search_query_selection
    ON icon.id = search_query_selection.icon_id
  LEFT JOIN search_query
    ON search_query.text = 'dog'
      AND search_query.id = search_query_selection.search_query_id
  ORDER BY score DESC
  LIMIT 50;
```

Should vector embedding distance and clicks be equally weighted? Should the order of magnitude of clicks matter more than the raw number? The algorithm might need tuning, but this is just an example of how the database handles the calculation nicely.

With a separate vector database, we might have to get values for all icons from both databases before comparing them in application code or making tradeoffs like pulling 100 results from the vector db and filtering the Postgres query for click score to those results or vice versa. Instead, we simply query for results and display them.

Additionally, we include a content forward section of each [icon page](https://www.v0.app/icon/lucide/arrow-up) with other icons in the same icon set and category. That way, you can see other ‘navigation’ icons when looking at `arrow-up` in case you need those. Unfortunately, not all of our icon sets have categories. In these cases, we make a similar icons section using embeddings. Instead of getting an embedding from user input, we can compare the same cosine similarity measure against the selected icon’s embedding.

```
WITH current_icon AS (
    SELECT
      embedding,
      slug,
      icon_set_slug
    FROM
      icon
    WHERE
      icon_set_slug = 'lucide'
      AND slug = 'activity'
)
SELECT
    *
FROM
    icon
INNER JOIN current_icon ON
icon.icon_set_slug = current_icon.icon_set_slug
AND icon.slug != current_icon.slug
ORDER BY
    1 - cosine_distance (
    icon.embedding,
    current_icon.embedding
)
LIMIT 50;
```

I hope you have a good experience playing with our app and vector embeddings! We owe a big thanks to all the engineers who pushed the state of the art so far that we can stand on their shoulders. I hope this modest contribution helps make adding embedding features to your app approachable!

Here is a summary of our implementation decisions and some sources for other options.

We chose [pgvector](https://github.com/pgvector/pgvector)/Postgres, but there are plenty of [other choices](https://lakefs.io/blog/12-vector-databases-2023/), including some for other standard databases like MongoDB.

We worked in Typescript and chose [drizzle-orm](https://github.com/pgvector/pgvector-node?tab=readme-ov-file#drizzle-orm). I’ve also worked in the Phoenix elixir ecosystem using the [elixir library](https://github.com/pgvector/pgvector-elixir) with Ecto. You can find a library for your client of choice [here](https://github.com/pgvector/pgvector/?tab=readme-ov-file#languages).

Our app is hosted on [Neon](https://neon.tech/). I’ve also used [fly.io](https://fly.io/), though their option wasn’t *really* a managed instance at the time. Now, they have a [managed solution](https://fly.io/docs/reference/supabase/) with Supabase. If you want to look up someone else, Pgvector has a clumsy list of hosts that support it in [this git issue](https://github.com/pgvector/pgvector/issues/54).

We chose OpenAI’s `text-embedding-3-small`. If you want to try something else, check out Huggingface’s [leaderboard](https://huggingface.co/spaces/mteb/leaderboard).

We embedded strings of key and value pairs for the attributes we thought best described our icons. It doesn’t seem like any of the major players are doing anything crazy here; the significant knobs appear to be whether or not to embed keys or just values and which attributes of your records are relevant. Other examples in OpenAI’s [cookbook](https://cookbook.openai.com/examples/vector_databases/readme) show [other](https://github.com/vercel/examples/blob/main/storage/postgres-pgvector/prisma/seed.ts#L35)  [choices](https://github.com/neondatabase/yc-idea-matcher/blob/18eb9dd6ddd14eeeb2167d78088f092ab6882f42/generate-embeddings.ts#L51).

We used cosine similarity as our distance function because that’s what OpenAI [recommends](https://platform.openai.com/docs/guides/embeddings/which-distance-function-should-i-use) for their embeddings. Other embeddings may be optimized for different strategies. Pgvector [supports](https://github.com/pgvector/pgvector?tab=readme-ov-file#distances) l2 distance, inner product, and cosine distance.

In these examples, we limited our queries to the top 50 results. You can also limit your search to be above or below a certain distance threshold. That doesn’t seem super reliable. Relative distances seem more meaningful than discrete amounts. If you’re set on using a threshold, I’d recommend keeping it pretty wide, like 0.1 or 0.05, and using it with a limit. You may get some mileage using a wide range to avoid returning irrelevant long-tail results.