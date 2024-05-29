<!--yml
category: 未分类
date: 2024-05-29 13:18:57
-->

# What is Tauri? | Tauri

> 来源：[https://beta.tauri.app/guides/](https://beta.tauri.app/guides/)

Tauri is a framework for building tiny, fast binaries for all major desktop and mobile platforms. Developers can integrate any frontend framework that compiles to HTML, JavaScript, and CSS for building their user experience while leveraging languages such as Rust, Swift, and Kotlin for backend logic when needed.

Get started building with [`create-tauri-app`](https://github.com/tauri-apps/create-tauri-app) by using one of the below commands. Be sure to follow the [prerequisites guide](/start/prerequisites/) to install all of the dependencies required by Tauri and then view the [Frontend Configuration guides](/start/frontend/) for recommended frontend configurations.

 <starlight-tabs class="astro-mkxqpaln">```
 sh  <(curl https://create.tauri.app/sh)  --beta 
```

```
 $env:CTA_ARGS="--beta"; irm https://create.tauri.app/ps | iex 
```

```
 npm  create  tauri-app@latest  --  --beta 
```

```
 yarn  create  tauri-app  --beta 
```

```
 pnpm  create  tauri-app  --beta 
```

```
 bun  create  tauri-app  --beta 
```

```
 cargo  install  create-tauri-app

cargo  create-tauri-app  --beta 
```</starlight-tabs> 

After you’ve created your first app you can explore the different features and recipes of Tauri in the [List of Features & Recipes](/plugin/).

Tauri has 3 main advantages for developers to build upon:

*   Secure foundation for building apps
*   Smaller bundle size by using the system’s native webview
*   Flexibility for developers to use any frontend and bindings for multiple languages

Learn more about the Tauri philosophy in the [Tauri 1.0 blog post](/blog/tauri-1-0/).

By being built on Rust, Tauri is able to take advantage of the memory, thread, and type-safety offered by Rust. Apps built on Tauri can automatically get those benefits even without needing to be developed by Rust experts.

Tauri also undergoes a security audit for major and minor releases. This not only covers code in the Tauri organization, but also for upstream dependencies that Tauri relies on. Of course this doesn’t mitigate all risks, but it provides a solid foundation for developers to build on top of.

Read the [Tauri security policy](https://github.com/tauri-apps/tauri/security/policy) and the [Tauri 1.0 audit report](https://github.com/tauri-apps/tauri/blob/dev/audits/Radically_Open_Security-v1-report.pdf).

Tauri apps take advantage of the web view already available on every user’s system. A Tauri app only contains the code and assets specific for that app and doesn’t need to bundle a browser engine with every app. This means that a minimal Tauri app can be less than 600KB in size.

Learn more about creating optimized apps in the [App Size concept](/concept/size/).

Since Tauri uses web technologies that means that virtually any frontend framework is compatible with Tauri. The [Frontend Configuration guide](/start/frontend/) contains common configurations for popular frontend frameworks and the [Rendering concept](/concept/rendering/) discusses which rendering techniques work best with Tauri (such as SPAs and SSGs).

Bindings between JavaScript and Rust are available to developers using the `invoke` function in JavaScript and Swift and Kotlin bindings are available for [Tauri Plugins](/develop/plugins/).

[TAO](https://github.com/tauri-apps/tao) is responsible for Tauri window creation and [WRY](https://github.com/tauri-apps/wry) is responsible for web view rendering. These are libraries maintained by Tauri and can be consumed directly if deeper system integration is required outside of what Tauri exposes.

In addition, Tauri maintains a number of plugins to extend what core Tauri exposes. You can find those plugins alongside those provided by the community in the [Plugins section](/plugin/).

© 2024 Tauri Contributors. CC-BY / MIT