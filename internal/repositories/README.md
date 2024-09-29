### **`/internal/repositories/` - Data Access Layer (Repositories)**

The **`/repositories/`** folder contains the code responsible for accessing and managing data in the application's persistence layer, such as a database or cache. Repositories provide an abstraction over the persistence mechanism, allowing the application to interact with the data without being tied to specific database implementations. This decouples the business logic from the data source, enabling easier testing and flexibility to change the data storage mechanism.

### **Responsibilities of Repositories:**
1. **Data Access**: Repositories handle all interactions with the database or other persistence layers (e.g., Redis, file storage), providing methods to **create**, **read**, **update**, and **delete** entities.
2. **Abstraction**: Repositories abstract away the details of how data is persisted and retrieved, allowing the application to remain agnostic about the underlying database technology.
3. **Encapsulation of Queries**: All database queries, whether simple or complex, are encapsulated within repositories, keeping the query logic separate from the business logic.
4. **Separation of Concerns**: The repository pattern ensures that the use cases and domain logic remain focused on business rules and not on how data is stored or retrieved.

### **Folder Structure:**
- **`user/`**: Contains the repository responsible for managing user data.
- **`role/`**: Contains the repository responsible for managing role data.
- **`product/`**: Contains the repository responsible for managing product data.

Each subfolder contains specific repositories and query logic for managing the respective entities.

### **Example - `user_repository.go`:**

```go
package repositories

import (
    "your_project/internal/domain/models"
    "gorm.io/gorm"
)

type UserRepository struct {
    DB *gorm.DB
}

// NewUserRepository creates a new instance of UserRepository.
func NewUserRepository(db *gorm.DB) *UserRepository {
    return &UserRepository{DB: db}
}

// Create saves a new user to the database.
func (r *UserRepository) Create(user *models.User) error {
    return r.DB.Create(user).Error
}

// FindByID retrieves a user by its ID.
func (r *UserRepository) FindByID(id uint) (*models.User, error) {
    var user models.User
    if err := r.DB.First(&user, id).Error; err != nil {
        return nil, err
    }
    return &user, nil
}

// FindByEmail retrieves a user by their email.
func (r *UserRepository) FindByEmail(email string) (*models.User, error) {
    var user models.User
    if err := r.DB.Where("email = ?", email).First(&user).Error; err != nil {
        return nil, err
    }
    return &user, nil
}
```

### **Explanation of the Example:**

1. **Repository Struct (`UserRepository`)**:
   - This struct contains a database connection (`*gorm.DB`) and provides methods to perform operations on the `User` entity, such as creating and retrieving users.

2. **`Create` Method**:
   - This method takes a `User` model as input and saves it to the database.

3. **`FindByID` and `FindByEmail` Methods**:
   - These methods retrieve user data from the database based on the user’s ID or email, using GORM’s query functions.

4. **Encapsulation**:
   - All database-related logic is encapsulated within the repository, allowing the rest of the application to simply call `UserRepository.Create()` or `UserRepository.FindByEmail()` without needing to know how the database works.

### **How Repositories Work in the Application:**

1. **Interaction with Use Cases**:
   - Use cases depend on repositories to perform data-related operations. For example, when creating a user, the use case would call `UserRepository.Create()` to save the user to the database.
   
2. **Persistence Abstraction**:
   - Repositories abstract the details of the database, meaning if the underlying storage mechanism changes (e.g., switching from PostgreSQL to MySQL), only the repository implementation needs to change, not the business logic.

3. **Queries Encapsulation**:
   - Complex queries and database-specific logic are encapsulated in the repository, ensuring that the business logic stays clean and focused on domain-specific rules.

### **Example - `product_repository.go`:**

```go
package repositories

import (
    "your_project/internal/domain/models"
    "gorm.io/gorm"
)

type ProductRepository struct {
    DB *gorm.DB
}

// NewProductRepository creates a new instance of ProductRepository.
func NewProductRepository(db *gorm.DB) *ProductRepository {
    return &ProductRepository{DB: db}
}

// Create saves a new product to the database.
func (r *ProductRepository) Create(product *models.Product) error {
    return r.DB.Create(product).Error
}

// FindByID retrieves a product by its ID.
func (r *ProductRepository) FindByID(id uint) (*models.Product, error) {
    var product models.Product
    if err := r.DB.First(&product, id).Error; err != nil {
        return nil, err
    }
    return &product, nil
}
```

This `ProductRepository` works similarly to the `UserRepository`, handling the creation and retrieval of products from the database.

### **How to Implement Repositories:**

1. **Define the Repository Interface**:
   - Define the methods that will be available in the repository (e.g., `Create`, `FindByID`, `FindAll`).

2. **Implement Repository Methods**:
   - Implement the methods using your preferred ORM (e.g., GORM) or raw SQL queries. Each method should handle a specific operation, such as inserting a record or querying the database.

3. **Use Repositories in Use Cases**:
   - The repository should be passed to use cases that need to access or manipulate data. For example, a `CreateUserUseCase` would use the `UserRepository` to persist the new user.

### **Example of Using Repositories in a Use Case:**

```go
package usecases

import (
    "your_project/internal/domain/models"
    "your_project/internal/repositories"
)

type CreateUserUseCase struct {
    UserRepository *repositories.UserRepository
}

// Execute creates a new user by using the UserRepository.
func (uc *CreateUserUseCase) Execute(input CreateUserInput) (*models.User, error) {
    user := &models.User{
        Name:     input.Name,
        Email:    input.Email,
        Password: hashPassword(input.Password), // Logic for hashing the password
    }

    // Use the repository to save the user
    err := uc.UserRepository.Create(user)
    if err != nil {
        return nil, err
    }

    return user, nil
}
```

In this example:
- The `CreateUserUseCase` interacts with the `UserRepository` to persist the user in the database.

### **Summary:**

The **`/repositories/`** folder is responsible for all data access operations in the application. Repositories provide an abstraction over the database, allowing use cases to interact with the persistence layer without needing to know the specifics of the database or storage mechanism. This separation allows for easier testing, maintenance, and flexibility in changing data sources.
