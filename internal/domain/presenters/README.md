### **`/internal/domain/presenters/` - Data Presentation Layer**

The **`/presenters/`** folder contains the **presenters** responsible for transforming domain models into **Data Transfer Objects (DTOs)** or other formats that are suitable for external consumption, typically by clients of the API. Presenters play a crucial role in ensuring that the internal details of domain models are not directly exposed to the outside world, and instead, only the necessary and formatted data is returned.

### **Responsibilities of Presenters:**
1. **Transform Domain Models**: Convert domain models (e.g., `User`, `Product`) into DTOs that are appropriate for returning in HTTP responses.
2. **Filter Sensitive Information**: Remove sensitive fields (e.g., passwords, internal IDs) from the domain models when preparing data for external systems.
3. **Format Data**: Presenters can also handle formatting, such as formatting dates, currencies, or other data in a way that is user-friendly.
4. **Maintain Separation**: Presenters help maintain a clear separation between the domain/business logic and the external API representation.

### **Folder Structure:**
- **`user_presenter.go`**: Handles the presentation of user-related data, transforming user models into user DTOs.
- **`role_presenter.go`**: Handles the presentation of role-related data.
- **`product_presenter.go`**: Handles the presentation of product-related data.

Each presenter file is responsible for transforming domain models into response DTOs for its specific entity.

### **Example - `user_presenter.go`:**

```go
package presenters

import (
    "your_project/internal/domain/dto"
    "your_project/internal/domain/models"
)

// UserPresenter is responsible for transforming user models into DTOs.
type UserPresenter struct{}

// NewUserPresenter creates a new instance of UserPresenter.
func NewUserPresenter() *UserPresenter {
    return &UserPresenter{}
}

// ToUserResponse transforms a User model into a CreateUserResponse DTO.
func (up *UserPresenter) ToUserResponse(user *models.User) dto.CreateUserResponse {
    return dto.CreateUserResponse{
        ID:    user.ID,
        Name:  user.Name,
        Email: user.Email,
    }
}
```

### **Explanation of the Example:**

1. **UserPresenter Struct**:
   - This struct is responsible for transforming `User` domain models into `CreateUserResponse` DTOs.

2. **`ToUserResponse` Method**:
   - This method takes a `User` model and converts it into a `CreateUserResponse` DTO, removing sensitive information like passwords and ensuring that only the necessary fields (e.g., `ID`, `Name`, `Email`) are returned to the client.

3. **Separation of Concerns**:
   - The presenter ensures that the transformation from domain model to DTO is handled in a separate layer, keeping the business logic (use cases) and external interfaces (HTTP) decoupled.

### **How Presenters Work in the Application:**

1. **Receive Domain Models**:
   - Presenters receive fully processed domain models from the use cases. The domain models contain all the relevant data about the entity (e.g., `User`, `Product`).

2. **Transform to DTOs**:
   - The presenter’s job is to convert these domain models into **DTOs** or **formatted objects** that will be returned as HTTP responses. It ensures that only the necessary information is sent back to the client.

3. **Format and Filter Data**:
   - Presenters may also format the data (e.g., formatting dates, hiding internal fields, or converting data types) to ensure that the output is user-friendly and secure.

### **Example - `product_presenter.go`:**

```go
package presenters

import (
    "your_project/internal/domain/dto"
    "your_project/internal/domain/models"
)

// ProductPresenter transforms Product models into DTOs.
type ProductPresenter struct{}

// NewProductPresenter creates a new instance of ProductPresenter.
func NewProductPresenter() *ProductPresenter {
    return &ProductPresenter{}
}

// ToProductResponse transforms a Product model into a ProductResponse DTO.
func (pp *ProductPresenter) ToProductResponse(product *models.Product) dto.ProductResponse {
    return dto.ProductResponse{
        ID:          product.ID,
        Name:        product.Name,
        Description: product.Description,
        Price:       product.Price,
        CreatedAt:   product.CreatedAt.Format("2006-01-02"),
    }
}
```

In this example:
- The `ProductPresenter` transforms the `Product` domain model into a `ProductResponse` DTO. 
- It also formats the `CreatedAt` date to be more user-friendly when returned to the client.

### **How to Implement Presenters:**

1. **Define a Presenter**:
   - Create a struct that represents the presenter for a particular entity (e.g., `UserPresenter`, `ProductPresenter`).

2. **Create Transformation Methods**:
   - Define methods that take a domain model (e.g., `User`, `Product`) and transform it into a DTO or another suitable format. These methods should only return the necessary data and format it appropriately.

3. **Use Presenters in Controllers**:
   - In the controller layer, after the use case processes the domain model, the presenter is used to transform the model into a response DTO before sending it back to the client.

### **Usage Example:**

Here’s an example of how a presenter might be used in a controller:

```go
package http

import (
    "encoding/json"
    "net/http"
    "your_project/internal/usecases"
    "your_project/internal/domain/presenters"
)

type ProductController struct {
    GetProductUseCase usecases.GetProductUseCase
    ProductPresenter  presenters.ProductPresenter
}

func (pc *ProductController) GetProductHandler(w http.ResponseWriter, r *http.Request) {
    productID := r.URL.Query().Get("id")

    // Retrieve the product using the use case
    product, err := pc.GetProductUseCase.Execute(productID)
    if err != nil {
        http.Error(w, "Product not found", http.StatusNotFound)
        return
    }

    // Transform the product model into a response DTO
    productResponse := pc.ProductPresenter.ToProductResponse(product)

    // Send the response to the client
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(productResponse)
}
```

In this example:
- The controller receives a `Product` from the use case.
- The `ProductPresenter` transforms the `Product` domain model into a `ProductResponse` DTO.
- The DTO is then returned to the client as a JSON response.

### **Summary:**

The **`/presenters/`** folder is responsible for transforming domain models into data formats (DTOs) that are returned to clients. Presenters ensure that sensitive data is removed, data is formatted properly, and there is a clean separation between the internal domain models and the external API representation.