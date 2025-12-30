# Nexus Development Methodologies

## Overview

Nexus implements a comprehensive set of proven software engineering methodologies to build robust, maintainable applications through AI-assisted development.

## Methodologies Implemented

### **Specification Driven Development**

Every feature starts with a specification that drives implementation, not the reverse.

**Key Principles**:
- Specifications drive implementation, not reverse
- Executable specifications serve as living documentation
- Tests ARE the documentation

**Benefits**:
- Prevents implementation drift from requirements
- Always-current documentation that actually works
- Clear expectations before coding

---

### **Test-Driven Development (TDD)**

Tests guide implementation through three phases:

**RED Phase**: Write failing test
**GREEN Phase**: Write minimal code to pass test
**REFACTOR Phase**: Improve code while tests pass

**Key Principles**:
- Tests written first, code written second
- Each test isolated and focused
- Tests enforce requirements remain satisfied

**Benefits**:
- Comprehensive test coverage
- Living documentation as executable specs
- Confidence in refactoring (tests guard against regressions)

---

### **Domain-Driven Design (DDD)**

Business logic at the center, not database tables or frameworks.

**Key Concepts**:
- **Bounded Contexts**: Clear boundaries between business domains
- **Ubiquitous Language**: Shared business language with stakeholders
- **Aggregates**: Consistency boundaries with business rules
- **Entities**: Business objects with identity
- **Value Objects**: Immutable objects identified by attributes
- **Domain Events**: State changes in domain
- **Domain Services**: Complex business operations

**Benefits**:
- Business logic isolated from infrastructure
- Common language with business experts
- Easier to understand business rules

---

### **Clean Architecture**

Four-layer separation with strict dependency inversion.

**Layers**:
1. **Domain Layer**: Core business logic (no dependencies)
2. **Use Cases Layer**: Application logic (uses domain)
3. **Adapters Layer**: External interfaces (controllers, presenters, gateways)
4. **Frameworks Layer**: Concrete implementations (Express, Database)

**Dependency Rule**: Dependencies point **inward only** (from outer to inner)

**Benefits**:
- Easy to test (mock interfaces)
- Technology-agnostic domain logic
- Easy to swap implementations
- Parallel development possible

---

### **Vertical Slice Architecture (Feature-Based)**

Code organized by business features, not technical layers.

**Structure**:
```
src/features/
├── user-management/
│   ├── domain/              # Domain Layer
│   ├── use-cases/           # Application Layer
│   ├── adapters/            # Adapter Layer
│   └── tests/               # Tests co-located
└── authentication/
```

**Benefits**:
- All related code in one place
- Easy to understand feature context
- Easier to add/remove features
- Clear feature boundaries

---

### **Behavior Driven Development (BDD)**

Features defined by behavior, not implementation details.

**Given-When-Then Format**:
- **Given**: Setup test data
- **When**: Execute code being tested
- **Then**: Verify expected outcome

**Benefits**:
- Focus on user value and business outcomes
- Tests serve as acceptance criteria
- Shared language with stakeholders

---

### **Problem Frames Approach** (Michael A. Jackson)

Understanding problems in their real-world context before designing software.

**Key Principles**:
- **Solution is in computer**: Software provides solutions
- **Problem is in the world**: Problems exist in real world (people, organizations)
- **Study the problem world**: Analyze problem domains deeply
- **Parallel decomposition**: Decompose requirements in parallel
- **Context diagram**: Show problem domains and relationships
- **Problem diagram**: Show requirements and phenomena references

**Key Concepts**:
- **Problem Domains**: Different parts of real world (Users, Products, Orders)
- **Phenomena**: Individuals, events, states in domains
- ✓︁ Example: "User clicks buy button" is event in both Users and Cart domains
- **Machine Interface**: Where software system meets problem domains
- **Machine**: The software system being built

**Recognized Problem Frames**:
1. **Required Behavior**: Control part of world to satisfy conditions
2. **Commanded Behavior**: Accept commands from operator and control accordingly
3. **Information Display**: Obtain information from world and present it in required form
4. **Simple Workpieces**: Allow users to create and edit text/graphic objects
5. **Transformation**: Transform input data to produce required output

**Benefits**:
- Clear understanding of problem before coding
- Prevents solving wrong problems
- Better alignment with business needs
- Easier communication with stakeholders

---

### **Design by Contract**

Interfaces specify precise contracts for interactions.

**Key Principles**:
- **Preconditions**: What must be true before calling
- **Postconditions**: What will be true after calling
- **Invariants**: What remains unchanged

**Example**:
```typescript
interface IUserRepository {
  /**
   * @precondition userId must be a valid UUID
   * @postcondition Returns User entity or throws UserNotFoundError
   * @invariant User id, email remain unchanged
   */
  findById(userId: string): Promise<User>;

  /**
   * @precondition email must be unique
   * @postcondition Returns new User with generated id
   * @invariant User email is validated
   */
  create(user: CreateUserDTO): Promise<User>;
}
```

**Benefits**:
- Clear expectations between modules
- Easier testing (contracts define expected behavior)
- Prevents integration bugs
- Self-documenting interfaces

---

### **Living Documentation**

Documentation that evolves with the codebase, always stays current.

**Key Principles**:
- Tests ARE documentation (executable specifications)
- README files reference specific test suites
- API docs generated from JSDoc + tests
- No outdated documentation possible

**Benefits**:
- Documentation always matches code
- Tests provide examples of usage
- Easy to verify documentation accuracy

---

### **Specification by Example**

Use concrete examples to clarify behavior.

**Benefits**:
- Edge cases demonstrated with test cases
- Valid and invalid inputs shown
- Expected outputs clearly defined
- Clear expectations for developers

---

### **Example Mapping**

Map requirements to executable test examples.

**Benefits**:
- Requirements validated through tests
- Clear examples of expected behavior
- Easy to add test cases for edge cases
- Test cases document business rules

---

### **Domain Event Storming**

Collaborative session to discover domain events and aggregates.

**Process**:
1. Gather domain experts and developers
2. Identify domain events (past tense, domain-relevant)
3. Map event sequences in time order
4. Identify aggregates (entities that generate events)
5. Discover relationships between aggregates
6. Document event payload and metadata

**Output**:
- Domain events catalog
- Aggregate boundaries identified
- Event flow narratives

---

### **Domain Event Storytelling**

Narrative approach to understanding domain event flows.

**Process**:
1. Create narrative from event sequence
2. Describe business process in plain language
3. Identify actors and their motivations
4. Document side effects and outcomes
5. Create event flow diagrams

**Example**:
```
"User creates post → PostCreated event → User publishes post → PostPublished event →
 Reader adds comment → CommentAdded event →
Moderator approves comment → CommentModerated event"
```

---

## Optional Advanced Patterns

### **Event Sourcing** (Optional)

Store state changes as sequence of events instead of current state.

**Benefits**:
- Complete audit trail
- Temporal queries (state at any point in time)
- Event replay for debugging
- Natural integration with event-driven architectures

**When to Use**:
- Complex domains requiring audit trails
- Need to reconstruct state at any point in time
- Event-driven architectures
- Systems with complex business rules

**Implementation**:
```typescript
// Traditional CRUD
UPDATE users SET email = 'new@example.com' WHERE id = '123'

// Event Sourcing
INSERT INTO events (aggregate_id, event_type, payload, created_at)
VALUES ('123', 'EmailUpdated', '{"email": "new@example.com"}', NOW())
```

---

### **CQRS** (Optional)

Command Query Responsibility Segregation: Separate read and write operations.

**Benefits**:
- Independent scaling of reads and writes
- Optimized read models for queries
- Simplified write model for consistency
- Better separation of concerns

**When to Use**:
- High-scale systems with many reads vs writes
- Complex read requirements with joins/aggregations
- Need different data models for reading vs writing
- Microservices with eventual consistency

**Implementation**:
```typescript
// Write Model (Commands)
class UserCommandHandler {
  async execute(command: CreateUserCommand): Promise<void> {
    const user = new User(command.email, command.password);
    await this.writeRepository.save(user);
  }
}

// Read Model (Queries)
class UserQueryHandler {
  async execute(query: GetUserQuery): Promise<UserDTO> {
    return await this.readRepository.findById(query.userId);
  }
}
```

---

## How Methodologies Work Together

### **Example Feature: Shopping Cart**

```
1. Problem Frames Analysis (Michael A. Jackson)
   Problem Domains: Users, Products, Cart, Orders
   Shared Phenomena: "User", "Product" exist across domains
   Machine: CartService API

2. Domain-Driven Design
   Bounded Context: Shopping Context
   Entities: Cart, CartItem, Product, Order
   Value Objects: CartId, Quantity, Money
   Aggregates: Cart (aggregate root), Order (aggregate root)
   Domain Events: CartCreated, ItemAdded, OrderPlaced

3. Clean Architecture
   - Domain Layer: Cart, CartItem entities with business logic
   - Use Cases Layer: AddItem, RemoveItem, Checkout use cases
   - Adapters Layer: CartController (REST), CartPresenter (JSON)
   - Ports: ICartRepository (output), ICartPresenter (output)

4. Specification Driven Development
   - Tests written first (TDD)
   - Tests define executable specifications
   - Tests serve as living documentation

5. Test-Driven Development
   - RED: Write failing test for addItem
   - GREEN: Implement minimal code to pass
   - REFACTOR: Improve while tests pass

6. Behavior Driven Development
   - Given: User has empty cart
   - When: User adds product with quantity 2
   - Then: Cart contains product with correct total

7. Clean Code with Patterns
   - Entity Pattern: Cart with identity and lifecycle
   - Value Object Pattern: Quantity (readonly, validated)
   - Aggregate Root Pattern: Cart manages CartItems
   - Repository Pattern: CartRepository abstracts data access
   - Factory Pattern: CartFactory.create(dto)
   - Mapper Pattern: CartMapper.toDto(cart)
   - DTO Pattern: CartDto for API transfer
   - Read-Only Interface: ReadOnlyCart for read operations
   - Strategy Pattern: Different calculation strategies
   - Command Pattern: AddItemCommand, RemoveItemCommand

8. Living Documentation
   - Tests ARE documentation
   - README references specific test suites
   - API docs generated from JSDoc + tests
   - No outdated documentation possible

9. Design by Contract
   - Precondition: Cart must exist before adding items
   - Postcondition: Total price recalculated after each operation
   - Invariant: Cart ID never changes

10. Optional: Event Sourcing
    - Events: CartCreated, ItemAdded, ItemRemoved, OrderPlaced
    - Benefits: Complete audit trail, temporal queries

11. Optional: CQRS
    - Commands: AddItemCommand, RemoveItemCommand, CheckoutCommand
    - Queries: GetCartQuery, GetTotalQuery
    - Benefits: Independent scaling of reads and writes
```

## Key Benefits

### **Robustness**
- Multiple methodologies provide defense in depth
- Problems properly understood before coding
- Business rules enforced throughout
- Architecture protects against technical debt
- Tests guarantee quality and correctness

### **Maintainability**
- Clear separation of concerns
- Easy to test each layer independently
- Features isolated and self-contained
- Documentation matches code always

### **Developer Experience**
- Small reviewable steps (<5 minutes each)
- Human-in-the-loop development
- Clear rollback paths for each step
- Rich CLI output for progress tracking

### **Quality Assurance**
- Strict code rules enforced automatically
- Comprehensive test coverage (85%+)
- All methodologies verified automatically
- Living documentation guaranteed

### **Flexibility**
- Swap implementations easily (repositories, databases)
- Add new features without breaking existing code
- Support multiple AI providers
- Frontend framework choices
- Database support for multiple environments

## See Also

- [Architecture Details](./architecture/ARCHITECTURE.md)
- [Folder Structure](./architecture/FOLDER_STRUCTURE.md)
- [Design Patterns](./engineering/patterns/PATTERNS.md)
- [Naming Conventions](./engineering/conventions/SOLID_PRINCIPLES.md)
- [Testing Conventions](./engineering/conventions/TESTING_CONVENTIONS.md)
- [Development Guidelines](./engineering/conventions/DEVELOPMENT_GUIDELINES.md)
- [Security Guidelines](./engineering/conventions/SECURITY_GUIDELINES.md)
- [Code Quality Standards](./engineering/quality/CODE_QUALITY_STANDARDS.md)
- [Problem Frames](./engineering/problem_frames/PROBLEM_FRAMES.md)
- [Specification Driven](./engineering/specification/SPECIFICATION_DRIVEN.md)
- [Domain Driven Design](./engineering/domain_driven/DOMAIN_DRIVEN.md)
- [Test Driven Development](./engineering/test_driven/TEST_DRIVEN.md)
- [Behavior Driven Development](./engineering/behavior_driven/BEHAVIOR_DRIVEN.md)
- [Living Documentation](./engineering/living_documentation/LIVING_DOCUMENTATION.md)
- [Event Sourcing](./engineering/event_sourcing/EVENT_SOURCING.md)
- [CQRS](./engineering/cqrs/CQRS.md)