---
aliases:
  - CDC
  - Database CDC
  - Database Change Data Capture
---
## Database-Based Solutions (Базы данных)

### Database Change Data Capture (CDC)

Пример реализации [[Publish-Subscribe]] в [[PostgreSQL]]

```sql

-- PostgreSQL LISTEN/NOTIFY
LISTEN user_events;

-- Уведомление
NOTIFY user_events, '{"type": "user_created", "data": {...}}';
```
