## 3. jQuery对象的属性和方法

为`jQuery`的原型添加方法和属性,这些方法和属性可以被`jQuery`的实例对象所使用.原型对象的属性会被实例对象继承,当然如果实例对象已经具备相应的属性,则会把原型对象的同名属性覆盖掉.

``` javascript
jQuery.fn = jQuery.prototype = {
    jquery:      版本号,
    constructor: 修正指向问题,
    init():      初始化和参数管理(构造函数),
    selector:    实例化对象时的初始化选择器,
    length:      默认的Jquery对象的长度是0,
    toArray():   转数组(也可以是对外的实例方法),
    get():       转原生集合,其实也是转成数组形式(对外方法),
    pushStack(): jQuery对象的一个入栈处理(外部用的不多,内部用的对),
    each():      遍历集合,
    ready():     DOM加载的接口,
    slice():     集合的截取,
    eq():        集合的第一项,
    last():      集合的最后一项,
    eq():        集合的指定项,
    map():       返回新集合,
    end():       栈回溯,可以看做popStack(),
    push():      (内部使用),
    sort():      (内部使用),
    splice():    (内部使用)
}
```

### 3.1 $().jquery
- 版本号
``` javascript
console.log($().jquery);    //2.0.3
```

### 3.2 $().constructor

默认的构造函数的原型的`constructor`属性指向该构造函数,但是`constructor`属性很容易被修改.所以可以在原型对象的`constructor`属性中进行修正.

>源码

```javascript
//[100]
constructor: jQuery,
```



>内容解析

``` javascript
function Obj() {}
alert(Obj.prototype.constructor);	//function Obj() {}

Obj.prototype.init = function(){
};
Obj.prototype.css = function(){
};

//使用对象字面量的方法,利用新的对象将Obj.prototype进行了覆盖
Obj.prototype = {
    init: function(){
	},
	css: function(){
	}
};
alert(Obj.prototype.constructor);	//function Object{[native code]}
```

### 3.3 $().init() (jQuery构造函数方法)

对外提供的实例对象的接口是`$()`或者`jQuery()`,当调用`$()`的时候其实是调用了`init()或者说jQuery()`,然后返回的是`jQuery的实例对象`,这样就可以使用`jQuery对象`的`prototype`的方法和属性(因为继承关系),`init()`方法的功能是`初始化jQuery的实例对象`.


>源码

``` javascript
//[101]

/**
- selector: 选择器
- context: 包含选择器的元素,选择器的上下文,详细说明:如果没有指定上下文context, 则默认情况将从根元素document对象开始查找,查找范围是整个文档,如果传入上下文,则可以限定查找范围,这样明显可以提高效率.
- rootjQuery
*/

init: function( selector, context, rootjQuery ) {
		var match, elem;

		// HANDLE: $(""), $(null), $(undefined), $(false)
		if ( !selector ) {
			return this;  //详见(一)、(二)
		}

		// Handle HTML strings
		if ( typeof selector === "string" ) {

			//判断是否为$('<li>'), $('<li>1</li><li>2</li>')类型,当然也可能是$('<li><p>')这种非合法类型
	        //即判断是否为复杂的HTML标签(注意也包括单个吧)
			if ( selector.charAt(0) === "<" && selector.charAt( selector.length - 1 ) === ">" && selector.length >= 3 ) {
				// Assume that strings that start and end with <> are HTML and skip the regex check
				match = [ null, selector, null ];

			} else {
			   //判断是否为单个标签或者id
		       // 比如'#div','<li>hello','<>','     <div>'
		       //注意'<div>',<li>1</li><li>2</li>,也能被匹配,但是第一个if已经匹配走了
		       //macth值
		       //如果是html标签,则 match[2] = undefined
		       //如果是#id,则match[1] = undefined
				match = rquickExpr.exec( selector );
			}

			//匹配html标签(注意html标签还是可以有context的)或者或者没有context的#id
			//类似于if( match && (match[1] || (macth[2] && !context) )
			//原因是match[1]如果是undefined那么macth[2]必定不是undefined
			//此时match[2]当然可以省略啦
			//还需要注意的是这里处理了包括第一部分的if
			//尽管第一部分的if里match[0] = null
			// Match html or make sure no context is specified for #id
			if ( match && (match[1] || !context) ) {
				//第一种情况匹配html标签
			    //包括第一部分的if情况
			    //需要注意的还有匹配html标签还是可以有context的
			    //HANDLE: $(html) -> $(array)
			    //这里当然是要把html转换成dom数组的形式
			    //例如$('<li>1</li><li>2</li>')
			    //转换成
			    //{
			    //  0: li,
			    //  1: li,
			    //  length:2
			    //}
				// HANDLE: $(html) -> $(array)
				if ( match[1] ) {
					//$('<li>',document)
			        //$('<li>',contentWindow.document) iframe的document
			        //当然如果context为undefined的情况下仍然是undefined
			        //$('<li>',$(document)) 此时 context instanceof jQuery = true
			        //var context = $(document);
			        //console.log(context instanceof jQuery);   //true
			        //之后可能会这么用
			        //$('<li>',$(document));
			        //console.log(context[0]);                  //document
					//所以如果context = $(document) jQuery对象
			        //那么必须使context = $(document)[0] DOM对象
			        //context详见(三)
					context = context instanceof jQuery ? context[0] : context;

					//合并DOM数组到this对象(this是json格式的类数组对象)
					//这样之后才可以进行css(),appendTo()等操作(操作this对象)
					// scripts is true for back-compat
					//详见(四）、(五)
					jQuery.merge( this, jQuery.parseHTML(
						match[1],
						context && context.nodeType ? context.ownerDocument || context : document,
						true
					) );


					//例如 $('<div>',{title: 'div', html: 'adcd', css: {background:'red'}})
					//创建标签的同时带属性,平时用的很少
					//rsingleTag匹配单标签 <li> 或 <li></li> 也就是selector第一个参数必须是单标签
					//第二个参数也就是context必须是对象字面量
					// HANDLE: $(html, props)
					if ( rsingleTag.test( match[1] ) && jQuery.isPlainObject( context ) ) {
						for ( match in context ) {
							// Properties of context are called as methods if possible
							//这里把match冲掉了,不用开辟新的变量
					        //同时我们知道$().html()方法是存在的
					        //所以遍历context时候,如果有html属性
					        //这个if就会满足
					        //并调用this.html('abcd')
							if ( jQuery.isFunction( this[ match ] ) ) {
								this[ match ]( context[ match ] );
							//否则例如像titile的属性,没有$().title()方法
					        //则调用this.attr(title,'div');
					        //当然也可以使用jQuery.attr()
							// ...and otherwise set as attributes
							} else {
								this.attr( match, context[ match ] );
							}
						}
					}

					//到这里就返回this对象了,我们当然始终要记得这个this对象是在什么时候使用
			        //$('div')返回带DOM节点的this对象,是一个类数组对象
			        //然后d调用css(), appendTo()等方法使用喽,在这些方法里操作this
			        //this始终是构造函数的上下文环境喽
			        //那么一旦创建了实例,this当然也是实例对象喽
			        //可以链式调用
					return this;

				// HANDLE: $(#id)
				//如果macth[1]是undefined
				//那么match[2]肯定不是undefined
				//match[2]当然是用来匹配#id的情况
				} else {
					//简单粗暴获取id为match[2]的节点试试
					elem = document.getElementById( match[2] );

					// Check parentNode to catch when Blackberry 4.6 returns
					// nodes that are no longer in the document #6963
					//如果节点存在
				    //当然如果黑莓4.6就算是节点不存在也会返回true
				    //所以此时需要多个判断条件,就是判断节点的父节点是否存在
				    //因为不存在的节点肯定是没有父节点的
					if ( elem && elem.parentNode ) {
						// Inject the element directly into the jQuery object
						//给构造函数jQuery添加length属性,因为是id嘛,所以当然只有一个DOM节点喽
						this.length = 1;
						//将DOM节点赋值给this类数组对象,方便日后操作
						this[0] = elem;
					}
					//没有context的#id上下文当然是document
					this.context = document;
					//选择器仍然是本身
					this.selector = selector;
					//返回this方便操作喽
					return this;
				}

			// HANDLE: $(expr, $(...))
			//context.jquery判断如果是jQuery对象
			} else if ( !context || context.jquery ) {
			    //例如$('ul',$(document)).find('li')
				//$(document).find()
				//调用了Sizzle选择器
				return ( context || rootjQuery ).find( selector );

			// HANDLE: $(expr, context)
			//例如$('ul',document).find('li')
			// (which is just equivalent to: $(context).find(expr)
			} else {
				return this.constructor( context ).find( selector );
			}
			//以上两种判断$(expr, $(...))、$(expr, context)最终变成$(document).find()调用Sizzle选择器

		// HANDLE: $(DOMElement)
		//如果不是字符串,是节点
		} else if ( selector.nodeType ) {
		    //这里仍然要将this转换成类数组对象
			//console.log($(document));
			/**
			 * {
			 *   0: document (节点对象)
			 *   context: document (仍然是节点对象,这个是传入的节点对象)
			 *   length: 1
			 * }
			 */
			this.context = this[0] = selector;
			this.length = 1;
			return this;

		// HANDLE: $(function)
		// Shortcut for document ready
		//例如$(function(){}),实际仍然调用,$(document).ready(function(){})
		} else if ( jQuery.isFunction( selector ) ) {
		    //$(document).ready(function(){})
			return rootjQuery.ready( selector );
		}

		//看传入的参数是不是jQuery对象 例如 $( $('div') )
		if ( selector.selector !== undefined ) {
			this.selector = selector.selector;
			this.context = selector.context;
		}
		//类似于jQuery.merge方法
		//写一个参数对外是用来转成数组
		//由于我们需要返回的是this这种类数组对象
		//所以写两个参数则是可以转成类数组对象this
		//详见(六)
		return jQuery.makeArray( selector, this );
	},
```


>内容解析

(一)、构造函数返回值问题

``` javascript
//构造函数一般不需要返回值
function Obj() {
	this.a = 1;
	this.b = 2;
}
var o = new Obj();
alert(o.a);

//返回this,实现链式调用
function Obj1() {
    this.a = 1;
    this.b = 2;
    return this;
}
var o1 = new Obj1();
alert(o1.a);	//1

//返回引用类型的值,则返回有效
function Obj2() {
    this.a = 1;
    this.b = 2;
    return {
        a: 3,
		b: 4
	};
}
var o2 = new Obj2();
alert(o2.a);	//3

//返回非引用类型,返回值无效
 function Obj3() {
	 this.a = 1;
     this.b = 2;
     return undefined;
 }
 var o3 = new Obj3();
 alert(o3.a);	//1
```

``` javascript
function a() {
    this.attr = 0;
    this.func = function() {
        alert("a-func");
    }
}
a.prototype = {
    func2:function() {
        alert("a-func2");
        return this;
    },
    func3:function() {
        alert('chain');
        return this;
    }
}
//链式调用
var aa = (new a()).func2().func3();
```

(二)、构造函数返回值`this`(可以理解为返回引用值类型有效么？)

`this`作为类数组对象可以进行`for`循环处理,需要注意`$()`是`jQuery`对象,而`$()[0]`是DOM元素。


| jQuery中构造函数的this属性(注意不是jQuery原型对象的属性) | 描述|
| :-------- | :--------|
| `this`中数字属性,例如0,1,2…,也可以说是类数组对象的数字属性(本来数组里的索引就是特殊的字符串属性嘛,例如数组a[0]其实就是a[‘0’]嘛)| 每一个属性代表一个被选中的DOM元素|
| `length`| 选取DOM元素数组的长度,也是`this`类数组的长度 |
| `context`| 选取的上下文环境,例如`document`或者`$(document)` |
| `selector`| 选择器 |


(三)、`context`具体使用案例（重点,加速查找DOM元素）

``` javascript
$('h1').click(function() {
    $('strong',this).css('color','blue');   //this指代h1 限定查找范围
});
//$('h1')因为没有指定上下文,所以调用浏览器原生方法document.getElementById方法查找属性为id的元素
//$('strong',this)因为指定了上下文,则通过jQuery的.find()方法查找,因此等价于$('h1').find('strong')
```

(四)、`$.parseHTML`、`$.merge`

```javascript
$('<li>1</li><li>2</li>').appendTo('ul'); //添加成功
//$('<li>1</li><li>2</li>')当然要变成如下格式,然后才能调用appendTo方法去处理
/**
 * this = {
 *  0:'li',
 *  1:'li',
 *  length: 2
 * }
 */
//jQuery.parseHTML 把字符串转成节点数组
//参数3个
//1.str字符串
//2.指定根节点
//3:true or false
var str = '<li>3</li><li>4</li><script>alert(4)<\/script>'; //字符串 </script>需要注意,需要转义
// var arr = jQuery.parseHTML(str,document);    //不会弹alert script标签不被添加
var arr = jQuery.parseHTML(str,document,true);  //弹alert script标签被添加
console.log(arr);
/**
 * [
 *      0: li,  //变成DOM中的li节点了
 *      1: li,
 *      2: script,
 *      length:3
 * ]
 *
 */
$.each(arr,function(i) {
    $('ul').append(arr[i]); //添加成功
});
//jQuery.parseHTML返回的是数组,但是最终在$('<li>1</li><li>2</li>')中我们发现需要的是转成json格式的this对象
//于是jQuery.merge起作用啦
var a = [1,2],
    b = [3,4];

console.log($.merge(a,b));      //[1,2,3,4] 对外是数组合并的功能
var json = {    //类数组对象
    0: 1,
    1: 2,
    length:2
};
var arr = [3,4];
console.log($.merge(json,arr)); //将json和数组合在了一起 源代码中的this就是json格式
/**
 *  {
    0: 1,
    1: 2,
    2: 3,
    3: 4
    length:4
};
 */
```


(五)、`this`在jQuery中的实际使用案例

``` javascript
//传统写法
var liArray = document.getElementsByTagName('li');
for(var index in liArray) {
    liArray[index].style.color = 'blue'; //liArray[index]是DOM元素对象
}

//jQuery使用案例
$('li').css('color','red'); 			//$('li')是jQuery对象


var J = $('li');    					//jQuery实例对象

J.css('color','red');       			//J并不是DOM元素对象
console.log(J);     					//是一个类数组对象
/*
{
    0:li,
    1:li,
    ...
    7:li,
    context:document,                   //上下文环境是一个根元素
    length:8,
    prevObject:init[1]
    selector:'li'
}
 */}
J[2].style.color = 'blue';             //那么明显J[2]是DOM元素对象操作


/*
然后可以想象在css()方法中遍历这个类数组去更改样式. 那么J和css()中怎么将这个类数组对象联系在一起,应为在两个函数中的变量都是局部变量哎,其实也很简单,因为两个方法都是实例对象的原型方法,那么在同一个实例对象上调用这两个方法的this是同一个啊,所以肯定是通过this对象啦,那么在css()方法中,会是这样处理

for(var i=0 len=this.length; i<len; i++) {
    this[i].style.color = 'red';  //类对象的数组元素是真正的DOM元素操作
}
*/
```


(六)、`$.makeArray`

- 一个参数可以提供给外部使用,转化为数组
- 两个参数提供给内部使用,转化为`this`需要的类数组对象

``` javascript
$('<div>', {html: 'this is a div', class:'div'}).appendTo('body');  //添加在body的末尾
$('<div>', {html: 'this is a div', class:'div'}).appendTo('body');  //添加在body的末尾
$('<div>', {html: 'this is a div', class:'div'}).appendTo('body');  //添加在body的末尾
var arrDiv = document.getElementsByTagName('div');
console.log(arrDiv);    //获取的是一个类数组对象
//arrDiv.push();    //arrDiv.push is not a function
$.makeArray(arrDiv).push(); //转化成数组就可以调用了
console.log($.makeArray(arrDiv));   //数组
//但是$()构造函数返回的是this类数组对象,而不是数组
console.log($.makeArray(arrDiv,{length:0}));    //类数组对象
```



### 3.4 $().selector
### 3.5 $().length
### 3.6 $().toArray()

- 类数组对象转数组

>源码
``` javascript
//[202]
toArray: function() {
	return core_slice.call( this ); //Array.prototype.slice.call(类数组对象)
}
```

>内容解析

`call /apply` 可以显示指定调用所需的`this`值,任何函数可以作为任何对象的方法来调用,哪怕这个函数不是那个对象的方法,第一个参数是要调用函数的母对象,即调用上下文,在函数体内通过`this`来获得对它的引用

``` javascript
function f() {
    alert(this.x);
}

f();				//undefined
f.call({x:100});	//100,函数中的this指代传入的对象,call可以使函数中的this值指向传入的第一个参数

//call和apply的区别
function f(x,y) {
    alert(this.x + x + y);
}

f.call({x:1},2,3);	  //6,传入一个个参数
f.apply({x:1},[3,4]); //8,传入一个数组
```


```javascript
var obj = {
    0:'apple',
	1:'huawei',
	length:2
};
console.log(obj);								//Object
console.log(Array.prototype.slice.call(obj));	//Array[2]

//Array.prototype.slice的可能源码
Array.prototype.slice = function(start,end){
    var result = new Array();
     start = start || 0;
     end = end || this.length; //this指向调用的对象,当用了call后,能够改变this的指向
     for(var i = start; i < end; i++){
          result.push(this[i]);
     }
     return result;
 }

//jQuery的用法
var $div = $('div');	//由于构造函数返回的是this, this是一个类数组对象

/*this可能的值
{
    0:div,
    1:div,
    ...
    7:div,
    context:document,                   //上下文环境是一个根元素
    length:8,
    selector:'li'
}
*/

$div.toArray();		//把$div的this对象传入了toArray的core_slice.call( this );后面懂了,就把this变成了数组
```

附上`[].slice`源码分析

``` javascript
//[].slice(start,end)
//返回一个新数组,该数组是原数组从tart到end(不包含该元素)的元素
//以下是slice的源码,我们可以发现它其实是可以将类数组对象进行转换为数组
//开头说明不光光是普通的数组,类数组对象,NamedNodeMap,NodeList,HTMLCollection,DOM objects 都是可以转化成数组的
// This will work for genuine arrays, array-like objects,
// NamedNodeMap (attributes, entities, notations),
// NodeList (e.g., getElementsByTagName), HTMLCollection (e.g., childNodes),
// and will not fail on other DOM objects (as do DOM elements in IE < 9)
Array.prototype.slice = function(begin, end) {
  // IE < 9 gets unhappy with an undefined end argument
  //如果没有传入end参数,则默认是this.length长度
		  end = (typeof end !== 'undefined') ? end : this.length; //这里终于知道为什么要用typeof a === 'undefined' 就是考虑到了IE的兼容性问题
  // 如果是原生数组对象,则调用原生数组对象的方法
  // For native Array objects, we use the native slice function
  // 以下方法普遍用来判断传入的参数是否是数组
  if (Object.prototype.toString.call(this) === '[object Array]'){
    return _slice.call(this, begin, end);
  }
  // For array like object we handle it ourselves.、
  //如果是类数组对象
  var i, cloned = [],
    size, len = this.length;

  // 如果start默认没有传入参数则是0
  // Handle negative value for "begin"
  var start = begin || 0;
  //如果start>=0则选择start,否则选择你懂得,防止负数的情况下少于数组的长度
  start = (start >= 0) ? start : Math.max(0, len + start);

  //如果传入的end是number,则比较end和len,这种情况防止传入end大于数组本身的长度
  //如果不是,则默认处理成数组的长度
  // Handle negative value for "end"
  var upTo = (typeof end == 'number') ? Math.min(end, len) : len;
  //当然end<0
  if (end < 0) {
    upTo = len + end;
  }
  // Actual expected size of the slice
  size = upTo - start;
  if (size > 0) {
    cloned = new Array(size);
    //字符串的情况
    if (this.charAt) {
      for (i = 0; i < size; i++) {
        cloned[i] = this.charAt(start + i);
      }
    } else {
       //类数组情况
      for (i = 0; i < size; i++) {
        cloned[i] = this[start + i];        //这里就是将类数组对象转化为数组
      }
    }
  }
  return cloned;
};
```


### 3.7 $().get()

- 不传参数功能是`$().toArray()`
- 传参数的功能是 `$()[num]`


>源码
``` javascript
//[206]
// Get the Nth element in the matched element set OR
// Get the whole matched element set as a clean array
get: function( num ) {
	return num == null ?

		// Return a 'clean' array
		this.toArray() :

		// Return just the object
		( num < 0 ? this[ this.length + num ] : this[ num ] );
},
```

>内容解析

``` javascript
<div>1</div>
<div>2</div>
<div>3</div>]

<script src='Jquery2.0.3.js'></script>

<script>
	console.log($('div').get());					//Array[3]
    console.log($('div').toArray());				//Array[3]
	console.log($('div')[0] == $('div').get(0));	//true
</script>
```

### 3.8 $().pushStack()

>源码

``` javascript
// Take an array of elements and push it onto the stack
// (returning the new matched element set)
// 使用元素数组并把当前选中的元素压入栈
// 返回的是新的被匹配的元素
pushStack: function( elems ) {

	// Build a new jQuery matched element set
	// this.constructor()是一个空的Jquery对象
    // 合并新的元素到新的this对象
	var ret = jQuery.merge( this.constructor(), elems );

	// Add the old object onto the stack (as a reference)
	// 老的对象被保留到一个prevObject属性
	ret.prevObject = this;
	ret.context = this.context;

	// Return the newly-formed element set
	// 返回新的元素的结果
	return ret;
},
```

> 内容解析


``` javascript
$('<div>', {html: 'this is a div', class:'div'}).appendTo('body');  //添加在body的末尾
$('<span>', {html: 'this is a span', class:'span'}).appendTo('body');   //添加在body的末尾
console.log($('div').pushStack( $('span') ));
$('div').pushStack( $('span') ).css('background','red');    //span变红 div没有
//因为在栈中span在div上面
$('div').pushStack( $('span') ).css('background','red').css('background','yellow'); //span变黄
//{
//  0: span,
//  length: 1,
//  context: document,
//  prevObject: {
//      0: div,
//      length: 1,
//      selector: "div",
//      prevObject: {
//          0: document,
//          context: document,
//          length: 1,
//      }
//  }
//}
console.log($('div').pushStack( $('span') ).css('background','red').prevObject);    //div
console.log($('div').pushStack( $('span') ).css('background','red').context);       //document
$('div').pushStack( $('span') ).css('background','red').prevObject.css('fontSize','100px'); //div的字体变了
//如果仍然想使用栈的下一层div(上一层是span),end方法回溯栈其实就利用了prevObject属性
$('div').pushStack( $('span') ).css('background','red').end().css('background','yellow');   //span红色,div黄色
```

### 3.9 $().end()

>源码

```javascript
// [271-273]
end: function() {
    //主要是对于pushStack的回溯,返回被push之前的jQuery实例对象
    return this.prevObject || this.constructor(null);
},
```

### 3.10 $().slice()

>源码
``` javascript
//[247-250]
slice: function() {
    return this.pushStack( core_slice.apply( this, arguments ) );
},
```

> 内容解析

```javascript
$('<div>', {html: 'this is a div', class:'div'}).appendTo('body');  //添加在body的末尾
$('<div>', {html: 'this is a div', class:'div'}).appendTo('body');  //添加在body的末尾
$('<div>', {html: 'this is a div', class:'div'}).appendTo('body');  //添加在body的末尾
$('<div>', {html: 'this is a div', class:'div'}).appendTo('body');  //添加在body的末尾
$('div').slice(1,3).css('background','red');    //中间两个div背景变红色,注意和数组的方法一样不包括第三个
//其实是在4个div的基础上又入栈了被选中的两个div
//如果利用end回溯
$('div').slice(1,3).css('background','red').end().css('background','red');  //4个div背景色都变成了红色
```

### 3.11 $().each()

>源码

``` javascript
// [233-238]
// 通过工具each方法,工具方法用于构建库的最底层,实例方法调用工具方法
// 实例方法可以看成更高级别的层次
each: function( callback, args ) {
	    return jQuery.each( this, callback, args ); //后续分析$.each()
},
```


### 3.12 $().ready()

>源码

```javascript
// [240-245]
ready: function( fn ) {
    // 使用Promise的形式等待回调
    jQuery.ready.promise().done( fn );
    return this;
},
```

### 3.13 $().first()/last() /eq()

>源码

``` javascript
first: function() {
    return this.eq( 0 );
},
last: function() {
    return this.eq( -1 );
},

//[259-263]
eq: function( i ) {
    var len = this.length,
        j = +i + ( i < 0 ? len : 0 );
    return this.pushStack( j >= 0 && j < len ? [ this[j] ] : [] );
},
```

>内容解析

``` javascript
$('<div>', {html: 'this is a div', class:'div'}).appendTo('body');  //添加在body的末尾
$('<div>', {html: 'this is a div', class:'div'}).appendTo('body');  //添加在body的末尾
$('<div>', {html: 'this is a div', class:'div'}).appendTo('body');  //添加在body的末尾
$('<div>', {html: 'this is a div', class:'div'}).appendTo('body');  //添加在body的末尾
$('div').first().css('background','red');   //第一个红
$('div').last().css('background','yellow'); //最后一个黄
$('div').eq(2).css('background','blue');    //第三个蓝
```

### 3.14 $().map()

>源码

```javascript
//[265-269]
map: function( callback ) {
    //入栈
    //最终调了底层的工具方法
    return this.pushStack( jQuery.map(this, function( elem, i ) {
        return callback.call( elem, i, elem );
    }));
},
```

>内容解析

```javascript
var arr = ['a','b','c'];
arr = $.map(arr,function(item,index) {
    return item + index;
});
console.log(arr);   //[a0,b1,c2]
```

### 3.15 $().push()/sort()/slice()

- 内部用,不建议在外面使用,内部使用增加性能

>源码

```javascript
// [275-279]
// 内部使用
// 将Array的方法挂载到了jQuery对象下面
push: core_push,
sort: [].sort,
splice: [].splice
```
