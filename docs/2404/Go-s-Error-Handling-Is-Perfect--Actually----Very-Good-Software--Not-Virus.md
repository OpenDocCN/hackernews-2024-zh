-   <!--yml

-   category: 未分类

-   date: 2024-05-27 12:56:36

-   -->

# -   Go的错误处理实际上是完美的 :: Very Good Software, Not Virus

> 来源：[https://blog.verygoodsoftwarenotvirus.ru/posts/errors-in-go/](https://blog.verygoodsoftwarenotvirus.ru/posts/errors-in-go/)

-   在编程圈子里花费任意时间，就像太阳升起和落下一样，你肯定会听到有人抱怨Go语言中的错误处理。这些抱怨，据传闻，很少有人深思熟虑地建议Go语言中的错误处理应该如何或者应该像什么，而常常仅仅 boil down to “我不喜欢看它”。

-   大部分人抱怨不得不在他们的应用程序中添加 `if err != nil` 的代码，认为这使得代码冗长，并且想念在像Python或Javascript这样更动态的语言中可以连续写下十行非常重要的代码。

-   较少见的是，您会看到人们抱怨Go语言中缺少类似于Rust中的 stdlib-defined [Result type](https://doc.rust-lang.org/std/result/)，它将具有值或错误，或者如何在高度并发的Go程序中工作时错误处理，或者 [一些错误可以同时是nil和非nil](https://stackoverflow.com/questions/53892508/golang-returning-nil-does-not-return-nil)。

## -   在捍卫 `if err != nil`[⌗](#in-defense-of-if-err--nil)

-   我对此有一个非常激进的看法：对于99%的程序来说，Go的错误处理不仅仅是好的，不仅仅是优秀的，而是完美的。（我很乐意承认关于错误nilness的问题很烦人，但是在我近十年的Go编程生涯中，我可以想到的情况可能仅有几次是它真正显现出来的。我也将这个问题归因于Go中的nil主题，这是一个完全不同的话题，本文是关于错误的。）

-   我认为事情就是这样的：你想知道你的程序什么时候会出现问题，还是不想知道？我要求ChatGPT `Please write some python code for me that goes through every row of a CSV file, performs a get request for the value in the first column, and parses the responses as a JSON map of strings to strings`:

```
import csv
import requests

def process_csv(csv_file):
    with open(csv_file, 'r') as file:
        csv_reader = csv.reader(file)
        next(csv_reader)
        for row in csv_reader:
            url = row[0]
            data = requests.get(url).json()
            print(data)

if __name__ == "__main__":
    csv_file = "your_csv_file.csv"
    process_csv(csv_file) 
```

-   这里基本上没有错误处理，但是可能会发生很多错误。如果文件不存在或损坏会发生什么？如果您没有权限读取它会发生什么？如果 `GET` 请求失败会发生什么？如果响应主体不是有效的JSON，或者不匹配预期的格式会发生什么？在Python的情况下，答案是会抛出一个异常，并且由于没有代码来捕获它，它将被更广泛的运行时处理，打印一个堆栈跟踪。

```
Traceback (most recent call last):
  File "main.py", line 15, in <module>
    process_csv(csv_file)
  File "main.py", line 5, in process_csv
    with open(csv_file, 'r') as file:
         ^^^^^^^^^^^^^^^^^^^
FileNotFoundError: [Errno 2] No such file or directory: 'your_csv_file.csv' 
```

-   注意，对于任何调用了许多依赖项的足够复杂的程序，这个堆栈跟踪将如此深入链条，以至于您甚至可能看不到您所发出的调用是什么导致了它。

-   我请求ChatGPT写同样的代码，但用Go语言：

```
package main

import (
	"encoding/csv"
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"os"
)

func fetchData(url string) (map[string]string, error) {
	response, err := http.Get(url)
	if err != nil {
		return nil, err
	}
	defer response.Body.Close()

	var data map[string]string
	if err := json.NewDecoder(response.Body).Decode(&data); err != nil {
		return nil, err
	}

	return data, nil
}

func processCSV(csvFile string) error {
	file, err := os.Open(csvFile)
	if err != nil {
		return err
	}
	defer file.Close()

	reader := csv.NewReader(file)
	if _, err = reader.Read(); err != nil && err.Error() != "EOF" {
		return err
	}

	for {
		row, err := reader.Read()
		if err != nil {
			if err.Error() == io.EOF {
				break
			}
			return err
		}

		url := row[0]
		data, err := fetchData(url)
		if err != nil {
			fmt.Printf("Failed to fetch data from %s: %v\n", url, err)
			continue
		}

		fmt.Printf("Data from %s:\n", url)
		for key, value := range data {
			fmt.Printf("%s: %s\n", key, value)
		}

		fmt.Println()
	}

	return nil
}

func main() {
	csvFile := "your_csv_file.csv"
	if err := processCSV(csvFile); err != nil {
		fmt.Printf("Error processing CSV: %v\n", err)
	}
} 
```

看起来，我们已经处理了所有上述的错误！如果文件有问题，那就会显现出来。如果`GET`请求有问题，那也会显现出来。如果响应不包含有效的JSON，那也会显现出来。Go版本中是否有更多的错误处理代码？是的，因为Go通常是这样写的。

我上面发布的Python代码，虽然肯定可以写得更好，但在我职业生涯中与之相比并没有明显的不同。我对Go变体唯一的主要抱怨是它在访问索引之前没有检查行的长度，但这就是ChatGPT为你展示的内容。

有人会说Go版本比Python版本更不易阅读。对我来说，这取决于你如何衡量可读性。如果从第一次看到代码到理解其尝试完成任务所需的时间来衡量，我可以接受Python赢的建议。如果从如何迅速分辨给定代码块可能的不同执行路径或结果的角度来衡量，我认为Go胜出。

即使你不知道文件读取可能会失败，或者网络请求可能会失败，你读过Go代码后就会明白这两者都是可能的，但从Python代码中就不会明白。

我宁愿在我的代码中看到十亿次`if err != nil`的语句，也不愿意因为一堆无关的噪音而不能快速有效地诊断错误。

## Go中的Result类型[⌗](#result-types-in-go)

我见过[RIIR](https://transitiontech.ca/random/RIIR)团队建议，使Go对他们更可接受的一件事情之一是在Go中有类似Rust中`Result`类型的等价物。`Result`类型允许你返回一个值，可以包含预期的值，或者一个错误，但不能同时包含两者。因此，你可以将在很多Go代码中看到的标准`res, err := someOperation()`模式简化为`res := someOperation()`。

我请求ChatGPT，请写一些Rust代码，返回一个带有字符串的Result容器，检查当前时间并在Unix时间戳为偶数时返回错误，或在Unix时间戳为奇数时返回正结果：

```
use std::time::{SystemTime, UNIX_EPOCH};

fn check_current_time() -> Result<String, String> {
    let current_time = SystemTime::now().duration_since(UNIX_EPOCH).unwrap().as_secs();

    if current_time % 2 == 0 {
        Err("Even timestamp".to_string())
    } else {
        Ok("Odd timestamp".to_string())
    }
}

fn main() {
    match check_current_time() {
        Ok(msg) => println!("Result: {}", msg),
        Err(err) => println!("Error: {}", err),
    }
} 
```

坦率地说，这很整洁，而且不是一个坏主意。有一个另类的宇宙，在那里Go从一开始就有泛型，并且`Result`类型在标准库中得到了实现和利用，这并不是我能想到的最糟糕的结果。现在可以使用一些后生成泛型的库来执行此操作，这与Rust的变体具有相似性，并且[一个旧的并长期关闭的提案](https://github.com/golang/go/issues/19991)专门将其添加到Go中。

我对 Go 中的 Result 类型唯一的反对意见是，如果不破坏向后兼容性，我们就不能在标准库中使用它，要么写出现有 API 调用的 `Result` 变体（比如 `NewRequest`、`NewRequestWithContext` 和 `NewRequestWithContextAndResult`），要么发布新的 `/v2` 变体的现有包（比如 [最近发布的 `math/rand/v2` 包](https://tip.golang.org/doc/go1.22#math_rand_v2)），这意味着我们将有一些库和程序使用旧风格的一个返回值，一些使用新风格的，还会有许多混淆的程序员使用错误的风格。这将是我认为我们能够管理的最接近 Go 等价于 Python 2/3 过渡灾难的事件。

我也不认为这真正地提升了可读性。将上述 Rust 代码与 Go 等效代码进行比较：

```
package main

import (
	"errors"
	"fmt"
	"time"
)

func checkCurrentTime() (string, error) {
	if time.Now().Unix()%2 == 0 {
		return "", errors.New("even timestamp")
	}
	return time.Now().Format(time.Kitchen), nil
}

func main() {
	result, err := checkCurrentTime()
	if err != nil {
		fmt.Println(err)
	} else {
		fmt.Println(result)
	}
} 
```

我们的主函数有 6 行，而 Rust 只有 4 行。我想随着时间和项目规模的增长，这些都会累积起来，但我仍然认为这并不像一些人宣称的那样对可读性带来巨大的提升。

## 结论[⌗](#conclusion)

本文并不是要贬低 Python、Rust、Javascript 或任何其他语言，也不是它们的粉丝，或者实际上任何事情。我只是觉得，关于 Go 编程语言特定元素的许多批评都是把大局忘了。
