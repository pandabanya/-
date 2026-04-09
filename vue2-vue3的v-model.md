# Vue2 vs Vue3 中的 v-model 详解

## 一、v-model 的本质

**核心概念**: v-model 是一个语法糖,本质上是 **属性绑定 + 事件监听** 的简写。

```vue
<!-- 这两种写法完全等价 -->
<input v-model="message">
<input :value="message" @input="message = $event.target.value">
```

**通俗理解**:
- `:value` 负责把数据显示到输入框(单向数据流:数据 → 视图)
- `@input` 负责监听用户输入,更新数据(事件流:视图 → 数据)
- 两者结合就实现了"双向绑定"

---

## 二、Vue2 中的 v-model

### 1. 原生表单元素

```vue
<template>
  <!-- 文本输入框 -->
  <input v-model="username" type="text">
  <!-- 等价于 -->
  <input :value="username" @input="username = $event.target.value">

  <!-- 复选框 -->
  <input v-model="checked" type="checkbox">
  <!-- 等价于 -->
  <input :checked="checked" @change="checked = $event.target.checked">

  <!-- 单选框 -->
  <input v-model="picked" type="radio" value="A">
  <!-- 等价于 -->
  <input :checked="picked === 'A'" @change="picked = $event.target.value">

  <!-- 下拉框 -->
  <select v-model="selected">
    <option value="A">选项A</option>
  </select>
  <!-- 等价于 -->
  <select :value="selected" @change="selected = $event.target.value">
</template>

<script>
export default {
  data() {
    return {
      username: '',
      checked: false,
      picked: '',
      selected: ''
    }
  }
}
</script>
```

### 2. 自定义组件中的 v-model (Vue2)

**规则**: 
- 默认绑定 `value` 属性
- 默认监听 `input` 事件

```vue
<!-- 父组件 -->
<template>
  <CustomInput v-model="searchText" />
  <!-- 等价于 -->
  <CustomInput :value="searchText" @input="searchText = $event" />
</template>

<script>
export default {
  data() {
    return {
      searchText: ''
    }
  }
}
</script>
```

```vue
<!-- 子组件: CustomInput.vue -->
<template>
  <input 
    :value="value" 
    @input="$emit('input', $event.target.value)"
  >
</template>

<script>
export default {
  props: {
    value: String // 必须接收 value 属性
  }
}
</script>
```

**关键点**:
1. 子组件必须声明 `value` prop
2. 子组件通过 `$emit('input', newValue)` 触发更新
3. 父组件的数据会自动更新

### 3. Vue2 的 model 选项(自定义prop和事件)

```vue
<!-- 父组件 -->
<template>
  <CustomCheckbox v-model="isChecked" />
  <!-- 实际编译为 -->
  <CustomCheckbox :checked="isChecked" @change="isChecked = $event" />
</template>
```

```vue
<!-- 子组件: CustomCheckbox.vue -->
<template>
  <input 
    type="checkbox"
    :checked="checked"
    @change="$emit('change', $event.target.checked)"
  >
</template>

<script>
export default {
  model: {
    prop: 'checked',  // 自定义prop名称
    event: 'change'   // 自定义事件名称
  },
  props: {
    checked: Boolean
  }
}
</script>
```

**为什么需要 model 选项?**
- 因为 checkbox 原生用的是 `checked` 属性,不是 `value`
- 原生触发的是 `change` 事件,不是 `input` 事件
- model 选项让我们可以自定义绑定的属性和事件

### 4. Vue2 的 .sync 修饰符(多个双向绑定)

**问题**: Vue2 的 v-model 只能绑定一个属性

```vue
<!-- 父组件 -->
<template>
  <Dialog 
    :visible="dialogVisible" 
    @update:visible="dialogVisible = $event"
    :title="dialogTitle"
    @update:title="dialogTitle = $event"
  />
  
  <!-- 使用 .sync 简化 -->
  <Dialog 
    :visible.sync="dialogVisible"
    :title.sync="dialogTitle"
  />
</template>
```

```vue
<!-- 子组件: Dialog.vue -->
<template>
  <div v-if="visible">
    <input :value="title" @input="$emit('update:title', $event.target.value)">
    <button @click="$emit('update:visible', false)">关闭</button>
  </div>
</template>

<script>
export default {
  props: {
    visible: Boolean,
    title: String
  }
}
</script>
```

---

## 三、Vue3 中的 v-model

### 1. 破坏性变更

| 特性 | Vue2 | Vue3 |
|------|------|------|
| 默认prop名 | `value` | `modelValue` |
| 默认事件名 | `input` | `update:modelValue` |
| model选项 | 支持 | 移除 |
| .sync修饰符 | 支持 | 移除(被v-model替代) |
| 多个v-model | 不支持 | 支持 |

### 2. Vue3 自定义组件中的 v-model

```vue
<!-- 父组件 -->
<template>
  <CustomInput v-model="searchText" />
  <!-- 等价于 -->
  <CustomInput 
    :modelValue="searchText" 
    @update:modelValue="searchText = $event" 
  />
</template>

<script setup>
import { ref } from 'vue'
const searchText = ref('')
</script>
```

```vue
<!-- 子组件: CustomInput.vue -->
<template>
  <input 
    :value="modelValue" 
    @input="$emit('update:modelValue', $event.target.value)"
  >
</template>

<script setup>
/**
 * 定义props
 */
defineProps({
  modelValue: String
})

/**
 * 定义emits
 */
defineEmits(['update:modelValue'])
</script>
```

### 3. Vue3 的多个 v-model(重要特性)

**这是Vue3最大的改进**: 可以在同一个组件上使用多个 v-model

```vue
<!-- 父组件 -->
<template>
  <UserForm 
    v-model:username="user.name"
    v-model:email="user.email"
    v-model:age="user.age"
  />
  
  <!-- 等价于 -->
  <UserForm
    :username="user.name"
    @update:username="user.name = $event"
    :email="user.email"
    @update:email="user.email = $event"
    :age="user.age"
    @update:age="user.age = $event"
  />
</template>

<script setup>
import { reactive } from 'vue'

const user = reactive({
  name: '',
  email: '',
  age: 0
})
</script>
```

```vue
<!-- 子组件: UserForm.vue -->
<template>
  <div>
    <input 
      :value="username" 
      @input="$emit('update:username', $event.target.value)"
      placeholder="用户名"
    >
    <input 
      :value="email" 
      @input="$emit('update:email', $event.target.value)"
      placeholder="邮箱"
    >
    <input 
      type="number"
      :value="age" 
      @input="$emit('update:age', Number($event.target.value))"
      placeholder="年龄"
    >
  </div>
</template>

<script setup>
/**
 * 定义多个props
 */
defineProps({
  username: String,
  email: String,
  age: Number
})

/**
 * 定义多个emits
 */
defineEmits(['update:username', 'update:email', 'update:age'])
</script>
```

### 4. Vue3 v-model 修饰符

**内置修饰符**:
```vue
<template>
  <!-- .lazy: 在change事件后同步,而不是input -->
  <input v-model.lazy="message">
  
  <!-- .number: 自动转换为数字 -->
  <input v-model.number="age" type="number">
  
  <!-- .trim: 自动去除首尾空格 -->
  <input v-model.trim="username">
</template>
```

**自定义修饰符**:
```vue
<!-- 父组件 -->
<template>
  <CustomInput v-model.capitalize="text" />
</template>
```

```vue
<!-- 子组件: CustomInput.vue -->
<template>
  <input 
    :value="modelValue"
    @input="handleInput"
  >
</template>

<script setup>
/**
 * 定义props,包含修饰符
 */
const props = defineProps({
  modelValue: String,
  modelModifiers: {
    default: () => ({})
  }
})

/**
 * 定义emits
 */
const emit = defineEmits(['update:modelValue'])

/**
 * 处理输入,应用修饰符
 */
function handleInput(e) {
  let value = e.target.value
  
  // 如果有 capitalize 修饰符,首字母大写
  if (props.modelModifiers.capitalize) {
    value = value.charAt(0).toUpperCase() + value.slice(1)
  }
  
  emit('update:modelValue', value)
}
</script>
```

---

## 四、实战案例对比

### 案例1: 对话框组件

**Vue2 实现**:
```vue
<!-- 父组件 -->
<template>
  <Dialog 
    v-model="visible" 
    :title.sync="dialogTitle"
  />
</template>

<script>
export default {
  data() {
    return {
      visible: false,
      dialogTitle: '提示'
    }
  }
}
</script>
```

```vue
<!-- Dialog.vue (Vue2) -->
<template>
  <div v-if="value" class="dialog">
    <input :value="title" @input="$emit('update:title', $event.target.value)">
    <button @click="$emit('input', false)">关闭</button>
  </div>
</template>

<script>
export default {
  props: {
    value: Boolean,  // v-model 绑定
    title: String    // .sync 绑定
  }
}
</script>
```

**Vue3 实现**:
```vue
<!-- 父组件 -->
<template>
  <Dialog 
    v-model:visible="visible"
    v-model:title="dialogTitle"
  />
</template>

<script setup>
import { ref } from 'vue'

const visible = ref(false)
const dialogTitle = ref('提示')
</script>
```

```vue
<!-- Dialog.vue (Vue3) -->
<template>
  <div v-if="visible" class="dialog">
    <input 
      :value="title" 
      @input="$emit('update:title', $event.target.value)"
    >
    <button @click="$emit('update:visible', false)">关闭</button>
  </div>
</template>

<script setup>
/**
 * 定义props
 */
defineProps({
  visible: Boolean,
  title: String
})

/**
 * 定义emits
 */
defineEmits(['update:visible', 'update:title'])
</script>
```

### 案例2: 表单组件

**Vue3 完整示例**:
```vue
<!-- 父组件: App.vue -->
<template>
  <div>
    <h2>用户信息</h2>
    <UserForm 
      v-model:name="form.name"
      v-model:email="form.email"
      v-model:gender="form.gender"
    />
    
    <pre>{{ form }}</pre>
  </div>
</template>

<script setup>
import { reactive } from 'vue'
import UserForm from './UserForm.vue'

const form = reactive({
  name: '',
  email: '',
  gender: ''
})
</script>
```

```vue
<!-- 子组件: UserForm.vue -->
<template>
  <div class="form">
    <div class="form-item">
      <label>姓名:</label>
      <input 
        :value="name"
        @input="updateName"
        placeholder="请输入姓名"
      >
    </div>
    
    <div class="form-item">
      <label>邮箱:</label>
      <input 
        :value="email"
        @input="updateEmail"
        type="email"
        placeholder="请输入邮箱"
      >
    </div>
    
    <div class="form-item">
      <label>性别:</label>
      <label>
        <input 
          type="radio" 
          :checked="gender === 'male'"
          @change="updateGender('male')"
        > 男
      </label>
      <label>
        <input 
          type="radio" 
          :checked="gender === 'female'"
          @change="updateGender('female')"
        > 女
      </label>
    </div>
  </div>
</template>

<script setup>
/**
 * 定义props
 */
defineProps({
  name: String,
  email: String,
  gender: String
})

/**
 * 定义emits
 */
const emit = defineEmits(['update:name', 'update:email', 'update:gender'])

/**
 * 更新姓名
 */
function updateName(e) {
  emit('update:name', e.target.value)
}

/**
 * 更新邮箱
 */
function updateEmail(e) {
  emit('update:email', e.target.value)
}

/**
 * 更新性别
 */
function updateGender(value) {
  emit('update:gender', value)
}
</script>

<style scoped>
.form-item {
  margin-bottom: 15px;
}

.form-item label {
  display: inline-block;
  width: 80px;
}
</style>
```

---

## 五、常见错误和注意事项

### 错误1: Vue3中还在用 value prop

```vue
<!-- ❌ 错误:Vue3中不再是value -->
<CustomInput :value="text" @input="text = $event" />

<!-- ✅ 正确:Vue3中是modelValue -->
<CustomInput :modelValue="text" @update:modelValue="text = $event" />

<!-- ✅ 或者直接用v-model -->
<CustomInput v-model="text" />
```

### 错误2: 忘记emit事件

```vue
<!-- 子组件 -->
<template>
  <!-- ❌ 错误:直接修改prop -->
  <input :value="modelValue" @input="modelValue = $event.target.value">
  
  <!-- ✅ 正确:通过emit通知父组件 -->
  <input :value="modelValue" @input="$emit('update:modelValue', $event.target.value)">
</template>
```

### 错误3: Vue3中使用.sync

```vue
<!-- ❌ 错误:Vue3移除了.sync -->
<Dialog :visible.sync="show" />

<!-- ✅ 正确:用v-model替代 -->
<Dialog v-model:visible="show" />
```

---

## 六、记忆口诀

**Vue2**: "value + input,单个绑定,.sync多个"
**Vue3**: "modelValue + update,多个v-model,统一天下"

---

## 七、动手练习任务

### 任务1: 实现一个计数器组件
要求:
- 父组件用 v-model 绑定数字
- 子组件有 +1 和 -1 按钮
- 分别用 Vue2 和 Vue3 实现

### 任务2: 实现一个搜索组件
要求:
- 父组件用 v-model 绑定搜索关键词
- 子组件有输入框和清空按钮
- 添加 .trim 修饰符

### 任务3: 实现一个复杂表单
要求:
- 包含姓名、年龄、爱好(多选)
- 用 Vue3 的多个 v-model 实现
- 父组件实时显示表单数据

需要我提供这些练习的完整代码吗?
