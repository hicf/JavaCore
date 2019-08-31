### 二叉树的遍历 ★★★★★



##### TreeNode 节点

```java
/* Definition for a binary tree node. */
public class TreeNode {
  int val;
  TreeNode left;
  TreeNode right;

  TreeNode(int x) {
    val = x;
  }
}
```



##### 1. 递归调用 (深度优先遍历) ★☆☆☆☆

```java
public void preorderTraversal(TreeNode root) {
    if (root == null) {
      return;
    }
  // 可调整前、中、后序遍历
  System.out.print(root.val);
  preorderTraversal(root.left);
  preorderTraversal(root.right);
}
```



##### 2.1 非递归遍历 - 基于栈 (前序遍历、深度优先遍历) ★★★☆☆（不推荐）

```java
public List<Integer> preorderTraversalWithStack(TreeNode root) {
    
    LinkedList<Integer> output = new LinkedList<>();
    
    // 用于回溯的栈
    Stack<TreeNode> stack = new Stack<>();

    while (root != null || !stack.isEmpty()) {
      // 访问结点的数据，并且移动到左子结点，直到无左子结点
      while (root != null) {
        output.add(root.val);
        stack.push(root); // 将当前节点 push 到栈中，方便后续回溯
        root = root.left; // 遍历左子结点
      }
      // 回溯，遍历右子结点
      if (!stack.isEmpty()) {
        root = stack.pop().right;
      }
    }
    
    return output;
}
```



##### 2.2非递归遍历 - 基于栈改进版（基于队列的栈模式）（执行时间较上者提升1倍） ★★★★☆

```java
public List<Integer> preorderTraversalWithStack(TreeNode root) {
    
    LinkedList<Integer> output = new LinkedList<>();
    
    // 用于回溯的栈(基于链表实现，不用担心栈的扩容问题)
    LinkedList<TreeNode> stack = new LinkedList<>();

    while (root != null || !stack.isEmpty()) {
      // 访问结点的数据，并且移动到左子结点，直到无左子结点
      while (root != null) {
        output.add(root.val);
        stack.add(root); // 将当前节点 add 到栈中，方便后续回溯
        root = root.left; // 遍历左子结点
      }
      // 回溯，遍历右子结点
      if (!stack.isEmpty()) {
        root = stack.pollLast().right;
      }
    }
    
    return output;
}
```





##### 3. 非递归遍历 - 基于队列 (层次遍历、广度优先遍历、O(n)) ★★★★☆

```java
public List<Integer> levelorderTraversal(TreeNode root) {
    LinkedList<Integer> output = new LinkedList<>();
    if (root == null) {
     	return output;
    }
    
    Queue<TreeNode> queue = new LinkedList<>();
    queue.add(root);
    
    // 遍历队列
    while (!queue.isEmpty()) {
        // 从队列头部取出一个结点
        root = queue.poll();
        output.add(root.val);
        // 将左右子结点放入队列尾部
        if (root.left != null) {
            queue.add(root.left);
        }
    	if (root.right != null) {
            queue.add(root.right);
    	}
    }
    
    return output;
}
```



##### 4. Morris Traversal(莫里斯遍历、O(n)) ★★★★☆

```java
public List<Integer> morrisTraversal(TreeNode root) {
    LinkedList<Integer> output = new LinkedList<>();
    
    TreeNode prev;
    TreeNode curr = root;
    while (curr != null) {
        // 向左子节点遍历
        if (curr.left != null) {
            prev = curr.left;
            while (prev.right != null && prev.right != curr) {
                prev = prev.right;
            }
            // 右子节点的回溯指针绑定
            if (prev.right == null) {
                prev.right = curr;
                curr = curr.left;
            } else {
                output.add(curr.val);
                prev.right = null;
                curr = curr.right;
            }
            // 向右子节点遍历
        } else {
            output.add(curr.val);
            curr = curr.right;
        }
    }
    
    return output;
}
```

