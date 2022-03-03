---
title: "使用universal link巧妙解决App端使用metamask App授权以及交互 (part 1)"
date: 2022-03-03T15:51:20+08:00
draft: false
tags: ["web3", "metamask", "reactnative"]
categories: ["web3"]
author: "Jobin Leung"
contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'
---
前两天遇到一个app端的钱包授权问题, 具体需求是: **在app端点击链接钱包跳转metamask钱包app 连接 签名返回app后和server交互验证签名用钱包登录.**

首先, 我查了metamask有关文档, 相关内容真的非常少. 后来发现 metamask有universal link的API.

好了那就开整吧, coding>调用>完成任务.

并没有那么顺利, 他的接口只有3个方法,

1.  打开一个Dapp(web link).
1.  支付请求(Eth或者Erc20 token).
1.  Payment Channel Request.

😭没有签名方法呀! 而且universal link有一个弊端就是在metamask做完操作无法主动跳回来源App.

怎么办! 无法解决问题怎么办呀!

等一下, 是不是可以打开一个dapp. 是不是可以通过web页面来处理详细业务逻辑呢, 好! 那我们就开始!

首先,我们先构建一个universal link, 这里有构建链接 <https://metamask.github.io/metamask-deeplinks/>

确定我们web3交互地址在 <https://www.jobinleung.me/sign> 生成的link是 <https://metamask.app.link/dapp/www.jobinleung.me/sign>

然后我们经过多次生成发现链接生成规则是

```ts
const hostname = 'www.jobinleung.me';
const path = '/sign';
const targetLink = `https://metamask.app.link/${hostname+path}`;
// metamask 目标链接只支持https
```
第二,让我们完成web端也就是dapp部分的编码和设计.

首先我们需要一个与metamask交互的类

这里我选择使用ether.js来处理, 当然使用web3.js也可以.

这个类包括了签名和验证签名的方法.

写一个页面来承载功能, 为了简化我把这个页面放在了我的个人网站的sign页面.

```tsx
import Button from "@mui/material/Button/Button";
import { useEffect, useState } from "react";
import { ethers } from "ethers";


class MetamaskSign {
    public static isMetamaskInstalled(): boolean {
        return typeof window.ethereum !== "undefined";
    }

    public static isMetamaskLocked(): boolean {
        return typeof window.ethereum.isMetaMask !== "undefined" && !window.ethereum.isMetaMask;
    }

    public static getAccount(): string {
        return window.ethereum.selectedAddress;
    }

    public static getProvider(): ethers.providers.JsonRpcProvider {
        return new ethers.providers.Web3Provider(window.ethereum);
    }

    public static getSigner(): ethers.Signer {
        return this.getProvider().getSigner();
    }

    public static async sign(msg: string): Promise<string> {
        try {
            const signer= new ethers.providers.Web3Provider(window.ethereum).getSigner();
            const signature= await signer.signMessage(msg);
            return signature;
        } catch (error: any) {
            return error.toString()
        }
    }

    public static async verify({msg, addr, sig}: {msg:string, addr: string, sig:string}): Promise<boolean>{
        try {
            const signerAddr = await ethers.utils.verifyMessage(msg, sig);
            return addr.toLowerCase() === signerAddr.toLowerCase();
        } catch (error) {
            console.log(error)
            return false
        }
    }
}

/**
 * sign page
 */
const Sign = ()=>{

    const [sig, setSig] = useState('');
    const [accountAdress, setAccountAdress] = useState('');
    

    useEffect(() => {
        if(!MetamaskSign.isMetamaskInstalled())(
            alert('Please open with Metamask')
        )
        if(!MetamaskSign.isMetamaskLocked()){
            window.ethereum.request({ method: 'eth_requestAccounts' }).then(()=>{
                return MetamaskSign.getSigner().getAddress()
            }).then((address:string)=>{
                setAccountAdress(address);
            })
        }
    }, []); 
    
    return (
        <>
            <div style={{height:'100vh',lineHeight:'100vh',textAlign:'center',}}>
                {accountAdress}
                {sig}
                <Button variant="contained" onClick={async ()=>{
                    const s = await MetamaskSign.sign('app metamask sign test');
                    setSig(`${accountAdress}:${s}`)
                    window.location.href = `https://medium.com/@jobinleung/%E4%BD%BF%E7%94%A8universal-link%E5%B7%A7%E5%A6%99%E8%A7%A3%E5%86%B3app%E7%AB%AF%E4%BD%BF%E7%94%A8metamask-app%E6%8E%88%E6%9D%83%E4%BB%A5%E5%8F%8A%E4%BA%A4%E4%BA%92-part-1-ee53bde721a`;
                }}>Sign</Button>
            </div>
        </>
    )
}

export default Sign;
```
首先我们需要模拟app打开metamask App.

我们可以点击这个链接 <https://metamask.app.link/dapp/www.jobinleung.me/sign> 跳转至metamask App 授权完成后我们再使用medium的universal link返回

这样App端使用metamask App授权以及交互的web部分就完成了.

可以参照: <https://github.com/JobinLeung/jobinleung.me/blob/main/src/pages/sign/sign.tsx>

下次我会继续介绍, app端的实现细节.

下次见.