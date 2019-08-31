### 二叉树的最小深度



> 给定一个二叉树，找出其最小深度。
>
> 最小深度是从根节点到最近叶子节点的最短路径上的节点数量。

**说明:** 叶子节点是指没有子节点的节点。



##### 0. 定义一个二叉树节点

```java
public class TreeNode {
	int val;
	TreeNode left;
	TreeNode right;
	TreeNode(int x) {
		val = x;
    }
}
```



##### 1. 递归

```java
public class Solution {
    public int minDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }
        
        // 递归计算
        int leftMinDepth = minDepth(root.left);
        int rightMinDepth = minDepth(root.right);
        
        // 有子节点为空的，则返回另一个节点的深度。否则返回两则最小的深度。
        return leftMinDepth == 0 || rightMinDepth == 0 
            ? leftMinDepth + rightMinDepth + 1 
            : Math.min(leftMinDepth, rightMinDepth) + 1;
    }
}
```



##### 2. 非递归（宽度优先搜索）

> 出现第一个无子节点的节点，则该节点的深度为树的最小深度

```java
public class Solution {
    public int minDepth(TreeNode root) {
    	if (root == null) {
            return 0;
        }
        
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        
        int minDepth = 0;
        while (!queue.isEmpty()) {
            ++minDepth;
            
            // 逐层遍历，判断一层是否存在没有子节点的节点，则该节点的深度为树的最小深度
            int size = queue.size();
            for (int i = 0; i < size; i++) {
                root = queue.poll();
                if (root.left == null && root.right == null) {
                    return minDepth;
                }
                if (root.left != null) {
                    queue.offer(root.left);
                }
                if (root.right != null) {
                    queue.offer(root.right);
                }
            }
        }
        
        return minDepth;
    }
}
```

