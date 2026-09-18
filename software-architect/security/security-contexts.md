---
aliases:
  - Conditional Access
  - MFA
  - Risk Score
  - Security Context
  - Security Contexts
  - SIEM
  - XDR
  - Zero Trust
  - Адаптивная аутентификация
  - Контекст безопасности
  - Контексты безопасности
  - Многофакторная аутентификация
  - Нулевое доверие
  - Оценка риска
  - Условный доступ
---

## Контексты безопасности (Security Contexts)

> **Контекст безопасности** — это **набор условий**, при которых выполняется операция:
> - Кто?
> - Откуда?
> - Какой уровень доступа?
> - Устройство доверенное?
> - Есть MFA?

Это основа **[[zero-trust|Zero Trust]]** и **[[adaptive-auth|Adaptive auth]]**.

### 💡 Пример контекста

```json
{
  "user": "alice@company.com",
  "ip": "192.168.1.100",
  "device": "macbook-pro-trusted",
  "location": "Москва",
  "time": "10:30",
  "mfa_verified": true,
  "risk_score": 0.2
}
```

### Где используется

- **Проверка доступа**: если `risk_score > 0.7` → требовать MFA
- **[[siem|SIEM]] / XDR**: корреляция событий
- **API Gateway**: разрешить/запретить вызов
- **[[cloud-security|Cloud Security]]**: AWS IAM, Azure AD Conditional Access

### 🔄 Аналогия

> **Контекст безопасности — это «медицинская карта» пользователя**
> На основе её принимается решение:
> _«Можно ли ему входить?»_
