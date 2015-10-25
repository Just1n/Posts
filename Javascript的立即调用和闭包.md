<!--
Title|Javascript的立即调用和闭包
Id|javascript-IIFE-and-closure
Date|2015-08-17 22:28:00
Status|Publish
Type|Post
Tags|javascript,tech
Excerpt|Javascirpt有三个特性：1、Javascript允许你引用在当前函数以外定义的变量。2、即使外部函数已经返回当前函数仍然可以引用在外部函数所定义的变量。3、闭包可以更新外部变量的值。
-->
先看一段简单的javascript代码：

    function warpEle(a){
        var result = [], i, n;
        for(i = 0, n = a.length; i < n; i++){
            result[i] = function() { return a[i]; };
        }
        return result;
    }
    
    var warpped = warpEle([10,20,30,40,50]);
    var f = warpped[0];
    f();
很多人可能会觉得这段代码最终输出结果是10，但实际上它输出的是`undefined`值。

要理解这段代码得和闭包结合起来看。

Javascirpt有三个特性：1、Javascript允许你引用在当前函数以外定义的变量。2、即使外部函数已经返回当前函数仍然可以引用在外部函数所定义的变量。3、闭包可以更新外部变量的值。

上述代码很符合第1和第2个特性，第3个特性讲的是实际上闭包所存储的外部变量并不是他们的值的副本，而是他们的引用。在该段代码中，在`for`循环体内，`result[i]`其实就是一个闭包，在这个函数内部访问了闭包外部的变量`i`，但是由于闭包存储的是变量的引用，在函数`warpEle`函数return以后，变量`i`的值依然变成了5，所以当调用`f()`的时候，就是寻找`a[5]`，这个时候没有数组`a`并没有第六个值，所以会抛出`undefined`。

要解决这个问题也很简单，就是使用Javascript**立即调用**的特性，把代码稍微修改一下即可。

    function warpEle(a){
        var result = [], i, n;
        for(i = 0, n = a.length; i < n; i++){
            (function(j){
                result[i] = function() { return a[j] ;};
            })(i);
        }
        return result;
    }
这时候再调用函数结果的输出就符合预期啦。
