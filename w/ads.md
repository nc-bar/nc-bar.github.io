# Algorithms and data strructures

Notes some algorithms and data structures.


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

The invariant of the tree, adding to those of a normal BFS, is that the balance factor of every node is 1, 0 or -1.

To mantain this invariant it is necesary to modify the _insert_ and _delete_ methods. The algorithms are the same as the respective BFS algorithms, but after the changes, the balance factor of the nodes from the root to the ones changed needs to be restored.

To restore the invariant in those nodes, there are operations known as _tree rotations_. They take a tree and produce an equivalent one with a different balance factor.

The rotations are:

This rotation is useful when the inserted node is `Z`:


	          X[-2]                  Z[0]
	         /    \                 /   \ 
	       Y[1]    Xr              Y     X
	       /  \        ----->     / \   / \
	      Yl   Z                 Yl Zl Zr  Xr
	          / \
	         Zl  Zr

This rotation is useful when the inserted node is in the place of `Z`


	     X[2]                  Y[0,1]
	    / \                   / \
	  Xl   Y[1]              X   Z
	      / \     ----->    / \
	    Yl   Z             Xl  Yl
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
