# فصل بیستم: تمرین‌های عملی

## Beginner

### ۱. ایجاد Queue و ارسال پیام
**خروجی:** یک پیام JSON در Queue با properties.  
**راهنمایی:** Docker را اجرا کنید و Producer ساده بسازید.  
**ارزیابی:** queue declare، publish و مشاهده در management UI.

### ۲. دریافت پیام
**خروجی:** یک Consumer پیام را چاپ کند و ACK کند.

### ۳. Direct Exchange
**خروجی:** routing key دقیق فقط به Queue مناسب برود.

### ۴. دو Consumer
**خروجی:** پیامها بین دو Consumer پخش شود.  
**چالش اضافه:** Prefetch را تغییر دهید.

## Intermediate

### ۵. Topic Exchange
مسیرهای `order.created`, `order.cancelled` و `order.*` را بسازید.

### ۶. Fanout Exchange
یک پیام به Email، SMS و Audit Queue برسد.

### ۷. Manual ACK و NACK
پردازش موفق ACK و خطا NACK requeue=false شود.

### ۸. Publisher Confirms
Producer فقط بعد از Ack broker وضعیت success را ثبت کند.

### ۹. DLQ
پیام ناموفق به DLQ برود و پیام پر شده قابل replay باشد.

### ۱۰. Prefetch
Prefetchهای 1،10 و50 را تست کنید و metrics را مقایسه کنید.

## Advanced

### ۱۱. Retry Exponential Backoff
`1,2,4,8` ثانیه با jitter و max attempts داشته باشید.

### ۱۲. Idempotent Consumer
یک MessageId را فقط یک بار اجرا کند.

### ۱۳. Outbox
DB transaction و Outbox Worker را پیاده کنید.

### ۱۴. BackgroundService
Worker ASP.NET Core با lifecycle و healthcheck بسازید.

### ۱۵. RPC
Request/Reply با CorrelationId و Timeout پیاده کنید.

### ۱۶. سیستم چندسرویسی
Order، Payment، Inventory و Notification را به Exchange topic متصل کنید.

### ۱۷. Monitoring
Prometheus و Grafana را برای ready/unacked و memory/disk راه‌اندازی کنید.

### ۱۸. High Availability
cluster سه Node و Quorum Queue را آزمایش کنید؛ یک Node را قطع کنید.

## قالب گزارش
هر تمرین را در README خود با این موارد ثبت کنید:
- معماری
- کد اجراشده
- خروجی و dashboard
- Failure scenario
- نتیجه و اقدامات بهبود
