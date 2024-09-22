### **`/internal/domain/dto/` - Data Transfer Objects (DTOs)**

The **`/dto/`** folder contains the **Data Transfer Objects (DTOs)** used in the application. DTOs are simple objects used to carry data between processes or layers, such as from the controller to the use cases, or between the client and server. They help ensure that the application only exposes or receives the required data in a structured format, minimizing exposure of internal logic or models.

### **Responsibilities of DTOs:**
1. **Input Validation**: Define the structure of incoming data and ensure that the expected data types and fields are present.
2. **Data Encapsulation**: Encapsulate the data that is sent and received between the client and the server, isolating the domain models from external entities.
3. **Security**: By using DTOs, sensitive fields in domain models (e.g., passwords, internal IDs) can be omitted from external responses.
4. **Mapping**: Provide a clear separation between internal domain models and external-facing data formats, making it easier to map or transform data between them.

### **Folder Structure:**
- **`user_dto.go`**: Defines DTOs for user-related requests and responses (e.g., user creation, user updates).
- **`role_dto.go`**: Defines DTOs for role-related data.
- **`product_dto.go`**: Defines DTOs for product-related data.

Each DTO file contains the necessary structures for requests and responses related to specific entities, like users, roles, and products.

### **Example - `user_dto.go`:**

```go
package dto

// CreateUserRequest is the DTO used to capture data for creating a new user
type CreateUserRequest struct {
    Name     string `json:"name" validate:"required,min=3"`
    Email    string `json:"email" validate:"required,email"`
    Password string `json:"password" validate:"required,min=8"`
}

// CreateUserResponse is the DTO used to return data after creating a new user
type CreateUserResponse struct {
    ID    uint   `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}
```

### **Explanation of the Example:**

1. **`CreateUserRequest`**:
   - This structure defines the expected input when a user is being created. Fields like `Name`, `Email`, and `Password` are required, and input validation can be enforced using tags (e.g., `validate:"required,min=3"` ensures that the name is at least 3 characters long).

2. **`CreateUserResponse`**:
   - This structure defines the data that is returned to the client after successfully creating a user. It includes non-sensitive fields such as the `ID`, `Name`, and `Email`, but omits sensitive information like the password.

### **How DTOs Work in the Application:**

1. **Receive Input in Controllers**:
   - In controllers, DTOs are used to capture incoming request data, ensuring that it is properly structured and validated before passing it to the use cases.

   ```go
   func (uc *UserController) CreateUserHandler(w http.ResponseWriter, r *http.Request) {
       var req dto.CreateUserRequest

       // Decode the incoming JSON payload into the CreateUserRequest DTO
       if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
           http.Error(w, "Invalid request", http.StatusBadRequest)
           return
       }

       // Validate the request DTO (this could use a validation library)
       if err := validate.Struct(req); err != nil {
           http.Error(w, "Validation failed", http.StatusBadRequest)
           return
       }

       // Pass the validated DTO to the use case
       user, err := uc.CreateUserUseCase.Execute(req)
       if err != nil {
           http.Error(w, err.Error(), http.StatusInternalServerError)
           return
       }

       // Return the response DTO
       resp := dto.CreateUserResponse{
           ID:    user.ID,
           Name:  user.Name,
           Email: user.Email,
       }
       json.NewEncoder(w).Encode(resp)
   }
   ```

2. **Send Output from Use Cases**:
   - Once the use case processes the input data, it may return the results using response DTOs to ensure that the output is consistent and secure.

3. **Validate Data**:
   - DTOs often include validation rules to ensure that incoming data adheres to business requirements (e.g., email format, password length, required fields).

### **How to Implement DTOs:**

1. **Define Request DTOs**:
   - Define DTOs that specify what fields are required for each incoming request. These should represent the data that the API expects from the client (e.g., for creating or updating a user).

2. **Define Response DTOs**:
   - Define DTOs for returning data back to the client, omitting any internal fields that should not be exposed (e.g., passwords, internal IDs).

3. **Use DTOs in Controllers**:
   - In controllers, map incoming HTTP request payloads to DTOs and validate them before passing them to the use case layer.

4. **Transform Domain Models to DTOs**:
   - In use cases or presenters, map the domain models (internal application objects) into DTOs before sending the data back to the client. This ensures that only the necessary data is exposed.

### **Usage Example:**

1. **Create a User Request**:
   - When the client makes a `POST` request to create a user, the request body is matched against the `CreateUserRequest` DTO. If the request data is valid, it is passed to the use case.

   ```json
   POST /users
   {
       "name": "John Doe",
       "email": "john.doe@example.com",
       "password": "supersecretpassword"
   }
   ```

2. **Response Example**:
   - After the user is created, a response is returned using the `CreateUserResponse` DTO, ensuring that sensitive data like the password is not included in the response.

   ```json
   {
       "id": 1,
       "name": "John Doe",
       "email": "john.doe@example.com"
   }
   ```

### **Summary:**

The **`/dto/`** folder is responsible for defining the data structures used to transfer data between the client and server, as well as between the different layers of the application. These **Data Transfer Objects (DTOs)** help ensure that data is properly validated, encapsulated, and mapped, providing a clear contract for how data should be structured at the boundaries of the system.