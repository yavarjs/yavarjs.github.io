+++
title = "تنظیم Docker برای یه وب‌اپ Node.js"
author = "حمیدرضا مهدوی پناه"
tags = ["node.js", "docker"]
keywords = ["web app", "داکر", "نود جی‌اس"]
year = "۱۴۰۰"
month = "۲"
day = "۹" 
date = "2021-04-29"
draft =  false
+++

هدف این نوشته معرفی یه مثاله برای این که چجوری یه اپلیکیشن
Node.js
رو به یه کانتینر
Docker
تبدیل کنی. این راهنما برای زمان توسعه اپ هست و نه برای
استقرار اپ برای
production.
همچنین فرض میکنم که یه
[نسخه‌ی نصب شده از داکر](https://docs.docker.com/engine/installation/)
روی سیستمت هست و یه دانش ابتدایی از ساختار یه اپلیکیشن
Node.js
داری.

<!--more-->

در بخش اول این راهنما یه وب‌اپلیکیشن ساده رو با
Node.js
میسازی و سپس یه
image
داکر برای اون اپلکیشن درست میکنی و در آخر هم یه کانتینر رو از روی اون
image
اجرا میکنی.

داکر این امکان رو بهت میده که یه اپلیکیشن رو با محیطش
(environment)
و تمام وابستگی‌هاش
(dependencies)
به یه جعبه تبدیل کنی که اسمش هست «کانتینر»
(container).
معمولا یه کانتینر از یه اپلکیشن تشکیل شده که روی یه سیستم‌عامل لینوکس خیلی سبک و ساده در حال اجراست. یه
image
یه نقشه و طرح برای کانتینره و یه کانتینر یه نمونه در حال اجرا از یه
image
هست.

## ساخت اپ Node.js

در ابتدا یه فولدر بساز که تمام فایلها قراره اونجا زندگی کنن. توی این فولدر یه فایل
`package.json`
بساز که توضیحاتیه درمورد اپ و وابستگی‌هاش:

```json
{
  "name": "docker_web_app",
  "version": "1.0.0",
  "description": "Node.js on Docker",
  "author": "First Last <first.last@example.com>",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.16.1"
  }
}
```

دستور
`npm install`
رو اجرا کن. اگه از نسخه‌ی ۵ به بالای
`npm`
استفاده میکنی، این دستور یه فایل
`package-lock.json`
درست میکنه که بعدا کپی میشه به
image
داکرت.

سپس یه فایل
`server.js`
بساز که یه وب‌اپ رو شامل میشه که از فریمورک
[Express.js](https://expressjs.com/)
استفاده میکنه:

```js
"use strict"

const express = require("express")

// مقادیر ثابت
const PORT = 8080
const HOST = "0.0.0.0"

// اپ
const app = express()
app.get("/", (req, res) => {
  res.send("Hello World")
})

app.listen(PORT, HOST)
console.log(`Running on http://${HOST}:${PORT}`)
```

توی مراحل بعدی، میبینی که چجوری میتونی این اپ رو با استفاده از ایمیج رسمی داکر، داخل یه کانتینر اجراش کنی. اول از همه نیاز داری تا یه
Docker image
برای اپلیکیشنت بسازی.

## ایجاد یه Dockerfile

یه فایل خالی بساز که اسمش هست
`Dockerfile`:

```bash
touch Dockerfile
```

این فایل رو داخل ادیتور مورد علاقت باز کن.

اولین کار اینه که ببینیم از چه ایمیجی کارمون رو شروع کنیم. اینجا از آخرین نسخه‌ی پشتیبانی بلندمدت
(LTS)
استفاده میکنیم که میشه نسخه‌ی
‍`14`
از
`node`
که در مخزن
[Docker Hub](https://hub.docker.com/_/node)
موجوده:

```docker
FROM node:14
```

سپس توی ایمیج یه فولدر بساز که داخلش کدهای اپلیکیشن رو قرار بدی، این فولدر قراره فولدر کاری
(working directory)
برای اپلیکیشنت باشه:

```docker
# رو بساز app دایرکتوری
WORKDIR /usr/src/app
```

ایمیج
(node:14)
از قبل داخلش
Node.js
و
NPM
نصب شده و کار بعدی که نیازه بکنی اینه که وابستگی‌های اپلیکیشنت رو با
`npm`
نصب کنی. لطفا دقت کن که اگه داری از نسخه‌ی ۴ به پایین
`npm`
استفاده میکنی، فایل
`package-lock.json`
ساخته
_نمیشه_.

```docker
# وابستگی‌های اپ رو نصب کن
# از علامت ستاره استفاده شده تا هم فایل قفل و هم فایل عادی کپی بشه
COPY package*.json ./

RUN npm install
# اگه داری اپلیکیشن رو برای پروداکشن میسازی از دستور زیر استفاده کن
# RUN npm ci --only=production
```

دقت کن که به جای کپی کردن کل فولدر کاریمون، فقط فایل
`package.json`
رو کپی کردیم. این کار بهمون اجازه میده تا از مزیت لایه‌های قابل کش در داکر استفاده کنیم
([بیشتر بخوانید](http://bitjudo.com/blog/2014/03/13/building-efficient-dockerfiles-node-dot-js/)).
علاوه‌بر این، فرمان
`npm ci`
کمک میکنه که
build
های سریعتر، قابل اتکاتر و قابل تکثیر برای محیط‌های پروداکشن بسازیم. در این مورد میتونی [اینجا](https://blog.npmjs.org/post/171556855892/introducing-npm-ci-for-faster-more-reliable) بیشتر بخونی.

برای بسته‌بندی کردن سورس‌کدهای اپ توی ایمیج داکر، از دستور
`COPY`
استفاده کن:

```docker
# سورس اپ رو بسته‌بندی کن
COPY . .
```

اپلیکیشنت خودشو به پورت
`8080`
میچسبونه، پس از دستور
`EXPOSE`
استفاده کن تا
داکر این پورت رو برات مپ کنه:

```docker
EXPOSE 8080
```

در آخر، فرمانی که اپلیکیشنت رو به اجرا درمیاره با استفاده از
`CMD`
تعریف کن. در اینجا ما از
‍`node server.js`
برای شروع سرور استفاده میکنیم:

```docker
CMD [ "node", "server.js" ]
```

فایل
`dockerfile`
الان باید اینشکلی باشه:

```docker
FROM node:14

# رو بساز app دایرکتوری
WORKDIR /usr/src/app

# وابستگی‌های اپ رو نصب کن
# از علامت ستاره استفاده شده تا هم فایل قفل و هم فایل عادی کپی بشه
COPY package*.json ./

RUN npm install
# اگه داری اپلیکیشن رو برای پروداکشن میسازی از دستور زیر استفاده کن

# سورس اپ رو بسته‌بندی کن
COPY . .

EXPOSE 8080
CMD [ "node", "server.js" ]
```

## فایل dockerignore.

یه فایل
‍‍`dockerignore`
داخل همون فولدری که فایل
` dockerfile`
هست بساز که محتواش ایناس:

```text
node_modules
npm-debug.log
```

اینکار باعث میشه که ماژول‌های محلی (ماژول‌هایی که روی سیستم خودت داخل فولدر اپ نصب کردی) و لاگ‌های دیباگ توی ایمیج داکر کپی نشه.

## ساخت image

برو به فولدری که
`dockerfile`
داخلشه و فرمان زیر رو اجرا کن تا ایمیج داکر رو بسازه. پرچم
`t-`
یه برچسب به ایمیج میچسبونه که بعدا راحت‌تر بتونی با فرمان
`docker images`
پیداش کنی:

```bash
docker build . -t <your username>/node-web-app
```

حالا باید داکر ایمیجت رو لیست کرده باشه:

```bash
$ docker images

# مثال
REPOSITORY                      TAG        ID              CREATED
node                            14         1934b0b038d1    5 days ago
<your username>/node-web-app    latest     d64d3505b0d2    1 minute ago
```

## اجرای image

اجرا کردن ایمیجل با پرچم
`d-`
کانتینر رو در پس‌زمینه اجرا میکنه. پرچم
`p-`
یه پورت خصوصی داخل کانتینر رو به یه پورت عمومی ریدایرکت میکنه. ایمیجی که از قبل ساختی رو اجرا کن:

```bash
docker run -p 49160:8080 -d <your username>/node-web-app
```

خروجی اپلیکیشنت رو چاپ کن:

```bash
# آیدی کانتینر رو پیدا کن
$ docker ps

# خروجی اپ رو چاپ کن
$ docker logs <container id>

# مثال
Running on http://localhost:8080
```

اگه میخوای بری داخل خود کانتینر، میتونی از فرمان
`exec`
استفاده کنی:

```bash
# به کانتینر وارد شو
$ docker exec -it <container id> /bin/bash
```

## تست

برای این که اپلیکیشنت رو تست کنی، پورتی رو که داکر مپ کرده پیدا کن:

```bash
$ docker ps

# مثال
ID            IMAGE                                COMMAND    ...   PORTS
ecce33b30ebf  <your username>/node-web-app:latest  npm start  ...   49160->8080
```

توی مثال بالا، داکر پورت داخلی
`8080`
کانتینر رو به پورت
`49160`
روی کامپیوترت مپ کرده.

حالا اپلیکیشنت رو با استفاده از
`curl`
فراخوانی کن:

```bash
$ curl -i localhost:49160

HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
Content-Length: 12
ETag: W/"c-M6tWOb/Y57lesdjQuHeB1P/qTV0"
Date: Mon, 13 Nov 2017 20:53:59 GMT
Connection: keep-alive

Hello world
```

امیدوارم این خودآموز بهت کمک کرده باشه که بتونی یه اپلیکیشن ساده
Node.js
رو با
Docker
اجرا کنی.

---

منبع:
[Dockerizing a Node.js web app](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/)
از وبسایت رسمی نود جی‌اس
