---
title: "About Signal"
date: 2023-03-02T14:07:09+08:00
draft: false
tags: ["signal", "前端", "react", "Qwik", "Solid.js"]
categories: ["前端"]
author: "Jobin Leung"
contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'
---

最近前端有个概念特别火, “Signal”

最近，Angular、Qwik 的作者 MIŠKO HEVERY 发文表示<a rel="license noopener" href="https://www.builder.io/blog/usesignal-is-the-future-of-web-frameworks#code-use-ref-code-does-not-render" target="_blank">useSignal() is the Future of Web Frameworks</a>，并考虑在 Angular 中实现它。

在此之前，Vue、Solid.js、Preact、Svelte 都已实现 Signal。实际上，signal 并不是一个新概念，他还有很多别名，比如：响应式更新

如果你了解过 Vue 响应式更新的实现原理，对 Signal 就不会陌生。

实际上，Signal 的技术在 10 年前 Knockout 框架中就有应用。为什么这项技术正受到越来越多前端框架的青睐？

### Signal 的本质

Signal 和 State 之间的主要区别在于 Signal 返回一个 getter 和一个 setter，而非响应式系统返回一个值和一个 setter。

```
useState() => value + setter
useSignal() => getter + setter
```

可以发现，state 耦合了对状态的引用以及对状态值的获取这两个含义。signal 区别就体现在 getter 上，其中：

getter 是对状态的引用

getter()是对状态值的获取

也就是说，我们可以不必立刻获取状态的值，而是在需要的时候再获取（即在需要时再执行 getter()）。

这么做有什么好处呢？如果我们在需要的时候再获取状态的值，就能感知当前的上下文环境。

相比于 React，基于 Signal 实现的框架优势：

使用 Signal 的框架通常能获得不错的运行时性能，所以不需要额外的性能优化 API。反观 React，开发者如果遇到性能问题，需要手动调用性能优化 API（比如 React.memo、useMemo、PureComponent）。

### 最后

有了以上这么多优点，难怪很多框架都使用 Signal。那么 React 对 Signal 的态度是什么呢？

React 团队成员的观点是：

- 有可能会引入类似 Signal 的原语
- Signal 确实有很好的性能，但不太符合 React 的理念

React 的理念可以用一句话概括：UI 反映状态在某一刻的快照。既然是快照，那就不是局部的，而是整体的概念。在 React 中，状态更新会引起整个应用重新渲染，这是对 React 快照理
