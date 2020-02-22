### Flex父元素被子元素撑开

需求：使用flex来使多个子元素水平排列，且自适应：

```html
<body>
  <div id="father">
    <div class="son">
      <img src="./pkq.jpg" alt="" />
      <div>Pokemon</div>
    </div>
    <div class="son">
      <img src="./pkq.jpg" alt="" />
      <div>Pokemon</div>
    </div>
    <div class="son">
      <img src="./pkq.jpg" alt="" />
      <div>Pokemon</div>
    </div>
  </div>
  <style>
    #father {
      display: flex;
    }
    .son {
      flex-direction: row;
      flex-grow: 1;
    }
  </style>
</body>
```

但效果如下：

![](CSS%E7%A4%BA%E4%BE%8B.assets/image-20200217194018794.png)

子元素虽然水平排列了，但它们将父元素撑开，而不是根据父元素自适应。

解决方案：

```html
#father {
  display: flex;
}
.son {
  flex-direction: row;
  flex-grow: 1;
  width: 0;
}
```

设置子元素（.son）的宽度为0，这样就使得子元素（.son）根据flex-grow来分配父元素的宽度。效果如下：

![](CSS%E7%A4%BA%E4%BE%8B.assets/image-20200217194517724.png)

虽然子元素（.son）实现类自适应，但子元素内的`<img>`没有`<div class="son">`自适应。这是因为，**flex特性仅限于该父容器的直接子元素，孙子元素默认不继承**。我们可以设置`<img>`的宽度为100%，使`<img>`与`<div class="son">`保持相同的宽度。

最终方案：

```css
#father {
  display: flex;
}
.son {
  flex-direction: row;
  flex-grow: 1;
  width: 0;
}
.son img {
  width: 100%;
}
```

最终效果：

![](CSS%E7%A4%BA%E4%BE%8B.assets/image-20200217195013472.png)

### 滚动容器

需求：指定一个父容器，当内部的子元素高度超出父容器时，使父容器可以滚动来查看所有子元素。

> **原生方案**

```html
<body>
  <div class="father" style="background-color: red;">
    <ul>
      <li>列表1</li>
      <li>列表2</li>
      <li>列表3</li>
			...
      <li>列表99</li>
      <li>列表100</li>
    </ul>
  </div>

  <style>
    .father {
      height: 150px;
      overflow: hidden;
      overflow-y: scroll;
    }
  </style>
</body>
```

> **better-scroll**

https://better-scroll.github.io/docs/zh-CN/

支持移动端、PC端的滚动效果。且具有各种高级特性，如下拉加载、轮播图、wheel picker等。

