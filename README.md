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

**分析如何提取 h 标签**

如提取文档

```html
<div>
  <h1> 1. xxx </h1>
  <h2> 1.1 xxx </h2>
  <p> 1.1的正文 </p>
  <h2> 1.2 xxx </h2>
  <p> 1.2的正文 </p>
  <h1> 2. xxx </h1>
  <h2> 2.1 xxx </h2>
  <p> 2.1的正文 </p>
</div>
```

若直接使用`document.querySelectorAll("Hx")` (x 表示 1~6), 这样确实能筛选出正文的全部 H 标签，但是提取结果却是['1', '2']，['1.1', '1.2', '2.1'], 显然这并不是我想要的结果，那怎么才能不打乱层级关系呢？

通过观察得知，h1和h2标签虽然有层级关系，但是两个标签之间实际上是「平铺」关系，所谓平铺关系，即h1不会嵌套h2, 也就是说不会出现`<h1><h2></h2></h1>`这样的情况, 既然是「平铺」关系, 那么就可以将其看作一条队列, 我们只需要从队首到队尾依次提取dom节点即可得知h1和h2的层级关系

所以这里应该采用 `document.querySelectorAll("*")` 提取, 依次遍历该队列并记录下h1~h6的节点，最终得到这样一个数组['1', '1.1', '1.2', '2', '2.1']，通过这个数组即可得到标题的层级关系

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

**锚点定位和滚动高亮的解法**有多种，解法也差不多，这里大概分析以下几种解法：

解法1，通过 *offsetTop* 来获取dom节点的位置

但是这种方法需要计算一大堆offsetTop和scrollTop，很麻烦，而且其实已经有现成的接口可以直接得出结果（就是getBoundingClientRect）

解法2，通过 *getBoundingClientRect* 来获取dom节点和当前可视区域的位置关系

以上两个解法都必须监听好几个事件（如：scroll，resize），甚至假如因为某个动画或操作将某个dom的高度发生变化甚至是添加删除，导致dom的位置发生了变化，如此一来，监听的事件就成几何倍数增长了，且无论怎么做都还是有遗漏的可能性

那怎么解决呢？

都2021了，你真的不打算尝试一下新的黑科技（Intersection Observer）吗？

解法3，通过 *Intersection Observer* 实现dom的观察者模式来得知dom的位置变化，不过这个api比较新，所以考虑业务场景的话，尤其是pc端需要兼容一下（可参考polyfill）

最后顺手贴下代码。。。

```javascript
lightHigh(item) {
  this.currentLi = item.href;
  let offsetTop = document.querySelector(item.href).offsetTop,
    clientH =
      document.documentElement.clientHeight || document.body.clientHeight;
  document.documentElement.scrollTop = offsetTop - clientH * 0.3; // 锚点定位
}
```

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
