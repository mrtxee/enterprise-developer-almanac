---
aliases:
  - Spring
  - Spring Security
---
## 📘 Spring Security: Полное руководство

**Spring Security** — это enterprise-фреймворк для **аутентификации** (кто ты?) и **авторизации** (что тебе можно?) в Java/Spring-приложениях. Это не просто библиотека, а целая архитектура фильтров, менеджеров и контекстов, которая глубоко интегрирована в Spring MVC, WebFlux и Boot.

---

## 🏗 Архитектура: как работает "под капотом"

Spring Security строится вокруг **цепочки фильтров (`SecurityFilterChain`)** и **контекста безопасности (`SecurityContext`)**.

```
HTTP Request
     ↓
┌─────────────────────────────────────┐
│         Filter Chain Proxy          │ ← Делегирует запрос нужной цепочке
└────────────────┬────────────────────┘
                 ↓
┌─────────────────────────────────────┐
│  1. CORS Filter                     │ (обработка preflight)
│  2. CSRF Filter                     │ (проверка токена для stateful)
│  3. Authentication Filter           │ (извлечение credentials: Basic, JWT, Form)
│  4. Authorization Filter            │ (проверка прав: hasRole, hasAuthority)
│  5. ExceptionTranslationFilter      │ (преобразование 401/403 в HTTP-ответ)
│  6. FilterSecurityInterceptor       │ (финальная проверка доступа к URL/методу)
└────────────────┬────────────────────┘
                 ↓
         Controller / Handler
```

### 🔑 Ключевые компоненты:

| Компонент                       | Назначение                                                                                                   |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `SecurityFilterChain`           | Конфигурируемая цепочка фильтров. Может быть несколько (например, `/api/**` → JWT, `/admin/**` → Form Login) |
| `AuthenticationManager`         | Координирует проверку credentials через `AuthenticationProvider` (Password, JWT, OAuth2, LDAP)               |
| `SecurityContext`               | `ThreadLocal`-хранилище текущего `Authentication`. Привязан к потоку, очищается после ответа                 |
| `AuthorizationManager` (Sec 6+) | Решает, разрешить ли доступ. Заменяет старый `AccessDecisionManager` + `Voter`                               |
| `UserDetailsService`            | Загружает `UserDetails` из БД/LDAP по username                                                               |

---

## 🔑 Ключевые концепции

| Термин | Что означает | Пример |
|--------|--------------|--------|
| **Authentication** | Процесс проверки личности | Логин/пароль, JWT, сертификат, OAuth2 token |
| **Authorization** | Проверка прав на ресурс | `hasRole('ADMIN')`, `hasAuthority('read:loans')` |
| **Principal** | Объект аутентифицированного пользователя | `UserDetails`, `JwtAuthenticationToken` |
| **GrantedAuthority** | Право/роль | `ROLE_USER`, `ROLE_ADMIN`, `SCOPE_read` |
| **Stateful** | Сессия хранится на сервере (`JSESSIONID`) | Form Login, Thymeleaf UI |
| **Stateless** | Сессия не хранится, токен в каждом запросе | JWT, API-клиенты, мобильные приложения |

> 💡 Роли в Spring Security **всегда начинаются с `ROLE_`**. Метод `hasRole('ADMIN')` автоматически ищет `ROLE_ADMIN`.

---

## ⚙️ Современная настройка (Spring Security 6+ / Spring Boot 3)

В Spring Security 6 **удалён `WebSecurityConfigurerAdapter`**. Конфигурация теперь через `@Bean SecurityFilterChain` с Lambda DSL.

### 🔹 Базовый конфиг для REST API (JWT / Stateless)
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@EnableWebSecurity
@EnableMethodSecurity  // ✅ Включаем @PreAuthorize, @PostAuthorize
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            // 1. Отключаем CSRF для stateless API
            .csrf(csrf -> csrf.disable())
            
            // 2. Stateless сессии
            .sessionManagement(sess -> sess.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            
            // 3. Настройка доступа к эндпоинтам
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/actuator/health", "/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            
            // 4. OAuth2 Resource Server (JWT) или кастомный фильтр
            .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()))
            
            // 5. Обработка ошибок аутентификации/авторизации
            .exceptionHandling(ex -> ex
                .authenticationEntryPoint(new BearerTokenAuthenticationEntryPoint())
                .accessDeniedHandler(new BearerTokenAccessDeniedHandler())
            )
            
            .build();
    }
}
```

### 🔹 Method Security (защита на уровне методов)
```java
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.stereotype.Service;

@Service
public class LoanService {

    @PreAuthorize("hasRole('LOAN_OFFICER') or #loan.ownerId == authentication.principal.id")
    public Loan approveLoan(Long loanId) { ... }

    @PostAuthorize("returnObject.status == 'APPROVED' ? hasRole('RISK_MANAGER') : true")
    public Loan getLoanDetails(Long loanId) { ... }
}
```

---

## 🛡 Типовые сценарии

| Сценарий | Конфигурация | Когда использовать |
|----------|--------------|-------------------|
| ✅ **REST API + JWT** | `oauth2ResourceServer().jwt()` + `STATELESS` | Микросервисы, мобильные приложения, SPA |
| ✅ **Form Login + Session** | `formLogin(Customizer.withDefaults())` + `CSRF enabled` | Веб-админки, legacy UI, внутренние порталы |
| ✅ **OAuth2 / OIDC (SSO)** | `oauth2Login()` + Keycloak/Auth0/Google | Единый вход, корпоративные SSO |
| ✅ **m2m / Client Credentials** | `oauth2Client()` + `JwtDecoder` | Сервис-сервис, background jobs |
| ✅ **LDAP / AD** | `ldapAuthentication()` | Enterprise-интеграция с Windows AD |
| ✅ **Кастомный токен** | Кастомный `AbstractAuthenticationProcessingFilter` | Проприетарные схемы, legacy gateway |

---

## 🚨 Частые ошибки & Best Practices

| ❌ Ошибка | 💡 Решение |
|-----------|------------|
| CSRF включён для REST API → `403 Forbidden` на `POST/PUT/DELETE` | `.csrf(csrf -> csrf.disable())` для stateless API |
| `@Async` теряет `SecurityContext` → `Authentication = null` | Использовать `DelegatingSecurityContextExecutor` или `SecurityContextTaskDecorator` |
| `@PreAuthorize` не работает | Забыли `@EnableMethodSecurity` или вызываете метод через `this.` внутри того же класса |
| Пароли хранятся в plain text | Использовать `PasswordEncoder` (BCrypt/Argon2). Spring Boot автоматически поддерживает `{bcrypt}...`, `{noop}...` |
| Неправильный порядок `SecurityFilterChain` | Использовать `@Order(1)`, `@Order(2)`. Первый совпавший `requestMatchers` побеждает |
| `open-in-view: true` + Security → lazy loading в транзакции авторизации | `spring.jpa.open-in-view=false`. Загружать данные явно или в `@Transactional` |
| Логирование `401/403` не помогает | `logging.level.org.springframework.security=DEBUG` + `ExceptionTranslationFilter` логирование |

### 🔍 Отладка цепочки фильтров
```java
@Configuration
public class DebugSecurityConfig {
    @Bean
    public Filter debugFilter() {
        return new Filter() {
            @Override
            public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) 
                    throws IOException, ServletException {
                System.out.println("🔍 Current SecurityContext: " + SecurityContextHolder.getContext());
                chain.doFilter(req, res);
            }
        };
    }
}
```

---

## 📊 Spring Security 5 vs 6 (Миграция)

| Spring Security 5 | Spring Security 6+ (Boot 3) |
|-------------------|-----------------------------|
| `WebSecurityConfigurerAdapter` | Удалён. Используйте `@Bean SecurityFilterChain` |
| `authorizeRequests()` | `authorizeHttpRequests()` |
| `AccessDecisionManager` / `Voter` | `AuthorizationManager` |
| `@EnableGlobalMethodSecurity` | `@EnableMethodSecurity` |
| `antMatchers()` | `requestMatchers()` |
| `httpBasic()` включён по умолчанию | Отключён по умолчанию, нужно явно включать |

---

## ✅ Production-чеклист

| ✅ | Действие |
|---|----------|
| 🔹 | Stateless API → `SessionCreationPolicy.STATELESS` + отключить CSRF |
| 🔹 | Использовать `AuthorizationManager` (`authorizeHttpRequests`) вместо `AccessDecisionManager` |
| 🔹 | Пароли → `PasswordEncoder` (BCrypt/Argon2). Никогда `noop` в prod |
| 🔹 | CORS настраивать **до** Security или через `http.cors()` |
| 🔹 | `@Async` + Security → `DelegatingSecurityContextExecutor` |
| 🔹 | Method Security → `@EnableMethodSecurity` + `@PreAuthorize` |
| 🔹 | Логирование ошибок → кастомный `AuthenticationEntryPoint` / `AccessDeniedHandler` |
| 🔹 | Тестирование → `@WithMockUser`, `MockMvc` + `SecurityMockMvcRequestPostProcessors` |

---

## 💡 Итог

| Вопрос | Ответ |
|--------|-------|
| ✅ Что делает Spring Security? | Аутентификация + авторизация через цепочку фильтров и контекст безопасности |
| ✅ Как настроить REST API с JWT? | `oauth2ResourceServer().jwt()` + `STATELESS` + отключить CSRF |
| ✅ Как защитить методы? | `@EnableMethodSecurity` + `@PreAuthorize("hasRole('ADMIN')")` |
| ✅ Почему `@Async` теряет контекст? | `SecurityContext` хранится в `ThreadLocal`. Нужно явно передавать или использовать `DelegatingSecurityContextExecutor` |
| ✅ Что изменилось в Sec 6? | Убран `WebSecurityConfigurerAdapter`, введён `AuthorizationManager`, Lambda DSL, `requestMatchers()` |

Spring Security — это **стандарт де-факто** для Java/Spring. При правильной настройке он покрывает 99% сценарий: от простых форм входа до распределённых OAuth2/OIDC систем с JWT и method-level авторизацией.

Нужен готовый шаблон:

- 🔹 JWT Resource Server + Keycloak?
- 🔹 Form Login + LDAP + CSRF?
- 🔹 Тесты с `@WithMockUser` + `SecurityMockMvc`?

Напишите ваш сценарий — подготовлю production-ready код с пояснениями.
