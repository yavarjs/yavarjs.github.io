+++
title = "کمربند سیاهِ Async Await در Node.js"
author = "حمیدرضا مهدوی پناه"
tags = ["node.js", "async/await", "promise", "javascript", "callback", "کمربند سیاه"]
keywords = ["async", "پرامیس", "نود جی‌اس", "جاوااسکریپت", "کالبک"]
year = "۱۴۰۰"
month = "۲"
day = "۱۲" 
date = "2021-05-02"
draft =  false
+++

در این نوشته یاد میگیری که چجوری اپلیکیشن‌های
Node.jsای
که با
callback
یا
Promise
نوشتی رو
با توابع
async
ساده‌ترشون کنی. 

<!--more-->

اگه قبلا یه نگاهی به الگوی
async/await
و
promiseها
در جاوااسکریپت انداختی ولی هنوز کامل بهشون مسلط نیستی و یا این که فقط نیاز داری تا مرورشون کنی، هدف این نوشته کمک به توئه.

## تابع‌های async چی هستن؟
توابع
async
به طور پیش‌فرض در
Node
در دسترسند و با کلمه‌ی کلیدی
`async`
علامت‌گذاری میشن. این توابع حتی اگه به طور صریح در بدنشون مشخص نکنی، همیشه یه
promise
برمیگردونن. در ضمن، فعلا کلمه‌ی کلیدی
`await`
*فقط*
در داخل توابع
async
قابل استفاده‌اس و نمیشه در دامنه سراسری
(global scope)
ازش استفاده کرد.

داخل یه تابع
async
میتونی منتظر یه
`Promise`
بمونی و یا این که در صورت مردود شدنش
(rejected)
میتونی خطاش رو دستگیر کنی
(catch).

پس اگه یه کدی داری که با
promiseها
پیاده‌سازی شده:
```js
function handler (req, res) {
  return request('https://user-handler-service')
    .catch((err) => {
      logger.error('Http error', err);
      error.logged = true;
      throw err;
    })
    .then((response) => Mongo.findOne({ user: response.body.user }))
    .catch((err) => {
      !error.logged && logger.error('Mongo error', err);
      error.logged = true;
      throw err;
    })
    .then((document) => executeLogic(req, res, document))
    .catch((err) => {
      !error.logged && console.error(err);
      res.status(500).send();
    });
}
```
میتونی با
‍‍`async/await`
شبیه به یه کد همگام
(synchronous)
بنویسیش:
```js
async function handler (req, res) {
  let response;
  try {
    response = await request('https://user-handler-service')  ;
  } catch (err) {
    logger.error('Http error', err);
    return res.status(500).send();
  }

  let document;
  try {
    document = await Mongo.findOne({ user: response.body.user });
  } catch (err) {
    logger.error('Mongo error', err);
    return res.status(500).send();
  }

  executeLogic(document, req, res);
}
```
در حال حاضر در
Node
اگه در یه
promise
خطایی رخ بده که بهش رسیدگی نشده،
Node
فقط بهت هشدار میده، پس بنابراین نیازی نیست تا خودتو توی دردسر ساختن یه
listener
بندازی. هرچند چون این اتفاق نشون‌دهنده‌ی یه حالت نامشخصه، توصیه میشه تا اپلیکیشن رو ببندی و کرش کنی؛ دقیقا شبیه به حالتی که یه خطای
catch
نشده در جایی از کد رخ میده. این کارو میتونی یا با استفاده از پرچم
`unhandled-rejections=strict--`
در کامند‌لاین انجام بدی یا با پیاده‌سازی چیزی شبیه به این:
```js
process.on('unhandledRejection', (err) => { 
  console.error(err);
  process.exit(1);
})
```
قراره تا قابلیت خارج شدن اتوماتیک از پروسه، در نسخه‌های آینده‌ی
Node
اضافه بشه. این که کدت رو از قبل برای اینکار آماده کنی زحمت زیادی نداره ولی خوبیش اینه که وقتی خواستی نسخه‌ها رو آپدیت کنی دیگه نگران این موضوع نیستی.

## الگوها با توابع async
از اونجایی که رسیدگی به عملیات ناهمگام
(asynchronous)
با استفاده از
Promiseها
و یا
callbackها
نیاز به الگوهای پیچیده‌ای داره، وقتی که بتونی این عملیات رو جوری پیاده‌سازی کنی که انگار همگام هستن، کار خیلی برات ساده‌تره.

از نسخه‌ی
10.0.0
نود جی‌اس، پشتیبانی از
async iterators
و حلقه‌ی مرتبطش یعنی
for-await
اضافه شده. این امکانات زمانی بدردت میخورن که میخوای روی مقداری که یه
iterator method
برمیگردونه و از قبل مشخص نیستن، حرکت کنی (حلقه بزنی) و کاری رو انجام بدی و حالت نهایی این تکرار
(iteration)
هم مشخص نیست - معمولا این حالت حین کار با
streamها
پیش میاد.

## تلاش مجدد با عقب‌نشینی نمایی (exponential backoff)
پیاده‌سازی الگوریتم تلاش مجدد با
Promiseها
خیلی بدترکیبه:
```js
function request(url) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      reject(`Network error when trying to reach ${url}`);
    }, 500);
  });
}

function requestWithRetry(url, retryCount, currentTries = 1) {
  return new Promise((resolve, reject) => {
    if (currentTries <= retryCount) {
      const timeout = (Math.pow(2, currentTries) - 1) * 100;
      request(url)
        .then(resolve)
        .catch((error) => {
          setTimeout(() => {
            console.log('Error: ', error);
            console.log(`Waiting ${timeout} ms`);
            requestWithRetry(url, retryCount, currentTries + 1);
          }, timeout);
        });
    } else {
      console.log('No retries left, giving up.');
      reject('No retries left, giving up.');
    }
  });
}

requestWithRetry('http://localhost:3000')
  .then((res) => {
    console.log(res)
  })
  .catch(err => {
    console.error(err)
  });
```
این پیاده‌سازی کاری که میخوایمو میکنه اما میتونیم بازنویسیش کنیم و با
`async/await`
خیلی راحت‌تر کارو انجام بدیم:
```js
function wait (timeout) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve()
    }, timeout);
  });
}

async function requestWithRetry (url) {
  const MAX_RETRIES = 10;
  for (let i = 0; i <= MAX_RETRIES; i++) {
    try {
      return await request(url);
    } catch (err) {
      const timeout = Math.pow(2, i);
      console.log('Waiting', timeout, 'ms');
      await wait(timeout);
      console.log('Retrying', err.message, i);
    }
  }
}
```
خیلی بیشتر به دل میشینه، نه؟

## مقادیر میانی (intermediate values)
با این که مثال پیش‌رو به اندازه‌ی قبلی ترسناک نیست، اما اگه حالتی داشته باشی که ۳ تابع ناهمگام مختلف به شکلی که در زیر توضیح میدم، بهم وابسته باشن، مجبوری تا از بین چندتا راه‌حل زشت و بدترکیب یکیشونو انتخاب کنی.

>تابع
>`functionA`
>یه
>Promise
>برمیگردونه که سپس
>`functionB`
>به مقدارش نیاز داره و بعد از اون
>`functionC`
>به مقدار نهایی
>Promiseهای
>جفت تابع
>`functionA`
>و
>`functionB`
>نیاز داره.

### راه حل ۱: درخت کریسمس then.
```js
function executeAsyncTask () {
  return functionA()
    .then((valueA) => {
      return functionB(valueA)
        .then((valueB) => {          
          return functionC(valueA, valueB)
        })
    })
}
```
توی این راه حل، برای انجام
`functionC`
مقدار
`valueA`
رو از کلوژر
(closure)
سومین
`then`
میگیری و مقدار
`valueB`
رو از
Promise
قبلش که انجام شده. نمیتونی این درخت کریسمس رو مسطح کنی چون در اون صورت کلوژر رو گم میکنی و
`valueA`
در دسترس
`functionC`
نخواهد بود.
### راه حل ۲: حرکت به یه اسکوپ (scope) بالاتر
```js
function executeAsyncTask () {
  let valueA
  return functionA()
    .then((v) => {
      valueA = v
      return functionB(valueA)
    })
    .then((valueB) => {
      return functionC(valueA, valueB)
    })
}
```
توی مثال درخت کریسمس شبیه به همین مثال، از یه اسکوپ بالاتر برای دسترسی به
`valueA`
استفاده کردیم. اما تفاوت این مثال اینه که متغیر
`valueA`
رو خارج از اسکوپ
`then.`
تعریف کردیم تا بتونیم مقدار نهایی اولین
Promise
رو بهش انتساب بدیم.

این مثال قطعا کار میکنه و زنجیره‌ی
`then.`
هم مسطح شده و از نظر معنایی هم درسته. با این وجود اگه از
`valueA`
در جاهای دیگه از تابع استفاده کنی، راه برای پیدا شدن باگ‌های جدید باز میشه. علاوه بر این، مجبوری برای یه مقدار یکسان از دوتا اسم مختلف استفاده کنی -
`valueA`
و
`v`.

### راه حل ۳: آرایه‌ی غیر ضروری
```js
function executeAsyncTask () {
  return functionA()
    .then(valueA => {
      return Promise.all([valueA, functionB(valueA)])
    })
    .then(([valueA, valueB]) => {
      return functionC(valueA, valueB)
    })
}
```
جز این که میخوای درخت رو مسطح کنی، هیچ دلیلی نداره که
`valueA`
رو همراه با
Promiseای
که
`functionB`
برمیگردونه داخل یه آرایه پاس بدیم. مقدار این دو عنصر آرایه ممکنه از دو جنس کاملا مختلف باشن و بنابراین جالب نیست که توی یه آرایه‌ی واحد قرار بگیرن.

### راه حل ۴: یه تابع کمکی بنویس
```js
const converge = (...promises) => (...args) => {
  let [head, ...tail] = promises
  if (tail.length) {
    return head(...args)
      .then((value) => converge(...tail)(...args.concat([value])))
  } else {
    return head(...args)
  }
}

functionA(2)
  .then((valueA) => converge(functionB, functionC)(valueA))
```
البته که میتونی یه تابع کمکی بنویسی تا این آش شله قلمکار رو درست کنی. اما از نظر خوانایی خیلی ضعیفه و بنابراین ممکنه درکش برای کسایی که توی برنامه‌نویسی
functional
موهاشون سفید نشده، سخت باشه.

### با استفاده از `async/await` به طور معجزه‌آسایی مشکلاتمون ناپدید میشه:
```js
async function executeAsyncTask () {
  const valueA = await functionA();
  const valueB = await functionB(valueA);
  return function3(valueA, valueB);
}
```

## چندین درخواست موازی با  async/await
این مثال شبیه قبلیه. فرض کن میخوای چند کار ناهمگام مختلف رو در یک لحظه شروع کنی و از مقادیر برگشتیشون تو جاهای مختلف استفاده کنی:
```js
async function executeParallelAsyncTasks () {
  const [ valueA, valueB, valueC ] = await Promise.all([ functionA(), functionB(), functionC() ]);
  doSomethingWith(valueA);
  doSomethingElseWith(valueB);
  doAnotherThingWith(valueC);
}
```

## متدهای iteration آرایه
اگرچه رفتارشون خیلی غیرمنتظرست ولی
میتونی
`map` ،`filter`
و
`reduce`
رو با توابع
async
استفاده کنی. تلاش کن حدس بزنی که خروجی اسکریپت‌های زیر چیه:

### ‍‍۱. map
```js
function asyncThing (value) {
  return new Promise((resolve) => {
    setTimeout(() => resolve(value), 100);
  });
}

async function main () {
  return [1,2,3,4].map(async (value) => {
    const v = await asyncThing(value);
    return v * 2;
  });
}

main()
  .then(v => console.log(v))
  .catch(err => console.error(err));
```

### ‍‍۲. filter
```js
function asyncThing (value) {
  return new Promise((resolve) => {
    setTimeout(() => resolve(value), 100);
  });
}

async function main () {
  return [1,2,3,4].filter(async (value) => {
    const v = await asyncThing(value);
    return v % 2 === 0;
  });
}

main()
  .then(v => console.log(v))
  .catch(err => console.error(err));
```

### ‍‍۳. filter
```js
function asyncThing (value) {
  return new Promise((resolve) => {
    setTimeout(() => resolve(value), 100);
  });
}

async function main () {
  return [1,2,3,4].reduce(async (acc, value) => {
    return await acc + await asyncThing(value);
  }, Promise.resolve(0));
}

main()
  .then(v => console.log(v))
  .catch(err => console.error(err));
```

**راه حل‌ها:**   
۱.
```js
[ Promise { <pending> }, Promise { <pending> }, Promise { <pending> }, Promise { <pending> } ]
```
۲.`[ 4 ,3 ,2 ,1 ]`   
۳. `10`

اگه خروجی هرکدوم از
promiseهایی
که
`map`
برمیگردونه رو چاپ کنی، میبینی که نتیجه‌ای که مورد انتظاره میده:
`[ 8 ,6 ,4 ,2 ]`.
تنها مشکل اینه که هرکدوم از این مقادیر توسط
`AsyncFunction`
توی یه
Promise
بسته‌بندی شدن.

پس اگه میخوای مقادیرتو بگیری، باید آرایه‌ی برگشتی رو به
`Promise.all`
پاس بدی:
```js
main()
  .then(v => Promise.all(v))
  .then(v => console.log(v))
  .catch(err => console.error(err));
```

بدون
`async/await`
باید اول منتظر تموم شدن
promiseها
میموندی و بعد روی مقادیرشون
`map`
رو اجرا میکردی:
```js
function main () {
  return Promise.all([1,2,3,4].map((value) => asyncThing(value)));
}

main()
  .then(values => values.map((value) => value * 2))
  .then(v => console.log(v))
  .catch(err => console.error(err));
``` 
**روش دوم یکم واضح‌تره، نه؟**   
روشی که از
`async/await`
استفاده میکنه زمانی که برای هر مقدار
promise
نیازه تا کارهای همگام طولانی بکنی و در عین حال کارهای ناهمگام هم طولانی‌اند، میتونه بدردت بخوره.

با این روش، به محض این که اولین مقدار رو بدست بیاری، میتونی محاسبتش رو شروع کنی و مجبور نیسی برای شروع محاسباتت منتظر بقیه‌ی
Promiseها
بمونی تا تکمیل بشن. اگرچه خروجی نهایی مقادیر توی
Promiseها
پیچیده شدن، اما این
Promiseها
خیلی سریع تکمیل میشن و در نهایت کل فرآیند سریع‌تر از زمانیه که بخوای به طور متوالی انجامش بدی.

**داستان `filter` چیه؟ مطمئنا یه چیزی این وسط اشتباهه...**   
خب، درست حدس زدی: اگرچه مقادیر برگشتی از این قراره:
`[ false, true, false, true ]`
اما هرکدومشون توی یه
Promise
پیچیده شدن، و از اونجایی که هر
Promise
یه مقدار اصطلاحا
truthy
هست (یعنی زمانیه که به شکل بولین ببینیمش، مقدارش `true` میشه)،
تمام مقادیر آرایه برمیگردن و هیچکدوم فیلتر نمیشن. متاسفانه تنها کاری که برای رفع این مشکل از دستت برمیاد اینه که اول منتظر بشی تا مقادیر
Promiseها
برگردن و بعد فیلترشون کنی.

متد
`reduce`
خیلی سرراسته. اگرچه یادت باشه که باید مقدار اولیه رو داخل
`Promise.resolve`
بپیچی، و همچنین مقدار متغیر تجمعی هم به صورت
Promise
برمیگرده و بنابراین نیازه تا براش از
`await`
استفاده کنی.

## بازنویسی اپلیکیشن‌های برپایه‌ی callback
توابع
async
به طور پیشفرض یه
`Promise`
برمیگردونن، بنابراین میتونی هر تابعی که برپایه‌ی
callback
هست رو جوری بازنویسی کنی که از
Promiseها
استفاده کنه و سپس مقدار بازگشتیش رو با
`await`
بگیری. توی
Node.js
برای تبدیل یه تابع برپایه‌ی
callback
به یه تابع برپایه‌ی
Promise
میتونی از تابع
[util.promisify](http://2ality.com/2017/05/util-promisify.html)
استفاده کنی.

## بازنویسی اپلیکیشن‌های برپایه‌ی Promise
زنجیره‌های ساده‌ی
`then.`
رو خیلی سرراست میشه با استفاده از
`async/await`
بازنویسی کرد.
```js
function asyncTask () {
  return functionA()
    .then((valueA) => functionB(valueA))
    .then((valueB) => functionC(valueB))
    .then((valueC) => functionD(valueC))
    .catch((err) => logger.error(err))
}
```
تبدیل میشه به:
```js
async function asyncTask () {
  try {
    const valueA = await functionA();
    const valueB = await functionB(valueA);
    const valueC = await functionC(valueB);
    return await functionD(valueC);
  } catch (err) {
    logger.error(err);
  }
}
```

## اپلیکیشن‌های Node.js رو با `async/await` بازنویسی کن اگه:
* مفاهیم قدیمی و باحالی مثل شرط‌های
`if-else`
و حلقه‌های
`for/while`
رو دوس داری.

* باور داری که بلوک‌های
`try-catch`
راه درست رسیدگی به خطاهاست.

همونطور که باهم دیدیم، استفاده از
`async/await`
هم خوانایی کدهارو بیشتر میکنه و هم نوشتنشون رو ساده‌تر میکنه و در بسیاری از کارها مناسب‌تر از زنجیره‌های
`()Promise.then`
هست. اما اگه به شور و اشتیاقی که در چند سال اخیر برای برنامه‌نویسی فانکشنال بوجود اومده دچار شدی، شاید بهتر باشه که از خیر این قابلیت جاوااسکریپت بگذری.

---

منبع: 
[Rewriting Node.js apps with async/await](https://blog.risingstack.com/mastering-async-await-in-nodejs/)
از وبلاگ
RisingStack