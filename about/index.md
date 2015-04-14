---
layout: page
title: About Me
excerpt: "So Simple is a responsive Jekyll theme for your words and images."
modified: 2014-08-08T19:44:38.564948-04:00
image:
  feature: so-simple-sample-image-4.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
---

---

# 联系方式


- 手机：***********
- Email：chenhuazhaoao@gmail.com
- QQ/微信号：742223410

---

（注：以下项目名称以及文章标题均为超链接，阅读纸质简历时请移步个人博客以及Github主页查看）


# 个人信息

 - ***/男/1992 
 - 本科/广东工业大学软件工程 
 - 工作年限：1年
 - 技术博客：[http://leaking.github.io](http://leaking.github.io)
 - GitHub:  [https://github.com/Leaking](https://github.com/Leaking)
 - 期望职位：Android程序员
 - 期望城市：深圳

---

# 工作经历


## 平安科技 （ 2014年7月至今 ）

### 爱加加
该项目是一个图片社交App，已经发布到360手机助手和腾讯应用宝。我负责Android端的实现，最大的收获是第一次参与了整个App从头到尾的完整开发流程。在技术上，主要遇到两个难点。第一个难点是App架构的设计，由于当初经验有限，为了避免项目后期难以维护，难以扩展，最开始广泛阅读了关于Android App开发中的架构设计的相关文档，以及参考了Oschina和Github的Android客户端源码，阅读源码过程中受益匪浅。第二个难点是实现中的“心愿卡相机”功能模块，该功能是在自定义相机上放置一个可以自由旋转移动的卡片，卡片上可以写字，再通过图片的合成，使得相片的人如同抱着/托着一个实体卡片。开发该功能的主要难度是关于自定义相机的清晰度以及在不同机型上的适配。最后通过传感器监听手机移动情况，当手机移动超出某个数值的距离时则相机聚焦一次，而相机在不同机型上预览和照片的长宽比例，借助了第三方依赖库。

<figure>
  <img src="{{ site.url }}/images/iplus.jpg" alt="二维码" width="200" height="200">
  <figcaption>爱加加二维码</figcaption>
</figure>



### 口袋E行销项目 
该项目是平安集团的保险业务员进行展业时使用的App，目前用户量有49万左右。该App主要使用了单页面技术以及PhoneGap框架。我参与开发了其中的WebView缓存机制，在客户端保存一份服务器端需要传递给客户端的页面文件以及相关资源文件，并记录其版本号。然后在WebView加载页面时，拦截请求，比较客户端和服务器端的页面文件的版本，若客户端的页面文件不是最新版本，则继续发送请求加载服务器端页面，并更新本地缓存的页面；当客户端的页面文件是最新版本，则WebView直接加载本地页面。通过这个缓存机制，大大减少了网络传输流量，缓解了服务端的压力，提高了页面加载速度。该项目让我学习了Hybrid App开发中的多项技术，并深入的学习了WebView的原理。


---

# 开源项目和作品


## 开源项目


 - [SlideSwitch](https://github.com/Leaking/SlideSwitch) : 这是一个自定义UI控件。主要实现了一个开关按钮，用户可以选择IOS风格的椭圆开关或者Android端常见的矩形侧滑开关，其中加入了颜色渐变和动画效果。截止目前有127个Star和43个Fork。目前依据使用该控件的用户反馈的问题仍在进行维护升级。
 - [xmppIM](https://github.com/Leaking/xmppIM) : 该项目主要实现一个Android端的即时通讯功能，旧版本里实现了文字，表情，文件，语音的通信，现在重构代码在最新的Material Design的风格上重新开发。目前业余时间在维护该项目。

## 技术文章


- [Android文档翻译----Processes and Threads](http://leaking.github.io/android/(translate)ProcessesAndThreads/)


# 技能清单




- Android开发：自定义UI控件/线程机制/HTTP/WebView/Hybrid App/Material Design等
- Hybrid App框架：PhoneGap
- Web技术：HTML/CSS/JS/Struts2/Spring3
- 数据库相关：MySQL/MSSQL/SQLite
- 版本管理：Git/SVN
- 单元测试：JUnit
- 云和开放平台：百度地图/微信开放平台
- 语言：Java/C/C++/Python

---




