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
     * 1    ll . 
     *        /
     * 2   l .        
     */
    val ll = Node(Some(l), None)
    val tree2 = Node(Some(ll), None) 
    assert(maxDepth(Some(tree2)) == 2)

    /**
     * 0      r 
     *       / \ 
     * 1  l .   . rrr
     *           \ 
     * 2          . rr
     *           / \
     * 3      l .   . r
     */ 
    val rr = Node(Some(l), Some(r))
    val rrr = Node(None, Some(rr))
    val tree3 = Node(Some(l),Some(rrr))
    assert(maxDepth(Some(tree3)) == 3)

    /**
     * 0        r 
     *         / \ 
     * 1  ll .   . rrr
     *       /     \ 
     * 2  l .       . rr
     *             / \
     * 3        l .   . r
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

#### Problem 9.2 - k balanced trees 

Design an algorithm that finds nodes in a binary tree that meet the following condition:
(1) The node itself is not k-balanced.
(2) All its descendants *are* k-balanced.

{% highlight scala %}

object App {
  def main(args: Array[String]): Unit = {
    test()
  }

  def isKBalanced(tree: Option[Node], k: Int): Boolean = {
    tree match {
      case None => true 
      case Some(t) => math.abs(countNodes(t.left) - countNodes(t.right)) <= k 
    }
  }

  def allDescendantsKBalanced(tree: Option[Node], k: Int): Boolean = {
    tree match {
      case None => true 
      case Some(t) => 
        isKBalanced(t.left, k) && 
        isKBalanced(t.right, k) && 
        allDescendantsKBalanced(t.left, k) && 
        allDescendantsKBalanced(t.right,k)
    }
  }

  def countNodes(tree: Option[Node]): Int = {
    tree match {
      case None => 0  
      case Some(t) => 1 + countNodes(t.left) + countNodes(t.right) 
    }
  }

  def searchForNode(tree: Option[Node], k: Int): Unit = {
    tree match {
      case None => 
      case Some(t) => {
        if(!isKBalanced(tree, k) && allDescendantsKBalanced(tree, k)){
          println(s"Found node matching condition: ${t.label}")
        } else {
          searchForNode(t.left, k)
          searchForNode(t.right, k)
        }
      }  
    }
  }
 
  def test(): Unit = {
    /*
     * 0     r 
     *      / \
     * 1 l .   . r
     */     
    val l = Node(None, None,"l") 
    val r = Node(None, None,"r")
    val tree = Node(Some(l),Some(r),"tree")
    assert(countNodes(Some(tree)) == 3)
    assert(isKBalanced(Some(tree),1))
    /**
     * 0         r
     *          /
     * 1    ll . 
     *        /
     * 2   l .        
     */
    val ll = Node(Some(l), None, "ll")
    val tree2 = Node(Some(ll), None, "tree2") 
    assert(countNodes(Some(tree2)) == 3)
    assert(isKBalanced(Some(tree2),1) == false)
    /**
     * 0      r 
     *       / \ 
     * 1  l .   . rrr
     *           \ 
     * 2          . rr
     *           / \
     * 3      l .   . r
     */ 
    val rr = Node(Some(l), Some(r), "rr")
    val rrr = Node(None, Some(rr), "rrr")
    val tree3 = Node(Some(l),Some(rrr), "tree3")
    assert(countNodes(Some(tree3)) == 6)
    assert(isKBalanced(Some(tree3),1) == false)
    assert(isKBalanced(Some(rrr),3))
    assert(isKBalanced(Some(rrr),1) == false)
    assert(isKBalanced(Some(rr),1))

    /**
     * 0        r 
     *         / \ 
     * 1   ll .   . rrr
     *       /     \ 
     * 2  l .       . rr
     *             / \
     * 3        l .   . r
     */ 
    val tree4 = Node(Some(ll), Some(rrr), "tree4")
    assert(countNodes(Some(tree4)) == 7)
    assert(isKBalanced(Some(tree4),2))
    assert(isKBalanced(Some(tree4),1) == false)
    assert(allDescendantsKBalanced(Some(r), 1) == true)
    /**
     * 0         r 
     *          / \ 
     * 1   lll .   . rrr
     *        /     \ 
     * 2  l2 .       . rr
     *      / \     / \
     * 3   l   r l .   . r
     */ 
    val l2 = Node(Some(l), Some(r), "l2")
    val lll = Node(Some(l2), None, "lll")
    val tree5 = Node(Some(lll), Some(rrr), "tree5")
    assert(allDescendantsKBalanced(Some(rr),1))
    assert(allDescendantsKBalanced(Some(rrr),1))
    assert(allDescendantsKBalanced(Some(lll),1))
    assert(allDescendantsKBalanced(Some(tree5),1) == false) 

    searchForNode(Some(tree5), 1)
  }
}

sealed trait Tree
case class Node(var left: Option[Node], var right: Option[Node], label: String) extends Tree

{% endhighlight %}

