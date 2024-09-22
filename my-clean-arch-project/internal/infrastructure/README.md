### **`/internal/infrastructure/` - Infrastructure Layer**

The **`/infrastructure/`** folder contains the code responsible for managing the infrastructure dependencies of the application, such as **database connections**, **cache systems**, and other third-party services. The infrastructure layer provides the concrete implementations that the application relies on for external interactions and resources, allowing the core business logic (domain and use cases) to remain independent of specific technologies.

### **Responsibilities of the Infrastructure Layer:**
1. **Manage External Services**: Handle the setup and connection to external services like databases, caches (e.g., Redis), or message brokers.
2. **Decouple Business Logic from Infrastructure**: Provide interfaces and abstractions so that business logic (use cases and domain) does not depend on specific implementations or services.
3. **Handle Configuration**: Define configuration and setup procedures for the infrastructure, such as database connection strings or API credentials for third-party services.

### **Folder Structure:**
- **`db/`**: Contains all the code related to database management, including setting up connections and migrations.
- **`cache/`**: Handles the configuration and management of caching services like Redis.
  
Each subfolder is dedicated to a specific type of infrastructure component and provides the necessary setup and utilities for that service.

### **Example - `db/postgres.go`:**

```go
package db

import (
    "log"
    "gorm.io/driver/postgres"
    "gorm.io/gorm"
)

// ConnectPostgres initializes the PostgreSQL database connection.
func ConnectPostgres() *gorm.DB {
    dsn := "host=localhost user=postgres password=secret dbname=mydb port=5432 sslmode=disable"
    db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatalf("Failed to connect to database: %v", err)
    }

    log.Println("Connected to PostgreSQL database successfully.")
    return db
}
```

### **Explanation of the Example:**

1. **Database Connection**:
   - The `ConnectPostgres` function sets up a connection to a PostgreSQL database using **GORM**. It takes in a **DSN** (Data Source Name) that includes the host, username, password, and other connection details.
   
2. **Error Handling**:
   - If the connection fails, the function logs an error and stops the application. If successful, it logs a message and returns the database connection.

3. **Separation from Business Logic**:
   - This database connection setup is isolated in the infrastructure layer, ensuring that the rest of the application (e.g., use cases) doesn’t need to be concerned with database connection details.

### **How Infrastructure Components Work in the Application:**

1. **Provide Interfaces**:
   - The infrastructure layer provides concrete implementations for interfaces that the application core depends on (e.g., repositories for database access, cache for temporary storage).

2. **Handle External Dependencies**:
   - Infrastructure handles connections to external services (like databases and caches), ensures that configurations are loaded properly, and manages errors that may occur during these interactions.

3. **Setup in the `main.go`**:
   - In the main entry point of the application (`main.go`), the infrastructure components are initialized, and their connections are passed to the use cases or repositories as needed.

### **Example - `cache/redis.go`:**

```go
package cache

import (
    "github.com/go-redis/redis/v8"
    "context"
    "log"
)

// ConnectRedis initializes the Redis connection.
func ConnectRedis() *redis.Client {
    rdb := redis.NewClient(&redis.Options{
        Addr:     "localhost:6379",
        Password: "", // No password set
        DB:       0,  // Use default DB
    })

    // Ping the Redis server to check the connection
    _, err := rdb.Ping(context.Background()).Result()
    if err != nil {
        log.Fatalf("Failed to connect to Redis: %v", err)
    }

    log.Println("Connected to Redis successfully.")
    return rdb
}
```

In this example:
- The `ConnectRedis` function connects to a Redis instance, logs the status of the connection, and returns a Redis client to be used by the application.

### **How to Implement Infrastructure Components:**

1. **Define Connection Functions**:
   - Create functions that handle the connection to external services like databases or caches. Each function should be responsible for initializing the service, handling errors, and returning a client or connection object.

2. **Use Configuration**:
   - Use configuration files (e.g., `config.yaml`) to store environment-specific settings such as database credentials or Redis addresses.

3. **Pass Connections to Repositories**:
   - The database or cache connections initialized in the infrastructure layer should be passed to repositories or services that depend on them.

### **Usage Example in `main.go`:**

```go
package main

import (
    "log"
    "net/http"
    "your_project/internal/infrastructure/db"
    "your_project/internal/infrastructure/cache"
    "your_project/internal/api/routes"
    "github.com/gorilla/mux"
)

func main() {
    // Initialize PostgreSQL connection
    dbConn := db.ConnectPostgres()
    defer dbConn.Close()

    // Initialize Redis connection
    redisClient := cache.ConnectRedis()
    defer redisClient.Close()

    // Create the main router
    router := mux.NewRouter()

    // Register application routes
    routes.RegisterUserRoutes(router, dbConn)
    routes.RegisterProductRoutes(router, dbConn, redisClient)

    // Start the server
    log.Println("Server running on port 8080")
    log.Fatal(http.ListenAndServe(":8080", router))
}
```

In this example:
- The `main.go` initializes the PostgreSQL and Redis connections using functions defined in the infrastructure layer. These connections are then passed to the repositories and services that need them.

### **Summary:**

The **`/infrastructure/`** folder is responsible for managing external service dependencies such as databases, cache systems, and any third-party integrations. It handles the setup and configuration of these services, ensuring that the business logic remains independent of specific technologies. This separation allows the application to easily swap or modify infrastructure components without affecting the core logic.