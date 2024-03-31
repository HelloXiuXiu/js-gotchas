# JS gotchas

## 1. type and comperison

```js
null > 0 // false
null == 0 // false
null >= 0 // true!
```
Comparisons (```>``` and ```>=```) convert ```null``` to a number, treating it as 0. On the other hand, the equality check ```==``` for ```undefined``` and ```null``` is defined such that, without any conversions, they equal each other and don’t equal anything else. That’s why ```null == 0``` is false.
```js
null > 0 // 0 > 0 (false)
null == 0 // only null and undefined return true null === undefined
null >= 0 // 0 >= 0 true!
```
<br>
<br>

## 2. array length

```js
let arr = [1, 2, 3, 4]
arr.length // 4
```
