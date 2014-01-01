avl-trees
=========

AVL trees are balanced binary search trees that are simple enough to understand but often
poorly described.

AVL trees are named after their inventors, G. M. Adelson-Velskii and E. M. Landis, who
described them in a 1962 paper that I found difficult to follow. You can read about
them in [Wikipedia](https://en.wikipedia.org/wiki/AVL_tree), but the presentation is
marred by unnecessary terminology, like "left-left case", which has little to do with
what is actually going on.

Binary Trees
------------

In a binary tree, each node has a left and right link, and data that can be ordered.
By convention, all the data in nodes in the left subtree of a node are less than
the data in the node, and all data in the right subtree are greater. Thus,

       b
      / \
     a   c

is a valid binary search tree. Here the links are represented by / and \ characters
and only the data is shown in the node.

Searching for Data
------------------

To locate a data node, begin at the root node. If the data
sought is in the node, the search is successful; otherwise, if the data sought is less
than the node data and the node has a valid left link, next consider it; otherwise,
if the node has a valid right link, next consider it. And so on.
Either the search will succeed or there will be no more links to follow and it
will fail.

Adding Data
-----------

To add data, search for the data to be added as above. If it is not found, the last node
considered will be a leaf node; simply create a new leaf node for the data and link to it
from the previous leaf. Thus, data are always added at the leaves and adding it always
increases the height of the subtree it is added to by 1.

      To add c to      a        =>     a
                        \               \
                         d               d
                                        /
                                       c

(If the data is already in the tree, there are two choices; either leave it
in the tree and return, or replace the existing data in the same node. The former is
usually done when the tree is used to represent a set, and the latter when it is used to
represent a map. In the latter case, comparisons will be done using a key embedded in
the data, which will also contain at least one other value. Neither will affect the
tree height.)

Deleting Data
-------------

Deletion is a little more complicated. If the node containing the data to be deleted
is a leaf, it can simply be removed and the link to it erased. If the node has only
a left or right link, the link to the node can be replaced by the left or right link.

     To delete d from   a    =>   a     a    =>  a      a     =>   a
                         \                \        \      \          \
                          d                d        c      d          e
                                          /                 \
                                         c                   e

When a node to be deleted has both right and left links, the tree must be reshaped
to maintain correctness. There are two possible ways to go about this; without loss
of generality we will consider the left link. Locate the node with the next lower
data value than the data to be deleted. This can be done by starting with the left
link and then following a continuous sequence of right links until there are no
more. This is the next lower node.

What we are going to do is replace the deleted node with the next lower node.
Note that the next lower node can never have a right link, so it is replaced
with the right link of the node to be deleted. The left link of the next lower
node replaces the current link to it, and is replaced by the left link of the
deleted node. This is simpler to understand in a diagram:

     To delete d from     d      =>     c
                         / \           / \
                        a   e         a   e
                         \             \
                          c             b
                         /
                        b

Deletion always reduces the height of a subtree by 1.

Tree Height
-----------

We have been talking about "height of the tree" without actually defining it.
The height of an empty link is zero. The height of a non-empty link is the
height of the node it points to. The height of a leaf node is thus 1 and
the height of a non-leaf node is _max(height(left),height(right))+1_.

The height of a node could obviously be recursively calculated by following links
down to the leaves, but since height is so important to an AVL tree it is kept
in each node. When a node is created and any time a left or right link is changed
the height is quickly recalculated by examining only two links.

Balance
-------

We talked about a "balanced binary tree". Balance can be defined in terms of
height. An AVL node is considered balanced if its left and right subtrees
differ in height by at most 1. (This is the best we can do, or we wouldn't be
able to have a two-node tree.)

In practice, we calculate _balance=height(left)-height(right)_. If balance
is greater than 1 we must reduce the height of the left subtree; if it is
less that -1 we must reduce the height of the right subtree.

Tree Rotations
--------------

How can we reduce the height of a subtree? As you may have noticed, the arrangement
of nodes in a binary tree is arbitrary depending on the order they were added to
the tree. Thus the following trees are equivalent:

         1         2      3      4         5
         b         c      c      a         a 
        / \       /      /        \         \ 
       a   c     b      a          b         c 
                /        \          \       / 
               a          b          c     b

However, they differ in an important way. Tree 1 is shorter than
any of the others, and it is balanced, while the root node in all the others is
out of balance. To bring them into balance, the left subtrees
of 2 and 3 must be shortened, while for 4 and 5, the right subtrees.

To shorten a subtree, we apply tree transformations called _rotations_. For example,
transforming tree 2 to tree 1 can be thought of as a _right rotation_ (clockwise),
while transforming tree 4 to tree 1 can be thought of as a _left rotation_
(counter-clockwise).

What about 3 and 5? If we try to rotate 3 right, we wind up with:

           3                     5
           c   rotate right to   a
          /                       \
         a                         c
          \                       /
           b                     b
           
Which is not progress! But what we can do is first rotate the 'a' subtree left and
then rotate right.

           3                      2                       1
           c                      c    rotate right to    b
          /                      /                       / \
         a    rotate left to    b                       a   c
          \                    /
           b                  a

Likewise, for case 5, we can rotate left only after we rotate the 'c' subtree right,
turning it into case 4. In general, at most two rotations are required to balance a
node.

(Of course, the newly balanced node is not the same node we started out with,
so the link to the original node must be replaced with a link to the balanced node.)

Pseudo-code
-----------

A right rotation can be expressed in pseudo-code (for a mutable tree):

    Node rotateRight(Node node) {
      Node left = node.left
      node.setLeft(left.right)
      left.setRight(node)
      return left
    }

Likewise, a left rotation:

     Node rotateLeft(Node node) {
       Node right = node.right
       node.setRight(right.left)
       right.setLeft(node)
       return right
     }

(The setLeft and setRight methods are assumed to recalculate the height of the
target node. The methods return the newly balanced node, so it can be linked to
as discussed above.)

We have omitted from our diagrams any consideration of the left.right link on a
right rotation or a right.left link on a left rotation. The following diagrams should
clear that up and enable you to better see what the code is doing.

          2      rotate right to      1
          c                           b
         /                           / \
        b                           a   c
       / \                             /
      a   ?                           ?
      
         4       rotate left to       1
         a                            b
          \                          / \
           b                        a   c
          / \                        \
         ?   c                        ?

In each case, the replacement for the rotated node is now balanced and the
resulting tree is correct. (These examples assume there is some ?
value between b and c in 2, perhaps 'bb', or between a and b in 4, perhaps 'aa'?)

The rotations are actually applied in a method we will call balanceNode,
which fixes any imbalance it finds in a tree node.

In pseudo-code, a balanceNode method will look like this:

     Node balanceNode(Node node) {
       int balance = height(node.left) - height(node.right)
       if (balance > 1) {
         // left subtree too high
         int leftBalance = height(node.left.left) - height(node.left.right)
         if (leftBalance < 0) {
           // left.right higher
           node.setLeft(rotateLeft(node.left))
         }
         node = rotateRight(node)
       } else if (balance < -1) {
         // right subtree too high
         int rightBalance = height(node.right.left) - height(node.right.right)
         if (rightBalance > 0) {
           // right.left higher
           node.setRight(rotateRight(node.right))
         }
         node = rotateLeft(node)
       }
       return node
     }

The rule is, whenever a node is added or moved up in the course of delete, balanceNode
must be called on the parent of the added or moved up node and all the way up the tree
to the root. Since the tree is usually traversed recursively to add or delete,
it is simple to call balanceNode before each return.

(Delete may take a little detour down to the next lower node, but again this is done
recursively and balanceNode can be applied at each return.)

Project Code
------------

The code examples show two different ways to apply AVL trees.

*AvlTree* is a functional implementation. Each node represents a complete AVL subtree
and is immutable. Adding or deleting a node returns a new tree that contains (or
does not contain) the added (deleted) node and shares structure with the unaffected
parts of the tree. The functional implementation is useful for concurrent
applications where state changes can be isolated to a reference to the single node
at the root of the tree, or for serial applications where the capability to
instantly revert to any previous state of the tree is desired.

*AvlTreeSet* is a procedural (normal Java style) implementation. It implements the
java.util.Set interface and is highly stateful. Instead of creating new nodes
for link changes, links are modified in place, etc. It "plays nice" with the
Collection classes, and can be used as a basis for java.util.Map, etc.

Performance
-----------

In a mini-benchmark adding and removing 100,000 strings, AvlTreeSet runs about the same
speed as TreeSet from the Collection classes. As expected, AvlTree is slower, but
only about 30% slower, which may not be an issue if its additional features are
desired. Some representative numbers:

     Class      Sec.  Difference
     TreeSet    3.44  0.00%
     AvlTreeSet 3.56  3.31%
     AvlTree    4.45  29.27%

There is a great deal of variability in the results, e.g.,

     TreeSet    4.66  0.00%
     AvlTreeSet 4.38  -6.05%
     AvlTree    5.39  15.67%

     TreeSet    3.29  0.00%
     AvlTreeSet 4.03  22.40%
     AvlTree    4.58  39.23%
     
Which is probably a comment on my ability to construct a micro-benchmark.

Finally, it should be noted that _any_ set implementation that maintains order
is seriously slower than a hash set. For example, java.util.HashSet runs the
same test in approx. 0.5 seconds.
