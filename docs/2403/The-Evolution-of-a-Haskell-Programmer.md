<!--yml

类别：未分类

日期：2024-05-27 14:40:35

-->

# 一个Haskell程序员的演变

> 来源：[https://pages.cpsc.ucalgary.ca/~robin/class/449/Evolution.htm](https://pages.cpsc.ucalgary.ca/~robin/class/449/Evolution.htm)

# 一个<it>Haskell</it>程序员的演变

Fritz Ruehr，Willamette大学

### 大一Haskell程序员

````
fac n = if n == 0 
           then 1
           else n * fac (n-1)

````

### MIT的大二Haskell程序员

（大一时学习Scheme）

````
fac = (\(n) ->
        (if ((==) n 0)
            then 1
            else ((*) n (fac ((-) n 1)))))

````

### 初级Haskell程序员

（初学Peano数）

````
fac  0    =  1
fac (n+1) = (n+1) * fac n

````

### 另一位初级Haskell程序员

（读到n+k模式是Haskell的一个令人讨厌的部分

[[1]](https://willamette.edu/~fruehr/haskell/evolution.html#references)

并参加了Ban n+k模式运动

[[2]](https://willamette.edu/~fruehr/haskell/evolution.html#references)

）

````
fac 0 = 1
fac n = n * fac (n-1)

````

### 资深Haskell程序员

（投票支持~~尼克松~~ ~~布坎南~~ 布什倾向右派）

````
fac n = foldr (*) 1 [1..n]

````

### 另一位资深Haskell程序员

（投票支持~~麦高文~~ ~~比亚弗拉~~ 纳德倾向左派）

````
fac n = foldl (*) 1 [1..n]

````

### 另一位资深Haskell程序员

（曾经倾向极右，后来又回到了左派！）

````
-- using foldr to simulate foldl

fac n = foldr (\x g n -> g (x*n)) id [1..n] 1

````

### 缓存记忆的Haskell程序员

（每天服用银杏叶提取物）

````
facs = scanl (*) 1 [1..]

fac n = facs !! n

````

### ~~毫无意义~~ *（啊咳）无点自由* Haskell程序员

（在牛津学习）

````
fac = foldr (*) 1 . enumFromTo 1

````

### 迭代Haskell程序员

（前Pascal程序员）

````
fac n = result (for init next done)
        where init = (0,1)
              next   (i,m) = (i+1, m * (i+1))
              done   (i,_) = i==n
              result (_,m) = m

for i n d = until d n i

````

### 迭代单行Haskell程序员

（曾是APL和C程序员）

````
fac n = snd (until ((>n) . fst) (\(i,m) -> (i+1, i*m)) (1,1))

````

### 累积Haskell程序员

（逐渐达到快速高潮）

````
facAcc a 0 = a
facAcc a n = facAcc (n*a) (n-1)

fac = facAcc 1

````

### 过渡传递的Haskell程序员

（早年饲养兔子，后来搬到新泽西州）

````
facCps k 0 = k 1
facCps k n = facCps (k . (n *)) (n-1)

fac = facCps id

````

### 童子军Haskell程序员

（喜欢打结；总是恭敬的，他

属于

*最小不动点教会* [[8]](https://willamette.edu/~fruehr/haskell/evolution.html#references)

）

````
y f = f (y f)

fac = y (\f n -> if (n==0) then 1 else n * f (n-1))

````

### 组合Haskell程序员

（避开变量，即使不是混淆；

所有这些吹捧只是一种阶段，尽管它很少会阻碍）

````
s f g x = f x (g x)

k x y   = x

b f g x = f (g x)

c f g x = f x g

y f     = f (y f)

cond p f g x = if p x then f x else g x

fac  = y (b (cond ((==) 0) (k 1)) (b (s (*)) (c b pred)))

````

### 列编码Haskell程序员

（更喜欢用一元法计数）

````
arb = ()    -- "undefined" is also a good RHS, as is "arb" :)

listenc n = replicate n arb
listprj f = length . f . listenc

listprod xs ys = [ i (x,y) | x<-xs, y<-ys ]
                 where i _ = arb

facl []         = listenc  1
facl n@(_:pred) = listprod n (facl pred)

fac = listprj facl

````

### 解释性Haskell程序员

（从未遇到他不喜欢的语言）

````
-- a dynamically-typed term language

data Term = Occ Var
          | Use Prim
          | Lit Integer
          | App Term Term
          | Abs Var  Term
          | Rec Var  Term

type Var  = String
type Prim = String

-- a domain of values, including functions

data Value = Num  Integer
           | Bool Bool
           | Fun (Value -> Value)

instance Show Value where
  show (Num  n) = show n
  show (Bool b) = show b
  show (Fun  _) = ""

prjFun (Fun f) = f
prjFun  _      = error "bad function value"

prjNum (Num n) = n
prjNum  _      = error "bad numeric value"

prjBool (Bool b) = b
prjBool  _       = error "bad boolean value"

binOp inj f = Fun (\i -> (Fun (\j -> inj (f (prjNum i) (prjNum j)))))

-- environments mapping variables to values

type Env = [(Var, Value)]

getval x env =  case lookup x env of
                  Just v  -> v
                  Nothing -> error ("no value for " ++ x)

-- an environment-based evaluation function

eval env (Occ x) = getval x env
eval env (Use c) = getval c prims
eval env (Lit k) = Num k
eval env (App m n) = prjFun (eval env m) (eval env n)
eval env (Abs x m) = Fun  (\v -> eval ((x,v) : env) m)
eval env (Rec x m) = f where f = eval ((x,f) : env) m

-- a (fixed) "environment" of language primitives

times = binOp Num  (*)
minus = binOp Num  (-)
equal = binOp Bool (==)
cond  = Fun (\b -> Fun (\x -> Fun (\y -> if (prjBool b) then x else y)))

prims = [ ("*", times), ("-", minus), ("==", equal), ("if", cond) ]

-- a term representing factorial and a "wrapper" for evaluation

facTerm = Rec "f" (Abs "n" 
              (App (App (App (Use "if")
                   (App (App (Use "==") (Occ "n")) (Lit 0))) (Lit 1))
                   (App (App (Use "*")  (Occ "n"))
                        (App (Occ "f")  
                             (App (App (Use "-") (Occ "n")) (Lit 1))))))

fac n = prjNum (eval [] (App facTerm (Lit n))) 
````

### 静态Haskell程序员

（他做事有条理，他有那个fundep Jones！

在Thomas Hallgren的“与函数依赖玩耍”之后

[[7]](https://willamette.edu/~fruehr/haskell/evolution.html#references)

)

````
-- static Peano constructors and numerals

data Zero
data Succ n

type One   = Succ Zero
type Two   = Succ One
type Three = Succ Two
type Four  = Succ Three

-- dynamic representatives for static Peanos

zero  = undefined :: Zero
one   = undefined :: One
two   = undefined :: Two
three = undefined :: Three
four  = undefined :: Four

-- addition, a la Prolog

class Add a b c | a b -> c where
  add :: a -> b -> c

instance              Add  Zero    b  b
instance Add a b c => Add (Succ a) b (Succ c)

-- multiplication, a la Prolog

class Mul a b c | a b -> c where
  mul :: a -> b -> c

instance                           Mul  Zero    b Zero
instance (Mul a b c, Add b c d) => Mul (Succ a) b d

-- factorial, a la Prolog

class Fac a b | a -> b where
  fac :: a -> b

instance                                Fac  Zero    One
instance (Fac n k, Mul (Succ n) k m) => Fac (Succ n) m

-- try, for "instance" (sorry):
-- 
--     :t fac four

````

### 开始研究生Haskell程序员

（研究生教育倾向于使人摆脱琐碎的担忧

关于，例如，基于硬件的整数的效率）

````
-- the natural numbers, a la Peano

data Nat = Zero | Succ Nat

-- iteration and some applications

iter z s  Zero    = z
iter z s (Succ n) = s (iter z s n)

plus n = iter n     Succ
mult n = iter Zero (plus n)

-- primitive recursion

primrec z s  Zero    = z
primrec z s (Succ n) = s n (primrec z s n)

-- two versions of factorial

fac  = snd . iter (one, one) (\(a,b) -> (Succ a, mult a b))
fac' = primrec one (mult . Succ)

-- for convenience and testing (try e.g. "fac five")

int = iter 0 (1+)

instance Show Nat where
  show = show . int

(zero : one : two : three : four : five : _) = iterate Succ Zero

````

### 折纸Haskell程序员

（总是从基本的Bird fold开始）

````
-- (curried, list) fold and an application

fold c n []     = n
fold c n (x:xs) = c x (fold c n xs)

prod = fold (*) 1

-- (curried, boolean-based, list) unfold and an application

unfold p f g x = 
  if p x 
     then [] 
     else f x : unfold p f g (g x)

downfrom = unfold (==0) id pred

-- hylomorphisms, as-is or "unfolded" (ouch! sorry ...)

refold  c n p f g   = fold c n . unfold p f g

refold' c n p f g x = 
  if p x 
     then n 
     else c (f x) (refold' c n p f g (g x))

-- several versions of factorial, all (extensionally) equivalent

fac   = prod . downfrom
fac'  = refold  (*) 1 (==0) id pred
fac'' = refold' (*) 1 (==0) id pred

````

### 笛卡尔倾向的Haskell程序员

（更喜欢希腊食品，避免辛辣的印度食物；

受Lex Augusteijns排序态射的启发

[[3]](https://willamette.edu/~fruehr/haskell/evolution.html#references)

）

````
-- (product-based, list) catamorphisms and an application

cata (n,c) []     = n
cata (n,c) (x:xs) = c (x, cata (n,c) xs)

mult = uncurry (*)
prod = cata (1, mult)

-- (co-product-based, list) anamorphisms and an application

ana f = either (const []) (cons . pair (id, ana f)) . f

cons = uncurry (:)

downfrom = ana uncount

uncount 0 = Left  ()
uncount n = Right (n, n-1)

-- two variations on list hylomorphisms

hylo  f  g    = cata g . ana f

hylo' f (n,c) = either (const n) (c . pair (id, hylo' f (c,n))) . f

pair (f,g) (x,y) = (f x, g y)

-- several versions of factorial, all (extensionally) equivalent

fac   = prod . downfrom
fac'  = hylo  uncount (1, mult)
fac'' = hylo' uncount (1, mult)

````

### 哈斯克尔程序员博士

（吃了太多香蕉，眼睛凸出了，现在需要新的镜片！）

````
-- explicit type recursion based on functors

newtype Mu f = Mu (f (Mu f))  deriving Show

in      x  = Mu x
out (Mu x) = x

-- cata- and ana-morphisms, now for *arbitrary* (regular) base functors

cata phi = phi . fmap (cata phi) . out
ana  psi = in  . fmap (ana  psi) . psi

-- base functor and data type for natural numbers,
-- using a curried elimination operator

data N b = Zero | Succ b  deriving Show

instance Functor N where
  fmap f = nelim Zero (Succ . f)

nelim z s  Zero    = z
nelim z s (Succ n) = s n

type Nat = Mu N

-- conversion to internal numbers, conveniences and applications

int = cata (nelim 0 (1+))

instance Show Nat where
  show = show . int

zero = in   Zero
suck = in . Succ       -- pardon my "French" (Prelude conflict)

plus n = cata (nelim n     suck   )
mult n = cata (nelim zero (plus n))

-- base functor and data type for lists

data L a b = Nil | Cons a b  deriving Show

instance Functor (L a) where
  fmap f = lelim Nil (\a b -> Cons a (f b))

lelim n c  Nil       = n
lelim n c (Cons a b) = c a b

type List a = Mu (L a)

-- conversion to internal lists, conveniences and applications

list = cata (lelim [] (:))

instance Show a => Show (List a) where
  show = show . list

prod = cata (lelim (suck zero) mult)

upto = ana (nelim Nil (diag (Cons . suck)) . out)

diag f x = f x x

fac = prod . upto

````

### 博士后Haskell程序员

（来自Uustalu，Vene和Pardos共同子回归方案

[[4]](https://willamette.edu/~fruehr/haskell/evolution.html#references)

）

````
-- explicit type recursion with functors and catamorphisms

newtype Mu f = In (f (Mu f))

unIn (In x) = x

cata phi = phi . fmap (cata phi) . unIn

-- base functor and data type for natural numbers,
-- using locally-defined "eliminators"

data N c = Z | S c

instance Functor N where
  fmap g  Z    = Z
  fmap g (S x) = S (g x)

type Nat = Mu N

zero   = In  Z
suck n = In (S n)

add m = cata phi where
  phi  Z    = m
  phi (S f) = suck f

mult m = cata phi where
  phi  Z    = zero
  phi (S f) = add m f

-- explicit products and their functorial action

data Prod e c = Pair c e

outl (Pair x y) = x
outr (Pair x y) = y

fork f g x = Pair (f x) (g x)

instance Functor (Prod e) where
  fmap g = fork (g . outl) outr

-- comonads, the categorical "opposite" of monads

class Functor n => Comonad n where
  extr :: n a -> a
  dupl :: n a -> n (n a)

instance Comonad (Prod e) where
  extr = outl
  dupl = fork id outr

-- generalized catamorphisms, zygomorphisms and paramorphisms

gcata :: (Functor f, Comonad n) =>
           (forall a. f (n a) -> n (f a))
             -> (f (n c) -> c) -> Mu f -> c

gcata dist phi = extr . cata (fmap phi . dist . fmap dupl)

zygo chi = gcata (fork (fmap outl) (chi . fmap outr))

para :: Functor f => (f (Prod (Mu f) c) -> c) -> Mu f -> c
para = zygo In

-- factorial, the *hard* way!

fac = para phi where
  phi  Z             = suck zero
  phi (S (Pair f n)) = mult f (suck n)

-- for convenience and testing

int = cata phi where
  phi  Z    = 0
  phi (S f) = 1 + f

instance Show (Mu N) where
  show = show . int

````

### 终身教授

（教授Haskell给新生）

````
fac n = product [1..n]

````

* * *

## 背景

2001年6月19日，在[OGI PacSoft周二早晨研讨会系列](http://www.cse.ogi.edu/~magnus/SeminarSeries/)上，[Iavor Diatchki](http://www.cse.ogi.edu/~diatchki/)介绍了Uustalu、Vene和Pardo的论文《从共模子代数到递归方案》[[4]](https://willamette.edu/~fruehr/haskell/evolution.html#references)。我参加了Iavors出色的演讲，并评论道，我觉得论文的结尾相当没趣：经过大量范畴化的努力和几个广义递归组合子的定义之后，主要的例子是阶乘和斐波那契函数。（当然，我自己也没有提供更好的例子，所以这有点不公平的挖苦。）

一段时间后，我找到了Iavors的“笑话”页面，其中包括一个有趣的部分称为[程序员的进化](http://www.cse.ogi.edu/~diatchki/jokes/programmer.html)，其中传统的命令式“Hello, world”程序通过几种变体从简单的开始发展到极其复杂的极端。稍加思考后，我发现阶乘函数是“Hello, world”最佳函数对应物。突然，灵感如泉涌般涌现，我知道我必须写出这些例子，达到（好吧，几乎）由Uustalu、Vene和Pardo提供的广义范畴化阶乘版本。

我想这应该是你必须称之为小众幽默。

**附言2：** 我已将所有代码放入[更好格式的文本文件](https://willamette.edu/~fruehr/haskell/evolution.hs)，供那些可能想尝试不同变体的人使用（您也可以直接从浏览器中剪切和粘贴部分）。

**附言：** 正如上文所述，Iavor并非《程序员的进化》的原始作者。快速的网络搜索表明，有数千份副本流传，而且（未署名地）出现在1995年以来的幽默新闻组中。但我怀疑某个版本的起源可能比那更早。当然，如果有人知道谁是原始作者，请告诉我，以便我在这里为他们署名。

* * *

## 但说真的，朋友们，……

更严肃地说，我认为笑话的基本思想（主题的连续变化，复杂度逐步增加）不仅可以起到幽默的作用，而且可以在教学上发挥良好的作用。为此，对于那些可能不熟悉以上所有观念的人，我对这些变化提供以下评论：

[第一个版本](https://willamette.edu/~fruehr/haskell/evolution.html#freshman)（带有条件的直接递归）可能对各类程序员来说都很熟悉；LISP和Scheme的粉丝会发现[第二个版本](https://willamette.edu/~fruehr/haskell/evolution.html#freshman)尤其易读，尽管lambda的拼写有些奇怪，而且没有定义（或者defun）。使用[模式](https://willamette.edu/~fruehr/haskell/evolution.html#junior)可能只是视角略微的转变，但除了反映数学符号外，模式还鼓励将数据类型视为初始代数（或归纳定义）。

使用更多结构递归组合子（如[foldr](https://willamette.edu/~fruehr/haskell/evolution.html#senior-right)和[foldl](https://willamette.edu/~fruehr/haskell/evolution.html#senior-left)）完全符合函数式编程的精神：这些高阶函数将递归定义的不同实例的共同细节抽象出来，通过函数参数恢复具体细节。[点自由](https://willamette.edu/~fruehr/haskell/evolution.html#points-free)风格（定义函数而无需显式引用其形式参数）可能非常引人入胜，但有时也会被滥用；这里的目的是预示稍后更为强调代数变体中的类似用法。

[累积参数](https://willamette.edu/~fruehr/haskell/evolution.html#accumulating)版本展示了一种加速函数式代码的传统技术。这是这里第二快的实现，至少按照Hugs报告的减少次数来衡量，[迭代](https://willamette.edu/~fruehr/haskell/evolution.html#iterative)版本排名第三。尽管后者有些违背函数式编程的精神，它们确实展示了作为表征语义或单子中使用的状态函数式模拟的风味。（在这里，单子显得非常缺乏；如果有人能贡献一些（渐进）示例，按照上面的开发精神，我将不胜感激。）[传递延续](https://willamette.edu/~fruehr/haskell/evolution.html#continuation)版本回忆起对控制的表征式解释（参考Steeles RABBIT编译器的Scheme和SML/NJ编译器的引用）。

[固定点版本](https://willamette.edu/~fruehr/haskell/evolution.html#boyscout)显示出我们可以通过通用的Y组合子孤立递归。[组合](https://willamette.edu/~fruehr/haskell/evolution.html#combinatory)版本提供了一种极端的无点式风格，灵感来源于组合逻辑，将对变量名的依赖隔离到少数几个组合子的定义中。当然，我们可以进一步定义自然数和布尔值，但请注意前驱函数将会有些难以适应（这是代数类型的一个很好的理由）。还要注意，我们不能在不遇到类型问题（主要是自我应用问题）的情况下，用其他组合子来定义Y组合子。有趣的是，这是所有实现中最快的，也许反映了在实现中使用的底层图还原机制。

[列表编码](https://willamette.edu/~fruehr/haskell/evolution.html#listencoding)版本利用了一个简单的观察：我们可以通过使用任意元素的列表来以一元方式计数，因此列表的长度编码了一个自然数。从某种意义上说，这个想法预示了基于递归类型定义Peanos自然数的后续版本，因为单位列表与自然数同构。这里唯一有趣的是，乘法（数值乘积）被看作是通过基数的组合（笛卡尔积）自然而然地产生。由于类型问题，很难像我们希望的那样直接表达这种对应关系：下面对listprod的定义将会由于出现检查/无限类型而破坏facl函数的定义。

> `listprod xs ys = [ (x,y) | x<-xs, y<-ys ]`

当然，我们也可以简化如下，但这样做只会模糊两种产品之间的关系：

> `listprod xs ys = [ arb | x<-xs, y<-ys ]`

[解释型](https://willamette.edu/~fruehr/haskell/evolution.html#interpretive)版本实现了一个足够丰富以表达阶乘的小型对象语言，并基于简单的环境模型实现了其解释器。沿这些线路的练习贯穿于弗里德曼、旺德和海恩斯的后半部分文本（[[6]](https://willamette.edu/~fruehr/haskell/evolution.html#references)），尽管在那里是用Scheme表达的。我们曾在[奥伯林](http://www.cs.oberlin.edu/)的学生中引起反感，当我们让他们在一个星期的实验室中实现十二个解释器时，逐步通过将真正的工作从元语言移动到解释器来暴露更多的实现细节。这个实现把很多事情留给了元语言，相当于他们一周中的周二或周三。勤奋的读者被邀请在类似于多态折叠和展开的Squiggol语言上实现一个编译器，目标是（并模拟）一个合适的范畴抽象机（参见[[9]](https://willamette.edu/~fruehr/haskell/evolution.html#references)），然后在该环境中实现阶乘（但如果这让你迟到吃午饭，请不要怪我…）。

[静态计算](https://willamette.edu/~fruehr/haskell/evolution.html#fundep)版本使用类型类和*功能依赖*在编译时促进计算（后者是马克·琼斯最近对Haskell 98标准的扩展，并在Hugs和GHC中可用）。同样类型的技术也可以用于编码更常见于依赖类型和多态编程的行为，因此是Haskell社区近期很感兴趣的主题。这里展示的代码基于托马斯·霍尔格伦的描述（见[[7]](https://willamette.edu/~fruehr/haskell/evolution.html#references)，扩展以包括阶乘。Prolog爱好者将发现这些定义特别易读，尽管有些有些“反向”。

第一个[研究生](https://willamette.edu/~fruehr/haskell/evolution.html#peano)版本更加严肃地对递归进行了定义，将自然数定义为递归代数数据类型，并突出了迭代和原始递归之间的区别。[折纸](https://willamette.edu/~fruehr/haskell/evolution.html#origami)和[笛卡尔](https://willamette.edu/~fruehr/haskell/evolution.html#cartesian)变体在这方面略有退步，因为它们回到了使用内部整数和列表类型。然而，它们在更熟悉的背景下引入了解扭曲和半解构的概念。

[博士](https://willamette.edu/~fruehr/haskell/evolution.html#categorical)示例在BMF/Squiggol的范畴风格中严肃地应用了（我们实际上可以更进一步，通过更直接地使用余积，从而消除一些对数据类型定义机制内部和的依赖）。

到达**巅峰之作**时，我们已经涵盖了大部分基本思想，并且（希望能够）更好地集中精力在它们的具体贡献上。使用了[共摄](https://willamette.edu/~fruehr/haskell/evolution.html#comonadic)版本的Uustalu，Vene和Pardo之后，是我认为这个函数在语言和Prelude定义方面最为清晰表达的**最终**版本，使用了Prelude定义的乘积函数和省略号表示法（这一定义至少可以追溯到David Turners的KRC*语言[[5]](https://willamette.edu/~fruehr/haskell/evolution.html#references)）。

安慰的是，我们可以放心地知道Prelude最终使用了一个递归组合子（foldl'，foldl的严格版本）来定义乘积。我想我们都可以希望看到Prelude为我们定义gcatamorphic、zygomorphic和paramorphic组合子的那一天，这样阶乘既可以方便地定义，*又*更有尊严 :)。

* * *

* *KRC可能是Research Software公司的商标，也可能不是。

但是你可以打赌你的甜蜜的Bippy，*Miranda *就是！

* * *

## 修订历史

+   *2010年代某个时间*：更正了*LFPP信条*的归属，这是Calvin Ostrum的贡献（并添加了一个照明版本的链接；请参见上文）。还添加了一个塞尔维亚克罗地亚语版本的链接，尽管后来有人告诉我，这个友好提供的服务是某种点击诱饵计划。我对这一切一窍不通：如果翻译本身出了什么问题，我想我可以将链接删掉。但或许整个事件只是表明我正在变成一个毫无头绪的老家伙，无法理解所有这些新潮的互联网骗局，这可能正在变得越来越真实。 （给任何结合了动态逻辑、指称语义和模糊逻辑以开发一个系统来探索命题随时间缓慢变为真实的人额外加分。我也能看到涉及变灰发的SORITES悖论的潜在应用……。）

+   *01年8月20日*：增加了基于小型对象语言环境模型的[解释性](https://willamette.edu/~fruehr/haskell/evolution.html#interpretive)版本（不，不是那种*对象*的意思……）。我在考虑重新排列示例的顺序，以便那些不属于主要开发路线的更长示例不会过多干扰。我还在[Haskell咖啡馆](http://haskell.org/mailman/listinfo/haskell-cafe)邮件列表上宣传了这个页面，并请求将链接添加到[Haskell幽默页面](http://www.haskell.org/humor)上。最后，我正在制作一个有一定原创研究价值的有趣新示例；关于这个的更多内容即将发布。

+   *01 年 08 月 14 日 (下午)*: 添加了 [combinatory](https://willamette.edu/~fruehr/haskell/evolution.html#combinatory) 版本，目前是由 Hugs 报告的减少数量最快的版本。

+   *01 年 08 月 14 日 (早晨)*：调整了 [大二/方案](https://willamette.edu/~fruehr/haskell/evolution.html#sophomore) 版本，使用了明确的 "lambda" (尽管在 Haskell 领域中我们的拼写有所不同)，并添加了 [fixed-point](https://willamette.edu/~fruehr/haskell/evolution.html#boyscout) 版本。

+   *01 年 08 月 10 日*: 添加了 [list-encoding](https://willamette.edu/~fruehr/haskell/evolution.html#listencoding) 和 [static computation](https://willamette.edu/~fruehr/haskell/evolution.html#fundep) 版本 (后者在类型检查期间使用类型类和功能依赖计算阶乘；这是从 Thomas Hallgrens 的 "Fun with Functional Dependencies" [[7]](https://willamette.edu/~fruehr/haskell/evolution.html#references) 中扩展的版本).

+   *01 年 08 月 1 日*: 添加了 [accumulating-parameter](https://willamette.edu/~fruehr/haskell/evolution.html#accumulating) 和 [continuation-passing](https://willamette.edu/~fruehr/haskell/evolution.html#continuation) 版本 (后者是从 Friedman, Wand 和 Haynes 的 "编程语言的基本要素" [[5]](https://willamette.edu/~fruehr/haskell/evolution.html#references) 中修改的转录).

+   *01 年 07 月 11 日*: 最初发布日期.

* * *

## 参考资料

1.  *从 nhc 中的亮点 - 一个高效的 Haskell 编译器,* Niklas Rjemo. 在 **FPCA 95 会议论文集** 中. ACM 出版社, 1995 (参见[CiteSeer](http://citeseer.nj.nec.com/nbib/5076458) 或 [Chalmers ftp 档案](ftp://ftp.cs.chalmers.se/pub/users/rojemo/fpca95.ps.gz)).

1.  *n+k 模式,* Lennart Augustsson. 发送至 Haskell 邮件列表的消息，1993 年 5 月 17 日 (参见[邮件列表档案](http://www.mail-archive.com/haskell@haskell.org/msg01261.html)).

1.  *排序态射,* Lex Augusteijn. 在 **高级函数编程** (LNCS 1608). Springer-Verlag, 1999 (参见[CiteSeer](http://citeseer.nj.nec.com/augusteijn99sorting.html)).

1.  *Comonad 中的递归方案,* T. Uustalu, V. Vene 和 A. Pardo. **北欧计算期刊**, 即将发表 (参见[Tarmo Uustalu 的论文页面](http://www.cs.ioc.ee/~tarmo/papers/)).

1.  *递归方程作为一种编程语言,* D. A. Turner. 在 **函数式编程及其应用** 中. 剑桥大学出版社, 1982.

1.  *编程语言的基本要素,* D. Friedman, M. Wand 和 C. Haynes. MIT 出版社和 McGraw-Hill, 1994.

1.  *功能依赖的乐趣,* Thomas Hallgren. **科学与计算机工程系联合冬季会议**, 瑞典 Chalmers 科技大学和 Gteborg 大学, Varberg, 2001 (可在[作者的网络档案](http://www.cs.chalmers.se/~hallgren/Papers/wm01.html)找到).

1.  *最小不动点教堂*，~~作者未知~~，作者为 Calvin Ostrum，1983 年 2 月。lambda 演算的一点幽默，传播于 1980 年代中期（至少我是那时看到的），可能来自于 comp.lang.functional 新闻组或类似的地方。（感谢 Ron，也许还有其他一些善良的人，提醒我正确的归属。显然我在某个时候创作了下面的[最小不动点教堂信条的彩绘手稿版本](http://www.willamette.edu/~fruehr/haskell/ChurchLFP.jpg)。）

1.  *分类组合子、顺序算法和函数式编程*，Pierre-Louis Curien 著。Springer Verlag（第二版），1993 年。

* * *
