---
layout: post
title:  "Elements of Programming Interviews in Scala (Chapter 8)"
date:   2018-09-27 18:08:55 -0700
categories: jekyll update
---

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
