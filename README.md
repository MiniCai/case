# case

基于 vue 提取文章正文的 H 标签作为目录

## 需求细定

- 提取 HTML 的 H 标签作为目录，根据 H1 ~ H6 的语义制定相应样式（字号、边距） ，目录模块固定在页面，悬浮时候添加背景色

- 点击目录，被点击者高亮，页面滚动到相应模块

- 滚动页面，正文某标题到了可视区域高度的 2/5 对应目录标题高亮

## 开发要点

由于 editor 使用的是第三方的，所以 viewer 也配套了第三方的。
```html
<viewer :initialValue="data.val" v-if="data.val" height="100%" />
<!-- 目录 -->
<ul class="directory-wrapper">
  <li v-for="(item, index) in fragments" :key="index" @click="lightHigh(item)">
    <a
      ref="directoryLi"
      class="li lighthigh-follow-li"
      :class="[{ active: currentLi === item.href }, item.value]"
      >{{ item.label }}</a
    >
  </li>
</ul>
```

**分析如何提取的 h 标签**

 `document.querySelectorAll("Hx")` (x表示1~6), 这样确实能筛选出正文的全部H标签，但是嵌套关系就会被打乱。所以改用 `document.querySelectorAll("*")` 提取, 防止嵌套关系被打乱。
 
 正文数据是由后端返回来，再经过第三方工具读取出来。这里涉及一个重渲染问题。
 
 第一想法是从生命周期着手，开始我是想在 updated 时期操作dom插入目录文档片段，好像很完美的样子。

 但是vue是mvvm模型框架，底层就处理了一堆dom操作，再自己操作dom，正如XXX吐槽说的：你这是搞事情！！！

 因此觅得更合理的方法 -- vue针对DOM 更新的常用方法 `Vue.nextTick(callback)`:
 
 在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。

```javascript
mounted() {
  this.$nextTick((_) => {
    this.getDom();
  });
  window.addEventListener("scroll", this.handleScroll, true);
}
destroyed() {
    window.removeEventListener("scroll", this.handleScroll, true);
}
```

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
  this.fragments = target.slice(2);
}
```

锚点定位需要获取正文标题的 offsetTop， 也可以用 getBoundingClientRect， ScrollIntoView 等方法。

```javascript
lightHigh(item) {
  this.currentLi = item.href;
  let offsetTop = document.querySelector(item.href).offsetTop,
    clientH =
      document.documentElement.clientHeight || document.body.clientHeight;
  document.documentElement.scrollTop = offsetTop - clientH * 0.3; // 锚点定位
}
```

最后一个是滚动高亮，这里使用了 getBoundingClientRect 这个 api，当然也可以用 offsetTop 等方法

```javascript
handleScroll() {
  let dom = document.querySelectorAll(".lighthigh-follow"),
    clientH =
      document.documentElement.clientHeight || document.body.clientHeight;
  if (dom.length > 2) {
    dom.forEach((item, index) => {
      if (index > 1) {
        let clientRectTop = item.getBoundingClientRect().top; // 内容区的top 距离窗口的高度;
        if (clientRectTop <= clientH * 0.4) {
          this.currentLi = `#${item.id}`;
        }
      }
    });
  }
}
```
