<!--yml

category: 未分类

date: 2024-05-27 13:10:40

-->

# Grok-1.5 视觉预览

> 来源：[https://x.ai/blog/grok-1.5v](https://x.ai/blog/grok-1.5v)

##### 用户

你能将这段内容翻译成 Python 代码吗？

##### Grok

当然可以！你提供的流程图描述了一个简单的猜数字游戏，计算机生成一个随机数，用户需要猜测这个数。以下是表示流程图逻辑的 Python 代码：

```
1import random 2
3def  guess_number():  # Generate a random number between 1 and 10  4
5 target = random.randint(1,  10)  6
7  while  True:  8 guess =  int(input("Guess a number between 1 and 10: "))  9
10  if guess == target:  11  print("You won!")  12  break  13  else:  14  print("Wrong guess, try again!")  15
16if __name__ ==  "__main__":  17 guess_number()
```

此代码将按照流程图描述的方式运行猜数字游戏。用户将被提示输入他们的猜测，直到他们猜对为止。
