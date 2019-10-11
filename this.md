**JavaScript 语言中的 this**
===
<font size='4'>
由于其运行期绑定的特性，JavaScript 中的 this 含义要丰富得多，它可以是全局对象、当前对象或者任意对象，这完全取决于函数的调用方式。JavaScript 中函数的调用有以下几种方式：作为对象方法调用，作为函数调用，作为构造函数调用，和使用 apply 或 call 调用。下面我们将按照调用方式的不同，分别讨论 this 的含义
</font>

作为对象方法调用
===
```
var point = {
x : 0, 
y : 0, 
moveTo : function(x, y) { 
    this.x = this.x + x; 
    this.y = this.y + y; 
    } 
}; 
point.moveTo(1, 1)//this 绑定到当前对象，即 point 对象
```
作为函数调用
===
函数也可以直接被调用，此时 this 绑定到全局对象。在浏览器中，window 就是该全局对象。比如下面的例子：函数被调用时，this 被绑定到全局对象，接下来执行赋值语句，相当于隐式的声明了一个全局变量，这显然不是调用者希望的。
```function makeNoSense(x) { 
this.x = x; 
} 
 
makeNoSense(5); 
x;// x 已经成为一个值为 5 的全局变量
```

对于内部函数，即声明在另外一个函数体内的函数，这种绑定到全局对象的方式会产生另外一个问题。我们仍然以前面提到的 point 对象为例，这次我们希望在 moveTo 方法内定义两个函数，分别将 x，y 坐标进行平移。结果可能出乎大家意料，不仅 point 对象没有移动，反而多出两个全局变量 x，y。

```
var point = { 
x : 0, 
y : 0, 
moveTo : function(x, y) { 
    // 内部函数
    var moveX = function(x) { 
    this.x = x;//this 绑定到了哪里？
   }; 
   // 内部函数
   var moveY = function(y) { 
   this.y = y;//this 绑定到了哪里？
   }; 
 
   moveX(x); 
   moveY(y); 
   } 
}; 
point.moveTo(1, 1); 
point.x; //==>0 
point.y; //==>0 
x; //==>1 
y; //==>1
```
这属于 JavaScript 的设计缺陷，正确的设计方式是内部函数的 this 应该绑定到其外层函数对应的对象上，为了规避这一设计缺陷，聪明的 JavaScript 程序员想出了变量替代的方法，约定俗成，该变量一般被命名为 that。
```
var point = { 
x : 0, 
y : 0, 
moveTo : function(x, y) { 
     var that = this; 
    // 内部函数
    var moveX = function(x) { 
    that.x = x; 
    }; 
    // 内部函数
    var moveY = function(y) { 
    that.y = y; 
    } 
    moveX(x); 
    moveY(y); 
    } 
}; 
point.moveTo(1, 1); 
point.x; //==>1 
point.y; //==>1
```

作为构造函数调用
===
JavaScript 支持面向对象式编程，与主流的面向对象式编程语言不同，JavaScript 并没有类（class）的概念，而是使用基于原型（prototype）的继承方式。相应的，JavaScript 中的构造函数也很特殊，如果不使用 new 调用，则和普通函数一样。作为又一项约定俗成的准则，构造函数以大写字母开头，提醒调用者使用正确的方式调用。如果调用正确，this 绑定到新创建的对象上。
```
function Point(x, y){ 
   this.x = x; 
   this.y = y; 
}
```

使用 apply 或 call 调用
===
让我们再一次重申，在 JavaScript 中函数也是对象，对象则有方法，apply 和 call 就是函数对象的方法。这两个方法异常强大，他们允许切换函数执行的上下文环境（context），即 this 绑定的对象。很多 JavaScript 中的技巧以及类库都用到了该方法。让我们看一个具体的例子：
```
function Point(x, y){ 
   this.x = x; 
   this.y = y; 
   this.moveTo = function(x, y){ 
       this.x = x; 
       this.y = y; 
   } 
} 
 
var p1 = new Point(0, 0); 
var p2 = {x: 0, y: 0}; 
p1.moveTo(1, 1); 
p1.moveTo.apply(p2, [10, 10]);
```
在上面的例子中，我们使用构造函数生成了一个对象 p1，该对象同时具有 moveTo 方法；使用对象字面量创建了另一个对象 p2，我们看到使用 apply 可以将 p1 的方法应用到 p2 上，这时候 this 也被绑定到对象 p2 上。另一个方法 call 也具备同样功能，不同的是最后的参数不是作为一个数组统一传入，而是分开传入的。

箭头函数中的this
===
在箭头函数中，this与封闭词法环境的this保持一致。在全局代码中，它将被设置为全局对象：
```
var globalObject = this;
var foo = (() => this);
console.log(foo() === globalObject); // true

// 作为对象的一个方法调用
var obj = {foo: foo};
console.log(obj.foo() === globalObject); // true

// 尝试使用call来设定this,但是没有效果，也就是说箭头函数是不接受绑定的
console.log(foo.call(obj) === globalObject); // true

// 尝试使用bind来设定this
foo = foo.bind(obj);
console.log(foo() === globalObject); // true
```
无论如何，foo 的 this 被设置为他被创建时的环境（在上面的例子中，就是全局对象）。这同样适用于在其他函数内创建的箭头函数：这些箭头函数的this被设置为封闭的词法环境的。
```
// 创建一个含有bar方法的obj对象，
// bar返回一个函数，
// 这个函数返回this，
// 这个返回的函数是以箭头函数创建的，
// 所以它的this被永久绑定到了它外层函数的this。
// bar的值可以在调用中设置，这反过来又设置了返回函数的值。
var obj = {
  bar: function() {
    var x = (() => this);
    return x;
  }
};

// 作为obj对象的一个方法来调用bar，把它的this绑定到obj。
// 将返回的函数的引用赋值给fn。
var fn = obj.bar();

// 直接调用fn而不设置this，
// 通常(即不使用箭头函数的情况)默认为全局对象
// 若在严格模式则为undefined
console.log(fn() === obj); // true

// 但是注意，如果你只是引用obj的方法，
// 而没有调用它
var fn2 = obj.bar;
// 那么调用箭头函数后，this指向window，因为它从 bar 继承了this。
console.log(fn2()() == window); // true
```
在上面的例子中，一个赋值给了 obj.bar的函数（称为匿名函数 A），返回了另一个箭头函数（称为匿名函数 B）。因此，在 A 调用时，函数B的this被永久设置为obj.bar（函数A）的this。当返回的函数（函数B）被调用时，它this始终是最初设置的。在上面的代码示例中，函数B的this被设置为函数A的this，即obj，所以即使被调用的方式通常将其设置为 undefined 或全局对象（或者如前面示例中的其他全局执行环境中的方法），它的 this 也仍然是 obj 。


作为一个DOM元素来处理函数
===
当函数被用作事件处理函数时，它的this指向触发事件的元素（一些浏览器在使用非addEventListener的函数动态添加监听函数时不遵守这个约定）。
```
// 被调用时，将关联的元素变成蓝色
function bluify(e){
  console.log(this === e.currentTarget); // 总是 true

  // 当 currentTarget 和 target 是同一个对象时为 true
  console.log(this === e.target);        
  this.style.backgroundColor = '#A5D9F3';
}

// 获取文档中的所有元素的列表
var elements = document.getElementsByTagName('*');

// 将bluify作为元素的点击监听函数，当元素被点击时，就会变成蓝色
for(var i=0 ; i<elements.length ; i++){
  elements[i].addEventListener('click', bluify, false);
}
```

作为一个内联事件处理函数
===
当代码被内联on-event 处理函数调用时，它的this指向监听器所在的DOM元素：
```
<button onclick="alert(this.tagName.toLowerCase());">
  Show this
</button>
```
上面的 alert 会显示button。注意只有外层代码中的this是这样设置的：
```
<button onclick="alert((function(){return this})());">
  Show inner this
</button>
```

getter 与 setter 中的 this
===
再次，相同的概念也适用于当函数在一个 getter 或者 setter 中被调用。用作 getter 或 setter 的函数都会把 this 绑定到设置或获取属性的对象。
```
function sum() {
  return this.a + this.b + this.c;
}

var o = {
  a: 1,
  b: 2,
  c: 3,
  get average() {
    return (this.a + this.b + this.c) / 3;
  }
};

Object.defineProperty(o, 'sum', {
    get: sum, enumerable: true, configurable: true});

console.log(o.average, o.sum); // logs 2, 6
```
换个角度理解
===
如果像作者一样，大家也觉得上述四种方式不方便记忆，过一段时间后，又搞不明白 this 究竟指什么。那么我向大家推荐 Yehuda Katz 的这篇文章：Understanding JavaScript Function Invocation and “this”。在这篇文章里，Yehuda Katz 将 apply 或 call 方式作为函数调用的基本方式，其他几种方式都是在这一基础上的演变，或称之为语法糖。Yehuda Katz 强调了函数调用时 this 绑定的过程，不管函数以何种方式调用，均需完成这一绑定过程，不同的是，作为函数调用时，this 绑定到全局对象；作为方法调用时，this 绑定到该方法所属的对象。

函数的执行环境
===
JavaScript 中的函数既可以被当作普通函数执行，也可以作为对象的方法执行，这是导致 this 含义如此丰富的主要原因。  
1.一个函数被执行时，会创建一个执行环境（ExecutionContext），函数的所有的行为均发生在此执行环境中，构建该执行环境时，JavaScript 首先会创建 arguments变量，其中包含调用函数时传入的参数。  
2.接下来创建作用域链。然后初始化变量，首先初始化函数的形参表，值为 arguments变量中对应的值，如果 arguments变量中没有对应值，则该形参初始化为 undefined。  
3.如果该函数中含有内部函数，则初始化这些内部函数。如果没有，继续初始化该函数内定义的局部变量，需要注意的是此时这些变量初始化为 undefined，其赋值操作在执行环境（ExecutionContext）创建成功后，函数执行时才会执行，这点对于我们理JavaScript 中的变量作用域非常重要，鉴于篇幅，我们先不在这里讨论这个话题。  
4.最后为 this变量赋值，如前所述，会根据函数调用方式的不同，赋给 this全局对象，当前对象等。至此函数的执行环境（ExecutionContext）创建成功，函数开始逐行执行，所需变量均从之前构建好的执行环境（ExecutionContext）中读取
