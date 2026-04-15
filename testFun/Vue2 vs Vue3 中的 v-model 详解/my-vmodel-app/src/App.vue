<script setup>
import { ref } from 'vue'

// 这里你可以自己写代码练习

const message = ref('')
</script>

<template>
  <div>
    <h1>Vue3 v-model 练习</h1>
    <!-- 这里你可以自己写代码练习 -->

    <input type="text" v-model="message">

    <p>{{ message }}</p>
  </div>
</template>

<style scoped>
/* 这里你可以自己写样式 */
</style>
