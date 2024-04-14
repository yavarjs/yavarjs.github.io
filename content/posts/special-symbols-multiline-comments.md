+++
title = "استفاده از @ و /* در کامنت‌های چندخطی"
author = "حمیدرضا مهدوی پناه"
tags = ["jsdoc", "typescript", "javascript"]
keywords = ["تایپ‌اسکریپت", "نود جی‌اس", "جاوااسکریپت", "کامنت"]
year = "۱۴۰۳"
month = "۱"
day = "۲۶" 
date = "2024-04-14"
draft =  false
+++

چجوری مشکل عدم امکان استفاده از علامت‌های خاص و رزرو‌ شده رو در کامنت‌های چندخطی (multiline comments) و [JSDoc](https://jsdoc.app/) حل کنیم؟

<!--more-->

در حال اضافه کردن کامنت JSDoc به کدهام بودم که به یه مشکل اعصاب خوردن‌کن برخوردم. وقتی داشتم یک تیکه کد مثال رو توی کامنتم اضافه میکردم که داخلش از علامت `/*` استفاده میشد، کل بلوک کامنتم خراب میشد. دلیلش هم اینه که این علامت در جاوااسکریپت به عنوان تگ پایان کامنت‌های چندخطی استفاده میشه و وقتی داخل خود کامنت میخوای ازش استفاده کنی جاوااسکریپت فکر میکنه که داری بلوک کامنتت رو میبندی!

این مشکل توی تیکه کد پایین قابل مشاهده‌است:

```ts
/**
 * Checks whether two permission strings are semantically equal or not.
 *
 * @example
 * // returns true
 * equals('a/b/c/d/allow', 'a/b/c/*⁣/*/d/allow');
 *
 * @returns {boolean} True in case of equality and false otherwise.
*/
const equals = (first: string, second: string) => {
    // function's logic
    // ...

    return true; // or false
}
```

این مشکل راه حل ساده و جالبی داره: باید یه کاراکتر جداکننده‌ی مخفی بین `*` و `/` قرار داده بشه! این کاراکتر یونیکد که اسمش هست Unicode invisible separator character باعث میشه که جاوااسکریپت علامت پایان کامنت رو تشخیص نده. این کاراکتر رو میتونید از [اینجا](https://unicode-explorer.com/c/2063) کپی کنید.

دقیقا مشابه همین مشکل وقتی به وجود میاد که بخواید از علامت `@` داخل کامنت‌های JSDoc استفاده کنید. چون این علامت معنی خاصی برای JSDoc داره باعث خراب شدن داکیومنتی میشه که JSDoc تولید میکنه. به این تیکه کد یه نگاه بندازید:

```ts
/**
 * A NestJS handler decorator that defines an access permission constraint and enforces it.
 *
 * @example
 * `⁣`⁣`ts
 * // the request only needs to be authenticated and doesn't need any specific permissions
 * @RequiresAccess()
 * Class MyController {}
 * `⁣`⁣`
 */
export const RequiresAccess = Reflector.createDecorator<
  PermissionPathGen | PermissionPathGen[]
>({
  transform(value) {
    return value == undefined ? MUST_BE_AUTHENTICATED : value
  },
})
```

حل این مشکل هم مشابه قبلیه؛ کافیه یه
[کاراکتر جداکننده‌ی مخفی](https://unicode-explorer.com/c/2063)
قبل از علامت
`@`
قرار بدید.

---

منبع:
[Using @ and \*/ symbols inside JS multiline comments](https://hamidreza.tech/using-special-symbols-inside-js-multiline-comments)
از وبلاگ
Software Alchemist
