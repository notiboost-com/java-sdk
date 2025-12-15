# NotiBoost Java SDK

Official Java SDK for NotiBoost - Notification Orchestration Platform.

## Installation

### Maven

```xml
<dependency>
    <groupId>com.notiboost</groupId>
    <artifactId>notiboost-sdk</artifactId>
    <version>1.0.0</version>
</dependency>
```

### Gradle

```gradle
implementation 'com.notiboost:notiboost-sdk:1.0.0'
```

## Requirements

- Java 8 or higher

## Quick Start

```java
import com.notiboost.NotiBoostClient;
import com.notiboost.model.Event;
import java.time.Instant;
import java.util.HashMap;
import java.util.Map;

NotiBoostClient client = new NotiBoostClient.Builder()
    .apiKey("YOUR_API_KEY")
    .build();

Event event = new Event.Builder()
    .eventName("order_created")
    .eventId("evt_001")
    .occurredAt(Instant.now())
    .userId("u_123")
    .properties(createProperties())
    .build();

EventResult result = client.events().ingest(event);
System.out.println("Trace ID: " + result.getTraceId());
```

## API Reference

### Constructor

```java
NotiBoostClient client = new NotiBoostClient.Builder()
    .apiKey("YOUR_API_KEY")
    .baseUrl("https://api.notiboost.com")  // optional
    .timeout(30)                            // optional, seconds
    .retries(3)                             // optional
    .build();
```

### Events

#### `events().ingest(event)`

Ingest a single event.

```java
Event event = new Event.Builder()
    .eventName("order_created")
    .eventId("evt_001")
    .occurredAt(Instant.now())
    .userId("u_123")
    .properties(properties)
    .build();

EventResult result = client.events().ingest(event);
```

#### `events().ingestBatch(events)`

Ingest multiple events in a single request.

```java
List<Event> events = Arrays.asList(
    new Event.Builder()
        .eventName("order_created")
        .eventId("evt_001")
        .userId("u_123")
        .properties(properties1)
        .build(),
    new Event.Builder()
        .eventName("payment_success")
        .eventId("evt_002")
        .userId("u_123")
        .properties(properties2)
        .build()
);

BatchResult result = client.events().ingestBatch(events);
```

### Users

#### `users().create(user)`

Create a new user.

```java
User user = new User.Builder()
    .userId("u_123")
    .name("Nguyễn Văn A")
    .email("user@example.com")
    .phone("+84901234567")
    .properties(properties)
    .build();

UserResult result = client.users().create(user);
```

#### `users().get(userId)`

Get user by ID.

```java
User user = client.users().get("u_123");
```

#### `users().update(userId, data)`

Update user.

```java
Map<String, Object> data = new HashMap<>();
data.put("name", "Nguyễn Văn B");
client.users().update("u_123", data);
```

#### `users().delete(userId)`

Delete user.

```java
client.users().delete("u_123");
```

#### `users().setChannelData(userId, channelData)`

Set channel data for user.

```java
Map<String, Object> channelData = new HashMap<>();
channelData.put("email", "user@example.com");
channelData.put("phone", "+84901234567");
channelData.put("push_token", "fcm_token_abc123");
channelData.put("push_platform", "android");
channelData.put("zns_oa_id", "123456789");

client.users().setChannelData("u_123", channelData);
```

#### `users().setPreferences(userId, preferences)`

Set user notification preferences.

```java
UserPreferences preferences = new UserPreferences.Builder()
    .channel("zns", true)
    .channel("email", true)
    .channel("sms", true)
    .channel("push", true)
    .category("order", true)
    .category("marketing", false)
    .build();

client.users().setPreferences("u_123", preferences);
```

#### `users().createBatch(users)`

Create multiple users in a single request.

```java
List<User> users = Arrays.asList(
    new User.Builder()
        .userId("u_123")
        .name("Nguyễn Văn A")
        .email("user1@example.com")
        .phone("+84901234567")
        .build(),
    new User.Builder()
        .userId("u_124")
        .name("Trần Thị B")
        .email("user2@example.com")
        .phone("+84901234568")
        .pushToken("fcm_token_xyz789")
        .pushPlatform("ios")
        .build()
);

BatchResult result = client.users().createBatch(users);
```

### Flows

#### `flows().create(flow)`

Create a notification flow.

```java
Flow flow = new Flow.Builder()
    .name("order_confirmation")
    .description("Send order confirmation via ZNS")
    .rule(new Rule("event_name == 'order_created'", "send_zns"))
    .channels(Arrays.asList("zns"))
    .templateId("tpl_order_confirm")
    .build();

FlowResult result = client.flows().create(flow);
```

### Templates

#### `templates().create(template)`

Create a template.

```java
Map<String, String> content = new HashMap<>();
content.put("header", "Xác nhận đơn hàng");
content.put("body", "Đơn hàng {{order_id}} đã được xác nhận. Tổng tiền: {{amount}} VNĐ");
content.put("footer", "Cảm ơn bạn đã mua sắm");

Template template = new Template.Builder()
    .name("order_confirmation_zns")
    .channel("zns")
    .content(content)
    .variables(Arrays.asList("order_id", "amount"))
    .build();

TemplateResult result = client.templates().create(template);
```

#### `templates().list(options)`

List templates.

```java
TemplateListOptions options = new TemplateListOptions.Builder()
    .channel("zns")
    .build();

List<Template> templates = client.templates().list(options);
```

#### `templates().get(templateId)`

Get template by ID.

```java
Template template = client.templates().get("tpl_order_confirm");
```

#### `templates().update(templateId, data)`

Update template.

```java
Map<String, Object> data = new HashMap<>();
Map<String, String> content = new HashMap<>();
content.put("body", "Updated body content");
data.put("content", content);

client.templates().update("tpl_order_confirm", data);
```

### Webhooks

#### `webhooks().create(webhook)`

Create a webhook.

```java
Webhook webhook = new Webhook.Builder()
    .url("https://your-app.com/webhooks/notiboost")
    .events(Arrays.asList("message.sent", "message.delivered", "message.failed"))
    .secret("your_webhook_secret")
    .build();

WebhookResult result = client.webhooks().create(webhook);
```

## Error Handling

```java
try {
    client.events().ingest(event);
} catch (NotiBoostException e) {
    switch (e.getStatusCode()) {
        case 429:
            // Rate limit exceeded
            System.out.println("Rate limit exceeded, retrying...");
            break;
        case 401:
            // Invalid API key
            System.out.println("Invalid API key");
            break;
        default:
            System.out.println("Error: " + e.getMessage());
    }
}
```

## Idempotency

Use `Idempotency-Key` header for idempotent requests:

```java
RequestOptions options = new RequestOptions.Builder()
    .header("Idempotency-Key", "unique-key-12345")
    .build();

client.events().ingest(event, options);
```

## Best Practices

1. Use singleton pattern for client instance
2. Store API key in environment variables or configuration
3. Handle exceptions properly
4. Use idempotency keys for critical operations
5. Use connection pooling for high-throughput applications

