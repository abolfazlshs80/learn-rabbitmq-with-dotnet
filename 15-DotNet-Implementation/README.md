# فصل پانزدهم: پیاده‌سازی RabbitMQ با C# و .NET

## فهرست مطالب
- [راه‌اندازی](#راهاندازی)
- [Producer](#producer)
- [Consumer و BackgroundService](#consumer-و-backgroundservice)
- [Confirms و Lifecycle](#confirms-و-lifecycle)
- [ساختار پروژه](#ساختار-پروژه)

## پیش‌نیازهای نسخه
- RabbitMQ 4.x
- RabbitMQ.Client 7.x
- .NET 8 یا جدیدتر

```bash
cd examples/RabbitMqDemo
docker compose up -d
```

داشبورد را در `http://localhost:15672` با `demo/demo-password` باز کنید.

## راه‌اندازی
فایل compose تنها در محیط آموزش و غیرقابل Production است:

```yaml
services:
  rabbitmq:
    image: rabbitmq:4-management
    ports:
      - "5672:5672"
      - "15672:15672"
```

بدون Docker:

```bash
# Windows: RabbitMQ را روی Chocolatey یا MSI رسمی RabbitMQ نصب کنید
# سپس Erlang/OTP سازگار را طبق مستند رسمی نصب کنید
rabbitmq-plugins enable rabbitmq_management
```

## اتصال
پکیج NuGet را نصب کنید:

```bash
dotnet add package RabbitMQ.Client
```

```csharp
var factory = new ConnectionFactory
{
    HostName = "localhost",
    Port = 5672,
    UserName = "demo",
    Password = "demo-password",
    VirtualHost = "/",
    AutomaticRecoveryEnabled = true
};

await using var connection = await factory.CreateConnectionAsync();
await using var channel = await connection.CreateChannelAsync();
```

## Producer
`IConnection` و Channel را برای عمر برنامه نگه دارید. برای هر پیام Connection یا Channel نسازید.

```csharp
await channel.ExchangeDeclareAsync("orders.topic", "topic", durable: true);
await channel.QueueDeclareAsync("orders.notifications", durable: true,
    exclusive: false, autoDelete: false);
await channel.QueueBindAsync("orders.notifications", "orders.topic", "order.*");

var order = new OrderCreatedEvent(
    MessageId: Guid.NewGuid(),
    OrderId: 1001,
    OccurredAt: DateTimeOffset.UtcNow);

var json = JsonSerializer.Serialize(order);
var body = Encoding.UTF8.GetBytes(json);

var properties = new BasicProperties
{
    Persistent = true,
    ContentType = "application/json",
    MessageId = order.MessageId.ToString(),
    Type = nameof(OrderCreatedEvent)
};

await channel.BasicPublishAsync(
    exchange: "orders.topic",
    routingKey: "order.created",
    mandatory: true,
    basicProperties: properties,
    body: body);
```

`mandatory=true` پیام بدون Routing را مفقود نمی‌کند؛ Consumer application باید callback `basic.return` را مدیریت کند یا رویکرد Confirm/Outbox را بررسی کند.

## Consumer
نمونه مفهومی با RabbitMQ.Client 7:

```csharp
await channel.BasicQosAsync(0, 10, false);

var consumer = new AsyncEventingBasicConsumer(channel);
consumer.ReceivedAsync += async (_, ea) =>
{
    try
    {
        var json = Encoding.UTF8.GetString(ea.Body.ToArray());
        var message = JsonSerializer.Deserialize<OrderCreatedEvent>(json);

        await ProcessAsync(message ?? throw new InvalidDataException());
        await channel.BasicAckAsync(ea.DeliveryTag, multiple: false);
    }
    catch (TemporaryException)
    {
        await channel.BasicNackAsync(ea.DeliveryTag, multiple: false, requeue: true);
    }
    catch
    {
        await channel.BasicNackAsync(ea.DeliveryTag, multiple: false, requeue: false);
    }
};

await channel.BasicConsumeAsync(
    queue: "orders.notifications",
    autoAck: false,
    consumer: consumer);
```

## BackgroundService

```csharp
public sealed class OrderConsumer(
    IConnection connection) : BackgroundService
{
    private IChannel? channel;

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        channel = await connection.CreateChannelAsync(cancellationToken: stoppingToken);
        await channel.BasicQosAsync(0, 10, false, stoppingToken);

        var consumer = new AsyncEventingBasicConsumer(channel);
        consumer.ReceivedAsync += async (_, ea) =>
        {
            try
            {
                var body = Encoding.UTF8.GetString(ea.Body.ToArray());
                await HandleAsync(body, stoppingToken);
                await channel.BasicAckAsync(ea.DeliveryTag, false, stoppingToken);
            }
            catch
            {
                await channel.BasicNackAsync(ea.DeliveryTag, false, requeue: false,
                    cancellationToken: stoppingToken);
            }
        };

        await channel.BasicConsumeAsync("orders.notifications", false,
            consumer, cancellationToken: stoppingToken);
    }

    public override async Task StopAsync(CancellationToken cancellationToken)
    {
        if (channel is not null)
            await channel.CloseAsync(cancellationToken: cancellationToken);

        await base.StopAsync(cancellationToken);
    }
}
```

این نمونه برای درس است؛ در Production پیامها را validate، Dead Letter را واقعی declare و خطاها را طبق Temporary/Permanent categorize کنید.

## Publisher Confirms

```csharp
await channel.ConfirmSelectAsync();

await channel.BasicPublishAsync(
    exchange: "orders.topic",
    routingKey: "order.created",
    mandatory: true,
    basicProperties: properties,
    body: body);

// Use the confirm-enabled overload/API and wait for broker confirmation
// supported by the installed RabbitMQ.Client version.
```

برای جلوگیری از ترکیب API قدیم و جدید، نسخه نصب‌شده را در `PackageReference` ثابت و overloadهای موجود را در IntelliSense پروژه بررسی کنید.

## DLX و Retry Queue

```csharp
await channel.ExchangeDeclareAsync("orders.dlx", "topic", durable: true);
await channel.QueueDeclareAsync("orders.dlq", durable: true,
    exclusive: false, autoDelete: false);
await channel.QueueBindAsync("orders.dlq", "orders.dlx", "#");

await channel.ExchangeDeclareAsync("orders.retry.dlx", "direct", durable: true);
await channel.QueueDeclareAsync("orders.retry.1", durable: true,
    exclusive: false, autoDelete: false,
    arguments: new Dictionary<string, object?>
    {
        ["x-message-ttl"] = 5000,
        ["x-dead-letter-exchange"] = "orders.topic"
    });
```

## مدیریت Lifecycle
- تنها یک IConnection به‌ازای برنامه نگه دارید
- Channel را برای Publisher و یک Consumer scope مدیریت کنید
- Channel را روی چند Thread همزمان استفاده نکنید
- Reconnection و consumer redelivery را idempotent کنید
- Secretها را از appsettings production حذف کنید

## ساختار پروژه

```text
RabbitMqDemo/
├── Producer.Api/
├── Consumer.Worker/
├── Shared/
│   ├── Contracts/
│   └── Events/
└── docker-compose.yml
```

## اشتباهات رایج
- استفاده از Channel برای هر HTTP request
- نگه داشتن Channel state روی چند thread
- ترکیب APIهای 6.x و 7.x
- عدم اتصال Log و trace به MessageId

## تمرین عملی
یک Producer API و یک Worker بسازید؛ درخواست ثبت سفارش را تبدیل به `OrderCreatedEvent` کنید و DLQ را برای خطاهای ناتوان پیاده‌سازی کنید.

## منابع
- [RabbitMQ .NET API Guide](https://www.rabbitmq.com/client-libraries/dotnet-api-guide)
- [.NET Worker Services](https://learn.microsoft.com/en-us/dotnet/core/extensions/workers)
- [RabbitMQ .NET Client](https://github.com/rabbitmq/rabbitmq-dotnet-client)
