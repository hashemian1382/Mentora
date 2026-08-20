# 🏗️ معماری فنی پروژه — Mentora Platform

> این سند، مرجع رسمی معماری، ساختار پوشه‌ها و استانداردهای فنی پروژه است.
> نام «Mentora» یک نام‌گذاری موقت و خنثی است تا در آینده قابل تغییر برند برای هر مشتری/مدرس باشد.

---

## ۱. فلسفه معماری

**Modular Monolith** — یک اپلیکیشن واحد، با مرزبندی داخلی منضبط بین ماژول‌ها.

اصول حاکم:
- سادگی و سرعت توسعه در اولویت است (بدون پیچیدگی زودهنگام)
- هر ماژول کسب‌وکاری کاملاً مستقل و قابل استخراج در آینده است
- Type Safety کامل از دیتابیس تا UI با یک منبع واحد حقیقت (Zod + TypeScript)
- بدون وابستگی غیرضروری به زیرساخت سنگین (Redis/Queue/WebSocket) در فاز اول؛ افزودن تدریجی در فازهای بعد

---

## ۲. استک فناوری نهایی

| لایه | فناوری | نسخه پیشنهادی |
|---|---|---|
| فریم‌ورک اصلی | Next.js (App Router) | 15+ |
| زبان | TypeScript (Strict Mode) | 5+ |
| مدیریت پکیج | pnpm | Latest |
| استایل‌دهی | Tailwind CSS | 3.4+ |
| کامپوننت‌های UI | shadcn/ui + Radix UI | Latest |
| فونت | Vazirmatn (فارسی) | Latest |
| فرم‌ها | React Hook Form + Zod Resolver | Latest |
| اعتبارسنجی | Zod | Latest |
| مدیریت state سرور | TanStack Query | v5 |
| مدیریت state کلاینت | Zustand | Latest |
| ORM | Prisma | 5+ |
| دیتابیس | PostgreSQL | 15+ |
| احراز هویت | Auth.js (NextAuth v5) + OTP سفارشی | Latest |
| نمودارها | Recharts / Tremor | Latest |
| ذخیره‌سازی فایل | S3-Compatible (لیارا Object Storage) | — |
| پیامک | Kavenegar / Melipayamak | — |
| Real-time (سبک) | Server-Sent Events (SSE) | Native |
| زمان‌بندی وظایف | Cron Route + External Trigger (Liara Cron) | — |
| تست | Vitest + Testing Library | Latest |
| Lint/Format | ESLint + Prettier | Latest |
| هاستینگ | Liara / ArvanCloud | — |

### نکته مهم درباره Real-time و Queue
با توجه به مقیاس واقعی پروژه (حداکثر ~10 کاربر همزمان، ~100 کاربر در روز):
- **از WebSocket/Socket.io و Redis/BullMQ در فاز اول استفاده نمی‌شود.**
- تایمر آزمون: محاسبه سمت سرور + Polling سبک هر چند ثانیه (کاملاً کافی برای این مقیاس)
- اعلان‌های لحظه‌ای ادمین: با Server-Sent Events (بدون نیاز به infrastructure اضافه)
- پردازش تصحیح آزمون: به‌صورت Synchronous در همان Request (حجم داده بسیار کم است)
- یادآوری پیامکی شبانه: یک Route با محافظت Token که توسط سرویس Cron خارجی (رایگان یا داخل پنل هاست) فراخوانی می‌شود

> ساختار پوشه‌بندی طوری طراحی شده که در صورت رشد شدید مقیاس، افزودن Redis/BullMQ/WebSocket در آینده بدون بازنویسی منطق تجاری ممکن باشد (پوشه‌های `infrastructure/cache` و `jobs` به همین منظور از قبل رزرو شده‌اند).

---

## ۳. نمودار معماری لایه‌ای

```

┌─────────────────────────────────────────────────────────┐
│                  Presentation Layer                      │
│     (React Server/Client Components — app/)              │
├─────────────────────────────────────────────────────────┤
│                    API Layer                              │
│   (Route Handlers — فقط ورودی/خروجی HTTP)                 │
├─────────────────────────────────────────────────────────┤
│                Application Layer (modules/)                │
│         قلب منطق تجاری — کاملاً مستقل از HTTP              │
├─────────────────────────────────────────────────────────┤
│                  Domain Layer (domain/)                    │
│     Zod Schemas, Types, Constants — منبع واحد حقیقت        │
├─────────────────────────────────────────────────────────┤
│              Infrastructure Layer (infrastructure/)         │
│      Prisma, Storage, SMS Client, (Future: Redis)          │
└─────────────────────────────────────────────────────────┘

```

---

## ۴. ساختار کامل پوشه‌ها و فایل‌ها

```

mentora/
├── .env.example
├── .eslintrc.json
├── .gitignore
├── .prettierrc
├── next.config.ts
├── package.json
├── pnpm-lock.yaml
├── postcss.config.js
├── tailwind.config.ts
├── tsconfig.json
├── middleware.ts                          ← گارد سراسری احراز هویت/نقش
├── ARCHITECTURE.md
├── README.md
│
├── prisma/
│   ├── schema.prisma                      ← مدل‌های دیتابیس
│   ├── seed.ts                            ← داده اولیه (نقش‌ها، ادمین پیش‌فرض)
│   └── migrations/
│
├── public/
│   ├── fonts/
│   │   └── Vazirmatn/
│   ├── images/
│   └── icons/
│
├── tests/
│   ├── unit/
│   └── e2e/
│
└── src/
    │
    ├── app/
    │   ├── layout.tsx                     ← RootLayout (فونت، تم، Providers)
    │   ├── globals.css
    │   ├── providers.tsx                  ← TanStack Query Provider, Theme Provider
    │   │
    │   ├── (public)/                      ← بخش عمومی — SSG/ISR
    │   │   ├── page.tsx                   ← صفحه اصلی
    │   │   ├── resume/page.tsx            ← رزومه / سوابق آکادمیک
    │   │   ├── services/page.tsx          ← خدمات مشاوره/تدریس
    │   │   ├── testimonials/page.tsx      ← نظرات دانش‌آموزان
    │   │   └── contact/page.tsx           ← فرم تماس و همکاری
    │   │
    │   ├── (blog)/
    │   │   └── articles/
    │   │       ├── page.tsx               ← لیست مقالات
    │   │       └── [slug]/page.tsx        ← صفحه مقاله (ISR)
    │   │
    │   ├── (auth)/
    │   │   ├── layout.tsx
    │   │   ├── login/page.tsx             ← ورود با شماره موبایل
    │   │   └── verify/page.tsx            ← تایید کد OTP
    │   │
    │   ├── (dashboard)/                   ← پنل دانش‌آموز — نیازمند لاگین
    │   │   ├── layout.tsx                 ← گارد نقش STUDENT + Sidebar
    │   │   ├── overview/page.tsx          ← داشبورد خلاصه وضعیت
    │   │   ├── planner/page.tsx           ← برنامه مطالعاتی هفتگی
    │   │   ├── daily-report/page.tsx      ← ثبت گزارش‌کار روزانه
    │   │   ├── question-bank/page.tsx     ← تمرین آزاد بانک تست
    │   │   ├── exams/
    │   │   │   ├── page.tsx               ← لیست آزمون‌های در دسترس
    │   │   │   └── [examId]/
    │   │   │       ├── page.tsx           ← صفحه معرفی/قوانین آزمون
    │   │   │       ├── run/page.tsx       ← محیط اجرای آزمون (ایزوله)
    │   │   │       └── result/page.tsx    ← کارنامه و تحلیل
    │   │   ├── analytics/page.tsx         ← نمودار روند پیشرفت
    │   │   └── files/page.tsx             ← دسترسی به جزوات
    │   │
    │   ├── (admin)/                       ← پنل مدیریت — نیازمند نقش ADMIN
    │   │   ├── layout.tsx                 ← گارد نقش ADMIN/ASSISTANT
    │   │   ├── dashboard/page.tsx         ← آمار کلی سیستم
    │   │   ├── students/page.tsx          ← لیست دانش‌آموزان
    │   │   ├── students/[studentId]/page.tsx  ← پروفایل کامل دانش‌آموز
    │   │   ├── reports-review/page.tsx    ← میز کار بررسی گزارش‌کارها
    │   │   ├── question-bank-manager/page.tsx ← مدیریت بانک سوال
    │   │   ├── exam-builder/page.tsx      ← ساخت آزمون از بانک سوال
    │   │   ├── articles-manager/page.tsx  ← مدیریت محتوا/مقالات
    │   │   └── settings/page.tsx          ← تنظیمات سایت (site.config)
    │   │
    │   └── api/                           ← Route Handlers (فقط دروازه HTTP)
    │       ├── auth/
    │       │   ├── [...nextauth]/route.ts
    │       │   ├── otp/send/route.ts
    │       │   └── otp/verify/route.ts
    │       ├── reports/
    │       │   ├── route.ts
    │       │   └── [id]/route.ts
    │       ├── plans/route.ts
    │       ├── articles/route.ts
    │       ├── files/route.ts
    │       ├── question-bank/route.ts
    │       ├── exams/
    │       │   ├── route.ts
    │       │   └── [examId]/
    │       │       ├── start/route.ts
    │       │       ├── answer/route.ts    ← ذخیره لحظه‌ای هر پاسخ
    │       │       ├── submit/route.ts    ← پایان آزمون و محاسبه نتیجه
    │       │       └── result/route.ts
    │       ├── analytics/route.ts
    │       ├── notifications/
    │       │   └── sse/route.ts           ← Server-Sent Events
    │       └── cron/
    │           └── daily-reminder/route.ts ← فراخوانی توسط Cron خارجی
    │
    ├── modules/                           ← 🔑 لایه Application (منطق تجاری خالص)
    │   ├── auth/
    │   │   ├── auth.service.ts
    │   │   ├── otp.service.ts
    │   │   └── auth.types.ts
    │   ├── user/
    │   │   ├── user.service.ts
    │   │   └── user.repository.ts
    │   ├── mentorship/
    │   │   ├── report.service.ts
    │   │   ├── report.repository.ts
    │   │   ├── study-plan.service.ts
    │   │   └── study-plan.repository.ts
    │   ├── question-bank/
    │   │   ├── question.service.ts
    │   │   ├── question.repository.ts
    │   │   └── topic.service.ts
    │   ├── exam/
    │   │   ├── exam.service.ts            ← شروع/مدیریت آزمون
    │   │   ├── exam-session.service.ts    ← وضعیت لحظه‌ای هر آزمون
    │   │   ├── exam-scoring.service.ts    ← محاسبه نمره/تراز/تحلیل
    │   │   └── exam.repository.ts
    │   ├── content/
    │   │   ├── article.service.ts
    │   │   └── file-resource.service.ts
    │   ├── analytics/
    │   │   └── analytics.service.ts
    │   └── notification/
    │       ├── sms.service.ts
    │       └── notification.service.ts
    │
    ├── domain/                            ← لایه Domain (مستقل از فریم‌ورک)
    │   ├── schemas/                       ← Zod Schemas — منبع واحد اعتبارسنجی
    │   │   ├── auth.schema.ts
    │   │   ├── report.schema.ts
    │   │   ├── exam.schema.ts
    │   │   ├── question.schema.ts
    │   │   └── article.schema.ts
    │   ├── types/
    │   │   └── index.ts
    │   └── constants/
    │       └── roles.ts                   ← STUDENT / ADMIN / ASSISTANT
    │
    ├── infrastructure/                    ← لایه زیرساخت
    │   ├── db/
    │   │   └── prisma.ts                  ← Prisma Client Singleton
    │   ├── storage/
    │   │   └── s3-client.ts               ← اتصال به Object Storage
    │   ├── sms/
    │   │   └── kavenegar-client.ts
    │   └── cache/                         ← 🔒 رزرو شده برای Redis آینده
    │       └── .gitkeep
    │
    ├── components/
    │   ├── ui/                            ← کامپوننت‌های پایه shadcn/ui
    │   ├── shared/                        ← Header, Footer, Navbar, Logo
    │   ├── dashboard/                     ← کامپوننت‌های اختصاصی پنل دانش‌آموز
    │   ├── admin/                         ← کامپوننت‌های اختصاصی پنل ادمین
    │   ├── exam/                          ← تایمر، پالت سوالات، ناوبری آزمون
    │   └── charts/                        ← کامپوننت‌های نموداری مشترک
    │
    ├── hooks/
    │   ├── use-exam-timer.ts
    │   ├── use-daily-report.ts
    │   └── use-current-user.ts
    │
    ├── stores/                            ← Zustand Stores
    │   ├── exam-store.ts
    │   └── ui-store.ts
    │
    ├── lib/
    │   ├── auth.ts                        ← تنظیمات Auth.js
    │   ├── api-client.ts                  ← Wrapper بر fetch برای TanStack Query
    │   ├── utils.ts                       ← توابع کمکی عمومی (cn, formatDate, ...)
    │   └── fonts.ts                       ← تعریف فونت Vazirmatn
    │
    └── config/
        └── site.config.ts                 ← تنظیمات برند (نام، لوگو، رنگ، بیوگرافی)

```

---

## ۵. توضیح مسئولیت ماژول‌ها

| ماژول | مسئولیت |
|---|---|
| `auth` | ورود با OTP پیامکی، صدور Session، مدیریت نقش‌ها |
| `user` | CRUD کاربران، پروفایل دانش‌آموز/ادمین |
| `mentorship` | برنامه‌ریزی مطالعاتی، ثبت و بررسی گزارش‌کار روزانه |
| `question-bank` | مدیریت درخت مباحث، بانک سوالات، سطح سختی |
| `exam` | برگزاری آزمون، ذخیره پاسخ لحظه‌ای، محاسبه نتیجه و تحلیل |
| `content` | مدیریت مقالات آموزشی و فایل‌های دانلودی |
| `analytics` | تولید داده نموداری برای روند پیشرفت دانش‌آموز |
| `notification` | ارسال پیامک (OTP، یادآوری، اطلاع‌رسانی) |

---

## ۶. قوانین انضباطی معماری (خط قرمزها)

1. **Route Handlers هرگز مستقیم به Prisma دسترسی ندارند** — همیشه از طریق `modules/*.service.ts`
2. **هر ماژول فقط از Repository خودش استفاده می‌کند**، نه مستقیم از دیتابیس
3. **اعتبارسنجی همیشه از طریق Zod در `domain/schemas`** تعریف می‌شود و همان schema هم در فرم کلاینت (`React Hook Form`) و هم در Route Handler استفاده می‌شود
4. **کامپوننت‌های Server و Client به‌وضوح جدا می‌شوند** — `"use client"` فقط جایی که تعامل واقعی لازم است
5. **تمام متن‌های قابل‌تغییر برند (نام، رنگ، لوگو، بیوگرافی) از `config/site.config.ts` خوانده می‌شوند** — هیچ‌کجای کد نام شخص هاردکد نمی‌شود
6. **نام‌گذاری فایل‌ها:** kebab-case برای فایل‌ها، PascalCase برای کامپوننت‌های React، camelCase برای توابع و متغیرها

---

## ۷. مدل‌های اصلی دیتابیس (سطح مفهومی)

```

User (role: STUDENT | ADMIN | ASSISTANT)
StudyPlan
DailyReport
Article
FileResource

Subject → Topic → Question
Exam → ExamQuestion
ExamAttempt → ExamAnswer → ExamResult

```

> طراحی دقیق فیلدها و روابط Prisma Schema در فاز جداگانه انجام می‌شود.

---

## ۸. متغیرهای محیطی مورد نیاز (`.env`)

```

DATABASE_URL=
NEXTAUTH_SECRET=
NEXTAUTH_URL=

SMS_PROVIDER_API_KEY=
SMS_PROVIDER_SENDER=

S3_ENDPOINT=
S3_ACCESS_KEY=
S3_SECRET_KEY=
S3_BUCKET_NAME=

CRON_SECRET_TOKEN=

```

---

## ۹. نقشه راه فازبندی توسعه

| فاز | محتوا |
|---|---|
| **فاز ۱** | صفحه عمومی، رزومه، مقالات، دانلود جزوات، فرم تماس |
| **فاز ۲** | احراز هویت OTP، پنل دانش‌آموز، گزارش‌کار روزانه، برنامه‌ریزی |
| **فاز ۳** | بانک تست + موتور آزمون آنلاین + کارنامه تحلیلی |
| **فاز ۴** | تحلیل پیشرفته، نمودار مقایسه‌ای، یادآوری پیامکی خودکار |
| **فاز ۵** | PWA، بهینه‌سازی سئو و پرفورمنس، آماده‌سازی چندبرندی (Multi-tenant) |

---

## ۱۰. مسیر مهاجرت آینده (در صورت رشد شدید مقیاس)

اگر روزی نیاز واقعی به مقیاس بسیار بالاتر بود:
- `infrastructure/cache/` → افزودن Redis برای Session/Rate-Limiting
- افزودن BullMQ + یک Worker مستقل (`workers/`) برای پردازش پس‌زمینه
- ارتقا از Polling به WebSocket برای آزمون آنلاین
- استخراج `modules/exam/` به یک سرویس مستقل در صورت لزوم واقعی

هیچ‌کدام از این موارد نیاز به بازنویسی منطق تجاری فعلی ندارند، چون از ابتدا در لایه `modules/` ایزوله شده‌اند.


