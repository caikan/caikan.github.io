# Nuxt中fetch的坑

根据官方文档，Nuxt从版本2.12开始，`fetch`钩子的用法发生了变化；在2.12版本之前，其用法类似于`asyncData`钩子。

那么Nuxt是如何区分我们写的`fetch`钩子是新用法还是旧用法呢——取决于`fetch`钩子的定义是否具有`context`参数。

一开始我没有注意这一点，向`fetch`钩子中传入了`context`参数，试图从中解构出`$axios`。

结果我调用组件实例上的`$fetch()`方法会失败，检测发现此方法并不存在，而且`$fetchState`也不存在，在`fetch`钩子内访问`this`得到的并不是组件实例，而是组件选项。

待我发现问题所在，去掉`fetch`的`context`参数后，才得到了预期的结果：组件实例上出现了`$fetch()`和`$fetchState`，`fetch`钩子内的`this`也得到组件实例了。

如果一开始没有碰到这个坑，我可能也不会遇到上一篇文章中写到的问题：
[一例Nuxt项目axios智能感知在VSCode中失效的问题](一例Nuxt项目axios智能感知在VSCode中失效的问题.md)