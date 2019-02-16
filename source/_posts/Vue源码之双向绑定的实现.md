---
title: Vue源码之双向绑定的实现
date: 2018-04-26 17:11:30
tags: 
- vue
---
## 入口 initData
代码： 

``` haskell
// core/instance/state.js
function getData (data: Function, vm: Component): any {
  try {
    return data.call(vm, vm)
  } catch (e) {
    handleError(e, vm, `data()`)
    return {}
  }
}

function initData (vm: Component) {
  let data = vm.$options.data
  
  // data 如果是函数 则调用getData方法，采用闭包的形式可以使每个实例中的data数据都是独立的，不会互相混淆
  data = vm._data = typeof data === 'function'
    ? getData(data, vm)
    : data || {}
  if (!isPlainObject(data)) {
    data = {}
    process.env.NODE_ENV !== 'production' && warn(
      'data functions should return an object:\n' +
      'https://vuejs.org/v2/guide/components.html#data-Must-Be-a-Function',
      vm
    )
  }
  // proxy data on instance
  const keys = Object.keys(data)
  const props = vm.$options.props
  const methods = vm.$options.methods
  let i = keys.length
  
  // 循环判定methods,props, data中是否有重复定义的名称
  while (i--) {
    const key = keys[i]
    if (process.env.NODE_ENV !== 'production') {
      if (methods && hasOwn(methods, key)) {
        warn(
          `Method "${key}" has already been defined as a data property.`,
          vm
        )
      }
    }
    if (props && hasOwn(props, key)) {
      process.env.NODE_ENV !== 'production' && warn(
        `The data property "${key}" is already declared as a prop. ` +
        `Use prop default value instead.`,
        vm
      )
    } else if (!isReserved(key)) {
      proxy(vm, `_data`, key)
    }
  }
  // observe data
  observe(data, true /* asRootData */)
}
```
1.  data 如果是函数 则调用getData方法，采用闭包的形式可以使每个实例中的data数据都是独立的，不会互相混淆，
2.  循环判定methods,props, data中是否有重复定义的名称，有则报错。
3.  启动observe，改造data中的数据
<!--more-->
## Observe观察者

### observe函数

``` cs
//  core/observe/index.js

function observe (value: any, asRootData: ?boolean): Observer | void {
  if (!isObject(value) || value instanceof VNode) {
    return
  }
  let ob: Observer | void
  
  // 判断是否已经实例化， 如果有__ob__ 说明已经实例化，直接取obser实例，如果没有则开始实例化。
  if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
    ob = value.__ob__
  } else if (
    observerState.shouldConvert &&
    !isServerRendering() &&
    (Array.isArray(value) || isPlainObject(value)) &&
    Object.isExtensible(value) &&
    !value._isVue
  ) {
    ob = new Observer(value)
  }
  if (asRootData && ob) {
    ob.vmCount++
  }
  return ob
}
```


### Observe构造函数  
``` typescript
//  core/observe/index.js

  constructor (value: any) {
    this.value = value
    this.dep = new Dep()
    this.vmCount = 0
	
	// 在data中添加一个__ob__的属性，值为当前observe实例
    def(value, '__ob__', this)
	
	// 如果是数组，对数组中的每个值递归执行observe
    if (Array.isArray(value)) {
      const augment = hasProto
        ? protoAugment
        : copyAugment
      augment(value, arrayMethods, arrayKeys)
      this.observeArray(value)
    } else {
      this.walk(value)
    }
  }
  
    observeArray (items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])
    }
  }
}
  
  // 对对象中的所有属性进行改造。
    walk (obj: Object) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i], obj[keys[i]])
    }
  }
  
  // util/lang.js
  function def (obj: Object, key: string, val: any, enumerable?: boolean) {
  Object.defineProperty(obj, key, {
    value: val,
    enumerable: !!enumerable,
    writable: true,
    configurable: true
  })
}
```


### defineReactive函数
1.defineReactive函数是一个闭包，也就是只有在访问get方法的时候，才能访问其中的dep对象，所以每个属性在调用defineReactive的时候都会生成一个闭包，其中的dep对象都是独立的。在get方法中，首先使用原来定义的get方法获取值。
2.判断是否有观察者（Dep.target）,如果有则把改观察者添加到订阅数组中，这地方很绕，可以看dep.js和watcher.js中的三个函数。
3.dep中的depend，也就是这里调用的函数：如果有观察者（watcher），调用它的addDdep方法,参数是当前dep对象。
``` aspectj
//  dep.js
  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }
```
4.watcher中的addDep方法：其中newDepIds是一个不重复的set的数据结构，可以在watcher中的构造函数中看出来（this.newDepIds = new Set()，参数是当前的watcher对象。

``` objectivec
// watcher.js
  addDep (dep: Dep) {
    const id = dep.id
    if (!this.newDepIds.has(id)) {
	// 把具有唯一性的id属性加入到newDepIds中，同时吧dep实例也加入到newDeps中
      this.newDepIds.add(id)
      this.newDeps.push(dep)
      if (!this.depIds.has(id)) {
        dep.addSub(this)
      }
    }
  }
```
5.调用dep.addSub(this)， 把当前的watcher实例push到dep的依赖管理数组中

``` perl
//  dep.js
  addSub (sub: Watcher) {
    this.subs.push(sub)
  }
```
6. 前面几步完成的是一个属性依赖收集的过程，当属性的值被重写，也就是调用了它的set方法的时候，会同时调用它的dep对象中的notify()，遍历依赖收集数组中的watcher， 调用watcher的update方法来更新。

``` cs
export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  const dep = new Dep()

  const property = Object.getOwnPropertyDescriptor(obj, key)
  if (property && property.configurable === false) {
    return
  }

  // cater for pre-defined getter/setters
  const getter = property && property.get
  const setter = property && property.set

 // 如果有子属性，也把该watcher添加到子属性的依赖收集中。确保子属性变化的时候也能通知更新。
  let childOb = !shallow && observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
	// 用原生的get方法获取值
      const value = getter ? getter.call(obj) : val
	  
	  // import Dep from './dep' Dep是由dep中导出
      if (Dep.target) {
        dep.depend()
		
		// 把watcher也添加到childOb这个observe对象的依赖收集中（observe对象没有get和set，不知道为什么也要添加）
        if (childOb) {
          childOb.dep.depend()
          if (Array.isArray(value)) {
            dependArray(value)
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      /* eslint-disable no-self-compare */
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      childOb = !shallow && observe(newVal)
      dep.notify()
    }
  })
}
```


## Dep 依赖管理

``` typescript
constructor () {
    this.id = uid++
    this.subs = []
  }

  addSub (sub: Watcher) {
    this.subs.push(sub)
  }

  removeSub (sub: Watcher) {
    remove(this.subs, sub)
  }

  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }

// 当属性被重写的时候会调用该方法，将依赖该属性的订阅者全部更新。
  notify () {
    // stabilize the subscriber list first
    const subs = this.subs.slice()
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}
```

上面的流程中，从开始初始化数据，为每个属性添加get，set以及独立的dep对象后，其他都是从调用属性的get方法开始的，因为需要获取该属性值的事物才能对其产生依赖。那是什么东西调用了get方法，就是watcher订阅者。

## Watcher 订阅者


``` typescript
  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: ?Object,
    isRenderWatcher?: boolean
  ) {
    this.vm = vm
	...
 	... 省略部分代码
	...
    // parse expression for getter
	// 如果是函数直接赋值，如果是xxx.xx.xx访问形式的，则调用parsePath方法。
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn
    } else {
      this.getter = parsePath(expOrFn)
      if (!this.getter) {
        this.getter = function () {}
        process.env.NODE_ENV !== 'production' && warn(
          `Failed watching path: "${expOrFn}" ` +
          'Watcher only accepts simple dot-delimited paths. ' +
          'For full control, use a function instead.',
          vm
        )
      }
    }
    this.value = this.lazy
      ? undefined
      : this.get()
  }

  get () {
    pushTarget(this)
    let value
    const vm = this.vm
    try {
      value = this.getter.call(vm, vm)
    } catch (e) {
     ...
 	... 省略部分代码
	...
    }
    return value
  }
  
  // util/lang.js
 const bailRE = /[^\w.$]/
export function parsePath (path: string): any {
  if (bailRE.test(path)) {
    return
  }
  const segments = path.split('.')
  return function (obj) {
    for (let i = 0; i < segments.length; i++) {
      if (!obj) return
      obj = obj[segments[i]]
    }
    return obj
  }
}

```
1. 首先，watcher的构造函数中，声明了getter方法。这里想说明的是，在parsePath函数中，返回函数的参数obj在后面调用的时候 value = this.getter.call(vm, vm)，传入的参数是vm实例，这里的实例直接访问了data中的属性（还能访问computed中的），这是vue对属性都设置了一层代理，可以直接通过实例（如vm）直接访问。

``` javascript
// vm是vue实例，_data是实例中存放data属性的属性，key是data中每个值的key值。
proxy(vm, `_data`, key)

export function proxy (target: Object, sourceKey: string, key: string) {
  sharedPropertyDefinition.get = function proxyGetter () {
    return this[sourceKey][key]
  }
  sharedPropertyDefinition.set = function proxySetter (val) {
    this[sourceKey][key] = val
  }
  Object.defineProperty(target, key, sharedPropertyDefinition)
}
```
2. 构造函数中调用watcher的get（）方法，其中有个pushTarget方法，也就是在收集依赖之前，会将Dep.target设置为当前watcher，也就是之前经常提到的那个属性。

``` fortran
export function pushTarget (_target: Watcher) {
  if (Dep.target) targetStack.push(Dep.target)
  Dep.target = _target
}
```


3. 调用了getter方法之后，就触发了属性的get方法，依赖就收集了。

但是，watcher对象又是怎么实例化的？一共有四种方式：

1. Vue实例化的过程中有watch选项

2. Vue实例化的过程中有computed计算属性选项

3. Vue原型上有挂载$watch方法: Vue.prototype.$watch，可以直接通过实例调用this.$watch方法

4. Vue生成了render函数，更新视图时

