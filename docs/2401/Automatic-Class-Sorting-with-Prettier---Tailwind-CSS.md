<!--yml

category: 未分类

date: 2024-05-27 14:54:28

-->

# Automatic Class Sorting with Prettier - Tailwind CSS

> 来源：[https://tailwindcss.com/blog/automatic-class-sorting-with-prettier](https://tailwindcss.com/blog/automatic-class-sorting-with-prettier)

人们已经讨论了至少四年关于在[Tailwind项目中对实用类进行排序的最佳方式](https://github.com/tailwindlabs/discuss/issues/97)。今天，我们很高兴地宣布，通过我们的官方[Prettier Tailwind CSS插件](https://github.com/tailwindlabs/prettier-plugin-tailwindcss)的发布，您终于可以不再担心这个问题了。

此插件扫描您的模板，查找包含Tailwind CSS类的类属性，然后自动按照我们的[推荐类顺序](/blog/automatic-class-sorting-with-prettier#how-classes-are-sorted)对这些类进行排序。

```
 <button class="text-white px-4 sm:px-8 py-2 sm:py-3 bg-sky-700 hover:bg-sky-800">...</button>

<button class="bg-sky-700 px-4 py-2 text-white hover:bg-sky-800 sm:px-8 sm:py-3">...</button>
```

它与自定义Tailwind配置无缝配合，并且因为它只是一个[Prettier](https://prettier.io/)插件，所以它可以在Prettier可用的任何地方工作——包括每个流行的编辑器和IDE，当然也包括命令行。

要开始使用，请将`prettier-plugin-tailwindcss`安装为dev依赖项：

```
npm install -D prettier prettier-plugin-tailwindcss
```

然后将插件添加到您的[Prettier配置文件](https://prettier.io/docs/en/configuration)中：

```
{
  "plugins": ["prettier-plugin-tailwindcss"]
}
```

您还可以通过使用Prettier CLI的`--plugin`标志或使用Prettier API的`plugins`选项来[加载插件](https://prettier.io/docs/en/plugins.html#using-plugins)。

* * *

本插件的核心功能是按照Tailwind在您的CSS中的顺序组织您的类。

这意味着基础层中的任何类将首先进行排序，然后是组件层中的类，最后是实用程序层中的类。

```
 <div class="container mx-auto px-6">

</div>
```

实用程序本身也按照我们在CSS中对它们进行排序的顺序进行排序，这意味着任何覆盖其他类的类始终出现在类列表的后面：

```
<div  class="pt-2 p-4">  <div  class="p-4 pt-2">  </div> 
```

不同实用工具的实际顺序基本上是基于盒模型，并尝试将影响布局的高影响类放在开头，装饰类放在末尾，同时还尝试保持相关的实用程序在一起：

```
<div  class="text-gray-700 shadow-md p-3 border-gray-300 ml-4 h-24 flex border-2">  <div  class="ml-4 flex h-24 border-2 border-gray-300 p-3 text-gray-700 shadow-md">  </div> 
```

`hover:`和`focus:`等修饰符被分组在一起，并在任何普通工具类之后进行排序：

```
<div  class="hover:opacity-75 opacity-50 hover:scale-150 scale-125">  <div  class="scale-125 opacity-50 hover:scale-150 hover:opacity-75">  </div> 
```

响应式修饰符（如`md:`和`lg:`）被分组在一起，按照主题中配置的顺序进行排序——默认情况下是从最小到最大：

```
<div  class="lg:grid-cols-4 grid sm:grid-cols-3 grid-cols-2">  <div  class="grid grid-cols-2 sm:grid-cols-3 lg:grid-cols-4">  </div> 
```

任何不来自Tailwind插件的自定义类（比如用于定位第三方库的类）始终排序到最前面，因此很容易看出元素是否正在使用它们：

```
<div  class="p-3 shadow-xl select2-dropdown">  <div  class="select2-dropdown p-3 shadow-xl">  </div> 
```

* * *

我们认为[Prettier的处理方式是正确的](https://prettier.io/docs/en/option-philosophy.html)，因为它具有明确的观点并且在定制性方面提供的内容很少——在一天结束时，对类进行排序的最大好处就是它少了一件事可以与团队争论。

我们非常努力地设计了一种易于理解并能尽快传达最重要信息的排序方式。

插件*会*尊重你的`tailwind.config.js`文件，并与你安装的任何 Tailwind 插件一起工作，但**没有办法改变排序顺序**。就像使用 Prettier 一样，我们认为自动格式化的好处很快就会超过你的任何风格偏好，并且你会很快适应它。

准备好试试吗？[在 GitHub 上查看完整文档 →](https://github.com/tailwindlabs/prettier-plugin-tailwindcss)
