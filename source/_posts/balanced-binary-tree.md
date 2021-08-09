---
title: AVL (Balanced Binary Search Tree)
tags:
  - binary search tree
  - algorithm
  - java
  - avl
categories:
  - Algorithm
date: 2021-08-05 10:38:00
---

![Binary Search Tree](https://miro.medium.com/max/2000/1*ic9Ou8ZXZeiADVCXpOaKIw.jpeg)

In our previous post([Binary Search Tree](https://www.mahfuz.info/2021/07/27/binary-search-tree/)), we have discussed binary search trees. We have checked out the insertion, deletion, and find operation for a BST.

In the best-case scenario, BST can provide O(log n) complexity but in the worst-case scenario, this gives us O(n) complexity. The worst-case can happen if the input data is sorted in any order. To prevent the worst case, there are extensions that can help us maintain the binary tree balanced and get the best outcome. Some of them are,

- AVL tree
- Red black tree
- Treap
- B-tree

Today, we will discuss the height-balanced AVL tree.

## AVL tree

AVL tree is an extension of the binary search tree. It has an additional field called the balance factor. After insert, delete, or modification operation, it always checks the balance factor of each node.

Based on the value of the balance factor, the tree self-balances itself.

## Balance factor

The balance factor of a node in an AVL tree is the difference between the height of the left subtree and the height of the right subtree of that node.
Balance factor = Height of Left Subtree — Height of Right Subtree (or inverse)
The value of the balance factor should always be [-1, 0 or+1]

```
public class TreeNode<T extends Comparable<T>> {
  private T key;
  private TreeNode<T> parent;
  private TreeNode<T> left;
  private TreeNode<T> right;
  private int height;
  private int balanceFactor;
  // getter / setter
public void updateBalanceFactor() {
  int left = 0, right = 0;
  if (this.left != null) {
    left = this.left.height;
  }
  if (this.right != null) {
    right = this.right.height;
  }
  this.height = Math.max(left, right) + 1;
  this.balanceFactor = left - right;
}
public boolean isBalanced() {
  return this.getBalanceFactor() <= 1 && this.getBalanceFactor() >= -1;
}

```

We will use a callback pattern for our AVL tree insert and delete methods.

```
public interface Callback<T> {
  void success(T value);
}
```

We have already discussed BST implementation in our previous post. We will extend the insert and delete operation here to maintain AVL properties.

```
public class BST<T extends Comparable<T>> {
  public void insert(T key, Callback<TreeNode<T>> callback) {
    callback.success(node);
  }
  public void delete(T key, Callback<TreeNode<T>> callback) {
    callback.success(node);
  }
}
```

## AVL implementation

```
public class AVL<T extends Comparable<T>> extends BST<T> {

  @Override
  public void insert(T key, Callback<TreeNode<T>> callback) {
    super.insert(key, (node) -> {
      fixAVLProperties(node, node.getKey());
      if (callback != null) {
        callback.success(node);
      }
    });
  }
  @Override
  public void delete(T key, Callback<TreeNode<T>> callback) {
    super.delete(key, (node) -> {
      TreeNode<T> parent = node.getParent();
      if (parent == null) {
        parent = root;
      }
      updateChildBalanceFactor(parent);
      TreeNode<T> unbalancedNode = getUnbalancedNode(parent);
      if (unbalancedNode != null) {
        T unbalancedKey = getUnbalancedKey(unbalancedNode);
        fixAVLProperties(unbalancedNode, unbalancedKey);
      }
      if (callback != null) {
        callback.success(node);
      }
    });
  }
}
```

## Insert

After the binary search tree insertion, we need to update the balance factor of each node. As we know we insert in any of the subtrees we are going to check all the way up to the parent to check if every node maintains the AVL properties.
If the node is unbalanced, we need to rotate the tree to maintain the balance factor ≤ 1 and ≥ -1

```
private void fixAVLProperties(TreeNode<T> node, T key) {
  TreeNode<T> item = node;
  while (item != null) {
    item.updateBalanceFactor();
    if (item.isBalanced() == false) {
      applyRotation(item, getUnbalancedDirection(item, key));
    }
    item = item.getParent();
  }
}
```

## Unbalanced Node

The subtree can be unbalanced in 4 different ways,

1. Left unbalanced (LL)
2. Right unbalanced (RR)
3. Left Right unbalanced (LR)
4. Right unbalanced (RL)

For these use cases, we have to rotate our tree to maintain our balance properties.

First, we have to find the unbalanced direction

```
private String getUnbalancedDirection(TreeNode<T> node, T key) {
  String str = "";
  TreeNode<T> item = node;
  int count = 0;
  while (item != null) {
    if (count == 2 || key == item.getKey()) {
      break;
    }
    int compareTo = key.compareTo(item.getKey());
    if (compareTo < 0) {
      str += "L";
      item = item.getLeft();
    } else {
      str += "R";
      item = item.getRight();
    }
    count++;
  }
  return str;
}
```

Next, we have to apply rotation based on the direction

```
private void applyRotation(TreeNode<T> node, String direction) {
  switch (direction) {
    case LEFT:
      rotateRight(node);
      break;
    case RIGHT:
      rotateLeft(node);
      break;
    case LEFT_RIGHT:
      rotateLeft(node.getLeft());
      rotateRight(node);
      break;
    case RIGHT_LEFT:
      rotateRight(node.getRight());
      rotateLeft(node);
      break;
    default:
      break;
  }
}
```

## Left rotation

To apply the rotation in left rotation,

```
private void rotateLeft(TreeNode<T> node) {
  if (node.getParent() != null) {
    if (node.getParent().getLeft() == node) {
      node.getParent().setLeft(node.getRight());
    } else {
      node.getParent().setRight(node.getRight());
    }
  }
  if (root == node) {
    root = node.getRight();
  }
  TreeNode<T> left = null;
  if (node.getRight()!= null && node.getRight().getLeft() != null) {
    left = node.getRight().getLeft();
    left.setParent(node);
  }
  node.getRight().setLeft(node);
  node.getRight().setParent(node.getParent());
  node.setParent(node.getRight());
  node.setRight(left);
  updateChildBalanceFactor(node.getParent());
}

```

## Right rotation

To apply the rotation in the right rotation

```
private void rotateRight(TreeNode<T> node) {
  if (node.getParent() != null) {
    if (node.getParent().getLeft() == node) {
      node.getParent().setLeft(node.getLeft());
    } else {
      node.getParent().setRight(node.getLeft());
    }
  }
  if (root == node) {
    root = node.getLeft();
  }
  TreeNode<T> right = null;
  if (node.getLeft() != null && node.getLeft().getRight() != null) {
    right = node.getLeft().getRight();
    right.setParent(node);
  }
  node.getLeft().setRight(node);
  node.getLeft().setParent(node.getParent());
  node.setParent(node.getLeft());
  node.setLeft(right);
  updateChildBalanceFactor(node.getParent());
}
```

## Update balance factor

```
private void updateChildBalanceFactor(TreeNode<T> node) {
  if (node == null) {
    return;
  }
  updateChildBalanceFactor(node.getLeft());
  updateChildBalanceFactor(node.getRight());
  node.updateBalanceFactor();
}
```

## Deletion

In terms of deletion, we need to check certain steps to determine if the balance factor is appropriate.

1. The parent of the deleted node has any child node unbalanced.
2. The unbalanced key that will determine the rotation direction

First, we need to check if any unbalanced node exists or not

```
private TreeNode<T> getUnbalancedNode(TreeNode<T> node) {
  if (node == null) {
    return null;
  }
  if (node.isBalanced() == false) {
    return node;
  }
  TreeNode<T> unbalancedNode = getUnbalancedNode(node.getLeft());
  if (unbalancedNode == null) {
    return getUnbalancedNode(node.getRight());
  }
  return leftUNode;
}
```

Then we have to check the key, for which the node is unbalanced. We just need to check two nodes here and because to find the unbalanced direction two nodes are sufficient.

```
private T getUnbalancedKey(TreeNode<T> node) {
  if (node == null || node.isBalanced()) {
    return null;
  }
  int count = 0;
  TreeNode<T> item = node;
  while (item != null) {
    if (count >= 2) {
      break;
    }
    if (item.getBalanceFactor() > 1) {
      item = item.getLeft();
    } else {
      item = item.getRight();
    }
    count++;
  }
  return item.getKey();
}
```

## Driver

We can check our AVL tree using a driver method

```
public class Main {
  public static void main(String[] args) {
    AVL<Integer> tree = new AVL<Integer>();
    int[] array = new int[] { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
    for (int i : array) {
      tree.insert(i, null);
    }
    tree.preorder(tree.getRoot(), (x) -> System.out.println(x));
  }
}
```

This will make our tree as

```
       4
      / \
     /   \
    /     \
   /       \
   2       6
  / \     / \
 /   \   /   \
 1   3   5   9
            / \
            8 10
```

Full [source code](https://gist.github.com/mahfuzsust/e67043588cbf42414ab3ecc0db1412fd) is shared.
