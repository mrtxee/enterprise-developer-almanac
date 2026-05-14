**GraphQL** — это язык запросов для API и runtime для их выполнения. Он применяется, когда REST API становится недостаточно гибким или эффективным.

**Кэшировани GraphQL**
1. Кеширование на уровне полей
2. Кеширование по хешу запроса
3. Использование постоянных запросов
4. Кеширование на клиенте

**реализации**
* Hasura GraphQL Engine
* Apollo Federation

**GraphQL Subscription**
Есть возможность получать уведомления об изменениях по инициативе сервера при помощи GraphQL API посредством «мутаций». Самая популярная реализация этой спецификации использует «под капотом» **Websockets**,

## 🎯 Когда применять GraphQL?

### 1. **Сложные данные с множеством связей**

```json
# Один запрос вместо нескольких REST вызовов
query {
  user(id: "123") {
    name
    email
    posts(limit: 5) {
      title
      comments(limit: 3) {
        text
        author {
          name
        }
      }
    }
    friends {
      name
      mutualFriends {
        name
      }
    }
  }
}
```

### 2. **Мобильные приложения с ограниченным трафиком**

```graphql
# Только нужные поля вместо полных объектов
query MobileUserProfile {
  user(id: "123") {
    name
    avatar(size: SMALL)
    onlineStatus
  }
}
```

### 3. **Агрегация данных из multiple источников**

```graphql
# Данные из разных микросервисов в одном запросе
query DashboardData {
  user: userService.user(id: "123") {
    name
  }
  orders: orderService.orders(userId: "123") {
    total
  }
  notifications: notificationService.unreadCount(userId: "123")
}
```

### 4. **Часто меняющиеся requirements клиентов**

```graphql
# Клиенты сами определяют какие данные им нужны
query CustomReport {
  products(category: "electronics") {
    name
    price
    reviews {
      rating
      comment
      createdAt
    }
    inventory {
      stock
      warehouse
    }
  }
}
```

### 5. **Real-time обновления**

```graphql
# Подписка на изменения
subscription {
  postLikes(postId: "456") {
    likeCount
    recentLikers {
      name
    }
  }
}
```

---

## 🛠️ Как применять GraphQL?

### 1. **Схема и типы (Schema First)**
# Schema definition
```graphql
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  comments: [Comment!]!
}

type Query {
  user(id: ID!): User
  posts(userId: ID!): [Post!]!
}

type Mutation {
  createPost(input: CreatePostInput!): Post!
}
```
### 2. **Резолверы (Resolvers)**

```javascript
// Resolver implementation
const resolvers = {
  Query: {
    user: async (parent, { id }, context) => {
      return context.db.user.findUnique({ where: { id } });
    }
  },
  User: {
    posts: async (parent, args, context) => {
      return context.db.post.findMany({ 
        where: { authorId: parent.id } 
      });
    }
  }
};
```

### 3. **Интеграция с существующей инфраструктурой**

```javascript
// Apollo Server with Express
const { ApolloServer } = require('apollo-server-express');
const express = require('express');

const app = express();
const server = new ApolloServer({ 
  typeDefs, 
  resolvers,
  context: ({ req }) => ({
    db: new Database(),
    user: req.user
  })
});

await server.start();
server.applyMiddleware({ app });
```

---

## 📊 Сравнение с REST

### **REST подход:**

bash

# Multiple requests для получения связанных данных
GET /users/123
GET /users/123/posts
GET /users/123/friends
GET /posts/456/comments

### **GraphQL подход:**

graphql

# Один запрос
```qraphql
query {
  user(id: "123") {
    name
    posts {
      title
      comments {
        text
      }
    }
    friends {
      name
    }
  }
}
```



## 🍃 GraphQL на Spring стеке (полное руководство)

Spring Boot имеет отличную официальную поддержку GraphQL через **Spring GraphQL**.

### Шаг 1: Добавляем зависимости


```xml

<!-- pom.xml -->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-graphql</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
</dependencies>
```

### Шаг 2: Создаем GraphQL схему

```graphql

# src/main/resources/graphql/schema.graphqls
type Query {
    users: [User!]!
    user(id: ID!): User
    posts(userId: ID): [Post!]!
}

type Mutation {
    createUser(input: UserInput!): User!
    updateUser(id: ID!, input: UserInput!): User!
}

type User {
    id: ID!
    name: String!
    email: String!
    posts: [Post!]!
}

type Post {
    id: ID!
    title: String!
    content: String!
    author: User!
}

input UserInput {
    name: String!
    email: String!
}
```

### Шаг 3: Создаем сущности JPA

```java

@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private String email;
    
    @OneToMany(mappedBy = "author", cascade = CascadeType.ALL)
    private List<Post> posts = new ArrayList<>();
    
    // getters, setters, constructors
}

@Entity
@Table(name = "posts")
public class Post {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String title;
    private String content;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private User author;
    
    // getters, setters, constructors
}```

### Шаг 4: Создаем репозитории

```java

public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
}

public interface PostRepository extends JpaRepository<Post, Long> {
    List<Post> findByAuthorId(Long authorId);
    List<Post> findAllByOrderByCreatedAtDesc();
}

### Шаг 5: Создаем DataFetchers (Controller-ы)

java

@Component
public class UserController implements GraphQlController {

    private final UserRepository userRepository;
    private final PostRepository postRepository;

    public UserController(UserRepository userRepository, PostRepository postRepository) {
        this.userRepository = userRepository;
        this.postRepository = postRepository;
    }

    @QueryMapping
    public List<User> users() {
        return userRepository.findAll();
    }

    @QueryMapping
    public Optional<User> user(@Argument Long id) {
        return userRepository.findById(id);
    }

    @QueryMapping
    public List<Post> posts(@Argument Optional<Long> userId) {
        return userId
            .map(postRepository::findByAuthorId)
            .orElseGet(postRepository::findAll);
    }

    @MutationMapping
    public User createUser(@Argument UserInput input) {
        User user = new User();
        user.setName(input.getName());
        user.setEmail(input.getEmail());
        return userRepository.save(user);
    }

    @SchemaMapping
    public List<Post> posts(User user) {
        return postRepository.findByAuthorId(user.getId());
    }
}

// Input DTO
public record UserInput(String name, String email) {}

### Шаг 6: Настройка (опционально)

yaml

# application.yml
spring:
  graphql:
    graphiql:
      enabled: true  # Включаем GraphiQL UI
      path: /graphiql
    schema:
      printer:
        enabled: true
  datasource:
    url: jdbc:postgresql://localhost:5432/graphqldb
    username: postgres
    password: password

### Шаг 7: Запуск и тестирование

Запускаем приложение и открываем: [http://localhost:8080/graphiql](http://localhost:8080/graphiql)

**Пример запроса:**

graphql

query {
  users {
    id
    name
    email
    posts {
      title
      content
    }
  }
}

**Пример мутации:**

graphql

mutation {
  createUser(input: {
    name: "John Doe"
    email: "john@example.com"
  }) {
    id
    name
    email
  }
}
```
---

## 🚀 Расширенные возможности Spring GraphQL

### 1. **DataLoader для N+1 проблемы**

```java

@Configuration
public class DataLoaderConfig {

    @Bean
    public DataLoaderRegistry dataLoaderRegistry() {
        DataLoaderRegistry registry = new DataLoaderRegistry();
        registry.register("userLoader", 
            DataLoader.newDataLoader(this::loadUsers));
        return registry;
    }

    private CompletableFuture<List<User>> loadUsers(List<Long> ids) {
        return CompletableFuture.supplyAsync(() -> 
            userRepository.findAllById(ids));
    }
}```

### 2. **Валидация**

```java

@MutationMapping
public User createUser(@Argument @Valid UserInput input) {
    // ...
}

public record UserInput(
    @NotBlank String name,
    @Email String email
) {}

### 3. **Security**

java

@PreAuthorize("hasRole('ADMIN')")
@QueryMapping
public List<User> users() {
    return userRepository.findAll();
}

### 4. **Подписки (WebSocket)**

java

@SubscriptionMapping
public Flux<Post> recentPosts() {
    return postRepository.findRecentPostsStream();
}

### 5. **Кастомные scalar-типы**

java

@Configuration
public class GraphQLConfig {

    @Bean
    public RuntimeWiringConfigurer runtimeWiringConfigurer() {
        return wiringBuilder -> wiringBuilder
            .scalar(ExtendedScalars.Date)
            .scalar(ExtendedScalars.DateTime);
    }
}```

---

## 📊 Мониторинг и метрики

```yaml

management:
  endpoints:
    web:
      exposure:
        include: graphql, metrics
  graphql:
    enabled: true
```

Доступные метрики:
- `graphql.query` - время выполнения запросов
- `graphql.error` - ошибки
- `graphql.data.loader` - производительность DataLoader

# GraphQL vs Rest

GraphQL вместо работы с жестко определенными на сервере конечными точками (endpoints) вы можете с помощью одного запроса получить именно те данные, которые вам нужны. 

GraphQL гибок при внедрении в организации, он делает совместную работу команд frontend- и backend-разработки гладкой, как никогда раньше. Однако на практике обе эти технологии подразумевают отправку HTTP-запроса и получение какого-то результата, и внутри GraphQL встроено множество элементов из модели REST.

Key Differences Summarized:

| Feature             | REST                                   | GraphQL                                        |
| ------------------- | -------------------------------------- | ---------------------------------------------- |
| Architecture        | Resource-based, multiple endpoints     | Schema-driven, single endpoint                 |
| Data Fetching       | Fixed data from pre-defined endpoints  | Client-defined data with precise queries       |
| Over/Under-fetching | Common issues                          | Minimized                                      |
| Caching             | Leverages HTTP caching                 | Requires custom caching solutions              |
| Complexity          | Simpler for basic resource interaction | More complex to set up initially, but flexible |
| Use Cases           | Public APIs, simple web apps           | Mobile apps, complex data-intensive apps       |


