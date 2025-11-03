# ![RealWorld Example App using Kotlin and Spring](example-logo.png)

> ### Spring boot + MyBatis codebase containing real world examples (CRUD, auth, advanced patterns, etc) that adheres to the [RealWorld](https://github.com/gothinkster/realworld-example-apps) spec and API.

This codebase was created to demonstrate a fully fledged fullstack application built with Spring boot + Mybatis including CRUD operations, authentication, routing, pagination, and more.

For more information on how to this works with other frontends/backends, head over to the [RealWorld](https://github.com/gothinkster/realworld) repo.

# How it works

The application uses Spring boot (Web, Mybatis).

* Use the idea of Domain Driven Design to separate the business term and infrastruture term.
* Use MyBatis to implement the [Data Mapper](https://martinfowler.com/eaaCatalog/dataMapper.html) pattern for persistence.
* Use [CQRS](https://martinfowler.com/bliki/CQRS.html) pattern to separate the read model and write model.

And the code organize as this:

1. `api` is the web layer to implement by Spring MVC
2. `core` is the business model including entities and services
3. `application` is the high level services for query with the data transfer objects
4. `infrastructure`  contains all the implementation classes as the technique details

# Architecture Overview

This application follows a **layered architecture** implementing **Domain-Driven Design (DDD)** principles, **CQRS (Command Query Responsibility Segregation)**, and the **Data Mapper** pattern to achieve clean separation of concerns, maintainability, and scalability.

## Architectural Patterns

The application follows several architectural patterns to ensure clean separation of concerns and maintainability:

### Domain-Driven Design (DDD)
The codebase is organized following DDD principles, separating business logic from infrastructure concerns. Domain entities (`core` layer) contain business rules and are independent of frameworks and persistence details.

**Key DDD Concepts Applied**:
- **Domain Entities**: Rich domain models with business logic (Article, User, Comment)
- **Repository Pattern**: Abstractions for data access (ArticleRepository, UserRepository)
- **Domain Services**: Business logic that doesn't belong to a single entity (AuthorizationService)
- **Bounded Context**: Clear boundaries between layers
- **Ubiquitous Language**: Domain terms used consistently throughout codebase

### CQRS (Command Query Responsibility Segregation)
The application separates read and write operations:
- **Commands (Write)**: Handled through domain repositories in the `core` layer
- **Queries (Read)**: Handled through query services in the `application` layer using optimized read models

This separation allows for:
- Optimized read queries without affecting write operations
- Different data models for reads and writes
- Better scalability and performance
- Independent optimization of read and write paths

**CQRS Implementation**:
```
Write Path (Command):
API → Domain Entity → Repository Interface → MyBatis Repository → Database

Read Path (Query):
API → Query Service → Read Service → MyBatis Mapper → Database → DTO
```

### Data Mapper Pattern
MyBatis implements the Data Mapper pattern, which provides a clean separation between domain objects and database tables. The mapper handles the mapping between objects and database rows.

**Benefits**:
- Domain objects don't know about database structure
- Database schema changes don't affect domain logic
- Mapping logic centralized in MyBatis XML mappers
- Clean separation between persistence and domain layers

## Layer Architecture

The application follows a layered architecture with four main layers:

```
┌─────────────────────────────────────────┐
│           API Layer                     │
│  (REST Controllers, Request/Response)   │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│        Application Layer                 │
│  (Query Services, DTOs, Use Cases)       │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│           Core Layer                    │
│  (Domain Entities, Repositories,        │
│   Business Services)                    │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│      Infrastructure Layer               │
│  (MyBatis Mappers, Repository           │
│   Implementations, External Services)   │
└─────────────────────────────────────────┘
```

### 1. API Layer (`io.spring.api`)
**Purpose**: Entry point for HTTP requests, handles request/response formatting

**Responsibilities**:
- REST endpoints (Controllers)
- Request validation
- Response formatting
- Exception handling
- Security (JWT authentication)

**Key Components**:
- `ArticlesApi`, `CommentsApi`, `UsersApi`, etc. - REST controllers
- `security/` - Security configuration and JWT filter
- `exception/` - Custom exception handlers

**Example Flow**:
```java
@RestController
public class ArticlesApi {
    // Receives HTTP request → Validates → Delegates to services → Returns response
}
```

### 2. Application Layer (`io.spring.application`)
**Purpose**: High-level orchestration and query services

**Responsibilities**:
- Query services for read operations
- Data Transfer Objects (DTOs)
- Business use case orchestration
- Pagination and filtering logic

**Key Components**:
- `ArticleQueryService`, `CommentQueryService`, `UserQueryService` - Query services
- `data/` - DTOs (ArticleData, UserData, CommentData, etc.)
- `Page` - Pagination utilities

**Characteristics**:
- Uses read services from infrastructure layer
- Returns DTOs optimized for API responses
- Handles complex queries and aggregations

### 3. Core Layer (`io.spring.core`)
**Purpose**: Domain model and business logic

**Responsibilities**:
- Domain entities (Article, User, Comment, etc.)
- Repository interfaces (abstractions)
- Domain services (AuthorizationService, JwtService)
- Business rules and validations

**Key Components**:
- `article/`, `user/`, `comment/`, `favorite/` - Domain entities
- `*Repository` interfaces - Repository contracts
- `service/` - Domain services

**Characteristics**:
- Framework-independent
- Contains business logic
- No direct database dependencies
- Pure Java objects

### 4. Infrastructure Layer (`io.spring.infrastructure`)
**Purpose**: Technical implementations and external integrations

**Responsibilities**:
- MyBatis mappers and repository implementations
- Database access logic
- External service implementations
- Technical utilities

**Key Components**:
- `repository/` - MyBatis implementations of core repositories
- `mybatis/mapper/` - MyBatis mapper interfaces
- `mybatis/readservice/` - Optimized read services
- `service/` - Technical service implementations (JWT, encryption)

## Data Flow

### Complete Request Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    Client Request                           │
│              HTTP POST /articles                            │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│         API Layer (io.spring.api)                           │
│  - ArticlesApi.createArticle()                               │
│  - Request validation (@Valid)                              │
│  - Security check (@AuthenticationPrincipal)                 │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│      Core Layer (io.spring.core)                            │
│  - Article entity created with business logic                │
│  - Slug generation                                           │
│  - Tag processing                                            │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│   Infrastructure Layer (io.spring.infrastructure)           │
│  - MyBatisArticleRepository.save()                           │
│  - ArticleMapper.insert()                                    │
│  - TagMapper operations                                      │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│                    Database (MySQL)                         │
│  - INSERT into articles table                               │
│  - INSERT into article_tags table                           │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│   Application Layer (io.spring.application)                │
│  - ArticleQueryService.findById()                          │
│  - ArticleReadService.findById()                            │
│  - DTO creation (ArticleData)                               │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│              API Response (JSON)                             │
│         { "article": { ... } }                              │
└─────────────────────────────────────────────────────────────┘
```

### Write Operation Flow (Command)

**Example: Creating an Article**

```
1. HTTP POST /articles
   Headers: Authorization: Token <jwt>
   Body: { "article": { "title": "...", "body": "..." } }
   ↓
2. ArticlesApi.createArticle()
   - Validates request body (@Valid)
   - Checks authentication (@AuthenticationPrincipal)
   - Extracts user from security context
   ↓
3. Creates Article domain entity
   - Article constructor generates slug from title
   - Processes tagList array
   - Sets userId, timestamps
   ↓
4. ArticleRepository.save(article)
   - Interface call (core layer)
   ↓
5. MyBatisArticleRepository.save()
   - Checks if article exists (update vs insert)
   - For new articles:
     - Inserts tags if not exist
     - Creates article-tag relationships
     - Inserts article record
   ↓
6. MyBatis Mapper (ArticleMapper)
   - Executes SQL via MyBatis XML mapper
   - Maps domain object to database rows
   ↓
7. Database Transaction
   - All operations in single transaction
   - Commit on success, rollback on error
   ↓
8. Return response
   - Query service fetches created article
   - Converts to DTO (ArticleData)
   - Returns JSON response
```

### Read Operation Flow (Query)

**Example: Fetching Article List**

```
1. HTTP GET /articles?tag=react&offset=0&limit=20
   ↓
2. ArticlesApi.getArticles()
   - Extracts query parameters
   - Optional authentication check
   ↓
3. ArticleQueryService.findRecentArticles()
   - Builds query parameters
   - Handles pagination (Page object)
   - Passes user for personalization
   ↓
4. ArticleReadService (infrastructure)
   - Optimized read service
   - Executes complex SQL joins
   - Fetches articles with author info
   - Includes favorite counts
   ↓
5. MyBatis Mapper (ArticleMapper)
   - Executes SELECT query
   - Joins multiple tables:
     - articles → users (author)
     - articles → article_favorites (count)
     - articles → article_tags → tags
   ↓
6. Database Query
   - Returns result set
   ↓
7. MyBatis Result Mapping
   - Maps rows to DTOs (ArticleData)
   - Handles nested objects (author, tags)
   ↓
8. ArticleQueryService.enrichData()
   - Adds user-specific data:
     - Is favorited by current user?
     - Is following author?
   ↓
9. Returns ArticleDataList
   - Contains articles array
   - Contains articlesCount for pagination
   ↓
10. JSON Response
    { "articles": [...], "articlesCount": 100 }
```

### Authentication Flow

```
1. HTTP POST /users/login
   Body: { "user": { "email": "...", "password": "..." } }
   ↓
2. UsersApi.userLogin()
   - Validates request
   ↓
3. UserRepository.findByEmail()
   - Finds user by email
   ↓
4. EncryptService.check()
   - Verifies password hash
   ↓
5. JwtService.toToken()
   - Generates JWT token
   - Includes user ID as subject
   ↓
6. UserQueryService.findById()
   - Fetches user data
   - Converts to DTO (UserData)
   ↓
7. Returns UserWithToken
   - Includes user data + token
   ↓
8. Subsequent Requests
   - Client includes: Authorization: Token <jwt>
   ↓
9. JwtTokenFilter (intercepts all requests)
   - Extracts token from header
   - Validates token via JwtService
   - Loads user from UserRepository
   - Sets authentication in SecurityContext
   ↓
10. Controller receives @AuthenticationPrincipal User
    - User object available in all controllers
```

### Transaction Management

**Write Operations**:
- Repository methods annotated with `@Transactional`
- Ensures atomicity for multi-step operations
- Example: Creating article with tags requires multiple inserts
- Rollback on any failure

**Read Operations**:
- No transactions required (read-only)
- Optimized for performance
- Can use read replicas in production

## Technology Stack

### Core Framework
- **Spring Boot 2.7.18** - Application framework
- **Spring MVC** - Web layer
- **Spring Security** - Security and authentication
- **Spring Validation** - Request validation

### Persistence
- **MyBatis 2.3.1** - SQL mapping framework
- **MySQL 8.0.33** - Production database
- **H2 Database** - In-memory database for testing
- **Flyway** - Database migration tool

### Security
- **JWT (JJWT 0.11.2)** - Token-based authentication
- **Spring Security** - Authentication and authorization
- **Custom JWT Filter** - Token validation middleware

### Utilities
- **Lombok** - Reduce boilerplate code
- **Joda Time** - Date/time handling
- **RestAssured** - API testing (test scope)

## Security Architecture

### Authentication Flow
1. User registers/logs in via `/users` or `/users/login`
2. Server generates JWT token and returns it
3. Client includes token in `Authorization: Token <jwt>` header
4. `JwtTokenFilter` intercepts requests and validates token
5. Spring Security sets authenticated user in security context
6. Controllers access user via `@AuthenticationPrincipal`

### Authorization
- **AuthorizationService** - Checks if user can perform operations
- Article/Comment operations require ownership verification
- Profile follow/unfollow requires authentication

## Database Schema

The application uses MyBatis with SQL mapping files:
- Tables: `users`, `articles`, `comments`, `tags`, `article_tags`, `article_favorites`, `follows`
- Relationships maintained through join tables
- Flyway manages schema migrations

## Dependency Direction

```
API → Application → Core ← Infrastructure
```

- **API** depends on **Application** and **Core**
- **Application** depends on **Core** and **Infrastructure**
- **Core** has no dependencies (pure domain)
- **Infrastructure** implements **Core** interfaces

This ensures that:
- Business logic remains independent
- Infrastructure can be swapped without changing core
- Testing is easier with clear boundaries

# Security

Integration with Spring Security and add other filter for jwt token process.

The secret key is stored in `application.properties`.

# Database

It uses a H2 in memory database (for now), can be changed easily in the `application.properties` for any other database.

# Getting started

## Prerequisites

Before you begin, ensure you have the following installed:

- **Java 17** (JDK 17 or higher)
- **Gradle** (or use the Gradle wrapper included in the project)
- **MySQL** (for production/local development, or H2 for testing)

## Environment Configuration

To run the application locally, you need to set the following environment variables:

```bash
DB_USERNAME - username which will be used to connect to MySQL DB instance
DB_PASSWORD - password which will be used to connect to MySQL DB instance
DB_URL      - url by which application can access MySQL (e.g., localhost)
DB_PORT     - port by which application can access MySQL (e.g., 3306)
DB_NAME     - database name which will be used to store information
```

### Setting Environment Variables

**Windows (PowerShell):**
```powershell
$env:DB_USERNAME="your_username"
$env:DB_PASSWORD="your_password"
$env:DB_URL="localhost"
$env:DB_PORT="3306"
$env:DB_NAME="your_database"
```

**Windows (Command Prompt):**
```cmd
set DB_USERNAME=your_username
set DB_PASSWORD=your_password
set DB_URL=localhost
set DB_PORT=3306
set DB_NAME=your_database
```

**Linux/Mac:**
```bash
export DB_USERNAME=your_username
export DB_PASSWORD=your_password
export DB_URL=localhost
export DB_PORT=3306
export DB_NAME=your_database
```

## Building the Application

To build the application, run:

```bash
./gradlew build
```

This will compile the code, run tests, and create a JAR file in `build/libs/`.

To build without running tests:

```bash
./gradlew build -x test
```

To clean previous build artifacts:

```bash
./gradlew clean
```

## Running the Application Locally

### Option 1: Using Gradle BootRun (Recommended for Development)

Run the application directly using Gradle:

```bash
./gradlew bootRun
```

Make sure you have set all required environment variables before running this command.

### Option 2: Using the JAR File

After building the application, you can run it using the generated JAR:

```bash
./gradlew build
java -jar ./build/libs/*.jar
```

Replace `*` with the actual JAR filename if needed.

### Verifying the Application

Once the application starts, you can verify it's running by:

1. Opening a browser and navigating to: http://localhost:8080/tags

2. Or using curl:
   ```bash
   curl http://localhost:8080/tags
   ```

The application will be available at http://localhost:8080

## Testing the Application

The repository contains comprehensive test cases covering both API tests and repository tests.

### Running All Tests

To run all tests:

```bash
./gradlew test
```

### Running Tests with Verbose Output

For more detailed test output:

```bash
./gradlew test --info
```

### Running Specific Test Classes

To run a specific test class:

```bash
./gradlew test --tests "com.example.YourTestClass"
```

### Test Coverage

The tests include:
- **API Tests**: Testing REST endpoints using RestAssured and Spring MockMvc
- **Repository Tests**: Testing database operations using MyBatis test utilities
- **Security Tests**: Testing JWT authentication and authorization

**Note**: Tests use H2 in-memory database by default, so no MySQL setup is required for testing.

# API Endpoints

The API follows the [RealWorld API specification](https://github.com/gothinkster/realworld). All endpoints are available at `http://localhost:8080`.

**Note**: Endpoints marked with 🔒 require authentication via JWT token in the `Authorization` header: `Authorization: Token <your-token>`

## Authentication

### Register User
- **POST** `/users`
- **Description**: Register a new user account
- **Authentication**: Not required
- **Request Body**:
  ```json
  {
    "user": {
      "email": "user@example.com",
      "username": "johndoe",
      "password": "password123"
    }
  }
  ```
- **Response**: Returns user object with JWT token

### Login User
- **POST** `/users/login`
- **Description**: Authenticate a user and receive a JWT token
- **Authentication**: Not required
- **Request Body**:
  ```json
  {
    "user": {
      "email": "user@example.com",
      "password": "password123"
    }
  }
  ```
- **Response**: Returns user object with JWT token

## Current User

### Get Current User 🔒
- **GET** `/user`
- **Description**: Get the currently authenticated user's information
- **Authentication**: Required

### Update Current User 🔒
- **PUT** `/user`
- **Description**: Update the currently authenticated user's information
- **Authentication**: Required
- **Request Body**:
  ```json
  {
    "user": {
      "email": "newemail@example.com",
      "username": "newusername",
      "password": "newpassword",
      "bio": "User bio",
      "image": "https://example.com/image.jpg"
    }
  }
  ```
- **Note**: All fields are optional; only include fields you want to update

## Tags

### Get All Tags
- **GET** `/tags`
- **Description**: Get a list of all tags used in articles
- **Authentication**: Not required
- **Response**: Returns array of tag strings

## Articles

### List Articles
- **GET** `/articles`
- **Description**: Get a paginated list of articles
- **Authentication**: Optional (returns additional data if authenticated)
- **Query Parameters**:
  - `offset` (default: 0) - Number of articles to skip
  - `limit` (default: 20) - Number of articles to return
  - `tag` (optional) - Filter by tag
  - `author` (optional) - Filter by author username
  - `favorited` (optional) - Filter by username who favorited
- **Example**: `/articles?tag=react&offset=0&limit=10`

### Get Article Feed 🔒
- **GET** `/articles/feed`
- **Description**: Get articles from users you follow
- **Authentication**: Required
- **Query Parameters**:
  - `offset` (default: 0) - Number of articles to skip
  - `limit` (default: 20) - Number of articles to return

### Create Article 🔒
- **POST** `/articles`
- **Description**: Create a new article
- **Authentication**: Required
- **Request Body**:
  ```json
  {
    "article": {
      "title": "How to build webapps that scale",
      "description": "Web development techniques",
      "body": "This is the body of the article",
      "tagList": ["react", "webdev", "tutorial"]
    }
  }
  ```

### Get Single Article
- **GET** `/articles/{slug}`
- **Description**: Get a single article by slug
- **Authentication**: Optional (returns additional data if authenticated)
- **Path Parameters**:
  - `slug` - Article slug (e.g., "how-to-build-webapps-that-scale")

### Update Article 🔒
- **PUT** `/articles/{slug}`
- **Description**: Update an article (must be the author)
- **Authentication**: Required
- **Request Body**:
  ```json
  {
    "article": {
      "title": "Updated title",
      "description": "Updated description",
      "body": "Updated body"
    }
  }
  ```
- **Note**: All fields are optional; only include fields you want to update

### Delete Article 🔒
- **DELETE** `/articles/{slug}`
- **Description**: Delete an article (must be the author)
- **Authentication**: Required

## Article Favorites

### Favorite Article 🔒
- **POST** `/articles/{slug}/favorite`
- **Description**: Mark an article as favorited
- **Authentication**: Required

### Unfavorite Article 🔒
- **DELETE** `/articles/{slug}/favorite`
- **Description**: Remove an article from favorites
- **Authentication**: Required

## Comments

### Get Comments for Article
- **GET** `/articles/{slug}/comments`
- **Description**: Get all comments for an article
- **Authentication**: Optional (returns additional data if authenticated)

### Add Comment to Article 🔒
- **POST** `/articles/{slug}/comments`
- **Description**: Create a new comment on an article
- **Authentication**: Required
- **Request Body**:
  ```json
  {
    "comment": {
      "body": "Great article!"
    }
  }
  ```

### Delete Comment 🔒
- **DELETE** `/articles/{slug}/comments/{id}`
- **Description**: Delete a comment (must be the comment author or article author)
- **Authentication**: Required
- **Path Parameters**:
  - `slug` - Article slug
  - `id` - Comment ID

## Profiles

### Get Profile
- **GET** `/profiles/{username}`
- **Description**: Get a user's profile information
- **Authentication**: Optional (returns additional data if authenticated)
- **Path Parameters**:
  - `username` - Username of the profile to retrieve

### Follow User 🔒
- **POST** `/profiles/{username}/follow`
- **Description**: Follow a user
- **Authentication**: Required

### Unfollow User 🔒
- **DELETE** `/profiles/{username}/follow`
- **Description**: Unfollow a user
- **Authentication**: Required

# Try it out with a RealWorld frontend

The entry point address of the backend API is at http://localhost:8080, **not** http://localhost:8080/api as some of the frontend documentation suggests.
