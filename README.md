# JS gotchas

There are lots of people out there saying `I hate JavaScript`. The thing that makes them say so is usually connected to the fact that JavaScript is full of so-called 'gotchas'.
What are these gotchas? Gotchas are 'unexpected' behaviors of some inbuilt functions and methods. The truth is that they are 'unexpected' only from our human logical point of view. All these behaviors have an explanation from the language implementation perspective. Yet, it's very easy to get caught in these mental traps, especially when you are still in the middle of the journey of mastering JavaScript.

I'm going to show you 6 of my favorite gotchas and try to explain what's going on under the hood as much as the timing allows me, and as much as I know myself.

<br>
<br>

## 1. Null comparison
In JavaScript, we have `null` which is a special type of data with only one possible value (`null`). But let's try to compare it to something that is logically very similar – to a zero.


```js
null > 0   // false
null == 0  // false
null >= 0  // true!
```
From the mathematical perspective, this behavior doesn't make any sense! If a value is equal or greater than 0, one of the first two lines must return `true`. Yet, it's not what happens in JavaScript. 

Comparisons (`>`,`>=`,`<`,`<=`) convert operands to numbers (if they both are not strings), so `null` becomes `0`. On the other hand, the equality check `==` for `undefined` and `null` is defined such that, without any conversions, they equal each other and don’t equal anything else. That’s why `null == 0` is false.
```js
null > 0    // 0 > 0 (false)
null == 0   // only null and undefined return true
null >= 0   // 0 >= 0 true!
```

<br>
<br>

## 2. Math.min() and Math.max()

If we try to go smart and run `Math.min()` and `Math.max()` without providing any arguments to get an absolute minimum and maximum value, we'll get a quite confusing result:
```js
Math.min() // Infinity
Math.max() // -Infinity

Math.max() > Math.min()  // false
Math.max() < Math.min()  // true
```
When we pass a single argument to the `Math.min()` method, the argument is compared with `Infinity`, and the passed value is returned. This is because when we compare any number to `Infinity`, `Infinity` will always be the higher value; so, the number becomes the min value. Therefore with no arguments, the method returns `Infinity` as it is.

<br>
<br>

## 3. Array.length()

Let's create an array with four values.

```js
let arr = [1, 2, 3, 4]
```
Now let's try to remove the second element using `delete()`.

```js
delete(arr[1])
```
And now let's check the array length.
```js
arr.length  // still 4!
```
Actually, `Array.length` shows us not the count of values in the array, but the last element index plus one (even if this element is an empty slot). Another visual proof for that:
```js
arr[99] = 'foo'
arr.length  // 100
```
These actions create a sparse array with an empty slot.
```js
arr  // [1, empty, 3, 4, empty × 95, 'foo']
```
So if you want to remove an array element, instead of using `delete()` (that is ment to be used for removing Object properties, not elements of an array) use the `splice()` method. Array methods update the whole array and its length property.

<br>
<br>

## 4. Coercion

```js
[] + {}  // '[object Object]'

{} + []  // 0

{} + {}  // NaN

[] + []  // ''
```

In JavaScript, two operators share the same `+` symbol - binary plus and unary plus. The binary addition `+` either performs string concatenation or numeric addition. There unary `+` converts the operand into a number.

Let's first try to find out, which operator was used on each line. <br><br>


### 1. Binary `+`

To execute binary addition operands must be converted either to strings or numbers. Objects are converted to primitives by calling its `ToPrimitive()` → `valueOf()` → `toString()` methods, in that order. Neither `{}` nor `[]` have a `ToPrimitive()` method. Both `{}` and `[]` inherit `valueOf()` from `Object.prototype.valueOf`, which returns the object itself. Since the return value is an object, it is ignored. Therefore, `toString()` is called instead.

Arrays have their own implementation of the `toString()` method that returns a comma-separated list of elements.

```js
let arr = []
let obj = {}

arr.toString()   // ''
obj.toString()   // '[object Object]'
```
As a result, we should have a string concatenation for all four cases. And it works as it is supposed to for the first example and the last one:

```js
[] + {}  // toString() conversion: '' + '[object Object]' = '[object Object]'

[] + []  // toString() conversion: '' + '' = ''
```
But what happened with the other two?...<br><br>


### 2. Unary `+`

We can see now that if the object `{}` goes first, the code acts a little weird. In fact, the first `{}` is treated there as an empty code block, and therefore just thrown away. 

```js
{} + []  // {} is thrown away, + [] is a numeric conversion Number([]) = 0

{} + {}  // {} is thrown away, Number({}) = NaN or ('[object Object][object Object]' for some V8 engine versions')
```

So here we simply have the unary addition `+` applied to the operand, which converts an empty array `+ []` to a number `0`, and an empty object to `NaN`.
```js
Number([])   // 0
Number({})   // NaN
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

Let's create a two-dimensional array using `new Array` constractor and `.fill()` method.<br>
The result seems quite satisfying so far.
```js
const arr = new Array(5).fill(new Array(5).fill(0))

//  [
//    [0, 0, 0, 0, 0],
//    [0, 0, 0, 0, 0],
//    [0, 0, 0, 0, 0],
//    [0, 0, 0, 0, 0],
//    [0, 0, 0, 0, 0]
//  ]
```
But now let's try to change one random element to 1.
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
The whole column has changed!!!

It happened because, in fact, inside of the `.fill()` method we created five copies of the same array (i.e. we duplicated pointers/links that are pointing to the same chunk of memory where our 'real' array is being stored).

To avoid this situation, we can use `.map()` or a classic `for` loop
```js
const correctArr = new Array(5).fill().map(() => new Array(5).fill(0))
```
..or `Array.from()`.
```js
const correctArr =  Array.from({ length: 5 }, () => Array.from({ length: 5 }, () => 0))
```

</br>
</br>
