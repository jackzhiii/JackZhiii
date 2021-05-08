# WebAssembly 是什么

WebAssembly 是一种新型的代码, 可以被运行在现代主流的浏览器上 - webAssembly 是一种低级的，类似汇编的，紧凑的二进制格式的语言。同时，WebAssembly 运行几乎能够达到原生的性能, 并且可以被其他语言，例如 C/C++, Rust 作为编译目标平台，也因此可以让这些语言能够在 web 平台运行。WebAssembly 被设计用来与 JavasScript 一同协作，也就意味这 JavaScript 可以调用 WebAssembly 的功能。

# 简而言之
WebAssembly 对于 web 平台有着巨大的影响 - 它提供了一种运行代码的方式，这个代码可以被多种语言编写，例如 C/C++, Rust, 同时能够在运行在 web 的同时，能够拥有接近原生的速度。这是之前运行在 web 的客户端应用程序所不能做到的。

WebAssembly 旨在补充(提供接近原生的性能) JavaScript 并与之一起运行 - 使用 WebAssembly 的 JavaScript 的 APIS， 你可以在 JavaScript 应用程序中共享 WebAssembly 模块，同时在两者之间共享功能。这允许你能够在同一个 web 应用程序中既能利用 WebAssembly 的性能和强大，也能够利用 JavaScript 的开发经验和灵活性，甚至在你根本不需要知道 WebAssembly 的代码。

更好的是，在 **W3C** 和 **Community Group**  以及主流浏览器厂商的参与下，WebAssembly 正在发展为一种 web 标准。

# 指南
### WebAssembly 概念
通过阅读 WebAssembly 背后的概念来开始学习 - 它是什么，为什么它如此有用，它如何适用于 web 平台及其之后发展，并且如何使用

### 将新的 C/C++ 模块编译为 WebAssembly
当你使用 C/C++编写代码，你可以使用 [Emscripten](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/Emscripten/) 将代码编译为后缀为 .wasm 的 WebAssembly 代码。让我们看看它是如何工作的

### 将已经存在的 C 模块编译为 WebAssembly
对于 WebAssembly, 一个核心的使用场景是利用已存在的 C 库，从而让开发者能够在 web 平台上使用这些已存在的 C 库。

### 将 Rust 代码编译为 WebAssembly
如果你写了 Rust 代码，你可以将其编译为 WebAssembly 代码。本教程将会教你如何将一个 Rust 项目编译为 WebAssembly, 并且如何在一个已经存在的 web 应用上使用它。

### 加载和运行 WebAssembly 代码
当你有了 .wasm 文件，这篇文章将会教你如何获取，编译和实例化它，结合 [WebAssembly Javascript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WebAssembly) API 使用 Fetch 和 XHR APIs。

### 使用 WebAssembly Javascript API
一旦你已经加载了一个 .wasm 模块，你会想要使用它。这篇文章将你向你展示如何利用 WebAssembly Javasript API 来使用 WebAssembly。

### 导出 WebAssembly 函数
导出 WebAssembly 函数是 WebAssembly 函数的 JavaScript 的反射，它们允许从 JavaScript 调用 WebAssembly 代码。这边文章将会为你讲解导出函数是什么。

### 理解 WebAssembly 文本格式
这边文章将会解释 wasm 文本格式。为浏览器开发人员调试的时候展示的低级别 .wasm 模块。

### 转化 WebAssembly 文本格式为 wasm
这篇文章将会讲解如何将 WebAssembly 文本格式的代码转化为一个 .wasm 二进制文件。

# API 参考
### WebAssembly
这个对象充当所有 WebAssembly 相关功能的命名空间

# 参考
[原文地址](https://developer.mozilla.org/en-US/docs/WebAssembly)