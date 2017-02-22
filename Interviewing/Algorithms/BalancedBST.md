# Balanced Search Trees
## 2-3 search trees
The primary step to get flexibity of guaranteed balance in search trees is to allow the nodes in our trees to hold more than one key.

**Definition**. A _2-3 search tree_ is a tree that either is empty or:

* A _2-node_, with one key (and associated value) and two links, a left link to a 2-3 search tree with smaller keys, and a right link to a 2-3 search tree with larger keys.

* A _3-node_, with two keys (and associated values) and three links, a left link to a 2-3 search tree with smaller keys, a middle link to a 2-3 search tree with keys between the node's keys and a right link to a 2-3 search tree with larger keys.

![](http://algs4.cs.princeton.edu/33balanced/images/23tree-anatomy.png)

A _perfectly balanced_ 2-3 search tree (or 2-3 tree for short) is one whose null links are all the same distance from the root.

* _Search_. To determine whether a key is in a 2-3 tree, we compare it against the keys at the root: If it is equal to any of them, we have a search hit; otherwise, we follow the link from the root to the subtree corresponding to the interval of key values that could contain the search key, and then recursively search in that subtree

![](http://algs4.cs.princeton.edu/33balanced/images/23tree-search.png)

* _Insert into a 2-node_. To insert a new node in a 2-3 tree, we might do an unsuccessful search and then hook on the node at the bottom, as we did with BSTs, but the new tree would not remain perfectly balanced. It is easy to maintain perfect balance if the node at which the search terminates is a 2-node: We just replace the node with a 3-node containing its key and the new key to be inserted.

![](http://algs4.cs.princeton.edu/33balanced/images/23tree-insert2.png)

* _Insert into a tree consisting of a single 3-node_. Suppose that we want to insert into a tiny 2-3 tree consisting of just a single 3-node. Such a tree has two keys, but no room for a new key in its one node. To be able to complete the insertion, we temporarily put the new key into a 4-node, a natural extension of our node type that has 3 keys and 4 links. Creating the 4-node is convenient because it is easy to convert it into a 2-3 tree made up of three 2-nodes, one with the middle key (at the root), one with the smallest of the three keys (pointed to by the left link of the root), and one with the largest of the three keys (pointed to by the right link of the root).

![](http://algs4.cs.princeton.edu/33balanced/images/23tree-insert3a.png)

* _Insert into a 3-node whose parent is a 2-node_. Suppose that the search ends at a 3-node at the bottom whose parent is a 2-node. In this case, we can still make room for the new key while maintaining perfect balance in the tree, by making a temporary 4-node as just described, then splitting the 4-node as just described, but then, instead of creating a new node to hold the middle key, moving the middle key to the nodes parent.

![](http://algs4.cs.princeton.edu/33balanced/images/23tree-insert3b.png)

* _Insert into a 3-node whose parent is a 3-node_. Now suppose that the search ends at a node whose parent is a 3-node. Again, we make a temporary 4-node as just described, then split it and insert its middle key into the parent. The parent was a 3-node, so we replace it with a temporary new 4-node containing the middle key from the 4-node split. Then, we perform precisely the same transformation on that node. That is, we split the new 4-node and insert its middle key into its parent. Extending to the general case is clear: we continue up the tree, splitting 4-nodes and inserting their middle keys in their parents until reaching a 2-node, which we replace with a 3-node that does not need to be further split, or until a 3-node at the root.

![](http://algs4.cs.princeton.edu/33balanced/images/23tree-insert3c.png)

* _Splitting the root._ If we have 3-nodes along the whole path from the insertion point to the root, we end up with a temporary 4-node at the root. In this case, we split the temporary 4-node into three 2-nodes.

![](http://algs4.cs.princeton.edu/33balanced/images/23tree-split.png)

* _Local transformations_. The basis of the 2-3 tree insertion algorithm is that all of these transformations are purely local. No part of the 2-3 tree needs to be examined or modified other than the specified node and links. The number of links changed for each transformation is bounded by a small constant. Each transformation passes up one of the keys from a 4-node to that nodes parent in the tree, and then restructures links accordingly, without touching any other part of the tree.

* _Global properties_. These local transformations preserve the _global_ properties that the tree is **ordered** and **balanced**. The number of links on the path from the root to any null link is the same.

**Proposition**. Search and insert operations in a 2-3 tree with `N` keys are guaranteed to visit at most `lg N` nodes.

![](http://algs4.cs.princeton.edu/33balanced/images/23tree-random.png)

However, we are only part of the way to an implementation. Although it would be possible to write code that performs transformations on distinct data types representing 2- and 3- nodes, most of the tasks that we have described are inconvenient to implement in this direct representation.

## Red-black BSTs
The insertion algorithm for 2-3 trees just described is not difficult to understand. We consider another simple representation known as a _red-black BST_ that leads to a natural implementation.

* _Encoding 3-nodes_. The basic idea behind red-black BSTs is to encode 2-3 trees by starting with standard BSTs (which are made of only 2-nodes) and adding extra information to encode 3-nodes. We think of the links as being of two different types: _red_ links, which bind together two 2-nodes to represent 3-nodes, and _black_ links, which bind together the 2-3 tree. Specifically, we represent 3-nodes as two 2-nodes connected by a single red link that leans left. We refer to BSTs that represent 2-3 trees in this way as red-black BSTs.

![](http://algs4.cs.princeton.edu/33balanced/images/redblack-encoding.png)

* _A 1-1 correspondence_. Given any 2-3 tree, we can immediately derive a corresonding red-black BST, just by converting each node as specified. Conversely, if we draw the red links horizontally in a red-black BST, all of the null links are the same distance from the root, and if we then collapse together the nodes connected by red links, the result is a 2-3 tree.

![](http://algs4.cs.princeton.edu/33balanced/images/redblack-1-1.png)

* _Color representation_. Since each node is pointed to by precisely one link (from its parent), we encode the color of links in _nodes_, by adding a `bool` instance variable color to our `Node` data type, which is `true` if the link from the parent is red and `false` if it is black. By convention, null links are black.

![](http://algs4.cs.princeton.edu/33balanced/images/redblack-color.png)

* _Rotations_. The implementation that we will consider might allow right-leaning red-links or two red-links in a row during an operation, but it always corrects these conditions before completion, through judicious use of an operation called _rotation_ that switches orientation of red links. First, suppose that we have a right-leaning red link that needs to be rotated to lean to the left. This operation is called _left rotation_. Implementing a _right rotation_ that converts a left-leaning red link to a right-leaning one amounts to the same code, with left and right interchanged.

* _Flipping colors_. The implementation that we will consider might also allow a black parent to have two red children. The _color flip_ operation flips the colors of the two red children to black and the color of the black parent to red.

<tbody><tr>
<td>![](http://algs4.cs.princeton.edu/33balanced/images/redblack-left-rotate.png)</td>
<td>![](http://algs4.cs.princeton.edu/33balanced/images/redblack-right-rotate.png)</td>
<td>![](http://algs4.cs.princeton.edu/33balanced/images/color-flip.png)</td>
</tr></tbody>

* _Insert into a single 2-node_.
* _Insert into a 2-node at the bottom_.
* _Insert into a tree with two keys (in a 3-node)_.
* _Keeping the root black_.
* _Insert into a 3-node at the bottom_.
* _Passing a red link up the tree_.

### Implementation
```C++
Node* insert(Node *h, Key k, Value v) {
	if (!h) { N++; root.color = BLACK; assert(check()); }
	if (k < h->key) h.left = insert(h.left, k, v);
	else if (k > h->key) h.right = insert(h.right, k, v);
	else h->val = v;

	// fix any right-leaning links
	if (isRed(h.right) && !isRed(h.left))		h = rotateLeft(h);
	if (isRed(h.left) && isRed(h.left.left))	h = rotateRight(h);
	if (isRed(h.left) && isRed(h.right))		flipColors(h);

	return h;
}

bool check() {
	if (!isBST()) { return false; }  // not in order
	if (!is23()) { return false; }  // not a 2-3 tree
	if (!isBalanced()) { return false; }  // not balanced
	return true;
}

bool isBST() {
	if (root == nullptr) return true;
	if (x->left && x->left->key >= x->key) return false;
	if (x->right && x->right->key <= x->key) return false;
	return isBST(x->left, KEY_MIN, x->key) &&
		isBST(x->right, x->key, KEY_MAX);
}
bool isBST(Node *x, Key min, Key max) {
	if (x == nullptr) return true;
	if (x->key >= max) return false;
	if (x->key <= min) return false;
	return isBST(x->left, KEY_MIN, x->key) &&
		isBST(x->right, x->key, KEY_MAX);
}

bool is23() { return is23(root); }
bool is23(Node *x) {
	if (!x) return true;
	if (isRed(x->right)) return false;
	if (x != root && isRed(x) && isRed(x->left))
		return false;
	return is23(x->left) && is23(x->right);
}

bool isBalanced() {
	// count number of black links from root to min
	int black = 0;
	Node *x = root;
	while (x) {
		if (!isRed(x)) black++;
		x = x.left;
	}
	return isBalanced(root, black);
}
bool isBalanced(Node *x, int black) {
	// does every path from the root to a leaf have the same
	// number of black links?
	if (!x) return black == 0;
	if (!isRed(x)) black--;
	return isBalanced(x->left, black) &&
		isBalanced(x->right, black);
}
```

![](http://algs4.cs.princeton.edu/33balanced/images/redblack-construction.png)

**Proposition**. The height of a red-black BST with `N` nodes is no more than `2 lg N`.

**Proposition**. In a red-black BST, the following operations take logarithmic time in the worst case: search, insertion, finding the minimum, finding the maximum, floor, ceiling, rank, select, delete the minimum, delete the maximum, delete, and range count.

**Property**. The average length of a path from the root to a node in a red-black BST with `N` nodes is `~1.00 lg N`.


## Red-Black Trees as B-trees

![](http://upload.wikimedia.org/wikipedia/commons/thumb/6/66/Red-black_tree_example.svg/500px-Red-black_tree_example.svg.png)
![](http://upload.wikimedia.org/wikipedia/commons/thumb/7/72/Red-black_tree_example_%28B-tree_analogy%29.svg/500px-Red-black_tree_example_%28B-tree_analogy%29.svg.png)

### Invariants

1. A node is either black or red.
2. The root is black.
3. All leaves are black.
4. Every red node has 2 black children.
5. Every path from a given node to its descendants contains the same number of black nodes

### Insertion

#### Case 1: `N` is the root.
Simply paint `N` black.

```C++
void insert_case1 (Node *n) {
	if (n->parent == nullptr) n->color = BLACK;
	else insert_case2(n);
}
```

#### Case 2: `N`'s parent `P` is black.
Just skip this case, as no properties are invalidated.

```C++
void insert_case2 (Node *n) {
	if (n->parent->color == BLACK) return;
	insert_case3(n);
}
```

#### Case 3: `N`'s parent `P` is red. So is uncle `U`.

![](http://upload.wikimedia.org/wikipedia/commons/thumb/d/d6/Red-black_tree_insert_case_3.svg/500px-Red-black_tree_insert_case_3.svg.png)

* `P` being red invalidates prop. 4, so paint it black.
* Painting `P` black upsets prop. 5 for `G`, so paint `U` black.
* If `G` is black, then `P, U, G` are all black - there are too many blacks. This upsets prop. 5 for the ancestor of `G`, so paint `G` to red just in case. Check if changing `G` to red upsets an properties.

```C++
void insert_case3 (Node *n) {
	Node *u = uncle(n);
	if (n->parent->color == RED && u->color == RED) {
		n->parent->color = BLACK;
		u->color = BLACK;
		g = grandparent(n);
		g->color = RED;
		insert_case1(g);
	} else
		insert_case4a(g);
}
```

#### Case 4: `N`'s parent `P` is red. `U` is black.


![](http://upload.wikimedia.org/wikipedia/commons/thumb/8/89/Red-black_tree_insert_case_4.svg/500px-Red-black_tree_insert_case_4.svg.png)

* If `N` is the right/left child of `P`, and `P` is the left/right child of `G`, first make it the left/right child.

```C++
void insert_case4a (Node *n) {
	Node *g = grandparent(n);
	if (n == n->parent->right && n->parent == g->left) {
		rotate_left(n->parent);
		n = n->left;
	}
	else if (n == n->parent->left && n->parent == g->right) {
		rotate_right(n->parent);
		n = n->right;
	}
	insert_case4b(n);
}
```

![](http://upload.wikimedia.org/wikipedia/commons/thumb/d/dc/Red-black_tree_insert_case_5.svg/500px-Red-black_tree_insert_case_5.svg.png)

* Both `N` and `P` are red, so prop. 4 is invalidated. Change `P` to black.
* `P` being black invalidates prop. 5 for `G`, so change `G` to red.
* `P` being black also invalidates prop. 5 for `G`'s ancestor, and no color changes to `N, P, U, or G` will resolve that. Thus, rotate to restore the number of black nodes under and including `G` to the orignal number.

```C++
void insert_case4b (Node *n) {
	Node *g = grandparent(n);
	n->parent->color = BLACK;
	g->color = RED;
	if (n == n->parent->left)
		rotate_right(g);
	else
		rotate_left(g);
}
```


### Deletion
Suppose we want to delete a targetted value from a red black tree. Let's first consider the algorithm for a (regular/plain/vanilla) BST:

> Let `N` be the node that stores the targetted key. If `N` has at most one child, delete `N` and replace it with its child. If `N` has 2 children, replace its value with that of its immediate successor `S`, then delete `S`.

Deleting a node outright would shorten at least one simple path from the root to a leaf. If the node we deleted was red, we will not have disturbed this property. However, if we delete a black node, we will destroy this property.

#### The Approach
Okay, so how do we fix this? Here's our strategy. If the node deleted was red, just delete it in the BST case. If the to-be-deleted node is black, we empty its value and associate the now valueless but still black node with whatever node comes to take its place. This can even be a terminal node. If we allow the empty but black node to be counted when measuring the black-node-length of a simple path, we will have retained the original black height of the tree. Our goal then is to remove this empty black node from the tree.

Note that we can discard the extra black node if it sits right below

* the root of the tree (as it is counted as an exra black node for every path).
* a red node (as this node can be re-colored black).

Let's try to either move this valueless black node up towards the root or arrange for the empty black carrier to have a red ancestor, all while retaining the properties of the red-black tree.

![](http://i.imgur.com/nO8vrMH.png)
![](http://i.imgur.com/hWKxr9a.png)
![](http://i.imgur.com/gj929l6.png)

#### The General Algorithm
We've developed a methodology for deletion - vanilla BST deletion plus a 'double-black' elimination routine. We've defined the basic goal of double-black elimination (pushing the extra-black to the root or to a red node), and worked out a few simple cases for how this goes. These are all the ingredients of the algorithm. While there are more cases outlined above, no matter the state of the red black tree, it's possible to either move the extra black up the tree, or rotate a neighboring red node above the double-black one (for eventual elimination).

