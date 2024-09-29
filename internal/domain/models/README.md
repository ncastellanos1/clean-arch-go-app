### **`/internal/domain/models/` - Domain Models**

The **`/models/`** folder contains the **domain models** of the application. Domain models represent the core business entities and encapsulate the logic that pertains to them. These models reflect the structure and relationships of the data that is central to the business domain, such as **users**, **roles**, and **products**.

### **Responsibilities of Domain Models:**
1. **Represent Business Entities**: Define the structure and relationships of entities like `User`, `Role`, `Product`, and others in the business domain.
2. **Encapsulate Domain Logic**: Models can contain basic validation rules or behaviors specific to the entity (e.g., checking permissions, calculating totals).
3. **Database Mapping**: In some cases, domain models may be used with an ORM (like GORM in Go) to map directly to database tables.
4. **Separation of Concerns**: Domain models do not interact directly with the external world (like HTTP requests); instead, they focus solely on the business logic.

### **Folder Structure:**
- **`user.go`**: Defines the `User` entity and its relationships (e.g., with roles or permissions).
- **`role.go`**: Defines the `Role` entity used to categorize users based on permissions.
- **`product.go`**: Defines the `Product` entity for managing product data in an e-commerce or inventory system.

Each file in this folder defines a specific business entity and can include its attributes, relationships, and behaviors.

### **Example - `user.go`:**

```go
package models

import "time"

// User represents a user in the system.
type User struct {
    ID        uint      `gorm:"primaryKey"`
    Name      string    `gorm:"size:100;not null"`
    Email     string    `gorm:"unique;not null"`
    Password  string    `gorm:"not null"`
    CreatedAt time.Time
    UpdatedAt time.Time
    Roles     []Role    `gorm:"many2many:user_roles;"`  // Relationship to roles
}

// HasRole checks if the user has a specific role.
func (u *User) HasRole(roleName string) bool {
    for _, role := range u.Roles {
        if role.Name == roleName {
            return true
        }
    }
    return false
}
```

### **Explanation of the Example:**

1. **Struct Definition (`User`)**:
   - This defines the attributes of the `User` entity, such as `ID`, `Name`, `Email`, and `Password`.
   - The `gorm` tags (e.g., `gorm:"primaryKey"`, `gorm:"not null"`) are used to define how this model should be mapped to a database table when using GORM.

2. **Relationships**:
   - The `Roles` field establishes a **many-to-many relationship** between `User` and `Role`. This allows users to be associated with multiple roles.

3. **Behavior**:
   - The `HasRole` method provides domain-specific behavior to check if a user has a specific role, encapsulating business logic within the model.

### **How Domain Models Work in the Application:**

1. **Interaction with Use Cases**:
   - Domain models are typically used by the use cases to execute business logic. For example, the `User` model may be created or retrieved by a use case and then passed back to the controller for further action.

2. **Interaction with Repositories**:
   - Domain models are often saved to or retrieved from a database using repositories. The repository is responsible for handling the persistence of these models without the use case needing to know the specifics of the database.

3. **Validation and Logic Encapsulation**:
   - Basic validation or entity-specific logic (like the `HasRole` function in the `User` model) can be included in the model itself, ensuring that business rules are enforced consistently.

### **Example - `role.go`:**

```go
package models

import "time"

// Role represents a user's role in the system.
type Role struct {
    ID        uint      `gorm:"primaryKey"`
    Name      string    `gorm:"size:50;unique;not null"`
    CreatedAt time.Time
    UpdatedAt time.Time
}
```

This `Role` model represents a user’s role within the system, with fields like `ID` and `Name`. The model can be extended with additional behaviors or relationships as needed.

### **How to Implement Domain Models:**

1. **Define the Entity**:
   - Define the structure of the entity, listing all the relevant fields like `ID`, `Name`, and `CreatedAt`. Optionally, add ORM-specific tags like `gorm:"primaryKey"` if you are using GORM.

2. **Add Relationships**:
   - Define relationships between different entities. For example, a `User` can have many `Roles`, and a `Product` can belong to a `Category`.

3. **Encapsulate Domain Logic**:
   - Add methods to the model to encapsulate common logic that is specific to the entity. For example, adding a method to check if a user has a particular role or calculating the total price for a product.

4. **Use the Model in Use Cases**:
   - In your use cases, these models are used to perform operations such as creating, updating, or retrieving entities. The models serve as the core data structures that represent the business objects.

### **Example of Using Domain Models in a Use Case:**

```go
package usecases

import (
    "your_project/internal/domain/models"
    "your_project/internal/repositories"
)

type CreateUserUseCase struct {
    UserRepository repositories.UserRepository
}

func (uc *CreateUserUseCase) Execute(input CreateUserInput) (*models.User, error) {
    user := &models.User{
        Name:     input.Name,
        Email:    input.Email,
        Password: hashPassword(input.Password), // Hashing logic omitted
    }

    err := uc.UserRepository.Create(user)
    if err != nil {
        return nil, err
    }

    return user, nil
}
```

In this example, the `CreateUserUseCase` creates a `User` domain model based on input data and then uses the `UserRepository` to persist it in the database.

### **Summary:**

The **`/models/`** folder contains the core **domain models** of the application. These models represent the business entities and encapsulate their associated logic and relationships. They interact with repositories to persist and retrieve data and are used by use cases to perform business operations.

