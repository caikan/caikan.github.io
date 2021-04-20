# 一例Nuxt项目axios类型声明在VSCode中提示失效的问题

场景是一个基于Nuxt.js 2.15.3的项目，通过`npm init nuxt-app`初始化，开启了TypeScript，已经安装了`nuxt/axios`模块。
编辑器Windows VSCode 1.55.2，插件Vetur 0.33.1 (octref.vetur).
在下面的例子中，编辑器提示`ctx.$axios`无法识别，错误信息为`Property '$axios' does not exist on type 'Context'.Vetur(2339)`，实际运行时却可以正常访问。
```vue
<script lang="ts">
import Vue from 'vue'
export default Vue.extend({
  fetch(ctx) {
    // Property '$axios' does not exist on type 'Context'.Vetur(2339)
    ctx.$axios('/') 
  },
})
</script>
```

检查过`tsconfig.json`文件，看上去是没问题的（创建项目时，选择了axios模块后自动生成的），但看上去未能生效。
```json
{
  "compilerOptions": {
    "types": [
      "@nuxt/types",
      "@nuxtjs/axios",
      "@types/node"
    ]
  },
}
```

后来尝试发现，在报错的模块中引入对应的类型声明，`$axios`就可以被识别了。
```vue
<script lang="ts">
import Vue from 'vue'
import '@nuxt/types'
export default Vue.extend({
  fetch(ctx) {
    ctx.$axios('')
  },
})
</script>
```

但这种办法需要在每个模块中单独引入类型声明，太过繁琐。是否可以在某处全局代码中去引入呢？可是Nuxt项目并没有一个总入口模块，我突然想到Nuxt插件配置可以写一些全局代码，于是我考虑尝试着在Nuxt插件配置中去引入类型声明，发现居然可以全局生效。
```js
// @/nuxt.config.js
export default {
  plugins: ['@/plugins/axios'],
}
```
```js
// @/plugins/axios.js
import '@nuxt/types'
```

在尝试通过插件配置来全局引入类型声明的办法时，我发现如果是在VSCode一直开启的状态下，新增一条插件配置来引入类型声明的话，是需要重启VSCode才能生效的；但如果是向本次启动VSCode之前就已经存在的插件配置模块中增加类型声明，是可以保存后立即生效的。
同时我还发现，如果不引入类型声明，虽然VSCode会报错，但是运行构建时并没有相关的错误信息输出。
所以我推测这可能仅仅是Vetur插件或者VSCode的bug，而不是代码配置问题。

利用插件模块全局引入虽然能够解决，但为解决一个编辑器提示的bug去专门写这么多插件配置显得有些奇怪。我突然想到，虽然Nuxt项目没有全局入口，但是否可以用`nuxt.config.js`来替代呢？尝试了一下，居然可以。
```js
// @/nuxt.config.js
import '@nuxt/types'
export default {
  //...
}
```

我又想，虽然Nuxt项目没有全局入口，但是我是否可以把这个引入声明放在根目录的`index.js`？试了一下居然也可以。感觉这种办法最适合，因为不会污染真正有用的项目文件。还可以添加到`.gitignore`以避免提交。不过如果其它项目协作者也会遇到同样问题的话，也是可以考虑提交到版本库，替所有人一并解决的。
```js
// @/index.js
import '@nuxt/types'
```

最后还尝试了一些其它方法，在`tsconfig.json`中添加一条类型声明的项目，也可以解决
```ts
// @/index.d.ts
export * from '@nuxt/types'
```
```json
// @/tsconfig.json
{
  "compilerOptions": {
    "types": [
      "@nuxt/types",
      "@nuxtjs/axios",
      "@types/node",
      "index.d.ts",
    ]
  },
}
```