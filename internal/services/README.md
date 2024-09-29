### **`/internal/services/` - Service Layer**

The **`/services/`** folder contains the **service layer** of the application, responsible for coordinating and executing complex business operations that may involve multiple use cases or require additional logic beyond simple data manipulation. Services are typically used when an operation requires interaction across multiple domains, or when handling logic that doesn't fit cleanly into a single use case.

### **Responsibilities of the Service Layer:**
1. **Coordinate Business Logic**: Services handle more complex business operations that span across multiple use cases, repositories, or domains.
2. **Encapsulate Cross-Domain Logic**: If an operation needs to involve multiple entities (e.g., users, roles, products), the service layer acts as the orchestrator.
3. **Abstract Complex Operations**: Services may abstract operations that involve external APIs, messaging systems, or other infrastructure components.
4. **Enhance Reusability**: Business logic that spans across different use cases can be centralized within services to make it reusable and easier to maintain.

### **Folder Structure:**
- **`user_service.go`**: Contains business logic related to users that spans multiple use cases.
- **`role_service.go`**: Contains business logic related to roles.
- **`product_service.go`**: Handles business logic related to products, such as product creation or stock management.

Each file in this folder is responsible for handling operations that involve specific entities, like users, roles, or products.

### **Example - `user_service.go`:**

```go
package services

import (
    "your_project/internal/domain/models"
    "your_project/internal/repositories"
)

type UserService struct {
    UserRepository *repositories.UserRepository
    RoleRepository *repositories.RoleRepository
}

// NewUserService creates a new instance of UserService.
func NewUserService(userRepo *repositories.UserRepository, roleRepo *repositories.RoleRepository) *UserService {
    return &UserService{
        UserRepository: userRepo,
        RoleRepository: roleRepo,
    }
}

// AssignRoleToUser assigns a role to a user.
func (s *UserService) AssignRoleToUser(userID uint, roleName string) error {
    // Retrieve the user from the repository
    user, err := s.UserRepository.FindByID(userID)
    if err != nil {
        return err
    }

    // Retrieve the role by name from the repository
    role, err := s.RoleRepository.FindByName(roleName)
    if err != nil {
        return err
    }

    // Add the role to the user's roles
    user.Roles = append(user.Roles, *role)

    // Save the updated user back to the repository
    return s.UserRepository.Update(user)
}
```

### **Explanation of the Example:**

1. **UserService Struct**:
   - The `UserService` struct contains references to the `UserRepository` and `RoleRepository`, allowing it to coordinate operations involving both users and roles.

2. **`AssignRoleToUser` Method**:
   - This method is responsible for assigning a role to a user. It retrieves the user and role from their respective repositories, updates the user with the new role, and then saves the changes.

3. **Cross-Domain Logic**:
   - The service handles a business operation that spans across multiple entities: users and roles. It encapsulates the logic for retrieving both a user and a role, ensuring that the operation is completed atomically.

### **How Services Work in the Application:**

1. **Interaction with Use Cases**:
   - Use cases can call services when they need to perform a complex business operation that spans across multiple repositories or requires additional logic beyond simple CRUD operations.

2. **Abstraction of Logic**:
   - Services abstract the complexity of multi-step business operations, making it easier for use cases to handle individual tasks without needing to know how everything is orchestrated.

3. **Coordination Across Domains**:
   - Services are responsible for orchestrating actions across multiple domains (e.g., users, roles, products), making them ideal for operations that require interaction between different parts of the system.

### **Example - `product_service.go`:**

```go
package services

import (
    "your_project/internal/domain/models"
    "your_project/internal/repositories"
)

type ProductService struct {
    ProductRepository *repositories.ProductRepository
    StockRepository   *repositories.StockRepository
}

// NewProductService creates a new instance of ProductService.
func NewProductService(productRepo *repositories.ProductRepository, stockRepo *repositories.StockRepository) *ProductService {
    return &ProductService{
        ProductRepository: productRepo,
        StockRepository:   stockRepo,
    }
}

// CreateProductAndAddStock creates a new product and updates the stock.
func (s *ProductService) CreateProductAndAddStock(product *models.Product, stockAmount int) error {
    // Create the product in the database
    if err := s.ProductRepository.Create(product); err != nil {
        return err
    }

    // Update the stock for the new product
    stock := models.Stock{
        ProductID: product.ID,
        Quantity:  stockAmount,
    }

    return s.StockRepository.Create(&stock)
}
```

In this example:
- The `ProductService` handles the creation of a product and its associated stock in a single operation. It interacts with both the `ProductRepository` and the `StockRepository` to ensure that both the product and its stock are properly managed.

### **How to Implement Services:**

1. **Define the Service**:
   - Define the service struct that contains references to the repositories or other services it will interact with.

2. **Create Business Logic Methods**:
   - Add methods that encapsulate complex business operations. These methods should coordinate interactions between multiple repositories or external services.

3. **Use Services in Use Cases**:
   - Use cases can invoke services to perform multi-step operations that require business logic that spans across multiple domains (e.g., assigning roles to users, creating products and managing stock).

### **Usage Example:**

Here’s an example of how a use case might call a service:

```go
package usecases

import (
    "your_project/internal/domain/models"
    "your_project/internal/services"
)

type CreateProductUseCase struct {
    ProductService *services.ProductService
}

// Execute creates a product and updates its stock using the ProductService.
func (uc *CreateProductUseCase) Execute(input CreateProductInput) (*models.Product, error) {
    product := &models.Product{
        Name:        input.Name,
        Description: input.Description,
        Price:       input.Price,
    }

    // Use the ProductService to create the product and add stock
    err := uc.ProductService.CreateProductAndAddStock(product, input.StockAmount)
    if err != nil {
        return nil, err
    }

    return product, nil
}
```

In this example:
- The `CreateProductUseCase` interacts with the `ProductService` to handle the creation of a product and updating its stock.

### **Summary:**

The **`/services/`** folder contains the business logic that spans multiple domains or requires complex operations involving multiple repositories. Services are responsible for coordinating and executing operations that are too complex or broad to fit into a single use case, and they ensure that logic is reusable and maintainable across the application.
