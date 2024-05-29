<!--yml
category: 未分类
date: 2024-05-29 12:41:15
-->

# Rust data structures with circular references - Eli Bendersky's website

> 来源：[https://eli.thegreenplace.net/2021/rust-data-structures-with-circular-references/](https://eli.thegreenplace.net/2021/rust-data-structures-with-circular-references/)

*November 12, 2021 at 20:30* *Tags [Rust](https://eli.thegreenplace.net/tag/rust)* *To implement its safety guarantees, the Rust compiler keeps careful track of ownership and references throughout a program. This makes writing certain kinds of data structures challenging; in particular, data structures that have circular references.

Let's start with a simple binary tree:

```
struct Tree  { root: Option<Node>, } struct Node  { data: i32, left: Option<Box<Node>>, right: Option<Box<Node>>, } 
```

Since the Rust compiler should be able to calculate the size of a struct at compile-time, `left` and `right` typically use a heap-allocated `Box`. These boxes are wrapped in an `Option` because a node's left or right child might be empty.

Now suppose we want to add a *parent* link to every node. This is useful for certain tree structures; for example, in a [binary search tree](https://en.wikipedia.org/wiki/Binary_search_tree) (BST), a parent link can be used to efficiently find the successor of a node. How can we do that?

## The "obvious" approach fails

We can't just add a `parent: Option<Box<Node>>` field to `Node`, because that would imply that a node owns its parent; this is clearly wrong. In fact, our original `Node` definition already makes it clear that a parent owns its children, not the other way around.

So we probably want to add a *reference* instead; a parent owns its children, but a child may *refert to* a parent. That sounds right; let's try it:

```
struct Node  { data: i32, left: Option<Box<Node>>, right: Option<Box<Node>>, parent: Option<&Node>, } 
```

Rust will refuse to compile this, asking for an explicit lifetime parameter. When we store a reference in a struct field, Rust wants to know how the lifetime of this reference relates to the lifetime of the struct itself. Fair enough, we can do that:

```
struct Tree<'a>  { root: Option<Node<'a>>, } struct Node<'a>  { data: i32, left: Option<Box<Node<'a>>>, right: Option<Box<Node<'a>>>, parent: Option<&'a  Node<'a>>, } 
```

Now the lifetime is explicit: we tell the compiler that the lifetime of the `parent` reference is the same as the `Node` itself. This struct definition will compile, but writing actual code manipulating it will very quickly get into an altercation with the borrow checker. Consider the code that would insert a new child node into the current node; to mutate the current node, a mutable reference to it has to be in scope. At the same time, the new child's parent link is a reference to the node. The borrow checker won't let us create a reference to an object which already has a live mutable reference to it; it also won't let us mutate an object while any other reference to it is alive.

As an exercise, try to write an insertion method for the tree using this `Node` definition; you'll run into the problem fairly quickly.

## Now what?

Clearly, the "obvious" way doesn't work. And indeed, thinking about it from first principles, it shouldn't. In this post I'm using BSTs with parent links as a simple case study, but there are even more obvious (and perhaps more useful) examples. Consider a graph data structure; a node has edges pointing to other nodes, and two nodes can easily point to each other. Who owns whom? Since this question cannot be answered at compile-time in the general case, it means we can't just use plain Rust references for these "points to" relations. We need to be somewhat more clever.

Having tackled this issue over and over again, Rust programmers have settled on three potential solutions:

1.  Defer borrow checking to *run-time*, by using a reference-counted pointer (`std::rc::Rc`) to a `std::cell:RefCell`.
2.  Centralize the ownership (e.g. all nodes are owned by a vector of nodes in the `Tree`), and then references become *handles* (indices into the abovementioned vector).
3.  Use raw pointers and `unsafe` blocks.

In this post, I'll present each of these approaches, applied to implementing a reasonably feature-complete BST with insertion, deletion, a "get successor" method and inorder traversal. The full code for this post is available in [this repository](https://github.com/eliben/code-for-blog/tree/main/2021/rust-bst); the post only presents a few snippets of interest for each approach. Refer to the code repository for the complete implementations with comments and extensive tests.

## Run-time borrow checking with Rc and RefCell

This approach uses a combination of two data structures from the Rust standard library:

*   `std::rc::Rc` is a reference-counted pointer, providing shared ownership of heap-allocated data. Multiple instances of `Rc` can refer to the same data; when all references are gone, the heap allication is dropped . This is quite similar to a `shared_ptr` in C++.

    `Rc` has a *weak dual*, `std::rc::Weak`; this represents a weak pointer to data owned by some other `Rc`. While we can access the data through a `Weak`, if only weak pointers remain the allocation will be dropped. In C++ this would be a `weak_ptr`.

*   `std::cell::RefCell` is a mutable memory location with *dynamically* checked borrow rules. `RefCell` allows us to take and pass around references to heap data without the scrutiny of the borrow checker. However, it's still safe; all the same borrow rules are enforced by `RefCell` at run-time.

You can see the full code implementing this approach [here](https://github.com/eliben/code-for-blog/blob/main/2021/rust-bst/src/rcrefcell.rs). In what follows, I'll highlight some interesting parts.

This is how can define our BST data structure :

```
use  std::cell::RefCell; use  std::rc::{Rc,  Weak}; pub  struct Tree  { count: usize, root: Option<NodeLink>, } type NodeLink  =  Rc<RefCell<Node>>; #[derive(Debug)] struct Node  { data: i32, left: Option<NodeLink>, right: Option<NodeLink>, parent: Option<Weak<RefCell<Node>>>, } 
```

Owning "links" are represented by a `Option<Rc<RefCell<Node>>>`; non-owning links are represented by `Option<Weak<RefCell<Node>>>`. Let's look at some representative code samples:

```
/// Insert a new item into the tree; returns `true` if the insertion
/// happened, and `false` if the given data was already present in the
/// tree.
pub  fn insert(&mut  self,  data: i32)  -> bool { if  let  Some(root)  =  &self.root  { if  !self.insert_at(root,  data)  { return  false; } }  else  { self.root  =  Some(Node::new(data)); } self.count  +=  1; true } // Insert a new item into the subtree rooted at `atnode`.
fn insert_at(&self,  atnode: &NodeLink,  data: i32)  -> bool { let  mut  node  =  atnode.borrow_mut(); if  data  ==  node.data  { false }  else  if  data  <  node.data  { match  &node.left  { None  =>  { let  new_node  =  Node::new_with_parent(data,  atnode); node.left  =  Some(new_node); true } Some(lnode)  =>  self.insert_at(lnode,  data), } }  else  { match  &node.right  { None  =>  { let  new_node  =  Node::new_with_parent(data,  atnode); node.right  =  Some(new_node); true } Some(rnode)  =>  self.insert_at(rnode,  data), } } } 
```

For simplicity, the code samples in this post separate the operation on the root into a top level function/method, which then calls a recursive method that operates at the node level. In this case, `insert_at` takes a link and inserts the new data as the child of that node. It preserves the BST invariant (smaller implies left child, larger implies right child). The interesting thing to note here is the `borrow_mut()` call at the very beginning. It obtains a mutable reference from the `RefCell` pointed to by `atnode`. But this isn't just a regular Rust mutable reference, as in `&mut`; instead, it's a special type called `std::cell::RefMut`. This is where the mutability magic happens - there is no `&mut` in sight, yet the code can actually mutate the underlying data .

To reiterate, this code remains safe; if you attempt to do another `borrow_mut()` on the `RefCell` while the previous `RefMut` is in scope, you'll get a run-time panic. The safety is guaranteed at run-time.

Another interesting example is the private `find_node` method, which finds and returns a node with the given data, starting from some node:

```
/// Find the item in the tree; returns `true` iff the item is found.
pub  fn find(&self,  data: i32)  -> bool { self.root .as_ref() .map_or(false,  |root|  self.find_node(root,  data).is_some()) } fn find_node(&self,  fromnode: &NodeLink,  data: i32)  -> Option<NodeLink>  { let  node  =  fromnode.borrow(); if  node.data  ==  data  { Some(fromnode.clone()) }  else  if  data  <  node.data  { node.left .as_ref() .and_then(|lnode|  self.find_node(lnode,  data)) }  else  { node.right .as_ref() .and_then(|rnode|  self.find_node(rnode,  data)) } } 
```

The `.borrow()` call at the beginning is how we ask a `RefCell` to provide a immutable reference to the internal data (naturally, this cannot coexist at run-time with any mutable references). When we return a node that was found, we `clone` the `Rc`, because we need a separate shared owner for the node. This lets Rust guarantee that the node cannot just be dropped while the returned `Rc` is still alive.

As the full code sample demonstrates, this approach is workable. It takes a lot of practice and patience to get right though, at least for inexperienced Rust programmers. Since each node is wrapped in three levels of indirection (`Option`, `Rc` and `RefCell`) writing the code can be somewhat tricky, since at any point you have to remember which level of inderection you're "currently in".

Another downside of this approach is that getting a plain reference to data stored in the tree isn't easy. As you can see in the sample above, the top-level `find` method doesn't return the node or its contents, but simply a boolean. This isn't great; for example, it makes the `successor` method sub-optimal. The problem here is that with `RefCell` we can't just return a regular `&` reference to the data, since the `RefCell` must keep a run-time track of all the borrows. We can only return a `std::cell::Ref`, but this leaks implementation details. This isn't a fatal flaw, but just something to keep in mind while writing code using these types.

## Using handles into a vector as Node references

The second approach we're going to dicuss has the `Tree` owning all the nodes created in it, using a simple `Vec`. Then, all node references become "handles" - indices into this vector. Here are the data structures:

```
pub  struct Tree  { // All the nodes are owned by the `nodes` vector. Throughout the code, a
  // NodeHandle value of 0 means "none".
  root: NodeHandle, nodes: Vec<Node>, count: usize, } type NodeHandle  =  usize; #[derive(Debug)] struct Node  { data: i32, left: NodeHandle, right: NodeHandle, parent: NodeHandle, } 
```

Once again, the full code is available [on GitHub](https://github.com/eliben/code-for-blog/blob/main/2021/rust-bst/src/nodehandle.rs); here I'll show some salient parts. Here's insertion:

```
/// Insert a new item into the tree; returns `true` if the insertion
/// happened, and `false` if the given data was already present in the
/// tree.
pub  fn insert(&mut  self,  data: i32)  -> bool { if  self.root  ==  0  { self.root  =  self.alloc_node(data,  0); }  else  if  !self.insert_at(self.root,  data)  { return  false; } self.count  +=  1; true } // Insert a new item into the subtree rooted at `atnode`.
fn insert_at(&mut  self,  atnode: NodeHandle,  data: i32)  -> bool { if  data  ==  self.nodes[atnode].data  { false }  else  if  data  <  self.nodes[atnode].data  { if  self.nodes[atnode].left  ==  0  { self.nodes[atnode].left  =  self.alloc_node(data,  atnode); true }  else  { self.insert_at(self.nodes[atnode].left,  data) } }  else  { if  self.nodes[atnode].right  ==  0  { self.nodes[atnode].right  =  self.alloc_node(data,  atnode); true }  else  { self.insert_at(self.nodes[atnode].right,  data) } } } 
```

Where `alloc_node` is:

```
// Allocates a new node in the tree and returns its handle.
fn alloc_node(&mut  self,  data: i32,  parent: NodeHandle)  -> NodeHandle  { self.nodes.push(Node::new_with_parent(data,  parent)); self.nodes.len()  -  1 } 
```

After writing code with `Option<Rc<RefCell<...>>>`, this handle approach feels *very simple*. There are no layers of indirection; a handle is an index; a reference to a node is a handle; handle 0 means "none", that's it .

This version is also likely to be much more efficient than the linked version, because it has far fewer heap allocations and the single vector is very cache friendly.

That said, there are some issues here as well.

First, we take some of the safety into our own hands. While this approach won't result in corrupted memory, double frees or accessing freed pointers, it can lead to run-time panics and other problems because we deal in "raw" indices to a vector. Due to bugs, these incides may point past the vector's bounds, or point at the wrong slot, etc. For example, nothing prevents us from modifying a slot while there are "live handles" to it.

The other issue is removing tree nodes. Right now, the code "removes" a node simply by not pointing to it with any live handle. This makes the node unreachable through the tree's methods, but it does not deallocate memory. In fact, this BST implementation never deallocates anything:

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

This is obviously wrong for real-world applications. At the very least, this implementation can be improved by creating a "free list" of unused indices which can be reused when nodes are added. A more ambitious approach could be to implement a full-fledged garbage collector for the nodes. If you're up for a challenge, give it a try ;-)

## Using raw pointers and unsafe blocks

The third and final approach to discuss is using *raw pointers* and `unsafe` blocks to implement our BST. This approach feels very familiar if you're coming from a C / C++ background. The full code is [here](https://github.com/eliben/code-for-blog/blob/main/2021/rust-bst/src/unsafeall.rs).

```
pub  struct Tree  { count: usize, root: *mut  Node, } #[derive(Debug)] struct Node  { data: i32, // Null pointer means "None" here; right.is_null() ==> no right child, etc.
  left: *mut  Node, right: *mut  Node, parent: *mut  Node, } 
```

Node links become `*mut Node`, which is a raw pointer to a mutable `Node`. Working with raw pointers is quite similar to writing C code, with a few idiosyncracies around allocating, deallocating and accsesing data through these pointers. Let's start with allocation; here are the `Node` constructors:

```
impl  Node  { fn new(data: i32)  -> *mut  Self  { Box::into_raw(Box::new(Self  { data, left: std::ptr::null_mut(), right: std::ptr::null_mut(), parent: std::ptr::null_mut(), })) } fn new_with_parent(data: i32,  parent: *mut  Node)  -> *mut  Self  { Box::into_raw(Box::new(Self  { data, left: std::ptr::null_mut(), right: std::ptr::null_mut(), parent, })) } } 
```

The simplest way I found to allocate new memory for a raw pointer is using `Box::into_raw`, and it works well as long as we remember that deallocating this memory is on us from that point on (more on this later).

Let's see how insertion works:

```
/// Insert a new item into the tree; returns `true` if the insertion
/// happened, and `false` if the given data was already present in the
/// tree.
pub  fn insert(&mut  self,  data: i32)  -> bool { if  self.root.is_null()  { self.root  =  Node::new(data); }  else  { if  !insert_node(self.root,  data)  { return  false; } } self.count  +=  1; true } // Inserts `data` into a new node at the `node` subtree.
fn insert_node(node: *mut  Node,  data: i32)  -> bool { unsafe  { if  (*node).data  ==  data  { false }  else  if  data  <  (*node).data  { if  (*node).left.is_null()  { (*node).left  =  Node::new_with_parent(data,  node); true }  else  { insert_node((*node).left,  data) } }  else  { if  (*node).right.is_null()  { (*node).right  =  Node::new_with_parent(data,  node); true }  else  { insert_node((*node).right,  data) } } } } 
```

The interesting point to notice here is the `unsafe` block the body of `insert_node` is wrapped in. This is required because this code dereferences raw pointers. In Rust it's OK to assign pointers and pass them around without special provisions, but dereferencing requires `unsafe`.

Let's see how removing nodes work; here's `replace_node`, which performs the same task as the similarly named method we've seen in the node handle approach:

```
// Replaces `node` with `r` in the tree, by setting `node`'s parent's
// left/right link to `node` with a link to `r`, and setting `r`'s parent
// link to the `node`'s parent. `node` cannot be null.
fn replace_node(&mut  self,  node: *mut  Node,  r: *mut  Node)  { unsafe  { let  parent  =  (*node).parent; if  parent.is_null()  { // Removing the root node.
  self.root  =  r; if  !r.is_null()  { (*r).parent  =  std::ptr::null_mut(); } }  else  { if  !r.is_null()  { (*r).parent  =  parent; } if  (*parent).left  ==  node  { (*parent).left  =  r; }  else  if  (*parent).right  ==  node  { (*parent).right  =  r; } } // node is unused now, so we can deallocate it by assigning it to
  // an owning Box that will be automatically dropped.
  Box::from_raw(node); } } 
```

This demonstrates how we deallocate heap data through raw pointers: by using `Box::from_raw`. This makes a `Box` that takes ownership of the data; a `Box` has a destructor, so it will actually deallocate it when it goes out of scope.

This brings us to an important point: we have to take care of releasing the memory of the `Tree` now. Unlike in the previous approaches, the default `Drop` implementation won't do here, since the only thing contained in our `Tree` is `root: *mut Node` and Rust has no idea how to "drop" that. If we run our tests without implementing the `Drop` trait explicitly, there will be memory leaks. Here's a simple implementation of `Drop` to fix that:

```
impl  Drop  for  Tree  { fn drop(&mut  self)  { // Probably not the most efficient way to destroy the whole tree, but
  // it's simple and it works :)
  while  !self.root.is_null()  { self.remove_node(self.root); } } } 
```

As an exercise, try to implement a more efficient `Drop`!

Writing the code with raw pointers felt fairly natural; while the final LOC count is similar, the raw pointers required significantly less mental burden than using `Option<Rc<RefCell<Node>>>` . Though I didn't benchmark it, my hunch would be that the pointer version is more efficient as well; at the very least, it eschews the dynamic borrow checks that `RefCell` does. The flip side is, of course, the loss of safety. With the `unsafe` version we can run into all the good-old C memory bugs.

## Conclusion

The goal of this post was to review the different approaches one could take to implement non-trivial linked data structures in Rust. It covers approaches with three levels of safety:

*   Fully safe with `Rc` and `RefCell`.
*   Memory safe but otherwise more prone to bugs (such as aliasing borrows) with integer handles into a `Vec`.
*   Fully unsafe with raw pointers.

All three approaches have their merits and are useful to know, IMHO. Anecdotal evidence from Rust's own standard library and some popular crates suggests that the third approach - using raw pointers and `unsafe` - is quite popular.

Thanks for reading! Since this is my first non-trivial Rust post, I'm particularly interested in any comments, feedback or suggestions. Feel free to send me [an email](mailto:eliben@gmail.com) or leave a comment on whatever aggregation site this gets reposted to - I keep an eye on these from time to time.

* * **