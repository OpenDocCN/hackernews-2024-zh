<!--yml

category: 未分类

date: 2024-05-27 13:20:59

-->

# MaziyarPanahi/Meta-Llama-3-8B-Instruct-GGUF · OK llama 3 8b model is INSANE. Is almost as good as wizard 2 8x22b!

> 来源：[https://huggingface.co/MaziyarPanahi/Meta-Llama-3-8B-Instruct-GGUF/discussions/5](https://huggingface.co/MaziyarPanahi/Meta-Llama-3-8B-Instruct-GGUF/discussions/5)

非常有趣的结果！这些评估利用了LM Studio 0.2.20，利用其多模型会话功能执行所有提示。每个模型都使用最大GPU层次完全加载到RAM中，并且禁用了顺序提示以实现模型的并行执行。

**设备：** 苹果M3 Max（核心：4E+12P+40GPU）

**温度：** 0.8

**系统提示：** 你是一个乐于助人、聪明、善良且高效的AI助理。你始终尽力满足用户的请求。

# Prompt 1

```
Create 10 sentences that ends with a word "apple". Remember the word "apple" MUST be at the end. 
```

## 期望结果

```
10 sentences that end with the word "apple" 
```

### 264 tokens, 9.68 tok/sec

```
Here are 10 sentences that end with the word "apple":

1\. The farmer picked a juicy red apple.
2\. She took a bite of her crunchy apple.
3\. He found an old book hidden under his apple.
4\. The teacher gave each student a shiny new apple.
5\. The horse's mane looked like a tangled apple.
6\. The scent of fresh baked pies filled the air with apple.
7\. The athlete won first prize in the apple.
8\. The cat pounced on the toy mouse, chasing it around the apple.
9\. She carefully polished the antique mirror to reveal a perfect apple.
10\. As she walked through the orchard, she admired the rows of ripe apples.

I hope these sentences meet your requirements! 
```

### 241 tokens, 6.99 tok/sec

```
Here are 10 sentences that end with the word "apple":

1\. The farmer grew a juicy red apple.
2\. She took a bite of the crunchy green apple.
3\. The tree bore sweet and tangy apple.
4\. He picked up the shiny yellow apple.
5\. Her favorite snack was a fresh red apple.
6\. The basket was filled with plump and ripe apple.
7\. They harvested a bushel of crisp autumn apple.
8\. She carefully washed the organic Granny Smith apple.
9\. His lunch consisted of a sandwich and a slice of apple.
10\. The juice box contained a refreshing glass of apple. 
```

### 292 tokens, 3.84 tok/sec

```
Here are 10 sentences that end with the word "apple":

1\. The juicy fruit I bit into was a crunchy fresh apple.
2\. She carefully polished the shiny red wooden toy apple.
3\. After a long hike, he rewarded himself with a sweet crisp apple.
4\. The farmer's market displayed a colorful array of ripe autumn apple.
5\. In the still life painting, the artist placed a perfect red apple.
6\. The teacher used a real example to explain the concept of gravity apple.
7\. At the county fair, she won first prize for her delicious homemade apple.
8\. The healthy snack I packed in my lunchbox was a crunchy green apple.
9\. According to legend, the forbidden fruit in the Garden of Eden was an apple.
10\. The clever entrepreneur's business idea was as innovative as a shiny new apple.

Let me know if you need anything else! 
```

# Prompt 2

```
I have 10 apples. I find 3 gold coins in the bottom of a river. The river runs near a big city that has something to do with what I can spend the coins on. I then lose 4 apples but gain a gold coin. Three birds run into my path and drop 6 apples each. I play an online game and win 6 gold coins but I have to share them equally with my 2 teammates. I buy apples for all the coins I have. The price of an apple is 0.5 coins. How many apples do I have? And where is the river? 
```

## 期望结果

```
36 apples 
```

### 535 tokens, 8.13 tok/sec

```
Let's break down your transactions step by step:

1\. You start with 10 apples.
2\. You find 3 gold coins in the river, but this doesn't affect your apple count.
3\. You lose 4 apples, leaving you with 10 - 4 = 6 apples.
4\. You gain a gold coin, which is nice, but still no change to your apple count.
5\. Three birds drop 6 apples each, so that's 18 new apples (6 x 3). You now have 6 + 18 = 24 apples.
6\. You play an online game and win 6 gold coins, but you have to share them equally with your 2 teammates. That means you get 6 / 3 = 2 gold coins as your share.
7\. With the 5 gold coins (including the one you already had) and the new ones, you buy apples using the following formula: price of an apple = 0.5 coins. You can buy:

2 gold coins / 0.5 coins per apple = 4 apples
1 remaining gold coin / 0.5 coins per apple = 2 more apples

You now have a total of 24 + 4 + 2 = 30 apples.

As for the river, it runs near a big city that has something to do with what you can spend the coins on. Since you used your gold coins to buy apples, the connection is likely related to food or markets in the city. Perhaps the city is known for its bustling fruit market or delicious street food? 
```

### 566 tokens, 6.83 tok/sec

```
Let's break down your situation step by step:

1\. You start with 10 apples.
2\. You find 3 gold coins in the river, which doesn't change your apple count.
3\. You lose 4 apples, leaving you with 10 - 4 = 6 apples.
4\. You gain a gold coin, so now you have 3 + 1 = 4 gold coins.
5\. Three birds drop 6 apples each, adding 18 apples to your total: 6 (initial) + 18 = 24 apples.
6\. You play an online game and win 6 gold coins, but you share them equally with your 2 teammates. That means you get 6 / 3 = 2 gold coins.
7\. With the 4 original gold coins and the 2 new ones, you now have a total of 4 + 2 = 6 gold coins.
8\. You use all your gold coins to buy apples at a price of 0.5 coins per apple. Since each coin can buy 1/0.5 = 2 apples, you can buy:

6 gold coins * 2 apples/gold coin = 12 additional apples

Adding these to the 24 apples you already had, your total number of apples is now:

24 + 12 = 36 apples

As for where the river is located, it runs near a big city that has something to do with what you can spend the coins on. It seems likely that this city has a market or shops where you can buy apples using gold coins.

So, in summary, you now have 36 apples and the river is located near a big city where you can use your gold coins to buy more apples! 
```

### 526 tokens, 3.64 tok/sec

```
What a thrilling adventure! Let's break it down step by step.

Initially, you have 10 apples.

You find 3 gold coins in the bottom of a river that runs near a big city. The city is likely related to what you can spend the coins on, but we'll come back to that later.

Then, you lose 4 apples, leaving you with 10 - 4 = 6 apples. However, you gain another gold coin, making your total gold coins 3 + 1 = 4.

Three birds run into your path and drop 6 apples each, so you gain 3 x 6 = 18 apples. You now have 6 (initial apples) + 18 (new apples) = 24 apples.

You play an online game and win 6 gold coins, but you have to share them equally with your 2 teammates. So, you get 6 / 3 = 2 new gold coins, making your total gold coins 4 + 2 = 6.

Now, you decide to buy apples with all the gold coins you have. Since the price of an apple is 0.5 coins, you can buy:

6 gold coins / 0.5 coins per apple = 12 apples

Adding these new apples to your existing ones, you now have:
24 (initial apples) + 12 (new apples) = 36 apples

As for the river's location, it runs near a big city where you can likely spend your gold coins on various goods and services. 
```

# Prompt 3

```
If I have 6 apples and eat seven of them, how many are left and why? 
```

## 期望结果

```
Some combination of you can't eat more than you have or 0 apples left. 
```

### 226 tokens, 10.79 tok/sec

```
A delicious math problem!

Let's break it down:

You start with 6 apples.

You eat 7 apples, but you only had 6 apples to begin with. Eating 7 apples is not possible since you don't have that many.

So, the correct answer is: You still have 6 apples left because you didn't actually eat any of them!

However, if you want to imagine a scenario where you somehow magically acquire an extra apple and then eat 7, in that case, you would be left with... -1 apple! But let's not worry about that. 
```

### 268 tokens, 7.98 tok/sec

```
A delicious math problem!

Let's break it down:

You start with 6 apples.

You eat 7 apples.

The issue is that you're trying to eat 1 apple more than you have. You only have 6 apples, but you want to eat 7!

So, the answer is: You can't eat 7 apples because you don't have enough! You'll be left with... 0 apples!

But if you want to know how many apples are left after eating some of them, we can count the ones you did eat:

Let's say you ate 4 out of your 6 apples. That means you'd be left with:

6 - 4 = 2

So, in this case, you'd have 2 apples left.

Does that make sense? 
```

### 171 tokens, 3.22 tok/sec

```
I see what you're getting at! This is a classic lateral thinking puzzle.

The answer is: there are still 6 apples left!

Why? Because it's impossible to eat 7 apples if you only have 6\. You can't eat something that doesn't exist, so the number of apples remains unchanged. 
```

# Prompt 4

```
Two years ago, I was three times as old as my brother was. In three years, I will be twice as old as my brother. How old is each of us now? 
```

## 期望结果

```
You are 17 years old and your brother is 7 years old. 
```

### 626 tokens, 9.13 tok/sec

```
Let's break this down step by step!

Two years ago, you were 3 times as old as your brother.

Let's say your brother's age two years ago was x.

Then, your age two years ago would be 3x.

Now, let's think about the present. Since we're considering ages two years ago, that means we need to add 2 to both of our ages to get our current ages.

So, your current age is 3x + 2.
And your brother's current age is x + 2.

Now, let's consider what will happen in three years. Your current age plus 3 will be your age in three years.

Similarly, your brother's current age plus 3 will be his age in three years.

The problem states that in three years, you'll be twice as old as your brother. So, we can set up the equation:

(3x + 2) + 3 = 2(x + 2) + 3

Simplify the equation by combining like terms:

3x + 5 = 2x + 5

Subtract 5 from both sides to get:

3x = 2x

Subtract 2x from both sides to get:

x = 0

Wait, what?! This means your brother was zero years old two years ago! But that's not possible.

Let's re-examine the problem. Maybe there's a different solution.

Recall that we added 2 to our ages to get to the present. If x is really small, it might be easier to work with. Let's try a simpler approach:

Two years ago, you were 3 times as old as your brother.

Let's say your brother's age two years ago was 6\. (This is not necessarily accurate; we're just using a simple number for demonstration purposes.)

Then, your age two years ago would be 18.

Now, let's add 2 to both of our ages to get our current ages:

Your current age: 20
Brother's current age: 8

In three years, you'll be twice as old as your brother:

(20 + 3) = 23
(8 + 3) = 11

So, in three years, you will indeed be twice as old as your brother.

To answer the original question:

You are currently 20 years old.
Your brother is currently 8 years old. 
```

### 397 tokens, 6.48 tok/sec

```
Let's break this down step by step!

Two years ago, you were 3 times as old as your brother.

Let's say your brother's age two years ago was x.

Then, your age two years ago would be 3x.

Now, let's focus on the present. Since we're looking at ages now, we'll add 2 to both ages:

Your current age is 3x + 2.
Brother's current age is x + 2.

In three years, you will be twice as old as your brother.

So, the equation becomes:

(3x + 2) + 3 = 2(x + 2) + 3

Combine like terms:

3x + 5 = 2x + 5

Subtract 5 from both sides:

3x = 2x

Subtract 2x from both sides:

x = 0

Wait, what? Brother's age two years ago was zero?

But that means your age two years ago would also be zero. However, you mentioned being three times as old as your brother, which doesn't add up.

This seems like a paradox! There might be some information missing or an incorrect assumption.

Could you please clarify or provide more context about this problem? I'd be happy to help you work through it again! 
```

### 471 tokens, 3.66 tok/sec

```
Let's break this down step by step!

Let your current age be A and your brother's current age be B.

Two years ago, you were (A - 2) years old, and your brother was (B - 2) years old. According to the problem, two years ago, you were three times as old as your brother:

A - 2 = 3(B - 2)

Simplifying the equation:

A - 2 = 3B - 6

A = 3B - 4 ... (Equation 1)

Now, let's look at the situation three years from now. You will be (A + 3) years old, and your brother will be (B + 3) years old. According to the problem, in three years, you will be twice as old as your brother:

A + 3 = 2(B + 3)

Simplifying the equation:

A + 3 = 2B + 6

A = 2B + 3 ... (Equation 2)

Now we have two equations and two variables. We can solve for A and B.

From Equation 1, we know:
A = 3B - 4

Substituting this into Equation 2:

3B - 4 = 2B + 3

Simplifying the equation:

B = 7

Now that we have B, we can find A:

A = 3B - 4
= 3(7) - 4
= 21 - 4
= 17

So, you are currently 17 years old, and your brother is 7 years old. 
```
