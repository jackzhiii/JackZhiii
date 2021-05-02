# WebAssembly 是什么

WebAssembly 是一种新型的代码, 可以被运行在现代主流的浏览器上 - webAssembly 是一种低级的，类似汇编的，紧凑的二进制格式的语言。同时，WebAssembly 运行几乎能够达到原生的性能, 并且可以被其他语言，例如 C/C++, Rust 作为编译目标平台，也因此可以让这些语言能够在 web 平台运行。WebAssembly 被设计用来与 JavasScript 一同协作，也就意味这 JavaScript 可以调用 WebAssembly 的功能。

# 简而言之
WebAssembly 对于 web 平台有着巨大的影响 - 它提供了一种运行代码的方式，这个代码可以被多种语言编写，例如 C/C++, Rust, 同时能够在运行在 web 的同时，能够拥有接近原生的速度。这是之前运行在 web 的客户端应用程序所不能做到的。

WebAssembly 旨在补充(提供接近原生的性能) JavaScript 并与之一起运行 - 使用 WebAssembly 的 JavaScript 的 APIS， 你可以在 JavaScript 应用程序中共享 WebAssembly 模块，同时在两者之间共享功能。这允许你能够在同一个 web 应用程序中既能利用 WebAssembly 的性能和强大，也能够利用 JavaScript 的开发经验和灵活性，甚至在你根本不需要知道 WebAssembly 的代码。

更好的是，在 **W3C** 和 **Community Group**  以及主流浏览器厂商的参与下，WebAssembly 正在发展为一种 web 标准。

# 参考
[原文地址](https://developer.mozilla.org/en-US/docs/WebAssembly)