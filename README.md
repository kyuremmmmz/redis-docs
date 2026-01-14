# Redis with Docker – Auth, Persistence & App Integration

This repository documents how to run **Redis using Docker** with:

- 🔐 Password authentication
- 💾 Data persistence
- 🔌 Connection examples for Node.js, Laravel, and Flutter

> ⚠️ This setup uses the official Redis Docker image (recommended for production).
> 

---

## 📦 Requirements

- Docker
- Docker Compose (recommended)
- Node / Laravel / Flutter (optional, for integration)

---

## 🧠 Architecture Overview

```
Host Machine
 ├── Docker
 │    ├── RedisContainer
 │    │    ├── Redis Server
 │    │    └── Persistent Volume
 │    └── AppContainer (Node / Laravel)
 │
 └── Flutter App (mobile / web)

```

---

## 🐳 Redis Setup (Docker)

### 1️⃣ Create `docker-compose.yml`

```yaml
version:"3.9"

services:
redis:
image:redis:7
container_name:redis
command: >
      redis-server
      --requirepass myStrongRedisPassword
      --appendonly yes
ports:
-"6379:6379"
volumes:
-redis_data:/data
restart:unless-stopped

volumes:
redis_data:

```

---

### 2️⃣ Start Redis

```bash
docker-compose up -d

```

Check logs:

```bash
docker logs redis

```

You should see:

```
Readyto accept connections

```

---

## 🔐 Redis Authentication

Redis is protected using:

```
--requirepass myStrongRedisPassword

```

### Connect via CLI

```bash
dockerexec -it redis redis-cli

```

Authenticate:

```
AUTH myStrongRedisPassword
PING

```

Expected:

```
OK
PONG

```

---

## 💾 Redis Persistence (Enabled)

Persistence is enabled via:

```
--appendonlyyes

```

- Redis data is stored in `/data`
- Docker volume: `redis_data`
- Data **survives container restarts**

Test:

```bash
docker restart redis

```

Reconnect and check keys — still there ✅

---

## 🧪 Redis Test Commands

```
SET name "Chrissa"
GET name

```

Expected:

```
OK
"Chrissa"

```

---

## 🔌 Connecting Redis to Applications

---

## 🟢 Node.js (ioredis)

### Install

```bash
npm install ioredis

```

### Usage

```jsx
importRedisfrom"ioredis";

const redis =newRedis({
host:"localhost",
port:6379,
password:"myStrongRedisPassword",
});

await redis.set("user","Chrissa");
const value =await redis.get("user");

console.log(value);

```

---

## 🔵 Laravel

### Install Redis Extension

```bash
composer require predis/predis

```

### `.env`

```
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=myStrongRedisPassword
REDIS_PORT=6379

```

### `config/database.php`

```php
'redis' => [
'client' =>'predis',
'default' => [
'host' =>env('REDIS_HOST'),
'password' =>env('REDIS_PASSWORD'),
'port' =>env('REDIS_PORT'),
'database' =>0,
    ],
],

```

### Usage

```php
useIlluminate\Support\Facades\Redis;

Redis::set('name','Chrissa');
$value =Redis::get('name');

```

---

## 🟣 Flutter

### Install

```bash
flutter pub add redis

```

### Usage

```dart
import 'package:redis/redis.dart';

final conn = RedisConnection();
final command = await conn.connect('127.0.0.1', 6379);

await command.send_object([
  'AUTH',
  'myStrongRedisPassword'
]);

await command.send_object([
  'SET',
  'name',
  'Chrissa'
]);

final value = await command.send_object([
  'GET',
  'name'
]);

print(value);

```

---

## 🧹 Maintenance Commands

### Flush all data

```bash
dockerexec -it redis redis-cli AUTH myStrongRedisPassword
FLUSHALL

```

### Stop Redis

```bash
docker-compose down

```

### Remove all Redis data (DANGER)

```bash
docker-compose down -v

```

---

## 🚫 Common Errors & Fixes

| Error | Fix |
| --- | --- |
| `Connection refused` | Redis not running |
| `NOAUTH Authentication required` | Run `AUTH` |
| `docker: command not found` | Run on host, not container |
| Data lost after restart | Missing volume |

---

## ⭐ Best Practices

- ✅ Always use **Redis official image**
- ✅ Enable **auth + persistence**
- ❌ Don’t install Redis manually in Ubuntu containers
- ✅ Use `docker-compose` for clarity

---

## 📌 Environment Variables (Recommended)

```
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=myStrongRedisPassword

```

---

## 📚 References

- https://redis.io/docs
- https://hub.docker.com/_/redis
- https://laravel.com/docs/redis

---

## 🏁 Summary

✔ Dockerized Redis

✔ Auth enabled

✔ Persistent storage

✔ Connected to Node, Laravel & Flutter
