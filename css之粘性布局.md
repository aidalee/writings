## 什么是粘性布局

> 受竖向滚动条滑动影响，当滚动条滑动到设置粘性定位的元素的位置时；再次往下滚动期望该元素可以像粘在屏幕上一样，位置不随滚动条的滑动而变动，直到该元素所有内容滚动完成后，才被下面的元素替代。

## 粘性定位和固定定位的区别
1. 固定定位 `position: fixed`是当元素设置了固定定位之后，元素随即脱离文档流，无论页面怎么滚动，元素都固定在相对视口的位置上不会改变
2. 粘性定位的元素一开始并没有脱离文档流，只有当页面发生滚动，并滚动到该元素位置，元素会粘在屏幕上不动
## 实现方法

1. 使用`position`的`sticky`属性：

> sticky跟postion的其它属性值都不一样，它会产生动态效果，很像relative和fixed的结合：一些时候是relative定位（定位基点是自身默认位置），另一些时候自动变成fixed定位（定位基点是视口）

> 使用sticky属性不生效时可以排查下有没有以下的一些原因：

- 父元素不能overflow:hidden或者overflow:auto属性。
- 必须指定top、bottom、left、right4个值之一，否则只会处于相对定位
- 父元素的高度不能低于sticky元素的高度
- sticky元素仅在其父元素内生效
- 当前浏览器不支持该属性

> 测试代码

```html
<!DOCTYPE html>
<html lang="en">
 
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <style>
    .wrap {
      height: 2000px;
      width: 100%;
      background-color: skyblue;
 
    }
 
    .box1 {
      width: 80%;
      height: 100px;
      background-color: teal;
      margin: 0 auto;
    }
 
    .box2 {
      width: 200px;
      height: 300px;
      background-color: pink;
      top: 0;
      margin: 0 auto;
    }
 
    .sticky {
      position: sticky;
      top: 0px;
    }
  </style>
</head>
 
<body>
  <div class="wrap">
    <div class="box1"></div>
    <div class="box2 sticky"></div>
    <div class="box1"></div>
  </div>
</body>
 
</html>
```

2. 监听滚动事件并动态设置fixed
