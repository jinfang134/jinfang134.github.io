---
title: 仿Jenkins Blue Ocean 实现的一个Vue Pipeline插件
date: 2020-12-13 17:11:15
categories: 
    - coding
tags:
    - vue
    - 前端
---

## Jenkins Blue Ocean
在`Jenkins`里有一款插件让我们耳目一新,这就是 `Jenkins blue ocean`, 可以很直观地显示当前build的进程和每个stage的详细日志.
![image.png](https://upload-images.jianshu.io/upload_images/2576372-7b61d51d07d4d091.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
出现问题以后,能够快速判断问题出现在哪,快速查看相关的日志,而不受别的stage的日志的影响.
![image.png](https://upload-images.jianshu.io/upload_images/2576372-b75f52423dcc0e67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 缘由
这一良好的设计也被我司的设计人员心水, 并融入到我司的产品设计当中, 设计交到我们开发这边以后,经过前期的调研发现,目前并没有组件能够实现类型的功能.既然找不到,那就只能参考Jenkins的设计,自行开发一个. 

Jenkins pipeline是用svg实现的,前端是react, 整个代码不是特别优雅,代码冗长, 业务逻辑和界面渲染的逻辑混在一块, 特别是曲线描绘部分的代码,看得云里雾里, 所以干脆弃之不用,只采用了他的一些样式.

## SVG简介
SVG 是一种基于 XML 语法的图像格式，全称是可缩放矢量图（Scalable Vector Graphics）。其他图像格式都是基于像素处理的，SVG 则是属于对图像的形状描述，所以它本质上是文本文件，体积较小，且不管放大多少倍都不会失真。
SVG 文件可以直接插入网页，成为 DOM 的一部分，然后用 JavaScript 和 CSS 进行操作。
SVG是基于xml的文本语言, 这就意味着他可以被搜索,索引和压缩,并且可以直接被任何的文本编辑器进行编辑处理.

## Vue 与 SVG
vue与SVG似乎天然是一对最好的搭档, 利用vue可以将svg里的一个个元素,一个个分组写成vue的组件, 将复杂的逻辑简单化,这样就可以将一个复杂的svg图形用vue化繁为简. 同时vue的动态绑定也给svg赋予了二次生命, 使svg可以实时响应数据的变化,而呈现不同的图形效果.

## 相关的算法
### 图的拓扑排序
对图进行拓扑排序是很有必要的，由此可以确定以下几个问题
1. 图的开始节点
2. 图的结束节点
3. 图是否有环

拓扑排序的算法非常简单，下面进行简单的描述：
1. 找到一个入度为0的节点v
2. 删除该节点
3. 重复1,2这个过程，直到所有的节点都被删除
```
  topologicalSorting() {
    let visited = [];
    let result = []
    for (let i = 0; i < this.nodes.length; i++) {
      if (visited[i] == true) {
        continue;
      }
      let list = this.findParents(i)
      if (list.length == 0 || list.every(it => visited[it] == true)) {
        visited[i] = true;
        result.push(i)
        i = 0;
      }
    }
    return result;
  }
```
### 是否有环
判断图是否有环其实有很多种方法，在前一个算法中，我们已经得到图的拓扑排序结果，那么只需要判断一下拓扑排序的结果的长度是否小于图的节点数，如果小于，那么图中有环。
```
hasCircle() {
    let list = this.topologicalSorting()
    return list.length < this.nodes.length;
  }
```
### 图的最长路径
在有向图的渲染当中，如果使用广搜或者深搜的算法来遍历节点，并按照遍历的顺序来安排节点的位置，那么会出现最终的节点不一定出现在最右侧的情况，这样pipeline图就显得不够直观，不能够一目了然地看出图的开始节点和结束节点。
所以，我使用的算法是求解图的最长路径，将路径当中的节点从拓扑排序的结果当中去除，不断重复这个过程，直到所有的节点都被计算出来，最终生成的效果如下图。
![image.png](https://upload-images.jianshu.io/upload_images/2576372-d35149f00e00c525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
最长路径使用递归算法实现：
```
/**
   * 查找从第{index}个节点开始的最长路径，返回经过的未被计算位置的节点,
   * @param {*} index
   */
  findLongestWay(index) {
    let children = this.findChildren(index)
    if (children.length == 0) {
      return [index]
    }
    let arr = [],
      maxLength = 0;
    for (let i = 0; i < children.length; i++) {
      if (this.solvedList[children[i]]) {
        continue;
      }
      let list = this.findLongestWay(children[i])
      if (list.length > maxLength) {
        maxLength = list.length;
        arr = list.slice();
      }
    }
    return [index].concat(arr);
  }
```
### 判断图是否是树
当给定的数据实际描述的是一棵树时，可以针对性地进行一些优化，所以我们首先需要判断是否是一棵树，判断树的方法就是判断某个节点在邻接表中重复出现，因为每个树节点的入度只能为1。所以如果某个节点的入度大于1,那么肯定是一个图。
```
  /**
   * 判断当前的图是否是一棵树
   */
  isTree() {
    let set = new Set();
    for (let i = 0; i < this.nodes.length; i++) {
      if (this.nodes[i].next) {
        if (this.nodes[i].next.some(it => set.has(it.index))) {
          return false;
        }
        this.nodes[i].next.forEach(it => set.add(it.index))
      }
    }
    return true;
  }
```
### 计算树在y轴上占据的宽度
为了给某个树节点分配坐标位置，除了要知道他的父节点以外，还需要知道他需要的宽度，这里使用递归算法来实现，树的宽度是子树宽度的和。
**`优化：`** 可以用记忆化搜索的方法来优化性能，用一个数组来存储子树的宽度，这样在递归调用的时候避免了大量的重复计算，这里因为问题规模一般不大，就没有这么做了。
```
  /**
    * 计算一个树要占的宽度
    * @param {*} index
    */
  getWidthOfTree(index) {
    let node = this.nodes[index]
    if (!node.next || node.next.length == 0) {
      return 1;
    }

    let width = 0;
    for (let i = 0; i < node.next.length; i++) {
      width += this.getWidthOfTree(node.next[i].index)
    }
    return width;
  }
```
### 计算每个树节点的位置
知道树的宽度和树的父节点坐标，就可以递归地计算每个子节点的坐标。
```
  /**
   * 为树计算每个节点的位置
   * @param {*} index
   * @param {*} x
   * @param {*} y
   */
  assignNodeForTree(index, x, y) {
    this.matrix[y][x] = index
    let node = this.nodes[index]
    if (!node.next || node.next.length == 0) {
      return;
    }
    let xx = x + 1; let yy = y;
    for (let i = 0; i < node.next.length; i++) {
      let width = this.getWidthOfTree(node.next[i].index)
      this.assignNodeForTree(node.next[i].index, xx, yy)
      yy += width;
    }
  }
```
最终渲染效果如下图：
![image.png](https://upload-images.jianshu.io/upload_images/2576372-6cdeec64897ebaa2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 项目源码
[https://github.com/jinfang134/vue-pipeline](https://github.com/jinfang134/vue-pipeline)
## Demo
[https://jinfang134.github.io/vue-pipeline/](https://jinfang134.github.io/vue-pipeline/)


## 参考
- [https://developer.mozilla.org/en-US/docs/Web/SVG](https://developer.mozilla.org/en-US/docs/Web/SVG)
