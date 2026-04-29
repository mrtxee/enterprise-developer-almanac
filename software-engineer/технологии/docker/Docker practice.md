## Dockerfile

пример `Dockerfile`

```Dockerfile
# Используем официальный образ с JDK 21 и Maven
FROM maven:3.9-eclipse-temurin-21

# Устанавливаем рабочую директорию внутри контейнера
WORKDIR /app

# Копируем файлы проекта (pom.xml и исходный код)
# Предполагается, что структура проекта стандартная (src/, pom.xml и т.д.)
COPY . .

# Выполняем сборку Maven (без тестов)
RUN mvn clean package -DskipTests

EXPOSE 8080

# Перемещаем JAR-файл в нужное место (если он не там уже)
# По умолчанию Maven кладёт JAR в target/
RUN mv target/*.jar /app/iot-world-xee-sens-producer.jar

# Указываем точку входа — запуск JAR-файла
ENTRYPOINT ["java", "-jar", "/app/iot-world-xee-sens-producer.jar"]
```

> ⚠️ **Примечание**: Убедитесь, что имя JAR-файла в `target/` действительно совпадает с тем, что вы ожидаете. Если у вас в `pom.xml` указано `<finalName>iot-world-xee-sens-producer</finalName>`, то всё в порядке. В противном случае имя может отличаться (например, `my-app-1.0.jar`). В таком случае можно использовать более надёжный способ:

```Dockerfile
RUN find target -name "*.jar" -exec mv {} /app/iot-world-xee-sens-producer.jar \;
```


### EXPOSE

```shell
EXPOSE <port> [<port>...]
```

Инструкция `EXPOSE` указывает Docker что контейнер слушает определенные порты после запуска. **EXPOSE не делает порты контейнера доступными для хоста.** Для этого, вы должны **использовать флаг -p** (что бы открыть диапазон портов) или флаг -P что бы открыть все порты из EXPOSE. Можно задать один номер порта и пробросить его на другой внешний порт.

---

### Как запустить этот Dockerfile?

1. **Сохраните его** в корне вашего Maven-проекта под именем `Dockerfile`.

2. **Соберите Docker-образ** из той же директории:

```bash
docker build -t iot-producer .
```

> `-t iot-producer` — задаёт имя образа. Можно выбрать любое.

3. **Запустите контейнер**:

```bash
docker run --rm iot-producer
```

> `--rm` — автоматически удалит контейнер после завершения работы.

Если вашему приложению нужны порты, переменные окружения или другие параметры — добавьте их:

```bash
docker run --rm -p 8080:8080 -e ENV_VAR=value iot-producer
```

---

### Дополнительно: Multi-stage сборка (рекомендуется для продакшена)

Если вы хотите уменьшить размер финального образа (без Maven и исходного кода), используйте multi-stage:

```Dockerfile
# Stage 1: сборка
FROM maven:3.9-eclipse-temurin-21 AS builder
WORKDIR /app
COPY . .
RUN mvn clean package -DskipTests

# Stage 2: запуск
FROM eclipse-temurin:21-jre
WORKDIR /app
COPY --from=builder /app/target/*.jar /app/iot-world-xee-sens-producer.jar
ENTRYPOINT ["java", "-jar", "/app/iot-world-xee-sens-producer.jar"]
```

Этот подход создаёт гораздо более лёгкий образ, так как в финальном контейнере остаётся только JRE и JAR-файл.

Запуск — такой же:

```bash
docker build -t iot-producer .
docker run --rm iot-producer
```

Удачи! 🚀