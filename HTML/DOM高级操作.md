操作DOM的高级API
================
旧浏览器提供了一些操作DOM对象的接口，如getElementById等API，但是有时操作Dom起来还是有些麻烦，所以现代浏览器提供了一些高级的API，方便对Dom进行操作，以下为一些高级的操作Dom的API,大多的现代浏览器都支持，所以兼容性还是比较好的。

## 向下获取dom节点
- element.querySelector(selector): 在element的所有子节点搜索目标dom节点(查找满足条件的第一个节点)。其中selector为css语法的节点选择器。
- element.querySelectorAll(selector): 在element的所有子节点搜索目标dom节点(查找满足条件的所有节点)。其中selector为css语法的节点选择器。

## 向上获取dom节点
旧浏览器都没有提供从目标元素先父级元素查找的API，现代浏览器提供这样的一个API，element.closest(selector)。该API能够从目标节点element向它的所有父级节点查找满足条件的一个节点（也可以是自身）。其中selector为css选择器。如果达到文档根节点还没有找到，这返回null。

如：
```
<div class="parent1">
    <div class="parent2">
        <div class="parent3">
        	<span class="child">孩子节点</span>
        </div>
    </div>
</div>

<script>
	var child = querySelector('.child')
	var parent2 = child.closest('.parent2') // 返回child的父节点parent2

	var parent4 = child.closest('.child') // 可以为自身
	
	var parnet0 = child.closest('.parent0') // null，找不到时，返回null
</script>
```

## 插入节点
- element.insertAdjacentHTML(position, DomString):
  该API用于在元素中插入html片段。
  position(字符串):
  1.beforebegin: 在元素节点element之前插入html片段。
  2.afterbegin: 在元素节点element第一个子元素之前插入html片段。
  3.beforeend:在元素节点element的最后一个子元素之后插入html片段。
  4.afterend:在元素节点element之后插入html片段。

     ```
     beforebegin<div id="container">afterbegin<span>1111</span>beforeend</div>afterend

     <script>
        var div = document.querySelector('#container')
        div.insertAdjacentHTML('afterbegin','<div>111</div><div>222</div>')
     </script>
     ```

- element.insertAdjacentElement(position, DomString):
  position和insertAdjacentHTML一致，insertAdjacentElement与insertAdjacentHTML不同的是，insertAdjacentElement用于插入一个元素，而insertAdjacentHTML可以插入html片段（可以是多个元素）。
  ```
  <script>
 	var div = document.querySelector('#container')
 	div.insertAdjacentElement('afterbegin','<div><span>111</span><span>222</span></div>')
 </script>
  ```
  
- element.insertAdjacentText(position, DomString):
	用法和element.insertAdjacentElement相似，不同的是插入的是文本节点。


## 移动元素
element.insertAdjacentElement API除了可以用来插入节点之外，也可以用来移动节点。
当传入的第二个参数是文档中已经存在的节点时，该API不会插入新的节点，而是把相应的节点移动到对应的位置。
```
<h1>1111</h1>
<h2>2222</h2>

<script>
	var h1 = document.querySelector('h1')
	var h2 = document.querySelector('h2')
	// 把h1移到h2的后面
	h2.insertAdjacentElement('afterend', h1)
</script>
```

## 替换元素
旧浏览器提供替换元素的API为replaceChild,不过操作起来比较麻烦，还需要获取直接父元素。
parent.replaceChild(newChild, oldChild)
而新的replaceWith API更加方便,不需要先获取到直接父级元素再进行替换：
oldNode.replaceWith(newNode)

## 移除元素
老的移除节点的removeChild Api需要先获取直接的父元素，再进行移除：
target.parent.removeChild(target)
而新的remove API只需要获取到目标元素即可移除
target.remove()

## 创建节点
旧的Api创建一个节点时，是比较麻烦的，需要先创建元素，然后再设置元素的属性。
```
var a = document.createElement('a')
a.alt = '图片'
a.href = 'url'
```
而新的API可以通过dom字符串来创建一个节点,方便了很多
```
// 创建一个Dom节点
function createElement (domString) {
    var parser = new DOMParser()
    // 返回dom对象
    return parser.parseFromString(domString, 'text/html').body.firstChild
  }

// 可以通过字符串很容易就可以创建一个节点
var a = createElement(‘<a href="url" alt="图片"></a>’)
  
```

## 搜索(检查)dom节点
- element.matches(selector): 判断一个节点是否匹配selector选择器.
  ```
  <div id="container" class="container"></div>
  
  <script>
	var div = document.querySelector('#container')
	console.log(div.matches('.container')) //true
	console.log(div.matches('.abc')) // false
  </script>
  ```
- element.contains(el):判断element是否包含子节点el.
  ```
	<div id="container" class="container">
        <p class="p_el">index</p>
        <div class="qq">erewter</div>
   </div>  
   <a id="a_el" href="#"></a>
   
   <script>
   	var div = document.querySelector('#container')
   	var p = document.querySelector('.p_el')
   	var a = document.querySelector('#a_el')
   	
   	div.contains(p) // true
   	div.contains(a) // false
   </script>
  ```
  
  ## 观察Dom节点的变化
  现代浏览器提供了MutationObserver API来监听DOM节点的变化，包扣节点属性，文本，孩子节点的增删改等的变化.
  ```
  <div id="container" class="container"></div>
  <script>
   // 观测到变化后的回调
  	var callback =  function (mutationRecords, observer) {
  		/* mutationRecords 数组，数组每一项都是一个变更记录，它是一个普通的对象
            type: 变更的类型，attributes / characterData / childList
            target: 发生变更的 DOM 元素
            addedNodes: 新增子元素组成的 NodeList
            removedNodes: 已移除子元素组成的的 NodeList
            attributeName: 值发生改变的属性名，如果不是属性变更，则返回 null
            previousSibling: 被添加或移除的子元素之前的兄弟节点
            nextSibling: 被添加或移除的子元素之后的兄弟节点
        */
  		// observer 观测实例本身
  	}
  	var observer = new MutationObserver(callback)
  	var div = document.querySelector('#container')
  	observer.observer(div, {
  	// options
  	/*
  	attributes: Boolean，是否监听元素属性的变化
    attributeFilter: String[]，需要监听的特定属性名称组成的数组
    attributeOldValue: Boolean，当监听元素的属性发生变化时，是否记录并传递属性的上一个值
    characterData: Boolean，是否监听目标元素或子元素树中节点所包含的字符数据的变化
    characterDataOldValue: Boolean，字符数据发生变化时，是否记录并传递其上一个值
    childList: Boolean，是否监听目标元素添加或删除子元素
    subtree: Boolean，是否扩展监视范围到目标元素下的整个子树的所有元素
    */
  	})
  	
  	observer.disconnect：解除监听
  	observer.takeRecords 方法从 observer 的通知队列中删除所有待处理的通知
  </script>
  ```
  
  ## 判断两个节点的位置关系
  element.compareDocumentPosition(otherElement) 是一个强大的 API ，它可以快速判断出两个 DOM 元素的位置关系，诸如：先于、跟随、是否包含。它返回一个整数，代表了两个元素之间的关系。
- 1: 两个元素不在同一个文档内
- 2: otherElement 在 element 之前
- 4: otherElement 在 element 之后
- 8: otherElement 包含 element
- 16: otherElement 被 element 所包含

```
<div id="container" class="container">
   <p class="p_el">index</p>
   <div class="qq">erewter</div>
</div>  
<script>
	var div = document.querySelector('#container')
   	var p = document.querySelector('.p_el')
   	div.compareDocumentPosition(p) //  20 -> p被div所包含: 16, p在div之后：4. 所以 16 + 4 = 20
</script>
```




