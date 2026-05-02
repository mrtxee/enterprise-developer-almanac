**`WebMvcConfigurer`** — это интерфейс в **Spring MVC**, который позволяет **настраивать и кастомизировать** работу Spring MVC **без отключения автоконфигурации**.

---

## ✅ Зачем он нужен?

Когда вы используете **Spring Boot** с **Spring MVC**, фреймворк автоматически настраивает:

- `DispatcherServlet`
- Обработчики запросов (`@RestController`, `@Controller`)
- Сериализацию JSON/XML
- Статические ресурсы
- ViewResolver и т.д.

Но иногда нужно **добавить свою логику**, например:

| Задача | Как решить |
|--------|------------|
| ✅ Добавить **интерцепторы** | `addInterceptors()` |
| ✅ Настроить **CORS** | `addCorsMappings()` |
| ✅ Добавить **обработчики статических ресурсов** | `addResourceHandlers()` |
| ✅ Зарегистрировать **ViewController** | `addViewControllers()` |
| ✅ Добавить **резолверы аргументов** | `addArgumentResolvers()` |
| ✅ Настроить **конвертеры сообщений** | `configureMessageConverters()` |
| ✅ Добавить **форматтеры** | `addFormatters()` |
| ✅ Настроить **content negotiation** | `configureContentNegotiation()` |

---

## ✅ Пример: Реализация `WebMvcConfigurer`

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.*;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    // 1. Добавить интерцептор
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyInterceptor())
                .addPathPatterns("/api/**")
                .excludePathPatterns("/api/public/**");
    }

    // 2. Настроить CORS
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOrigins("https://example.com")
                .allowedMethods("GET", "POST", "PUT", "DELETE");
    }

    // 3. Добавить обработчики статических ресурсов
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/**")
                .addResourceLocations("classpath:/static/");
    }

    // 4. Зарегистрировать ViewController (без контроллера)
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/home").setViewName("home");
    }

    // 5. Настроить конвертеры сообщений
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        // Добавить свой конвертер
        converters.add(new MyCustomConverter());
    }
}
```

---

## ✅ Важно: `WebMvcConfigurer` vs `WebMvcConfigurationSupport`

| Класс | Описание |
|-------|----------|
| ✅ **`WebMvcConfigurer`** | **Добавляет** настройки **поверх** автоконфигурации (рекомендуется) |
| ❌ **`WebMvcConfigurationSupport`** | **Отключает** автоконфигурацию Spring Boot (не рекомендуется) |

> ⚠️ **Никогда не расширяйте `WebMvcConfigurationSupport`**, если используете Spring Boot — это **отключит автоконфигурацию**!

---

## ✅ Когда использовать `WebMvcConfigurer`?

| Сценарий | Пример |
|----------|--------|
| ✅ Нужно добавить **интерцептор** | Логирование, аутентификация, аудит |
| ✅ Нужно настроить **CORS** | Для фронтенда на другом домене |
| ✅ Нужно добавить **статические ресурсы** | CSS, JS, изображения |
| ✅ Нужно зарегистрировать **простой view** | Без контроллера |
| ✅ Нужно добавить **свой конвертер** | Кастомная сериализация |
| ✅ Нужно настроить **форматтеры** | Дата, число, валюта |

---

## ✅ Резюме:

| Вопрос | Ответ |
|--------|-------|
| ✅ Что такое `WebMvcConfigurer`? | Интерфейс для **кастомизации Spring MVC** |
| ✅ Отключает ли автоконфигурацию? | ❌ **Нет** — добавляет настройки поверх |
| ✅ Когда использовать? | Когда нужно **добавить** логику, а не **заменить** |
| ✅ Чем отличается от `WebMvcConfigurationSupport`? | `WebMvcConfigurationSupport` **отключает** автоконфигурацию |

---

✅ **Готово!**  
Теперь вы знаете, **для чего нужен `WebMvcConfigurer`** и **как его использовать** в Spring Boot.