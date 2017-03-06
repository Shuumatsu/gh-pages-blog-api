---
title: Quick-Sort
categories: [JavaScript, algorithm, sort]
tags: [JavaScript, sort, algorithm]
---

快速排序是图灵奖得主 C. R. A. Hoare 于 1960 年提出的一种划分交换排序。它采用了一种分治的策略，通常称其为分治法(Divide-and-ConquerMethod)。

<!-- more -->

利用分治法可将快速排序的分为三步：

1. 在数据集之中，选择一个元素作为”基准”（pivot）。
2. 所有小于”基准”的元素，都移到”基准”的左边；所有大于”基准”的元素，都移到”基准”的右边。这个操作称为分区 (partition) 操作，分区操作结束后，基准元素所处的位置就是最终排序后它的位置。
3. 对”基准”左边和右边的两个子集，不断重复第一步和第二步，直到所有子集只剩下一个元素为止。

---

快速排序的核心就是分区的实现，用伪代码表示如下
```
function partition(a, left, right, pivotIndex)
  pivotValue := a[pivotIndex]
  swap(a[pivotIndex], a[right]) // 把pivot移到結尾
  storeIndex := left
  for i from left to right-1
    if a[i] <＝ pivotValue
      swap(a[storeIndex], a[i])
      storeIndex := storeIndex + 1
  swap(a[right], a[storeIndex]) // 把pivot移到它最後的地方
  return storeIndex
```

### JavaScript 实现    

在数组较小的时候，希尔排序等有着更好的表现。于是较小范围内不进行分区操作，改为直接排序。 

```
const shellSort = (arr, left, right) => {
  const length = right - left + 1
  if (length <= 1) return arr

  for (let gap = Math.floor(length / 2); gap > 0; gap = Math.floor(gap / 2)) {
    for (let group = left; group < left + gap; group++) {
      for (let target = group + gap; target <= right; target += gap) {
        const targetValue = arr[target]

        let time = 1
        while (target - time * gap >= left && targetValue < arr[target - time * gap])
          time++
        time--

        for (let n = 0; n < time; n++)
          arr[target - n * gap] = arr[target - n * gap - gap]

        arr[target - time * gap] = targetValue
      }

    }
  }

  return arr
}

const quickSort = arr => {
  const par = (left, right) => {
    const p = arr[right]
    let index = left
    for (let i = left; i < right; i++)
      if (arr[i] < p) {
        swap(arr, i, index)
        index++
      }

    swap(arr, index, right)
    return index
  }

  const sort = (left, right) => {
    if (right - left < 12)
      return shellSort(arr, left, right)

    const index = par(left, right)
    sort(left, index - 1)
    sort(index + 1, right)
  }

  sort(0, arr.length - 1)

  return arr
}
```  

---

完整的代码以及测试文件：[https://github.com/VanishingDante/frequently-used-sort-algorithm](https://github.com/VanishingDante/frequently-used-sort-algorithm)