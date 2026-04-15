# flex: 1 详解 - 面试必考

## 一、flex: 1 是什么?

`flex: 1` 是一个**简写属性**,它实际上是三个属性的缩写:

```css
.item {
  flex: 1;
  
  /* 等价于 */
  flex-grow: 1;      /* 放大比例 */
  flex-shrink: 1;    /* 缩小比例 */
  flex-basis: 0%;    /* 基础大小 */
}
```

**面试重点:** 很多人以为 `flex: 1` 等于 `flex-grow: 1`,这是错的!

---

## 二、三个属性详解

### 1. flex-grow (放大比例)

**作用:** 当容器有**剩余空间**时,子项如何分配这些空间

```css
/* 场景:容器 600px,三个子项各 100px,剩余 300px */
.container {
  display: flex;
  width: 600px;
}

.item1 { width: 100px; flex-grow: 1; }  /* 分到 150px */
.item2 { width: 100px; flex-grow: 1; }  /* 分到 150px */
.item3 { width: 100px; flex-grow: 0; }  /* 不分配,保持 100px */
```

**计算公式:**
```
剩余空间 = 容器宽度 - 所有子项基础宽度之和
每个子项分到的空间 = 剩余空间 × (自己的 flex-grow / 所有 flex-grow 之和)

例子:
剩余空间 = 600 - (100 + 100 + 100) = 300px
item1 分到 = 300 × (1 / (1+1+0)) = 150px
item1 最终宽度 = 100 + 150 = 250px
```

### 2. flex-shrink (缩小比例)

**作用:** 当容器**空间不足**时,子项如何缩小

```css
/* 场景:容器 300px,三个子项各 200px,超出 300px */
.container {
  display: flex;
  width: 300px;
}

.item1 { width: 200px; flex-shrink: 1; }  /* 会缩小 */
.item2 { width: 200px; flex-shrink: 2; }  /* 缩小更多 */
.item3 { width: 200px; flex-shrink: 0; }  /* 不缩小,保持 200px */
```

**计算公式:**
```
溢出空间 = 所有子项宽度之和 - 容器宽度
每个子项缩小的空间 = 溢出空间 × (自己的宽度 × flex-shrink) / Σ(每项宽度 × flex-shrink)

例子:
溢出空间 = (200 + 200 + 200) - 300 = 300px
item1 缩小 = 300 × (200×1) / (200×1 + 200×2 + 200×0) = 100px
item1 最终宽度 = 200 - 100 = 100px
```

### 3. flex-basis (基础大小)

**作用:** 在分配剩余空间之前,子项的**初始大小**

```css
.item {
  flex-basis: 200px;  /* 初始宽度 200px */
  flex-basis: 0;      /* 初始宽度 0 */
  flex-basis: auto;   /* 根据内容或 width 决定 */
}
```

**优先级:**
```
flex-basis > width > 内容宽度
```

---

## 三、常见的 flex 简写

### 1. flex: 1

```css
flex: 1;

/* 完整写法 */
flex-grow: 1;
flex-shrink: 1;
flex-basis: 0%;
```

**含义:** 
- 可以放大(grow: 1)
- 可以缩小(shrink: 1)
- 初始大小为 0(basis: 0%)

**实际效果:** 所有设置 `flex: 1` 的子项会**平分容器空间**

```html
<div class="container">
  <div class="item">A</div>  <!-- flex: 1 -->
  <div class="item">B</div>  <!-- flex: 1 -->
  <div class="item">C</div>  <!-- flex: 1 -->
</div>

<!-- 结果:A、B、C 各占 33.33% -->
```

### 2. flex: auto

```css
flex: auto;

/* 完整写法 */
flex-grow: 1;
flex-shrink: 1;
flex-basis: auto;
```

**含义:**
- 可以放大(grow: 1)
- 可以缩小(shrink: 1)
- 初始大小根据内容决定(basis: auto)

**实际效果:** 先根据内容分配大小,然后平分剩余空间

```html
<div class="container">
  <div class="item">A</div>           <!-- flex: auto,内容少 -->
  <div class="item">BBBBBBBBB</div>   <!-- flex: auto,内容多 -->
</div>

<!-- 结果:B 比 A 宽,因为 B 的内容多 -->
```

### 3. flex: none

```css
flex: none;

/* 完整写法 */
flex-grow: 0;
flex-shrink: 0;
flex-basis: auto;
```

**含义:**
- 不放大(grow: 0)
- 不缩小(shrink: 0)
- 根据内容或 width 决定大小

**实际效果:** 固定大小,不参与弹性布局

### 4. flex: 0

```css
flex: 0;

/* 完整写法 */
flex-grow: 0;
flex-shrink: 1;
flex-basis: 0%;
```

**含义:**
- 不放大
- 可以缩小
- 初始大小为 0

---

## 四、flex: 1 vs flex: auto 的区别(重点!)

这是面试最爱考的!

### 对比表格

| 属性 | flex: 1 | flex: auto |
|------|---------|------------|
| flex-grow | 1 | 1 |
| flex-shrink | 1 | 1 |
| flex-basis | 0% | auto |
| 初始大小 | 0,忽略内容 | 根据内容 |
| 分配方式 | 完全平分 | 内容+剩余空间 |

### 实际案例对比

```html
<style>
.container {
  display: flex;
  width: 600px;
}

.item1 { flex: 1; }      /* flex-basis: 0% */
.item2 { flex: auto; }   /* flex-basis: auto */
</style>

<div class="container">
  <div class="item1">短</div>
  <div class="item2">这是很长很长很长的内容</div>
</div>
```

**flex: 1 的计算:**
```
item1 初始宽度 = 0
item2 初始宽度 = 0
剩余空间 = 600 - 0 = 600px
item1 分到 = 600 × (1/2) = 300px
item2 分到 = 600 × (1/2) = 300px

结果: 两个都是 300px (完全平分)
```

**flex: auto 的计算:**
```
item1 初始宽度 = 内容宽度 = 30px (假设)
item2 初始宽度 = 内容宽度 = 200px (假设)
剩余空间 = 600 - 230 = 370px
item1 分到 = 30 + 370 × (1/2) = 215px
item2 分到 = 200 + 370 × (1/2) = 385px

结果: item2 更宽 (因为内容多)
```

---

## 五、实战场景

### 场景1: 三栏布局(左右固定,中间自适应)

```css
.container {
  display: flex;
}

.left {
  width: 200px;
  flex-shrink: 0;  /* 不缩小 */
}

.center {
  flex: 1;  /* 占据剩余空间 */
}

.right {
  width: 200px;
  flex-shrink: 0;  /* 不缩小 */
}
```

### 场景2: 等高布局

```css
.container {
  display: flex;
}

.item {
  flex: 1;  /* 宽度平分 */
  /* 高度自动相等(flex 容器特性) */
}
```

### 场景3: 表单布局

```css
.form-row {
  display: flex;
  gap: 10px;
}

.label {
  width: 100px;
  flex-shrink: 0;  /* 标签固定宽度 */
}

.input {
  flex: 1;  /* 输入框占据剩余空间 */
}
```

---

## 六、面试常见问题

### Q1: flex: 1 1 0% 和 flex: 1 1 auto 有什么区别?

**答案:**
- `flex: 1 1 0%` (即 `flex: 1`): 初始大小为 0,完全平分空间
- `flex: 1 1 auto` (即 `flex: auto`): 初始大小根据内容,然后平分剩余空间

### Q2: 为什么 flex: 1 可以让子项平分空间?

**答案:**
因为 `flex-basis: 0%` 让所有子项的初始大小都是 0,然后 `flex-grow: 1` 让它们平分所有空间。

### Q3: flex: 1 和 width: 100% 有什么区别?

**答案:**
- `width: 100%`: 固定占据父容器 100% 宽度,不会自动调整
- `flex: 1`: 弹性占据剩余空间,会根据兄弟元素自动调整

### Q4: 多个子项设置不同的 flex 值会怎样?

```css
.item1 { flex: 1; }  /* 占 1 份 */
.item2 { flex: 2; }  /* 占 2 份 */
.item3 { flex: 3; }  /* 占 3 份 */

/* 结果: 按 1:2:3 的比例分配空间 */
```

---

## 七、常见坑点

### 坑1: flex: 1 不生效

```css
/* ❌ 错误:父容器没有设置 display: flex */
.container {
  /* 忘记写 display: flex */
}

.item {
  flex: 1;  /* 不生效! */
}

/* ✅ 正确 */
.container {
  display: flex;
}

.item {
  flex: 1;
}
```

### 坑2: flex: 1 被内容撑开

```css
/* 问题:子项内容太长,撑破了布局 */
.item {
  flex: 1;
}

/* 解决方案:限制最小宽度 */
.item {
  flex: 1;
  min-width: 0;  /* 允许缩小到 0 */
  overflow: hidden;  /* 或者隐藏溢出 */
}
```

### 坑3: flex-basis 和 width 冲突

```css
.item {
  flex: 1;  /* flex-basis: 0% */
  width: 200px;  /* 会被 flex-basis 覆盖 */
}

/* 如果想设置初始宽度,应该用 flex-basis */
.item {
  flex: 1 1 200px;  /* 初始 200px,然后弹性伸缩 */
}
```

---

## 八、记忆口诀

**"一长一短一基础"**
- flex-grow: 有剩余空间时,我能长多少
- flex-shrink: 空间不足时,我能缩多少
- flex-basis: 开始分配前,我的基础大小

**"flex: 1 平分天下,flex: auto 内容为王"**

---

## 九、完整对比表

| 简写 | flex-grow | flex-shrink | flex-basis | 使用场景 |
|------|-----------|-------------|------------|----------|
| flex: 1 | 1 | 1 | 0% | 平分空间 |
| flex: auto | 1 | 1 | auto | 内容+剩余 |
| flex: none | 0 | 0 | auto | 固定大小 |
| flex: 0 | 0 | 1 | 0% | 不占空间 |
| flex: 2 | 2 | 1 | 0% | 占 2 份 |

---

## 十、动手练习

### 练习1: 实现三栏布局
左边 200px,右边 200px,中间自适应

### 练习2: 实现表单布局
标签 100px,输入框占据剩余空间

### 练习3: 实现卡片列表
3 个卡片平分容器宽度,但内容多的卡片稍微宽一点

需要我提供练习的完整代码吗?
