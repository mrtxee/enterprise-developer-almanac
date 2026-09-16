---
aliases:
  - CDC
  - Change Data Capture
  - Database CDC
  - Database Change Data Capture
---
## Database-Based Solutions (Базы данных)

### Database Change Data Capture (CDC)

Пример реализации [[publish-subscribe|Publish-subscribe]] в [[PostgreSQL|PostgreSQL]]

```sql
-- PostgreSQL LISTEN/NOTIFY
LISTEN user_events;

-- Уведомление
NOTIFY user_events, '{"type": "user_created", "data": {...}}';
```
