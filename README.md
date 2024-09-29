# Project: my-clean-arch-project

This project implements a Golang application following the principles of Clean Architecture. The architecture is designed to provide a clear separation of responsibilities, enhance maintainability, and facilitate scalable development. The project includes HTTP controllers, middlewares, use cases, repositories, and services, organized into well-defined layers.

## Project Structure

```
/cmd/app/                # Main entry point of the application
/internal/api/           # Contains logic related to HTTP and routes
  ├── http/              # HTTP controllers and middlewares
  ├── routes/            # Route definitions for each entity
/internal/domain/        # Defines domain models, DTOs, and presenters
  ├── models/            # Domain models representing business entities
  ├── dto/               # Data Transfer Objects (DTOs) defining data sent and received through the API
  ├── presenters/        # Presenters to transform entities into DTOs
/internal/usecases/      # Use cases to handle business logic
  ├── user/              # Use cases related to users
  ├── role/              # Use cases related to roles
  ├── product/           # Use cases related to products
/internal/services/      # Services that coordinate logic between use cases
/internal/repositories/  # Repositories responsible for interacting with the database
  ├── user/              # Repositories related to users
  ├── role/              # Repositories related to roles
  ├── product/           # Repositories related to products
/internal/infrastructure/# Infrastructure configuration (database, cache, etc.)
  ├── db/                # Database configuration and connection
  ├── cache/             # Redis configuration and connection (if caching is used)
/configs/                # Configuration files such as credentials and environment variables
/pkg/                    # Utility packages such as logging
/scripts/                # Scripts for migrations, deployment, and administrative tasks
/docs/                   # Project documentation and API specifications
```

## Project Configuration

- **Database**: Configuration to connect to PostgreSQL defined in `/internal/infrastructure/db/`.
- **Cache**: Configuration to connect to Redis defined in `/internal/infrastructure/cache/`.
- **Routes**: Routes and middlewares are defined in the `/internal/api/routes/` folder.

### Useful Commands

- **Database Migration**:
  ```
  ./scripts/migrate_db.sh
  ```

- **Run the Application**:
  ```
  go run ./cmd/app/main.go
  ```