### **`configs/` - Application Configuration**

The **`/configs/`** folder contains the configuration files for the application. These files define important settings, such as database connection details, API credentials, environment variables, and other configuration settings that the application uses to run properly.

#### **Common Responsibilities of Configuration Files:**
1. **Database Settings**: Define connection details like host, username, password, and database name.
2. **API Credentials**: Store sensitive information like API keys, tokens, and third-party service credentials.
3. **Environment Variables**: Manage runtime environment settings (development, staging, production).
4. **Global Application Settings**: Store configurations that affect the global behavior of the application (e.g., logging levels, session settings).

### **Example Configuration File - `config.yaml`:**

A common configuration file is `config.yaml`, which is often used for storing settings in a YAML format. Here’s an example of what it might look like:

```yaml
database:
  host: "localhost"
  port: 5432
  user: "postgres"
  password: "mysecretpassword"
  dbname: "mydb"

redis:
  host: "localhost"
  port: 6379

server:
  port: 8080
  logLevel: "debug"

auth:
  jwtSecret: "supersecretkey"
  tokenExpiry: "24h"
```

### **Explanation of Each Section:**

1. **`database`**: Contains all the database connection details, such as the host, port, user, password, and database name.
2. **`redis`**: Configuration for connecting to a Redis instance, including host and port.
3. **`server`**: Global server settings, such as the port the application will run on and the logging level (e.g., `debug`, `info`, `error`).
4. **`auth`**: Stores authentication-related configuration, such as the JWT secret used for signing tokens and the token expiration time.

### **How the Configuration is Used in the Application:**

In the Go application, you can use a package like **Viper** to load and access the configuration from `config.yaml`:

```go
import (
    "log"
    "github.com/spf13/viper"
)

func initConfig() {
    viper.SetConfigFile("configs/config.yaml")

    if err := viper.ReadInConfig(); err != nil {
        log.Fatalf("Error reading config file: %v", err)
    }

    // Accessing specific values
    dbHost := viper.GetString("database.host")
    dbPort := viper.GetInt("database.port")
    serverPort := viper.GetInt("server.port")
}
```

### **Usage Example:**

1. **Update the Config File**:
   Modify the `config.yaml` file with the necessary settings for your application, such as the database connection details and server settings.

2. **Load Configuration in Code**:
   In your `main.go` or initialization logic, use **Viper** or another configuration package to load the config file and access the values.

3. **Environment-Specific Configurations**:
   You can create multiple configuration files for different environments, like `config.dev.yaml` or `config.prod.yaml`, and switch between them depending on the environment.