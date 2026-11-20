# فصل هجدهم: اشتباهات رایج و Best Practices

## اشتباهات و راه اصلاح

| اشتباه رایج | مشکل | راه اصلاح |
|---|---|---|
| Connection برای هر پیام | فشار TCP و resource leak | IConnection مشترک و بلندمدت |
| Channel unmanaged | race و resource leak | ownership مشخص، channel per thread/task |
| Auto ACK نادرست | از دست رفتن پیام در crash | Manual ACK بعد از commit |
| بدون Publisher Confirm | عدم اطلاع از پذیرش broker | Confirm در Publisher حساس |
| فرض Exactly-once | تکرار واقعی ignored | Idempotency و deduplication |
| Requeue بی‌نهایت | poison message و loop | max retry + DLQ |
| نداشتن DLQ | گم شدن خطاها | DLX/DLQ و alert |
| Prefetch نامناسب | memory یا low throughput | load test و backpressure |
| Consumerهای بیش از حد | DB و CPU contention | semaphore و تعداد محدود |
| پیام بزرگ | copy و memory pressure | reference یا object storage |
| Routing نامناسب | coupling و routing wildcard | contract و Topic namespace |
| Durable/Persistent اشتباه | گمان zero-loss | هر سه جزء را طراحی کنید |
| بدون recovery | outage بعد از reset | automatic recovery و retry/backoff |
| لاگ نکردن خطا | debug پیچیده | MessageId، correlation و trace |
| بدون monitoring | queue backlog unseen | ready/unacked/rates/alerts |
| Queue به‌عنوان Database | کندی و از دست دادن ساختار | DB برای state، Queue برای work |

## چک‌لیست Production

### Reliability
- [ ] Quorum Queue برای workload حساس
- [ ] Durable Exchange/Queue و Persistent Message
- [ ] Publisher Confirms
- [ ] Manual ACK
- [ ] Idempotent Consumer
- [ ] Outbox برای dual write
- [ ] Retry و DLQ

### Operational
- [ ] Resource limits و alerts
- [ ] Disk/memory alarms
- [ ] Prometheus/Grafana
- [ ] Health check و reconnect
- [ ] Dead Letter review
- [ ] backup و recovery test

### Security
- [ ] TLS
- [ ] secret manager
- [ ] least privilege
- [ ] isolated VHost
- [ ] management access محدود

### Application
- [ ] schema/versioning
- [ ] correlation/message IDs
- [ ] bounded concurrency
- [ ] prefetch load test
- [ ] timeout و jitter
- [ ] structured logs
