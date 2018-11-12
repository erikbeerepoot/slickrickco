---
layout: post
title:  "Elements of Programming Interviews in Scala (Chapter 9)"
date:   2018-11-12 11:00:00 -0700
categories: jekyll update
--- 

#### Problem 9.1 - Balanced Binary Tree

Given a binary tree, figure out if it is balanced. 

{% highlight scala %}
object App {
  def main(args: Array[String]): Unit = {
    test()
  }

  def isBalanced(tree: Node): Boolean = {
    math.abs(maxDepth(tree.left) - maxDepth(tree.right)) <= 1
  }

  def maxDepth(tree: Option[Node]): Int = {
    tree match {
      case None => -1 
      case Some(tree) => 1 + math.max(maxDepth(tree.left),maxDepth(tree.right))
    } 
  }
 
  def test(): Unit = {
    /*
     * 0     r 
     *      / \
     * 1 l .   . r
     */     
    val l = Node(None, None) 
    val r = Node(None, None)
    val tree = Node(Some(l),Some(r))
    assert(maxDepth(Some(tree)) == 1) 

    /**
     * 0         r
     *          /
     * 1     l . 
     *        /
     * 2  ll .        
     */
    val ll = Node(Some(l), None)
    val tree2 = Node(Some(ll), None) 
    assert(maxDepth(Some(tree2)) == 2)

    /**
     * 0      r 
     *       / \ 
     * 1  l .   . r
     *           \ 
     * 2          . rr
     *           / \
     * 3        .   . rrr
     */ 
    val rr = Node(Some(l), Some(r))
    val rrr = Node(None, Some(rr))
    val tree3 = Node(Some(l),Some(rrr))
    assert(maxDepth(Some(tree3)) == 3)

    /**
     * 0        r 
     *         / \ 
     * 1    l .   . r
     *       /     \ 
     * 2 ll .       . rr
     *             / \
     * 3          .   . rrr
     */ 
    val tree4 = Node(Some(ll), Some(rrr))
    assert(maxDepth(Some(tree4)) == 3)

    assert(isBalanced(tree))
    assert(isBalanced(tree2) == false)
    assert(isBalanced(tree3) == false)
    assert(isBalanced(tree4))
  }
}

sealed trait Tree
case class Node(var left: Option[Node], var right: Option[Node]) extends Tree

{% endhighlight %}

