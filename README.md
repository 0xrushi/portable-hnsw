<h1><img src="https://github.com/jasonjmcghee/portable-hnsw/assets/1522149/8fab793b-e0a2-4fc9-b813-952c95822705" height="24"> Portable HNSW </h1>

<a href="https://www.loom.com/share/a5cb417115684a3e82a5fd3f20266489"><img style="max-width:300px;" src="https://cdn.loom.com/sessions/thumbnails/a5cb417115684a3e82a5fd3f20266489-with-play.gif"></a>

To build your own index:

```bash
poetry install

# To run original version
poetry run python build_index.py <path to text file> [output folder]

# To run quantized version
poetry run python build_index_quantized.py <path to text file> [output folder]
# You also need to change the script path to quantized (import './scripts/vectordb_utils_quantized.js';  in index.html)

```

Or you can jump into the code and do more complex use cases.

Then throw it in a GitHub repo and enable GitHub Pages. You can add / edit the index.html or test it by pasting the link to the folder in the "path" input [here](https://jasonjmcghee.github.io/portable-hnsw/)

Note: `rangehttpserver` works well as a simple server to support range requests for locally testing duckdb parquet + large indices.

---------------

So what's going on here?

Yeah - fair question.

So I had this idea. 

What if an HNSW index ([hierarchical navigable small world graphs](https://arxiv.org/abs/1603.09320) - _a good way to enable searching for stuff by their underlying meaning_) was just a file, and you could serve it from a CDN, and search it directly in the browser?

And what if you didn't need to load the entire thing in memory, so you could search a massive index
without being RAM rich?

That would be cool.

A vector store without a server...

So yeah. Here's a proof of concept.

---

There's a Python file called `build_index.py` that builds an index using a custom hnsw algorithm that 
can be serialized to a couple of parquet files.

_There are very likely bugs and performance problems. But it's within an order of magnitude or two
of `hnswlib` which was fast enough that my development cycle wasn't impacted by repeatedly re-indexing
the same files while building the search and front-end bits. I welcome pull requests to fix the problems
and make it halfway reasonable._

Then I wrote a webpage that uses `transformers.js`, `duckdb` and some SQL to read the parquet files and 
search it (similar to HNSW approx nearest neighbor search) and then retrieve the associated text.

A big part of the original idea was how this could scale to massive indices.

So, I also tested using parquet range requests and only retrieving what we need from the parquet file,
which worked! But since the index is only like 100MB, and each range request added overhead, loading
it all into memory was about twice as fast. But, it means you could have a 1TB index and it would
(theoretically) still work, which is pretty crazy.

You can try this yourself by swapping out the `nodes.parquet` bits in the SQL for `read_parquet('${path}/nodes.parquet')`. And the same with edges. DuckDB takes care of the rest.

---

Anyway, would love feedback and welcome contributions.

It was a fun project!
