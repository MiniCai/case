# case
提取文章正文的H标签作为目录


# 需求细定

* 提取 HTML 的 H 标签作为目录，根据 H1 ~ H6 的语义制定相应样式（字号、边距） ，目录模块固定在页面，悬浮时候添加背景色

* 点击目录，被点击者高亮，页面滚动到相应模块

* 滚动页面，正文某标题到了可视区域高度的2/5 对应目录标题高亮

目录大概长这样~~~~

![image](https://github.com/MiniCai/images/blob/main/images/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_ac640596-d6ae-4ce1-99e3-b52c062a102a.png)

# 开发要点

来，有事没事先上一波代码~~~

```html

<ul
      class="directory-wrapper"
      ref="directoryUl"
      @click="lightHigh($event)"
></ul>
```

```css
<style>
.li {
  display: block;
  color: #313131;
  line-height: 42px;
  cursor: pointer;
  padding: 0 24px;
}
.li.active {
  color: rgba(255, 208, 75, 1) !important;
}
.li.active::before {
  background-color: rgba(255, 208, 75, 1) !important;
}

.li:hover {
  background-color: rgba(255, 255, 255, 0.5);
}
.H1 {
  font-size: 18px;
  font-weight: 700;
}
.H1::before {
  content: "";
  display: inline-block;
  width: 10px;
  height: 10px;
  margin-right: 8px;
  background-color: #999;
  border-radius: 50%;
  position: relative;
  top: -2px;
}
.H2 {
  font-size: 16px;
  padding-left: 44px;
}
.H2::before {
  content: "";
  display: inline-block;
  width: 7px;
  height: 7px;
  margin-right: 8px;
  background-color: #999;
  border-radius: 50%;
  position: relative;
  top: -2px;
}
.H3 {
  padding-left: 62px;
}
.H4,
.H5,
.H6 {
  font-size: 12px;
  padding-left: 76px;
}
.H3::before,
.H4::before,
.H5::before,
.H6::before {
  content: "";
  display: inline-block;
  width: 6px;
  height: 6px;
  margin-right: 8px;
  background-color: #999;
  border-radius: 50%;
  position: relative;
  top: -2.5px;
}
</style>
```

#### 由于提取的h标签，可能会包含嵌套关系，所以采用document.querySelectorAll("*") 先提取全部标签，然后再筛选出H1~ H6。

根据H标签的语义及常规写作习惯，规定相应样式

![image](https://github.com/MiniCai/images/blob/main/images/WechatIMG1.jpeg)

lighthigh-follow 是为了滚动高亮需要筛选出目标标题缩添加的特定class

id 是点击锚点定位所需

```javascript
getDom() {
      const hArr = ["H1", "H2", "H3", "H4", "H5", "H6"],
        target = [];
      let dom = document.querySelectorAll("*");
      dom.forEach((item, index) => {
        if (hArr.indexOf(item.nodeName) > -1) {
          item.id = `${item.nodeName}_${index}`;
          item.classList.add("lighthigh-follow");
          target.push({
            value: item.nodeName,
            label: item.innerText,
            href: `#${item.id}`,
          });
        }
      });
      var directory = this.$refs.directoryUl,
        fragments = "";
      target.slice(2).forEach((item) => {
        fragments += this.integrationLi(item);
      });
      directory.insertAdjacentHTML("beforeend", fragments);
}
```

#### 因为正文数据是后端返回来的，等待数据返回后需要重新渲染，所以在mounted 的时候是拿不到真实全部渲染完的html，经分析后需在updated 时期获取（ 也可以尝试一下$nextTick，毕竟是mvvm架构）。updated不能操作data里的数据，否则造成死循环。。。

```javascript
updated() {
    this.getDom();
}
```

![image](https://github.com/MiniCai/images/blob/main/images/WeChat8835faa64bfcc7b4dae9fb696c77f2ae.png)

#### 根据上述生成节点插入dom树。为了减少重排，这里推荐一个很好用api —— insertAdjacentHTML

![image](https://github.com/MiniCai/images/blob/main/images/WeChat4d3979595db124e3f81fbdd86af54fff.png)

这里的text是 目录fragments

```javascript
integrationLi(item) {
      let li = `<li><a class="${item.value} li lighthigh-follow-li" data-id="${
        item.href
      }">${item.label}</a></li>`;
      return li;
}
```

#### 上面的步骤完成后数据就出来了，duang！点击锚点定位，样式交互这里采用添加、移除class 进行 目录标题的高亮，以减少重绘。锚点定位需要获取正文标题的offsetTop， 也可以用getBoundingClientRect， ScrollIntoView等方法。lightHigh 方法是绑定在父元素，目录li 委托父级代为执行事件，也就是事件委托，利用冒泡机制大大减少了对dom的操作，提高性能

```javascript
lightHigh(e) {
      var directoryLi = this.$refs.directoryUl.childNodes,
        target = e.target,
        id = target.dataset.id;
      directoryLi.forEach((item) => {
        item.childNodes[0].classList.remove("active");
      });
      target.classList.add("active");
      // 锚点定位
      let offsetTop = document.querySelector(id).offsetTop;
      document.documentElement.scrollTop = offsetTop - 80;
}
```

#### 最后一个是滚动高亮，我是用getBoundingClientRect这个 api，当然也可以用offsetTop等方法

Element.getBoundingClientRect()方法

rectObject = object.getBoundingClientRect();

返回值是一个 DOMRect 对象，这个对象是由该元素的 getClientRects() 方法返回的一组矩形的集合, 即：是与该元素相关的CSS 边框集合 。DOMRect 对象包含了一组用于描述边框的只读属性——left、top、right和bottom，单位为像素。除了 width 和 height 外的属性都是相对于视口的左上角位置而言的。

![image](https://github.com/MiniCai/images/blob/main/images/rect.png)

```javascript
handleScroll() {
      let dom = document.querySelectorAll(".lighthigh-follow"),
        clientH =
          document.documentElement.clientHeight || document.body.clientHeight,
        directoryLi = document.querySelectorAll(".lighthigh-follow-li");
      if (dom.length > 2) {
        dom.forEach((item, index) => {
          if (index > 1) {
            let clientRectTop = item.getBoundingClientRect().top; // 内容区的top 距离窗口的高度;
            if (clientRectTop <= clientH * 0.4) {
              directoryLi[index - 2].classList.add("active");
              directoryLi.forEach((item, idx) => {
                if (idx !== index - 2) {
                  item.classList.remove("active");
                }
              });
            }
          }
        });
      }
}
```
</br>
