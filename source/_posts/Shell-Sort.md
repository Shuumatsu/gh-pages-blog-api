---
title: Shell-Sort
categories: [JavaScript, algorithm, sort]
tags: [JavaScript, sort, algorithm]
---

希尔排序，也称递减增量排序算法，是插入排序的一种更高效的改进版本。希尔排序是非稳定排序算法。

<!-- more -->

希尔排序是基于插入排序的以下两点性质而提出改进方法的：
插入排序在对几乎已经排好序的数据操作时，效率高，即可以达到线性排序的效率
但插入排序一般来说是低效的，因为插入排序每次只能将数据移动一位

### 算法描述

将数组列在一个表中并对列排序（用插入排序）。重复这过程，不过每次用更长的列来进行。最后整个表就只有一列了。

将数组转换至表是为了更好地理解这算法，算法本身仅仅对原数组进行排序（通过增加索引的步长，例如是用i += step_size而不是i++）。


例如，假设有这样一组数[ 13 14 94 33 82 25 59 94 65 23 45 27 73 25 39 10 ]，如果我们以步长为5开始进行排序，我们可以通过将这列表放在有5列的表中来更好地描述算法，这样他们就应该看起来是这样： 

```
13 14 94 33 82
25 59 94 65 23
45 27 73 25 39
10
```

然后我们对每列进行排序：

```
10 14 73 25 23
13 27 94 33 39
25 59 94 65 82
45
```

将上述四行数字，依序接在一起时我们得到：[ 10 14 73 25 23 13 27 94 33 39 25 59 94 65 82 45 ].这时10已经移至正确位置了，然后再以3为步长进行排序：

```
10 14 73
25 23 13
27 94 33
39 25 59
94 65 82
45
```

排序之后变为：

```
10 14 13
25 23 33
27 25 59
39 65 73
45 94 82
94
```

最后以1步长进行排序（此时就是简单的插入排序了）。

### 步长序列

已知的最好步长序列是由Sedgewick提出的(1, 5, 19, 41, 109,...)，该序列的项来自 `9 * (4 ** i) - 9 * (2 ** i) + 1`，`(2 ** (i + 2)) * (2 ** (i + 2) - 3) + 1` 这两个算式。这项研究也表明“比较在希尔排序中是最主要的操作，而不是交换。”用这样步长序列的希尔排序比插入排序要快，甚至在小数组中比快速排序和堆排序还快，但是在涉及大量数据时希尔排序还是比快速排序慢。

### JavaScript 实现

```
const sort = (a, b) => {
  if (a > b) return [b, a]
  if (b > a) return [a, b]
  return [a]
}

const generateSequences = (arrLength) => {
  const gap = arrLength / 2
  const arr = []

  for (let i = 0; ; i++) {
    const a = 9 * (4 ** i) - 9 * (2 ** i) + 1
    const b = (2 ** (i + 2)) * (2 ** (i + 2) - 3) + 1

    arr.push(...sort(a, b))

    if (a > gap || b > gap)
      break
  }

  let index = arr.length - 1
  while (arr[index] > gap)
    index--

  return arr.slice(0, index + 1)
}

const insertionSort = arr => {
  const length = arr.length
  if (length <= 1) return arr

  const insert = (start, end, toInsert) => {
    const targetValue = arr[toInsert]

    let index = end
    while (index >= start && arr[index] >= targetValue) {
      index--
    }
    index++

    for (let i = end + 1; i > index; i--)
      arr[i] = arr[i - 1]

    arr[index] = targetValue
  }

  for (let i = 1; i < length; i++) {
    insert(0, i - 1, i)
  }

  return arr
}

```

---

完整的代码以及测试文件：[https://github.com/VanishingDante/frequently-used-sort-algorithm](https://github.com/VanishingDante/frequently-used-sort-algorithm)