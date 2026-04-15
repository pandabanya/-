# 理解 v-model 的关键 - 一句话说清楚

## 核心问题:为什么有时需要 props,有时不需要?

### 一句话答案:
**看数据归谁管!**

---

## 两种情况对比

### 情况1: 数据自己管 (不需要 props)

```vue
<!-- 组件内部 -->
<script setup>
import { ref } from 'vue'

// 自己的数据,自己管
const searchText = ref('')
</script>

<template>
  <input v-model="searchText">
</template>
```

**特点:**
- ✅ 数据只在这个组件用
- ✅ 父组件不需要知道这个值
- ✅ 不需要传给父组件
- ✅ 简单直接

**适用场景:**
- 搜索框(搜索结果在当前组件显示)
- 评论输入框(提交后清空,不需要父组件知道)
- 临时的筛选条件

---

### 情况2: 数据父组件管 (需要 props + v-model)

```vue
<!-- 父组件 -->
<script setup>
import { ref } from 'vue'
import CustomInput from './CustomInput.vue'

// 父组件的数据
const username = ref('')
</script>

<template>
  <!-- 通过 v-model 把数据传给子组件 -->
  <CustomInput v-model="username" />
  
  <!-- 父组件可以使用这个数据 -->
  <p>用户名: {{ username }}</p>
  <button @click="submit">提交</button>
</template>
```

```vue
<!-- 子组件: CustomInput.vue -->
<script setup>
// 接收父组件传来的数据
defineProps({
  modelValue: String
})

// 定义事件,用于通知父组件
defineEmits(['update:modelValue'])
</script>

<template>
  <input 
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
  >
</template>
```

**特点:**
- ✅ 数据由父组件管理
- ✅ 父组件可以读取和修改
- ✅ 可以在多个地方复用
- ✅ 适合表单数据收集

**适用场景:**
- 封装的输入框组件(要在多处使用)
- 表单组件(父组件需要收集所有数据统一提交)
- 对话框(父组件控制显示/隐藏)

---

## 数据流向对比

### 情况1: 内部数据流
```
组件内部:
用户输入 → 更新 searchText → 显示在输入框
         ↓
      (结束,不出去)
```

### 情况2: 父子数据流
```
父组件:
username = 'tom'
    ↓ (通过 :modelValue 传下去)
子组件:
接收 modelValue = 'tom' → 显示在输入框
    ↓ (用户输入)
触发 @input 事件
    ↓ (通过 $emit 传上去)
父组件:
username 更新为新值
```

---

## 实际判断方法

问自己三个问题:

### 1. 父组件需要知道这个值吗?
- ❌ 不需要 → 用内部状态
- ✅ 需要 → 用 props + v-model

### 2. 这个组件会在多个地方用吗?
- ❌ 不会 → 用内部状态
- ✅ 会 → 用 props + v-model

### 3. 这个值需要传给其他组件或提交到服务器吗?
- ❌ 不需要 → 用内部状态
- ✅ 需要 → 用 props + v-model

---

## 实战案例对比

### 案例A: 页面内的搜索框

```vue
<!-- ❌ 不需要 props -->
<script setup>
import { ref } from 'vue'

const keyword = ref('')
const results = ref([])

function search() {
  // 搜索逻辑
  results.value = mockSearch(keyword.value)
}
</script>

<template>
  <div>
    <input v-model="keyword">
    <button @click="search">搜索</button>
    <div v-for="item in results">{{ item }}</div>
  </div>
</template>
```

**为什么不需要 props?**
- 搜索关键词只在当前组件用
- 搜索结果也在当前组件显示
- 父组件不需要知道搜索了什么

---

### 案例B: 用户注册表单

```vue
<!-- 父组件: RegisterPage.vue -->
<script setup>
import { reactive } from 'vue'
import FormInput from './FormInput.vue'

const form = reactive({
  username: '',
  email: '',
  password: ''
})

function register() {
  // 提交表单到服务器
  api.register(form)
}
</script>

<template>
  <form @submit.prevent="register">
    <FormInput v-model="form.username" label="用户名" />
    <FormInput v-model="form.email" label="邮箱" />
    <FormInput v-model="form.password" label="密码" type="password" />
    <button type="submit">注册</button>
  </form>
</template>
```

```vue
<!-- 子组件: FormInput.vue -->
<script setup>
defineProps({
  modelValue: String,
  label: String,
  type: { type: String, default: 'text' }
})

defineEmits(['update:modelValue'])
</script>

<template>
  <div class="form-item">
    <label>{{ label }}</label>
    <input 
      :type="type"
      :value="modelValue"
      @input="$emit('update:modelValue', $event.target.value)"
    >
  </div>
</template>
```

**为什么需要 props?**
- 父组件需要收集所有表单数据
- 表单数据要统一提交到服务器
- FormInput 组件要在多个地方复用

---

## Vue3 的 v-model:xxx 语法

### 为什么要用 v-model:xxx?

**问题**: 一个组件有多个值需要双向绑定怎么办?

**Vue2 的做法**: 只能用一个 v-model,其他用 .sync
```vue
<!-- Vue2 -->
<Dialog v-model="visible" :title.sync="title" />
```

**Vue3 的做法**: 可以用多个 v-model
```vue
<!-- Vue3 -->
<Dialog v-model:visible="visible" v-model:title="title" />
```

### v-model:xxx 的本质

```vue
<!-- 这个写法 -->
<Dialog v-model:visible="show" v-model:title="dialogTitle" />

<!-- 等价于 -->
<Dialog 
  :visible="show"
  @update:visible="show = $event"
  :title="dialogTitle"
  @update:title="dialogTitle = $event"
/>
```

### 子组件怎么写?

```vue
<!-- Dialog.vue -->
<script setup>
defineProps({
  visible: Boolean,  // 对应 v-model:visible
  title: String      // 对应 v-model:title
})

defineEmits([
  'update:visible',  // 对应 v-model:visible
  'update:title'     // 对应 v-model:title
])
</script>

<template>
  <div v-if="visible">
    <input 
      :value="title"
      @input="$emit('update:title', $event.target.value)"
    >
    <button @click="$emit('update:visible', false)">关闭</button>
  </div>
</template>
```

---

## 记忆口诀

**"自己用,自己管;别人用,props传"**

- 自己用 = 数据只在当前组件使用 → 用 ref/reactive
- 别人用 = 父组件需要这个数据 → 用 props + v-model

---

## 最后的建议

### 开始写组件时,先问自己:

1. **这个组件是给谁用的?**
   - 只给当前页面用 → 可能不需要 props
   - 要在多个地方复用 → 需要 props

2. **数据最终去哪里?**
   - 只在组件内部用 → 不需要 props
   - 要传给父组件或提交到服务器 → 需要 props

3. **父组件需要控制这个值吗?**
   - 不需要 → 不需要 props
   - 需要 → 需要 props

### 实在不确定?

**默认规则**: 如果是封装的通用组件(比如输入框、对话框、选择器),就用 props + v-model,这样最灵活!
