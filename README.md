# JS gotchas

## 1. type and comperison

```js
null > 0 // false
null == 0 // false
null >= 0 // true!
```
Comparisons (```>```,```>=```,```<```,```<=```) convert operands to numbers (or srtings?) (```null``` becomes ```0```. On the other hand, the equality check ```==``` for ```undefined``` and ```null``` is defined such that, without any conversions, they equal each other and don’t equal anything else. That’s why ```null == 0``` is false.
```js
null > 0 // 0 > 0 (false)
null == 0 // only null and undefined return true
null >= 0 // 0 >= 0 true!
```
<br>
<br>

## 2. Array.length()

```js
let arr = [1, 2, 3, 4]
arr.length // 4
```
<br>
<br>

## 5. Array.fill()

Let's create a two-dimentional array using ```new Array``` constractor and ```.fill()``` method.<br>
The result seems quite satisfying so far.
```js
const arr = Array(5).fill(Array(5).fill(0))

//  [
//    [0, 0, 0, 0, 0],
//    [0, 0, 0, 0, 0],
//    [0, 0, 0, 0, 0],
//    [0, 0, 0, 0, 0],
//    [0, 0, 0, 0, 0]
//  ]
```
But now let's try to change a random element to 1.
```js
arr[2][1] = 1

//  [
//    [0, 1, 0, 0, 0],
//    [0, 1, 0, 0, 0],
//    [0, 1, 0, 0, 0],
//    [0, 1, 0, 0, 0],
//    [0, 1, 0, 0, 0]
//  ]
```
The whole column has changed!<br>
<br>
It happened because, in fact, inside of the ```.fill()``` method we created only one array<br>
and duplicated it 5 times (aka we duplicated pointers/links that are pointing to the same chank of memory <br>
where our 'real' array is being stored).<br>
<br>
To avoide this situation, we can use ```.map()``` or a classic ```for``` loop
```js
const correctArr = Array(5).fill().map(() => Array(5).fill(0))
```

<br>
<br>
