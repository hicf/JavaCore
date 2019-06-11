### 二叉树的遍历 ★★★★★



##### 0. TreeNode 结点

```java
private static class TreeNode<T> {
    T data;
    TreeNode left, right;

    TreeNode(T data) {
      this.data = data;
    }
    TreeNode() {
    }
}
```



##### 1. 递归调用 (深度优先遍历) ★☆☆☆☆

```java
public void preorderTraversal(TreeNode node) {
    if (node == null) {
      return;
    }
  // 可调整前、中、后序遍历
  System.out.print(node.data);
  preorderTraversal(node.left);
  preorderTraversal(node.right);
}
```



##### 2. 非递归遍历 - 基于栈 (深度优先遍历) ★★★☆☆

```java
public void preorderTraversalWithStack(TreeNode root) {
    Stack<TreeNode> stack = new Stack<>();
    TreeNode node = root;
    while (node != null || !stack.isEmpty()) {
      // 访问结点的数据，并且移动到左子结点
      while (node != null) {
        System.out.println(node.data);
        stack.push(node);
        node = node.left;
      }
      // 回溯，遍历右子结点
      if (!stack.isEmpty()) {
        node = stack.pop();
        node = node.right;
      }
    }
}
```



##### 3. 非递归遍历 - 基于队列 (层次遍历、广度优先遍历、O(n)) ★★★★☆

```java
public void levelorderTraversal(TreeNode root) {
    if (root == null) {
        return;
    }
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    TreeNode node;
    // 遍历队列
    while (!queue.isEmpty()) {
        // 从队列头部取出一个结点
        node = queue.poll();
        System.out.println(node.data);
        // 将左右子结点放入队列尾部
        if (node.left != null) {
            queue.offer(node.left);
        }
    	if (node.right != null) {
            queue.offer(node.right);
    	}
    }
}
```



##### 4. Morris Traversal(莫里斯遍历、O(n)) ★★★★☆

```java
public void morrisTraversal(TreeNode root) {
    TreeNode curr = root;
    TreeNode prev;
    while (curr != null) {
        // 向左子结点遍历
        if (curr.left != null) {
            prev = curr.left;
            while (prev.right != null && prev.right != curr) {
                prev = prev.right;
            }
            // 右子结点的回溯指针绑定
            if (prev.right == null) {
                prev.right = curr;
                curr = curr.left;
            } else {
                System.out.println(curr.data);
                prev.right = null;
                curr = curr.right;
            }
            // 向右子结点遍历
        } else {
            System.out.print(curr.data);
            curr = curr.right;
        }
    }
}
```

