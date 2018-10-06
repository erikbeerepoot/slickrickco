---
layout: post
title:  "Elements of Programming Interviews in Scala (Chapter 8)"
date:   2018-09-27 18:08:55 -0700
categories: jekyll update
---

#### Problem 8.1
Design a stack that supports a max operation. Time complexity must be O(1), memory complexity O(n).

**Solution**

Use a second stack to keep track of the maximum value, operate on it when operating on the other stack. Since stacks have O(1) time complexity, two stacks will still have O(1) time complexity. 

{% highlight scala %}
class MaxStack[T <% Ordered[T]] {
  val stack = scala.collection.mutable.Stack[T]()
  val maxStack = scala.collection.mutable.Stack[T]()

  def push(value: T): Unit = {
    stack.push(value)
    if(maxStack.isEmpty){
      maxStack.push(value)
    } else if(maxStack.top <= value){
      maxStack.push(value)
    }
  }

  def pop(): T = {
    if(stack.isEmpty){
      throw new RuntimeException("Called pop on an empty stack")
    }

    val popped = stack.pop()
    if(maxStack.top == popped){
      maxStack.pop()
    }
    popped
  }

  def max: T = {
    if(maxStack.isEmpty){
      throw new RuntimeException("The maximum of an empty stack is not defined!")
    }
    maxStack.top
  }
}

object MaxStackTest {
  val testStack = new MaxStack[Int]

  def main(args: Array[String]): Unit = {
    testStack.push(1)
    testStack.push(2)
    assert(testStack.max == 2)
    testStack.push(3)
    testStack.push(4)
    testStack.push(4)
    testStack.push(3)
    assert(testStack.max == 4)
    testStack.pop()
    assert(testStack.max == 4)
    testStack.pop()
    testStack.pop()
    assert(testStack.max == 3)
    testStack.pop()
    assert(testStack.max == 2)
    testStack.pop()
    assert(testStack.max == 1)
    testStack.pop()
  }
}
{% endhighlight %}

#### Problem 8.2
Write a function that takes an arithmetic expression in Reverse Polish Notation and evaluates it.

{% highlight scala %}
object ReversePolishNotation {
  val stack = scala.collection.mutable.Stack[Int]()
  def main(args: Array[String]): Unit = {
    assert(reversePolish("1,2,+")==3)
    assert(reversePolish("1,2,+,4,5,x,+")==23)
    assert(reversePolish("3,4,+,3,4,+,+")==14)
  }

  def reversePolish(operand: String): Int = {
    for(character <- operand){
      if(character.isDigit){
        stack.push(character.toInt - '0'.toInt)
      } else if(isOperator(character)){
        performOperation(character)
      }
    }
    stack.pop
  }

  def isOperator(character: Char) = character match {
      case 'x' | '/' | '-' | '+' => true
      case _ => false
    }
  }

  def performOperation(operation: Char) = {
    if(stack.isEmpty){
      throw new RuntimeException(
        "Attempted to perform an operation with no operands!"
      )
    }
    val a = stack.pop
    val b = stack.pop
    val result = operation match {
      case 'x' => a * b
      case '/' => a / b
      case '-' => a - b
      case '+' => a + b
    }
    stack.push(result)
  }
}
{% endhighlight %}	


#### Problem 8.5 
Write a function that lists a sequence of operations to solve the Tower of Hanoi. Manual working of the solution below. For convenience, I've labelled the rings with a number. The constraint that we can't put a larger ring on a smaller ring can than easily be expressed numerically.

```
1
2
3
4
5
6 . . 

2
3
4
5
6 1 .                   
1 operation 

2   
3    3        3
4        4      4
5      5        5   1
6 1 . -> 6 1 2 -> 6 . 2         
2 operations


                                    1        1
3                          2        2        2        2
4        4        4        4        4        4        4        4        4   1
5   1    5   1    5 1      5 1      5        5        5        5   2    5   2
6 . 2 -> 6 3 2 -> 6 3 2 -> 6 3 . -> 6 3 . -> 6 . 3 -> 6 1 3 -> 6 1 3 -> 6 . 3 
 
3         1        1
4      4        4        4        4 1
5   1    5   1    5        5 2      5 2 
6 . 2 -> 6 3 2 -> 6 3 2 -> 6 3 . -> 6 3 .
4 operations 

                    1        1                 1 
4 1        1               2        2        2         2            2        2 
5 2    5 2      5 2 1    5   1    5        5   3     5   3    5   3    5   3 
6 3 . -> 6 3 4 -> 6 3 4 -> 6 3 4 -> 6 3 4 -> 6 . 4  -> 6 1 4 -> 6 1 4 -> 6 . 4    
8 operations  

    1        1                                                          1        1
    2        2        2               1        1               2        2        2        2 1
5   3        3    1   3    1 2 3      2 3    3 2      3 2 1    3   1    3        3 4      3 4 
6 . 4 -> 6 5 4 -> 6 5 4 -> 6 5 4 -> 6 5 4 -> 6 5 4 -> 6 5 4 -> 6 5 4 -> 6 5 4 -> 6 5 . -> 6 5 . -> 

                                               1
                                      2        2
  1                 3        3        3        3
3 4      3 4 1      4 1    1 4      1 4        4
6 5 2 -> 6 5 2 -> 6 5 2 -> 6 5 2 -> 6 5 . -> 6 5 .
16 operations
etc.


```

The solution is:
1. Recursively transfer n - 1 rings from P1 to P3 using P2.
2. Transfer ring n - 1 from P1 to P2 
3. Recursively transfer n - 1 rings from P3 to P2 using P1.


In code:
{% highlight scala %}
object TowersOfHanoiTest {
  def main(args: Array[String]): Unit = {
    val towers = new TowersOfHanoi(6)
    towers.solve()
  }
}

class TowersOfHanoi(numberOfRings: Int) {
  val pegs = List[scala.collection.mutable.Stack[Int]](
    scala.collection.mutable.Stack[Int](),
    scala.collection.mutable.Stack[Int](),
    scala.collection.mutable.Stack[Int]()
  )

  def initialize: Unit = {
    (0 to 2).foreach(peg => pegs(peg).clear())
    (6 to 1 by -1).foreach(num => pegs(0).push(num))
  }

  def solve(): Unit = {
    initialize
    transfer(numberOfRings,0,1,2)
  }

  def transfer(n: Int, from: Int, to: Int, use: Int){
    print()
    if(n > 0){
      transfer(n - 1, from, use, to)
      moveRing(from,to)
      transfer(n - 1, use, to, from)
    }
  }

  def moveRing(from: Int, to: Int){
    val popped = pegs(from).pop()
    pegs(to).push(popped)
  }

  def print(): Unit = {
    println(pegs(0).toString)
    println(pegs(1).toString)
    println(pegs(2).toString)
    println("-----------")
  }
}
{% endhighlight %}  


