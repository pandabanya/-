## *SSR 优化*

### 1. 渲染模式选型（不是所有页面都要SSR）:
---
首先要分清场景：
1. SSR 适合强SEO、首屏依赖动态数据的页面（如商品详情，咨询文章等）
```javascript
// ========== SSR：实时渲染 ==========
// 适用：用户中心、实时数据、个性化推荐
'/user/**': { ssr: true },           // 用户信息必须实时
'/dashboard': { ssr: true },         // 仪表盘数据实时
'/search': { ssr: true },            // 搜索结果实时
```
2. SSG（静态生成）：适合不常变得内容 （关于页，帮助页、法律条款）
```javascript
// nuxt.config.js
'/about': { prerender: true },
```

3. ISR（增量静态再生，定期重新生成部分页面）
```javascript
'/news/**': { isr: 300 },  // 新闻5分钟更新
```

4. CSR 客户端渲染 适合后台管理、聊天室、游戏
```javascript
'/admin/**': { ssr: false },
```


### 2. 数据依赖的优化 （并行 + 缓存）
---
如果必须SSR, 数据请求要优化
1. 并行请求
用Promise.all 并行请求多个接口，不要串行等待

2. 服务端缓存
相同接口在服务端做缓存（Redis）,减少重复请求

3. 数据下沉： 不依赖用户的数据提前注入html,不走接口请求

什么是数据下沉：
简单理解就是所有用户看到的都一样的数据就可以下沉.
```plaintext
用户打开页面
    ↓
浏览器加载 HTML（空的）
    ↓
加载 JS
    ↓
JS 执行，调用 fetch('/api/config')  ← 多一次网络请求
    ↓
拿到数据：{ siteName: '我的网站', nav: [...] }
    ↓
渲染页面
问题：网站名称、导航菜单这些几乎不变的数据，每个用户每次访问都要请求一次接口，浪费。数据下沉的做法直接把这些数据写进 HTML，浏览器一打开就有，不用请求接口。
```

```javascript
// nuxt.config.ts
export default defineNuxtConfig({
  runtimeConfig: {
    public: {
      siteName: '我的网站',
      apiBase: 'https://api.example.com',
      copyright: 'Copyright 2024'
    }
  }
})

<script setup>
// 直接用，不用调接口
const config = useRuntimeConfig()
console.log(config.public.siteName)  // '我的网站'
</script>

<template>
  <footer>{{ config.public.copyright }}</footer>
</template>
```

4. 流式渲染 Streaming SSR
解决了传统SSR 的痛点， 慢数据阻塞了整个页面的加载，是的用户啥也看不见。
```plaintext
传统 SSR 流程：
服务端 → 等所有数据（3秒）→ 生成完整 HTML（1秒）→ 一次性发送给浏览器
         ↑ 用户白屏等待 4 秒 ↑

用户视角：白屏 4 秒，然后页面突然全部出现
问题：慢数据阻塞了整个页面，用户什么都看不到。
```
也就是说流式渲染不是等所有数据都准备好了在进行渲染，而是分阶段，分成关键和非关键部分，逐步填充页面，让用户进款看到页面内容。



### 3. 服务端组件 （减少客户端js）
合理设计API 

### 4. 性能监控与回退

首先我们先说几个概念
- LCP (页面最大可见元素（图片、标题、视频）渲染完成的时间)
```plaintext
用户打开页面
    ↓
0ms     开始加载
    ↓
800ms   导航栏出来了
    ↓
1.2s    标题出来了  ← LCP（最大内容）
    ↓
2.5s    图片出来了
    ↓
3.0s    全部加载完

标准
🟢 优秀：≤ 2.5 秒
🟡 需要改进：2.5～4 秒
🔴 差：> 4 秒
```

- TTFB （浏览器发送请求到收到第一个字节的时间）
```plaintext
用户点击链接
    ↓
0ms     发送请求
    ↓
100ms   DNS 解析
    ↓
200ms   建立连接
    ↓
500ms   服务端处理（查数据库、渲染 HTML）
    ↓
600ms   收到第一个字节  ← TTFB
```

- FID （用户第一次交互（点击、输入）到浏览器响应的时间）
```plaintext
用户看到页面
    ↓
点击按钮
    ↓
100ms   浏览器正在执行大量 JS（hydration）
    ↓
300ms   终于响应点击  ← FID = 200ms
```


上线后要持续看这几个指标，采用SSR失败降级，不要让用户一直白屏.
最后通过A/B测试验证效果，确保技术指标提升可以真正带来业务价值。
---
