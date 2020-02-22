## 项目框架

### 项目目录结构

```
mallProject/
├── dist/  ==>打包输出目录
├── public/
│   ├── favicon.ico  ==>icon图标
│   └── index.html  ==>入口HTML模版
├── src/
│   ├── assets/  ==>资源
│   │   ├── css/
│   │   └── img/
│   ├── common/  ==>公共代码
│   │   ├── const.js
│   │   ├── mixin.js
│   │   └── utils.js
│   ├── components/  ==>公共组件
│   │   ├── common  ==>通用公共组件
│   │   └── content  ==>业务公共组件
│   ├── network/  ==>网络请求
│   ├── router/  ==>vue-router路由
│   ├── store/  ==>vuex状态管理
│   ├── views/  ==>页面组件
│   ├── App.vue
│   └── main.js  ==>入口模块
├── .browserslistrc
├── .editorconfig  ==>编码风格
├── babel.config.js
├── package-lock.json
├── package.json
├── postcss.config.js
├── README.md
└── vue.config.js  ==>配置文件
```

### .editorconfig

.editorconfig用来统一项目在不同编辑器下的编码风格。

```
# 告诉EditorConfig插件，这是根文件，不用继续往上查找
root = true

# 匹配全部文件
[*]
# 字符集
charset = utf-8
# 缩进风格，可选"space"、"tab"
indent_style = space
# 缩进的空格数
indent_size = 2
# 结尾换行符，可选"lf"、"cr"、"crlf"
end_of_line = lf
# 在文件结尾插入新行
insert_final_newline = true
# 删除一行中的前后空格
trim_trailing_whitespace = true
```

### 配置路径别名

```js
// vue.config.js
module.exports = {
  configureWebpack: {
    resolve: {
      alias: {
        // "@": "src",默认已经提供了更该路径别名
        assets: "@/assets",
        common: "@/common",
        img: "@/assets/img",
        css: "@/assets/css",
        components: "@/components",
        views: "@/views",
        network: "@/network"
      }
    }
  }
}
```

### 项目基础CSS样式

1. 创建base.css：

```css
/* src/assets/css/base.css */
/* 引入https://github.com/necolas/normalize.css */
@import "./normalize.css";

/*:root，获取根元素html*/
:root {
  /* 定义了一些变量 */
  --color-text: #666;
  --color-high-text: #ff5777;
  --color-tint: #ff8198;
  --color-background: #fff;
  --font-size: 14px;
  --line-height: 1.5;
}

*,
*::before,
*::after {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: "Helvetica Neue", Helvetica, "PingFang SC", "Hiragino Sans GB",
    "Microsoft YaHei", "微软雅黑", Arial, sans-serif;
  user-select: none; /* 禁止用户鼠标在页面上选中文字/图片等 */
  -webkit-tap-highlight-color: transparent; /* webkit是苹果浏览器引擎，tap点击，highlight背景高亮，color颜色，颜色用数值调节 */
  background: var(--color-background);
  color: var(--color-text);
  /* rem vw/vh */
  width: 100vw;
}

a {
  color: var(--color-text);
  text-decoration: none;
}

.clear-fix::after {
  clear: both;
  content: "";
  display: block;
  width: 0;
  height: 0;
  visibility: hidden;
}

.clear-fix {
  zoom: 1;
}

.left {
  float: left;
}

.right {
  float: right;
}
```

2. 在App.vue中引入base.css

```vue
<style lang="css">
/* 在HTML、CSS中，用~来标记，这样loader才能识别这里使用的是路径别名 */
@import "~css/base.css";
</style>
```
### 网络请求架构

> **基础请求模块**

```js
// src\network\request.js
import axios from "axios";

export const request = config => {
  const instance = axios.create({
    baseURL: "http://123.207.32.32:8000/",
    timeout: 3000
  });
  instance.interceptors.response.use(
    res => res.data, // 使用拦截器，将response响应中的data取出，返回给调用者
    err => console.log(err)
  );
  return instance(config);
};
```

> **业务请求模块**

1. 将每个页面的请求抽取为单独的请求模块，降低耦合。

```js
//src\network\home.js
import { request } from "network/request";

export const getHomeMultiData = params => {
  return request({
    url: "/home/multidata"
  });
};
```

2. 在页面组件中引入请求模块，并使用它来请求数据：

```vue
<!-- src\views\home\Home.vue -->
<script>
import { getHomeMultiData } from "network/home";
export default {
  data() {
    return {
      banner: [],
      recommend: []
    };
  },
  created() {
    getHomeMultiData().then(res => {
      this.banner = res.data.banner.list;
      this.recommend = res.data.recommend.list;
    });
  }
};
</script>
```

### 页面模型

> **Home**

![image-20200220134807256](Vue%E5%95%86%E5%9F%8E%E9%A1%B9%E7%9B%AE.assets/image-20200220134807256.png)

![image-20200220135837103](Vue%E5%95%86%E5%9F%8E%E9%A1%B9%E7%9B%AE.assets/image-20200220135837103.png)

## 通用组件

### TabBar

> **通用TabBar组件**

1. TabBar父容器

TabBar是用来包裹TabBarItem的容器。

```vue
<!-- src\components\common\tabBar\TabBar.vue -->
<template>
  <div id="tab-bar">
    <slot></slot>
  </div>
</template>

<script>
// export default {};
</script>

<style lang='less' scoped>
@tabBarBgColor: #f6f6f6;

#tab-bar {
  display: flex;
  flex-direction: row;
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  background: @tabBarBgColor;
  box-shadow: 0px -2px 2px 0px rgba(0, 0, 0, 0.1);
}
</style>
```

2. TabBarItem子组件

```vue
<!-- src\components\common\tabBar\TabBarItem.vue -->
<template>
  <div class="tab-bar-item" @click="itemClicked">
    <!-- <slot>插槽会被插槽内容替换掉，所以不要在<slot>上绑定属性，如class。应该用div将其包裹 -->
    <div v-if="isActive">
      <!-- 激活状态下显示的图标 -->
      <slot name="imgActived"></slot>
    </div>
    <div v-else>
      <!-- 非激活状态下显示的图标 -->
      <slot name="imgDeactived"></slot>
    </div>
    <div :style="activeStyle">
      <!-- 标题 -->
      <slot name="title"></slot>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    link: String, //当前TabBarItem关联的导航地址
    activeColor: { //TabBarItem激活时标题字体颜色
      type: String,
      default: "red"
    }
  },
  computed: {
    // 计算是否处于激活状态
    isActive() {// 判断当前路由记录的path是否包含当前TabBarItem中的link
      return this.$route.path.indexOf(this.link) !== -1;
    },
    // 计算标题颜色样式
    activeStyle() {
      return this.isActive ? { color: this.activeColor } : {};
    }
  },
  methods: {
    itemClicked() {
      if (this.isActive) return; // 当前页面已经激活了，不必再执行跳转
      this.$router.replace(this.link);
    }
  }
};
</script>

<style lang='less' scoped>
@tabBarHeight: 49px;
.tab-bar-item {
  flex-grow: 1;
  text-align: center;
  height: @tabBarHeight;
  img {
    width: 24px;
    height: 24px;
  }
}
</style>
```

> **业务TabBar组件**

针对当前业务实现的MainTabBar，其结构如下：

```
+--------------------------------+
|  MainTabBar                    |
| -----------------------------+ |
| |  TabBar                    | |
| |  +----------------------+  | |
| |  |  TabBarItem          |  | |
| |  |  +----------------+  |  | |
| |  |  |  imgDeactived  |  |  | |
| |  |  +----------------+  |  | |
| |  |  +----------------+  |  | |
| |  |  |  imgActived    |  |  | |
| |  |  +----------------+  |  | |
| |  |  +----------------+  |  | |
| |  |  |  title         |  |  | |
| |  |  +----------------+  |  | |
| |  +----------------------+  | |
| |                            | |
| |  +----------------------+  | |
| |  |  TabBarItem          |  | |
| |  +----------------------+  | |
| |                            | |
| |  +----------------------+  | |
| |  |  TabBarItem          |  | |
| |  +----------------------+  | |
| |                            | |
| |  +----------------------+  | |
| |  |  TabBarItem          |  | |
| |  +----------------------+  | |
| +----------------------------+ |
+--------------------------------+
```

```vue
<!-- src\components\content\mainTabBar\MainTabBar.vue -->
<template>
  <TabBar>
    <TabBarItem link="/home">
      <template v-slot:imgDeactived>
        <img src="~img/tabbar/home.svg" />
      </template>
      <template v-slot:imgActived>
        <img src="~img/tabbar/home_active.svg" />
      </template>
      <template v-slot:title>
        <div>首页</div>
      </template>
    </TabBarItem>
    <TabBarItem link="/cart">
      <template v-slot:imgDeactived>
        <img src="~img/tabbar/category.svg" />
      </template>
      <template v-slot:imgActived>
        <img src="~img/tabbar/category_active.svg" />
      </template>
      <template v-slot:title>
        <div>分类</div>
      </template>
    </TabBarItem>
    <TabBarItem link="/profile">
      <template v-slot:imgDeactived>
        <img src="~img/tabbar/shopcart.svg" />
      </template>
      <template v-slot:imgActived>
        <img src="~img/tabbar/shopcart_active.svg" />
      </template>
      <template v-slot:title>
        <div>购物车</div>
      </template>
    </TabBarItem>
    <TabBarItem link="/setting">
      <template v-slot:imgDeactived>
        <img src="~img/tabbar/profile.svg" />
      </template>
      <template v-slot:imgActived>
        <img src="~img/tabbar/profile_active.svg" />
      </template>
      <template v-slot:title>
        <div>我的</div>
      </template>
    </TabBarItem>
  </TabBar>
</template>

<script>
import TabBar from "components/common/tabBar/TabBar.vue";
import TabBarItem from "components/common/tabBar/TabBarItem.vue";
export default {
  components: {
    TabBar,
    TabBarItem
  }
};
</script>
<style lang='css' scoped>
</style>
```

> **引入TabBar组件**

```
<!-- src\App.vue -->
<template>
  <div id="app">
    <router-view></router-view>
    <MainTabBar></MainTabBar>
  </div>
</template>

<script>
import MainTabBar from "components/content/mainTabBar/MainTabBar";
export default {
  components: {
    MainTabBar
  }
};
</script>
  
<style lang="css">
@import "~css/base.css";
</style>
```

App.vue的结构如下：

```
+---------------------------+
| +-----------------------+ |
| |                       | |
| |                       | |
| |                       | |
| |     router view       | |
| |                       | |
| |                       | |
| |                       | |
| |                       | |
| +-----------------------+ |
| +-----------------------+ |
| |     MainTabBar        | |
| |                       | |
| +-----------------------+ |
+---------------------------+
```

### NavBar

> **通用NavBar组件**

NavBar的结构如下：

```
+--------+-----------------+---------+
|        |                 |         |
|  left  |     center      |  right  |
|        |                 |         |
+--------+-----------------+---------+
```

```vue
<!-- src\components\common\navbar\NavBar.vue -->
<template>
  <div class="nav-bar">
    <div class="left">
      <slot name="left"></slot>
    </div>
    <div class="center">
      <slot name="center"></slot>
    </div>
    <div class="right">
      <slot name="right"></slot>
    </div>
  </div>
</template>

<script>
</script>

<style lang='css' scoped>
.nav-bar {
  display: flex;
  height: 44px;
  line-height: 44px;
  text-align: center;
}
.left,
.right {
  width: 60px;
}
.center {
  flex: 1;
}
</style>
```

> **引入NavBar组件**

```vue
<!-- src\views\home\Home.vue -->
<template>
  <div>
    <NavBar class="home-nav-bar">
      <template v-slot:center>
        <div>购物街</div>
      </template>
    </NavBar>
  </div>
</template>
```

PS：通常NavBar的高度为44px。

### 轮播图

> **通用轮播图组件**

轮播图单页面组件：

```vue
<!-- src\components\common\swiper\SwiperItem.vue -->
<template>
  <div class="slide">
    <slot></slot>
  </div>
</template>

<script>
	export default {
		name: "Slide"
	}
</script>

<style scoped>
  .slide {
    width: 100%;
    flex-shrink: 0;
  }

  .slide img {
    width: 100%;
  }
</style>
```

轮播图父容器组件：

```vue
<!-- src\components\common\swiper\Swiper.vue -->
<template>
  <div id="hy-swiper">
    <div class="swiper" @touchstart="touchStart" @touchmove="touchMove" @touchend="touchEnd">
      <slot></slot>
    </div>
    <slot name="indicator"></slot>
    <div class="indicator">
      <slot name="indicator" v-if="showIndicator && slideCount>1">
        <div
          v-for="(item, index) in slideCount"
          class="indi-item"
          :class="{active: index === currentIndex-1}"
          :key="index"
        ></div>
      </slot>
    </div>
  </div>
</template>

<script>
export default {
  name: "Swiper",
  props: {
    interval: {
      type: Number,
      default: 3000
    },
    animDuration: {
      //轮播图延时开启
      type: Number,
      default: 300
    },
    moveRatio: {
      //拖动图片多少比例，就会触发移动到下一页
      type: Number,
      default: 0.25
    },
    showIndicator: {
      type: Boolean,
      default: true
    }
  },
  data: function() {
    return {
      slideCount: 0, // 元素个数
      totalWidth: 0, // swiper的宽度
      swiperStyle: {}, // swiper样式
      currentIndex: 1, // 当前的index
      scrolling: false // 是否正在滚动
    };
  },
  mounted: function() {
    // 1.操作DOM, 在前后添加Slide
    setTimeout(() => {
      this.handleDom();

      // 2.开启定时器
      this.startTimer();
    }, 100);
  },
  methods: {
    /**
     * 定时器操作
     */
    startTimer: function() {
      this.playTimer = window.setInterval(() => {
        this.currentIndex++;
        this.scrollContent(-this.currentIndex * this.totalWidth);
      }, this.interval);
    },
    stopTimer: function() {
      window.clearInterval(this.playTimer);
    },

    /**
     * 滚动到正确的位置
     */
    scrollContent: function(currentPosition) {
      // 0.设置正在滚动
      this.scrolling = true;

      // 1.开始滚动动画
      this.swiperStyle.transition = "transform " + this.animDuration + "ms";
      this.setTransform(currentPosition);

      // 2.判断滚动到的位置
      this.checkPosition();

      // 4.滚动完成
      this.scrolling = false;
    },

    /**
     * 校验正确的位置
     */
    checkPosition: function() {
      window.setTimeout(() => {
        // 1.校验正确的位置
        this.swiperStyle.transition = "0ms";
        if (this.currentIndex >= this.slideCount + 1) {
          this.currentIndex = 1;
          this.setTransform(-this.currentIndex * this.totalWidth);
        } else if (this.currentIndex <= 0) {
          this.currentIndex = this.slideCount;
          this.setTransform(-this.currentIndex * this.totalWidth);
        }

        // 2.结束移动后的回调
        this.$emit("transitionEnd", this.currentIndex - 1);
      }, this.animDuration);
    },

    /**
     * 设置滚动的位置
     */
    setTransform: function(position) {
      this.swiperStyle.transform = `translate3d(${position}px, 0, 0)`;
      this.swiperStyle[
        "-webkit-transform"
      ] = `translate3d(${position}px), 0, 0`;
      this.swiperStyle["-ms-transform"] = `translate3d(${position}px), 0, 0`;
    },

    /**
     * 操作DOM, 在DOM前后添加Slide
     */
    handleDom: function() {
      // 1.获取要操作的元素
      let swiperEl = document.querySelector(".swiper");
      let slidesEls = swiperEl.getElementsByClassName("slide");

      // 2.保存个数
      this.slideCount = slidesEls.length;

      // 3.如果大于1个, 那么在前后分别添加一个slide
      if (this.slideCount > 1) {
        let cloneFirst = slidesEls[0].cloneNode(true);
        let cloneLast = slidesEls[this.slideCount - 1].cloneNode(true);
        swiperEl.insertBefore(cloneLast, slidesEls[0]);
        swiperEl.appendChild(cloneFirst);
        this.totalWidth = swiperEl.offsetWidth;
        this.swiperStyle = swiperEl.style;
      }

      // 4.让swiper元素, 显示第一个(目前是显示前面添加的最后一个元素)
      this.setTransform(-this.totalWidth);
    },

    /**
     * 拖动事件的处理
     */
    touchStart: function(e) {
      // 1.如果正在滚动, 不可以拖动
      if (this.scrolling) return;

      // 2.停止定时器
      this.stopTimer();

      // 3.保存开始滚动的位置
      this.startX = e.touches[0].pageX;
    },

    touchMove: function(e) {
      // 1.计算出用户拖动的距离
      this.currentX = e.touches[0].pageX;
      this.distance = this.currentX - this.startX;
      let currentPosition = -this.currentIndex * this.totalWidth;
      let moveDistance = this.distance + currentPosition;

      // 2.设置当前的位置
      this.setTransform(moveDistance);
    },

    touchEnd: function(e) {
      // 1.获取移动的距离
      let currentMove = Math.abs(this.distance);

      // 2.判断最终的距离
      if (this.distance === 0) {
        return;
      } else if (
        this.distance > 0 &&
        currentMove > this.totalWidth * this.moveRatio
      ) {
        // 右边移动超过0.5
        this.currentIndex--;
      } else if (
        this.distance < 0 &&
        currentMove > this.totalWidth * this.moveRatio
      ) {
        // 向左移动超过0.5
        this.currentIndex++;
      }

      // 3.移动到正确的位置
      this.scrollContent(-this.currentIndex * this.totalWidth);

      // 4.移动完成后重新开启定时器
      this.startTimer();
    },

    /**
     * 控制上一个, 下一个
     */
    previous: function() {
      this.changeItem(-1);
    },

    next: function() {
      this.changeItem(1);
    },

    changeItem: function(num) {
      // 1.移除定时器
      this.stopTimer();

      // 2.修改index和位置
      this.currentIndex += num;
      this.scrollContent(-this.currentIndex * this.totalWidth);

      // 3.添加定时器
      this.startTimer();
    }
  }
};
</script>

<style scoped>
#hy-swiper {
  overflow: hidden;
  position: relative;
}

.swiper {
  display: flex;
}

.indicator {
  display: flex;
  justify-content: center;
  position: absolute;
  width: 100%;
  bottom: 8px;
}

.indi-item {
  box-sizing: border-box;
  width: 8px;
  height: 8px;
  border-radius: 4px;
  background-color: #fff;
  line-height: 8px;
  text-align: center;
  font-size: 12px;
  margin: 0 5px;
}

.indi-item.active {
  background-color: rgba(212, 62, 46, 1);
}
</style>
```

轮播图导出模块：

```js
// src\components\common\swiper\index.js
import Swiper from './Swiper'
import SwiperItem from './SwiperItem'

export {
  Swiper, SwiperItem
}
```

> **使用轮播图组件**

业务轮播图组件：

```vue
<!-- src\views\home\child-components\HomeSwiper.vue -->
<template>
  <div class>
    <Swiper>
      <SwiperItem v-for="(item, index) in banners" :key="index">
        <a :href="item.link">
          <img :src="item.image" alt />
        </a>
      </SwiperItem>
    </Swiper>
  </div>
</template>

<script>
import { Swiper, SwiperItem } from "components/common/swiper";
export default {
  name: "HomeSwiper",
  components: {
    Swiper,
    SwiperItem
  },
  props: {
    banners: {
      type: Array,
      default() {
        return [];
      }
    }
  }
};
</script>
```

在Home页面中引入HomeSwiper组件：

```vue
<!-- src\views\home\Home.vue -->
<template>
  <div>
    <!-- 导航栏 -->
    <NavBar class="home-nav-bar">
      <template v-slot:center>
        <div>购物街</div>
      </template>
    </NavBar>
    <!-- 轮播图 -->
    <HomeSwiper :banners="banners"></HomeSwiper>
  </div>
</template>
<script>
import NavBar from "components/common/nav-bar/NavBar";
import HomeSwiper from "./child-components/HomeSwiper";
import { getHomeMultiData } from "network/home";
export default {
  components: {
    NavBar,
    HomeSwiper
  },
  data() {
    return {
      banners: [],
      recommends: []
    };
  },
  created() {
    getHomeMultiData().then(res => {
      this.banners = res.data.banner.list;
      this.recommends = res.data.recommend.list;
    });
  }
};
</script>
```

### 滚动组件better-scroll

#### 概述

better-scroll是一个用来解决移动端（已支持 PC）滚动需求的库。

> **原理**

浏览器默认的滚动原理是：当页面内容超过视口高度/宽度时，将出现滚动效果。

也可以通过CSS来设置一个滚动效果，要求子元素的高度必须超过父容器的高度：

```html
<body>
<div id="wrapper">
  <ul>
    <li>列表1</li>
    <li>列表2</li>
    <li>列表3</li>
    ...
    <li>列表98</li>
    <li>列表99</li>
    <li>列表100</li>
  </ul>
</div>

<style>
  #wrapper {
    height: 300px;
    overflow-y: scroll;
  }
</style>
```

better-scroll的原理与之类似，但从底层来说，better-scroll是通过不断修改content内容元素的transition过渡效果来实现滚动的：

![原理图](Vue%E5%95%86%E5%9F%8E%E9%A1%B9%E7%9B%AE.assets/schematic.png)

better-scroll需要注意以下几点：
- 父容器wrapper有固定高度；
- content必须是父容器内的第一个子元素，其他的子元素不会被处理；
- 只有当content的高度超过父容器高度时，才有滚动效果。

> 安装与基本使用

better-scroll包含core核心库与拓展插件库，其他拓展功能可以按需引入。

1. 安装：`npm install @better-scroll/core@next --save`
2. 基本使用

```
import BScroll from '@better-scroll/core'
let bs = new BScroll('.wrapper', {
  // ...... 详见配置项
})
```

为了保证BScroll在内存中始终存在，不至于在创建后就弹栈，通常要将其挂载到window对象或Vue实例上。

详细的使用和配置信息参考[官方文档](https://better-scroll.github.io/docs/zh-CN/)。

#### 封装better-scroll

> **将better-scroll封装为Scroller组件**

```vue
<!-- src\components\common\scroller\Scroller.vue -->
<template>
  <!-- wrapper -->
  <div ref="wrapper">
    <!-- content必须为第一个子元素 -->
    <div>
      <slot></slot>
    </div>
  </div>
</template>

<script>
import BScroll from "@better-scroll/core";
// observe-dom插件用来开启对 scroll 区域 DOM 改变的探测。当 scroll 的 dom 元素发生时，将自动触发 refresh 方法。
import ObserveDOM from "@better-scroll/observe-dom";

BScroll.use(ObserveDOM);

export default {
  props: {
    scrollY: {
      type: Boolean,
      default: true
    }
  },
  mounted() {
    this.initBscroll();
  },
  methods: {
    // 初始化BetterScroll
    initBscroll() {
      // 创建BScroll示例，并绑定掉vue组件实例上。如果不绑定的话，BScroll实例将随着方法调用的结束而弹栈
      this.scroll = new BScroll(this.$refs.wrapper, {
        observeDOM: true, // 开启对DOM改变的探测，自动refresh
        click: true, //派发scroller内的子元素的点击事件
        scrollY: this.scrollY
      });
    },
  }
};
</script>
```

> **在Home页面中使用Scroller**

```vue
<template>
  <div id="home">
    <!-- 导航栏 -->
    <NavBar class="home-nav-bar"></NavBar>
    <!-- 用Scroller包裹想要使其滑动的内容 -->
    <Scroller id="scroller">
    	<!-- Scroller插槽内容，以下元素将作为Scroller的Content -->
      <HomeSwiper :banners="banners"></HomeSwiper>
      <RecommendView :recommends="recommends"></RecommendView>
      <FeatureView></FeatureView>
      <TabControl></TabControl>
      <GoodsList :goods="currentGoodsList"></GoodsList>
    </Scroller>
  </div>
</template>
```

这里有一个关键点，要想Scroller的滚动生效，必须给Scoller设置固定的高度，如何计算出这个高度？这里使用position+top/bottom来自动计算出Scroller的高度：

```vue
<style lang='css' scoped>
#home {
  position: relative;
  /* vh表示视口高度，这里让Home页面的高度固定为100%视口高度 */
  height: 100vh;
}

#scroller {
  overflow: hidden;
  position: absolute;
  /* 预留出NavBar的高度 */
  top: 44px;
  bottom: 0;
  left: 0;
  right: 0;
}
</style>
```

#### 增添BackTop

需求：当Scroller向下滚动一定距离后，在页面右下角显示一个BackTop按钮，点击后回滚页面顶部。

> **修改Scroller组件**

要实现通过Scroller滚动来控制BackTop的显示与隐藏，就需要监听Scroller的滚动事件。

```vue
<template>
  <div ref="wrapper">
    <div>
      <slot></slot>
    </div>
  </div>
</template>

<script>
import BScroll from "@better-scroll/core";
import ObserveDOM from "@better-scroll/observe-dom";
BScroll.use(ObserveDOM);

export default {
  props: {
    scrollY: {
      type: Boolean,
      default: true
    },
    probeType: {
      /**
       *  在什么情况下派发scroll事件
       *  - 0表示不派发scroll事件；
       *  - 1表示非实时（屏幕滑动超过一定时间后）派发scroll 事件
       *  - 2表示在屏幕滑动的过程中实时派发
       *  - 3表示不仅在屏幕滑动的过程中，而且在 momentum 滚动动画运行过程中实时派发
       */
      type: Number,
      default: 0
    }
  },
  mounted() {
    this.initBscroll();
  },
  methods: {
    initBscroll() {
      this.scroll = new BScroll(this.$refs.wrapper, {
        observeDOM: true,
        click: true,
        scrollY: this.scrollY,
        pullUpLoad: this.pullUpLoad,
        probeType: this.probeType
      });
      // 监听scroll事件，pos是当前滚动的位置
      if (this.probeType !== 0) {
        this.scroll.on("scroll", pos => {
          this.$emit("scroll", pos);
        });
      }
    },

    /**
     * @description: 滚动到指定的位置
     * @param {Number}x x坐标
     * @param {Number}y y坐标
     * @param {Number}time　滚动时间
     */
    scrollTo(x, y, time = 500) {
      this.scroll.scrollTo(x, y, time);
    }
  }
};
</script>
```

> **在Home.vue中加入BackTop**

```vue
<!-- src\views\home\Home.vue -->
<template>
  <div id="home">
    <NavBar class="home-nav-bar"></NavBar>
    <Scroller id="scroller" ref="scroller" :probeType="3" @scroll="onScroll">
      ...
    </Scroller>
    <BackTop @click.native="onBackTop" v-show="isBackTopShowing"></BackTop>
  </div>
</template>

<script>
...
methods: {
  onBackTop() {
    // 让<Scroller>滚到顶部
    this.$refs.scroller.scrollTo(0, 0);
  },
  onScroll(pos) {
    // 当<Scroller>滑动超过某个阈值，则隐藏<BackTop>
    this.isBackTopShowing = pos.y < -1000;
  },
}
</script>
```

技术点：

- 使用@scroll="onScroll"来实时监听Scroller滚动到的位置；
- 通过@click.native="onBackTop"来绑定BackTop组件自身的点击时间；
- 使用元素的ref属性和vm.$refs.xxx来获取Scroller组件实例，然后就可以调用该组件内的scrollTo方法；

#### 上拉加载更多

需求：当Scroller向上拉到底部时，自动加载更多的GoodsList。

> **修改Scroller组件**

```vue
<script>
import BScroll from "@better-scroll/core";
// pull-up是用来实现监听上拉事件的插件
import Pullup from "@better-scroll/pull-up";
import ObserveDOM from "@better-scroll/observe-dom";
BScroll.use(Pullup);
BScroll.use(ObserveDOM);

export default {
  props: {
    scrollY: {
      type: Boolean,
      default: true
    },
    probeType: {
      // 在什么情况下派发scroll事件
      type: Number,
      default: 0
    },
    pullUpLoad: {
      // 是否监听上拉动作
      type: Boolean | Object,
      default: false
    }
  },
  mounted() {
    this.initBscroll();
  },
  methods: {
    // 初始化BetterScroll
    initBscroll() {
      this.scroll = new BScroll(this.$refs.wrapper, {
        observeDOM: true,
        click: true,
        scrollY: this.scrollY,
        pullUpLoad: this.pullUpLoad,
        probeType: this.probeType
      });
      // 监听scroll事件，pos是当前滚动的位置
      if (this.probeType !== 0) {
        this.scroll.on("scroll", pos => {
          this.$emit("scroll", pos);
        });
      }

      // 监听pullingUp上拉动作事件
      if (this.pullUpLoad) {
        this.scroll.on("pullingUp", () => {
          // 将上拉事件发射到父组件，在父组件中完成数据加载
          this.$emit("pullingUp");
        });
      }
    },

    scrollTo(x, y, time = 500) {
      this.scroll.scrollTo(x, y, time);
    },

    /**
     * @description: 标识一次上拉加载动作结束。
     */
    finishPullUp() {
      this.scroll.finishPullUp(); //标识一次上拉加载动作结束。
      this.scroll.refresh(); //刷新scroll
    },
    /**
     * @description: 重新计算 BetterScroll，当 DOM 结构发生变化的时候务必要调用确保滚动的效果正常
     */
    refresh() {
      this.scroll.refresh();
    }
  }
};
</script>
```

> **在Home.vue中实现上拉加载更多**

```vue
<template>
  <div id="home">
    <NavBar class="home-nav-bar"></NavBar>
    <Scroller
      id="scroller"
      ref="scroller"
      :probeType="3"
      :pullUpLoad="true"
      @scroll="onScroll"
      @pullingUp="onPullingUp"
    >
      ...
      <!-- 商品列表 -->
      <GoodsList :goods="currentGoodsList"></GoodsList>
    </Scroller>
    <BackTop @click.native="onBackTop" v-show="isBackTopShowing"></BackTop>
  </div>
</template>

<script>
// 网络数据加载方法
import { getGoodsData } from "network/home";
export default {
  methods: {
    // 数据请求方法
    getGoodsData(type) {
      getGoodsData(type, this.goods[type].page + 1).then(res => {
        this.goods[type].list.push(...res.data.list);
        this.goods[type].page += 1;
        this.$refs.scroller.finishPullUp(); //标识一次上拉加载动作结束
      });
    },
    onBackTop() {
      this.$refs.scroller.scrollTo(0, 0);
    },
    onScroll(pos) {
      this.isBackTopShowing = pos.y < -1000;
    },
    onPullingUp() {
      // <Scroller>上拉加载更多数据
      this.getGoodsData(this.currentTabType);
    }
  }
};
</script>
```

#### 处理加载图片导致的bug

如果内容存在图片的情况，可能会出现 DOM 元素渲染时图片还未下载，因此内容元素的高度小于预期，出现滚动不正常的情况。此时你应该在图片加载完成后，比如 onload 事件回调中，调用 `bs.refresh` 方法，它会重新计算最新的滚动距离。

在Home页面中，GoodsList内的GoodsListItem内会加载商品图片。如何在Home页面中监听GoodsListItem组件内的图片onload事件？

```
+---------------------+
|    Home             |
| +-----------------+ |
| |  GoodsList      | |
| | +-------------+ | |
| | |GoodsListItem| | |
| | +-------------+ | |
| +-----------------+ |
+---------------------+
```

方案1：`GoodsListItem--$emit-->GoodsList--$emit-->Home`

方案2：[事件总线](http://www.imooc.com/article/289043)

1. 在main.js中注册事件总线：Vue.prototype.$EventBus = new Vue();
2. 在GoodsListItem中发射图片的onload事件

```vue
<template>
  <div class="goods-list-item">
    <img :src="goodItem.imglink" @load="onLoad" />
    <div>{{goodItem.title}}</div>
  </div>
</template>

<script>
export default {
  name: "GoodsListItem",
  props: {
    goodItem: {
      type: Object,
      default() {
        return {};
      }
    }
  },
  methods: {
    onLoad() {
      this.$EventBus.$emit("goods-item-img-load");
    }
  }
};
</script>
```

3. 在Home.vue中监听并处理onload事件

```vue
mounted() {
  this.scrollerRefresh = this.$refs.scroller.refresh;
  // 注册goods-item-img-load监听事件
  // 在<GoodsList>中的图片加载成功后，刷新scroller
  this.$EventBus.$on("goods-item-img-load", this.scrollerRefresh);
},
activated() {
  // 当路由回到Home页面时，复原<Scroller>的位置
  this.$refs.scroller.scrollTo(0, this.currentScrollerY, 0);
  this.$refs.scroller.refresh();
  this.$EventBus.$on("goods-item-img-load", this.scrollerRefresh);
},
deactivated() {
  // 当路由到其他页面时，记录<Scroller>的位置
  this.currentScrollerY = this.$refs.scroller.getY();
  // 当跳转到其他Tab标签页时，为了防止Home页面仍接收到goods-item-img-load事件，这里应该取消监听
  this.$EventBus.$off("goods-item-img-load", this.scrollerRefresh);
},
destroyed() {
  this.$EventBus.$off("goods-item-img-load", this.scrollerRefresh);
}
```

> **优化：对图片加载导致的refresh进行防抖处理**

上拉加载会加载一匹新的商品图片，每个图片加载成功后都refresh Scroller，显然是没必要的。这里我们进行防抖处理：

1. 在utils.js中编写防抖函数

```js
/**
 * @description: 防抖动函数
 * @param {Function} fn 原函数
 * @param {Number} delay 延迟时间
 * @return: 防抖动处理后的函数
 */
export const debounce = (fn, delay) => {
  let timer = null;
  return function(...args) {
    if (timer) clearTimeout(timer);
    timer = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
};
```

2. 在Home.vue中将refresh进行防抖处理

```js
import { debounce } from "common/utils";

export default {
  mounted() {
    // 防抖动处理后的refresh方法
    this.scrollerRefresh = debounce(this.$refs.scroller.refresh, 500); 
    this.$EventBus.$on("goods-item-img-load", this.scrollerRefresh);
  },
}
```

> **优化：使用mixin抽取通用逻辑**

在任何Scroller中包含图片加载的组件中，都需要在生命周期钩子中处理图片onload事件。我们可以使用mixin来抽取这些重复代码。

```js
// src\common\mixin.js
import { debounce } from "./utils";

export const imgOnLoadListenerMixin = {
  mounted() {
    this.scrollerRefresh = debounce(this.$refs.scroller.refresh, 500);
    this.$EventBus.$on("goods-item-img-load", this.scrollerRefresh);
  },
  activated() {
    this.$EventBus.$on("goods-item-img-load", this.scrollerRefresh);
  },
  deactivated() {
    this.$EventBus.$off("goods-item-img-load", this.scrollerRefresh);
  },
  destroyed() {
    this.$EventBus.$off("goods-item-img-load", this.scrollerRefresh);
  }
};
```

#### Scroller内的吸顶框

需求：当Scroller内的TabControl滚动到页面顶部时，自动吸顶。

在使用better-scroll后，页面的TabControl即使设置为`display:fixed;`也无法使其具有吸顶效果。这里的解决方案是，在页面内复制另一份TabControl，当Scroller内嵌的TabControl移动到顶部时，让吸顶TabControl显示。

```vue
<template>
  <div id="home">
    <!-- 导航栏 -->
    <NavBar class="home-nav-bar"></NavBar>
    <!-- 用于吸顶的TabControl -->
    <TabControl
      id="tab-control"
      :titles="tabLabel"
      @change-tab="onChangeTab"
      :currentIndex="currentTabIndex"
      v-show="isTabControlShowing"
    ></TabControl>
    <Scroller
      id="scroller"
      ref="scroller"
      :probeType="3"
      :pullUpLoad="true"
      @scroll="onScroll"
      @pullingUp="onPullingUp"
    >
      ...
      <!-- Scroller内嵌的TabControl -->
      <TabControl
        ref="withInTabControl"
        :titles="tabLabel"
        @change-tab="onChangeTab"
        :currentIndex="currentTabIndex"
      ></TabControl>
      <GoodsList :goods="currentGoodsList"></GoodsList>
    </Scroller>
    <BackTop @click.native="onBackTop" v-show="isBackTopShowing"></BackTop>
  </div>
</template>

<script>
  methods: {
    onScroll(pos) {
      // 当<Scroller>滑动超过某个阈值，则隐藏<BackTop>
      this.isBackTopShowing = pos.y < -1000;
      // 当<Scroller>滑动超过某个阈值，则显示吸顶<TabControl>
      this.isTabControlShowing =
        -pos.y > this.$refs.withInTabControl.$el.offsetTop; // 计算吸顶<TabControl>相对父容器的顶部偏移量
    },
  },
</script>
```

#### Scroller最终代码

```vue
<!--
在使用Scroller组件时，注意配置以下prop：
scrollY：是否纵向滚动
probeType：在什么情况下派发scroll事件：
  - 0表示不派发scroll事件；
  - 1表示非实时（屏幕滑动超过一定时间后）派发scroll 事件
  - 2表示在屏幕滑动的过程中实时派发
  - 3表示不仅在屏幕滑动的过程中，而且在 momentum 滚动动画运行过程中实时派发
pullUpLoad：是否监听上拉动作
 -->
<template>
  <div ref="wrapper">
    <div>
      <slot></slot>
    </div>
  </div>
</template>

<script>
import BScroll from "@better-scroll/core";
import Pullup from "@better-scroll/pull-up";
import ObserveDOM from "@better-scroll/observe-dom";
BScroll.use(Pullup);
BScroll.use(ObserveDOM);

export default {
  props: {
    scrollY: {
      type: Boolean,
      default: true
    },
    probeType: {
      // 在什么情况下派发scroll事件
      type: Number,
      default: 0
    },
    pullUpLoad: {
      // 是否监听上拉动作
      type: Boolean | Object,
      default: false
    }
  },
  mounted() {
    this.initBscroll();
  },
  methods: {
    // 初始化BetterScroll
    initBscroll() {
      // 创建BScroll示例，并绑定掉vue组件实例上。如果不绑定的话，BScroll实例将随着方法调用的结束而弹栈
      this.scroll = new BScroll(this.$refs.wrapper, {
        observeDOM: true, // 开启对DOM改变的探测，自动refresh
        click: true, //派发scroller内的子元素的点击事件
        scrollY: this.scrollY,
        pullUpLoad: this.pullUpLoad,
        probeType: this.probeType
      });
      // 监听scroll事件，pos是当前滚动的位置
      if (this.probeType !== 0) {
        this.scroll.on("scroll", pos => {
          this.$emit("scroll", pos);
        });
      }

      // 监听pullingUp上拉动作事件
      if (this.pullUpLoad) {
        this.scroll.on("pullingUp", () => {
          // 将上拉事件发射到父组件，在父组件中完成数据加载
          this.$emit("pullingUp");
        });
      }
    },

    /**
     * @description: 滚动到指定的位置
     * @param {Number}x x坐标
     * @param {Number}y y坐标
     * @param {Number}time　滚动时间
     * @return:
     */
    scrollTo(x, y, time = 500) {
      this.scroll.scrollTo(x, y, time);
    },

    /**
     * @description: 标识一次上拉加载动作结束。
     */
    finishPullUp() {
      this.scroll.finishPullUp(); //标识一次上拉加载动作结束。
      this.scroll.refresh(); //刷新scroll
    },
    /**
     * @description: 重新计算 BetterScroll，当 DOM 结构发生变化的时候务必要调用确保滚动的效果正常
     */
    refresh() {
      this.scroll.refresh();
    },
    /**
     * @description: 获取y轴坐标
     */
    getY() {
      return this.scroll.y;
    }
  }
};
</script>

<style lang='css' scoped>
</style>
```

### Toast

1. 封装Toast组件

```vue
<!-- src\components\common\toast\Toast.vue -->
<template>
  <div v-show="isToastShowing" class="toast">{{message}}</div>
</template>

<script>
export default {
  name: "Toast",
  data() {
    return {
      isToastShowing: false,
      message: ""
    };
  },
  methods: {
    show(msg, duration = 1500) {
      this.message = msg;
      this.isToastShowing = true;
      setTimeout(() => {
        this.message = "";
        this.isToastShowing = false;
      }, duration);
    }
  }
};
</script>

<style lang='css' scoped>
.toast {
  background: rgba(255, 0, 0, 0.507);
  padding: 10px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translateX(-50%) translateY(-50%);
  z-index: 999;
}
</style>
```

2. 创建Toast插件导入模块

```js
// src\components\common\toast\index.js
import Toast from "./Toast";

const obj = {};

// 当调用Vue.use时，会自动执行目标模块的install函数，并传入一个Vue
obj.install = function(Vue) {
  // 使用Vue.extend创建基于Toast的构造器
  const ToastConstructor = Vue.extend(Toast);
  // 创建toast实例，并将其挂载到一个div元素上
  const toast = new ToastConstructor().$mount(document.createElement("div"));
  // 将toast元素插入文档
  document.body.appendChild(toast.$el);
  // 在Vue原型上注册toast实例
  Vue.prototype.$toast = toast;
};

export default obj;
```

3. 在main.js中使用Toast插件

```js
// src\main.js
import Toast from "components/common/toast";

Vue.use(Toast);
```

4. 在其他组件中使用Toast

```js
this.$toast.show(this.message, 1500);
```

## 功能模块

### 数据整合

在一些情况下，服务器返回的数据结构复杂，或需要从多个接口来获取完整的数据。为了方便使用和组件间传递数据，通常用构造函数或class类来封装实际的业务数据。

```js
export class Goods {
  // 构造函数
  constructor(itemInfo, columns, services) {
    this.title = itemInfo.title;
    this.desc = itemInfo.desc;
    this.newprice = itemInfo.price;
    this.oldprice = itemInfo.oldprice;
    this.discount = itemInfo.discountDesc;
    this.columns = columns;
    this.services = services;
    this.realprice = itemInfo.lowlowprice;
  }
}
```

### 购物车

使用vuex来实现购物车功能。

```js
// src\store\index.js
import Vue from 'vue'
import Vuex from 'vuex'

import mutations from './mutations'
import actions from './actions'
import getters from './getters'

Vue.use(Vuex)

const state = {
  cartList: []
}

const store = new Vuex.Store({
  state,
  mutations,
  actions,
  getters
})

export default store

// src\store\mutations.js
const mutations = {
  addCart(state, info) {
    console.log(info);
    // 1.查看是否添加过
    const oldInfo = state.cartList.find(item => item.iid === info.iid)

    // 2.+1或者新添加
    if (oldInfo) {
      oldInfo.count += 1
    } else {
      info.count = 1
      info.checked = true
      state.cartList.push(info)
    }
  }
}

export default mutations
```

### fastclick

**问题产生**：在移动端浏览器上，点击事件会有300ms的延迟，即js捕获click事件的回调函数处理，需要300ms后才生效。

**问题原因**：这与双击缩放的方案有关。浏览器捕获第一次单击后，会先等待一段时间，如果在这段时间区间里用户未进行下一次点击，则浏览器会做单击事件的处理。如果这段时间里用户进行了第二次单击操作，则浏览器会做双击事件处理。这段时间就是上面提到的300毫秒延迟。

**解决方案**：[fastclick](https://github.com/ftlabs/fastclick)

1. 安装：`npm install fastclick`
2. 在main.js中使用：

```js
var attachFastClick = require('fastclick');
attachFastClick(document.body);
```

### 图片懒加载：vue-lazyload

图片懒加载，即图片在需要显示时才去加载。

[vue-lazyload](https://github.com/hilongjw/vue-lazyload)

### 移动端适配：postcss-px-to-viewport

[`postcss-px-to-viewport`](https://github.com/evrone/postcss-px-to-viewport)插件主要用来把`px`单位转换为`vw`、`vh`、`vmin`或者`vmax`这样的视窗单位。

参考阅读：https://juejin.im/entry/5aa09c3351882555602077ca