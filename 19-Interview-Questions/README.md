# فصل نوزدهم: سؤالات مصاحبه RabbitMQ

## مبانی

### ۱. Message Broker چیست؟
**خلاصه:** برنامه میانی برای دریافت، نگهداری، مسیریابی و تحویل پیام.  
**توضیح کامل:** Publisher و Consumer را به‌لحاظ زمان و مکان از هم جدا می‌کند؛ میتواند ACK، retry و DLX را مدیریت کند.

### ۲. تفاوت Sync و Async چیست؟
Sync منتظر پاسخ است؛ Async پذیرش یا تحویل را از پردازش جدا می‌کند.

### ۳. Point-to-Point با Pub/Sub تفاوت دارد؟
P2P یک پیام معمولاً به یک Consumer از یک Queue می‌رود؛ Pub/Sub یک event به Queueهای مستقل چند سرویس می‌رود.

## AMQP و معماری

### ۴. Connection و Channel را مقایسه کنید.
Connection ارتباط TCP است و Channel مسیر AMQP داخل آن. Connection کم و Channelهای متعدد ایجاد می‌شوند.

### ۵. AMQP 0-9-1 با AMQP 1.0 چه تفاوتی دارد؟
مدل نخست بر Exchange/Queue/Binding است؛ مدل دوم بر Link/Session و message transfer. قابل جایگزینی مستقیم نیستند.

### ۶. Virtual Host چیست؟
namespace ایزوله برای Exchange، Queue، Binding و Permission.

## Exchange و Queue

### ۷. انواع Exchange؟
Direct، Fanout، Topic، Headers و Default.

### ۸. Topic و Direct چه تفاوتی دارد؟
Direct تطبیق دقیق دارد؛ Topic wildcard در چند بخش Routing Key دارد.

### ۹. Durable با Persistent فرق دارد؟
Durable ساختار؛ Persistent پیام. هر دو برای recovery مطلوب‌اند.

### ۱۰. Classic، Quorum و Stream چه‌اند؟
Classic سبک قدیمی/ساده؛ Quorum replicated بر Raft؛ Stream append-only replayable.

## Reliability

### ۱۱. ACK چیست؟
تأیید پردازش موفق Consumer. فقط پس از commit یا اثر قطعی.

### ۱۲. NACK چیست؟
رد delivery و انتخاب Requeue یا DLX.

### ۱۳. تفاوت Consumer ACK و Publisher Confirm؟
ACK تأیید پردازش Consumer است؛ Confirm تأیید پذیرش broker برای Publisher.

### ۱۴. At-least-once چیست؟
پیام حداقل یک بار تحویل می‌شود؛ duplicate ممکن است.

### ۱۵. Exactly-once پردازش چطور است؟
توسط Broker واحد قابل تأیید کامل نیست؛ Inbox، Deduplication و Transaction لازم است.

### ۱۶. Redelivery چه زمانی رخ می‌دهد؟
ACK نیامده، NACK requeue شده یا consumer ارتباط را از دست داده باشد.

## Routing و خطا

### ۱۷. Routing و Binding Key چیست؟
Routing Key در پیام است؛ Binding Key در اتصال Exchange/Queue استفاده می‌شود.

### ۱۸. DLX چه شرایطی دارد؟
NACK requeue=false، TTL، Queue overflow policy یا reject.

### ۱۹. Retry با DLQ تفاوت دارد؟
Retry تلاش دوباره موقت است؛ DLQ ذخیره پیام‌های ناموفق دائمی/تمام‌شده.

### ۲۰. Poison Message چیست؟
پیامی که با همه تلاش‌ها fail می‌شود و باید به DLQ منتقل شود.

## Consumer

### ۲۱. Prefetch چیست؟
تعداد حداکثر deliveryهای unacked برای Consumer.

### ۲۲. Competing Consumers چطور کار می‌کند؟
Consumerهای روی یک Queue، پیام‌ها را تقسیم می‌کنند.

### ۲۳. چرا Prefetch روی Ordering تأثیر دارد؟
چون چند delivery همزمان به چند worker می‌روند و ترتیب نهایی ممکن است تغییر کند.

### ۲۴. Backpressure چیست؟
کنترل سرعت درخواست یا دریافت تا Consumer نشکند.

## مقیاس، امنیت و مانیتورینگ

### ۲۵. Quorum Queue چرا مناسب HA است؟
داده را با raft روی اکثریت Replica نگه می‌دارد.

### ۲۶. Federation با Shovel فرق دارد؟
Federation federation policy و exchange/queue linking؛ Shovel انتقال نقطه‌ای data plane است.

### ۲۷. Cluster با replication فرق دارد؟
Cluster memberها را پیوند می‌دهد؛ replication کپی داده از Queue را بیشتر ماندگار می‌کند.

### ۲۸. Permissionهای RabbitMQ کدام‌اند؟
configure، write و read؛ روی vhost و resource regex اعمال می‌شوند.

### ۲۹. Memory alarm چه می‌کند؟
Broker flow control را سخت‌تر می‌کند تا حافظه آزاد شود.

### ۳۰. چه متریکهایی برای Queue مهم‌اند؟
Ready، unacked، consumer count، publish/delivery/ack/redelivery و memory/disk.

## معماری و .NET

### ۳۱. Outbox Pattern چیست؟
ثبت Event و تغییر DB در یک transaction، انتشار توسط worker و publisher confirm.

### ۳۲. Saga چیست؟
workflow طولانی توزیع‌شده با تراکنش‌های جبرانی.

### ۳۳. چرا یک Channel برای هر پیام بد است؟
هزینه lifecycle و امکان misuse؛ Broker connection/channels باید pooled/manage شوند.

### ۳۴. Channel thread safety در .NET چطور است؟
باید channel را همزمان در چند thread استفاده کرد؟ بهتر نیست؛ ownership مشخص و APIهای async هماهنگ باشند.

### ۳۵. Publisher API جدید را چگونه انتخاب کنیم؟
نسخه RabbitMQ.Client را ثابت نگه دارید و overloadهای async همان نسخه را استفاده کنید؛ API قدیمی را ترکیب نکنید.
