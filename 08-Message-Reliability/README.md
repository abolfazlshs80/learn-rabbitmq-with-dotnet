# فصل هشتم: قابلیت اطمینان پیام

## فهرست مطالب
- [ACK و NACK](#ack-و-nack)
- [Persistence و Delivery Semantics](#persistence-و-delivery-semantics)
- [Publisher Confirms](#publisher-confirms)
- [Idempotency](#idempotency)

## مقدمه
قابلیت اطمینان یعنی پیام را به‌گونه‌ای طراحی کنیم که از برنامه‌های شکست و redelivery اثر ناخواسته نداشته باشد.

## ACK و NACK
**Manual ACK** قابل اعتمادتر است. **Automatic ACK** بلافاصله بعد از تحویل ACK می‌فرستد؛ اگر پردازش Crash کند پیام از دست می‌رود.

`Basic.Nack(deliveryTag, multiple, requeue)` می‌تواند چند delivery را رد کند. از `multiple=true` فقط وقتی مطمئن هستید که همه deliveryهای کمتر یا مساوی tag در محدوده قابل مدیریت‌اند استفاده کنید.

## Persistence
پیام `Persistent`، Queue `Durable` و Exchange `Durable` با هم یک مسیر restart-safe می‌سازند؛ اما گارانتی Durability همیشه برابر zero-loss در Power loss نیست. Disk و Replication هم مهم‌اند.

| توجه | معنا |
|---|---|
| Durable Exchange/Queue | ساختار broker پس از restart باقی بماند |
| Persistent Message | پیام برای disk persistence mark شود |
| ACK | کارگر پردازش را تأیید کند |
| Confirm | broker پذیرش یا commit را تأیید کند |

## Delivery Semantics
- **At-most-once:** ACK قبل از پردازش، ممکن است پیام از دست برود اما تکرار کم‌تر است.
- **At-least-once:** اگر Confirm، ACK و Redelivery درست باشند، احتمال تکرار وجود دارد.
- **Exactly-once processing:** در سرویس توزیع‌شده یک قرارداد انتهابه‌انتها است و RabbitMQ به‌تنهایی آن را تضمین نمی‌کند؛ استفاده از Inbox/Deduplication ضروری است.

## Publisher Confirms
Publisher channel را در حالت confirm قرار می‌دهد؛ Broker برای پذیرش یا نپذیرفتن `Ack/Nack` برمیگرداند.

RabbitMQ.Client 7.x نمونه سبک:
```csharp
await channel.ExchangeDeclareAsync("orders.topic", "topic", true);

var properties = new BasicProperties { Persistent = true };
var body = JsonSerializer.SerializeToUtf8Bytes(new { OrderId = 1234 });

await channel.BasicPublishAsync(
    exchange: "orders.topic",
    routingKey: "order.created",
    mandatory: true,
    basicProperties: properties,
    body: body);
// در API جدید Publisher Confirm ممکن است با await/confirm callbacks مدیریت شود
```

برای نمونه production از overloadهای دارای Publisher Confirmation و timeout مشخص استفاده کنید و طبق مستندات نصب‌شده .NET Client پیاده‌سازی کنید.

## Redelivery و Duplicate
`Redelivered` نشان می‌دهد پیام قبلاً تحویل شده است؛ آن را برای تصمیم منطق استفاده کنید اما فقط به آن اعتماد نکنید. هر Consumer Idempotent باشد:
- `MessageId` را ذخیره کنید
- پردازش Atomic یا Transactional انجام دهید
- به‌جای Charge کردن چندباره، وضعیت قبلی را بررسی کنید

## سناریوی پرداخت
1. ثبت پیام پرداخت با `MessageId`
2. Publisher Confirm دریافت کند
3. Consumer بررسی `processed_payments`
4. انجام عملیات و ثبت Deduplication
5. ارسال ACK فقط پس از commit

## اشتباهات رایج
- نداشتن Publisher Confirm در مسیر حساس
- فرض Exactly-once بدون Inbox/Outbox
- Retry بی‌پایان
- Ack بدون ذخیره نتیجه

## تمرین عملی
یک Payment Consumer بسازید که با یک `PaymentId` فقط یک‌بار Charge ثبت کند و پیام تکراری را Ignore کند.

## منابع
- [RabbitMQ Reliability](https://www.rabbitmq.com/docs/reliability)
- [Publisher Confirms](https://www.rabbitmq.com/docs/confirms)
