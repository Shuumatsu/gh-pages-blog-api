---
title: Heap-Sort
categories: [JavaScript, algorithm, sort]
tags: [JavaScript, sort, algorithm, data structure]
---

堆（二叉堆）可以视为一棵完全的二叉树，完全二叉树的一个“优秀”的性质是，除了最底层之外，每一层都是满的，这使得堆可以利用数组来表示（普通的一般的二叉树通常用链表作为基本容器表示），每一个结点对应数组中的一个元素。

<!-- more -->

### 堆

因为堆的这个性质，对于给定的某个结点的下标 i，可以很容易的计算出这个结点的父结点、孩子结点的下标

```
const getParent = i => {
  if (i % 2 === 0) return i / 2 - 1
  return i / 2 - 0.5
}

const parent = getParent(i)
const leftChild = i * 2 + 1
const rightChild = i * 2 + 2
```

### 性质

堆的实现通过构造二叉堆（binary heap），实为二叉树的一种；由于其应用的普遍性，当不加限定时，均指该数据结构的这种实现。这种数据结构具有以下性质。

- 任意节点小于（或大于）它的所有后裔，最小元（或最大元）在堆的根上（堆序性）。

- 堆总是一棵完全树。即除了最底层，其他层的节点都被元素填满，且最底层尽可能地从左到右填入。

将根节点最大的堆叫做最大堆或大根堆，根节点最小的堆叫做最小堆或小根堆。常见的堆有二叉堆、斐波那契堆等。

### 构造

以最小堆为例，构造堆的关键是保证每个节点和他的子节点以及父节点的关系。
流程如下：

![building-a-heap](building-a-heap.png)

可通过以下两个函数来保证：

```
// ensure element at index is larger than its parent
// and iterate up
const up = (arr, index) => {
  let i = index

  while (true) {
    const parent = Math.floor(i / 2)

    if (parent <= 0 || arr[parent] < arr[i]) break

    swap(arr, i, parent)
    i = parent
  }

  return i < index
}

// ensure element at index is smaller than its direct child
// and iterate down
const down = (arr, index) => {
  const size = arr.length
  let i = index

  while (true) {
    const left = i * 2 + 1
    if (left >= size) break

    const right = left + 1
    let j = left
    if (right < size && arr[right] < arr[left])
      j = right

    if (arr[i] < arr[j]) break

    swap(arr, i, j)
    i = j
  }

  return i > index
}
```

将一般数组排为堆：
```
const init = arr => {
  const size = arr.length
  for (let i = getParent(size - 1); i >= 0; i--)
    down(arr, i)

  return arr
}

const fix = (arr, index) => down(arr, index) && up(arr, index)

export default class Heap {

  get size() {
    return this.store.length
  }

  constructor(arr = []) {
    this.store = init(arr)
  }

  push(el) {
    this.store.push(el)
    up(this.store, this.size - 1)
  }

  pop() {
    swap(this.store, 0, this.size - 1)
    const el = this.store.pop()
    down(this.store, 0)

    return el
  }

  remove(index) {
    this.store[index] = this.store.pop()
    fix(this.store, index)
  }
}
```

### 堆排序

堆排序（Heap-Sort）是堆排序的接口算法，Heap-Sort先将数组改造为最大堆，然后将堆顶和堆底元素交换，之后将底部上升，最后重新调用 down 保持最大堆性质。由于堆顶元素必然是堆中最大的元素，所以一次操作之后，堆中存在的最大元素被分离出堆，重复n-1次之后，数组排列完毕。整个流程如下：

![HeapSort](HeapSort.png)

用 JavaScript 代码表示如下：
```
const hs = arr => {
  init(arr)

  for (let i = arr.length - 1; i > 0; i--) {
    swap(arr, 0, i)
    down(arr, 0, i - 1)
  }

  return arr
}
```

---

完整的代码以及测试文件：[https://github.com/VanishingDante/frequently-used-sort-algorithm](https://github.com/VanishingDante/frequently-used-sort-algorithm)