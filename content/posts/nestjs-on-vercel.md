+++
title = "استقرار سریع و ساده اپلیکیشن NestJS روی Vercel"
author = "حمیدرضا مهدوی پناه"
tags = ["vercel", "nestjs", "node.js", "deployment"]
keywords = ["js", "جاوااسکریپت", "deployment", "express", "nestjs", "node.js", "vercel", "typescript"]
year = "۱۴۰۳"
month = "۹"
day = "۷" 
date = "2024-11-27"
draft = false
+++

بر اساس راهنمای رسمی Vercel و روش به‌روز

<!--more-->

این راهنما برای کسانی که از [Express](https://expressjs.com/) adapter استفاده می‌کنند مفید است. برای اپلیکیشن‌های NestJS که از [Fastify](https://fastify.dev/) adapter استفاده می‌کنند، لینک‌های زیر ممکن است کمک‌کننده باشد:

- [Fastify Guide for Serverless on Vercel](https://fastify.dev/docs/latest/Guides/Serverless/#vercel)
- [Fastify Examples on GitHub](https://github.com/vercel/examples/tree/main/starter/fastify)

🚀 شما می‌توانید کد کامل مطرح‌شده در این مقاله را در این مخزن GitHub پیدا کنید: [nestjs-on-vercel](https://github.com/mahdavipanah/nestjs-on-vercel)

## پشتیبانی Vercel از Express Apps

Vercel ویژگی مناسبی برای استقرار Express app ارائه می‌دهد که شامل مراحل زیر است:

1. expose کردن شیء Express app در یک API.
2. تعریف یک rewrite rule که تمام ترافیک ورودی را به این API هدایت کند.

من برای استقرار NestJS از [راهنمای رسمی Vercel برای Express](https://vercel.com/guides/using-express-with-vercel) استفاده کردم، به این صورت که شیء Express app زیربنایی NestJS را اکسپوز کردم.

## مرحله ۱ - ساخت یک اپلیکیشن NestJS

اگر قبلاً یک اپ NestJS دارید، می‌توانید این مرحله را رد کنید.

[نصب NestJS](https://docs.nestjs.com/first-steps) و ایجاد اپ جدید:

```bash
nest new my-app
```

## مرحله ۲ - نصب بسته‌های NPM موردنیاز

```bash
npm install express @nestjs/platform-express
npm install -D @types/express
```

## مرحله ۳ - ساخت فایل `src/AppFactory.ts`

این فایل به‌عنوان یک ماژول واحد عمل می‌کند که تمامی پیکربندی‌های ضروری اپلیکیشن NestJS را مدیریت کرده و هم NestJS app و هم شیء Express app زیربنایی آن را اکسپورت می‌کند.

ساخت فایل `AppFactory.ts` در پوشه `src`:

```typescript
import { ExpressAdapter } from "@nestjs/platform-express"
import { NestFactory } from "@nestjs/core"
import express, { Request, Response } from "express"
import { Express } from "express"
import { INestApplication } from "@nestjs/common"
import { AppModule } from "./app.module.js"

export class AppFactory {
  static create(): {
    appPromise: Promise<INestApplication<any>>
    expressApp: Express
  } {
    const expressApp = express()
    const adapter = new ExpressAdapter(expressApp)
    const appPromise = NestFactory.create(AppModule, adapter)

    appPromise
      .then((app) => {
        app.enableCors({
          exposedHeaders: "*",
        })

        app.init()
      })
      .catch((err) => {
        throw err
      })

    expressApp.use((req: Request, res: Response, next) => {
      appPromise
        .then(async (app) => {
          await app.init()
          next()
        })
        .catch((err) => next(err))
    })

    return { appPromise, expressApp }
  }
}
```

## مرحله ۴ - ویرایش فایل `src/main.ts`

به‌طور پیش‌فرض، فایل `src/main.ts` نقطه ورودی اپلیکیشن است. این فایل را تغییر دهید تا تنها متد `listen` را فراخوانی کند:

```typescript
import { AppFactory } from "./AppFactory.js"

async function bootstrap() {
  const { appPromise } = AppFactory.create()
  const app = await appPromise

  await app.listen(process.env.PORT ?? 3000)
}
bootstrap()
```

## مرحله ۵ - افزودن فایل `api/index.ts`

[Vercel](https://vercel.com/docs/functions/runtimes/node-js) به‌طور پیش‌فرض هر فانکشنی که در پوشه `api` قرار دارد را اجرا می‌کند. بنابراین، یک فایل در این پوشه ایجاد کنید که شیء Express app را اکسپورت کند:

```typescript
/**
 * This file exports Express instance for specifically for the deployment of the app on Vercel.
 */

import { AppFactory } from "../src/AppFactory.js"

export default AppFactory.create().expressApp
```

## مرحله ۶ - افزودن فایل `vercel.json`

در دایرکتوری روت پروژه یک فایل `vercel.json` ایجاد کنید تا Vercel را پیکربندی کنید:

```json
{
  "version": 2,
  "buildCommand": "npm run build",
  "outputDirectory": ".",
  "rewrites": [
    {
      "source": "/(.*)",
      "destination": "/api"
    }
  ]
}
```

## مرحله ۷ - ایجاد پروژه روی Vercel

تبریک 🎉! تقریباً تمام شد. حالا یک مخزن Git ایجاد کنید و کدتان را در آن push کنید. سپس وارد حساب Vercel خود شوید، یک پروژه جدید ایجاد کنید و مخزن Git را ایمپورت کنید. [همچنین می‌توانید از مخزن GitHub مثال این مقاله استفاده کنید.](https://github.com/mahdavipanah/nestjs-on-vercel)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732642786627/ff53c571-d062-486f-9ba0-db195dfb7b93.png)

---

منبع:
[Fast and Simple NestJS App Deployment on Vercel](https://hamidreza.tech/nestjs-on-vercel)
از وبلاگ Software Alchemist
