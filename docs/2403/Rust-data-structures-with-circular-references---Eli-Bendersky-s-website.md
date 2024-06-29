<!--yml

category: 未分类

date: 2024-05-29 12:41:15

-->

# Rust数据结构与循环引用 - Eli Bendersky的网站

> 来源：[https://eli.thegreenplace.net/2021/rust-data-structures-with-circular-references/](https://eli.thegreenplace.net/2021/rust-data-structures-with-circular-references/)

*2021年11月12日20:30* *标签 [Rust](https://eli.thegreenplace.net/tag/rust)* *为了实现其安全性保证，Rust编译器在整个程序中精确跟踪所有权和引用。这使得编写某些类型的数据结构变得具有挑战性；特别是具有循环引用的数据结构。

让我们从一个简单的二叉树开始：

```
struct Tree  { root: Option<Node>, } struct Node  { data: i32, left: Option<Box<Node>>, right: Option<Box<Node>>, } 
```

由于Rust编译器应能在编译时计算结构体的大小，`left`和`right`通常使用堆分配的`Box`。这些盒子被包装在`Option`中，因为节点的左子节点或右子节点可能为空。

现在假设我们想为每个节点添加一个*父链接*。这对于某些树结构很有用；例如，在[二叉搜索树](https://zh.wikipedia.org/wiki/%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91)（BST）中，父链接可用于高效地查找节点的后继节点。我们该如何做呢？

## “显而易见”的方法失败了

我们不能简单地向`Node`添加一个`parent: Option<Box<Node>>`字段，因为这将意味着节点拥有其父节点；显然这是错误的。事实上，我们最初的`Node`定义已经清楚地表明，父节点拥有其子节点，而不是反过来。

所以我们可能想要添加一个*引用*而不是一个父链接；父节点拥有其子节点，但子节点可能*指向*父节点。听起来合理；让我们试试：

```
struct Node  { data: i32, left: Option<Box<Node>>, right: Option<Box<Node>>, parent: Option<&Node>, } 
```

Rust将拒绝编译此代码，并要求显式生命周期参数。当我们将引用存储在结构字段中时，Rust希望知道此引用的生命周期如何与结构本身的生命周期相关联。公平地说，我们可以这样做：

```
struct Tree<'a>  { root: Option<Node<'a>>, } struct Node<'a>  { data: i32, left: Option<Box<Node<'a>>>, right: Option<Box<Node<'a>>>, parent: Option<&'a  Node<'a>>, } 
```

现在生命周期是明确的：我们告诉编译器，`parent`引用的生命周期与`Node`本身相同。这个结构定义将会编译通过，但编写实际操作它的代码将很快与借用检查器发生冲突。考虑一下插入新子节点到当前节点的代码；要改变当前节点，必须在范围内拥有可变引用。与此同时，新子节点的父链接是指向该节点的引用。借用检查器不允许我们创建对已经有活跃可变引用的对象的引用；它也不允许我们在任何其他引用活动时修改对象。

作为练习，请尝试使用这个`Node`定义编写树的插入方法；你会很快遇到问题。

## 现在怎么办？

显然，“显而易见”的方法行不通。实际上，从第一原则考虑，它不应该。在本文中，我使用带有父链接的BST作为简单的案例研究，但还有更明显（也许更有用）的例子。考虑一个图数据结构；一个节点具有指向其他节点的边，两个节点可以轻松地相互指向。谁拥有谁？由于这个问题在一般情况下无法在编译时回答，这意味着我们不能仅仅使用普通的Rust引用来处理这些“指向”关系。我们需要稍微聪明一点。

多次解决这个问题后，Rust程序员已经确定了三种潜在的解决方案：

1.  将借用检查推迟到*运行时*，通过对`std::rc::Rc`到`std::cell::RefCell`的引用计数指针来实现。 

1.  集中所有权（例如，所有节点由 `Tree` 中的节点向量拥有），然后引用变成*句柄*（指向上述向量的索引）。

1.  使用原始指针和`unsafe`块。

在本文中，我将介绍每一种方法，应用于实现具有插入、删除、"获取后继"方法和中序遍历的合理功能完整BST。此文的完整代码可在 [此存储库](https://github.com/eliben/code-for-blog/tree/main/2021/rust-bst) 中找到；本文仅展示了每种方法的一些感兴趣的片段。有关带有注释和广泛测试的完整实现，请参阅代码存储库。

## 使用Rc和RefCell进行运行时借用检查

此方法使用了Rust标准库中的两种数据结构的组合：

+   `std::rc::Rc`是一个引用计数指针，提供对堆分配数据的共享所有权。多个`Rc`实例可以引用相同的数据；当所有引用消失时，堆分配将被丢弃。这与C++中的`shared_ptr`非常相似。

    `Rc`有一个*弱对等物*，`std::rc::Weak`；这表示对由其他`Rc`拥有的数据的弱指针。虽然我们可以通过`Weak`访问数据，但如果只剩下弱指针，分配将被丢弃。在C++中，这将是`weak_ptr`。

+   `std::cell::RefCell`是一个带有*动态*检查借用规则的可变内存位置。`RefCell`允许我们在堆数据周围获取和传递引用，而不受借用检查器的严格控制。但是，它仍然是安全的；`RefCell`在运行时强制执行所有相同的借用规则。

您可以在这里看到实现此方法的完整代码 [here](https://github.com/eliben/code-for-blog/blob/main/2021/rust-bst/src/rcrefcell.rs)。接下来，我将突出显示一些有趣的部分。

这是我们可以定义我们的BST数据结构的方法：

```
use  std::cell::RefCell; use  std::rc::{Rc,  Weak}; pub  struct Tree  { count: usize, root: Option<NodeLink>, } type NodeLink  =  Rc<RefCell<Node>>; #[derive(Debug)] struct Node  { data: i32, left: Option<NodeLink>, right: Option<NodeLink>, parent: Option<Weak<RefCell<Node>>>, } 
```

拥有的“链接”由`Option<Rc<RefCell<Node>>>`表示；非拥有的链接由`Option<Weak<RefCell<Node>>>`表示。让我们看一些代表性的代码示例：

```
/// Insert a new item into the tree; returns `true` if the insertion
/// happened, and `false` if the given data was already present in the
/// tree.
pub  fn insert(&mut  self,  data: i32)  -> bool { if  let  Some(root)  =  &self.root  { if  !self.insert_at(root,  data)  { return  false; } }  else  { self.root  =  Some(Node::new(data)); } self.count  +=  1; true } // Insert a new item into the subtree rooted at `atnode`.
fn insert_at(&self,  atnode: &NodeLink,  data: i32)  -> bool { let  mut  node  =  atnode.borrow_mut(); if  data  ==  node.data  { false }  else  if  data  <  node.data  { match  &node.left  { None  =>  { let  new_node  =  Node::new_with_parent(data,  atnode); node.left  =  Some(new_node); true } Some(lnode)  =>  self.insert_at(lnode,  data), } }  else  { match  &node.right  { None  =>  { let  new_node  =  Node::new_with_parent(data,  atnode); node.right  =  Some(new_node); true } Some(rnode)  =>  self.insert_at(rnode,  data), } } } 
```

简单起见，本文代码示例将根节点的操作分离为顶级函数/方法，然后调用一个递归方法在节点级别进行操作。在这种情况下，`insert_at`接受一个链接，并将新数据插入为该节点的子节点。它保留了BST不变性（较小的意味着左子节点，较大的意味着右子节点）。这里需要注意的有趣之处是起始处的`borrow_mut()`调用。它从`atnode`指向的`RefCell`获取了一个可变引用。但这不只是一个常规的Rust可变引用，如`&mut`；相反，它是一种特殊类型，称为`std::cell::RefMut`。这就是可变性魔法发生的地方 - 看不到`&mut`，但代码实际上可以改变底层数据。

再次强调，此代码是安全的；如果在先前的`RefMut`仍在作用域内时尝试对`RefCell`进行另一个`borrow_mut()`，将导致运行时恐慌。这种安全性在运行时得到了保证。

另一个有趣的例子是私有的`find_node`方法，它从某个节点开始查找并返回具有给定数据的节点。

```
/// Find the item in the tree; returns `true` iff the item is found.
pub  fn find(&self,  data: i32)  -> bool { self.root .as_ref() .map_or(false,  |root|  self.find_node(root,  data).is_some()) } fn find_node(&self,  fromnode: &NodeLink,  data: i32)  -> Option<NodeLink>  { let  node  =  fromnode.borrow(); if  node.data  ==  data  { Some(fromnode.clone()) }  else  if  data  <  node.data  { node.left .as_ref() .and_then(|lnode|  self.find_node(lnode,  data)) }  else  { node.right .as_ref() .and_then(|rnode|  self.find_node(rnode,  data)) } } 
```

起初的`.borrow()`调用是我们要求`RefCell`提供对内部数据的不可变引用的方式（自然地，在运行时这不能与任何可变引用同时存在）。当我们返回找到的节点时，我们`clone`了`Rc`，因为我们需要一个独立的共享所有者来持有节点。这让Rust能够保证，只要返回的`Rc`还活着，节点就不会被简单丢弃。

正如完整的代码示例所示，这种方法是可行的。对于经验不足的Rust程序员来说，确实需要大量的实践和耐心才能做得好。由于每个节点都包装在三个间接层次（`Option`、`Rc`和`RefCell`）中，编写代码可能有些棘手，因为在任何时候你都必须记住当前处于哪个间接层次。

这种方法的另一个缺点是，获取存储在树中数据的普通引用并不容易。正如你在上面的示例中看到的，顶级的`find`方法不返回节点或其内容，而只是一个布尔值。这并不好；例如，它使得`successor`方法的子优化不足。这里的问题是，使用`RefCell`时我们不能直接返回数据的常规`&`引用，因为`RefCell`必须在运行时跟踪所有借用。我们只能返回`std::cell::Ref`，但这会泄露实现细节。这不是致命的缺陷，只是在使用这些类型编写代码时要牢记的事情。

## 使用句柄作为向量中的节点引用

我们将讨论的第二种方法是`Tree`拥有其中创建的所有节点，使用简单的`Vec`。然后，所有节点引用都变成了"句柄" - 即这个向量中的索引。以下是数据结构：

```
pub  struct Tree  { // All the nodes are owned by the `nodes` vector. Throughout the code, a
  // NodeHandle value of 0 means "none".
  root: NodeHandle, nodes: Vec<Node>, count: usize, } type NodeHandle  =  usize; #[derive(Debug)] struct Node  { data: i32, left: NodeHandle, right: NodeHandle, parent: NodeHandle, } 
```

再次，完整代码在[Github上可用](https://github.com/eliben/code-for-blog/blob/main/2021/rust-bst/src/nodehandle.rs)；这里我将展示一些显著部分。这是插入：

```
/// Insert a new item into the tree; returns `true` if the insertion
/// happened, and `false` if the given data was already present in the
/// tree.
pub  fn insert(&mut  self,  data: i32)  -> bool { if  self.root  ==  0  { self.root  =  self.alloc_node(data,  0); }  else  if  !self.insert_at(self.root,  data)  { return  false; } self.count  +=  1; true } // Insert a new item into the subtree rooted at `atnode`.
fn insert_at(&mut  self,  atnode: NodeHandle,  data: i32)  -> bool { if  data  ==  self.nodes[atnode].data  { false }  else  if  data  <  self.nodes[atnode].data  { if  self.nodes[atnode].left  ==  0  { self.nodes[atnode].left  =  self.alloc_node(data,  atnode); true }  else  { self.insert_at(self.nodes[atnode].left,  data) } }  else  { if  self.nodes[atnode].right  ==  0  { self.nodes[atnode].right  =  self.alloc_node(data,  atnode); true }  else  { self.insert_at(self.nodes[atnode].right,  data) } } } 
```

其中`alloc_node`是：

```
// Allocates a new node in the tree and returns its handle.
fn alloc_node(&mut  self,  data: i32,  parent: NodeHandle)  -> NodeHandle  { self.nodes.push(Node::new_with_parent(data,  parent)); self.nodes.len()  -  1 } 
```

在编写带有`Option<Rc<RefCell<...>>>`的代码之后，这种处理方式感觉*非常简单*。没有间接层；一个句柄就是一个索引；对节点的引用就是一个句柄；句柄0表示“无”，就是这样。

这个版本也可能比链式版本更高效，因为它有远少于堆分配的内存和一个非常适合缓存的单一向量。

尽管如此，这里也有一些问题。

首先，我们需要自己保证一些安全性。虽然这种方法不会导致内存损坏、双重释放或访问已释放的指针，但它可能会导致运行时恐慌和其他问题，因为我们处理向量的“原始”索引。由于错误，这些索引可能会指向向量边界之外，或者指向错误的插槽等等。例如，没有任何防止我们在有“活动句柄”时修改插槽的机制。

另一个问题是删除树节点。目前，代码通过不在任何活动句柄中指向它来“删除”节点。这使得通过树的方法无法访问节点，但不会释放内存。事实上，这个BST实现从不释放任何东西：

```
// Replaces `node` with `r` in the tree, by setting `node`'s parent's
// left/right link to `node` with a link to `r`, and setting `r`'s parent
// link to `node`'s parent.
// Note that this code doesn't actually deallocate anything. It just
// makes self.nodes[node] unused (in the sense that nothing points to
// it).
fn replace_node(&mut  self,  node: NodeHandle,  r: NodeHandle)  { let  parent  =  self.nodes[node].parent; // Set the parent's appropriate link to `r` instead of `node`.
  if  parent  !=  0  { if  self.nodes[parent].left  ==  node  { self.nodes[parent].left  =  r; }  else  if  self.nodes[parent].right  ==  node  { self.nodes[parent].right  =  r; } }  else  { self.root  =  r; } // r's parent is now node's parent.
  if  r  !=  0  { self.nodes[r].parent  =  parent; } } 
```

显然，这对于实际应用是错误的。至少，通过创建一个未使用索引的“空闲列表”，可以改进此实现，当添加节点时可以重复使用。一个更雄心勃勃的方法可能是为节点实现一个完整的垃圾收集器。如果你想挑战自己，可以试试看 ;-)

## 使用原始指针和不安全块

第三种讨论的方法是使用*原始指针*和`unsafe`块来实现我们的BST。如果你来自C/C++背景，这种方法会感觉非常熟悉。完整代码在[这里](https://github.com/eliben/code-for-blog/blob/main/2021/rust-bst/src/unsafeall.rs)。

```
pub  struct Tree  { count: usize, root: *mut  Node, } #[derive(Debug)] struct Node  { data: i32, // Null pointer means "None" here; right.is_null() ==> no right child, etc.
  left: *mut  Node, right: *mut  Node, parent: *mut  Node, } 
```

节点链接变成`*mut Node`，这是一个可变`Node`的原始指针。使用原始指针与编写C代码非常相似，在分配、释放和通过这些指针访问数据时有一些特殊情况。让我们从分配开始；这里是`Node`的构造函数：

```
impl  Node  { fn new(data: i32)  -> *mut  Self  { Box::into_raw(Box::new(Self  { data, left: std::ptr::null_mut(), right: std::ptr::null_mut(), parent: std::ptr::null_mut(), })) } fn new_with_parent(data: i32,  parent: *mut  Node)  -> *mut  Self  { Box::into_raw(Box::new(Self  { data, left: std::ptr::null_mut(), right: std::ptr::null_mut(), parent, })) } } 
```

我找到的为原始指针分配新内存的最简单方法是使用`Box::into_raw`，只要记住从那时起我们负责释放这些内存（稍后详述）。

让我们看看插入是如何工作的：

```
/// Insert a new item into the tree; returns `true` if the insertion
/// happened, and `false` if the given data was already present in the
/// tree.
pub  fn insert(&mut  self,  data: i32)  -> bool { if  self.root.is_null()  { self.root  =  Node::new(data); }  else  { if  !insert_node(self.root,  data)  { return  false; } } self.count  +=  1; true } // Inserts `data` into a new node at the `node` subtree.
fn insert_node(node: *mut  Node,  data: i32)  -> bool { unsafe  { if  (*node).data  ==  data  { false }  else  if  data  <  (*node).data  { if  (*node).left.is_null()  { (*node).left  =  Node::new_with_parent(data,  node); true }  else  { insert_node((*node).left,  data) } }  else  { if  (*node).right.is_null()  { (*node).right  =  Node::new_with_parent(data,  node); true }  else  { insert_node((*node).right,  data) } } } } 
```

这里有趣的一点是`insert_node`体中包裹的`unsafe`块。这是必需的，因为此代码解引用原始指针。在Rust中，可以分配指针并将其传递给其他地方而无需特殊条款，但解引用则需要`unsafe`。

让我们看看如何删除节点的工作；这里是`replace_node`，它执行与我们在节点处理方法中看到的同名方法相同的任务：

```
// Replaces `node` with `r` in the tree, by setting `node`'s parent's
// left/right link to `node` with a link to `r`, and setting `r`'s parent
// link to the `node`'s parent. `node` cannot be null.
fn replace_node(&mut  self,  node: *mut  Node,  r: *mut  Node)  { unsafe  { let  parent  =  (*node).parent; if  parent.is_null()  { // Removing the root node.
  self.root  =  r; if  !r.is_null()  { (*r).parent  =  std::ptr::null_mut(); } }  else  { if  !r.is_null()  { (*r).parent  =  parent; } if  (*parent).left  ==  node  { (*parent).left  =  r; }  else  if  (*parent).right  ==  node  { (*parent).right  =  r; } } // node is unused now, so we can deallocate it by assigning it to
  // an owning Box that will be automatically dropped.
  Box::from_raw(node); } } 
```

通过使用`Box::from_raw`来释放堆数据的示例展示了我们如何通过原始指针来释放堆数据：这将创建一个`Box`，它接管数据的所有权；`Box`有一个析构函数，因此当它超出作用域时，它会实际释放它。

这引出了一个重要的点：我们现在必须小心释放`Tree`的内存。与之前的方法不同，由于我们`Tree`中唯一包含的是`root: *mut Node`，Rust不知道如何“drop”它。如果我们在不显式实现`Drop`特性的情况下运行我们的测试，将会出现内存泄漏。这里是一个简单的`Drop`实现来解决这个问题：

```
impl  Drop  for  Tree  { fn drop(&mut  self)  { // Probably not the most efficient way to destroy the whole tree, but
  // it's simple and it works :)
  while  !self.root.is_null()  { self.remove_node(self.root); } } } 
```

作为练习，尝试实现一个更高效的`Drop`！

使用原始指针编写代码感觉相当自然；虽然最终的行数计数相似，但原始指针需要的心理负担显著少于使用`Option<Rc<RefCell<Node>>>`。虽然我没有对其进行基准测试，但我猜测指针版本也更有效率；至少它避开了`RefCell`所做的动态借用检查。另一面是安全性的损失。使用`unsafe`版本时，我们可能会遇到所有传统的C内存错误。

## 结论

本文的目标是回顾在Rust中实现非平凡链式数据结构时可以采用的不同方法。它涵盖了三个安全级别的方法：

+   使用`Rc`和`RefCell`则完全安全。

+   内存安全但是更容易出现漏洞（例如别名借用）的是将整数句柄用作`Vec`。

+   完全使用原始指针是不安全的。

我认为这三种方法各有其优点，并且都是有用的。来自Rust标准库和一些流行的crate的经验证据表明，第三种方法——使用原始指针和`unsafe`——非常受欢迎。

感谢阅读！由于这是我第一篇非平凡的Rust文章，我特别感兴趣听取任何评论、反馈或建议。请随时给我发送[电子邮件](mailto:eliben@gmail.com)或在这篇文章被转发到的任何聚合网站上留下评论——我会不时关注这些的。

* * **
