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
