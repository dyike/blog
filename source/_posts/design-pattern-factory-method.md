---
title: 设计模式之“工厂方法模式”——计算器实例
date: 2017-09-12 22:44:06
tags: 设计模式
---

### 什么是工厂模式？

就是定义一个创建对象的接口，让子类是实例化具体的类，工厂方法就是让类的实例化，延迟到子类中。这是属于创建类模式。

### 怎么实现？

用模板方法的方式创建对象来解决，父类定义所有标准通用行为，然后将创建细节放在子类中实现并输出给客户端。工厂方法模式主要四个要素：

* 工厂接口。工厂接口是工厂方法模式的核心，与调用者直接交互用来提供产品。
* 工厂实现。工厂实现决定如何实例化产品，是实现扩展的途径，需要多少种产品，就需要多少具体工厂实现。
* 产品接口。产品接口的主要目的是定义产品的规范，所有的产品实现都必须遵循产品接口定义的规范。
* 产品实现。实现产品接口的具体类，决定了产品在客户端中的具体行为。

### 使用场景：

工厂模式是一种典型的解耦模式，当需要系统有比较好的扩展性时，可以考虑工厂模式，不同的产品用不同的实现工厂来组装。


### 实例：

```swift
import UIKit

// 协议
protocol Operator {
    var num: (Double, Double) {
        get
        set
    }
    func getResult() -> Double?
    // 工厂
    static func createOperation() -> Operator
}

// 遵守协议
struct Addition: Operator {
    var num = (0.0, 0.0)
    
    func getResult() -> Double? {
        return num.0 + num.1
    }
    
    static func createOperation() -> Operator {
        return Addition()
    }
}

struct Subtraction: Operator {
    var num = (0.0, 0.0)
    
    func getResult() -> Double? {
        return num.0 - num.1
    }
    
    static func createOperation() -> Operator {
        return Subtraction()
    }
}


struct Multiplication: Operator {
    var num = (0.0, 0.0)
    
    func getResult() -> Double? {
        return num.0 * num.1
    }
    
    static func createOperation() -> Operator {
        return Multiplication()
    }
}

struct Division: Operator {
    var num = (0.0, 0.0)
    
    func getResult() -> Double? {
        var result: Double?
        if num.1 != 0 {
            result = num.0 / num.1
        }
        return result
    }
    static func createOperation() -> Operator {
        return Division()
    }
}

var testSubtraction = Subtraction.createOperation()
testSubtraction.num = (5, 2)
print(testSubtraction.getResult())

var testMultiplication = Multiplication.createOperation()
testMultiplication.num = (2, 3)
print(testMultiplication.getResult())

```



