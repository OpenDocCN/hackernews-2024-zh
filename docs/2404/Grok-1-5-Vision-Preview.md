<!--yml
category: 未分类
date: 2024-05-27 13:10:40
-->

# Grok-1.5 Vision Preview

> 来源：[https://x.ai/blog/grok-1.5v](https://x.ai/blog/grok-1.5v)

##### User

Can you translate this into Python code?

##### Grok

Certainly! The flowchart you’ve provided describes a simple guessing game where the computer generates a random number, and the user has to guess it. Here’s the Python code that represents the logic in the flowchart:

```
1import random 2
3def  guess_number():  # Generate a random number between 1 and 10  4
5 target = random.randint(1,  10)  6
7  while  True:  8 guess =  int(input("Guess a number between 1 and 10: "))  9
10  if guess == target:  11  print("You won!")  12  break  13  else:  14  print("Wrong guess, try again!")  15
16if __name__ ==  "__main__":  17 guess_number()
```

This code will run the guessing game as described in the flowchart. The user will be prompted to enter their guess until they guess the correct number.