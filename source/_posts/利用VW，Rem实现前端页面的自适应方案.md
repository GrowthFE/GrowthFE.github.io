---
title: 利用VW，Rem实现前端页面的自适应方案
---

==题记==：目前增长团队手机端适配方案不统一，应对WAP页面的需求开发，怎么解决？标准统一对于后续开发迭代很关键，所以基于这个出发点前端团队提出需要制定相关技术标准方案。其中今天说到的关于移动端适配就是我们开始的第一步。

*** 

#### 技术背景 ####
* 1.响应式web设计能做什么？
    
    让一个网站同时适配多种设备和多个屏幕，可以让网站的布局和功能随用户的使用环境（屏幕大小、输入方式、设备/浏览器能力）而变化。
* 2.什么是viewport？

    浏览器中用于呈现网页的区域叫视口（viewport）视口通常并不等于屏幕大小，特别是可以缩放浏览器窗口的情况下。为了解决前面的问题，可以在网页的<head>中添加下面这行代码：
    ```
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no, minimal-ui">
    ```

* 3.Rem,Vw具体指的是什么？

    rem的——“font size of the root element”
    viewport宽度有关，100%视口宽度 = 100vw  
    
    [vw兼容性]
    
#### 技术方案 ####

==利用Rem Vw 实现响应式布局的基准==

++实现方案：++

**利用less的函数方式实现相关转换**
假定对应的设计稿750px宽的，100vw,那么量出来的宽度直接用less中定义的函数解决就好：
```
    @clientW:750;
    .pxtovw(@value) {
        @pxtovw:100vw*@value/@clientW;
        @pxtorem:@value/100rem;
    }
    
    //具体使用方式
    .blk_main{
        .pxtovw(750);
         min-width:@pxtorem;
         min-width:@pxtovw;
    }
```

**纯css3属性实现无需编译** (基于rem的 设计稿750宽，假定字体大小100px)

1.通用方案 rem  js实现
```
(function(doc, win,baseDesignValue) {
        var Dpr = 1,
            uAgent = window.navigator.userAgent;
        var isIOS = uAgent.match(/iphone/i);
        var is2345 = uAgent.match(/Mb2345/i);
        var ishaosou = uAgent.match(/mso_app/i);
        var isSogou = uAgent.match(/sogoumobilebrowser/ig);
        var isLiebao = uAgent.match(/liebaofast/i);
        var isGnbr = uAgent.match(/GNBR/i);

        function resizeRoot() {
            var wWidth = (screen.width > 0) ? (window.innerWidth >= screen.width || window.innerWidth == 0) ? screen.width : window.innerWidth : window.innerWidth,
                wDpr, wFsize;
            var wHeight = (screen.height > 0) ? (window.innerHeight >= screen.height || window.innerHeight == 0) ? screen.height : window.innerHeight : window.innerHeight;
            if (window.devicePixelRatio) {
                wDpr = window.devicePixelRatio;
            } else {
                wDpr = isIOS ? wWidth > 818 ? 3 : wWidth > 480 ? 2 : 1 : 1;
            }
            if (isIOS) {
                wWidth = screen.width;
                wHeight = screen.height;
            }
           
            if (wWidth > wHeight) {
                wWidth = wHeight;
            }
            wFsize = wWidth > 540 ? 72 : wWidth*100 / baseDesignValue;
            wFsize = wFsize > 32 ? wFsize : 32;
            window.screenWidth_ = wWidth;
            if (is2345 || ishaosou || isSogou || isLiebao || isGnbr) { //这里有个刚调用系统浏览器时候的bug，需要一点延迟来获取
                setTimeout(function() {
                    wWidth = (screen.width > 0) ? (window.innerWidth >= screen.width || window.innerWidth == 0) ? screen.width : window.innerWidth : window.innerWidth;
                    wHeight = (screen.height > 0) ? (window.innerHeight >= screen.height || window.innerHeight == 0) ? screen.height : window.innerHeight : window.innerHeight;
                    wFsize = wWidth > 540 ? 72 : wWidth / 7.5;
                    wFsize = wFsize > 32 ? wFsize : 32;
                    document.getElementsByTagName('html')[0].style.fontSize = wFsize + 'px';
                }, 500);
            } else {
                document.getElementsByTagName('html')[0].style.fontSize = wFsize + 'px';
            }
            // alert("fz="+wFsize+";dpr="+window.devicePixelRatio+";UA="+uAgent+";width="+wWidth+";sw="+screen.width+";wiw="+window.innerWidth+";wsw="+window.screen.width+window.screen.availWidth);
        }
        if (!doc.documentElement.addEventListener) return;
        doc.addEventListener('DOMContentLoaded', resizeRoot, false);
        resizeRoot();
    })(document, window,750);

```

2.激进的方案：rem + vw + 1000px宽设计稿


```
@clientW:1000;
.pxtovw(@value) {
    @pxtovw:100vw*@value/@clientW; //@value/10
    @pxtorem:@value/100rem;
}

html {
    height: 100%;
    font-size: 10vw; //100vw *100px/1000px
    line-height: 1;
    max-width: 750px;
    overflow-x: hidden;
    font-family: Microsoft YaHei, STHeiti, Helvetica, Arial, sans-serif!important
}

.blk-main{
    .pxtovw(640);
     min-width:@pxtorem;
     min-width:@pxtovw;
}

最终??

优化：
    属性值：
        vw => 除以10 单位vw
        rem => 除以100 单位rem


```
最终如果要兼容低版本终端，我建议就使用REM就够了，VW也只是一个新的方法。


