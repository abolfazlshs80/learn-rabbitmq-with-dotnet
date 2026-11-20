# فصل دوم: پروتکل‌های پیام‌رسانی

## فهرست مطالب
- [Protocol](#protocol)
- [AMQP 0-9-1](#amqp-0-9-1)
- [AMQP 1.0](#amqp-10)
- [MQTT و STOMP](#mqtt-و-stomp)
- [جدول مقایسه](#جدول-مقایسه)

## مقدمه
پروتکل قرارداد ارتباطی بین Client و Broker است. AMQP 0-9-1 زبان اصلی RabbitMQ نیست به‌تنهایی؛ بلکه مدل Client/Broker مورد استفاده توسط RabbitMQ و اکوسیستم .NET است.

## Protocol
پروتکل مشخص می‌کند Client چگونه وصل، Authentication کند، کانال بگیرد، Exchange و Queue declare کند و ACK بفرستد. RabbitMQ می‌تواند با Pluginهایی بیشتر از یک پروتکل پشتیبانی کند، اما پشتیبانی هر پروتکل باعث سازگاری کامل تمام قابلیت‌های AMQP نمی‌شود.

## AMQP 0-9-1
AMQP 0-9-1 نسخه‌ای است که RabbitMQ آن را به‌طور بومی درک می‌کند. مفاهیم اصلی آن:

- **Connection:** ارتباط TCP و Authentication
- **Channel:** کانال منطقی سبک داخل Connection
- **Exchange:** مسیریابی
- **Queue:** نگهداری و تحویل
- **Binding:** ارتباط Exchange و Queue
- **Consumer:** تحویل از Queue

پیام Producer با Exchange ارسال می‌شود؛ Exchange آن را با Routing/Binding به Queueها می‌برد.

## AMQP 1.0
AMQP 1.0 یک نسخه و مدل متفاوت است؛ مفاهیمی مانند Session و Link دارد و با AMQP 0-9-1 قابل جایگزینی مستقیم نیست. برای استفاده از آن نیاز به پشتیبانی خاص Broker/Connector و شناخت قراردادهای آن پروتکل است.

| موضوع | AMQP 0-9-1 | AMQP 1.0 |
|---|---|---|
| مدل اصلی | Exchange/Queue/Binding | Sender/Receiver با Link و Session |
| رابطه | RabbitMQ-centric | Broker و سیستم‌های enterprise متنوع |
| پشتیبانی RabbitMQ | Native | نیازمند integration/plugin یا gateway ممکن |
| Encoding | Model مشخص RabbitMQ | framing و messaging layer مستقل‌تر |

## MQTT
MQTT سبک و مناسب دستگاه‌های IoT است. RabbitMQ MQTT Plugin می‌تواند پیام‌ها را به مدل RabbitMQ پیوند دهد. QoSها: `0` حداکثر یک‌بار انتقال تلاش، `1` حداقل یک‌بار، و `2` تلاش برای exactly once در سطح پروتکل MQTT. Retained Message آخرین پیامِ نگه‌داشته‌شده برای Subscriberهای بعدی است.

## STOMP
STOMP یک پروتکل متنی ساده برای ارسال و Subscription روی Brokerهاست؛ معمولاً برای Web و Clientهای ساده استفاده می‌شود. STOMP باید با Plugin یا gateway مناسب فعال شود و قابلیت‌های Native AMQP را بدون تفاوت ارایه نمی‌کند.

## جدول مقایسه

| ویژگی | AMQP | MQTT | HTTP |
|---|---|---|---|
| هدف | Enterprise Messaging | IoT کم‌سربار | Request/Response |
| ارتباط | Pub/Sub و Queue | Pub/Sub | معمولاً Client-Server |
| Reliability | ACK، Confirm، Persistence | QoS سطح MQTT | Retry و status code |
| زمان | غیرهم‌زمان | غالباً Publisher/Subscriber | هم‌زمان یا async |

## مثال C#
RabbitMQ .NET Client از AMQP 0-9-1 استفاده می‌کند:

```csharp
var factory = new ConnectionFactory
{
    HostName = "localhost",
    Port = 5672,
    UserName = "demo",
    Password = "demo-password"
};

await using var connection = await factory.CreateConnectionAsync();
await using var channel = await connection.CreateChannelAsync();
```

## اشتباهات رایج
- فرض کردن AMQP 1.0 همان AMQP 0-9-1
- فعال نکردن Plugin MQTT قبل از اتصال
- استفاده از STOMP بدون بررسی محدودیت‌های پشتیبانی

## تمرین عملی
سه پروتکل را برای سناریوهای پرداخت بانکی، سنسور IoT و وب‌چت مقایسه کنید و برای هرکدام مناسب‌ترین گزینه را توضیح دهید.

## منابع
- [AMQP 0-9-1 Reference](https://www.rabbitmq.com/amqp-0-9-1-reference)
- [AMQP 1.0](https://www.amqp.org/)
- [MQTT](https://mqtt.org/)
- [STOMP](https://stomp.github.io/)
