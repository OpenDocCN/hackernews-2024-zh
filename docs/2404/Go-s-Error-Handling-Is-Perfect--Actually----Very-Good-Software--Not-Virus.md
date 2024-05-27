<!--yml
category: 未分类
date: 2024-05-27 12:56:36
-->

# Go’s Error Handling Is Perfect, Actually :: Very Good Software, Not Virus

> 来源：[https://blog.verygoodsoftwarenotvirus.ru/posts/errors-in-go/](https://blog.verygoodsoftwarenotvirus.ru/posts/errors-in-go/)

Spend any amount of time in programming circles, and just as the sun rises and falls, you are certain to hear someone complain about error handling in Go. These complaints are, anecdotally, rarely well thought out suggestions on what error handling could or should be like in a language like Go, but often merely boil down to “I don’t like having to look at it”.

You’ll mainly see people complain about having to litter their apps with `if err != nil`, how they feel it makes the code verbose, and how they miss just writing ten very consequential lines one after the other in more dynamic languages like Python or Javascript.

Less often, you’ll see folks bemoan the lack of a stdlib-defined [Result type](https://doc.rust-lang.org/std/result/), which would have either a value or an error, like in Rust. Every now and then you’ll see someone bemoan how error handling works in highly concurrent Go programs, or how [some errors can be nil and not-nil simultaneously](https://stackoverflow.com/questions/53892508/golang-returning-nil-does-not-return-nil).

## In defense of `if err != nil`[⌗](#in-defense-of-if-err--nil)

I have a spicy hot take on the matter: Go’s error handling is not just fine, not just great, but actually perfect for 99% of programs. (I will readily admit the last thing about error nilness is annoying, but in the nearly decade of time I’ve spent writing Go, I can think of perhaps a handful of times where it actually reared its head. I also ascribe that issue to the subject of nil in Go, which is a whole other topic, this article is about errors.)

I regard the matter like this: do you want to know when something bad can happen in your program, or don’t you? I asked ChatGPT to `Please write some python code for me that goes through every row of a CSV file, performs a get request for the value in the first column, and parses the responses as a JSON map of strings to strings`:

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

There’s basically no error handling here, but there’s a ton that can go wrong. What happens if the file doesn’t exist, or is corrupted? What happens if you don’t have permissions to read it? What happens if the `GET` request fails? What happens if the response body isn’t valid JSON, or doesn’t match the expected shape? The answer, in the case of Python, is an exception gets thrown, and since there’s no code to catch it, it’s handled by the broader runtime, printing a stack trace.

```
Traceback (most recent call last):
  File "main.py", line 15, in <module>
    process_csv(csv_file)
  File "main.py", line 5, in process_csv
    with open(csv_file, 'r') as file:
         ^^^^^^^^^^^^^^^^^^^
FileNotFoundError: [Errno 2] No such file or directory: 'your_csv_file.csv' 
```

Note that for any sufficiently complex program that invokes many dependencies, this stack trace will be so far down the chain that you may not even see where you’re making the call that causes it.

I asked ChatGPT to write the same code, but in Go:

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

Wouldn’t you know it, we have all the aforementioned errors handled! If there’s something wrong with the file, that will get surfaced. If there’s something wrong with the `GET` request, that will be surfaced. If the response doesn’t contain valid JSON, that will get surfaced. Is there more error handling code in the Go version? Yes, because that’s how Go is idiomatically written.

The Python I posted above, while it certainly could be written a better way, doesn’t look meaningfully different from 90%+ of the Python I’ve ever had to work with professionally. My only major gripe with the Go variant is that it doesn’t check the length of row before accessing that index, but that’s ChatGPT for you.

There are folks who will say that the Go version is less readable than the Python version. For me, this depends on how you measure readability. If you measure it from the time you first see the code to when you understand what it’s trying to accomplish, I could entertain the suggestion that Python wins. If you measure it from the perspective of how long it takes to suss out what the different execution paths or outcomes could possibly be from a given chunk of code, I think Go wins.

Even if you didn’t know that file reads could fail, or network requests could fail, you would understand that both are possible after reading the Go code, but not from the Python code.

I’d rather see a billion `if err != nil` statements in my code than have an error occur that I cannot quickly and effectively diagnose because it comes with a bunch of unrelated noise.

## Result types in Go[⌗](#result-types-in-go)

I’ve seen the [RIIR](https://transitiontech.ca/random/RIIR) crew suggest that one of the things that would make Go tolerable to them would be some equivalent of the `Result` type in Rust. The `Result` type allows for you to return one value that can contain either the anticipated value, or an error, but not both. So you sort of get the chance to collapse the standard `res, err := someOperation()` pattern you see in a lot of Go into just `res := someOperation()`.

I asked ChatGPT to `Please write some rust code that returns a Result container with a string, have it check the current time and return an error if the unix timestamp is even or a positive result if the unix timestamp is odd`:

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

This is, frankly, neat, and not a bad idea. There’s an alternative universe where Go had generics from the jump and the `Result` type was implemented/utilized in the standard library, and it’s not the worst outcome I could think of. There [are libraries](https://github.com/MisterKaiou/go-functional) you can use now, post-generics, that do just this, and [an old and long-closed proposal](https://github.com/golang/go/issues/19991) to add it to Go specifically cites the Rust variant.

The only opposition I have to the Result type in Go is that we wouldn’t be able to make use of it in the standard library without either breaking backwards compatibility, writing `Result` variants of existing API calls (so `NewRequest`, `NewRequestWithContext`, and `NewRequestWithContextAndResult`), or issuing new `/v2` variants of existing packages (like the [recently-released `math/rand/v2` package](https://tip.golang.org/doc/go1.22#math_rand_v2)), which then means we’ll have some libraries and programs that use the old style with one return value, some with the new style, and many instances of confused programmers using the wrong one. It would be as close to a Go equivalent of the Python 2/3 transition debacle as I think we could manage.

I also don’t really think it meaningfully improves readability. Compare the above Rust code to the Go equivalent:

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

Our main function is 6 lines, compared to Rust’s 4\. I suppose that adds up over time and with a larger project, but I still just don’t think it’s the massive win for readability that some folks proclaim it to be.

## Conclusion[⌗](#conclusion)

None of this post was meant to denigrate Python, Rust, Javascript, or any other language, or its fans, or indeed anything at all. I just think a lot of the criticism around this particular element of the Go programming language is missing the forest for the trees.