# JS gotchas

## 1. Different types comperison

```js
null > 0   // false
null == 0  // false
null >= 0  // true!
```
Comparisons (```>```,```>=```,```<```,```<=```) convert operands to numbers (or srtings?) (```null``` becomes ```0```. On the other hand, the equality check ```==``` for ```undefined``` and ```null``` is defined such that, without any conversions, they equal each other and don’t equal anything else. That’s why ```null == 0``` is false.
```js
null > 0    // 0 > 0 (false)
null == 0   // only null and undefined return true
null >= 0   // 0 >= 0 true!
```

<br>
<br>

## 2. Math.min() and Math.max()

```js
Math.min() // Infinity
Math.max() // -Infinity

Math.max() > Math.min()  // false
Math.max() < Math.min()  // true
```

<br>
<br>


## 3. Binary +

```js
{} + []  // '[object Object]'

[] + {}  // 0

{} + {}  // NaN  or '[object Object][object Object]' ???

[] + []  // ''
```

In JavaScript, the ```+``` operator has two jobs: <br>
1. Produce a sum of numeric operands <br>
2. Concatenate strings <br>
<br>
So the operands will be converted either to a ```string``` or a ```number``` . <br>
If any of the operands is a ```string```, then the other one is converted to a ```string``` too.

```js
Number({})   // NaN
Number([])   // 0

String([])   // ''
String({})   // '[object Object]'
```

From the other hand, execution (and therefore conversion) happenes from left to right. That's why secuence makes change.

```js
{} + []  // concatenation

[] + {}  // summ

{} + {}  // summ

[] + []  // concatenation
```

<br>
<br>

## 4. Array.length()

```js
let arr = [1, 2, 3, 4]
arr.length // 4
```

<br>
<br>

## 5. 'new' keyword

```js
const Car = function(color) {
    this.color = color
}

const a = new Car('blue')
console.log(a.color)    // 'blue'

const b = Car('blue')
console.log(b.color)    // error! (we don't have window.color)
```

Calling a function with the new keyword creates a new object and then calls the function with that new object as its context. The object is then returned. Conversely, invoking a function without 'new' will result in the context being the global object if that function is not invoked on an object (which it won't be anyway if it's used as a constructor!)

The danger in accidentally forgetting to include 'new' means that a number of alternative object-constructing patterns have emerged that completely remove the requirement for this keyword.

<br>
<br>

## 6. Array.fill()

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
The whole column has changed!!!<br>
<br>
It happened because, in fact, inside of the ```.fill()``` method we created five copies of the same array<br>
(aka we duplicated pointers/links that are pointing to the same chank of memory <br>
where our 'real' array is being stored).<br>
<br>
To avoide this situation, we can use ```.map()``` or a classic ```for``` loop
```js
const correctArr = Array(5).fill().map(() => Array(5).fill(0))
```

<br>
<br>
