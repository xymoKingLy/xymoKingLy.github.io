---
title: Proxy and Reflect
date: 2018-04-04 01:15:35
categories:
- JavaScript
tags: 
- ES6

---

# Proxy and Reflect

**Reflect** 对象与 **Proxy** 对象都是是 ES6 为了操作对象而提供的新 API。
> **Proxy** 的概述
> 
> >从 **Proxy** 字面意思来理解更好，就是代理器的意思，**Proxy** 会代理你某些对 Object 的操作，你你这些对 Object 的操作都会经过 **Proxy** 过滤和处理。

> **Reflect** 的概述
> >**Reflect** 是一个内置的全局对象，它提供可拦截 `JavaScript`操作的方法。方法与**Proxy**的方法相同。**Reflect** 不是一个函数对象，因此它是不可构造的。与大多数全局对象不同，**Reflect**没有构造函数。您不能将其与一个 `new` 运算符一起使用，也不能将 **Reflect** 对象作为一个函数来调用。

<!-- more -->

## Proxy

ES6 原生提供 `Proxy` 构造函数，用来生成 `Proxy` 实例。

```
var proxy = new Proxy(target, handler);
```
Proxy 对象的所有用法，都是上面这种形式，不同的只是`handler`参数的写法。

 - `target`参数表示所要拦截的目标对象。
 - `handler`参数也是一个对象，用来定制拦截行为。

下面是另一个拦截读取属性行为的例子。

```
var target = {
	name: 'Tom',
	age: 20
};

var proxy = new Proxy(target, {
  get: function(target, property) {
    return 1000;
  }
});

proxy.name 	// 1000
proxy.age 	// 1000
proxy.title	// 1000

target.name 	// Tom
target.age	//	20
target.title // undefined
```

上面代码中，`handler`是一个配置对象, 在这里面配置你想要被代理的Object操作，每一个别被代理的操作都有个自己处理函数。上面的例子中配置对象有个`get`方法，用来拦截目标对象的属性访问操作。`get`方法有两个参数：`target`就是代理的那个对象，`property`是所要访问的目标对象的属性。这个拦截函数无论访问什么属性总是返回1000。

通过上面两组数据的访问，可以看出要：**使得`Proxy`起作用，必须针对`Proxy`实例（上例是`proxy`对象）进行操作，而不是针对目标对象（上例是空对象）进行操作。**

```
var target = {
	name: 'Tom',
	age: 20
};

var hanler = {};

var proxy = new Proxy(target, hanler);

proxy.name 	// Tom
proxy.age 	// 20
proxy.title	// undefined

target.name 	// Tom
target.age	//	20
target.title // undefined
```

**如果`handler`是一个空对象，没有任何拦截效果，访问`proxy`就等同于访问`target`。**



### Proxy 实例的方法
### get()

`get`方法用于拦截某个属性的读取操作。

```javascript
var person = {
  name: 'Tom'
};

var proxy = new Proxy(person, {
  get: function(target, property) {
    if (property in target) {
      return target[property];
    } else {
      throw new ReferenceError("Property \"" + property + "\" does not exist.");
    }
  }
});

proxy.name // Tom
proxy.age  // Uncaught ReferenceError: Property "age" does not exist.
```
通过`Proxy`当访问对象没有定义的属性时不返回`undefined`，而是抛出错误。


### set()

`set`方法用来拦截某个属性的赋值操作。

```javascript
var handler = {
  get (target, key) {
    invariant(key, 'get');
    return target[key];
  },
  
  set (target, key, value) {
    invariant(key, 'set');
    target[key] = value;
    return true;
  }
};

function invariant (key, action) {
  if (key[0] === '_') {
    throw new Error(`Invalid attempt to ${action} private "${key}" property`);
  }
}
var target = {};
var proxy = new Proxy(target, handler);

proxy._prop
// Error: Invalid attempt to get private "_prop" property

proxy._prop = 'c'
// Error: Invalid attempt to set private "_prop" property
```

上面代码中，`invariant`方法是一个判断是否为内部属性（以_开头命名的变量）的一个方法，如果是就抛出异常，通过`Proxy`的`set`和`get`方法可以实现禁止内部属性的读写。

```
var handler = {
  set (target, key, value) {
  	target[key] = value;
    return true;
  }
};

var target = {
	name: 'Tom'
};

Object.defineProperty(target, 'name', {
  configurable: false,
  writable: false
});

var proxy = new Proxy(target, handler);

proxy.name = 'Jam'; 
// TypeError: 'set' on proxy: trap returned truish for property 'name' which exists in the proxy target as a non-configurable and non-writable data property with a different value

proxy.name = 'Tom'; // Tom
```

**如果目标对象自身的某个属性，不可写也不可配置，那么`set`不得改变这个属性的值，只能返回同样的值，否则报错。**

### apply()
`apply`方法拦截函数的调用、`call`和`apply`操作。


```javascript
var target = function () { return 'I am the target'; };
var handler = {
  apply: function () {
    return 'I am the proxy';
  }
};

var p = new Proxy(target, handler);

p()
// "I am the proxy"
```

上面代码中，变量`p`是 Proxy 的实例，当它作为函数调用时（`p()`），就会被`apply`方法拦截，返回`I am the proxy`。

## 实例：Web 服务的客户端

Proxy 对象可以拦截目标对象的任意属性，这使得它很合适用来写 Web 服务的客户端。

```javascript
const service = createWebService('http://example.com/data');

service.employees().then(json => {
  const employees = JSON.parse(json);
  // ···
});
```

上面代码新建了一个 Web 服务的接口，这个接口返回各种数据。Proxy 可以拦截这个对象的任意属性，所以不用为每一种数据写一个适配方法，只要写一个 Proxy 拦截就可以了。

```javascript
function createWebService(baseUrl) {
  return new Proxy({}, {
    get(target, propKey) {
      return () => httpGet(baseUrl+'/' + propKey);
    }
  });
}
```

##用ES5简单的实现一下Proxy

```
/**浅拷贝工具方法**/
function shallowClone(source) {
    if (!source || typeof source !== 'object') {
        throw new Error('error arguments');
    }
    var targetObj = source.constructor === Array ? [] : {};
    for (var keys in source) {
        if (source.hasOwnProperty(keys)) {
            targetObj[keys] = source[keys];
        }
    }
    return targetObj;
}

/*代理实现*/
function ProxyCopy(target,handle){
  var targetCopy = shallowClone(target);
  Object.keys(targetCopy).forEach(function(key){
    Object.defineProperty(targetCopy, key, {
      get: function() {
        return handle.get && handle.get(target,key);
      },
      set: function(newVal) {
        handle.set && handle.set();
        target[key] = newVal;
      }
    });
  })
  return targetCopy;
}

var person = {name:''};
var personCopy = new ProxyCopy(person,{
  get(target,key){
    console.log('get方法被拦截。。。');
    return target[key];
  },
  set(target,key,value){
    console.log('set方法被拦截。。。')
    // return true;
  }
})
person.name = 'arvin';  // 未有拦截日志打出
personCopy.name = 'arvin';  // set方法被拦截。。。
console.log(person.name);   // 未有拦截日志打出
console.log(personCopy.name);   // get方法被拦截。。。

```



## Reflect

（1） 将`Object`对象的一些明显属于语言内部的方法（比如`Object.defineProperty`），放到`Reflect`对象上。现阶段，某些方法同时在`Object`和`Reflect`对象上部署，未来的新方法将只部署在`Reflect`对象上。也就是说，从`Reflect`对象上可以拿到语言内部的方法。

（2） 修改某些`Object`方法的返回结果，让其变得更合理。比如，`Object.defineProperty(obj, name, desc)`在无法定义属性时，会抛出一个错误，而`Reflect.defineProperty(obj, name, desc)`则会返回`false`。

```
// 老写法
try {
  Object.defineProperty(target, property, attributes);
  // success
} catch (e) {
  // failure
}

// 新写法
if (Reflect.defineProperty(target, property, attributes)) {
  // success
} else {
  // failure
}
```

（3） 让`Object`操作都变成函数行为。某些`Object`操作是命令式，比如`name in obj`和`delete obj[name]`，而`Reflect.has(obj, name)`和`Reflect.deleteProperty(obj, name)`让它们变成了函数行为。

```
// 老写法
'assign' in Object // true

// 新写法
Reflect.has(Object, 'assign') // true
```

（4）`Reflect`对象的方法与`Proxy`对象的方法一一对应，只要是`Proxy`对象的方法，就能在`Reflect`对象上找到对应的方法。这就让`Proxy`对象可以方便地调用对应的`Reflect`方法，完成默认行为，作为修改行为的基础。也就是说，不管`Proxy`怎么修改默认行为，你总可以在`Reflect`上获取默认行为。

```
Proxy(target, {
  set: function(target, name, value, receiver) {
    var success = Reflect.set(target,name, value, receiver);
    if (success) {
      log('property ' + name + ' on ' + target + ' set to ' + value);
    }
    return success;
  }
});
```

上面代码中，`Proxy`方法拦截`target`对象的属性赋值行为。它采用`Reflect.set`方法将值赋值给对象的属性，确保完成原有的行为，然后再部署额外的功能。

下面是另一个例子。

```
var loggedObj = new Proxy(obj, {
  get(target, name) {
    console.log('get', target, name);
    return Reflect.get(target, name);
  },
  deleteProperty(target, name) {
    console.log('delete' + name);
    return Reflect.deleteProperty(target, name);
  },
  has(target, name) {
    console.log('has' + name);
    return Reflect.has(target, name);
  }
});
```

上面代码中，每一个`Proxy`对象的拦截操作（`get`、`delete`、`has`），内部都调用对应的`Reflect`方法，保证原生行为能够正常执行。添加的工作，就是将每一个操作输出一行日志。

有了`Reflect`对象以后，很多操作会更易读。

```
// 老写法
Function.prototype.apply.call(Math.floor, undefined, [1.75]) // 1

// 新写法
Reflect.apply(Math.floor, undefined, [1.75]) // 1
```

##Summary
 `Reflect`和`Proxy`中的方法一一对应，没有一一举例子，列举一下。
 
- `get()` `set()` `has()` `deleteProperty()` `difineProperty()`//与属性有关的方法
- `getOwnPropertyDescriptor()` `ownKeys()`//Own的属性描述和属性keys
- `getPrototypeOf()` `setPrototypeOf()`//与原型有关的方法
- `isExtensible()`判断是否可以扩展 preventExtensions()阻止添加新属性
- `apply()`//调用方法有关
- `construct()`//和new 有关的

`Proxy`相当于去修改设置对象的属性行为，而`Reflect`则是获取对象的这些行为。

###Proxy的特点
>1.行为转发与状态同步

>>代理的行为：将代理的所有内部方法转发至目标

>2.proxy !== target

>>```
var obj = {};
var proxy = new Proxy(obj, {});
console.log(obj === proxy);  // false
console.log(obj == proxy);  // false
```

>3.handler拦截

>>可以通过handler对象重写内部方法，拦截并改变target的默认行为

>>未被handler拦截（未在handler对象中重写）的内部方法会直接指向目标

>4.代理关系可以解除

>>代理可以解除，用Proxy.revocable(target, handler)创建返回Object { proxy: Object, revoke: revoke() }，调用revoke()解除代理关系，解除后访问代理对象会报错，例如：

>>```
// 解除代理关系
var rProxy = Proxy.revocable({}, {
    set: function(target, key, value) {
        return Reflect.set(target, key, 0);
    }
});
var p = rProxy.proxy;
p.a = 123;
console.log(p.a);   // 0
// 解除代理
rProxy.revoke();
// p.b = 213;  // 报错TypeError: illegal operation attempted on a revoked proxy
```

###Proxy的一些其他应用场景
>1.防篡改对象
>
>2.分离校验逻辑

>>setter/getter的优点，通过代理机制能将校验逻辑内置于属性访问行为中，比如可以校验逻辑放在这里面

>3.记录对象访问
>>主要用于测试/调试中，比如测试框架，通过代理机制可以全程记录一切

>4.增强普通对象

>>例如防篡改对象，在API设计方面有很大想象空间