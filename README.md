# deep-copy

## 深拷贝和浅拷贝的定义

浅拷贝：
![pic](https://user-gold-cdn.xitu.io/2019/9/1/16ce894a1f1b5c32?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> 创建一个新对象，这个对象有着原始对象属性值的一份精确拷贝。如果属性是基本类型，拷贝的就是基本类型的值，如果属性是引用类型，拷贝的就是内存地址 ，所以如果其中一个对象改变了这个地址，就会影响到另一个对象。

深拷贝：
![pic](https://user-gold-cdn.xitu.io/2019/9/1/16ce893a54f6c13d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> 将一个对象从内存中完整的拷贝一份出来,从堆内存中开辟一个新的区域存放新对象,且修改新对象不会影响原对象

话不多说，浅拷贝就不再多说，下面我们直入正题：

## 乞丐版
---
在不使用第三方库的情况下，我们想要深拷贝一个对象，用的最多的就是下面这个方法。
```
JSON.parse(JSON.stringify());
```

这种写法非常简单，而且可以应对大部分的应用场景，但是它还是有很大缺陷的，比如拷贝其他引用类型、拷贝函数、循环引用等情况。

显然，面试时你只说出这样的方法是一定不会合格的。

接下来，我们一起来手动实现一个深拷贝方法。

## 基础版本
---
如果是浅拷贝的话，我们可以很容易写出下面的代码：
```
function clone(clone) {
  let cloneTarget = {}
  for (let key in clone) {
    cloneTarget[key] = clone[key]
  }
  return cloneTarget
}
```

创建一个新的对象，遍历需要克隆的对象，将需要克隆对象的属性依次添加到新对象上，返回。
如果是深拷贝的话，考虑到我们要拷贝的对象是不知道有多少层深度的，我们可以用递归来解决问题，稍微改写上面的代码：

如果是原始类型，无需继续拷贝，直接返回
如果是引用类型，创建一个新的对象，遍历需要克隆的对象，将需要克隆对象的属性执行深拷贝后依次添加到新对象上。

很容易理解，如果有更深层次的对象可以继续递归直到属性为原始类型，这样我们就完成了一个最简单的深拷贝：

```
function clone(target) {
  if (typeof target === 'object') {
    let cloneTarget = {}
    for (let key in target) {
      cloneTarget[key] = clone(target[key])
    }
    return cloneTarget;
  } else {
    return target;
  }
}
```

我们可以打开测试代码中的clone1.test.js对下面的测试用例进行测试：
```
const target = {
    field1: 1,
    field2: undefined,
    field3: 'ConardLi',
    field4: {
        child: 'child',
        child2: {
            child2: 'child2'
        }
    }
};
```

这是一个最基础版本的深拷贝，这段代码可以让你向面试官展示你可以用递归解决问题，但是显然，他还有非常多的缺陷，比如，还没有考虑数组。

## 考虑数组
---
在上面的版本中，我们的初始化结果只考虑了普通的`object`，下面我们只需要把初始化代码稍微一变，就可以兼容数组了：

```
function clone(target) {
  if (typeof target === 'object') {
    let cloneTarget = Array.isArray(target) ? [] : {}
    for (let key in target) {
      cloneTarget[key] = clone(target[key])
    }
    return cloneTarget
  } else {
    return target
  }
}
```

在clone2.test.js中执行下面的测试用例：

```
const target = {
    field1: 1,
    field2: undefined,
    field3: {
        child: 'child'
    },
    field4: [2, 4, 8]
};
```

## 循环引用
---
我们执行下面这样一个测试用例：
```
const target = {
    field1: 1,
    field2: undefined,
    field3: {
        child: 'child'
    },
    field4: [2, 4, 8]
};
target.target = target;

```
很明显，因为递归进入死循环导致栈内存溢出了。

原因就是上面的对象存在循环引用的情况，即对象的属性间接或直接的引用了自身的情况：

解决循环引用问题，我们可以额外开辟一个存储空间，来存储当前对象和拷贝对象的对应关系，当需要拷贝当前对象时，先去存储空间中找，有没有拷贝过这个对象，如果有的话直接返回，如果没有的话继续拷贝，这样就巧妙化解的循环引用的问题。

这个存储空间，需要可以存储`key-value`形式的数据，且`key`可以是一个引用类型，我们可以选择`Map`这种数据结构：
+ 检查map中有无克隆过的对象
+ 有 - 直接返回
+ 没有 - 将当前对象作为key，克隆对象作为value进行存储
+ 继续克隆

```
function clone(target, map = new Map()) {
  if (typeof target === 'object') {
    let cloneTarget = Array.isArray(target) ? [] : {}
    if (map.get(target)) {
      return map.get(target)
    }
    map.set(target, cloneTarget)
    for (let key in target) {
      cloneTarget[key] = clone(target[key], map)
    }
    return cloneTarget
  } else {
    return target
  }
}
```

再来执行上面的测试用例：
可以看到，执行没有报错，且`target`属性，变为了一个`Circular`类型，即循环应用的意思。

接下来，我们可以使用，`WeakMap`提代`Map`来使代码达到画龙点睛的作用。

```
function clone(target, map = new WeakMap()) {
  if (typeof target === 'object') {
    let cloneTarget = Array.isArray(target) ? [] : {}
    if (map.get(target)) {
      return map.get(target)
    }
    map.set(target, cloneTarget)
    for (let key in target) {
      cloneTarget[key] = clone(target[key], map)
    }
    return cloneTarget
  } else {
    return target
  }
}
```

为什么要这样做呢？，先来看看`WeakMap`的作用：
```
WeakMap 对象是一组键/值对的集合，其中的键是弱引用的。其键必须是对象，而值可以是任意的。
```

什么是弱引用呢？
```
在计算机程序设计中，弱引用与强引用相对，是指不能确保其引用的对象不会被垃圾回收器回收的引用。 一个对象若只被弱引用所引用，则被认为是不可访问（或弱可访问）的，并因此可能在任何时刻被回收。
```

我们默认创建一个对象：`const obj = {}`，就默认创建了一个强引用的对象，我们只有手动将`obj = null`，它才会被垃圾回收机制进行回收，如果是弱引用对象，垃圾回收机制会自动帮我们回收。
举个例子：

如果我们使用`Map`的话，那么对象间是存在强引用关系的：

```
let obj = { name : 'ConardLi'}
const target = new Map();
target.set(obj,'code秘密花园');
obj = null;
```
虽然我们手动将`obj`，进行释放，然是`target`依然对`obj`存在强引用关系，所以这部分内存依然无法被释放。
再来看`WeakMap`：
```
let obj = { name : 'ConardLi'}
const target = new WeakMap();
target.set(obj,'code秘密花园');
obj = null;
```
如果是`WeakMap`的话，t`arget`和`obj`存在的就是弱引用关系，当下一次垃圾回收机制执行时，这块内存就会被释放掉。
设想一下，如果我们要拷贝的对象非常庞大时，使用Map会对内存造成非常大的额外消耗，而且我们需要手动清除Map的属性才能释放这块内存，而`WeakMap`会帮我们巧妙化解这个问题。

## 性能优化
---
在上面的代码中，我们遍历数组和对象都使用了for in这种方式，实际上for in在遍历时效率是非常低的，我们来对比下常见的三种循环for、while、for in的执行效率：

while的效率是最好的，所以，我们可以想办法把for in遍历改变为while遍历。

我们先使用while来实现一个通用的forEach遍历，iteratee是遍历的回掉函数，他可以接收每次遍历的value和index两个参数：
```
function forEach(array, iteratee) {
  let index = -1
  const length = array.length
  while (++index < length) {
      iteratee(array[index], index);
  }
  return array;
}
```

下面对我们的clone函数进行改写：当遍历数组时，直接使用forEach进行遍历，当遍历对象时，使用Object.keys取出所有的key进行遍历，然后在遍历时把forEach会调函数的value当作key使用：

```
function clone(target, map = new WeakMap()) {
  if (typeof target === 'object') {
    const isArray = Array.isArray(target)
    let cloneTarget = isArray ? [] : {}
    if (map.get(target)) {
      return map.get(target)
    }
    map.set(target, cloneTarget)
    
    const keys = isArray ? undefined : Object.keys(target)
    forEach(keys || target, (value, key) => {
      if (keys) {
        key = value
      }
      cloneTarget[key] = clone(target[key], map)
    })
    return cloneTarget
  } else {
    return target
  }
}
```

下面，我们执行clone4.test.js分别对上一个克隆函数和改写后的克隆函数进行测试：

```
const target = {
    field1: 1,
    field2: undefined,
    field3: {
        child: 'child'
    },
    field4: [2, 4, 8],
    f: { f: { f: { f: { f: { f: { f: { f: { f: { f: { f: { f: {} } } } } } } } } } } },
};

target.target = target;

console.time();
const result = clone1(target);
console.timeEnd();

console.time();
const result2 = clone2(target);
console.timeEnd();
```

很明显，我们的性能优化是有效的。

到这里，你已经向面试官展示了，在写代码的时候你会考虑程序的运行效率，并且你具有通用函数的抽象能力。

## 其他数据类型
---
在上面的代码中，我们其实只考虑了普通的object和array两种数据类型，实际上所有的引用类型远远不止这两个，还有很多，下面我们先尝试获取对象准确的类型。
### 合理的判断引用类型
首先，判断是否为引用类型，我们还需要考虑function和null两种特殊的数据类型：
```
function isObject(target) {
  const type = typeof target
  return target !== null && (type === 'object' || type === 'function')
}
```

```
 if (!isObject(target)) {
     return target;
 }
  // ...
```

### 获取数据类型
我们可以使用toString来获取准确的引用类型：
> 每一个引用类型都有toString方法，默认情况下，toString()方法被每个Object对象继承。如果此方法在自定义对象中未被覆盖，toString() 返回 "[object type]"，其中type是对象的类型。
注意，上面提到了如果此方法在自定义对象中未被覆盖，toString才会达到预想的效果，事实上，大部分引用类型比如Array、Date、RegExp等都重写了toString方法。
我们可以直接调用Object原型上未被覆盖的toString()方法，使用call来改变this指向来达到我们想要的效果。
```
function getType(target) {
  return Object.prototype.toString.call(target)
}
```

下面我们抽离出一些常用的数据类型以便后面使用：
```
const mapTag = '[object Map]';
const setTag = '[object Set]';
const arrayTag = '[object Array]';
const objectTag = '[object Object]';

const boolTag = '[object Boolean]';
const dateTag = '[object Date]';
const errorTag = '[object Error]';
const numberTag = '[object Number]';
const regexpTag = '[object RegExp]';
const stringTag = '[object String]';
const symbolTag = '[object Symbol]';
```
在上面的集中类型中，我们简单将他们分为两类：

+ 可以继续遍历的类型
+ 不可以继续遍历的类型
我们分别为它们做不同的拷贝。

### 可继续遍历的类型
上面我们已经考虑的object、array都属于可以继续遍历的类型，因为它们内存都还可以存储其他数据类型的数据，另外还有Map，Set等都是可以继续遍历的类型，这里我们只考虑这四种，如果你有兴趣可以继续探索其他类型。


