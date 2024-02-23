# Algorithms and data strructures

Notes on some algorithms and data structures. [INCOMPLETE]


## binary search trees

A tree where each node value is bigger than the value of the left child and smaller or equal than the one on the right.


	          5
	         / \
	        4   6
	       / \
	      3  2

The invariant applies recursively to the subtrees rooted in each child.

The ordering of the tree allows fast search and insert, in `O(log(n))`, where `n` is the amount of elements.

Insert:

	insert v in tree T:
	  if T is empty
	    return a tree with a single node with value v
	
	  if T.value > v
	    if T.left is empty
	      create node with value v in T.left
	      return
	    insert v on T.left
	  else
	    if T.right is empty
	      create node with value v in T.right
	      return
	    insert v on T.right

Search:

	search value v in tree T:
	  if T is empty
	    return false
	  if T.value == v
	    return true
	
	  if T.value <= v
	    return search v in Tree.right
	
	  else
	    return search v in Tree.left


## avl

An example of a balanced binary search tree. To the properties of a _binary search tree_, this structure adds balancing: the difference in heights of the trees from any node is bounded by 1 and -1. In that way, searching elements within the tree grows in proportion to its height, logarithmicaly.

The cost is an increase in the implementation complexity, but not that much.

The usual operations performed in these trees are, like in BST's:

- `insert`
- `delete`
- `search`
- etc

A possible structure to implement a _node_:

	Node:
	    heigth: height of the tree rooted in this node
	    right_tree: reference to the right tree
	    left_tree: reference to the left tree
	    parent: reference to the parent node
	    value: value

To mantain a balance in the left and right branch height, each node has a _balance factor_. It is computed by:

	bf(node) = node.right_tree.height - node.left_tree.height

The invariant of the tree, adding to those of a normal BST, is that the balance factor of every node is 1, 0 or -1.

When inserting or deleting a node the balance factor may change, resulting in an inbalanced tree. A set of operations, called _rotations_, can fix them:


	
	      X[-2]                   Z[0]
	      /    \                 /   \ 
	   Y[1]   Xr   rotation     Y     X
	   /  \        -------->   / \   / \
	  Yl   Z                  Yl Zl Zr  Xr
	       / \
	     Zl  Zr
	
	     X[2]                   Y[0 or 1]
	    / \                       / \
	  Xl   Y[1]    rotation      X   Z
	       / \     -------->    / \
	    Yl   Z                 Xl  Yl
	         / \
	       Zl  Zr


([] holds the balance factor)

There are two other rotations which are symmetrical whith the above.


## b-tree

A tree with the invariant (taken from aocp vol III ed.2 p.483):

- every node has at most m children
- except for the root and leaves, nodes have at least `m/2` children
- the root has at least 2 children (unless is a leaf)
- all leaves appear on the same level and carry no information
- a non-leaf node with `k` children contains `k-1` keys

## todo


- fibonacci trees
- graphs: dfs, bfs, dijkstra, floyd, mst-algorithms, etc.
- heapsort, quicksort
- dynamic programming
