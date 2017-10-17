---
title: Programming iOS 9 学习笔记-view实验
date: 2016-11-27 21:11:30
tags: ios，swift
---

>首先有几点需要说明的是：Programming iOS 9是一本很不错的书，纸质书很厚，看完真的是需要毅力的。我看过几次了，每次看的都是从第一个章节开始，过不了两张就不看了。汗死了，现在都是iOS10了，swift3都发布很久了，swift3相对前两个版本来说好很多，改动相当的大，所以说swift的每一个版本都可以算一个新语言。我最近把swift3的文档过了一遍（之前只看过swift1的，swift2没怎么看过）现在又拿出这个本书出来学习。

>这个笔记谈不上翻译，当然目前很大一部分是借鉴http://wdxtub.com/ 这个blog里的笔记【这个blog文章我常看】，我只是在上面作了一些修改。

>为什么这么说呢？因为我操作的环境是在swift3+Xcode8+iOS10。

>我只是业余的、业余的、业余的学习iOS开发，如果里面什么错误，欢迎指正！欢迎iOS大牛带我飞。


### View的实验

这里做几个简单的介绍和实验操作：

#### single View Application ：
一个single view application项目创建后，你将会看到一个包含一个页面的main storyboard 以及一个包含一个main view的view controller的实例。app运行起来，view controller将会成为app的main window的rootViewController，它的main view将会变成window的root view。

在ViewController.swift文件中修改viewDidLoaded()：
```swift
    override func viewDidLoad() {
        super.viewDidLoad()
        let mainview = self.view
        let v = UIView(frame: CGRect(x: 100, y: 200, width: 50, height: 50))
        v.backgroundColor = UIColor.blue   // small blue square
        mainview?.addSubview(v)    // add it to main view
    }
```

#### App without a main storyboard（ 这个例子我就省略了）


### Subview 和 Superview

在iOS中，一个的subview的一部分或者全部，可以出现在superview之外。一个view可以与另外一个view重叠，也可以在一个view前面绘制部分或全部。
![view-demo](https://raw.githubusercontent.com/yf92/yf92.github.io/master/images/ios/view-demo.png)

上图三个view都有一个背景颜色，每一个view都相当于是一个带有颜色的矩形。光看上面的图是没法区别这个三个view之间的关系，其实，中间的view（水平方向）是左边view的sibling view。右边的view是中view的一个subview。

 view hierarchy的一些特点：
* 一个view被移出，或者引入他的superview，它的subview也会被引入。
* 一个view的透明度会被其subview继承。
* 一个view可以限制subview的显示范围。比如不让subview超出view本身的范围，这个叫做clipping，被设置在view的clipsToBounds属性中。
* 一个superview拥有它的subview。
* 如果改变一个view的大小，那么它的subview也会被自动设置尺寸。

一个UIView有一个superview的属性（一个UIView）和一个subviews属性（一个UIView对象的数组，back-to-front的顺序），在代码中可以判断view hierarchy。有一个方法isDescendantOfView：用来检查一个view是否是另一个view的subview。View还有一个tag属性，有一个方法viewWithTag进行引用。

> 注意：这里的这两个func 在swift3中跟swift2有点不一样： 
```swift
	func isDescendantOfView(view: UIView) -> Bool  
	func viewWithTag(tag: Int) -> UIView?
```

在代码中操作view hierarchy是非常简单的！

> addSubview方法添加一个subview。比如：view.addSubview(button)。
> removeFromSuperview方法移除一个subview。比如：xxxView.removeFromSuperview();

注意从superview中移除subview同时也会释放它。如果以后需要重用的话，最好先确定能够把它保存在内存中，通常的方法是把这个view保存在一个属性中。

在进行这些操作时系统也会给出通知，重写下列方法就可以根据需要在不同的情况下进行不同的操作：

* didAddSubview, willRemoveSubview
* didMoveToSuperview, willMoveToSuperview
* didMoveToWindow, willMoveToWindow

当调用了addSubview，这个view也会被放到其superview的subview数组中的最后一个，也就是说这个view会被最后一个渲染出来。如果view的subviews是被索引的，从0开始（rearmost），这样就可以把view插入到指定位置，已经放到前面/后面，或两个view进行交互。

* swift3：// Insert subview at specific position
insertSubview(_:at:)
 比如: 
let f1 = someView.insertSubview(view: at:)
* swift3：// Insert subview above/below a specific subview
insertSubview(_:aboveSubview:)
insertSubview(_:belowSubview:)
比如：
let f2 = someView.insertSubview(view: aboveSubview:)
let f3 = someView.insertSubview(view: belowSubview:)
* swift3: // Exchange two subview's position
exchangeSubview(at:withSubviewAt:)
* swift3：// Bring the subview to the front or back of its sibling
bringSubview(toFront:)
sendSubview(toBack:)

看到这里，你可能会感觉有点奇怪，没有一个方法可以直接移除一个superview的所有subview。然后，因为一个superview的subivew数组是一个不可变的数组，遍历subiviews数组，然后依次执行removeFromSuperview()就大差不差了！比如：
```swift
rootView.subviews.forEach { $0.removeFromSuperview() }
```


### Visibility and Opacity(可见度和不透明)

一个view的可见度是可以通过hidden属性来设置的。隐藏一个view（包括subview）只是从界面上隐藏，而不需要从view hierarchy结构中删除。一个隐藏的view不能接受触摸的事件。（用户看不到而已，其实是存在的）

backgroundColor属性可以设置view的背景颜色，颜色是属于UIColor类，颜色背景为nil的view，默认是具有透明背景的。

可以通过设置 view 的 alpha 属性来修改透明程度，1.0 是完全不透明，0.0 是透明。假设一个 view 的 alpha 是 0.5，那么它的 subview 的 alpha 都是以 0.5 为基准的，不可能高于 0.5。而 UIColor 也有 alpha 这个属性，所以即使一个 view 的 alpha 是 1.0，它仍旧可能是透明的，因为其 backgroundColor 可以是透明的。一个 alpha 为 0.0 的 view 是完全透明的所以是不可见的，通常来说也不可能被点击。

View 的 alpha 属性不仅影响背景颜色，也会影响其内容的透明度。

view的opaque属性的修改不会影响view的样子，更多的是对于系统绘制的提示。如果一个view的opaque设置true，因为不用考虑透明的绘制，所以效率高一点。并且再设置透明的背景颜色或者 alpha 属性都无效。可能会让人吃惊，它的默认值是 true。


### Frame

view的frame属性（CGRect类）是他本身的长方形在superview中位置，注意是在superview的坐标系中国的位置。默认来说，superview的坐标系圆点在坐上，向右x增加，向下y增加。

给view的frame设置不同的CGRect属性值，能够改变view的view的位置或者改变尺寸大小（同时改变也可以）。

现在我们画上图大差不差的图：我们在AppDelegate.swift中func application()增加下面几行代码：
```swift
        //here we can add subviews
        let mainview = self.window!.rootViewController!.view

        let v1 = UIView(frame: CGRect(x: 113, y: 111, width: 132, height: 194))
        v1.backgroundColor = UIColor(red: 1, green: 0.4, blue: 1, alpha: 1)
        
        let v2 = UIView(frame: CGRect(x: 41, y: 56, width: 132, height: 194))
        v2.backgroundColor = UIColor(red: 0.5, green: 1, blue: 0, alpha: 1)
        
        let v3 = UIView(frame: CGRect(x: 43, y: 197, width: 160, height: 230))
        v3.backgroundColor = UIColor(red: 1, green: 0, blue: 0, alpha: 1)
        
        mainview?.addSubview(v1)
        v1.addSubview(v2)
        mainview?.addSubview(v3)
```

效果图如下：（v2是添加在v1上）

![view-ios](https://raw.githubusercontent.com/yf92/yf92.github.io/master/images/ios/view-ios10-1.png)






