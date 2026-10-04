# KISS VMP Backend Architecture

## Technology Stack

- Java 25
- Spring Boot 4.1.1
- Maven
- Spring MVC
- Spring Data JPA
- Hibernate
- PostgreSQL
- Supabase
- Spring Security
- JWT
- Validation
- Lombok

## Request Flow

React Frontend
        ↓
REST API
        ↓
Controller
        ↓
Service
        ↓
Repository
        ↓
JPA / Hibernate
        ↓
PostgreSQL / Supabase

## Backend Layers

### Controller
Handles HTTP requests and responses.

### Service
Contains business logic and application rules.

### Repository
Provides database access through Spring Data JPA.

### Entity
Represents persistent database objects/tables.

### DTO
Defines the data exchanged through the API.

### Security
Handles authentication, JWT processing and authorization.

### Exception
Provides centralized error handling.

### Config
Contains application configuration.

### Seeder
Creates development/mock data.

## Planned Package Structure

com.kiss.vmp

├── controller
├── service
├── repository
├── entity
├── dto
├── security
├── exception
├── config
└── seeder