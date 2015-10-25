<!--
Title|Javascript的对象和原型
Id|javascript-object-and-prototype
Date|2015-08-30 14:20:00
Status|Publish
Type|Post
Tags|javascript,tech
Excerpt|对象是Javascript中的基本数据结构。直观地看，对象表示字符串与值映射的另一个表格。Javascript支持继承，但不像许多传统的语言，它的继承机制基于原型而不是类。在Javascript中对象是从其他对象中继承而来。每个对象与其他一些对象是相关的，这些对象称为它的原型。
-->
对象是Javascript中的基本数据结构。直观地看，对象表示字符串与值映射的另一个表格。Javascript支持继承，但不像许多传统的语言，它的继承机制基于原型而不是类。在Javascript中对象是从其他对象中继承而来。每个对象与其他一些对象是相关的，这些对象称为它的原型。

###`prototype`、`getPrototypeOf`和`__proto__`之间的不同

 - `C.prototype`用于建立由`new C()`创建的对象的原型
 - `Object.getPrototypeOf(obj)`是ES5中用来获取obj对象的原型对象的标准方法。
 - `obj.__proto__`是获取obj对象的原型对象的非标准方法

假设有User这样的一个类型：

    function User(name,passwordHash){
	    this.name = name;
	    this.passwordHash = passwordHash;
	}
	User.prototype.toString = function(){
		return "[User " + this.name + "]"
	}
	User.prototype.checkPassword = function(password){
		return hash(password) === this.passwordHash;
	}
	var u = new User("Just1n","0ef33ae790........");

User函数带有一个默认的prototype属性，其包含一个开始几乎为空的对象。我们添加了两个方法到`User.prototype`对象:`toString`和`checkPasswor`。

`getPrototypeOf`是ES5中的函数，可以用于检索现有对象的原型。我们创建了上述例子中的`u`对象以后，下面这段代码返回的就是true：

    Object.getPrototypeOf(u) === User.prototype; //true
`__proto__`属性是在某些环境下的非标准检索对象的原型的方法。这可以作为在不支持ES5的`Object.getPrototypeOf`方法的环境中的一个权宜之计。在这些环境中，下面这句代码也是返回`true`：

    u.__proto__ === User.prototype; //true

###尽量使用`Object.getPrototypeOf`函数而不要使用`__proto__`属性
ES5引入`Object.getPrototypeOf`函数作为获取对象原型的标准API，但在这之前大量的Javascript引擎早就使用一个特殊的`__proto__`属性来达到相同的目的。然而并不是所有的Javascript环境都支持通过`__proto__`属性来获取对象的原型，因此该属性并不是完全兼容的。例如，对于拥有`null`原型的对象，不同的环境处理得就不一样。在一些环境中，`__proto__`属性继承自Object.prototype，因此拥有null原型的对象没有这个特殊的`__proto__`属性。

    var empty = Object.create(null);
    "__proto__" in empty; //false
而其他的环境总是特别的处理`__proto__`属性而不管对象的状态。

    var empty = Object.create(null);
    "__proto__" in empty; //true
无论在什么情况下，`Object.getPrototypeOf`函数都是有效的，而且它也是提取对象原型更加标准、可移植的方法。而且`__proto__`属性会污染所有的对象，因此它会导致大量的Bug。对于那些没有提供标准ES5 API的Javascript环境，可以轻易的用`__proto__`属性来实现`Object.getPrototypeOf`函数：

    if(typeof Object.getPrototypeOf === "undefined"){
	    Object.getPrototypeOf = function(obj){
		    var t typeof obj;
		    if(!obj || (t !== "Object" && t !== "function")){
			    throw new TypeError("not an object");
		    }
		    return obj.__proto__;
	    }
    }

###始终不要修改`__proto__`属性
`__proto__`属性很特殊，它提供了`Object.getPrototypeOf`方法所不具备的额外的能力，即修改对象原型链的能力。这种能力实际上上会造成严重的影响，应当避免使用。避免修改`__proto__`属性的最明显的原因是可移植性问题。因为并不是所有的平台都支持改变对象原型的特性，所以根本无法编写可移植的代码。

避免修改`__proto__`属性的另一个原因是性能问题。所有现代的Javascript引擎都深度优化了获取和设置对象属性的行为，因为这些都是最常见的Javascript程序的操作。这些优化都是基于引擎对对象结构的认识上。当更改了对象的内部结构（如添加和删除该对象或其原型链中的对象的属性），将会使一些优化失效。修改`__proto__`属性实际上改变了继承结构本身，这可能是最具破坏性的修改。比起普通的属性修改，修改`__proto__`属性会导致更多的优化失效。

###构造函数与`new`操作符的关系
当使用上述User函数构造一个新的对象时，程序依赖于调用者是否记得使用new操作符来调用该构造函数。如果调用者忘记使用new关键字，那么函数的接收者将是全局对象。

    var u = User("Just1n","d8b74.......");
    u;   // undefined
    this.name; // "Just1n"
    this.passwordHash; // "d8b74......."
该函数不但会返回无意义的undefined，而且会灾难性地创建全局变量name和passwordHash。
User函数是脆弱的，当和new操作符一同使用时，它能按预期的工作，然而当将它作为一个普通的函数调用时便会失败。一个更为健壮的方式是提供一个不管怎样调用都工作如构造函数的函数。实现该函数的一个简单方法是检查函数的接收者是否是一个正确的User实例。

    function User(name,passwordHash){
	    if(!(this instanceof User)){
		    return new User(name,passwordHash);
	    }
	    this.name = name;
	    this.passwordHash = passwordHash;
    }
使用这种方式，不管是以函数的额方式还是以构造函数的方式调用User的函数，它都返回一个继承自User.prototype的对象。

    var x = User("Just1n","d8b74.....");
    var y = new User("Just1n","d8b74.....");
    x instanceof User; //true
    y instanceof User; //true
这种模式的一个缺点是它需要额外的函数调用，因此代价有点高。而且它也很难适用于可变参数函数，因为没有一种直接模拟apply方法将可变参数函数作为构造函数调用的方式。还有另外一种利用ES5的Object.create函数的方式：

    function User(name,passwordHash){
	    var self = this instanceof User
				    ? this
				    : Object.create(User.prototype);
		self.name = name;
		self.passwordHash = passwordHash;
		reurn self;
    }
Object.create需要一个原型对象作为参数，并返回一个继承自该原型对象的新对象。因此当以函数的方式调用该版本的User函数时，结果将返回一个继承自User.prototype的新对象，并且该对象具有已经初始化的name和passwordHash属性。

###在原型中存储方法
Javascript完全又可能不借助原型进行编程。我们可以实现前面User类，而不用在其原型中定义任何特殊的方法。

    function User(name,passwordHash){
	    this.name = name;
	    this.passwordHash = passwordHash;
	    this.toString = function(){
		    return "[User " + this.name + "]";
	    };
	    this.checkPassword = function(password){
			return hash(password) === this.passwordHash;    
		};
	}
大多数情况下，这个类的行为与其原始实现版本几乎是一样的。但当我们多构造几个User
实例时，一个重要的区别就暴露出来了。

    var u1 = new User(....);
    var u2 = new User(....);
    var u3 = new User(....);
这三个实例每个实例都包含了toSting和checkPassword方法的副本，而不是通过原型共享的方法。而如果放在原型中实现着两个方法，那么它们就只被创建了一次。实例方法要比原型方法占用更多的内存。
