# Col 组件分析

## Gutter（间隔）的实现

实现方式：

```javaScript
gutter() {
  let parent = this.$parent;

  // 一直向上层寻找 ElRow 组件，并获取它的 gutter props
  while (parent && parent.$options.componentName !== 'ElRow') {
    parent = parent.$parent;
  }
  return parent ? parent.gutter : 0;
}

render() {
  // 如果 ElRow 存在 gutter props，则计算对应的内边距 / 2

  // 1/2内边距的原因是，相邻 col 组件的内边距叠加起来就是 gutter 的值
  if (this.gutter) {
    style.paddingLeft = this.gutter / 2 + 'px';
    style.paddingRight = style.paddingLeft;
  }
}
```

### 为何不用 margin

因为 margin 是元素与元素之间的间距，不会算在元素宽度之内，如果想让三个元素（每个元素宽计算为 100/3≈33.333333%）占满一行，由于margin的特性，元素实际会换行

## 栅格布局的实现

源码中 Col 存在 `xs, sm, md, lg, xl` 这几个 props 属性，通过这几个属性以及以前使用过的 bootstrap 栅格系统，也就可以明白，核心其实是 @media query

我之前的思考存在一个误区，一直在想着如何通过 js 的手段去实现 @media query，直到看到 element 的源码实现**（预先定义好 24 栅格的样式，通过 class 匹配对应的样式）**，才恍然大悟。

### 具体实现

```javaScript

// 遍历几个栅格 props 属性，给元素添加 el-col-xs-${宽度} 的class
['xs', 'sm', 'md', 'lg', 'xl'].forEach(size => {
  if (typeof this[size] === 'number') {
    classList.push(`el-col-${size}-${this[size]}`);
  } else if (typeof this[size] === 'object') {
    // 此处针对 xs: Object 的情况处理

    let props = this[size];
    Object.keys(props).forEach(prop => {
      classList.push(
        prop !== 'span'
          ? `el-col-${size}-${prop}-${props[prop]}`
          : `el-col-${size}-${props[prop]}`
      );
    });
  }
});
```

预先定义好的样式

```scss
// res mixin 的定义，可以看到 核心的实现之一 @media query
@mixin res($key, $map: $--breakpoints) {
  // 循环断点Map，如果存在则返回
  @if map-has-key($map, $key) {
    @media only screen and #{inspect(map-get($map, $key))} {
      @content;
    }
  } @else {
    @warn "Undefeined points: `#{$map}`";
  }
}

// 这里使用了 res mixin，提高复用性
@include res(xs) {
  .el-col-xs-0 {
    display: none;
  }
  @for $i from 0 through 24 {
    .el-col-xs-#{$i} {
      width: (1 / 24 * $i * 100) * 1%;
    }

    .el-col-xs-offset-#{$i} {
      margin-left: (1 / 24 * $i * 100) * 1%;
    }

    .el-col-xs-pull-#{$i} {
      position: relative;
      right: (1 / 24 * $i * 100) * 1%;
    }

    .el-col-xs-push-#{$i} {
      position: relative;
      left: (1 / 24 * $i * 100) * 1%;
    }
  }
}

@include res(sm) {
  .el-col-sm-0 {
    display: none;
  }
  @for $i from 0 through 24 {
    .el-col-sm-#{$i} {
      width: (1 / 24 * $i * 100) * 1%;
    }

    .el-col-sm-offset-#{$i} {
      margin-left: (1 / 24 * $i * 100) * 1%;
    }

    .el-col-sm-pull-#{$i} {
      position: relative;
      right: (1 / 24 * $i * 100) * 1%;
    }

    .el-col-sm-push-#{$i} {
      position: relative;
      left: (1 / 24 * $i * 100) * 1%;
    }
  }
}

```

## 最终使用效果

```html
<el-row :gutter="20" :lg="4" :xl="5">
  <el-col></el-col>
  <el-col></el-col>
  <el-col></el-col>
  <el-col></el-col>
</el-row>
```

## 关于 gutter 的缺点

element 的这套实现也不是没有问题的，如果是以下的情况，就会存在宽度溢出的问题

```html
<html>
  <body>
    <!-- 会因为 margin-left/right: -10px 产生宽度溢出，出现横向滚动条，可以使用 overflow: hidden 解决这个问题 -->
    <div style="width: 100%;">
      <el-row :gutter="20">
        <el-col></el-col>
        <el-col></el-col>
        <el-col></el-col>
      </el-row>
    </div>
  </body>
</html>
```
