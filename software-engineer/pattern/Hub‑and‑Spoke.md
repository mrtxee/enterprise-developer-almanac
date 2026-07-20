---
aliases:
  - Hub‑and‑Spoke
  - Hub
  - Spoke
  - Spoke‑клиент
  - Spoke‑client
---
Hub‑and‑Spoke – паттерн, применяемый при интеграционном тестировании.
* **архитектурный паттерн или термин из области тестирования производительности / интеграционных тестов** — хотя сам термин не является стандартным в JUnit или Mockito.

Если вы встретили слово **Spoke** в контексте тестирования, скорее всего, речь идёт об одном из двух:

1. **Тестирование в распределённых системах (Hub‑and‑Spoke)** — когда тесты проверяют взаимодействие между центральным узлом (Hub) и периферийными (Spoke). Например, тестирование очередей сообщений, маршрутизаторов или ESB.
2. **Тестирование сетевых протоколов** — Spoke‑клиенты, имитирующие реальных потребителей API.

В большинстве случаев **Spoke** — это не библиотека и не аннотация, а **роль** компонента в архитектуре.

---

### 📦 Базовый пример в Java (имитация Spoke‑клиента)

Представим, что мы тестируем систему, где есть центральный сервис (`Hub`) и несколько клиентских приложений (`Spoke`). Мы хотим проверить, что все клиенты корректно получают обновления.

```java
// SpokeClient — имитация клиента, который получает данные от хаба
public class SpokeClient {
    private final String name;
    private final HubService hub;

    public SpokeClient(String name, HubService hub) {
        this.name = name;
        this.hub = hub;
    }

    public void fetchData() {
        String data = hub.getDataForClient(name);
        System.out.println("Клиент " + name + " получил: " + data);
    }
}

// Тест: несколько Spoke‑клиентов обращаются к Hub
public class HubSpokeTest {
    @Test
    public void testSpokeClientsReceiveData() {
        HubService hub = new HubService();
        SpokeClient client1 = new SpokeClient("spoke-1", hub);
        SpokeClient client2 = new SpokeClient("spoke-2", hub);

        client1.fetchData();
        client2.fetchData();

        // Проверяем, что оба клиента получили свои данные
        assertNotNull(client1.getLastReceived());
        assertNotNull(client2.getLastReceived());
    }
}
```

---

### 🧩 Когда это используется?

- **Интеграционные тесты** нескольких сервисов.
- **Тестирование очередей** (JMS, Kafka) — продюсеры/консумеры.
- **Нагрузочное тестирование** — много клиентов, которые бьют в один эндпоинт.

---