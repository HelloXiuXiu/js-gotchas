# JS gotchas

There are lots of people out there saying `I hate JavaScript`. The thing that makes them say so is usually connected to the fact that JavaScript is full of so-called 'gotchas'.
What are these gothcas? Gotchas are 'unexpected' behaviours of some inbuilt functions and methods. The truth is that they are 'unexpected' only from our human logic poin of view. All these behaviuor have an explanation from the language implementations perspective. Yet, it's very easy to get cought into this mental traps, espasially when you still in the middle of the journy of mastering JavaScript.

I'm going to show you 6 of my favourite gotchas, and try to explain what's going on under the hood as mush as the timing allows me, and as much as I know myself.

<br>
<br>

## 1.Null comperison
In JavaScript we have `null` which is spacial type of data with only one possible value (`null`). But let's try to compare it to something which is logically very simmilar – to a zero.


```js
null > 0   // false
null == 0  // false
null >= 0  // true!
```
From the matematical perspective, this behavior doesn't make any sense! If a value is eaqual or grater that 0, one of the first two lines must return `true`. Yet, it's not what happens in JavaScript. 

Comparisons (`>`,`>=`,`<`,`<=`) convert operands to numbers (if they both are not strings), so `null` becomes `0`. On the other hand, the equality check `==` for `undefined` and `null` is defined such that, without any conversions, they equal each other and don’t equal anything else. That’s why `null == 0` is false.
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
When we pass a single argument to the `Math.min()` method, the argument is compared with `Infinity` and the passed value is returned. This is because when we compare any number to `Infinity`, `Infinity` will always be the higher value; so, the number becomes the min value. Therefore with no arguments the method returns `Infinity` as it is.

<br>
<br>


## 3. Coercion

```js
{} + []  // '[object Object]'

[] + {}  // 0

{} + {}  // NaN  or '[object Object][object Object]' ???

[] + []  // ''
```

To execute one of these operations, the operands must be converted to `primitive` values. Objects are converted to primitives by calling
its `ToPrimitive()` → `valueOf()` → `toString()` methods, in that order. Note that primitive conversion calls `valueOf()` before `toString()`,
which is similar to the behavior of number coercion but different from string coercion. If any of the operands is a `string`, then the other
one is converted to a `string` too. <br>

The `ToPrimitive()` method, if present, must return a primitive — returning an object results in a TypeError (`'[object Object]'`).<br>
For `valueOf()` and `toString()`, if one returns an object, the return value is ignored and the other's return value is used instead; <br>
if neither is present, or neither returns a primitive, a TypeError is thrown(`'[object Object]'`). <br>

Neither `{}` nor `[]` have a `ToPrimitive()` method. Both `{}` and `[]` inherit `valueOf()` from `Object.prototype.valueOf`, which returns the object itself. Since the return value is an object, it is ignored. Therefore, `toString()` is called instead. `{}.toString()` returns `'[object Object]'`, while `[].toString()` returns `''`, so the result is their concatenation: `'[object Object]'` (first example).

Arrays have their own implementation of `toString()` method that returns a comma-separated list of elements.
```js
String([])   // ''
String({})   // '[object Object]'
```

From the other hand, execution (and therefore conversion) happenes from left to right. That's why secuence makes change.

<br>
<br>

## 4. Array.length()

Let's create an array with four values.

```js
let arr = [1, 2, 3, 4]
```
Now let's use `delete` to remove the second element.

```js
delete(arr[1])
```
And now let's try to check the array length.
```js
arr.length  // still 4!
```
Actually, `Array.length` shows us not the count of values in the array, but the last element index plus one (even if this element is an empty slot).
```js
arr[99] = 'foo'
arr.length  // 100
```
This creates a sparse array with an empty slot.
```js
arr  // [1, empty, 3, 4, empty × 95, 'foo']
```
So if you want to remove an array element by changing the contents of the array, instead of using `delete()`, use the `splice()` method.


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

Let's create a two-dimentional array using `new Array` constractor and `.fill()` method.<br>
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
It happened because, in fact, inside of the `.fill()` method we created five copies of the same array<br>
(aka we duplicated pointers/links that are pointing to the same chank of memory <br>
where our 'real' array is being stored).<br>
<br>
To avoide this situation, we can use `.map()` or a classic `for` loop.
```js
const correctArr = Array(5).fill().map(() => Array(5).fill(0))
```
Or we can use `Array.from()`.
```js
const correctArr =  Array.from({ length: 5 }, () => Array.from({ length: 5 }, () => 0))
```

</br>
</br>
