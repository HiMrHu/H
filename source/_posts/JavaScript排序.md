---
title: JavaScript排序
date: 2017-12-11 18:43:41
tags: JavaScript
---

> 本文内容来自《啊哈算法》一书，记录学习过程

<!-- more -->

### 桶排序

桶排序的原理是将数组分到优先的桶里，然后对每个桶进行排序，最后将各个桶中的数据有序的合并起来。

1. 最快
   当输入数据可以均匀的分配到每个桶中
2. 最慢
   当输入的数据被分配到同一个桶中
3. 代码

```js
function main(arr, arrSize) {
  //声明一个函数，函数接受两个参数一个是需要排序的数组，另一个是木桶数
  var ArrSize = arrSize + 1; // 数组是从0开始的所以要+1
  var a = new Array(ArrSize);
  for (var i = 0; i <= ArrSize; i++) {
    a[i] = 0; // 初始化数组为零;
  }
  arr.forEach(item => {
    a[item]++; // 遍历输入的数组对，并对相应位置的数据进行计数；
  });
  var nextArr = [];
  for (var i = arrSize; i >= 0; i--) {
    //从大到小进行for循环遍历
    for (var j = 0; j < a[i]; j++) {
      nextArr.push(i);
    }
  }
  return nextArr;
}
main([1, 2, 3, 5, 4], 10); // [5, 4, 3, 2, 1]
```

### 冒泡排序(前后两两比较)

> 冒泡排序的基本思想是：每次比较相邻的元素，如果他们的顺序错误就把他们交换过来
> 其原理是：每次排列一趟只能讲一个数归位，即第一趟只能讲末位上的数归位，
> 例如将 12 35 99 18 76 这五个数组从大到小排列顺序。
> 第一次循环 [35,99,18,76,12]
> 第二次循环 [99,35,76,18,12]
> 第三次循环 [99,76,35,18,12]
> 底四次循环 [99,76,35,18,12]

冒泡排序的原理是，每一趟只能确定一个数的归位，即第一趟只能将末位上的数归位，第二趟只能将倒数第二位归位，第三趟只能将倒数第三位归位以此类推，直至完成。
总结就是如果有 N 个数进行排序，只需将 N-1 个数归位，也就是需要 N-1 趟操作。而每一趟都需要从第一位开始进行相邻两位数的比较，将较小的一个数放到后面，比较完毕后向后挪以为继续比较下面两个相邻数的大小，重复此步骤直至结束。

看一下代码怎么实现。

```js
function main(arr) {
  // 声明一个函数函数接受一个参数
  for (var i = 0; i < arr.length - 1; i++) {
    // for循环，i< arr.length 因为数组是从0开始的所以—1
    for (var j = i + 1; j < arr.length; j++) {
      // 获取前一个值和后一个值
      var cur = arr[i]; // 储存前一个值
      if (cur < arr[j]) {
        // 对比前后相邻的两个值 如果后一个值大于前一个值则交换两个值的位置
        var index = arr[j]; // 记录后一个值
        arr[j] = cur; // 交换两个值的位置
        arr[i] = index;
      }
    }
  }
  return arr;
}
main([9, 1, 2, 5, 7]); //[9, 7, 5, 2, 1]
```

### 快速排序

快速排序是针对冒泡排序的一种改进，也是一种最常用的排序。
从数组中哪一个值，然后通过这个值挨个和数组里面的值进行比较，如果大于的放一边，小于的放另一边，然后把这些合并起来在进行比较，如此反复即可完成排序。

```js
function quickSort(arr) {
  if (arr.length <= 1) return arr; // 长度为零或者1直接返回
  var pivotIndex = Math.floor(arr.length / 2); //找到数组中的中间数索引，如果如果为小书则向下取整
  var pivot = arr.splice(pivotIndex, 1)[0]; // 找到中间数的值,splice返回的是一个数组所以取0位
  var left = [];
  var right = [];
  for (var i = 0; i < arr.length; i++) {
    //遍历数组
    if (arr[i] < pivot) {
      // 如果小于中间数则放入左数组
      left.push(arr[i]);
    } else {
      right.push(arr[i]); // 如果大于等于则放入右数组
    }
  }
  // 最后使用递归不断重复这个过程就可以得到排序后的数组(递归到左右数组都只剩一个然后组合起来)
  return quickSort(left).concat([pivot], quickSort(right));
}
quickSort([2, 3, 6, 5, 4, 1, 25, 4]); // [1, 2, 3, 4, 4, 5, 6, 25]
```
