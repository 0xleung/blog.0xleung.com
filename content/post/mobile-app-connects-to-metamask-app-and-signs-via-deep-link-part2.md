---
title: "使用universal link巧妙解决App端使用metamask App授权以及交互 (part 2)"
date: 2022-03-04T18:39:55+08:00
draft: false
tags: ["web3", "metamask", "reactnative"]
categories: ["web3"]
author: "Jobin Leung"
contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'
---
今天我们介绍一下app端的实现, 一直没写也是因为我觉得app端确实没啥可以写的.

它真的很简单.

我就基于RN + iOS一起回顾一下.

主要是两点.

### 一是universal link的配置!

这个网上讲这个的太多了我就找了一个感觉讲的还不错的给大家贴个链接 [https://abhimuralidharan.medium.com/universal-links-in-ios-79c4ee038272](https://abhimuralidharan.medium.com/universal-links-in-ios-79c4ee038272) (主要是因为懒)

### 二是 React Linking的使用

#### 打开metamask

引入link和button

```ts
import {
  Button,
  Linking,
} from 'react-native';
```

点击按钮通过链接打开metamask

```ts
<Button
    color="#fff"
    title="Go to Sign"
    onPress={() => {
        Linking.openURL(
        'https://metamask.app.link/dapp/www.jobinleung.me/sign',
        );
    }}>
    Sign
</Button>
```

#### 返回App

监听返回链接以及参数

```ts
const onAppWokUp = (event: {url: string}) => {
    setInintUrl(event.url);
};

useEffect(() => {
    Linking.getInitialURL().then(url => {
        if (url) {
        setInintUrl(url);
        }
    });
    Linking.addEventListener('url', onAppWokUp);
    return () => {
        Linking.removeEventListener('url', onAppWokUp);
    };
}, []);
```

就是这么简单 我把app端的代码也放到GitHub感兴趣的大家可以去瞧一瞧!

[https://github.com/JobinLeung/ConnectWithMetamaskDemo](https://github.com/JobinLeung/ConnectWithMetamaskDemo)