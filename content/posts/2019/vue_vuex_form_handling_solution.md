---
id: "2a716f532886cf7371d9f9f776adcc5a7bff312e"
title: "使用 vuex-map-fields 讓 Vuex state 也可使用 v-model 綁定"
description: "有在用 Vue 寫應用程式的人，一定也會接觸到 Vuex，它有很多優點但也有限制，其中一項就是不能在 Mutation 以外的地方修改 State ，所以也就不能直接使用 v-model 去綁定，雖然官方有提供一段替代的寫法，但還是比原本直接用 v-model 來的麻煩很多，特別是大量的時候。"
date: 2019-05-16T21:17:40+08:00
draft: false
images:
    - "/images/og_image.png"
categories:
    - 軟體開發
    
tags:
    - Front-End
    - Vue
    - Vuex
    - Web
    - vuex-map-fields
    - v-model
aliases:
    - /post/2019/vue_vuex_form_handling_solution/  
---

有在用 Vue 寫應用程式的人，一定也會接觸到 Vuex，它有很多優點但也有限制，
其中一項就是不能在 Mutation 以外的地方修改 State ，所以也就不能直接使用 
v-model 去綁定，雖然官方有提供一段替代的寫法，但還是比原本直接用 v-model 
來的麻煩很多，特別是大量的時候。

<!--more-->

下面這段是官方提供的替代寫法，相信有用 Vuex 的人遇到這個問題去搜尋時都會看到這段 Code。

這是一般開發者一定都會想到的一個解法，很符合 Vuex 的精神。

```html
<input :value="message" @input="updateMessage">
```

```js
// ...
computed: {
  ...mapState({
    message: state => state.obj.message
  })
},
methods: {
  updateMessage (e) {
    this.$store.commit('updateMessage', e.target.value)
  }
}
```

還有更簡單的寫法嗎？ 

官方文件最後又提供了另個可以直接用 v-model 的解法。

可以看到，他它使用 Compute 的 Getter 與 Setter 去分開處理 v-model 的讀取與賦予，
比上一段更簡潔，也不破壞 Vuex 的精神。

```html
<input v-model="message">
```

```js
// ...
computed: {
  message: {
    get () {
      return this.$store.state.obj.message
    },
    set (value) {
      this.$store.commit('updateMessage', value)
    }
  }
}
```

雖然它很簡潔了，但要綁定的量一多，一個個寫 Getter 與 Setter 也是很煩，且他們都是類似行為，應該能抽出來共用？

經過一番搜尋，找到了這個套件 [vuex-map-fields](https://github.com/maoberlehner/vuex-map-fields) ，
這是作者將官方那個方法抽成共用方法並包成套件，只要輸入欄位就能使用直接用相同名稱進行綁定。

該套件的範例 Code 如下，可以看到這比官方提供的解法更為簡潔。

```js
<template>
  <div id="app">
    <input v-model="fieldA">
    <input v-model="fieldB">
  </div>
</template>

<script>
import { mapFields } from 'vuex-map-fields';

export default {
  computed: {
    // The `mapFields` function takes an array of
    // field names and generates corresponding
    // computed properties with getter and setter
    // functions for accessing the Vuex store.
    ...mapFields([
      'fieldA',
      'fieldB',
    ]),
  },
};
</script>
```

甚至是多層的 State 他也能處理，詳細就自己去看 Readme 囉！使用上頗簡單的。

這是到目前為止最佳的解法，也是我最能接受的解法！
