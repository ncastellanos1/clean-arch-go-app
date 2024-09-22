### **`/internal/api/http/` - HTTP Layer of the Application**

The **`/http/`** folder contains all the components responsible for handling incoming HTTP requests and sending responses back to the client. This is where the **controllers**, **middlewares**, and **route handlers** are defined and managed. This layer acts as the interface between the outside world (HTTP requests) and the internal application logic (use cases and services).

### **Components in the HTTP Layer:**

1. **Controllers**: 
   - Handle specific routes and manage the flow of HTTP requests. Controllers typically receive requests, validate input, call the appropriate use cases, and return the HTTP response.
   - Example controllers: `user_controller.go`, `role_controller.go`, `product_controller.go`.

2. **Middlewares**:
   - Middleware functions are executed before or after the main logic of a route. They handle tasks like authentication, logging, and request validation. Middlewares can be applied globally or on specific routes.
   - Middlewares are located in the `/middlewares/` folder, such as `auth_middleware.go` and `logging_middleware.go`.

3. **Route Handlers**:
   - Route handlers define the available routes for the API and map them to the corresponding controllers and middlewares. They determine how different endpoints are processed by the application.
   - Routes are defined in the `/routes/` folder, where different modules (users, roles, products) have their own route definitions.

### **Folder Structure:**

- **`controllers/`**: Contains controllers that handle different HTTP requests for specific entities (users, roles, products).
- **`middlewares/`**: Handles reusable middleware logic that can be applied across the application.
- **`routes/`**: Responsible for defining the API routes and mapping them to the appropriate controllers and middlewares.

### **Controller Example - `user_controller.go`:**

```go
package http

import (
    "encoding/json"
    "net/http"
    "your_project/internal/usecases"
    "your_project/internal/domain/dto"
)

type UserController struct {
    CreateUserUseCase usecases.CreateUserUseCase
}

func NewUserController(useCase usecases.CreateUserUseCase) *UserController {
    return &UserController{CreateUserUseCase: useCase}
}

func (uc *UserController) CreateUserHandler(w http.ResponseWriter, r *http.Request) {
    var req dto.CreateUserRequest

    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, "Invalid request", http.StatusBadRequest)
        return
    }

    user, err := uc.CreateUserUseCase.Execute(req)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(user)
}
```

### **How to Implement HTTP Controllers:**

1. **Define the Controller**:
   Controllers receive requests, validate the input, and pass the data to the appropriate use cases or services. After processing the data, they generate a response to send back to the client.

2. **Map Routes to Controllers**:
   Once the controller functions are defined, they are mapped to specific API routes using a routing library like `gorilla/mux`.

3. **Apply Middlewares**:
   Middleware can be applied globally or to individual routes for handling common concerns like authentication or logging.

### **How HTTP Layer Works:**

1. **Client Request**: The client sends an HTTP request to the server (e.g., a `POST /users` request to create a new user).
   
2. **Middleware Execution**: If any middleware (e.g., authentication or logging) is applied to the route, it will run first to validate or log the request.

3. **Controller Handling**: Once the request passes through the middlewares, it is handled by the appropriate controller, which calls the use case logic to process the request.

4. **Response to Client**: The controller processes the result of the use case and sends back an appropriate HTTP response (e.g., a 200 OK status with the newly created user data).

### **Usage Example**:

In your `main.go`, you set up the router to use the controllers and middlewares. For example:

```go
package main

import (
    "log"
    "net/http"
    "your_project/internal/api/http"
    "your_project/internal/api/routes"
    "your_project/internal/usecases"
    "github.com/gorilla/mux"
)

func main() {
    router := mux.NewRouter()

    // Initialize the user controller
    userUseCase := usecases.NewCreateUserUseCase() // You can inject repositories here
    userController := http.NewUserController(userUseCase)

    // Define the route for user creation
    router.HandleFunc("/users", userController.CreateUserHandler).Methods("POST")

    // Start the server
    log.Fatal(http.ListenAndServe(":8080", router))
}
```

### **Summary:**

The **HTTP layer** is responsible for handling incoming requests, managing middlewares, and interacting with use cases. Controllers in this layer are mapped to routes and are responsible for managing the request-response cycle, ensuring that the data is validated and passed to the relevant business logic.