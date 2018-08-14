---
layout: post
category: offer
title: binary-tree-preorder-traversal
---

## title
[problem link](https://www.nowcoder.com/practice/501fb3ca49bb4474bf5fa87274e884b4?tpId=46&tqId=29036&rp=1&ru=/ta/leetcode&qru=/ta/leetcode/question-ranking)

Given a binary tree, return the preorder traversal of its nodes' values.

For example:
Given binary tree{1,#,2,3},

	   1
	    \
	     2
	    /
	   3

return[1,2,3].

Note: Recursive solution is trivial, could you do it iteratively?

## solution


```java
/**
 * Definition for binary tree
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
import java.util.*;
public class Solution {
    public ArrayList<Integer> preorderTraversal(TreeNode root) {
        ArrayDeque<TreeNode> stack=new ArrayDeque<>();
        if(root==null) return new ArrayList<>();
        TreeNode p=root;
        ArrayList<Integer> res=new ArrayList<>();
        while (p!=null||!stack.isEmpty()){
            while (p!=null){
                res.add(p.val);
                stack.push(p);
                p=p.left;
            }
            if(!stack.isEmpty()){
                p=stack.peek().right;
                stack.pop();
            }
           // System.out.print("1");
        }
        return res;
    }
}

```