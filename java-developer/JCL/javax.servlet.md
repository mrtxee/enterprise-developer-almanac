---
aliases:
  - Servlet API
  - Servlet
  - сервелет
---
**Servlet API** — это **стандарт Java-интерфейсов и классов** для построения веб-приложений, работающих на стороне сервера. Он определяет, как Java-код взаимодействует с HTTP-запросами и ответами.

> 💡.Servlet — это **Java-класс**, который обрабатывает HTTP-запросы (GET, POST и др.) и формирует HTTP-ответы.

---

## 🧱 Основы Servlet API

### 1. **Что делает сервлет?**
- Принимает `HttpServletRequest` (данные от клиента: URL, заголовки, тело, параметры).
- Обрабатывает логику (бизнес-логика, работа с БД).
- Формирует `HttpServletResponse` (HTML, JSON, статус-код и т.д.).

### 2. **Главный интерфейс: `javax.servlet.Servlet`**
```java
public interface Servlet {
    void init(ServletConfig config) throws ServletException;
    void service(ServletRequest req, ServletResponse res) throws ServletException, IOException;
    void destroy();
    // ...
}
```

- `init()` — вызывается **один раз** при запуске (инициализация).
- `service()` — вызывается **на каждый запрос**.
- `destroy()` — при остановке приложения.

> В реальности чаще наследуются от `HttpServlet` (абстрактный класс из `javax.servlet.http`).

---

## 🔄 Жизненный цикл сервлета

1. **Загрузка** → контейнер создаёт экземпляр сервлета.
2. **Инициализация** → вызов `init()`.
3. **Обработка запросов** → многократный вызов `service()`.
4. **Уничтожение** → вызов `destroy()` при остановке.

> ⚠️ **Один экземпляр сервлета обслуживает множество запросов** → должен быть **потокобезопасным**!

---

## 📦 Пакеты Servlet API

| Пакет | Назначение |
|-------|-----------|
| `javax.servlet` | Базовые интерфейсы (`Servlet`, `ServletRequest`) |
| `javax.servlet.http` | HTTP-специфичные классы (`HttpServlet`, `HttpServletRequest`) |

> 🔹 В Java EE / Jakarta EE этот API входит в **спецификацию Servlet**, реализуемую веб-контейнерами (Tomcat, Jetty, WildFly и др.).

---

## 🌐 Пример простого сервлета

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) 
            throws IOException {
        resp.setContentType("text/plain");
        resp.getWriter().println("Hello from Servlet!");
    }
}
```

→ При запросе `GET /hello` сервер вернёт текст `Hello from Servlet!`.

---

## 🏗️ Где используется Servlet API?

### 1. **Веб-фреймворки**
- **Spring MVC** — под капотом использует сервлеты (главный — `DispatcherServlet`).
- **Struts**, **Vaadin**, **JSF** — тоже построены поверх Servlet API.

### 2. **Веб-контейнеры**
- **Tomcat**, **Jetty**, **Undertow** — реализуют Servlet API.
- Они:
  - Принимают HTTP-запросы
  - Создают `HttpServletRequest`/`HttpServletResponse`
  - Вызывают нужный сервлет

### 3. **WAR-приложения**
- Java-веб-приложения упаковываются в **WAR** (Web Application Archive).
- Развертываются в контейнер, поддерживающий Servlet API.

---

## 🆚 Servlet API vs. Реактивные фреймворки (WebFlux)

| Характеристика | Servlet API | Spring WebFlux |
|----------------|-------------|----------------|
| **Модель** | Блокирующая, thread-per-request | Неблокирующая, event-loop |
| **Потоки** | Много потоков (1 на запрос) | Мало потоков (ядер CPU + N) |
| **Масштабируемость** | Ограничена памятью/потоками | Высокая при I/O-нагрузке |
| **Совместимость** | Все Java-веб-серверы | Требует реактивный сервер (Netty) |
| **Стандарт** | Jakarta EE (ранее Java EE) | Реактивный стек (Reactor) |

> 🔸 Spring WebFlux **не требует Servlet API** — он может работать на Netty напрямую.  
> 🔸 Spring MVC **обязательно требует Servlet API**.

---

## 📜 Эволюция: от `javax` к `jakarta`

- До 2017: пакеты — `javax.servlet.*`
- После передачи Java EE в Eclipse Foundation:  
  → переименовано в **Jakarta Servlet** → пакеты `jakarta.servlet.*`
- **Tomcat 10+**, **Jetty 11+** используют `jakarta.*`

> ⚠️ Это **не совместимо на уровне байт-кода** — миграция требует перекомпиляции.

---

## ✅ Зачем знать Servlet API?

1. Понимать, **как работают веб-фреймворки** (Spring, и др.).
2. Отлаживать **низкоуровневые проблемы** (фильтры, сессии, заголовки).
3. Писать **кастомные фильтры, слушатели, сервлеты**.
4. Выбирать архитектуру: **традиционная (Servlet)** vs **реактивная (WebFlux)**.

---

## 🎯 Итог

> **Servlet API — это фундамент Java-веба.**  
> Он стандартизирует взаимодействие между веб-сервером и Java-кодом, позволяя писать переносимые веб-приложения.  
> Даже если вы используете Spring Boot, под капотом — **всё равно сервлеты** (если не WebFlux).

---

Хочешь пример с фильтрами, слушателями, или как Spring MVC использует `DispatcherServlet`? Скажи — объясню! 😊