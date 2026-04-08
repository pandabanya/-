## Vue3 为何弃用 defineProperty？ 

### 首先 ，Vue2 的Obj.defineProperty和 Vue3的proxy到底是什么

1. Obj.defineProperty （ES5）
定义：直接在一个对象上定义/ 修改一个属性，并可以监听该属性的get 和set行为。
特点：只能监听对象的【单个属性】，无法监听这个对象，还有数组

2. proxy (es6)
定义：创建一个对象的【代理器】，对目标对象的所有操作（读取、赋值、删除、调用、遍历等）进行拦截监听
特点：监听是一个对象，支持13种拦截方式其中就包括了 Obj.definProperty,包括对象、数组、函数、Symbol 等所有引用类型。


### 核心缺陷对比：defineProperty 天生硬伤

Vue 团队弃用的根本原因 以下几点： 

1. 缺陷1： 无法监听【新增/删除】的对象属性

defineProperty必须在初始化就明确了监听对象的每一个属性，如果后期动态添加或者删除属性，监听不到

```javascript
// 1. 定义响应式函数（Vue2 底层简化版）
function defineReactive(obj, key, val) {
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get() {
      console.log(`读取属性：${key}`);
      return val;
    },
    set(newVal) {
      if (newVal === val) return;
      console.log(`修改属性：${key}，新值：${newVal}`);
      val = newVal;
    },
  });
}

// 2. 初始化对象
const user = { name: "张三" };

// 3. 手动监听已有属性
defineReactive(user, "name", user.name);

// 测试 1：读取/修改已有属性 ✅ 正常监听
user.name; // 输出：读取属性：name
user.name = "李四"; // 输出：修改属性：name，新值：李四

// 测试 2：动态新增属性 ❌ 无法监听
user.age = 18; // 无任何输出，响应式失效

// 测试 3：删除属性 ❌ 无法监听
delete user.name; // 无任何输出，响应式失效

```

Proxy 天生支持监听新增、删除对象属性
```javascript
// 1. 创建 Proxy 代理
const user = { name: "张三" };
const proxyUser = new Proxy(user, {
  get(target, key) {
    console.log(`读取属性：${key}`);
    return Reflect.get(target, key);
  },
  set(target, key, newVal) {
    console.log(`修改/新增属性：${key}，新值：${newVal}`);
    return Reflect.set(target, key, newVal);
  },
  deleteProperty(target, key) {
    console.log(`删除属性：${key}`);
    return Reflect.deleteProperty(target, key);
  },
});

// 测试 1：读取/修改已有属性 ✅
proxyUser.name; // 读取属性：name
proxyUser.name = "李四"; // 修改/新增属性：name，新值：李四

// 测试 2：动态新增属性 ✅ 完美监听
proxyUser.age = 18; // 修改/新增属性：age，新值：18

// 测试 3：删除属性 ✅ 完美监听
delete proxyUser.name; // 删除属性：name

```

2. 缺陷2： 无法原生监听数组（hack方案太笨重）

这是 defineProperty 最被诟病的问题：完全不支持监听数组。
因为数组的 push/pop/shift/unshift/splice 等方法，以及通过索引修改数组、修改数组长度，defineProperty 都监听不到。
Vue2 为了解决这个问题，不得不重写数组的 7 个原型方法，做了一层 hack 兼容，但依然有两个致命盲区：

无法监听 arr[index] = val（通过索引赋值）；
无法监听 arr.length = 0（修改长度）。

defineProperty 对数组是「残废状态」，必须 hack 修复，还有盲区；
Proxy 对数组是「完美支持」，原生监听所有操作，零兼容成本。


3. 缺陷3： 深层对象必须「递归遍历」（性能灾难）
defineProperty 只能监听单层属性，如果对象是多层嵌套（比如 user.info.age），必须递归遍历整个对象，给每一个子属性都绑定 defineProperty。

这会带来两个严重问题：

初始化性能差：数据越大，递归遍历越耗时；
内存占用高：所有属性都被提前监听，哪怕从未使用。
Proxy 惰性监听（按需递归，性能拉满） ，proxy监听的是整个对象，不需要提前递归遍历。只有当真正读取到深层对象时，才会递归生成代理（惰性监听）。


4. 缺陷4： 功能支持度有限度

### 性能深度对比：Proxy 全面碾压

1. 初始化速度

defineProperty：需要递归遍历整个对象，数据量越大，速度越慢；
Proxy：直接代理整个对象，不遍历，初始化速度接近 O (1)。

测试结果：1000 个属性的嵌套对象，Proxy 初始化速度比 defineProperty 快 5~10 倍。
2. 内存占用

defineProperty：所有属性都绑定监听，内存占用随属性数量线性增长；
Proxy：只存一个代理对象，惰性递归，内存占用极低。

3. 运行时开销

defineProperty：属性访问直接调用 getter，运行时开销低；
Proxy：通过代理访问，有极微小的开销，但完全可以忽略。





架构级总结：Vue3 为何弃用 defineProperty
一句话核心答案
Proxy 是 ES6 原生提供的对象代理机制,从语言层面解决了 defineProperty 在响应式系统设计上的四大架构缺陷,是 Vue3 响应式系统重构的必然选择。

技术决策的本质：从"属性劫持"到"对象代理"
维度	defineProperty (Vue2)	Proxy (Vue3)
设计理念	属性级劫持（微观控制）	对象级代理（宏观拦截）
监听粒度	单个属性	整个对象
API 能力	2 种拦截（get/set）	13 种拦截（增删改查等）
兼容性	IE9+	现代浏览器（无法 polyfill）
四大架构缺陷深度剖析
缺陷 1：动态属性监听缺失（破坏响应式完整性）
问题本质： defineProperty 必须在初始化时明确声明每个属性,后续动态新增/删除的属性无法被监听。

架构影响：

开发者必须使用 Vue.set() / Vue.delete() 这种"补丁 API"
破坏了 JavaScript 原生对象操作的直觉性
增加了心智负担和代码复杂度
Proxy 解决方案： 通过 set 和 deleteProperty 拦截器,原生支持动态属性的监听,无需任何补丁 API。

缺陷 2：数组监听的 Hack 方案（技术债务）
问题本质： defineProperty 完全无法监听数组的变更方法（push/pop/splice 等）和索引赋值。

Vue2 的妥协方案：

// Vue2 重写了数组的 7 个原型方法
```javascript
const arrayProto = Array.prototype
const arrayMethods = Object.create(arrayProto)

;['push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse'].forEach(method => {
  arrayMethods[method] = function(...args) {
    const result = arrayProto[method].apply(this, args)
    // 手动触发更新
    notify() 
    return result
  }
})
```

架构问题：

侵入式修改原生对象原型（违反开闭原则）
依然无法监听 arr[0] = val 和 arr.length = 0
增加了维护成本和潜在 bug 风险
Proxy 解决方案： 原生支持所有数组操作的拦截,零 hack 成本。

缺陷 3：深层对象的性能灾难（O(n) vs O(1)）
问题本质： defineProperty 只能监听单层属性,嵌套对象必须递归遍历,提前绑定所有属性。

性能对比：

场景	defineProperty	Proxy
初始化	递归遍历所有属性（O(n)）	直接代理对象（O(1)）
内存占用	所有属性都绑定监听	只存一个代理对象
深层访问	提前递归（浪费）	惰性递归（按需）
实测数据： 1000 个嵌套属性的对象,Proxy 初始化速度比 defineProperty 快 5~10 倍。

架构优势： Proxy 的惰性监听机制,完美契合现代前端"按需加载"的设计理念。

缺陷 4：功能扩展性受限
defineProperty 只支持 get 和 set 两种拦截,而 Proxy 支持 13 种拦截操作：
```javascript
new Proxy(target, {
  get,           // 属性读取
  set,           // 属性设置
  has,           // in 操作符
  deleteProperty,// delete 操作符
  ownKeys,       // Object.keys()
  getOwnPropertyDescriptor,
  defineProperty,
  preventExtensions,
  getPrototypeOf,
  isExtensible,
  setPrototypeOf,
  apply,         // 函数调用
  construct      // new 操作符
})

```

架构意义： 为 Vue3 提供了更强的元编程能力,支持更复杂的响应式场景（如 Map/Set/WeakMap 等）。

架构决策的权衡：为什么不早点用 Proxy？
Vue2 时代（2016 年）：

Proxy 刚发布,浏览器兼容性差（IE 完全不支持）
无法通过 Babel 等工具 polyfill（语言层面的特性）
市场上大量 IE 用户,必须兼容
Vue3 时代（2020 年）：

现代浏览器全面支持 Proxy
IE 逐步退出历史舞台
技术债务到了必须重构的时刻
尤雨溪的技术决策：

"Vue3 不再支持 IE11,是为了拥抱更先进的 JavaScript 特性,Proxy 是响应式系统重构的基石。"

面试回答的黄金结构（30 秒版）
第一层（结论）： Vue3 用 Proxy 替代 defineProperty,是因为 Proxy 从语言层面解决了响应式系统的四大缺陷。

第二层（核心问题）：

defineProperty 无法监听动态新增/删除的属性
无法原生监听数组,必须 hack 数组原型
深层对象必须递归遍历,性能差
只支持 get/set,功能扩展性受限
第三层（技术优势）： Proxy 是对象级代理,支持 13 种拦截操作,初始化性能提升 5~10 倍,且支持惰性监听。

第四层（架构权衡）： Vue2 时代受限于 IE 兼容性,Vue3 果断放弃 IE,拥抱现代浏览器特性。

追问应对策略
Q1: Proxy 有什么缺点吗？ A: 唯一缺点是兼容性,IE 完全不支持且无法 polyfill。但 Vue3 的定位就是面向现代浏览器,这是可接受的技术取舍。

Q2: Vue2 的 $set 是怎么实现的？ A: 手动调用 defineReactive 给新属性绑定 getter/setter,然后触发依赖更新。本质是绕过 defineProperty 的限制。

Q3: Proxy 的性能真的比 defineProperty 好吗？ A: 初始化阶段 Proxy 碾压（O(1) vs O(n)）,运行时 defineProperty 略快（直接访问 vs 代理访问）,但差距可忽略。综合来看 Proxy 完胜。

Q4: 为什么 Proxy 无法 polyfill？ A: Proxy 是 JavaScript 引擎层面的特性,需要修改对象的内部槽（internal slot）,无法通过 JavaScript 代码模拟。

记忆口诀（面试现场快速回忆）
"动数深功"四字诀：

动态属性监听缺失
数组监听需要 hack
深层对象性能差
功能扩展性受限