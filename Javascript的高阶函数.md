<!--
Title|Javascript的高阶函数
Id|js-higher-order-func
Date|2015-08-24 21:43:00
Status|Publish
Type|Post
Tags|javascript,tech
Excerpt|所谓高阶函数其实就是那些将函数作为参数或返回值的函数。将函数作为参数是一种特别强大而富有表现力的惯用法，该函数通常也成为回掉函数，因为高阶函数会“随后调用”它，这在node里是最常见的。
-->
所谓高阶函数其实就是那些将函数作为参数或返回值的函数。将函数作为参数是一种特别强大而富有表现力的惯用法，该函数通常也成为回掉函数，因为高阶函数会“随后调用”它，这在node里是最常见的。

Javascript有很多元素的函数都属于高阶函数，比如数组的sort方法：

    function compareNumbers(x,y){
        if(x < y){
            return -1;
        }
        if(x > y){
            return 1;
        }
        return 0;
    }
    [3,1,4,1,5,9].sort(compareNumbers);//[1,1,3,4,5,9]
    
学会使用高阶函数通常可以简化代码并消除繁琐的样板代码。如果在代码中出现大量重复或者相似的地方，那么就很有可能可以用高阶函数来代替。假设我们发现程序的部分代码使用英文字母构造一个字符串：

    var aIndex = 'a'.charCodeAt(0); //97
    
    var alphabet = "";
    for(var i = 0; i < 26; i++){
        alphabet += String.fromCharCode(aIndex + i)
    }
    alphabet; // "abcdefghijklmnopqrstuvwxyz"
同时，还有另一部分代码生成一个包含数字的字符串:

    var digits = "";
    for(var i = 0; i < 10; i++){
        digits += i;
    }
    digits; //"0123456789"
此外，程序的其他地方还存在创建一个随机的字符串：

    var random = "";
    
    for(var i = 0; i < 8; i++){
        random += String.fromCharCode(Math.floor(Math.random()*26) + aIndex);
    }
    random; //"qejkoitv"
可以看到每个例子之间都用共同的逻辑。每个循环都是通过连接每个独立部分的积算结果来创建一个字符串。我们可以提取出共用的部分，将它们移到单个工具函数里：

    function buildString(n, callback){
        var result = "";
        for(var i = 0; i < n; i++){
            result += callback(i);
        }
        return result;
    }
buildString函数的实现包含了每个循环的所有共用部分，并使用参数来替代变化的部分。循环迭代的次数由变量n来替代，每个字符串片段的构造由callback函数替代。那么以上三个例子就可以简化如下：

    var alphabet = buildString(26, function(i){
        return String.fromCharCode(aIndex + i);
    });
    
    var digits = buildString(10, function(i){
        return i;
    });
    
    var random = buildString(8, function(){
        return String.fromCharCode(Math.floor(Math.random() * 26) + aIndex);
    });
创建高阶函数抽象有很多好处。实现中存在的一些棘手部分，比如正确地获取循环边界条件，它们可以被放置在高阶函数的实现中。这使得你可以一次性地修复所有逻辑上的错误，而不必去搜寻散布在程序中的该编码模式的所有实例。
