+++
title = "انواع loop برای آرایه‌ها: for و for-in و ()forEach. و for-of"
author = "حمیدرضا مهدوی پناه"
tags = ["javascript", "array", "for"]
keywords = ["js", "جاوااسکریپت", "loop", "array", "for-in", "for-of", "forEach"]
year = "۱۴۰۰"
month = "۲"
day = "۹" 
date = "2021-04-28"
draft =  false
+++

در این نوشته چهار روش مختلف برای استفاده از حلقه‌ها روی آرایه‌ها رو مقایسه می‌کنیم.

<!--more-->

* حلقه‌ی for:
```js
for (let index=0; index < someArray.length; index++) {
  const elem = someArray[index];
  // ···
}
```
* حلقه‌ی for-in:
```js
for (const key in someArray) {
  console.log(key);
}
```
* متد
()forEach.
از کلاس
Array:
```js
someArray.forEach((elem, index) => {
  console.log(elem, index);
});
```
* حلقه‌ی for-of:
```js
for (const elem of someArray) {
  console.log(elem);
}
```
for-of
در اغلب موارد بهترین انتخابه. در ادامه میبینم چرا.

---

##  فهرست مطالب

* [۱ - حلقه‌ی for [جاوااسکریپت ES1]](#for-loop)
* [۲ - حلقه‌ی for-in [جاوااسکریپت ES1]](#for-in-loop)
* [۳ - متد ()forEach. [جاوااسکریپت ES5]](#for-each-loop)
  + [۱.۳ - خارج شدن از ()forEach. - یه راه حل](#for-each-loop-workaround)
* [۴ - حلقه‌ی for-if [جاوااسکریپت ES6]](#for-of-loop)
  + [۱.۴ - حلقه‌ی for-of و آبجکت‌های iterable](#for-of-loop-objects)
  + [۲.۴ - حلقه‌ی for-of و اندیس‌های آرایه](#for-of-loop-indices)
  + [۳.۴ - حلقه‌ی for-of و دسترسی همزمان به مقادیر و اندیس‌ها با ()entries.](#for-of-loop-entries)
* [۵ - نتیجه‌گیری](#conclusion)

---

## ۱ - حلقه‌ی for [جاوااسکریپت ES1] {#for-loop}
حلقه‌ی ساده
for
در جاوااسکریپت قدمت داره و از نسخه‌ی ۱
ECMAScript
وجود داشته.
حلقه زیر اندیس و مقدار هر عنصر آرایه
*arr*
رو چاپ میکنه:
```js
const arr = ['a', 'b', 'c'];
arr.prop = 'property value';

for (let index=0; index < arr.length; index++) {
  const elem = arr[index];
  console.log(index, elem);
}

// Output:
// 0, 'a'
// 1, 'b'
// 2, 'c'
```
نقاط مثبت و منفی این حلقه چیه؟
* کاملا همه‌کارس اما از طرفی وقتی فقط میخوایم روی آرایه حلقه بزنیم، نوشتنش طولانی و پرجزئیاته.
* اگه نمیخوایم از اولین عنصر آرایه شروع کنیم، این حلقه خیلی بدردبخوره. هیچکدوم از روش‌های دیگه این امکان رو بهمون نمیدن.

## ۲ - حلقه‌ی for-in [جاوااسکریپت ES1] {#for-in-loop}
حلقه‌ی
for-in
به اندازه‌ی حلقه‌ی
for
قدمت داره. حلقه‌ی زیر کلید‌های آرایه‌ی
*arr*
رو چاپ میکنه:
```js
const arr = ['a', 'b', 'c'];
arr.prop = 'property value';

for (const key in arr) {
  console.log(key);
}

// Output:
// '0'
// '1'
// '2'
// 'prop'
```
for-in
انتخاب خوبی برای حلقه‌زدن روی آرایه‌های نیست، چون:
* کلیدهای
property
رو مرور میکنه و نه مقادیر رو.
* اندیس‌های آرایه وقتی به صورت کلید‌های
property
دیده‌میشن، به جای این که عدد باشن، از جنس
string
هستن.
([اطلاعات بیشتر درمورد نحوه کار عناصر آرایه‌ها](https://exploringjs.com/impatient-js/ch_arrays.html#array-indices))
* به جای اندیس عناصر آرایه، تمام کلید‌های
property
که قابل شمارش هستن
(enumerable)
رو مرور میکنه (هم اونایی که مال خود آرایه هستن و هم اونایی که به ارث رسیدن).

این ویژگی
for-in
که تمام
propertyهای
به ارث‌رسیده رو مرور میکنه یه جا کاربرد داره: زمانی که میخوایم روی
propertyهای
قابل شمارش یه
object
حلقه بزنیم. اما حتی در اون مورد هم حلقه زدن روی زنجیره
prototype
به شکل دستی بهتره، چون کنترل بیشتری در اختیارمون میذاره.

## ۳ - متد ()forEach. [جاوااسکریپت ES5] {#for-each-loop}
با توجه به این که نه حلقه‌ی
for
و نه حلقه‌ی
for-in
انتخاب‌های خوبی برای حلقه‌زدن روی آرایه‌ها نیستن، یه متد کمکی در
ECMAScript 5
معرفی شد:
()Array.prototype.forEach:
```js
const arr = ['a', 'b', 'c'];
arr.prop = 'property value';

arr.forEach((elem, index) => {
  console.log(elem, index);
});

// Output:
// 'a', 0
// 'b', 1
// 'c', 2
```
این متد خیلی راحته: بدون نیاز به انجام کار زیاد ، هم دسترسی به اندیس و هم به عناصر آرایه رو میده. تابع‌های فِلشی
(Arrow functions)
که در
ES6
معرفی شدن، این متد رو از قبل هم زیباترش کرده.

معایب اصلی
*()forEach.*
از این قراره:
* امکان استفاده از
*await*
در «بدنه» این نوع حلقه وجود نداره.
* از حلقه‌ی
*()forEach.*
نمیشه زودتر از موعد خارج شده. درصورتیکه در حلقه‌های
*for*
میشه از
*break*
استفاده کرد.

### ۱.۳ - خارج شدن از ()forEach. - یه راه حل{#for-each-loop-workaround}
یه راه حل برای خارج شدن از این حلقه هست: استفاده از
*()some.*
که روی تمام عناصر آرایه حلقه میزنه و زمانی که تابع
callback
یه مقدار
truthy
(یه مقداری که بشه به معنی true تفسیرش کرد)
رو برگردونه، متوقف میشه.
```js
const arr = ['red', 'green', 'blue'];
arr.some((elem, index) => {
  if (index >= 2) {
    return true; // از حلقه خارج میشه
  }
  console.log(elem);
  // رو برمیگردونه undefined تابع به طور ضمنی مقدار
  // هست حلقه ادامه پیدا میکنه falsy که چون یه مقدار
});

// Output:
// 'red'
// 'green'
```
## ۴ - حلقه‌ی for-of [جاوااسکریپت ES6] {#for-of-loop}
این حلقه در
ECMAScript 6
اضافه شد:
```js
const arr = ['a', 'b', 'c'];
arr.prop = 'property value';

for (const elem of arr) {
  console.log(elem);
}
// Output:
// 'a'
// 'b'
// 'c'
```
for-of
برای حلقه زدن روی آرایه‌ها خیلی خوبه، چون:
* روی عناصر آرایه تکرار میشه.
* داخلش میتونیم از
*await*
استفاده کنیم.
  + و اگه نیاز پیدا کنیم خیلی راحت میتونیم به 
  for-await-of
  کوچ کنیم.
* میتونیم از
*break*
و
*continue*
استفاده کنیم - حتی در اسکوپ
(scope)
های بیرونی‌تر.
### ۱.۴ - حلقه‌ی for-of و آبجکت‌های iterable{#for-of-loop-objects}
یه مزیت اضافه‌ی
for-of
اینه که با اون نه فقط روی آرایه‌ها بلکه روی آبجکت‌های
iterable
هم میشه حلقه زد - برای مثال، روی
Mapها:
```js
const myMap = new Map()
  .set(false, 'no')
  .set(true, 'yes')
;
for (const [key, value] of myMap) {
  console.log(key, value);
}

// Output:
// false, 'no'
// true, 'yes'
```
حلقه زدن روی
*myMap*
جفت
[key, value]
رو برمیگردونه که در کد بالا از روش
[destructre کردن](https://exploringjs.com/impatient-js/ch_destructuring.html)
برای دسترسی مستقیم به مقادیر این جفت استفاده کردیم.
### ۲.۴ - حلقه‌ی for-of و اندیس‌های آرایه{#for-of-loop-indices}
متد
()keys.
در آرایه‌ها، یه
iterable
روی اندیس‌های آرایه رو برمیگردونه:
```js
const arr = ['chocolate', 'vanilla', 'strawberry'];

for (const index of arr.keys()) {
  console.log(index);
}
// Output:
// 0
// 1
// 2
```
### ۳.۴ - حلقه‌ی for-of و دسترسی همزمان به مقادیر و اندیس‌ها با ()entries.{#for-of-loop-entries}
متد
.entries()
در آرایه‌ها یه
iterable
روی جفت‌های
[index, value]
برمیگردونه. با استفاده از
for-of
و
destructuring
خیلی راحت میتونیم روی مقادیر و اندیس‌های آرایه به طور همزمان حلقه بزنیم:
```js
const arr = ['chocolate', 'vanilla', 'strawberry'];

for (const [index, value] of arr.entries()) {
  console.log(index, value);
}
// Output:
// 0, 'chocolate'
// 1, 'vanilla'
// 2, 'strawberry'
```
## ۵ - نتیجه‌گیری {#conclusion}
همونطور که دیدیم، وقتی معیار کاربرد باشه حلقه‌ی
for-of
از بقیه‌ی روش‌ها بهتره.

هر تفاوتی در سرعت (performance) بین این چهار روش حلقه‌زدن، در حالت عادی نباید اهمیتی داشته باشه. اگه اهمیت داره احتمالا داری یه کار خیلی سنگین از نظر محاسباتی انجام میدی و بنابراین استفاده از
WebAssembly
احتمالا گزینه‌ی معقول‌تری باشه.

---

منبع: 
[Looping over Arrays: for vs. for-in vs. .forEach() vs. for-of](https://2ality.com/2021/01/looping-over-arrays.html)
از وبلاگ
2ality - JavaScript and more