+++
title = "مشکلات URL و URLSearchParams"
author = "حمیدرضا مهدوی پناه"
tags = ["javascript", "typescript", "node.js"]
keywords = ["js", "جاوااسکریپت", "URLSearchParams", "URL", "node.js", "url", "typescript"]
year = "۱۴۰۳"
month = "۹"
day = "۲" 
date = "2024-11-22"
draft =  false
+++

بررسی تفاوت‌های ریزی که حین کار با URLها باعث به وجود آمدن باگ‌های غیرمنتظره می‌شود.

<!--more-->

## همه چیز از یک باگ شروع شد

کار با URL‌ها در JavaScript و Node.js باید ساده باشد، اما یک باگ اخیر در پروژه ما، من را به دنیایی از جزئیات پیچیده در [API‌های `URL` و `URLSearchParams`](https://nodejs.org/api/url.html) برد. در این پست، به بررسی این جزئیات و مشکلات احتمالی آن‌ها در کد و نحوه اجتناب از آن‌ها خواهیم پرداخت.

---

## مشکل: مدیریت URL با Axios

ما این مشکل را هنگام تولید URL‌ها و اضافه کردن امضا‌های هش به آن‌ها پیدا کردیم. پارامترهای کوئری به طور یک‌پارچه percent-encode نمی‌شدند که منجر به رفتار غیرمنتظره و امضا‌های هش اشتباه می‌شد.

واضح شد که تعامل بین اشیای `URL` و `URLSearchParams` نیاز به دقت بیشتری دارد.

---

### مشکل شماره 1: تفاوت بین `URL.search` و `()URLSearchParams.toString`

اولین شگفتی تفاوت بین `URL.search` و `()URLSearchParams.toString` بود.

هنگام استفاده از `searchParams.` برای تغییر `URL`، دقت کنید، زیرا طبق [مشخصات WHATWG](https://url.spec.whatwg.org/) ، شیء `URLSearchParams` از قوانین متفاوتی برای تعیین اینکه کدام کاراکترها باید percent-encode شوند استفاده می‌کند. به عنوان مثال، شیء `URL` کاراکتر تیلد ASCII (`~`) را percent-encode نمی‌کند، در حالی که `URLSearchParams` همیشه آن را encode می‌کند.

```typescript
// Example 1
const url = new URL("https://example.com?param=foo bar")
console.log(url.search) // prints param=foo%20bar
console.log(url.searchParams.toString()) // prints ?param=foo+bar

// Example 2
const myURL = new URL("https://example.org/abc?foo=~bar")
console.log(myURL.search) // prints ?foo=~bar
// Modify the URL via searchParams...
myURL.searchParams.sort()
console.log(myURL.search) // prints ?foo=%7Ebar
```

در پروژه ما، لازم بود به طور صریح `()url.search = url.searchParams.toString` را دوباره اختصاص دهیم تا اطمینان حاصل شود که رشته کوئری به طور یکنواخت encode شده است.

---

### مشکل شماره 2: چالش علامت بعلاوه

یکی دیگر از نکات ظریف این است که چگونه `URLSearchParams` با کاراکترهای `+` برخورد می‌کند. به طور پیش‌فرض، `URLSearchParams` کاراکتر `+` را به عنوان فضای خالی تفسیر می‌کند که ممکن است هنگام encode داده‌های باینری یا رشته‌های Base64 منجر به خرابی داده‌ها شود.

```typescript
const params = new URLSearchParams("bin=E+AXQB+A")
console.log(params.get("bin")) // "E AXQB A"
```

یک راه حل این است که قبل از افزودن مقادیر به `URLSearchParams` از `encodeURIComponent` استفاده کنید:

```typescript
params.append("bin", encodeURIComponent("E+AXQB+A"))
```

جزئیات بیشتر در [مستندات MDN](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams#preserving_plus_signs) موجود است.

---

### مشکل شماره 3: `URLSearchParams.get` در مقابل `()URLSearchParams.toString`

یکی دیگر از جزئیات ظریف زمانی به وجود می‌آید که خروجی‌های `URLSearchParams.get` و `URLSearchParams.toString` را مقایسه می‌کنید. به عنوان مثال:

```typescript
const params = new URLSearchParams("?key=value&key=other")
console.log(params.get("key")) // "value" (اولین مورد)
console.log(params.toString()) // "key=value&key=other" (همه موارد سریالایز شده)
```

در سناریوهای چند مقداری، `get` فقط اولین مقدار را برمی‌گرداند، در حالی که `toString` همه را سریالایز می‌کند.

---

## راه‌حل در کد ما

در پروژه ما، مشکل را با اختصاص صریح `search` حل کردیم:

```typescript
url.search = url.searchParams.toString()
url.searchParams.set(
  "hash",
  cryptography.createSha256HmacBase64UrlSafe(url.href, SECRET_KEY ?? "")
)
```

این اطمینان حاصل کرد که تمام پارامترهای کوئری قبل از اضافه کردن مقدار `hash` به درستی encode شده بودند.

---

## ماژول `querystring` در Node.js

رابط کاربری WHATWG `URLSearchParams` و [ماژول `querystring`](https://nodejs.org/api/querystring.html) هدف مشابهی دارند، اما هدف ماژول `querystring` عمومی‌تر است، زیرا امکان سفارشی‌سازی کاراکترهای جداکننده (`&` و `=`) را فراهم می‌کند. از سوی دیگر، API `URLSearchParams` به طور خاص برای رشته‌های کوئری URL طراحی شده است.

ماژول `querystring` از `URLSearchParams` کارآمدتر است اما یک API استاندارد نیست. از `URLSearchParams` زمانی استفاده کنید که عملکرد بحرانی نیست یا وقتی سازگاری با کد مرورگر مطلوب است.

هنگام استفاده از `URLSearchParams` برخلاف ماژول `querystring`، کلیدهای تکراری به صورت آرایه مجاز نیستند. آرایه‌ها با استفاده از `()array.toString` سریالایز می‌شوند که به سادگی همه عناصر آرایه را با کاما جدا می‌کند.

```typescript
const params = new URLSearchParams({
  user: "abc",
  query: ["first", "second"],
})
console.log(params.getAll("query"))
// Prints [ 'first,second' ]
console.log(params.toString())
// Prints 'user=abc&query=first%2Csecond'
```

با ماژول `querystring`، رشته کوئری `'foo=bar&abc=xyz&abc=123'` به این صورت پارس می‌شود:

```typescript
{
  "foo": "bar",
  "abc": ["xyz", "123"]
}
```

---

## نکات کلیدی

1. هنگام استفاده از `URLSearchParams` به نحوه مدیریت کاراکترهای خاص (مانند `~`) و فضاهای خالی توجه کنید. در صورت نیاز از `encodeURIComponent` استفاده کنید.

2. تفاوت بین `URL.search`، `URLSearchParams.get` و `URLSearchParams.toString` را برای جلوگیری از رفتار غیرمنتظره درک کنید.

3. در Node.js از ماژول `querystring` استفاده کنید اگر می‌خواهید پارامترهای کوئری تکراری را به عنوان یک آرایه پارس کنید.

---

منبع:
[Pitfalls of URL and URLSearchParams in JavaScript](https://hamidreza.tech/pitfalls-of-url-and-urlsearchparams-in-nodejs)
از وبلاگ
Software Alchemist
