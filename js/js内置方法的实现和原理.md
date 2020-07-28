javascript内置方法的原理和实现
============================

## call方法

### 用法：
fn.call(context[, arg1[,arg2...]])

### 应用：
* 绑定函数执行上下文

### 模拟实现
* 1.可以接收多个参数，并且有原函数执行的返回值
* 2.传递的上下文为null,undefined时，绑定为window对象
* 3.传递的上下文为除了null,undefined的原始值时，绑定为使用new Object的对象

1. 使用es6实现

```
	function callPolyfill(context, ...args) {
		// 此处的this会为function
		const fn = this

		// 如果context传进的值为null,undefined,则绑定为window对象，如果是其他的原始值，则为它的包装对象(使用new Object转成对象)，
		// 否则为该传递的对象
		if (context === null || context === undefined) { // 不能使用!context，因为传递值为false时，应该是包装对象
			context = window
		} else if (typeof context !== 'object' || typeof context !== 'function') {
			// 其他的原始值类型: string number boolean symbol
			context = new Object(context)
		}

		const symbol = Symbol('key') // 为了不影响context对象中的同名属性，这里使用symbol来创建唯一的属性
		context[symbol] = fn
		const res = context[symbol](...args)
		delete context[symbol]

		// 返回fn执行的结果
		return res		
	}

	Function.prototype.callPolyfill = callPolyfill

```

2. 不使用es6的实现
	和使用es6的区别主要是接收的参数的处理和原函数的调用时，参数的传递形式的不同

```
	function callPolyfill2(context) {
		// 此处的this会为function
		var fn = this

		// 如果context传进的值为null,undefined,则绑定为window对象，如果是其他的原始值，则为它的包装对象，
		// 否则为该传递的对象
		if (context === null || context === undefined) { // 不能使用!context，因为传递值为false时，应该是包装对象
			context = window
		} else if (typeof context !== 'object' || typeof context !== 'function') {
			// 其他的原始值类型: string number boolean symbol
			context = new Object(context)
		}

		// 获取传递的参数
		var args = []
		for (var i = 1; i < arguments.length; i++) {
			args.push(arguments[i])
		}

		// 防止影响原来的方法
		var preMethod = context.fn
		context.fn = fn

		// 不能直接使用context.fn(args),这样的话，fn函数只能接收到一个参数
		// 要使用eval来传递参数到执行的函数, 因为参数是多个，不能直接调用函数，那样只能传递成一个参数.
		var res = eval('context.fn('+ args +')') // args会自动调用数组的toString转成字符串

		// 或者使用new Function来实现, 注意：使用new Function创建的方法，他的作用域为函数本地作用域和全局的作用域，和调用new Function的上下文无关
		// var fun = new Function('context', 'return context.fn('+ args +')')
		// var res = fun(context)

		context.fn = preMethod

		// 返回fn执行的结果
		return res
	}

	Function.prototype.callPolyfill2 = callPolyfill2

```


## apply方法

### 用法：
fn.apply(context[, args])

### 应用：
* 绑定函数执行上下文

### 模拟实现
* 1.可以接收一个数组参数，并且有原函数执行的返回值
* 2.传递的上下文为null,undefined时，绑定为window对象
* 3.传递的上下文为除了null,undefined的原始值时，绑定为使用new Object的对象

1. 使用es6实现

```
	function applyPolyfill(context, args) {
		// 此处的this会为function
		const fn = this

		// 如果context传进的值为null,undefined,则绑定为window对象，如果是其他的原始值，则为它的包装对象(使用new Object转成对象)，
		// 否则为该传递的对象
		if (context === null || context === undefined) { // 不能使用!context，因为传递值为false时，应该是包装对象
			context = window
		} else if (typeof context !== 'object' || typeof context !== 'function') {
			// 其他的原始值类型: string number boolean symbol
			context = new Object(context)
		}

		const symbol = Symbol('key') // 为了不影响context对象中的同名属性，这里使用symbol来创建唯一的属性
		context[symbol] = fn
		const res = context[symbol](...args)
		delete context[symbol]

		// 返回fn执行的结果
		return res		
	}

	Function.prototype.applyPolyfill = applyPolyfill

```

2. 不使用es6的实现
	和使用es6的区别主要是接收的参数的处理和原函数的调用时，参数的传递形式的不同

```
	function applyPolyfill2(context, args) {
		// 此处的this会为function
		var fn = this

		// 如果context传进的值为null,undefined,则绑定为window对象，如果是其他的原始值，则为它的包装对象，
		// 否则为该传递的对象
		if (context === null || context === undefined) { // 不能使用!context，因为传递值为false时，应该是包装对象
			context = window
		} else if (typeof context !== 'object' || typeof context !== 'function') {
			// 其他的原始值类型: string number boolean symbol
			context = new Object(context)
		}

		// 防止影响原来的方法
		var preMethod = context.fn
		context.fn = fn

		// 不能直接使用context.fn(args),这样的话，fn函数只能接收到一个参数
		// 要使用eval来传递参数到执行的函数, 因为参数是多个，不能直接调用函数，那样只能传递成一个参数.
		var res = eval('context.fn('+ args +')') // args会自动调用数组的toString转成字符串

		// 或者使用new Function来实现, 注意：使用new Function创建的方法，他的作用域为函数本地作用域和全局的作用域，和调用new Function的上下文无关
		// var fun = new Function('context', 'return context.fn('+ args +')')
		// var res = fun(context)

		context.fn = preMethod

		// 返回fn执行的结果
		return res
	}

	Function.prototype.callPolyfill2 = callPolyfill2

```

## bind方法

### 用法：
var f = fn.bind(context[,arg1...])
f(arg2...)

### 应用：
* 参数柯里化
* 绑定函数执行上下文

### 模拟实现：
* 1.bind方法返回一个新的函数, 内部是借助apply或者call来绑定执行上下文
* 2.传递的上下文为null,undefined时，绑定为window对象
* 3.传递的上下文为除了null,undefined的原始值时，绑定为使用new Object的对象
* 4.当使用new操作符来使用新返回的函数时，行为要和使用new使用原函数一致

1. 使用es6实现

```
	function bindPloyfill (context, ...args1) {
		const fn = this

		const newFn = function (...args2) {
			const args = [...args1, ...args2]
			if (this instanceof newFn) {
				return new fn(...args)	
			} else {
				return fn.apply(context, args)
			}
			// return fn.apply(
			// 	this instanceof newFn ? this : context,
			// 	args
			// )
		}

		// 让使用new来使用绑定函数时，创建的实例对象也能继承原型上的属性和方法
		// newFn.prototype = fn.prototype

		return newFn
	}

	Function.prototype.bindPloyfill = bindPloyfill

```

2.不使用es6实现

```
	function bindPloyfill2 (context) {
		var fn = this

		// 先获取传递的一部分参数
		var args1 = []
		for (var i = 1; i < arguments.length; i++) {
			args1.push(arguments[i])
		}

		var newFn = function () {
			// 获取剩余的参数
			var args2 = []
			for (var i = 1; i < arguments.length; i++) {
				args2.push(arguments[i])
			}

			var args = args1.concat(args2)

			return fn.apply(
			 	this instanceof newFn ? this : context,
			 	args
			)
		}

		// 让使用new来使用绑定函数时，创建的实例对象也能继承原型上的属性和方法
		newFn.prototype = fn.prototype

		return newFn
	}

	Function.prototype.bindPloyfill2 = bindPloyfill2

```

## new操作符

### 用法
var obj = new Fn(args)

### 使用new操作符时的内部执行流程
- 创建一个继承构造函数原型对象prototype的新对象
- 使用指定的参数调用构造函数，并且绑定的执行上下文为上面新创建的对象
- 由构造函数返回的对象即为new操作符的运算结果，如果构造函数返回的不是一个对象，即返回上面新创建的对象作为运算的结果。

### 模拟实现

```
function Fun (name) {
    this.name = name
    return null
}

Fun.prototype.say = function() {
	console.log(this.name)
}

var obj = new Fun('hxc')
console.log(obj)

/*
@params fn 作为构造函数使用
@params args 作为构造函数的参数
*/
function create (fn, ...args) {
    // 创建一个继承自构造函数原型的新对象
    var obj = Object.create(fn.prototype)
    // var obj = {}
    // obj.__proto__ = fn.prototype
    
    // 新创建的对象作为执行上下文执行构造函数
    var res = fn.apply(obj, args)
    
    // 如果构造函数返回的为一个对象(纯对象，数组，函数等非基本数据类型的值)，则作为new的操作结果，否则使用上面新创建对象作为运算结果
    if (res !== null && typeof res === 'object' || typeof res === 'function') {
    	return res
    } else {
    	return obj
    }
}
var obj1 = create(Fun, 'hxc')
console.log(obj1)
```
# instanceof
用法： a instanceof b
其中b要为构造函数，a一般为实例对象，为基本数据类型时，返回false。
instanceof 的底层原理为判断b.prototype是否在a的原型链上

模拟实现：
```
// instanceof的底层原理为判断right_hand.prototype是否在left_hand的原型链上
function instanceOf (left_hand, right_hand) {
	if (typeof right_hand !== 'function') {
		throw new TypeError("Right-hand side of 'instanceof' is not callable");
	}
	// 如果左边的操作数为基本数据类型时，直接返回false
	if (left_hand === null || (typeof left_hand !== 'object' && typeof left_hand !== 'function')) {
		return false;
	}
	leftValue = left_hand.__proto__;
	rightValue = right_hand.prototype;
	while (leftValue) {
		if (leftValue === rightValue) {
			return true;
		}
		leftValue = leftValue.__proto__;
	}
	return false;
}
```

测试用例：
```
function fun () {

}

fn.prototype = new fun();

function fn () {

}


var f = new fn();

console.log(1 instanceof Number)
console.log("1" instanceof String)
console.log(true instanceof Boolean)
console.log(Symbol(1) instanceof Object)
console.log(null instanceof Object)
console.log(1 instanceof Object)
console.log("1" instanceof Object)

console.log(new Number(1) instanceof Number)
console.log(new Number(1) instanceof Object)
console.log({} instanceof Object)
console.log(f instanceof fn)
console.log(f instanceof fun)
console.log(Object instanceof Object)
console.log(Function instanceof Function)
console.log(Function instanceof Object)
console.log(Object instanceof Function)

console.log('-----------')

console.log(instanceOf(1, Number))
console.log(instanceOf("1", String))
console.log(instanceOf(true, Boolean))
console.log(instanceOf(Symbol(1), Object))
console.log(instanceOf(null, Object))
console.log(instanceOf("1", Object))
console.log(instanceOf(1, Object))

console.log(instanceOf(new Number(1), Number))
console.log(instanceOf(new Number(1), Object))
console.log(instanceOf({}, Object))
console.log(instanceOf(f, fn))
console.log(instanceOf(f, fun))
console.log(instanceOf(Object, Object))
console.log(instanceOf(Function, Function))
console.log(instanceOf(Function, Object))
console.log(instanceOf(Object, Function))
```

# Array.filter
为数组中的每个元素调用一次 callback 函数，并利用所有使得 callback 返回 true 或等价于 true 的值的元素创建一个新数组。
**callback 只会在已经赋值的索引上被调用，对于那些已经被删除或者从未被赋值的索引不会被调用。那些没有通过 callback 测试的元素会被跳过，不会被包含在新数组中**
如：
```
var arr = [0,1];
arr[5] = null;
newArr = arr.filter(function(x) { return x === undefined;});

console.log(newArr.length, newArr);// 0, []
```
模拟实现：
```
// 数组的有效下标
function isPosInteger32 (number) {
	return number >= 0 && (number | 0) === number;
}

function filter_polyfill(cb) {
	const arr = this;
	const ret= [];
	const keys = Object.keys(arr);
	for (let k of keys) {
	// 这里要过滤掉没有初始化的数组项，和无效的数组下标的key
		if (isPosInteger32(k * 1) && cb(arr[k])) {
			ret.push(arr[k]);
		}
	}
	return ret;
}

Array.prototype.filter_polyfill = filter_polyfill;
```