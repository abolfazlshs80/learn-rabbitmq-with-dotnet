# فصل دهم: کنترل پردازش Consumer

## فهرست مطالب
- [Prefetch](#prefetch)
- [Concurrency و Competing Consumers](#concurrency-و-competing-consumers)
- [Ordering و Backpressure](#ordering-و-backpressure)

## Prefetch
**Prefetch Count** تعداد deliveryهای unacked مجاز به یک Consumer را محدود می‌کند. در RabbitMQ از طریق Consumer Qos/BasicQos تنظیم می‌شود.

مثال: Prefetch=10 به معنی حداکثر 10 پیام نهایی‌نشده برای آن Consumer است؛ به معنای تعداد دقیق Thread نیست.

## Consumer Concurrency
RabbitMQ یک Consumer می‌تواند چند پیام را Concurrent پردازش کند اما تعداد پردازش باید با CPU، DB pool و I/O محدود شود. در ASP.NET Core از Channel و semaphore استفاده کنید.

```csharp
var semaphore = new SemaphoreSlim(4);

await channel.BasicQosAsync(prefetchSize: 0, prefetchCount: 20, global: false);

// در Worker هر handler با semaphore محدود شود
await semaphore.WaitAsync();
try
{
    await HandleAsync(body);
}
finally
{
    semaphore.Release();
}
```

## Competing Consumers
چند Consumer روی یک Queue پیام‌ها را تقسیم می‌کنند. این الگو مقیاس‌پذیری را بالا می‌برد اما ترتیب پردازش مشترک تضمین نمی‌شود.

## ACK
- Manual: کنترل دقیق
- Auto: ساده اما ممکن است پیام قبل از پردازش از دست برود
- ACK بعد از عملیات Transaction/Idempotency

## Ordering
ترتیب در Queue حملات مختلف حفظ می‌کند اما چند Consumer، Requeue و redelivery می‌تواند ترتیب پردازش را تغییر دهد. برای نگهداری ترتیب سخت، یک partition/ordering key و یک Consumer فعال را در نظر بگیرید.

## Backpressure
Prefetch پایینتر، حافظه و unacked را کم می‌کند؛ Prefetch بسیار بالا ممکن است مصرف‌کننده را غرق کند. تعداد مناسب باید با زمان پردازش، concurrency و تست بار تعیین شود.

## Fair Dispatch
Prefetch بر توزیع عادلانه تأثیر می‌گذارد زیرا Consumerهای سریع‌تر می‌توانند ظرفیت آزادتری داشته باشند. نباید Prefetch=1 را به معنی گارانتی throughput بالا دانست.

## سناریوی عملی
یک Queue با:
- Consumerهای 1، 2 و 4
- Prefetchهای 1، 10 و 50

مفروضات را با زمان‌های پردازش 50ms و 2s مقایسه کنید. معیارها: throughput، queue lag و معدل memory.

## اشتباهات رایج
- Prefetch خیلی بزرگ برای پردازش سنگین
- تعداد Consumer بیش از thread pool یا DB capacity
- ACK دیرهنگام و بی‌شمار
- فرض کردن Single Consumer برای ترتیب مطلق

## منابع
- [RabbitMQ Consumer Prefetch](https://www.rabbitmq.com/docs/consumers)
- [Consumer Acknowledgements](https://www.rabbitmq.com/docs/confirms)
