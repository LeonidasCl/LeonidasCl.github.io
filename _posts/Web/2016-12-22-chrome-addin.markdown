---
layout: post
title:  "开发Chrome插件，给原有网页插上代码"
date:   2016-12-22 23:16:06
categories: Web
comments: true
---

　　有时候我们想要在网页上做一些自动化操作来解放自己的双手，或者重复刷取某些东西。各种浏览器可以很方便地添加插件，今天就来展示下如何自己添加Chrome插件。

　　插件的文件都要放在一个目录里，我们先建一个这样的目录，然后再建立manifest文件，这是一个json格式的文件，用来描述插件的基本配置：

```json
{
  "manifest_version": 2,/*版本1或者2在一些配置字段名上会有不同，这里使用2*/

  "name": "excited add-in",					/*插件名称*/
  "description": "js add-in",				/*描述*/
  "version": "1.0",							/*自己的版本号*/
  "permissions": [							/*可选的权限*/
    "tabs", 
    "http://*/*",
    "https://*/*"		
  ],
  "content_scripts": [
    {"js":["script1.js"],"matches":["https://www.hao123.com/"]},
    {"js":["script2.js"],"matches":["https://*.taobao.com/*"]}
  ]
}
```

　　最后的content_scripts就是要加载的js文件与网址的匹配情况，这里可以加载js也可以加载css或html，当打开的网页符合matches中的规则时，对应的脚本就会被加载，非常简单的逻辑。当然，manifest文件的可用字段远不止这些，还有很多可以探寻，比如插件的icon之类。

　　写好了manifest，就可以来写一写两个js了，因为只是简单尝试功能，所以js也很简单：

script1中有语句：

```javascript
function check(){

alert('开启了hao123');

location.reload();

}

setInterval("check()",4000);
```

script2中有语句：

`alert("正在访问淘宝");`

　　最后，在chrome中加载这个插件，选择chrome的菜单--更多工具--扩展程序，点击“加载已解压的扩展程序”，选择好刚才的路径就可以了，可以看到插件已经加载进来，且可以很方便地启动和停止。

![图1](http://obdvl7z18.bkt.clouddn.com/img/20161223/01.jpg)



现在打开hao123，就会发现页面弹出提示框，且每四秒刷新一次:

![图2](http://obdvl7z18.bkt.clouddn.com/img/20161223/02.jpg)



打开淘抢购页面，脚本2就会被加载：

![图3](http://obdvl7z18.bkt.clouddn.com/img/20161223/03.jpg)



真是so simple，so naive!