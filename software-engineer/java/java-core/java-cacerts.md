---
aliases:
  - cacerts
---
## Шаг 1: Получить сертификат сервера

**Linux / macOS / WSL:**

```bash
openssl s_client -connect db-node.test.sbt:443 -showcerts </dev/null 2>/dev/null \
  | openssl x509 -outform PEM > db-node-test-sbt.crt
```

Если в цепочке несколько сертификатов и нужен только корневой, посмотри вывод:

```bash
openssl s_client -connect xxxworks.ru:443 -showcerts </dev/null 2>/dev/null
```

Найди последний сертификат в выводе (обычно это корневой CA) и сохрани его.

**Windows (PowerShell, без openssl):**

```powershell
$resp = Invoke-WebRequest -Uri "https://xxxworks.ru/jira"
$cert = $resp.RawContent
# или через .NET:
$cert = [System.Net.HttpWebRequest]::Create("https://xxxworks.ru/jira").GetResponse().ServicePoint.Certificate
$bytes = $cert.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Cert)
[System.IO.File]::WriteAllBytes("xxxworks-ru.crt", $bytes)
```

---

## Шаг 2: Найти путь к `cacerts`

```bash
# Linux / macOS
echo $JAVA_HOME

# macOS (если JAVA_HOME не задан)
/usr/libexec/java_home
# cacerts будет в: $(/usr/libexec/java_home)/lib/security/cacerts
```

```powershell
# Windows
echo $env:JAVA_HOME
# cacerts будет в: %JAVA_HOME%\lib\security\cacerts
```

Пароль по умолчанию: `changeit`

Проверь, что файл существует:

```bash
ls -la $JAVA_HOME/lib/security/cacerts
# C:\Program Files\Eclipse Adoptium\jdk-17.0.14.7-hotspot\
```

---

## Шаг 3: Добавить сертификат в `cacerts`

```bash
keytool -import -trustcacerts \
  -alias xxxworks-ru \
  -file xxxworks-ru.crt \
  -keystore $JAVA_HOME/lib/security/cacerts \
  -storepass changeit \
  -noprompt
```

Флаг `-noprompt` пропускает интерактивное подтверждение. Если хочешь увидеть запрос «Trust this certificate? \[no]:» — убери `-noprompt` и введи `yes`.

---

## Шаг 4: Проверить, что сертификат добавлен

```bash
keytool -list -keystore $JAVA_HOME/lib/security/cacerts -storepass changeit | grep xxxworks-ru
```

Вывод должен показать что-то вроде:

```
xxxworks-ru, 15 сент. 2026, trustedCertEntry, ...
```

---

## Шаг 5: Проверить соединение

```bash
# Простой тест через keytool
keytool -printcert -sslserver xxxworks.ru:443 -rfc
```

Или запусти тестовое соединение из Java:

```java
import java.net.URL;
import java.net.HttpURLConnection;

public class SslTest {
    public static void main(String[] args) throws Exception {
        var url = new URL("https://xxxworks.ru/jira");
        var conn = (HttpURLConnection) url.openConnection();
        conn.setRequestMethod("GET");
        System.out.println("Response code: " + conn.getResponseCode());
    }
}
```

Если раньше падало с `PKIX path building failed`, а теперь `Response code: 200` (или `401`/`403` — если нужна авторизация) — сертификат добавлен правильно.

---

## Если используешь Docker

В Docker-контейнере `cacerts` обычно в `/opt/java/openjdk/lib/security/cacerts` (зависит от базового образа). Добавь в `Dockerfile`:

```dockerfile
COPY xxxworks-ru.crt /tmp/xxxworks-ru.crt
RUN keytool -import -trustcacerts \
    -alias xxxworks-ru \
    -file /tmp/xxxworks-ru.crt \
    -keystore $JAVA_HOME/lib/security/cacerts \
    -storepass changeit \
    -noprompt \
 && rm /tmp/xxxworks-ru.crt
```

---

## Если не хочешь трогать системный `cacerts`

Создай отдельный truststore и подключи его через JVM-аргументы:

```bash
# Создать новый truststore с сертификатом
keytool -import -trustcacerts \
  -alias xxxworks-ru \
  -file xxxworks-ru.crt \
  -keystore my-truststore.jks \
  -storepass mypassword \
  -noprompt
```

Запуск приложения:

```bash
java \
  -Djavax.net.ssl.trustStore=/path/to/my-truststore.jks \
  -Djavax.net.ssl.trustStorePassword=mypassword \
  -jar app.jar
```

В Spring Boot через `application.yml`:

```yaml
# Это для встроенного сервера, не для исходящих HTTP-клиентов
server:
  ssl:
    trust-store: classpath:my-truststore.jks
    trust-store-password: mypassword
```

Для исходящих клиентов (RestTemplate, JIRA REST Client) JVM-аргументы `-Djavax.net.ssl.trustStore` работают глобально — все HTTP-клиенты на стандартном `SSLSocketFactory` будут использовать этот truststore.

---

Если после добавления сертификата ошибка сохранится, возможно, в цепочке не хватает промежуточного сертификата. В этом случае скачай всю цепочку и добавь каждый сертификат отдельно с разными алиасами.

## JVM variables set
```java
System.setProperty("javax.net.ssl.trustStore", "/path/to/truststore.jks"); System.setProperty("javax.net.ssl.trustStorePassword", "changeit");
-Djavax.net.ssl.trustStore=/Users/saparin.a.a/Documents/cert/cacerts  
-Djavax.net.ssl.trustStorePassword=123456
```
