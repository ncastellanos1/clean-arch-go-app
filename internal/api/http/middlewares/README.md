### **`/internal/api/http/middlewares/` - Application Middlewares**

The **`/middlewares/`** folder contains the middleware components of the application. Middlewares are functions that are executed before or after the main logic of a route handler. They are used to handle cross-cutting concerns such as authentication, logging, rate limiting, and request validation.

### **Responsibilities of Middlewares:**
1. **Authentication and Authorization**: Ensure that the user is authenticated and has the appropriate permissions to access certain routes.
2. **Logging**: Log incoming requests and their details, such as method, URL, and execution time.
3. **Request Validation**: Validate the format and contents of incoming requests before passing them to the route handler.
4. **Error Handling**: Capture and handle errors that may occur during the request lifecycle.
5. **Rate Limiting**: Control the rate of incoming requests to prevent overloading the application.

### **Example Middlewares:**

#### **1. Authentication Middleware (`auth_middleware.go`):**

This middleware checks if the user has a valid authentication token before allowing access to protected routes.

```go
package middlewares

import (
    "net/http"
    "strings"
)

type AuthMiddleware struct {
    // Dependency on a user repository or JWT handler can be injected here
}

func (am *AuthMiddleware) RequireAuth(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        token := r.Header.Get("Authorization")
        if token == "" || !strings.HasPrefix(token, "Bearer ") {
            http.Error(w, "Unauthorized", http.StatusUnauthorized)
            return
        }

        // Token validation logic here

        // If valid, proceed to the next handler
        next.ServeHTTP(w, r)
    })
}
```

### **2. Logging Middleware (`logging_middleware.go`):**

This middleware logs details about the incoming HTTP requests.

```go
package middlewares

import (
    "log"
    "net/http"
    "time"
)

func LoggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        log.Printf("Started %s %s", r.Method, r.URL.Path)
        
        next.ServeHTTP(w, r)

        log.Printf("Completed in %v", time.Since(start))
    })
}
```

### **How to Implement Middlewares:**

1. **Define the Middleware**:
   Middlewares are defined as functions that take in a `http.Handler` and return a `http.Handler`. They are responsible for intercepting requests before they reach the final route handler.

2. **Inject Dependencies**:
   Some middlewares may require access to services or repositories, such as an authentication middleware needing access to a user repository or JWT handler. These dependencies can be injected via the middleware constructor.

3. **Use Middlewares in Routes**:
   To use a middleware, it needs to be wrapped around the handler functions in your route registration. Here’s an example of how to apply a middleware in a route:

   ```go
   package routes

   import (
       "your_project/internal/api/http/middlewares"
       "github.com/gorilla/mux"
       "net/http"
   )

   func RegisterRoutes(router *mux.Router, authMiddleware *middlewares.AuthMiddleware) {
       // Protected route with authentication middleware
       router.Handle("/protected-route", authMiddleware.RequireAuth(http.HandlerFunc(ProtectedHandler))).Methods("GET")

       // Public route without authentication
       router.HandleFunc("/public-route", PublicHandler).Methods("GET")
   }

   func ProtectedHandler(w http.ResponseWriter, r *http.Request) {
       w.Write([]byte("You are authenticated!"))
   }

   func PublicHandler(w http.ResponseWriter, r *http.Request) {
       w.Write([]byte("This is a public route"))
   }
   ```

### **Common Middlewares:**
- **Authentication**: Verifies that a request is made by an authenticated user.
- **Authorization**: Checks if an authenticated user has the necessary permissions to access a specific resource.
- **Logging**: Logs request details such as method, URL, and execution time.
- **Rate Limiting**: Limits the number of requests a user or client can make within a given timeframe.
- **CORS Middleware**: Allows or restricts cross-origin requests based on configured policies.

### **How to Use Middlewares in the Application:**

In your `main.go` or routes configuration, middlewares are typically applied globally or to specific routes.

- **Global Middleware**: Applied to every route in the application.
- **Route-Specific Middleware**: Applied to individual routes or groups of routes that require specific checks (e.g., authentication for protected routes).

```go
func main() {
    router := mux.NewRouter()

    // Apply global logging middleware
    router.Use(middlewares.LoggingMiddleware)

    // Register routes with middleware applied to specific routes
    authMiddleware := &middlewares.AuthMiddleware{}
    routes.RegisterRoutes(router, authMiddleware)

    log.Fatal(http.ListenAndServe(":8080", router))
}
```

### **Summary:**

The middlewares in this folder handle cross-cutting concerns like **authentication**, **authorization**, and **logging**. They are reusable components that intercept requests to perform specific tasks before passing the request to the final route handler.