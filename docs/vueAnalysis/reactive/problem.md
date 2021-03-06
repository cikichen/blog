# 变化侦测注意事项
虽然`Object.defineProperty()`方法很好用，但也会存在一些例外情况，这些例外情况的变动不能触发`setter`。这种情况，我们分为对象和数组两类来分析。

## 对象
假设我们有如下例子：
```js
export default {
  data () {
    return {
      obj: {
        a: 'a'
      }
    }
  },
  created () {
    // 1.新增属性b，属性b不是响应式的，不会触发obj的setter
    this.obj.b = 'b'
    // 2.delete删除已有属性，无法触发obj的setter
    delete this.obj.a
  }
}
```
从以上例子我们可以看到：
* 当为一个响应式对象新增一个属性的时候，新增的属性不是响应式的，后续对于这个新增属性的任何修改，都无法触发其`setter`。为了解决这种问题，`Vue.js`提供了一个全局的`Vue.set()`方法和实例`vm.$set()`方法，它们实际上都是同一个`set`方法，我们会在后续的章节中介绍与响应式相关的全局`API`的实现。
* 当一个响应式对象删除一个已有属性的时候，不会触发`setter`。为了解决这个问题，`Vue.js`提供了一个全局的`vue.delete()`方法和实例`vm.$delete()`方法，它们实际上都是同一个`del`方法，我们会在后续的章节中介绍与响应式相关的全局`API`的实现。

## 数组
假设我们有如下例子：
```js
export default {
  data () {
    return {
      arr: [1, 2, 3]
    }
  },
  created () {
    // 1.通过索引进行修改，无法捕获到数组的变动。
    this.arr[0] = 11
    // 2.通过修改数组长度，无法捕获到数组的变动。
    this.arr.length = 0
  }
}
```
从以上例子我们可以看到：
* 通过索引直接修改数组，无法捕捉到数组的变动。
* 通过修改数组长度，无法捕获到数组的变动。

对于第一种情况，我们可以使用前面提到过的`Vue.set`或者`vm.$set`来解决，对于第二种方法，我们可以使用数组的`splice()`方法解决。

在最新版`Vue3.0`中，使用到了`Proxy`来代替`Object.defineProperty()`实现响应式，使用`Proxy`后以上问题全部可以解决，然而`Proxy`属于`ES6`的内容，因此对于浏览器兼容性方面有一定的要求。