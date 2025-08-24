# Data Layer Analysis

## Overview

This document analyzes the requirements and design for the data persistence layer of the TODO List application. The data layer will be responsible for storing and retrieving TODO items using Spring Data JPA with an H2 database.

## Technology Stack

- **ORM Framework**: Spring Data JPA
- **Database**: H2 Database (embedded)
- **Entity Mapping**: JPA Annotations
- **Data Access Pattern**: Repository Pattern

## Database Schema Design

### TODO Item Table

**Table Name**: `todo_items`

| Column Name | Data Type | Constraints | Description |
|-------------|-----------|------------|-------------|
| id | BIGINT | PK, AUTO_INCREMENT | Unique identifier for the TODO item |
| name | VARCHAR(255) | NOT NULL | Name/title of the TODO item |
| location | VARCHAR(255) | | Location associated with the TODO item |
| due_date | TIMESTAMP | | Date and time when the TODO item is due |
| status | VARCHAR(20) | NOT NULL | Current status of the TODO item (e.g., PENDING, COMPLETED) |
| notes | TEXT | | Additional notes or description |
| created_at | TIMESTAMP | NOT NULL | When the TODO item was created |
| updated_at | TIMESTAMP | NOT NULL | When the TODO item was last updated |

### Indexes

- Primary key index on `id`
- Index on `status` for filtering
- Index on `due_date` for sorting and filtering

## JPA Entity Design

```java
@Entity
@Table(name = "todo_items")
public class TodoItem {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String name;
    
    private String location;
    
    @Column(name = "due_date")
    private LocalDateTime dueDate;
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private TodoStatus status;
    
    @Column(columnDefinition = "TEXT")
    private String notes;
    
    @Column(name = "created_at", nullable = false, updatable = false)
    @CreationTimestamp
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at", nullable = false)
    @UpdateTimestamp
    private LocalDateTime updatedAt;
    
    // Getters, setters, constructors
}
```

### Enum for Status

```java
public enum TodoStatus {
    PENDING,
    IN_PROGRESS,
    COMPLETED,
    ARCHIVED,
    DELETED
}
```

## Repository Interface

```java
@Repository
public interface TodoRepository extends JpaRepository<TodoItem, Long> {
    
    // Find all todo items with pagination and sorting
    Page<TodoItem> findAll(Pageable pageable);
    
    // Find by status with pagination and sorting
    Page<TodoItem> findByStatus(TodoStatus status, Pageable pageable);
    
    // Find by due date range
    List<TodoItem> findByDueDateBetween(LocalDateTime start, LocalDateTime end);
    
    // Find todos due today
    @Query("SELECT t FROM TodoItem t WHERE DATE(t.dueDate) = CURRENT_DATE")
    List<TodoItem> findTodosDueToday();
    
    // Find overdue todos
    @Query("SELECT t FROM TodoItem t WHERE t.dueDate < CURRENT_TIMESTAMP AND t.status = 'PENDING'")
    List<TodoItem> findOverdueTodos();
    
    // Custom count query
    @Query("SELECT COUNT(t) FROM TodoItem t WHERE t.status = :status")
    long countByStatus(@Param("status") TodoStatus status);
}
```

## Data Mapper

A mapper will be implemented to convert between JPA entities and DTOs:

```java
@Component
public class TodoItemMapper {

    public TodoItemDTO toDto(TodoItem entity) {
        TodoItemDTO dto = new TodoItemDTO();
        dto.setId(entity.getId());
        dto.setName(entity.getName());
        dto.setLocation(entity.getLocation());
        dto.setDueDate(entity.getDueDate());
        dto.setStatus(entity.getStatus());
        dto.setNotes(entity.getNotes());
        return dto;
    }
    
    public TodoItem toEntity(TodoItemDTO dto) {
        TodoItem entity = new TodoItem();
        entity.setId(dto.getId());
        entity.setName(dto.getName());
        entity.setLocation(dto.getLocation());
        entity.setDueDate(dto.getDueDate());
        entity.setStatus(dto.getStatus());
        entity.setNotes(dto.getNotes());
        return entity;
    }
    
    public List<TodoItemDTO> toDtoList(List<TodoItem> entities) {
        return entities.stream()
                .map(this::toDto)
                .collect(Collectors.toList());
    }
    
    public Page<TodoItemDTO> toDtoPage(Page<TodoItem> entityPage) {
        return entityPage.map(this::toDto);
    }
}
```

## Database Configuration

H2 database configuration in `application.properties`:

```properties
# H2 Database Configuration
spring.datasource.url=jdbc:h2:file:./todoapp;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=password
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# JPA Configuration
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# Connection Pooling
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5
```

## Data Migration Strategy

- Use Flyway or Liquibase for database schema migrations
- Initial schema creation script
- Data seeding for testing or initial setup

## Data Access Patterns

### Service Layer Implementation

```java
@Service
public class TodoServiceImpl implements TodoService {

    private final TodoRepository todoRepository;
    private final TodoItemMapper todoItemMapper;
    
    @Autowired
    public TodoServiceImpl(TodoRepository todoRepository, TodoItemMapper todoItemMapper) {
        this.todoRepository = todoRepository;
        this.todoItemMapper = todoItemMapper;
    }
    
    @Override
    @Transactional(readOnly = true)
    public Page<TodoItemDTO> getAllTodos(int page, int size, String sortBy, String direction, TodoStatus status) {
        Sort.Direction sortDirection = direction.equalsIgnoreCase("DESC") ? Sort.Direction.DESC : Sort.Direction.ASC;
        Pageable pageable = PageRequest.of(page, size, Sort.by(sortDirection, sortBy));
        
        Page<TodoItem> todoPage;
        if (status != null) {
            todoPage = todoRepository.findByStatus(status, pageable);
        } else {
            todoPage = todoRepository.findAll(pageable);
        }
        
        return todoItemMapper.toDtoPage(todoPage);
    }
    
    @Override
    @Transactional(readOnly = true)
    public TodoItemDTO getTodoById(Long id) {
        return todoRepository.findById(id)
                .map(todoItemMapper::toDto)
                .orElseThrow(() -> new ResourceNotFoundException("Todo item not found with id: " + id));
    }
    
    @Override
    @Transactional
    public TodoItemDTO createTodo(TodoItemDTO todoItemDTO) {
        TodoItem todoItem = todoItemMapper.toEntity(todoItemDTO);
        todoItem.setStatus(TodoStatus.PENDING); // Default status
        TodoItem savedTodoItem = todoRepository.save(todoItem);
        return todoItemMapper.toDto(savedTodoItem);
    }
    
    @Override
    @Transactional
    public TodoItemDTO updateTodo(Long id, TodoItemDTO todoItemDTO) {
        TodoItem existingTodoItem = todoRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Todo item not found with id: " + id));
        
        existingTodoItem.setName(todoItemDTO.getName());
        existingTodoItem.setLocation(todoItemDTO.getLocation());
        existingTodoItem.setDueDate(todoItemDTO.getDueDate());
        existingTodoItem.setStatus(todoItemDTO.getStatus());
        existingTodoItem.setNotes(todoItemDTO.getNotes());
        
        TodoItem updatedTodoItem = todoRepository.save(existingTodoItem);
        return todoItemMapper.toDto(updatedTodoItem);
    }
    
    @Override
    @Transactional
    public TodoItemDTO updateTodoStatus(Long id, TodoStatus status) {
        TodoItem existingTodoItem = todoRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Todo item not found with id: " + id));
        
        existingTodoItem.setStatus(status);
        TodoItem updatedTodoItem = todoRepository.save(existingTodoItem);
        return todoItemMapper.toDto(updatedTodoItem);
    }
    
    @Override
    @Transactional
    public void deleteTodo(Long id) {
        TodoItem todoItem = todoRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Todo item not found with id: " + id));
        
        todoRepository.delete(todoItem);
    }
}
```

## Testing Strategy

- Repository layer tests with `@DataJpaTest`
- Integration tests for database operations
- Test data initialization using `TestEntityManager`
- Transaction management testing
- Query performance testing

## Performance Considerations

- Pagination for large result sets
- Indexing for frequently queried fields
- Query optimization for complex searches
- Connection pooling configuration
- Consider caching for frequently accessed data

## Data Security

- Sensitive data handling (if applicable)
- Database connection security
- SQL injection prevention (handled by JPA/Hibernate)
- User data isolation (for future multi-user support)