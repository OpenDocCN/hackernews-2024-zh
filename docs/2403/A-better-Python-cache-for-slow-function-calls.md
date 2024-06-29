<!--yml

category: 未分类

date: 2024-05-29 12:30:08

-->

# 一个更好的Python缓存用于缓慢的函数调用

> 来源：[https://docs.sweep.dev/blogs/file-cache](https://docs.sweep.dev/blogs/file-cache)

<main class="nx-w-full nx-min-w-0 nx-max-w-6xl nx-px-6 nx-pt-4 md:nx-px-12">

📚 博客

📂 一个更好的Python缓存用于缓慢的函数调用

# 一个更好的Python缓存用于缓慢的函数调用

**William Zeng** - 2024年3月18日

* * *

我们编写了一个文件缓存 - 它类似于Python的`lru_cache`，但它将值存储在文件中而不是内存中。这为运行我们的LLM基准测试节省了大量时间，并且我们希望将其分享为自己的Python模块。感谢[Luke Jaggernauth (在新标签页中打开)](https://github.com/lukejagg)（前Sweep工程师）为构建此初始版本而感谢！

这里是链接：[https://github.com/sweepai/sweep/blob/main/docs/public/file_cache.py (在新标签页中打开)](https://github.com/sweepai/sweep/blob/main/docs/public/file_cache.py)。要使用它，只需将`file_cache`装饰器添加到您的函数中。这里有一个示例：

```
import time
from file_cache import file_cache

@file_cache()
def  slow_function(x,  y):
 time.sleep(30)
 return x + y

print(slow_function(1, 2))  # -> 3, takes 30 seconds
print(slow_function(1, 2))  # -> 3, takes 0 seconds
```

## 背景

我们在Sweep花费大量时间进行prompt工程。我们的agents接受一组输入字符串，将它们格式化为提示，然后将提示和任何其他信息发送给LLM。我们链接多个agents来将GitHub问题转换为拉取请求。例如，要修改代码，我们将输入旧代码、任何相关上下文和指令，然后输出新代码。

典型的改进包括调整我们管道的一个小部分（如改进我们的规划算法），然后再次运行整个管道。我们使用pdb（Python的本地调试器）设置断点和检查我们的提示、输入值和解析逻辑的状态。例如，我们可以检查某个字符串是否与正则表达式匹配：

```
(Pdb) print(todays_date)
'2024-03-14'
(Pdb) re.match(r"^\d{4}-\d{2}-\d{2}$",  todays_date)
<re.Match object; span=(0,  10),  match='2024-03-14'>
```

这使我们能够在实际数据运行时调试。

## Cached pdb

pdb效果很好，但我们必须等待整个管道再次运行。想象一个pdb的版本，不仅中断执行，而且还缓存到该点为止的整个程序状态。我们解决同样的错误可能从10分钟缩短到15秒（提升40倍）。

我们没有构建这个，但我们认为我们的`file_cache`同样有效。

LLM调用虽然慢，但它们的输入和输出很容易缓存，从而节省了大量时间。我们可以使用输入提示/字符串作为缓存键，输出字符串作为缓存值。

### 与`lru_cache`有什么不同？

`lru_cache`非常适合对重复的函数调用进行记忆化，但它不支持两个关键功能。

1.  我们需要在运行之间持久化缓存。

    +   `lru_cache`将结果存储在内存中，这意味着下次运行程序时缓存将为空。`file_cache`将结果存储在磁盘上。我们也考虑过使用Redis，但写入磁盘更容易设置/管理。

1.  `lru_cache`不支持忽略使缓存无效的参数。

    +   我们将使用自定义的 `chat_logger`，用于可视化存储聊天记录。它包含当前时间戳 `chat_logger.expiration`，如果被序列化，将使缓存无效。

    +   为了解决这个问题，我们添加了忽略的参数，使用方式如下：`file_cache(ignore_params=["chat_logger"])`。这会从缓存键的构建中移除 `chat_logger`，并防止由于不断变化的 `expiration` 导致的无效缓存。

## 实现

我们的两个主要方法是 `recursive_hash` 和 `file_cache`。

### recursive_hash

我们希望稳定地对对象进行哈希，这在 [Python 中并非原生支持 (在新标签页打开)](https://death.andgravity.com/stable-hashing)。

```
import hashlib
from cache import recursive_hash

class  Obj:
 def  __init__(self,  name):
 self.name = name

obj =  Obj("test")
print(recursive_hash(obj))  # -> this works fine
try:
 hashlib.md5(obj).hexdigest()
except  Exception  as e:
 print(e)  # -> this doesn't work
```

hashlib.md5 单独对对象无效，导致错误：`TypeError: object supporting the buffer API required`。我们使用 `recursive_hash`，它适用于任意的 Python 对象。

```
def  recursive_hash(value,  depth=0,  ignore_params=[]):
 """Hash primitives recursively with maximum depth."""
 if depth > MAX_DEPTH:
 return hashlib.md5("max_depth_reached".encode()).hexdigest()

 if  isinstance(value, (int, float, str, bool, bytes)):
 return hashlib.md5(str(value).encode()).hexdigest()
 elif  isinstance(value, (list, tuple)):
 return hashlib.md5(
 "".join(
 [recursive_hash(item, depth +  1, ignore_params)  for item in value]
 ).encode()
 ).hexdigest()
 elif  isinstance(value, dict):
 return hashlib.md5(
 "".join(
 [
 recursive_hash(key, depth +  1, ignore_params)
 +  recursive_hash(val, depth +  1, ignore_params)
 for key, val in value.items()
 if key not  in ignore_params
 ]
 ).encode()
 ).hexdigest()
 elif  hasattr(value, "__dict__")  and value.__class__.__name__  not  in ignore_params:
 return  recursive_hash(value.__dict__, depth +  1, ignore_params)
 else:
 return hashlib.md5("unknown".encode()).hexdigest()
```

### file_cache

file_cache 是一个为我们处理缓存逻辑的装饰器。

```
@file_cache()
def  search_codebase(
 cloned_github_repo,
 query,
):
 # ... take a long time ...
 # ... llm agent logic to search through the codebase ...
 return top_results
```

如果没有缓存，使用我们的 LLM 代理在代码库中搜索 `top_results` 需要 5 分钟 - 如果我们实际上不在测试中，这时间太长了。相反，使用 file_cache，我们只需等待拾取对象的反序列化 - 对于搜索结果来说基本瞬间完成。

#### 包装器

首先，我们将缓存存储在 `/tmp/file_cache` 中。这让我们可以通过简单删除目录（运行 `rm -rf /tmp/file_cache`）来删除缓存。我们还可以使用 `rm -rf /tmp/file_cache/search_codebase*` 有选择地移除函数调用。

```
def  wrapper(*args,  **kwargs):
 cache_dir =  "/tmp/file_cache"
 os.makedirs(cache_dir, exist_ok=True)
```

然后我们可以创建一个缓存键。

#### 缓存键的创建 / 未命中条件

我们还有另一个问题 - 我们希望在两种情况下错过我们的缓存：

1.  函数的参数变化由 `recursive_hash` 处理

1.  代码更改

为了处理第 2 个问题，我们使用 `inspect.getsource(func)` 将函数的源代码添加到哈希中，从而在代码更改时正确地错过缓存。

```
 func_source_code_hash =  hash_code(inspect.getsource(func))
```

```
 args_names = func.__code__.co_varnames[: func.__code__.co_argcount]
 args_dict =  dict(zip(args_names, args))

 # Remove ignored params
 kwargs_clone = kwargs.copy()
 for param in ignore_params:
 args_dict.pop(param, None)
 kwargs_clone.pop(param, None)

 # Create hash based on argument names, argument values, and function source code
 arg_hash = (
 recursive_hash(args_dict, ignore_params=ignore_params)
 +  recursive_hash(kwargs_clone, ignore_params=ignore_params)
 +  func_source_code_hash
 )
 cache_file = os.path.join(
 cache_dir, f"{func.__module__}_{func.__name__}_{arg_hash}.pickle"
 )
```

#### 缓存命中和未命中

最后，我们检查缓存键的存在性，并在缓存未命中的情况下写入缓存。

```
 try:
 # If cache exists, load and return it
 if os.path.exists(cache_file):
 if verbose:
 print("Used cache for function: "  + func.__name__)
 with  open(cache_file, "rb")  as f:
 return pickle.load(f)
 except  Exception:
 logger.info("Unpickling failed")

 # Otherwise, call the function and save its result to the cache
 result =  func(*args, **kwargs)
 try:
 with  open(cache_file, "wb")  as f:
 pickle.dump(result, f)
 except  Exception  as e:
 logger.info(f"Pickling failed: {e}")
 return result
```

## 结论

我们希望这段代码对你有用。我们发现在调试 LLM 调用时，它能大大节省时间。我们很乐意听取你的反馈和贡献，网址是 [https://github.com/sweepai/sweep (在新标签页打开)](https://github.com/sweepai/sweep)!

</main>
