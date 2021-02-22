# case
提取文章正文的H标签作为目录


需求细定
一、提取 HTML 的 H 标签作为目录，根据 H1 ~ H6 的语义制定相应样式（字号、边距） ，目录模块固定在页面，悬浮时候添加背景色
二、点击目录，被点击者高亮，页面滚动到相应模块
三、滚动页面，正文某标题到了可视区域高度的2/5 对应目录标题高亮

目录大概长这样~~~~
https://github.com/MiniCai/images/blob/main/images/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_ac640596-d6ae-4ce1-99e3-b52c062a102a.png

开发要点
来，有事没事先上一波 html 的代码~~~
https://github.com/MiniCai/images/blob/main/images/WeChatc368f1e3408599be183289f5259333c7.png

由于提取的h标签，可能会包含嵌套关系，所以采用document.querySelectorAll("*") 先提取全部标签，然后再筛选出H1~ H6。
根据H标签的语义及常规写作习惯，规定相应样式

https://github.com/MiniCai/images/blob/main/images/WechatIMG1.jpeg

lighthigh-follow 是为了滚动高亮需要筛选出目标标题缩添加的特定class
id 是点击锚点定位所需

https://github.com/MiniCai/images/blob/main/images/WechatIMG3.jpeg

因为正文数据是后端返回来的，等待数据返回后需要重新渲染，所以在mounted 的时候是拿不到真实全部渲染完的html，经分析后需在updated 时期获取（ 也可以尝试一下$nextTick，毕竟是mvvm架构）。updated不能操作data里的数据，否则造成死循环。。。

https://github.com/MiniCai/images/blob/main/images/WechatIMG2.jpeg
https://github.com/MiniCai/images/blob/main/images/WeChat8835faa64bfcc7b4dae9fb696c77f2ae.png

根据上述生成节点插入dom树。为了减少重排，这里推荐一个很好用api —— insertAdjacentHTML
https://github.com/MiniCai/images/blob/main/images/WeChat4d3979595db124e3f81fbdd86af54fff.png

这里的text是 目录fragments

https://github.com/MiniCai/images/blob/main/images/WechatIMG4.jpeg


上面的步骤完成后数据就出来了，duang！点击锚点定位，样式交互这里采用添加、移除class 进行 目录标题的高亮，以减少重绘。锚点定位需要获取正文标题的offsetTop， 也可以用getBoundingClientRect， ScrollIntoView等方法

https://github.com/MiniCai/images/blob/main/images/WechatIMG5.jpeg

最后一个是滚动高亮，我是用getBoundingClientRect这个 api，当然也可以用offsetTop等方法
Element.getBoundingClientRect()方法
rectObject = object.getBoundingClientRect();
返回值是一个 DOMRect 对象，这个对象是由该元素的 getClientRects() 方法返回的一组矩形的集合, 即：是与该元素相关的CSS 边框集合 。DOMRect 对象包含了一组用于描述边框的只读属性——left、top、right和bottom，单位为像素。除了 width 和 height 外的属性都是相对于视口的左上角位置而言的。

https://github.com/MiniCai/images/blob/main/images/rect.png
https://github.com/MiniCai/images/blob/main/images/WechatIMG6.jpeg
