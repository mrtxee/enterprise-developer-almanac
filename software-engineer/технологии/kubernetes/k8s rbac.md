Компоненты RBAC:

- **Role (роль)** — набор разрешений, который определяет, какие действия можно выполнять над ресурсами в определённом неймспейсе.
- **ClusterRole** аналогичен Role, но применяется ко всему кластеру. Используется для управления ресурсами, которые не привязаны к определённому неймспейсу, например `nodes` или `persistentvolumes`.
- **RoleBinding** связывает пользователя, группу или сервисный аккаунт с ролью (Role) в конкретном неймспейсе и предоставляет разрешения, которые определены в роли.
	- свяжем созданную роль с конкретным пользователем или сервисным аккаунтом
- **ClusterRoleBinding** аналогичен RoleBinding, но применяется ко всему кластеру. Связывает пользователя, группу или сервисный аккаунт с кластерной ролью (ClusterRole).

При определении полномочий роли (role) Kubernetes оперирует глаголом (verb) и выделяет две категории глаголов:
- Для права на запись (write) — `create`, `update`, `patch`
- Для права на чтение (read) — `get`, `list`, `watch`

Отлично!  
**RBAC (Role-Based Access Control)** в **Kubernetes** — это **система управления доступом**, которая определяет, кто и что может делать в кластере.

---

## ✅ Что такое Kubernetes RBAC?

> **RBAC** — это механизм, который отвечает на вопросы:
- Кто ты? → **Аутентификация**
- Что ты можешь сделать? → **Авторизация через роли**

### 🔹 Пример:
```yaml
"Пользователь: dev-team"
→ Может читать Pod'ы в namespace `dev`
→ Не может трогать `kube-system`
```

---

## 🔧 Основные компоненты RBAC

| Компонент                            | Описание                           |
| ------------------------------------ | ---------------------------------- |
| **User / Group / ServiceAccount**    | Кто запрашивает доступ             |
| **Role / ClusterRole**               | Что разрешено (правила)            |
| **RoleBinding / ClusterRoleBinding** | Связывает субъект с ролью          |
| **API Server**                       | Проверяет каждый запрос через RBAC |

---

## ✅ 1. Субъекты (Subjects)

Кто хочет получить доступ:

| Тип | Пример |
|-----|--------|
| **User** | `alice@company.com`, `admin` |
| **Group** | `developers`, `system:masters` |
| **ServiceAccount** | `default`, `my-app-sa` (для подов) |

> 💡 Внутри кластера чаще используются **ServiceAccounts**, а не Users.

---

## ✅ 2. Роли: Role и ClusterRole

### 🟦 **Role** — для одного Namespace

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
meta
  namespace: dev
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

→ Разрешает читать Pod'ы только в `namespace: dev`

---

### 🟨 **ClusterRole** — для всего кластера

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
meta
  name: node-admin
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["*"]  # все действия
```

→ Управляет узлами кластера

---

## ✅ 3. Привязки: Binding

### 🟦 **RoleBinding** — связывает в одном namespace

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
meta
  namespace: dev
  name: read-pods
subjects:
- kind: User
  name: alice@company.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

→ Alice может читать Pod'ы в `dev`

---

### 🟨 **ClusterRoleBinding** — глобальная привязка

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
meta
  name: cluster-read-all
subjects:
- kind: Group
  name: developers
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: view
  apiGroup: rbac.authorization.k8s.io
```

→ Все из группы `developers` могут просматривать ресурсы во всех namespaces

---

## ✅ 4. Глаголы (Verbs) — что можно делать?

| Глагол | Что означает |
|--------|------------|
| `get` | Получить объект |
| `list` | Перечислить все |
| `create` | Создать |
| `update` | Обновить |
| `delete` | Удалить |
| `watch` | Отслеживать изменения |
| `patch` | Частичное обновление |
| `*` | Все глаголы |

---

## ✅ 5. API Groups и Resources

| Поле | Примеры |
|------|---------|
| `apiGroups: [""]` | Core API (Pod, Service) |
| `apiGroups: ["apps"]` | Deployments, StatefulSets |
| `apiGroups: ["networking.k8s.io"]` | Ingress |
| `resources: ["pods"]` | Pod'ы |
| `resources: ["secrets"]` | Секреты (опасно!) |
| `resources: ["deployments"]` | Деплои |

---

## 🆚 Role vs ClusterRole

| | **Role** | **ClusterRole** |
|--|---------|---------------|
| **Область действия** | Один namespace | Весь кластер |
| **Используется для** | Разработчиков, приложений | Администраторов, операторов |
| **Пример** | Dev читает свои Pod'ы | SRE управляет Node'ами |

---

## 🆚 RoleBinding vs ClusterRoleBinding

| | **RoleBinding** | **ClusterRoleBinding** |
|--|----------------|-----------------------|
| **Сфера** | Один namespace | Весь кластер |
| **Связь с Role** | ✅ Да | ✅ Да |
| **Связь с ClusterRole** | ✅ Да (в пределах ns) | ✅ Да (глобально) |
| **Рекомендация** | Для команды | Для админов |

---

## ✅ Пример: Dev команда

```yaml
# role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
meta
  namespace: frontend
  name: frontend-dev-role
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "patch", "update"]
```

```yaml
# binding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
meta
  namespace: frontend
  name: dev-team-binding
subjects:
- kind: Group
  name: frontend-developers
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: frontend-dev-role
  apiGroup: rbac.authorization.k8s.io
```

→ Группа `frontend-developers` может работать в `namespace: frontend`

---

## ✅ Пример: Только просмотр (read-only)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
meta
  name: custom-view
rules:
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  verbs: ["get", "list", "watch"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]
```

> ⚠️ Будьте осторожны с `secrets`!

---

## ✅ Как проверить права?

```bash
kubectl auth can-i get pods --as alice@company.com -n dev
# yes/no

kubectl auth can-i create deployments --as bob@company.com -n prod
# no
```

---

## ✅ Лучшие практики

| Правило | Объяснение |
|--------|------------|
| ✅ **Начинайте с минимальных прав** | `view`, а не `edit` или `admin` |
| ✅ **Используйте ServiceAccount для подов** | Не User |
| ✅ **Не давайте доступ к secrets без причины** | Это как пароль от всех систем |
| ✅ **Используйте Namespace-изоляцию** | `dev`, `prod`, `staging` |
| ✅ **Группируйте по team/role** | `backend-team`, `db-admins` |
| ✅ **Регулярно аудитите** | `kubectl get roles,rolebindings --all-namespaces` |
| ✅ **Включите audit-логи** | Чтобы видеть, кто что делал |

---

## ❌ Распространённые ошибки

| Ошибка | Последствия |
|--------|-------------|
| `cluster-admin` для всех | Полный доступ к кластеру → риск |
| Нет изоляции между командами | Одна команда ломает другую |
| Доступ к secrets | Утечка паролей, ключей |
| Использование `*` в rules | Невозможно контролировать |
| Нет аудита | Не знаете, кто и что делал |

---

## ✅ Финальный вывод

| RBAC позволяет вам... | Без него вы... |
|----------------------|--------------|
| ✅ Ограничить доступ по namespace | ❌ Все видят всё |
| ✅ Дать dev-команде только нужные права | ❌ Либо полный доступ, либо ничего |
| ✅ Защитить продакшен | ❌ QA случайно удалит production DB |
| ✅ Соответствовать стандартам | ❌ Нарушение PCI-DSS, GDPR |

> 💬 _“With great power comes great responsibility.”_  
> — Но в K8s: _“Without RBAC — you have no control.”_

---

## 📚 Где учиться дальше?

- [Официальная документация](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- YouTube: *“Kubernetes RBAC Explained”* — TechWorld with Nana
- Book: *“Kubernetes in Action”* — Marko Luksa

---

✅ **RBAC — основа безопасности в Kubernetes.**  
Если вы не используете его — ваш кластер **открыт всем**.

> 🔐 Настройте RBAC — и вы перестанете бояться, что кто-то случайно удалит `etcd`.

---

📌 **Готово!**  
Теперь вы можете создавать **надёжную модель доступа** в любом k8s-кластере.