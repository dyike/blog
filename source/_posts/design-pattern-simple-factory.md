---
title: 设计模式之”简单工厂模式“——计算器示例
date: 2017-09-11 22:30:12
tags: 设计模式
---

学习设计模式，其实跟语言是无关的，我们主要是学习其中的思想。今天换一个语言来实现吧，主要好久没有写swift了，今天就拿swift来写好了。

还是先来说说概念吧，简单工厂模式（simple factory pattern）又称为静态工厂方法模式，是属于创建型模式。可以根据参数的不同返回不同类的实例，说通俗一点就是一家工厂，能够生产轮胎，还能生产齿轮等等，只需要跟工厂说一声（传参数），就能生产出来了。一般情况下被创建的实例都是具有共同的父类。

什么时候使用？比如在计算器中，有加法运算，有减法运算，有乘法运算，有除法运算等等。各个运算都是一种运算操作（operator）,我们只是修改部分属性从而让他们具备了不同的运算能力。如果我们希望我们在使用这个计算器的时候，我们不需要知道具体是怎么计算的，就只需要得到正确的结果，此时，不是不就可以使用简单工厂模式。

下面直接看源码：

```swift
import UIKit

// 协议
protocol Operator {
    var num: (Double, Double) {
        get
        set
    }
    
    func getResult() -> Double?
}

// 遵循此协议
struct Addition: Operator {
    var num = (0.0, 0.0)
    
    func getResult() -> Double? {
        return num.0 + num.1
    }
}

struct Subtraction: Operator {
    var num = (0.0, 0.0)
    
    func getResult() -> Double? {
        return num.0 - num.1
    }
    
}


struct Multiplication: Operator {
    var num = (0.0, 0.0)
    
    func getResult() -> Double? {
        return num.0 * num.1
    }
}


struct Division: Operator {
    var num = (0.0, 0.0)
    
    func getResult() -> Double? {
        var result: Double?
        if num.1 != 0 {
            return num.0 / num.1
        }
        return result
    }
}


// 操作符枚举
enum Operators {
    case addition, subtraction, multiplication, divsion
}


// 工厂 
struct OperatorFactory {
	  // 计算操作
    static func calculateForOperator(_ opt: Operators) -> Operator {
        switch opt {
        case .addition:
            return Addition()
        case .subtraction:
            return Subtraction()
        case .multiplication:
            return Multiplication()
        case .divsion:
            return Division()
        }
    }
}


var testDivision = OperatorFactory.calculateForOperator(.divsion)
testDivision.num = (1, 3)
print(testDivision.getResult() ?? "Error")

var testAddition = OperatorFactory.calculateForOperator(.addition)
testAddition.num = (2, 3)
print(testAddition.getResult() ?? "Error")

```

