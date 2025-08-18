# Table of contents
<!-- TOC -->
* [Table of contents](#table-of-contents)
* [BST](#bst)
* [B-trees](#b-trees)
<!-- TOC -->

<br>

# BST
A **binary search tree** (**BST**) is a **sorted in-memory** data structure.<br>
Each tree node stores a **key**, a **value** associated with key and **2 pointers**: to **left subtree** and **right subtree**.<br>
The **leaf node** is a node that **doesn't** have any child.<br>

Thus, each node splits the **key space** into **left** and **right** subtrees.<br>
The **main BST property**:
- all keys $`K_{left}`$ in the **left** subtree are **less** than **node key** $`K`$;
- all keys $`K_{right}`$ in the **right** subtree are **greater** than **node key** $`K`$;

<br>

![binary-tree-invariant](/img/binary-tree-invariant.png)

<br>

So:
- following only **left** pointers from **root node** leads us to the **leaf node** holding the **smallest** key in the tree;
- following only **right** pointers from **root node** leads us to the **leaf node** holding the **largest** key in the tree;

<br>

Insertion to the BST may lead to the situation where BST is **unbalanced**.<br>
Tree is **unbalanced** when one of its **subtree** (aka **branch**) is **longer** than the others.<br>
In the **worst case** scenario a tree **degrades to linked list**.<br>
The **balanced tree** is a tree that has a **height** of $`log_{2}N`$, where $`N`$ is the **total number** of items in the tree.<br>
**Without** balancing, we **lose** _performance benefits_ of the BST.<br>
The **balancing** operation **reorganizes** nodes in the tree in the way that **minimizes tree height**.<br>

<br>

The **locality** of neighboring keys _depends on_ **fanout**.<br>
The **number of jumps** between nodes during tree traversal _depends on_ **tree height**: we need perform $`log_{2}N`$ jumps to locate the searched key.<br>
In turn, the **tree height** depends on **fanout** too.<br>
So, the **low** fanout of BST **increases** _height of tree_ and **decreases** _locality_ of neighboring keys.<br>

<br>

# B-trees
When the amounts of data are so large that keeping an entire dataset in memory is impossible or not feasible, 
only a fraction of data can be cached in memory at any time, and the rest has to be stored on disk on a manner that allows efficiently accessing it.<br>
**Not** every data structure can be effectively used for on-disk storage.<br>
There are 2 types of disk device: **SSD** and **HDD**.<br>
On **spinning disks** (**HDD**), **seeks increase costs of random reads** because they require disk **rotation** and 
**mechanical head movements** to position the head to the desired location.<br>
However, once the **expensive** part is done, reading or writing **contiguous** bytes (i.e., **sequential operations**) is relatively **cheap**.<br>
Unlike HDD SSDs **don't** have moving parts and random access in SSD is fast.<br>
**But** in both SSD and HDD device types we address _blocks of memory_ **rather** than _individual bytes_.<br>
That's why most OSs have **block device abstraction**. So, when we read a **single byte** from block device, the **whole block** containing it is read.<br>
And we should take into account this feature when design **on-disk data structure**.<br>

A **cost** of _disk seek_ is **expensive**.<br>
So, the **following child pointers** (jumps between nodes) in a tree determines **number of disk seeks** during tree traversal.<br>
A **BST** as **on-disk data structure** will require $`log_{2}N`$ **disk seeks** to locate the searched key. This fact makes BST 
**impractical** as _on-disk data structure_.<br>

<br>

A data structure can be **effectively** used for on-disk storage must have **high fanout** and **low height**:
- **high fanout** **improves** _locality_ of the neighboring keys;
- **low height** **reduces** the _number of seeks_ during lookup;

<br>

**B-Trees** are suited to be used on disk: they have **small** _height_ and **high** _fanout_.<br>
**B-Trees** allow to store values on **any level**: in _root_, _internal_ and _leaf nodes_.<br>

<br>

The layout of **B-Tree node** with **3** keys example (here $`K_{s}`$ denotes all keys that belong to the appropriate subtree):<br>
![btrees-node-layout](/img/btrees-node-layout.png)
<br>

In **B⁺-Trees** values can be stored **only** in _leaf nodes_.<br>
The most RDBMS use **B⁺-Trees**.<br>

<br>

**B-Tree** properties:
- the **root node** must hold $`1; 2k`$ keys;
- the **other nodes** must hold $`k-1; 2k-1`$ keys;
- **all** leafs must be at the same layer;
- _B-Trees_ are **sorted**: keys inside every node are stored **in order**;

<br>

If $`N`$ is a number of all nodes in tree then the **height** of **B-Tree** is $`H=log_{k}N`$.<br>
It means **B-Tree** requires $`k`$ times **less** disk seeks than **BST**.<br>
Because of _B-Trees_ are **sorted**, to locate a searched key we can use a binary search and the number of comparisons $`H=log_{2}N`$.<br>
