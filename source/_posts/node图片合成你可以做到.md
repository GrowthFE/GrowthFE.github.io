---
title: node 实现图片合成
---

### node 实现图片操作 ###

在一个偶然的下午，我发现某城同学利用canvas 浏览器端合成图片的项目，遇到性能优化的需求，我觉得后端图片合成可以解决浏览器合成图片延迟问题，因此就私下调研了关于node图片合成相关的技术方案，下面就介绍一下比较流行的解决方案。

#### 目前流行的技术  ####

1. 利用gm库实现图片的压缩，合成等。
2. 利用node-canvas实现node端的canvas绘图。

先说一下第一种，也就是今天要介绍的主角，轻量无痛老牌顺手，值得码农青睐。第二种下次介绍，主要是canvas技术应用。





gm就是GraphicsMagick+ImageMagick。

何为GraphicsMagick？
> GraphicsMagick号称图像处理领域的瑞士军刀。 短小精悍的代码却提供了一个鲁棒、高效的工具和库集合，来处理图像的读取、写入和操作，支持超过88种图像格式，包括重要的DPX、GIF、JPEG、JPEG-2000、PNG、PDF、PNM和TIFF。 其实GraphicsMagick是从 ImageMagick 5.5.2 分支出来的，但是现在他变得更稳定和优秀。

何为ImageMagick?
> ImageMagick是一款创建、编辑、合成，转换图像的命令行工具。支持格式超过 200 种，包括常见的 PNG, JPEG, GIF, HEIC, TIFF, DPX, EXR, WebP, Postscript, PDF, SVG 等。功能包括调整，翻转，镜像(mirror)，旋转，扭曲，修剪和变换图像，调整图像颜色，应用各种特殊效果，或绘制文本，线条，多边形，椭圆和贝塞尔曲线等。

但是对于node来说他们好像帮不上什么忙，因此gm为了node，将二者做了封装。

---
在开始gm之前首先需要安装GraphicsMagick和ImageMagick，
```javascript
brew install imagemagick graphicsmagick
```
安装完毕后，安装gm

npm i gm --save

上代码：
```javascript
const gm = require('gm')
gm('./nodeimg.png')
  .draw('image Over 46, 706, 102, 102 "./code.png"')
  .draw('image Over 479,704, 152,56 "./beike.png"')
  //.stream() 
  .write(`./output/${Date.now()}.jpg`, function(err) {
    if (!err) {
      console.log('done')
    }else {
      console.log(err.message || "出错了！");
    }
  })
```
node执行相关js。成功返回图片信息。

对于图片来说我们会将其存储到cdn中。但是对于我们的业务来说，这个需要结合BUCKY的上传组件，此组件是基于AWS.S3，bucky没有过多阐述相关用法，详细 API 说明可见 http://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/S3.html。

我们看以下文档发现，s3 上传图片需要一个stream object对象存在。因此我们需要将gm的绘图结果通过stream()API进行转化，实现s3需要的stream对象。上传操作切记需要async/await支持哈。
![image](http://image-cherry.test.upcdn.net/cherryDesign/upload.png)

最终实现就留给大家上手了。

node实践：
```javascript
const gm = require('gm')
let  body = gm('./nodeimg.png')
.draw('image Over 46, 706, 102, 102 "./code.png"')
.draw('image Over 479,724, 152,56 "./beike.png"')
.fontSize(36)
.font('./font/FZZXHJW.TTF')
.stroke("#999999",2)
.drawText(200,764, "增长线前端出品")       
.write(`./output/gm${Date.now()}.jpg`, function(err) {
    if (!err) {
      console.log('done')
    }else {
      console.log(err.message || "出错了！");
    }
  })
```
温馨提示：使用字体时注意版权问题哈。

![demo](http://image-cherry.test.upcdn.net/cherryDesign/node绘图.png)