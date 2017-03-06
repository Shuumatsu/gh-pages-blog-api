---
title: ArrayBuffer
date: 2017-03-07 02:10:03
tags: [JavaScript, ArrayBuffer, Buffer, TypedArray, DataView]
---

ArrayBuffer 对象被用来表示一个通用的，固定长度的二进制数据缓冲区。你不能直接操纵 ArrayBuffer 的内容；相反，你应该创建一个表示特定格式的buffer的类型化数组对象(typed array objects)或数据视图对象 DataView 来对 buffer 的内容进行读取和写入操作。

<!--more-->

### Endianness 

在几乎所有的机器上，多字节对象都被存储为连续的字节序列。例如在C语言中，一个类型为int的变量x地址为0x100，那么其对应地址表达式&x的值为0x100。且x的四个字节将被存储在存储器的0x100, 0x101, 0x102, 0x103位置。

- 大端序


数据以8bit为单位:

地址增长方向  →

...	0x0A	0x0B	0x0C	0x0D	...

示例中，最高位字节是 0x0A 存储在最低的内存地址处。下一个字节 0x0B 存在后面的地址处。正类似于十六进制字节从左到右的阅读顺序。



数据以16bit为单位:

地址增长方向  →

...	0x0A0B	0x0C0D	...

最高的 16bit 单元 0x0A0B 存储在低位。

- 小端序

数据以 8bit 为单位:

地址增长方向  →

...	0x0D	0x0C	0x0B	0x0A	...

最低位字节是 0x0D 存储在最低的内存地址处。后面字节依次存在后面的地址处。

数据以 16bit 为单位:

地址增长方向  →

...	0x0C0D	0x0A0B	...

最低的 16bit 单元 0x0C0D 存储在低位。


### ArrayBuffer

ArrayBuffer 对象被用来表示一个通用的，固定长度的二进制数据缓冲区。你不能直接操纵 ArrayBuffer 的内容；相反，你应该创建一个表示特定格式的buffer的类型化数组对象(typed array objects)或数据视图对象 DataView 来对 buffer 的内容进行读取和写入操作。

### DataView

DataView视图提供了一个与平台中字节在内存中的排列顺序(字节序)无关的从ArrayBuffer读写多数字类型的底层接口.（我怎么觉得有关呢。。）

```
new DataView(buffer [, byteOffset [, byteLength]])
```

- buffer
	一个现有的ArrayBuffer，用 作DataView 实例的存储空间.
- byteOffset 可选
	视图实例引用的buffer的字节偏移量.如果没有指定,buffer视图会以首字节作为开始。
- byteLength 可选
	字节数组中元素的个数。如果未指定，视图的长度将会以buffer的长度匹配。
    
RangeError
Thrown if the byteOffset and byteLength result in the specified view extending past the end of the buffer.    
    
DataView 的读写方法以 `dataview.getInt32(byteOffset [, littleEndian])` 和 `dataview.setInt32(byteOffset, value [, littleEndian])` 为例。

```
const arrayBuffer = new ArrayBuffer(8)
const int32Array = new Int32Array(arrayBuffer)
const dataView = new DataView(arrayBuffer)

dataView.setInt32(0, 1, true)
console.log(dataView.getInt32(0), int32Array[0])
／／ 16777216 1
```

DataView 的读写方法默认是以大端序（可通过方法的最后一个参数控制），而接下来要提到的 TypedArray 是以小端序进行读写（至少在 chrome 56 上是这样）。16777216 写成32位的二进制为 00000001 00000000 00000000 00000000，即 00000000 0000000 00000000 00000001 的小端序写法。

因此提供一种检验本机字节序的方法：
```
const littleEndian = () => {
  const buffer = new ArrayBuffer(8)
  const dataView = new DataView(buffer)
  const int32Buffer = new Int32Array(buffer)
    
  dataView.set(0, 1, true)
  return dataView.get(0, true) === int32Buffer[0]
}
```

### TypedArray 

一个TypedArray 对象描述一个表示底层的二进制数据缓存区的类似数组(array-like)视图。没有名为TypedArray的全局属性，也没有直接可见的TypedArray构造函数。相反，有许多不同的全局属性，其值是下面列出的特定元素类型的类型化数组构造函数。

```
new TypedArray(length);
new TypedArray(typedArray);
new TypedArray(object);
new TypedArray(buffer [, byteOffset [, length]]);

// where TypedArray() is one of:

Int8Array();
Uint8Array();
Uint8ClampedArray();
Int16Array();
Uint16Array();
Int32Array();
Uint32Array();
Float32Array();
Float64Array();
```

可以通过通常的数组下标的方式来访问 typed array 中的元素。然而设置或者访问这样的下标属性并不会在原型链上搜索。Indexed properties will consult the ArrayBuffer and will never look at object properties.

```
// Setting and getting using standard array syntax
const int16 = new Int16Array(2);
int16[0] = 42;
console.log(int16[0]); // 42

// Indexed properties on prototypes are not consulted (Fx 25)
Int8Array.prototype[20] = 'foo';
(new Int8Array(32))[20]; // 0
// even when out of bound
Int8Array.prototype[20] = 'foo';
(new Int8Array(8))[20]; // undefined
// or with negative integers
Int8Array.prototype[-1] = 'foo';
(new Int8Array(8))[-1]; // undefined

// Named properties are allowed, though (Fx 30)
Int8Array.prototype.foo = 'bar';
(new Int8Array(32)).foo; // "bar"
```

---

Working with complex data structures

By combining a single buffer with multiple views of different types, starting at different offsets into the buffer, you can interact with data objects containing multiple data types. This lets you, for example, interact with complex data structures from WebGL, data files, or C structures you need to use while using js-ctypes.

Consider this C structure:

```
struct someStruct {
  unsigned long id;
  char username[16];
  float amountDue;
};
```

You can access a buffer containing data in this format like this:

```
var buffer = new ArrayBuffer(24);

// ... read the data into the buffer ...

var idView = new Uint32Array(buffer, 0, 1);
var usernameView = new Uint8Array(buffer, 4, 16);
var amountDueView = new Float32Array(buffer, 20, 1);
```

Then you can access, for example, the amount due with amountDueView[0].

[Data structure alignment](https://en.wikipedia.org/wiki/Data_structure_alignment)


## 参考

[https://zh.wikipedia.org/wiki/%E5%AD%97%E8%8A%82%E5%BA%8F](https://zh.wikipedia.org/wiki/%E5%AD%97%E8%8A%82%E5%BA%8F)

[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DataView](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DataView)

[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray)

[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Typed_arrays](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Typed_arrays)
