webpack的模块实现原理
====================
### webpack的打包流程：(省略了webpack配置参数获取与合并，webpack的compiler初始化等过程，这里只关注模块处理部分)。
1.从入口文件出发。对文件进行加载解析成ast树，在这个过程中，可能会用到loader来翻译文件模块成js模块。
2.找出入口文件依赖的模块，加载并解析。在这个过程中，会维护模块之间的依赖关系。
3.在模块输出阶段，会根据模块之间的依赖关系，确定输出的chunk，一个chunk可能包含多个模块。其中入口文件属于一个chunk，动态加载的模块，也会组成一个chunk。
4.模块的处理：会把每个模块的代码生成一个函数，函数传入参数为module, __webpack_exports__, __webpack_require__，其中__webpack_exports__为module.exports,
__webpack_require__为模块导入函数，是import/require的转换。
如：
```
function(module, __webpack_exports__, __webpack_require__) {
	// 这里是模块中的代码，对于模块文件源码中的import/require会换成__webpack_require__
	
}
```
模块代码生成阶段（由ast树生成chunk代码），会根据es模块之间导入导出关系，会标记模块中哪些导出或者导入是没有意义的（即没有被使用到的）。标记的意义是可以为其他插件（如ugifyjs压缩插件进行tree-shaking时用到）。之所以webpack能够标记出哪些导出没有被用到，依赖的是es模块中的import/export可以静态分析，是在构建ast树时会增加一些标志。具体的标志方法是挺复杂的。
5.chunk的生成
每个chunk会生成一个独立的js文件。文件中的内容如下：
```
(window["webpackJsonp"] = window["webpackJsonp"] || []).push([
	[chunkids], // chunkid
	[
	//模块,因为模块是并列的函数，所以访问不到其他模块中的变量，这样变实现了模块作用域
		function(module, __webpack_exports__, __webpack_require__) {},
		...
	]
])
```


## webpack运行时代码
webpack会把runtime代码打包成一个立即执行的代码(会在脚本下载完成后，立即执行以下的代码)：
```
(function(modules) {
	// 这里是webpack运行时代码，包含了一些方法和属性的定义，如
	// install a JSONP callback for chunk loading
	// 使用jsonp下载异步脚本(chunk)后，会执行该方法
	function webpackJsonpCallback(data) {}
	
	// The module cache
	var installedModules = {};
	
    // object to store loaded and loading chunks
    // undefined = chunk not loaded, null = chunk preloaded/prefetched
    // Promise = chunk loading, 0 = chunk loaded
    var installedChunks = {
        0: 0
    };
	
	// script path function 通过chunkid来获取对应的chunk的路径
	function jsonpScriptSrc(chunkId) {}
	
	// The require function 
	// webpack中加载模块的函数，即模块源码的import、require语句都会替换成该函数，
	// 该函数的作用是，通过模块的id，去加载模块
	function __webpack_require__(moduleId) {}
	
	// The chunk loading function for additional chunks
	// 加载异步模块的函数，即源码中的import()异步加载语句会替换成该方法
	__webpack_require__.e = function requireEnsure(chunkId) {}
	
	// 保存着所有依赖的模块，异步模块加载之后，也会添加到这里
	__webpack_require__.m = modules;
	
	// 模块缓存相关，即保存着所有已经加载缓存过的模块
	__webpack_require__.c = installedModules;
	
	// define getter function for harmony exports
	// 在模块输出对象exports定义属性的属性的getter，因为没有定义setter，所以在导入的地方尝试修改导入的变量时，会报错提示。对于es模块的导出，因为该属性是定义生寄存器属性，所以是值的引用。
	__webpack_require__.d = function(exports, name, getter) {}
	
	// define __esModule on exports
	// 如果是es模块，会在exports对象上定一个__esModule属性，标志它是es模块
	__webpack_require__.r = function(exports) {}
	
	// getDefaultExport function for compatibility with non-harmony modules
	// 这个函数的作用是获取默认的导出，是为了兼容非es模块
	__webpack_require__.n = function(module) {}
	
	// Object.prototype.hasOwnProperty.call
	__webpack_require__.o = function(object, property) {}
	
	// __webpack_public_path__
	__webpack_require__.p = "";
	
	// on error function for async loading 异步加载发送错误时的提示函数
	__webpack_require__.oe = function(err) {}
	
	// 定义window["webpackJsonp"]来存储所有已经下载的异步chunk
    var jsonpArray = window["webpackJsonp"] = 			      window["webpackJsonp"] || [];
    // 数组的原生push方法
    var oldJsonpFunction = jsonpArray.push.bind(jsonpArray);
    // 重新定义window["webpackJsonp"]数组的push方法，把它定义成webpackJsonpCallback。这样做的原因是，如果异步webpack runtime代码先于异步chunk加载完成，那么当异步chunk下载完成后，即可立即执行webpackJsonpCallback，因为异步chunk中代码为(window["webpackJsonp"] = window["webpackJsonp"] || []).push())
    jsonpArray.push = webpackJsonpCallback;
    jsonpArray = jsonpArray.slice();
    // 如果异步chunk先于webpack运行时代码加载完成，那么执行(window["webpackJsonp"] = window["webpackJsonp"] || []).push())语句中的push是原生的push方法，会添加到window["webpackJsonp"]中，所以当runtime代码下载完成执行后，要加载已经下载好的异步chunk，对于异步chunk的逻辑处理，见webpackJsonpCallback函数
    for(var i = 0; i < jsonpArray.length; i++) webpackJsonpCallback(jsonpArray[i]);
    var parentJsonpFunction = oldJsonpFunction;
    
    // Load entry module and return exports 加载入口模块
    return __webpack_require__(__webpack_require__.s = 0);
})([
// 这里是模块列表
]);
```

## 模块加载的逻辑
webpack中的模块加载是通过__webpack_require__函数来实现的，它的作用是，执行模块函数(对于模块，webpack会把它包裹成一个函数), 然后获得模块中的输出，并且缓存模块输出的结果。
```
// The require function
 	function __webpack_require__(moduleId) {

 		// Check if module is in cache 如果模块已经加载缓存，从缓存中获取，这就是为什么多次加载同一个模块，该模块只会执行一次的原因
 		if(installedModules[moduleId]) {
 			return installedModules[moduleId].exports;
 		}
 		// Create a new module (and put it into the cache) 创建一个模块
 		var module = installedModules[moduleId] = {
 			i: moduleId, // 模块id
 			l: false, // 是否加载完成
 			exports: {} // 模块中的输出
 		};

 		// Execute the module function 执行模块函数获得输出，从该语句中可以看出，模块中的this，是指向module.exports的。传入module.exports，所以在模块内对应它的赋值操作，都是模块的输出内容。
 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);

 		// Flag the module as loaded 表示模块已近加载完成
 		module.l = true;

 		// Return the exports of the module返回模块的输出
 		return module.exports;
 	}
```

## 异步chunk下载完成后的处理逻辑
对于异步chunk下载的逻辑
```
// This file contains only the entry chunk.
// The chunk loading function for additional chunks
__webpack_require__.e = function requireEnsure(chunkId) {
    var promises = [];


    // JSONP chunk loading for javascript

    var installedChunkData = installedChunks[chunkId];
    if(installedChunkData !== 0) { // 0 means "already installed".

        // a Promise means "currently loading".
        if(installedChunkData) {
        	// 如果正在加载中，把加载的promise加入到installedChunkData
        	promises.push(installedChunkData[2]);
        } else {
            // setup Promise in chunk cache
            var promise = new Promise(function(resolve, reject) {
            	installedChunkData = installedChunks[chunkId] = [resolve, reject];
            });
            // [resolve, reject, promise]
            promises.push(installedChunkData[2] = promise);

            // start chunk loading 创建script标签去下载chunk
            var script = document.createElement('script');
            var onScriptComplete;

            script.charset = 'utf-8';
            script.timeout = 120;
            if (__webpack_require__.nc) {
            	script.setAttribute("nonce", __webpack_require__.nc);
            }
            script.src = jsonpScriptSrc(chunkId);

            // create error before stack unwound to get useful stacktrace later
            var error = new Error();
            // 下载超时或者失败时的处理函数
            onScriptComplete = function (event) {
                // avoid mem leaks in IE.
                script.onerror = script.onload = null;
                clearTimeout(timeout);
                var chunk = installedChunks[chunkId];
                // 如果下载成功，会执行webpackJsonpCallback，会把installedChunks[chunkId]设置成0，不为0则说明下载失败
                if(chunk !== 0) {
                    if(chunk) {
                        var errorType = event && (event.type === 'load' ? 'missing' : event.type);
                        var realSrc = event && event.target && event.target.src;
                        error.message = 'Loading chunk ' + chunkId + ' failed.\n(' + errorType + ': ' + realSrc + ')';
                        error.name = 'ChunkLoadError';
                        error.type = errorType;
                        error.request = realSrc;
                        chunk[1](error);
                    }
            	installedChunks[chunkId] = undefined;
            	}
     	};
     		// 下载超时的定时器
            var timeout = setTimeout(function(){
            	onScriptComplete({ type: 'timeout', target: script });
            }, 120000);
            // 下载超时或者下载失败时的回调，onload事件下载成功和失败都会触发，但是它的触发时机为脚本下载完成并且执行完成之后才会触发，因为如果是成功执行后，installedChunks[chunkId]会被设置成0，所以不会执行错误提示，如果是失败，这会错误提示
            script.onerror = script.onload = onScriptComplete;
            document.head.appendChild(script);
        }
	}
    // 该函数返回一个promise对象，即使用import()加载模块时，返回一个promise对象
    return Promise.all(promises);
};
```

异步chunk文件中的代码格式：
(window["webpackJsonp"] = window["webpackJsonp"] || []).push(
    [
        [1], // chunksid
        [
        // 模块
        	function(module, __webpack_exports__, __webpack_require__) {},
        ], // 模块列表
    ])
)

如果异步chunk先于webpack runtime代码加载完成那么执行chunk中文件时，push方法是数组的原生方法，所以只是简单地添加到window["webpackJsonp"]数组中。然后会在webpack runtime代码中遍历window["webpackJsonp"]数组，执行webpackJsonpCallback函数：
```
var jsonpArray = window["webpackJsonp"] = window["webpackJsonp"] || [];
jsonpArray = jsonpArray.slice();
for(var i = 0; i < jsonpArray.length; i++) webpackJsonpCallback(jsonpArray[i]);
```
如果异步chunk下载完成时，webpack runtime代码已经加载完成，因为在webpack runtime代码中，会把window["webpackJsonp"]数组的push方法定义成webpackJsonpCallback函数，所以chunk执行时的push方法即为webpackJsonpCallback函数：
```
var oldJsonpFunction = jsonpArray.push.bind(jsonpArray);
jsonpArray.push = webpackJsonpCallback; // 重定义了push方法
var parentJsonpFunction = oldJsonpFunction;
```

所以不管是哪一个先加载完成，最终对chunk的处理都是调用webpackJsonpCallback函数：

### webpackJsonpCallback函数函数的解析：
```
// 接收的参数[[// chunkids], [// 模块列表]]
function webpackJsonpCallback(data) {
	var chunkIds = data[0]; // 获取所有的chunkid
	var moreModules = data[1]; // 获取模块列表

	// add "moreModules" to the modules object,
	// then flag all "chunkIds" as loaded and fire callback
	var moduleId, chunkId, i = 0, resolves = [];
	for(;i < chunkIds.length; i++) {
		chunkId = chunkIds[i];
		if(Object.prototype.hasOwnProperty.call(installedChunks, chunkId) && installedChunks[chunkId]) {
//	对于加载中的installedChunks[chunkId] -> [resolve, reject, promise],
// 所以这里是把下载chunk的promise的resolve函数加进resolves中
			resolves.push(installedChunks[chunkId][0]);
		}
		// 表示已成功安装了chunk
		installedChunks[chunkId] = 0;
	}
	for(moduleId in moreModules) {
		if(Object.prototype.hasOwnProperty.call(moreModules, moduleId)) 	{
		// 把模块赋值给全局modules，modules保存着所有的已加载的模块
			modules[moduleId] = moreModules[moduleId];
		}
	}
	// 如chunk在webpack runtime之后加载完成，那么parentJsonpFunction即为原生数组的push,所以这句代码表示把chunk的相关信息加入到window["webpackJsonp"]中
	if(parentJsonpFunction) parentJsonpFunction(data);

	while(resolves.length) {
		// 执行加载chunk的promise的resolve,去内存中加载对应的模块，因为上面modules[moduleId] = moreModules[moduleId];只是把模块存储到modules中，还没有去加载，所以这里才是真正去加载获取模块的输出，因为对于import()动态加载语句，webpack会转换成:__webpack_require__.e(/* import() */ 1).then(__webpack_require__.bind(null, 4)).then((res) => {
		// 这里是开发者定义的回调
		console.log(res);
	}); 可以看出，执行resolve时，会去执行__webpack_require__，而__webpack_require__函数是从内存中加载模块(执行模块函数)并获得模块的输出
		resolves.shift()();
	}

};
```

## 总结
1.webpack对应模块的处理：
会把模块处理成一个函数，模块中的代码进行函数体，并传入参数module, __webpack_exports__, __webpack_require__，分别为该模块对象，模块输出对象，模块加载函数。同时会把源码中的模块引入语句import/require替换成__webpack_require__。

2.es模块中export语句提升的实现：
会把es模块中的export语句提升到模块函数的开头，并且在输出对象中定义同名的寄存器属性，如：
es模块源码：
```
const name = "hxc";
export function getName () {
	return name;
}
````
会处理成：
```
function(module, __webpack_exports__, __webpack_require__) {

"use strict";
/* harmony export (binding) */ 
// 会在函数的开头创建es模块的导出链接，所以在模块的循环引用中，可能出现访问es块中导出的变量时，报undefined的错误，因为可能改es模块还没完全执行完成，但是通过导出链接，是可以在其他模块中导出的。
__webpack_require__.d(__webpack_exports__, "a", function() { return getName; });
const name ="hxc";
function getName () {
	return name;
}
```

3. 在导入模块中，不能修改导入的变量的实现原理：
对于导出的变量，会在导出的对象中实现同名的寄存器属性的getter，但是没有定义setter：
```
__webpack_require__.d = function(exports, name, getter) {
    if(!__webpack_require__.o(exports, name)) {
    Object.defineProperty(exports, name, { enumerable: true, get: getter });
    }
};
```
所以设置导入的变量时，会报错。

4.异步chunk的加载实现原理：
会把源码中的动态加载语句(如：import())转化成__webpack_require__.e(/* import() */ 春chunkid).then(__webpack_require__.bind(null, 4))
__webpack_require__.e函数会通过创建script标签去下载chunk脚本，下载完成后，会执行webpackJsonpCallback函数来把模块函数保存到全局modules变量中，然后触发resolve，所以会执行then回调__webpack_require__函数。而该函数会执行模块函数获取模块中的输出并且缓存模块输出。然后就可以把模块输出的结果resolve给开发者的then回调函数。

5.tree-shaking删除无用代码的原理
	webpack在处理源码文件时，会加载解析源码文件 -> token -> ast 树 -> 代码生成模块函数。
	在生成ast的时候，依赖于es模块的import、export静态语法分析的能力，能够识别出一些无用的代码，然后生成模块代码时，会通过注释的形式，标记了那些导出的变量没有使用。
	对于无用代码的删除压缩则是交由其他的插件进行处理，如：uglifyjs来处理。
如：
源码模块
```
// es模块
export const name = "hxc";
export const age = 12;
export const color = 344;
export default "esModule";

```
如果该模块只有导出的name属性会被使用到，会被处理成：
```
function(module, __webpack_exports__, __webpack_require__) {

    "use strict";
    /* harmony export (binding) name属性的输出链接 */ __webpack_require__.d(__webpack_exports__, "b", function() { return name; });
    /* unused harmony export age 会标记没有被使用到的导出，然后uglifyjs插件即可根据这些标记信息来进行tree-shaking*/
    /* unused harmony export color */

    // es模块
    const name = "hxc";
    const age = 12;
    const color = 344;
    // 默认输出的链接
    /* harmony default export */ __webpack_exports__["a"] = ("esModule");
}
```

6.模块作用域的实现原理：
webpack对于每一个模块都会处理成一个包裹的函数，又因为每个模块的定义是并列的，所以一个模块的内部(模块函数内部)是不能访问到另一个模块内部的变量。

7.对于es模块中import/exports语句的实现原理：
对于export支持见第2点
对于import的支持：会把import替换成__webpack_require__，并且定义一个变量保存导入的对象，然后在使用的某个变量的地方，替换成该对象的属性：
源码语句：
```
import{ name } from './es.js';
console.log(name)
```

处理成：
```
/* harmony import */ var _es_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(1);

console.log(_es_js__WEBPACK_IMPORTED_MODULE_0__[/* name */ "b"]);

```


