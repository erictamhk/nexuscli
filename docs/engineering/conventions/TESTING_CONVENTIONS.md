# Testing Conventions

## Overview

Nexus follows comprehensive testing conventions to ensure code quality, maintainability, and reliability. Tests are written following Test-Driven Development (TDD) principles and serve as executable specifications.

## Test Organization

### Separate Test Package Structure

Test structure mirrors main source structure for easy navigation:

```
src/
├── features/
│   ├── task/
│   │   ├── domain/
│   │   │   ├── Task.ts
│   │   │   └── TaskId.ts
│   │   ├── usecases/
│   │   │   ├── AddTaskService.ts
│   │   │   └── ShowService.ts
│   │   └── adapters/
│   │       └── controllers/
│   │           └── AddConsoleController.ts
│   └── tests/              # All tests co-located with feature
│       ├── unit/
│       │   ├── domain/
│       │   │   ├── Task.test.ts
│       │   │   └── TaskId.test.ts
│       │   ├── usecases/
│       │   │   ├── AddTaskService.test.ts
│       │   │   └── ShowService.test.ts
│       │   └── adapters/
│       │       └── AddConsoleController.test.ts
│       ├── integration/
│       │   └── usecases/
│       │       └── TaskUseCases.test.ts
│       └── e2e/
│           └── TaskFeature.test.ts
```

### Unit Tests

- **Location**: `src/features/xxx/tests/unit/`
- **Purpose**: Test individual components in isolation
- **Scope**: Single class or function
- **Dependencies**: Mocked or stubbed

### Integration Tests

- **Location**: `src/features/xxx/tests/integration/`
- **Purpose**: Test business logic flows
- **Scope**: Use cases and their interactions
- **Dependencies**: Real repositories (in-memory or test database)

### E2E Tests

- **Location**: `src/features/xxx/tests/e2e/`
- **Purpose**: Test complete feature workflows
- **Scope**: Multiple use cases working together
- **Dependencies**: Real infrastructure (test database, etc.)

## Test Structure

### Arrange-Act-Assert Pattern

All tests follow the AAA pattern for clarity:

```typescript
@Test
public descriptive_test_name_with_underscores(): void {
  // Arrange - Set up test data and dependencies
  const task = Task.create('Test task');
  const repository = new InMemoryTaskRepository();
  repository.save(task);

  // Act - Execute the code being tested
  const found = repository.findById(task.id);

  // Assert - Verify the expected outcome
  assertNotNull(found);
  assertEquals(task.id, found.id);
}
```

### Descriptive Test Method Names

- **Describe behavior**: What is being tested, not how
- **Use underscores**: Separate words for readability
- **Test one thing**: Single responsibility
- **Include edge cases**: Clearly describe what's being tested

#### Examples

```typescript
// GOOD - Descriptive test names
@Test
public add_task_to_project_increases_task_count(): void {
  // ...
}

@Test
public add_task_with_empty_description_throws_exception(): void {
  // ...
}

@Test
public add_task_to_nonexistent_project_returns_not_found_error(): void {
  // ...
}

@Test
public set_task_status_to_done_updates_completion_date(): void {
  // ...
}

// BAD - Vague test names
@Test
public test_add_task(): void {
  // What does it test?
}

@Test
public task_test(): void {
  // What is being tested?
}

@Test
public test1(): void {
  // What is this test for?
}
```

## Test Organization by Component

### Domain Tests

Test domain entities and value objects:

```typescript
describe('Task Entity', () => {
  describe('create', () => {
    @Test
    public create_task_with_valid_data_succeeds(): void {
      // Arrange
      const description = 'Test task description';

      // Act
      const task = Task.create(description);

      // Assert
      assertNotNull(task);
      assertEquals(description, task.getDescription());
      assertFalse(task.isDone());
    }

    @Test
    public create_task_with_empty_description_throws_exception(): void {
      // Arrange
      const description = '';

      // Act & Assert
      expect(() => Task.create(description))
        .toThrow('Task description cannot be empty');
    }

    @Test
    public create_task_generates_unique_id(): void {
      // Arrange
      const description = 'Test task';

      // Act
      const task1 = Task.create(description);
      const task2 = Task.create(description);

      // Assert
      assertNotEquals(task1.getId(), task2.getId());
    }
  });

  describe('setDone', () => {
    @Test
    public set_done_changes_status_to_completed(): void {
      // Arrange
      const task = Task.create('Test task');

      // Act
      task.setDone(true);

      // Assert
      assertTrue(task.isDone());
    }

    @Test
    public set_done_updates_completion_date(): void {
      // Arrange
      const task = Task.create('Test task');
      const before = new Date();

      // Act
      task.setDone(true);
      const after = new Date();

      // Assert
      const completionDate = task.getCompletionDate();
      assertNotNull(completionDate);
      assertTrue(completionDate >= before && completionDate <= after);
    }
  });
});
```

### Use Case Tests

Test use cases with business logic:

```typescript
describe('AddTaskUseCase', () => {
  let useCase: AddTaskService;
  let repository: InMemoryTaskRepository;

  beforeEach(() => {
    repository = new InMemoryTaskRepository();
    useCase = new AddTaskService(repository);
  });

  @Test
  public add_task_saves_to_repository(): async void {
    // Arrange
    const input: AddTaskInput = {
      description: 'Test task',
      projectId: null
    };

    // Act
    const output = await useCase.execute(input);

    // Assert
    assertTrue(output.success);
    assertNotNull(output.taskId);

    const saved = await repository.findById(output.taskId);
    assertNotNull(saved);
    assertEquals('Test task', saved.getDescription());
  }

  @Test
  public add_task_with_invalid_description_returns_error(): async void {
    // Arrange
    const input: AddTaskInput = {
      description: '',
      projectId: null
    };

    // Act
    const output = await useCase.execute(input);

    // Assert
    assertFalse(output.success);
    assertEquals('Description cannot be empty', output.message);
  }

  @Test
  public add_task_to_project_links_task_to_project(): async void => {
    // Arrange
    const project = Project.create('Test Project');
    await repository.saveProject(project);

    const input: AddTaskInput = {
      description: 'Test task',
      projectId: project.getId().value
    };

    // Act
    const output = await useCase.execute(input);

    // Assert
    assertTrue(output.success);
    const task = await repository.findById(output.taskId);
    assertEquals(project.getId(), task.getProjectId());
  }
});
```

### Controller Tests

Test controllers with input handling:

```typescript
describe('AddConsoleController', () => {
  let controller: AddConsoleController;
  let useCase: jest.Mocked<AddTaskService>;

  beforeEach(() => {
    useCase = createMockUseCase();
    controller = new AddConsoleController(useCase);
  });

  @Test
  public execute_with_valid_command_calls_use_case(): async void => {
    // Arrange
    useCase.execute.mockResolvedValue({
      taskId: 'task-123',
      success: true
    });

    // Act
    await controller.execute('add Test task');

    // Assert
    expect(useCase.execute).toHaveBeenCalledWith({
      description: 'Test task',
      projectId: null
    });
  }

  @Test
  public execute_with_invalid_command_shows_error(): async void => {
    // Arrange
    const consoleError = jest.spyOn(console, 'error').mockImplementation();

    // Act
    await controller.execute('add');

    // Assert
    expect(consoleError).toHaveBeenCalledWith(
      'Usage: add <description> [--project <project-id>]'
    );
  }

  @Test
  public execute_with_use_case_error_shows_message(): async void => {
    // Arrange
    useCase.execute.mockResolvedValue({
      success: false,
      message: 'Description cannot be empty'
    });
    const consoleError = jest.spyOn(console, 'error').mockImplementation();

    // Act
    await controller.execute('add ""');

    // Assert
    expect(consoleError).toHaveBeenCalledWith('Description cannot be empty');
  }
});
```

## Test Naming

### Behavior-Driven Naming

Tests describe behavior, not implementation:

```typescript
// GOOD - Behavior-driven
@Test
public when_user_logs_in_with_valid_credentials_then_token_is_returned(): void {
  // ...
}

@Test
public when_user_registers_with_duplicate_email_then_error_is_returned(): void {
  // ...
}

@Test
public when_product_quantity_is_negative_then_exception_is_thrown(): void {
  // ...
}

// BAD - Implementation-focused
@Test
public test_login_method(): void {
  // ...
}

@Test
public register_duplicate_email(): void {
  // ...
}

@Test
public check_quantity(): void {
  // ...
}
```

### Given-When-Then Format

Use Given-When-Then for complex scenarios:

```typescript
describe('Shopping Cart', () => {
  @Test
  public given_empty_cart_when_item_added_then_cart_contains_one_item(): void {
    // Given
    const cart = new ShoppingCart();

    // When
    cart.addItem(new CartItem('product-1', 'Product 1', 2, 10.00));

    // Then
    assertEquals(1, cart.getItemCount());
  }

  @Test
  public given_cart_with_items_when_cleared_then_cart_is_empty(): void {
    // Given
    const cart = new ShoppingCart();
    cart.addItem(new CartItem('product-1', 'Product 1', 2, 10.00));
    cart.addItem(new CartItem('product-2', 'Product 2', 1, 5.00));

    // When
    cart.clear();

    // Then
    assertEquals(0, cart.getItemCount());
  }
});
```

## Test Isolation

### Independent Tests

Each test should be independent of others:

```typescript
describe('Task Repository', () => {
  let repository: InMemoryTaskRepository;

  beforeEach(() => {
    // Create fresh repository for each test
    repository = new InMemoryTaskRepository();
  });

  @Test
  public first_test_creates_task(): void {
    const task = Task.create('Task 1');
    repository.save(task);
    // ...
  }

  @Test
  public second_test_creates_task(): void {
    // This test should not be affected by first_test
    const task = Task.create('Task 2');
    repository.save(task);
    assertEquals(1, repository.findAll().length);
  }
});
```

### Cleanup

Clean up after tests:

```typescript
describe('File Storage', () => {
  let storage: FileStorage;

  beforeEach(() => {
    storage = new FileStorage('/tmp/test-storage');
  });

  afterEach(async () => {
    // Clean up test files
    await storage.deleteAll();
  });

  @Test
  public save_and_retrieve_file(): void {
    // ...
  }
});
```

## Test Doubles

### Mocks

Simulate dependencies with predefined behavior:

```typescript
describe('TaskService', () => {
  let service: TaskService;
  let repository: jest.Mocked<ITaskRepository>;

  beforeEach(() => {
    repository = {
      findById: jest.fn(),
      save: jest.fn(),
      findAll: jest.fn()
    } as jest.Mocked<ITaskRepository>;

    service = new TaskService(repository);
  });

  @Test
  public find_existing_task_returns_task(): void {
    // Arrange
    const task = Task.create('Test task');
    repository.findById.mockResolvedValue(task);

    // Act
    const result = await service.findById('task-123');

    // Assert
    assertEquals(task, result);
    expect(repository.findById).toHaveBeenCalledWith('task-123');
  }
});
```

### Stubs

Provide fixed responses for specific calls:

```typescript
describe('Email Service', () => {
  let service: UserService;
  let emailService: IEmailService;

  beforeEach(() => {
    emailService = {
      sendEmail: jest.fn().mockResolvedValue(undefined)
    } as jest.Mocked<IEmailService>;

    service = new UserService(/* ... */, emailService);
  });

  @Test
  public register_user_sends_welcome_email(): async void => {
    // Arrange
    emailService.sendEmail.mockResolvedValue(undefined);

    // Act
    await service.registerUser('test@example.com', 'password');

    // Assert
    expect(emailService.sendEmail).toHaveBeenCalledWith(
      'test@example.com',
      'Welcome',
      expect.any(String)
    );
  }
});
```

### Fakes

Simple implementations for testing:

```typescript
class InMemoryTaskRepository implements ITaskRepository {
  private tasks: Map<string, Task> = new Map();

  async findById(id: string): Promise<Task | null> {
    return this.tasks.get(id) || null;
  }

  async save(task: Task): Promise<void> {
    this.tasks.set(task.getId().value, task);
  }

  async findAll(): Promise<Task[]> {
    return Array.from(this.tasks.values());
  }
}

describe('Task Service', () => {
  @Test
  public create_and_retrieve_task(): void {
    // Arrange
    const repository = new InMemoryTaskRepository();
    const service = new TaskService(repository);

    // Act
    const task = await service.createTask('Test task');

    // Assert
    const found = await repository.findById(task.getId().value);
    assertNotNull(found);
  }
});
```

## Testing Strategies

### Boundary Testing

Test at boundaries:

```typescript
describe('Password Validator', () => {
  @Test
  public password_with_7_characters_is_too_short(): void {
    assertFalse(PasswordValidator.isValid('Abc123!'));
  }

  @Test
  public password_with_8_characters_is_valid(): void {
    assertTrue(PasswordValidator.isValid('Abc12345!'));
  }

  @Test
  public password_with_127_characters_is_valid(): void {
    const password = 'A' + 'a'.repeat(125) + '1!';
    assertTrue(PasswordValidator.isValid(password));
  }

  @Test
  public password_with_128_characters_is_too_long(): void {
    const password = 'A' + 'a'.repeat(126) + '1!';
    assertFalse(PasswordValidator.isValid(password));
  }
});
```

### Edge Case Testing

Test edge cases and error conditions:

```typescript
describe('Calculator', () => {
  @Test
  public divide_by_zero_throws_exception(): void {
    expect(() => Calculator.divide(10, 0))
      .toThrow('Cannot divide by zero');
  }

  @Test
  public divide_negative_by_positive_returns_negative(): void {
    assertEquals(-2, Calculator.divide(10, -5));
  }

  @Test
  public divide_zero_by_any_number_returns_zero(): void {
    assertEquals(0, Calculator.divide(0, 10));
  }

  @Test
  public divide_positive_by_positive_returns_positive(): void {
    assertEquals(2, Calculator.divide(10, 5));
  }
});
```

### Integration Testing

Test component interactions:

```typescript
describe('Task Management Integration', () => {
  let useCase: AddTaskService;
  let repository: InMemoryTaskRepository;

  beforeEach(() => {
    repository = new InMemoryTaskRepository();
    useCase = new AddTaskService(repository);
  });

  @Test
  public add_and_retrieve_task_integration(): void {
    // Act - Add task
    const output1 = await useCase.execute({
      description: 'Test task',
      projectId: null
    });

    // Assert - Task was added
    assertTrue(output1.success);
    const task = await repository.findById(output1.taskId);
    assertNotNull(task);

    // Act - Retrieve task
    const output2 = await useCase.execute({
      taskId: output1.taskId
    });

    // Assert - Task retrieved correctly
    assertEquals('Test task', task.getDescription());
  }
});
```

## Test Coverage Targets

### Coverage Requirements

- **Domain logic**: 100% coverage
- **Use cases**: 95%+ coverage
- **Adapters**: 80%+ coverage
- **Overall**: 85%+ coverage

### Coverage Reporting

Run coverage reports:

```bash
# Run tests with coverage
npm run test:coverage

# Generate HTML report
npm run test:coverage:html

# Check coverage thresholds
npm run test:coverage:check
```

## Testing Best Practices

### AAA Pattern

Always use Arrange-Act-Assert:

```typescript
@Test
public user_login_test(): void {
  // Arrange
  const user = new User('test@example.com', 'password');
  const service = new AuthService(repository);

  // Act
  const result = service.authenticate(user);

  // Assert
  assertTrue(result.success);
}
```

### One Assertion Per Test

Keep tests focused:

```typescript
// GOOD - Single assertion
@Test
public task_id_is_unique(): void {
  const task1 = Task.create('Task 1');
  const task2 = Task.create('Task 2');
  assertNotEquals(task1.getId(), task2.getId());
}

// BAD - Multiple assertions
@Test
public task_properties(): void {
  const task = Task.create('Task 1');
  assertNotNull(task.getId()); // First assertion
  assertEquals('Task 1', task.getDescription()); // Second assertion
  assertFalse(task.isDone()); // Third assertion
}
```

### Test Behavior, Not Implementation

Test what happens, not how:

```typescript
// GOOD - Tests behavior
@Test
public add_task_increases_task_count(): void {
  const cart = new ShoppingCart();
  cart.addItem(new CartItem('1', 'Product', 1, 10.00));
  assertEquals(1, cart.getItemCount());
}

// BAD - Tests implementation
@Test
public add_task_pushes_to_items_array(): void {
  const cart = new ShoppingCart();
  cart.addItem(new CartItem('1', 'Product', 1, 10.00));
  assertEquals(1, cart.getItems().length); // Exposes internal structure
}
```

## References

- [docs/architecture/ARCHITECTURE.md](../architecture/ARCHITECTURE.md) - Architecture overview
- [docs/engineering/METHODOLOGIES.md](../METHODOLOGIES.md) - All methodologies
- [docs/engineering/conventions/SOLID_PRINCIPLES.md](SOLID_PRINCIPLES.md) - SOLID principles
- [docs/engineering/conventions/NAMING_CONVENTIONS.md](NAMING_CONVENTIONS.md) - Naming conventions
- [CODING_AGENTS.md](../../CODING_AGENTS.md) - AI agent development guidelines
