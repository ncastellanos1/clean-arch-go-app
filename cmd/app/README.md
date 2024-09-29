### **`cmd/app/` - Main Application Entry Point**

The **`/cmd/app/`** folder contains the **main entry point** of the Go application. This is where the server is initialized, routes and middlewares are set up, and external services like the database or cache are launched. The main file in this folder is `main.go`.

#### **Responsibilities of `main.go`:**
1. **Start the HTTP server**.
2. **Configure global middlewares** (such as authentication and logging).
3. **Register API routes** for different entities.
4. **Establish connections** to external services (databases, Redis, etc.).
5. **Handle initialization errors** and application startup.

### **Structure of `main.go`**

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

    // Register routes from different modules (users, roles, products)
    routes.RegisterUserRoutes(router, dbConn)
    routes.RegisterRoleRoutes(router, dbConn)
    routes.RegisterProductRoutes(router, dbConn)

    // Start the server
    log.Println("Server running on port 8080")
    log.Fatal(http.ListenAndServe(":8080", router))
}
```

### **Explanation of Each Section:**

1. **Imports**:
   - Required packages for handling routes (`mux`), database connection (`db`), and registering specific routes from the `routes` folder.

2. **Database Connection**:
   - The function `db.ConnectPostgres()` initializes the PostgreSQL database connection.

3. **Creating the Router**:
   - The **main router** is created using `mux.NewRouter()`. This is where all routes will be added.

4. **Registering Routes**:
   - Routes for users, roles, and products are registered by calling specific route handlers from the `/internal/api/routes/` folder.

5. **Starting the Server**:
   - The server starts on port `8080` using `http.ListenAndServe()` and the **router** is passed to it.

### **How to Implement**

The **`main.go`** file is used to start the application. This is where you define the global server behavior and how it responds to incoming requests.

### **Usage Example**

1. **Run the Project**:
   From the root directory of your project, run the following command to start the server:

   ```bash
   go run ./cmd/app/main.go
   ```

2. **Expected Output**:
   The server will initialize and you should see the following in the console:
   ```bash
   Server running on port 8080
   ```

3. **Accessing the API**:
   Once the server is running, you can make requests to the defined routes, for example:
   ```bash
   GET http://localhost:8080/users
   ```