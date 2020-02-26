JavaScript中易错的知识点
====================

1. 箭头函数中的this:
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
 所以想要改变箭头函数中的this指向，只能改变定义它是的上下文中的this的指向。

