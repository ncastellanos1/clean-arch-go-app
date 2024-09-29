### **`/internal/api/routes/` - API Route Definitions**

The **`/routes/`** folder contains the route definitions for the API. This is where different endpoints are mapped to their respective controllers and middlewares. Each entity (such as users, roles, or products) has its own set of routes that handle specific actions like creating, reading, updating, or deleting data.

### **Responsibilities of the Routes:**
1. **Map API Endpoints**: Define the URL paths (e.g., `/users`, `/products`) and the HTTP methods (e.g., `GET`, `POST`) for each resource.
2. **Connect Controllers**: Link the routes to the corresponding controller functions that will handle the incoming requests.
3. **Apply Middlewares**: Attach middlewares to routes, such as authentication or logging, to manage cross-cutting concerns.
4. **Organize API by Entity**: Routes are organized based on entities (e.g., users, roles, products) to maintain modularity and clarity.

### **Folder Structure:**
- **`user_routes.go`**: Defines all routes related to user operations (e.g., creating a user, getting user information).
- **`role_routes.go`**: Defines routes related to role management.
- **`product_routes.go`**: Defines routes related to product management.

Each route file handles the registration of routes for the specific entity and connects them to their corresponding controller.

### **Example - `user_routes.go`:**

```go
package routes

import (
    "your_project/internal/api/http"
    "github.com/gorilla/mux"
    "gorm.io/gorm"
)

func RegisterUserRoutes(router *mux.Router, db *gorm.DB) {
    // Initialize the UserController with necessary dependencies (e.g., use case, db)
    userController := http.NewUserController(db)

    // Define routes related to user management
    router.HandleFunc("/users", userController.CreateUserHandler).Methods("POST")
    router.HandleFunc("/users", userController.ListUsersHandler).Methods("GET")
    router.HandleFunc("/users/{id}", userController.GetUserHandler).Methods("GET")
    router.HandleFunc("/users/{id}", userController.UpdateUserHandler).Methods("PUT")
    router.HandleFunc("/users/{id}", userController.DeleteUserHandler).Methods("DELETE")
}
```

### **Explanation of the Example:**

1. **RegisterUserRoutes**: This function registers all the routes related to the **users** entity.
   
2. **Controller Initialization**: The `UserController` is initialized by passing any necessary dependencies (like a database connection or use cases).
   
3. **Route Definitions**: Each route is defined with a corresponding HTTP method and path. For example:
   - `POST /users` maps to the `CreateUserHandler` for creating a new user.
   - `GET /users` maps to the `ListUsersHandler` for retrieving all users.
   - `GET /users/{id}` maps to the `GetUserHandler` for fetching a user by ID.
   
4. **Middleware Integration**: Middlewares can be added to routes where necessary, such as adding an authentication middleware for protected routes.

### **How to Implement Routes:**

1. **Define Route Functions**:
   - For each entity (users, roles, products), create a function to register the routes related to that entity. This function takes in a router and any necessary dependencies, like the database connection or the controller.

2. **Map HTTP Methods to Handlers**:
   - Use the router to map HTTP methods (GET, POST, PUT, DELETE) to the appropriate handler functions in the controllers.

3. **Use Placeholders for Dynamic Data**:
   - Use `{id}` or other placeholders in the route paths to accept dynamic data like user IDs or product IDs.

4. **Apply Middlewares**:
   - Attach middlewares to routes or groups of routes as needed. For example, authentication middleware can be applied to routes that require authorization.

### **How Routes are Used in the Application:**

In the **`main.go`** file, routes are registered by calling the route functions for each entity. For example:

```go
package main

import (
    "log"
    "net/http"
    "your_project/internal/api/routes"
    "your_project/internal/infrastructure/db"
    "github.com/gorilla/mux"
)

func main() {
    // Initialize the database connection
    dbConn := db.ConnectPostgres()
    defer dbConn.Close()

    // Create the main router
    router := mux.NewRouter()

    // Register routes for users, roles, and products
    routes.RegisterUserRoutes(router, dbConn)
    routes.RegisterRoleRoutes(router, dbConn)
    routes.RegisterProductRoutes(router, dbConn)

    // Start the server
    log.Println("Server running on port 8080")
    log.Fatal(http.ListenAndServe(":8080", router))
}
```

### **Example of Adding a Route with Middleware:**

You can add middleware to specific routes when registering them:

```go
package routes

import (
    "your_project/internal/api/http"
    "your_project/internal/api/http/middlewares"
    "github.com/gorilla/mux"
    "gorm.io/gorm"
)

func RegisterUserRoutes(router *mux.Router, db *gorm.DB, authMiddleware *middlewares.AuthMiddleware) {
    userController := http.NewUserController(db)

    // Define routes with authentication middleware
    router.Handle("/users", authMiddleware.RequireAuth(http.HandlerFunc(userController.CreateUserHandler))).Methods("POST")
    router.Handle("/users", authMiddleware.RequireAuth(http.HandlerFunc(userController.ListUsersHandler))).Methods("GET")
}
```

### **Summary:**

The **`/routes/`** folder is responsible for defining and organizing the API routes for the application. Each file within this folder defines the routes for a specific entity (users, roles, products), connects them to the respective controllers, and applies middlewares where needed. The routes are registered in the `main.go` file to make them available when the application starts.