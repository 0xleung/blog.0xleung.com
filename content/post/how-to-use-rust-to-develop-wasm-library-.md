---
title: "怎样使用rust开发wasm库"
date: 2023-01-13T17:52:24+08:00
draft: false
tags: ["rust", "wasm", "react", "WebAssembly", "npm"]
categories: ["wasm"]
author: "Jobin Leung"
contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'
---

记录一下使用rust为网站写一个wasm来运算接口签名库的关键步骤.

### 第一, 环境

除了rust cargo等rust开发必需品使用rust开发wasm还需要以下3点

#### 1. wasm-pack

wasm-pack 是构建、测试和发布 Rust 生成的 WebAssembly 的一站式工具。安装命令:

```bash
curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh
```

#### 2. cargo-generate

cargo-generate 通过利用预先存在的 git 存储库作为模板，帮助您快速启动并运行新的 Rust 项目。安装命令:

```bash
cargo install cargo-generate
```

#### 3. npm

由于我自己的本职工作是前端这个我就不多做介绍了, nodejs的包管理器. 如果后面开发完成后可以将开发的wasm发布到npm, 如果安装完nodejs自动就已经安装好了npm, 但是请确保它是最新的版本, 使用命令:

```
npm install npm@latest -g
```

环境准备就已经告一段落了, 接下来我们一起学习使用rust来开发wasm的基础操作

### 第二, 编码

#### 1. 生成工程

使用以下命令生成工程, 

```
cargo generate --git https://github.com/rustwasm/wasm-pack-template
```

这命令会提示您输入新项目的名称。 我们将使用“wasm-greet”。

#### 2. 工程结构

进入`wasm-greet`工程内部

```
cd wasm-greet
```

然后让我们一起看一看工程内部长啥样

```
wasm-greet/
├── Cargo.toml
├── LICENSE_APACHE
├── LICENSE_MIT
├── README.md
└── src
    ├── lib.rs
    └── utils.rs
```

`wasm-greet/Cargo.toml`

稍微了解rust就会知道这是rust的包管理器类似于node的npm, 这个预配置了一个 wasm-bindgen 依赖项，一些我们稍后将深入研究的可选依赖项，并且为生成 .wasm 库正确初始化了 crate-type。

`wasm-greet/src/lib.rs`

src/lib.rs 文件是我们正在编译为 WebAssembly 的 Rust crate 的根目录。 它使用 wasm-bindgen 与 JavaScript 交互。 它导入 window.alert JavaScript 函数，并导出 greet Rust 函数，该函数会发出问候消息。

`wasm-greet/src/utils.rs`

src/lib.rs 文件是我们正在编译为 WebAssembly 的 Rust crate 的入口文件。 它使用 wasm-bindgen 与 JavaScript 交互。 它导入 window.alert JavaScript 函数，并导出 greet Rust 函数，该函数会发出问候消息。

```
mod utils;

use wasm_bindgen::prelude::*;

// When the `wee_alloc` feature is enabled, use `wee_alloc` as the global
// allocator.
#[cfg(feature = "wee_alloc")]
#[global_allocator]
static ALLOC: wee_alloc::WeeAlloc = wee_alloc::WeeAlloc::INIT;

#[wasm_bindgen]
extern {
    fn alert(s: &str);
}

#[wasm_bindgen]
pub fn greet() {
    alert("Hello, wasm-greet!");
}

```

`wasm-greet/src/utils.rs`

src/utils.rs 模块提供了常用的实用库，可以更轻松地使用编译为 WebAssembly 的 Rust。 但就这个程序而言可以忽略此文件。

#### 3. 编译项目

我们使用 wasm-pack 来编排以下构建步骤：

1. 确保我们有 Rust 1.30 或更新版本，并且通过 rustup 安装了 wasm32-unknown-unknown 目标，
2. 通过 cargo 将我们的 Rust 源代码编译成 WebAssembly .wasm 二进制文件，
3. 使用 wasm-bindgen 生成 JavaScript API 以使用我们的 Rust 生成的 WebAssembly。

要完成所有这些，请在项目目录中运行此命令：

```
wasm-pack build
```

构建完成后，我们可以在 pkg 目录中找到它的工件，它应该包含以下内容：

```
pkg/
├── package.json
├── README.md
├── wasm_game_of_life_bg.wasm
├── wasm_game_of_life.d.ts
└── wasm_game_of_life.js
```

#### 4. 发布

```
wasm-pack login
wasm-pack build
wasm-pack publish
```

### 第三, 使用

在前端工程中使用, 在package.json中加入引用.(没有发布到npm也可以使用本地目录或者git)

如果你使用的是create-react-app创建的react项目, 很不幸它将不支持 WebAssembly。 我们必须对为应用程序提供支持的底层 webpack 配置进行一些更改。 不幸的是，create-react-app 没有公开 webpack 配置文件。 所以，我们需要引入一些开发依赖来帮助我们。 react-app-rewired 将允许我们在不 ejecting 的情况下修改 webpack，而 wasm-load 将帮助 webpack 处理 WebAssembly。

```
yarn add react-app-rewired wasm-loader --dev
```

修改`config-overrides.js`

```
const path = require("path");

module.exports = function override(config, env) {
  const wasmExtensionRegExp = /\.wasm$/;

  config.resolve.extensions.push(".wasm");

  config.module.rules.forEach(rule => {
    (rule.oneOf || []).forEach(oneOf => {
      if (oneOf.loader && oneOf.loader.indexOf("file-loader") >= 0) {
        // make file-loader ignore WASM files
        oneOf.exclude.push(wasmExtensionRegExp);
      }
    });
  });

  // add a dedicated loader for WASM
  config.module.rules.push({
    test: wasmExtensionRegExp,
    include: path.resolve(__dirname, "src"),
    use: [{ loader: require.resolve("wasm-loader"), options: {} }]
  });

  return config;
};
```

使用

```
function Welcome(props) {
  useEffect(()=>{
    import("wasm-greet").then((wasm)=>{
        wasm.greet();
    });
  },[])
  return <h1>Hello, {props.name}</h1>;
}
```

本次极简版rust开发wasm库到此结束, 后续会持续推出一些有用的rust wasm开发的实用库.

最后对rust感兴趣的可以加我微信我们一起探讨, 还在不断学习中.