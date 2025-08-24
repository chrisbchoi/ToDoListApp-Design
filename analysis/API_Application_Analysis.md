# API Application Analysis

## Overview

This document analyzes the requirements and design constraints for the API component of the TODO List application. The API will serve as a Backend-For-Frontend (BFF) for the React SPA, providing all necessary endpoints to manage TODO items.

## Technology Stack

- **Framework**: Latest Java SpringBoot
- **API Type**: REST API
- **Package Structure**: Controller, Service, Repository pattern

## API Endpoints Design

### TODO Item Management

#### 1. Get All TODO Items
- **Endpoint**: `GET /api/todos`
- **Parameters**:
  - `page`: Page number (default: 0)
  - `size`: Page size (default: 10)
  - `sort`: Field to sort by (default: "dueDate")
  - `direction`: Sort direction (ASC/DESC)
  - `status`: Filter by status (optional)
- **Response**:
  ```json
  {
    "content": [
      {
        "id": 1,
        "name": "Buy groceries",
        "location": "Supermarket",
        "dueDate": "2025-08-25T10:00:00",
        "status": "PENDING",
        "notes": "Get milk, eggs, and bread"
      }
    ],
    "totalElements": 50,
    "totalPages": 5,
    "currentPage": 0,
    "size": 10
  }
  ```

#### 2. Get TODO Item by ID
- **Endpoint**: `GET /api/todos/{id}`
- **Response**: Individual TODO item object
- **Error Handling**: 404 if item not found

#### 3. Create New TODO Item
- **Endpoint**: `POST /api/todos`
- **Request Body**: TODO item object (without ID)
- **Response**: Created TODO item with generated ID
- **Validation**: Proper validation for required fields

#### 4. Update TODO Item
- **Endpoint**: `PUT /api/todos/{id}`
- **Request Body**: Updated TODO item object
- **Response**: Updated TODO item
- **Error Handling**: 404 if item not found

#### 5. Update TODO Status
- **Endpoint**: `PATCH /api/todos/{id}/status`
- **Request Body**:
  ```json
  {
    "status": "COMPLETED"
  }
  ```
- **Response**: Updated TODO item
- **Error Handling**: 404 if item not found

#### 6. Delete TODO Item
- **Endpoint**: `DELETE /api/todos/{id}`
- **Response**: 204 No Content
- **Error Handling**: 404 if item not found

## Data Transfer Objects (DTOs)

### TodoItemDTO
```java
public class TodoItemDTO {
    private Long id;
    private String name;
    private String location;
    private LocalDateTime dueDate;
    private TodoStatus status;
    private String notes;
    
    // Getters, setters, constructors
}
```

### TodoStatusUpdateDTO
```java
public class TodoStatusUpdateDTO {
    private TodoStatus status;
    
    // Getters, setters, constructors
}
```

### TodoPageResponseDTO
```java
public class TodoPageResponseDTO {
    private List<TodoItemDTO> content;
    private long totalElements;
    private int totalPages;
    private int currentPage;
    private int size;
    
    // Getters, setters, constructors
}
```

## Error Handling

- Implement a global exception handler using `@ControllerAdvice`
- Standard error response format:
  ```json
  {
    "status": 400,
    "message": "Validation failed",
    "errors": [
      "Name is required",
      "Due date must be in the future"
    ],
    "timestamp": "2025-08-24T10:30:45"
  }
  ```

## API Security Considerations

- CORS configuration for frontend access
- Consider rate limiting for API endpoints
- Input validation for all endpoints
- Data sanitization to prevent injection attacks

## Unit Testing Strategy

- Controller tests using MockMvc
- Service layer tests with Mockito
- Integration tests with TestRestTemplate or RestAssured
- Test coverage for happy paths and edge cases
- Validate error handling and response formats

## API Documentation

- Implement OpenAPI/Swagger documentation
- Document all endpoints, request/response models, and error cases
- Provide examples for common operations