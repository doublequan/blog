---

layout: post
title: 'Binary Tree的O(1)space遍历 Morris Traversal'
date: '2016-3-10'
header-img: "img/post-bg-web.jpg"
tags:
     - Binary Tree Traversal
     - O(1)
author: 'Bill Quan'

---

## Morris Traversal

树的遍历通常有Iterative和Recursive两种方法。但是这两种方法都要extra space。（Note that even recursive solution is O(logn) space since it needs space in system function stack）

Morris Traversal方法可以做到只需要O(1)空间，而且同样只有O(n)的时间复杂度。

先上代码。下面是inorder traversal的morris方法。 

	public void morrisTraversal(TreeNode root){
        TreeNode temp = null;
        while(root!=null){
            if(root.left!=null){
                // connect threading for root
                temp = root.left;
                while(temp.right!=null && temp.right != root)
                    temp = temp.right;
                // the threading already exists
                if(temp.right!=null){
                    temp.right = null;
                    System.out.println(root.val);
                    root = root.right;
                }else{
                    // construct the threading
                    temp.right = root;
                    root = root.left;
                }
            }else{
                System.out.println(root.val);
                root = root.right;
            }
        }
    }




详细解释可见 Reference: http://www.cnblogs.com/AnnieKim/archive/2013/06/15/morristraversal.html


<br>

> 如有任何知识产权、版权问题或理论错误，还请指正。
>
> 转载请注明原作者及以上信息。
