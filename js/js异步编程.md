JavaScript中的异步编程
==========

## 1. async/await.
1.本质：
async函数本质上是Generator函数自执行的语法糖。 await语句只能在async函数内部使用。执行async函数时，会立即返回一个promise对象。async函数内部的返回值，会作为async函数返回的promise的then回调onFulled的参数。所以async函数相当于：
```
async fun () {
	return 11;
}

// 相当于
function fun () {
	// 立即返回一个promise对象
	return new Promise(function (resolve, reject) {
	// async函数内部的返回值，作为async函数返回的promise对象的then回调的参数
		resolve(11);
	})
}

```
2. async函数抛出错误时(内部未被处理的错误，包括await语句后面的promise对象未处理的reject)，async函数返回的promise对象会立即reject。该错误会被async返回的promise对象的catch语句或者then的onRejected回调捕获。
```
async fun () {
	await new Promise(function (resolve, reject) {
		reject(1111);
	})
	// 未被处理的错误抛出后，async函数终止执行，所以下面的代码是不会得到执行的
	console.log(124) // 不会得到执行
}

async fun () {
	throw(new Error('rwee'))
	// 下面的代码不会得到执行
	await 11
	console.log(333)
}
// 所以为了防止await后面的表达式发生错误，导致下面的代码得不到执行，可以使用try/catch来处理

async fun () {
	try {
		await new Promise(function (resolve, reject) {
            reject(1111);
        })
	} catch (e) {};
	// 这里的代码会被执行
	console.log(4556)
}

```
3. await语句：
   1.在执行async函数遇到await语句时，会先执行完await语句右边的表达式，得到该表达式的结果之后，然后暂停async函数的执行，去执行async函数外面的同步代码或者任务循环队列的执行。
   2.执行完同步代码或者一轮任务队列循环之后，回到上次暂停的await语句。
   3.如果await 右边表达式得到的结果不是一个promise对象或者是一个已经resolved的promise对象。那么整个await语句得到执行的值，然后执行await语句下面的代码。
```
   async function async1() {
  		console.log('async1 start');
  		// async2 返回了一个已经resolved的promise对象，当再次回到await语句时，继续执行await下面的代码
  		await async2();
  		console.log('async1 end');
   }
   async function async2() {
  		console.log('async2');
  		return 2222;
   }
   console.log('script start');

    setTimeout(function () {
      console.log('setTimeout');
    }, 0);

    async1();

    new Promise(function (resolve) {
      console.log('promise1');
      resolve();
    }).then(function () {
      console.log('promise2');
    });
    console.log('script end');
    /*
    	结果为：
    	script start
    	async1 start
    	async2
    	promise1
    	script end
    	async1 end
    	promise2
    	setTimeout
    */
```
   4.如果await右边表达式得到的结果是一个promise对象：

      2. 如果该promise对象还没有resolved，那么会先进行任务队列的循环，直到等待该promise resolved(如果resolve又是一个promise，那么会等待该promise状态resolved后才会执行await后面的代码。**反正整个await语句最后要得到一个不是promise的值或者是一个已经resolved的promise以后，才能继续执行await语句下面的代码**)，然后把它的加入微观队列，然后进行任务队列的循环。知道await语句得到的值不是promise时，才会继续执行await下面的代码。

