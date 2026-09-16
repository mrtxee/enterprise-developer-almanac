---
aliases:
  - Apollo Federation
  - Apollo Server
  - DataLoader
  - GraphQL
  - GraphQL caching
  - GraphQL Subscription
  - Hasura GraphQL Engine
  - Persisted queries
  - Query language
  - Resolvers
  - REST
  - REST API
  - Schema First
  - Spring GraphQL
  - WebSocket
  - Вебсокеты
  - Кэширование GraphQL
  - Постоянные запросы
  - Резолверы
  - Язык запросов
---

## GraphQL

**Суть**

GraphQL — это язык запросов для API и runtime для их выполнения. Он применяется, когда REST API становится недостаточно гибким или эффективным.

**Подписки (GraphQL Subscription)**

Есть возможность получать уведомления об изменениях по инициативе сервера при помощи GraphQL API посредством «мутаций». Самая популярная реализация этой спецификации использует «под капотом» WebSockets.

**Кэширование GraphQL**

- Кэширование на уровне полей
- Кэширование по хешу запроса
- Использование постоянных запросов (Persisted Queries)
- Кэширование на клиенте

**Реализации**

- Hasura GraphQL Engine
- Apollo Federation

### Когда применять GraphQL

**Сложные данные с множеством связей**

Один запрос вместо нескольких REST-вызовов:

```graphql
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

**Мобильные приложения с ограниченным трафиком**

Только нужные поля вместо полных объектов:

```graphql
query MobileUserProfile {
  user(id: "123") {
    name
    avatar(size: SMALL)
    onlineStatus
  }
}
```

**Агрегация данных из нескольких источников**

Данные из разных [[microservice|микросервисов]] в одном запросе:

```graphql
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

**Часто меняющиеся требования клиентов**

Клиенты сами определяют, какие данные им нужны:

```graphql
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

**Real-time обновления**

Подписка на изменения:

```graphql
subscription {
  postLikes(postId: "456") {
    likeCount
    recentLikers {
      name
    }
  }
}
```

### Как применять GraphQL

**Схема и типы (Schema First)**

Определение схемы:

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

**Резолверы (Resolvers)**

Реализация резолверов:

```javascript
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

**Интеграция с существующей инфраструктурой**

Apollo Server с Express:

```javascript
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

### GraphQL vs REST

GraphQL вместо работы с жёстко определёнными на сервере конечными точками (endpoints) позволяет одним запросом получить именно те данные, которые нужны. GraphQL гибок при внедрении в организации и делает совместную работу команд frontend- и backend-разработки гладкой. На практике обе технологии подразумевают отправку HTTP-запроса и получение какого-то результата, и внутри GraphQL встроено множество элементов из модели REST.

**REST подход**

Множество запросов для получения связанных данных:

```http
GET /users/123
GET /users/123/posts
GET /users/123/friends
GET /posts/456/comments
```

**GraphQL подход**

Один запрос:

```graphql
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

**Ключевые различия**

| Feature | REST | GraphQL |
| ------------------- | -------------------------------------- | ---------------------------------------------- |
| Architecture | Resource-based, multiple endpoints | Schema-driven, single endpoint |
| Data Fetching | Fixed data from pre-defined endpoints | Client-defined data with precise queries |
| Over/Under-fetching | Common issues | Minimized |
| Caching | Leverages HTTP caching | Requires custom caching solutions |
| Complexity | Simpler for basic resource interaction | More complex to set up initially, but flexible |
| Use Cases | Public APIs, simple web apps | Mobile apps, complex data-intensive apps |

### Spring GraphQL

[[spring|Spring Boot]] имеет официальную поддержку GraphQL через Spring GraphQL.

**Добавление зависимостей**

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

**Создание GraphQL-схемы**

Файл `src/main/resources/graphql/schema.graphqls`:

```graphql
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

**Создание сущностей JPA**

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
}
```

**Создание репозиториев**

```java
public interface UserRepository extends JpaRepository<User, Long> {
  Optional<User> findByEmail(String email);
}

public interface PostRepository extends JpaRepository<Post, Long> {
  List<Post> findByAuthorId(Long authorId);
  List<Post> findAllByOrderByCreatedAtDesc();
}
```

**Создание DataFetchers (контроллеров)**

```java
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

public record UserInput(String name, String email) {}
```

**Настройка (опционально)**

```yaml
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
```

**Запуск и тестирование**

Интерфейс GraphiQL доступен по адресу `/graphiql` (по умолчанию `localhost:8080`).

**Пример запроса**

```graphql
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
```

**Пример мутации**

```graphql
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

### Расширенные возможности Spring GraphQL

**DataLoader для N+1 проблемы**

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
}
```

**Валидация**

```java
@MutationMapping
public User createUser(@Argument @Valid UserInput input) {
  // ...
}

public record UserInput(
  @NotBlank String name,
  @Email String email
) {}
```

**Security**

```java
@PreAuthorize("hasRole('ADMIN')")
@QueryMapping
public List<User> users() {
  return userRepository.findAll();
}
```

**Подписки (WebSocket)**

```java
@SubscriptionMapping
public Flux<Post> recentPosts() {
  return postRepository.findRecentPostsStream();
}
```

**Кастомные scalar-типы**

```java
@Configuration
public class GraphQLConfig {

  @Bean
  public RuntimeWiringConfigurer runtimeWiringConfigurer() {
    return wiringBuilder -> wiringBuilder
      .scalar(ExtendedScalars.Date)
      .scalar(ExtendedScalars.DateTime);
  }
}
```

### Мониторинг и метрики

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

- `graphql.query` — время выполнения запросов
- `graphql.error` — ошибки
- `graphql.data.loader` — производительность DataLoader
