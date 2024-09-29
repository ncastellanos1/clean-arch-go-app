### **`/internal/usecases/` - Application Use Cases**

The **`/usecases/`** folder contains the **application use cases** that define the business logic of the application. Use cases represent the actions that a user or system can perform, and they are responsible for orchestrating the flow of data between the repositories and the domain models. Each use case corresponds to a specific operation in the system (e.g., creating a user, updating a product).

### **Responsibilities of Use Cases:**
1. **Encapsulate Business Logic**: Use cases are the central place where business logic is implemented. They are responsible for ensuring that the application behaves according to business rules and policies.
2. **Coordinate Data Flow**: Use cases coordinate the flow of data between repositories, services, and domain models, ensuring that the right data is retrieved, modified, and persisted.
3. **Isolate Domain Logic**: Use cases isolate the domain logic from the infrastructure and delivery layers (e.g., HTTP controllers), making the application easier to maintain and test.
4. **Invoke Services**: Use cases can interact with services to handle complex operations that involve multiple repositories or other components.

### **Folder Structure:**
- **`user/`**: Contains use cases for user-related operations (e.g., creating a user, retrieving user details).
- **`role/`**: Contains use cases for managing roles (e.g., assigning roles to users).
- **`product/`**: Contains use cases for product-related actions (e.g., creating a product, updating stock).

Each subfolder corresponds to a specific entity or domain of the application, and contains use cases related to that domain.

### **Example - `user/create_user.go`:**

```go
package usecases

import (
    "your_project/internal/domain/models"
    "your_project/internal/repositories"
)

type CreateUserUseCase struct {
    UserRepository *repositories.UserRepository
}

// NewCreateUserUseCase initializes a new CreateUserUseCase.
func NewCreateUserUseCase(repo *repositories.UserRepository) *CreateUserUseCase {
    return &CreateUserUseCase{UserRepository: repo}
}

// Execute creates a new user and saves it to the repository.
func (uc *CreateUserUseCase) Execute(input CreateUserInput) (*models.User, error) {
    user := &models.User{
        Name:     input.Name,
        Email:    input.Email,
        Password: hashPassword(input.Password), // Assume hashing logic is handled here
    }

    // Use the repository to save the user
    if err := uc.UserRepository.Create(user); err != nil {
        return nil, err
    }

    return user, nil
}

// CreateUserInput defines the input structure for creating a user.
type CreateUserInput struct {
    Name     string
    Email    string
    Password string
}
```

### **Explanation of the Example:**

1. **CreateUserUseCase Struct**:
   - This struct contains a reference to the `UserRepository`, which is used to perform database operations related to users.

2. **`Execute` Method**:
   - The `Execute` method is where the actual business logic of creating a user is performed. It takes an input DTO (`CreateUserInput`), creates a `User` model, and saves it to the database using the `UserRepository`.

3. **Input and Output**:
   - The `CreateUserInput` struct represents the data needed to create a user, while the output is the newly created `User` model.

4. **Separation of Concerns**:
   - The use case handles the business logic for creating a user, keeping this logic separate from the controller (which handles HTTP requests) and the repository (which interacts with the database).

### **How Use Cases Work in the Application:**

1. **Interaction with Repositories**:
   - Use cases interact with repositories to perform data operations. For example, when creating a user, the use case will use the `UserRepository` to save the new user to the database.

2. **Encapsulation of Business Logic**:
   - Use cases encapsulate the business logic, ensuring that business rules are enforced in a single place. This makes the application easier to maintain and test, as the logic is centralized.

3. **Separation of Layers**:
   - Use cases are part of the application core and do not depend on external layers such as HTTP controllers or databases. This allows the core logic to remain decoupled from infrastructure concerns.

### **Example - `product/create_product.go`:**

```go
package usecases

import (
    "your_project/internal/domain/models"
    "your_project/internal/repositories"
)

type CreateProductUseCase struct {
    ProductRepository *repositories.ProductRepository
}

// NewCreateProductUseCase initializes a new CreateProductUseCase.
func NewCreateProductUseCase(repo *repositories.ProductRepository) *CreateProductUseCase {
    return &CreateProductUseCase{ProductRepository: repo}
}

// Execute creates a new product and saves it to the repository.
func (uc *CreateProductUseCase) Execute(input CreateProductInput) (*models.Product, error) {
    product := &models.Product{
        Name:        input.Name,
        Description: input.Description,
        Price:       input.Price,
    }

    // Save the product to the repository
    if err := uc.ProductRepository.Create(product); err != nil {
        return nil, err
    }

    return product, nil
}

// CreateProductInput defines the input structure for creating a product.
type CreateProductInput struct {
    Name        string
    Description string
    Price       float64
}
```

In this example:
- The `CreateProductUseCase` handles the logic of creating a product and saving it to the database using the `ProductRepository`.
- The `CreateProductInput` struct defines the input data needed to create the product.

### **How to Implement Use Cases:**

1. **Define the Use Case Struct**:
   - Create a struct that represents the use case. This struct typically contains references to repositories or services that the use case will interact with.

2. **Create Business Logic Methods**:
   - Implement methods on the use case struct (e.g., `Execute`) that contain the business logic for performing specific actions (e.g., creating a user, updating a product).

3. **Use Repositories and Services**:
   - The use case interacts with repositories to perform data operations and with services to perform more complex operations that span multiple domains.

4. **Use Input/Output DTOs**:
   - Define input DTOs that represent the data required for the use case, and return domain models or output DTOs as the result of the operation.

### **Usage Example:**

Here’s an example of how a controller might use a use case:

```go
package http

import (
    "encoding/json"
    "net/http"
    "your_project/internal/usecases"
)

type ProductController struct {
    CreateProductUseCase *usecases.CreateProductUseCase
}

func (pc *ProductController) CreateProductHandler(w http.ResponseWriter, r *http.Request) {
    var input usecases.CreateProductInput

    // Decode the JSON request body into the input DTO
    if err := json.NewDecoder(r.Body).Decode(&input); err != nil {
        http.Error(w, "Invalid input", http.StatusBadRequest)
        return
    }

    // Execute the use case
    product, err := pc.CreateProductUseCase.Execute(input)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    // Return the created product as JSON
    json.NewEncoder(w).Encode(product)
}
```

In this example:
- The `ProductController` uses the `CreateProductUseCase` to handle the business logic of creating a product and returning it to the client.

### **Summary:**

The **`/usecases/`** folder contains the core business logic of the application, organized into specific use cases. Use cases are responsible for handling actions that involve retrieving, manipulating, and persisting data, and they serve as the orchestrators of business logic, ensuring that domain rules are enforced.
