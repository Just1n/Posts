<!--
Title|Javascript数组的几个微妙点
Id|javascript-array-subtle
Date|2014-09-02 21:03:00
Status|Publish
Type|Post
Tags|javascript,tech
Excerpt|Array.prototype中的标准方法被设计成其他对象可服用的方法，即使这些对象并没有继承Array。许多这样的类数组对象接踵而至地出现在Javascript的不同地方。
-->
Array.prototype中的标准方法被设计成其他对象可服用的方法，即使这些对象并没有继承Array。许多这样的类数组对象接踵而至地出现在Javascript的不同地方。

一个非常好的例子就是arguments对象。arguments对象并没有继承Array.prototype，它看起来很像是一个数组，但它并不是一个数组对象，所以我们不能简单的通过调用arguments.forEach方法来遍历每一个参数。我们可以用call方法来指定Array.prototype.forEach的接收者：

    function highlight(){
        [].forEach.call(arguments,function(widget){
            widget.setBackground("yellow");
        });
    }
forEach方法是一个Function对象，这意味着它继承了Function.prototype中的call方法。这使得我们可以使用一个自定义的值作为forEach方法内部的this绑定来调用它，并紧随任意数量的参数。

那到底怎样使得一个对象“看起来像数组”呢？数组对象的基本契约一共有两个简单的规则：
- 具有一个范围在0到2^32 - 1的整型length属性。
- length属性大于该对象的最大索引。索引是一个范围在0到2^32 - 2的整数，它的字符串表示的是该对象的一个key。

那么这样看来我们就可以创建一个简单的类数组对象字面量：

    var arrayLike = {0: "a",1: "b",2: "c",length: 3};
    var result = Array.prototype.map.call(arrayLike,function(s){
        return s.toUpperCase();
    }); //["A","B","C"]

字符串也表现为不可变数组，因为它们是可索引的，并且其长度也可以通过length属性获取。因此，Array.prototype中的方法操作字符串时并不会修改原始数组。

    var result = Array.prototype.map.call("abc",function(s){
        return s.toUpperCase();
    }); //["A","B","C"]
    
数组还有另外两个需要注意的行为：
- 将length属性值设为小于n的值会自动地删除索引值大于或等于n的所有属性。
- 增加一个索引值为n（大于或等于length属性值）的属性会自动地设置length属性为n+1。

第二条行为尤其难以完成，因为它需要监控索引属性的增加以自动地更新length属性。幸运的是，对于使用Array.prototype中的方法，这两条规则都不是必须的，因为在增加或删除索引属性的时候它们都会强制地更新length属性。

只有一个Array方法不是完全通用的，即数组连接方法concat。该方法可以由任意的类数组接收者调用。如果参数是一个真实的数组，那么concat会将该数组的内容连接起来作为结果。否则，参数将以一个单一的元素连接。这意味着，例如，我们就不能简单地连接一个以arguments对象作为内容的数组。

    function namesColumn(){
        return ["Names"].concat(arguments);
    }
    namesColumn("Alice","Bob","Chris");// ["Names",{0: "Alice",1: "Bob",2: "Chris"}]
    
为了使concat方法将一个类数组对象视为真实的数组，我们不得不自己转换该数组。实现该转换的一个流行且简洁的惯用法是在类数组对象上调用slice方法。

    function namesColumn(){
        return ["Names"].concat([].slice.call(arguments));
    }
    namesColumn("Alice","Bob","Chris");// ["Names","Alice","Bob","Chris"]
    

最后一个关于数组对象上需要注意的是，**数组字面量优于数组构造函数**。

Javascript的优雅很大程度上要归功于其程序中最常见的构造块的简洁的字面量语法。字面量是一种表示数组的优雅的方法。

    var a = [1, 2, 3, 4, 5];
另外一种方式是用数组构造函数来替代：

    var a = new Array(1, 2, 3, 4, 5);
但是事实证明，即使抛开美学不谈，Array构造函数都存在一些微妙的问题。首先，你必须要确保，没有人重新包装过Array变量。

    function f(Array){
        return new Array(1, 2, 3, 4, 5);
    }
    f(String);// new String(1)
你还必须确保没有人修改过全局的Array变量。

    Array = String;
    new Array(1, 2, 3, 4, 5); // new String(1)
而且，你还要担心一种特殊的情况。如果使用单个数字参数来调用Array构造函数，效果完全不同。它会试图创建一个没有元素的数组，但其长度属性为给定的参数。这意味着`["Hello"]`和`new Array("Hello")`的行为相同，但是`[17]`和`new Array(17)`的行为完全不同！
