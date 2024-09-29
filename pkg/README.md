### **`/pkg/` - Shared Utility Packages**

The **`/pkg/`** folder contains shared utility packages and common functionality that can be used across the application. These packages are typically reusable components that provide general-purpose utilities, such as logging, error handling, or helper functions. The code inside the **`/pkg/`** directory is designed to be reusable and modular, so it can be used in multiple parts of the application or even in other projects.

### **Responsibilities of the `/pkg/` Folder:**
1. **Provide Reusable Utilities**: Utility functions or components that can be shared across different parts of the application to avoid code duplication.
2. **Encapsulate Cross-Cutting Concerns**: Handle concerns such as logging, configuration management, or common middleware that are not tied to a specific domain but are needed throughout the application.
3. **Modularization**: Code within this folder should be designed to be as independent as possible, so it can be easily reused and tested in isolation.

### **Folder Structure:**
- **`log/`**: Contains utilities and packages related to logging.
- **(Additional subfolders may be added)**: The structure can include other shared utilities like custom error handlers, authentication utilities, or third-party service integrations.

### **Example - `log/logger.go`:**

```go
package log

import (
    "github.com/sirupsen/logrus"
    "os"
)

var Logger = logrus.New()

func InitLogger() {
    // Set log format
    Logger.SetFormatter(&logrus.JSONFormatter{})

    // Output to stdout instead of the default stderr
    Logger.SetOutput(os.Stdout)

    // Set log level
    Logger.SetLevel(logrus.InfoLevel)
}
```

### **Explanation of the Example:**

1. **Logger Initialization**:
   - The `InitLogger` function configures the logger by setting its output to `stdout`, using the JSON format for logs, and setting the log level to `Info`.
   - This is a reusable logging utility that can be used across the entire application for consistent logging practices.

2. **Use of Logrus**:
   - The logger is built using **Logrus**, a structured logging library for Go that allows for rich logging with support for various log levels and formats.

3. **Global Logger**:
   - The `Logger` variable is initialized globally so that it can be accessed from anywhere in the application.

### **How Utility Packages Work in the Application:**

1. **Reusable Components**:
   - Utility packages in the **`/pkg/`** folder are designed to be reusable across the application. For example, the logging package (`log`) can be used in any part of the application, such as controllers, services, or repositories, to log events consistently.

2. **Initialization and Setup**:
   - Utility packages like logging may require some initialization at the start of the application. In the case of `Logger`, you might call `log.InitLogger()` in your `main.go` file to configure logging for the entire app.

3. **Modular Design**:
   - The goal of packages in **`/pkg/`** is to keep them independent of the application’s core logic. This allows them to be reused not just across this project, but potentially across different projects as well.

### **How to Use the Logger in the Application:**

Here’s an example of how the logger from **`/pkg/log`** might be used in other parts of the application:

```go
package http

import (
    "net/http"
    "your_project/pkg/log"
)

func ExampleHandler(w http.ResponseWriter, r *http.Request) {
    // Log an informational message
    log.Logger.Info("Handling an example request")

    // Write response
    w.WriteHeader(http.StatusOK)
    w.Write([]byte("Request handled successfully"))
}
```

### **Other Utility Packages**:

You can create additional utility packages in the **`/pkg/`** folder depending on the needs of your application. Some common examples include:

1. **Error Handling**: A package for custom error types and error handling logic.
   - **Example**: `pkg/errors/` for defining custom error types and a standardized way to handle errors throughout the application.

2. **Authentication Utilities**: A package for handling token generation, validation, and user authentication.
   - **Example**: `pkg/auth/` for managing authentication tokens like JWT, token expiration, and validation.

3. **Configuration Management**: A package to handle application configuration, loading environment variables, and managing settings.
   - **Example**: `pkg/config/` for reading and managing configurations from files like `.env` or `config.yaml`.

4. **Middleware Utilities**: A package to define common HTTP middlewares such as logging, CORS, or authentication.
   - **Example**: `pkg/middleware/` to define reusable middlewares across multiple routes.

### **How to Implement Utility Packages:**

1. **Define the Utility Functions or Components**:
   - Start by identifying utility code that is used in multiple places across your application (e.g., logging, error handling, config management) and move it into a separate package.

2. **Make Functions or Variables Public**:
   - Export utility functions and variables by capitalizing the first letter of the function/variable name so that they can be accessed from other parts of the application. For example, `InitLogger()` or `Logger`.

3. **Keep Utilities Modular**:
   - Ensure that the utilities do not have dependencies on the domain logic or specific implementations. The more independent they are, the more reusable they become.

4. **Testable and Reusable**:
   - Write utility code in a way that is easy to test in isolation and can be reused across different parts of the project or other projects.

### **Usage Example in `main.go`:**

Here’s how you might initialize the logging utility from the **`/pkg/log`** package in the `main.go` file:

```go
package main

import (
    "log"
    customLog "your_project/pkg/log"
    "net/http"
    "your_project/internal/api/routes"
    "github.com/gorilla/mux"
)

func main() {
    // Initialize the logger
    customLog.InitLogger()

    // Log application startup
    customLog.Logger.Info("Starting the application...")

    // Create the main router
    router := mux.NewRouter()

    // Register routes
    routes.RegisterUserRoutes(router)

    // Start the server
    log.Fatal(http.ListenAndServe(":8080", router))
}
```

### **Summary:**

The **`/pkg/`** folder contains utility packages that can be shared across the application. These packages provide common functionality such as logging, error handling, and configuration management. By keeping this code modular and reusable, it promotes clean and maintainable code across the application.
