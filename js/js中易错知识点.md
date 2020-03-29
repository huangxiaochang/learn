JavaScript中易错的知识点
====================

## 1. 箭头函数中的this:
	箭头函数中的this是在箭头函数**真正创建时确定的**，而不是调用时确定，也不能通过call, apply, bind来改变箭头函数的this执行。并且一经定义，是不能更改其中的this指向的。如：
 ```
let id = 'global'

function fun (argument) {
    return () => { // arrow fn1 
        return () => { // arrow fn2 
            return () => { // arrow fn3 
            	console.log(this.id)
            }
        }
    }
}
// 在没有执行fun函数时，arrow fn1都是还没有定义的，只有执行了fun函数时，才会进行定义

// 这里执行了fun函数，并且绑定其this为{id: 1},在执行fun时创建arrow fn1,所以arrow fn1中的this为{id: 1}
let f = fun.call({id: 1})

// 就算这了在执行f的时候，使用call进行绑定this,但是由于f是箭头函数，所以它this不能改变，并且只能指向创建它的时候的this.即{id: 1}
let t1 = f.call({id: 2})()() // 1

// f()执行时，创建arrow fn2,由于arrow fn1的this为{id: 1},所以arrow fn2的this也是{id: 1}, arrow fn3同理
let t2 = f().call({id: 3})() // 1

// 同理	
let t3 = f()().call({id: 4}) // 1

 ```
 所以想要改变箭头函数中的this指向，只能改变**真正定义它时**的上下文中的this的指向。

## 连等赋值
```
var a = {n: 1}
var b = a
a.x = a = {n: 2}
console.log(a.x) // undefined
console.log(b.x) // {n: 2}
```
连等赋值语句的执行：
1.先进行所有赋值等号左边的变量的LHS查询，获取到指向的地址引用。如果查找的时候，不存在，会先创建对应的变量，然后返回该变量的地址引用。
2.从右到左进行赋值
3.每个=的赋值时，只会把该=号的右边的值(而不是最右边的值)赋给左边的地址引用。

a.x = a = {n: 2}的赋值过程
1.进行所有赋值等号左边的变量的LHS查询。a.x不存在，所以会在a的对象中创建一个x属性，并返回它的引用。查询a变量，返回a的引用。
2.进行赋值：(1)把{n: 2}赋值给a的引用.(2)把a的值赋值给a.x的引用。（注意，这里的a.x是一个地址引用，是在第一步就已经确定了，它为{n: 1}(b)的x属性的引用，所以a.x = a是把a({n: 2})的值赋给b.x）.

