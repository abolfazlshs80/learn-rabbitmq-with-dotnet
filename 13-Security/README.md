# فصل سیزدهم: امنیت RabbitMQ

## فهرست مطالب
- [Authentication و Authorization](#authentication-و-authorization)
- [Permissions](#permissions)
- [Virtual Host Isolation](#virtual-host-isolation)
- [TLS و Secrets](#tls-و-secrets)

## Authentication
Authentication هویت Client را مشخص می‌کند؛ RabbitMQ internal users، LDAP و extensions را پشتیبانی می‌کند. در Production رمزها و credentialها را در Secret Manager نگه دارید و `guest/guest` را فعال نگذارید.

## Authorization
Authorization مجوز عملکرد را تعیین می‌کند؛ کاربران روی vhostها permission دارند.

سه سطح دسترسی:
- **configure:** ساخت/حذف Exchange و Queue
- **write:** Publish
- **read:** Consumer و دریافت

## VHost Isolation
هر VHost namespace مستقل Exchange، Queue و Binding دارد؛ Permissionها روی آن و regexهای منابع اعمال می‌شوند.

```text
service:order  -> /orders
configure: ^order\..*
write:     ^(order.events|order.topic)$
read:      ^order\..*
```

## TLS
TLS داده‌های AMQP را رمزنگاری می‌کند. Certificateها را در Keystore یا secret mount نگه دارید، Cipherهای قدیمی را غیرفعال کنید و Broker management را با TLS محافظت کنید.

## Security Practices
- username و password منحصر به سرویس
- کمترین سطح دسترسی
- جداسازی VHost
- security patching و resource limits
- حفاظت شبکه و management port
- no credential در log یا کد
- audit مدیریت و حذف کاربران زنجیره‌ای

## سناریوی عملی
سه کاربر بسازید:
- `order-api`: configure/write روی `order.events`
- `order-worker`: read/write روی Queue و DLX
- `auditor`: read-only روی audit VHost

## اشتباهات رایج
- دادن `.*` به همه serviceها
- استفاده از guest در Production
- رمزگذاری نکردن AMQP/TLS
- قرار دادن secret در image یا compose

## تمرین عملی
دو VHost برای production و staging ایجاد کنید و دسترسی هر service را فقط به VHost خود محدود کنید.

## منابع
- [RabbitMQ Access Control](https://www.rabbitmq.com/docs/access-control)
- [RabbitMQ Networking](https://www.rabbitmq.com/docs/networking)
