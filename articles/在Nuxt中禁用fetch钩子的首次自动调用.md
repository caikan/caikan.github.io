# 在Nuxt中禁用fetch钩子的首次自动调用

Nuxt 2.12增加了一个新的`fetch`钩子。

如果定义了此钩子，组件实例上会出现一个`$fetchState`属性，可以用来方便地获取当前fetch的状态。
可以通过访问实例上的`$fetch()`方法主动调用此钩子，配合`fetchDelay`选项还能够方便地实现自定义防抖效果。

以上这些特点让我感觉这个新`fetch`钩子挺好用的，能够减少大量判断请求状态的样板代码。但它的另外一个特点却让其使用场景受到了限制——此钩子会在所属组件创建后不久被自动调用一次。
从名称上看，`fetch`的含义是“取”，意味着组件的渲染需要先获取数据，所以原设计者设计了自动调用这个特性。

但有时候我只是想利用此钩子来方便地处理请求，并不希望它被自动调用，而是通过`$fetch`主动调用。
比如某页面上有一个表单需要提交，我需要在点击提交按钮后再发出请求，而不是进入页面就立即发出。

我们可以通过一个自定义Nuxt插件来实现这个效果。
```ts
import Vue from 'vue'

declare module 'vue/types/options' {
  // eslint-disable-next-line no-unused-vars,@typescript-eslint/no-unused-vars
  interface ComponentOptions<V extends Vue> {
    fetch2?(): Promise<void> | void
  }
}

const weakSet = new WeakSet()
Vue.mixin({
  beforeCreate() {
    const fetch2 = this.$options.fetch2
    if (fetch2) {
      this.$options.fetch = async function () {
        if (weakSet.has(this)) {
          return await fetch2.call(this)
        }
        weakSet.add(this)
      }
    }
  },
})
```

用法：在组件选项上定义一个`fetch2`钩子，此钩子除了不会被自动调用，其它特性和相关用法都与`fetch`钩子一致。