## Vue3 为何弃用 defineProperty？ 

### 首先 ，Vue2 的Obj.defineProperty和 Vue3的proxy到底是什么

1. Obj.defineProperty （ES5）
直接在一个对象上定义/ 修改一个属性，并可以监听该属性的get 和set行为。
特点：只能监听对象的【单个属性】，无法监听这个对象，还有数组

2. proxy (es6)
