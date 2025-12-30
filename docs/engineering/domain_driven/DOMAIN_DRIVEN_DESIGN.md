# Domain-Driven Design (DDD)

## Overview

Domain-Driven Design (DDD) is an approach to software development that focuses on solving complex business problems by connecting the implementation to an evolving model of the core business domain. Nexus uses DDD as a foundational methodology for building maintainable, business-centric software.

## Core Concepts

### Bounded Contexts

A bounded context is a distinct part of the domain logic where terms and rules have specific meaning. It provides clear boundaries within which a particular domain model applies.

**Key Principles**:
- Each bounded context has its own ubiquitous language
- Contexts communicate through well-defined interfaces
- No sharing of domain logic between contexts
- Explicit boundaries prevent conceptual confusion

**Example**:
```typescript
// Sales Context - "Product" has pricing, inventory, catalog
class SalesProduct {
  private id: ProductId;
  private price: Money;
  private inventory: InventoryLevel;
  private catalog: CatalogEntry;
}

// Shipping Context - "Product" has weight, dimensions, shipping class
class ShippingProduct {
  private id: ProductId;
  private weight: Weight;
  private dimensions: Dimensions;
  private shippingClass: ShippingClass;
}

// These are DIFFERENT entities in DIFFERENT contexts
// They communicate only through ProductId (value object)
```

### Ubiquitous Language

A shared, rigorous language used by both developers and domain experts. It evolves through continuous collaboration and must be used consistently in code, documentation, and conversations.

**Key Principles**:
- Same terms in code and domain conversations
- No translation between technical and business language
- Evolves as understanding deepens
- Validated through conversation with domain experts

**Example**:
```typescript
// GOOD - Uses ubiquitous language
class ShoppingCart {
  addItem(product: Product): void { }
  removeItem(itemId: string): void { }
  calculateTotal(): Money { }
  checkout(): Order { }
}

// BAD - Uses technical terms
class ShoppingCart {
  pushProduct(productData: ProductData): void { }
  deleteItemByIndex(index: number): void { }
  computeSum(): number { }
  persistTransaction(): OrderRecord { }
}
```

### Entities

Objects with a distinct identity that runs through time and different states. Entities are defined by their identity, not their attributes.

**Key Principles**:
- Identity is unique and constant
- Equality based on identity, not attributes
- Can undergo significant state changes
- Lifecycle managed by aggregate root

**Example**:
```typescript
class User implements Entity<UserId> {
  constructor(
    private readonly id: UserId,
    private email: Email,
    private name: string,
    private status: UserStatus
  ) {}

  equals(other: User): boolean {
    return this.id.equals(other.id);
  }

  changeName(name: string): void {
    if (!name || name.trim().length === 0) {
      throw new Error('Name cannot be empty');
    }
    this.name = name;
  }

  activate(): void {
    if (this.status === UserStatus.ACTIVE) {
      throw new Error('User already active');
    }
    this.status = UserStatus.ACTIVE;
  }
}

// Equality based on ID, not attributes
const user1 = new User(userId1, email, 'John', UserStatus.INACTIVE);
const user2 = new User(userId1, email, 'John', UserStatus.INACTIVE);
const user3 = new User(userId2, email, 'John', UserStatus.INACTIVE);

user1.equals(user2); // true - same ID
user1.equals(user3); // false - different ID
```

### Value Objects

Objects defined by their attributes, not identity. They are immutable and represent concepts in the domain.

**Key Principles**:
- Immutable (use `readonly`)
- Equality based on attribute values
- Self-validating
- Replace primitive types with domain concepts

**Example**:
```typescript
class Email {
  private constructor(public readonly value: string) {
    if (!this.isValid(value)) {
      throw new Error('Invalid email format');
    }
  }

  static of(value: string): Email {
    return new Email(value);
  }

  private isValid(email: string): boolean {
    const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return regex.test(email);
  }

  equals(other: Email): boolean {
    return this.value === other.value;
  }
}

// Usage
const email1 = Email.of('user@example.com');
const email2 = Email.of('user@example.com');
const email3 = Email.of('another@example.com');

email1.equals(email2); // true - same values
email1.equals(email3); // false - different values
```

### Aggregates

Clusters of domain objects that can be treated as a unit. The aggregate root is the only member accessible from outside the aggregate.

**Key Principles**:
- Aggregate root controls all access
- Invariants maintained within aggregate
- External references only to aggregate root
- Transactions preserve aggregate consistency

**Example**:
```typescript
class Order implements Entity<OrderId> {
  constructor(
    private readonly id: OrderId,
    private readonly customerId: CustomerId,
    private items: OrderItem[],
    private status: OrderStatus
  ) {}

  addItem(product: Product, quantity: number): void {
    const item = OrderItem.create(product, quantity);
    this.items.push(item);
    this.recalculateTotal();
  }

  removeItem(itemId: string): void {
    this.items = this.items.filter(item => item.id !== itemId);
    this.recalculateTotal();
  }

  getTotal(): Money {
    return this.items.reduce((total, item) =>
      total.add(item.getTotal()),
      Money.zero()
    );
  }

  private recalculateTotal(): void {
    // Invariant: Total always reflects item prices
    // Maintained within aggregate
  }
}

// OrderItem is NOT accessible from outside Order
// All operations go through Order (aggregate root)
```

### Domain Services

Stateless services that perform domain operations that don't naturally fit within an entity or value object.

**Key Principles**:
- Stateless operations
- Business logic that involves multiple aggregates
- Operations that don't belong to any entity
- Pure functions preferred

**Example**:
```typescript
class OrderCalculationService {
  calculateDiscount(order: Order, coupon: Coupon): Money {
    if (!this.isCouponValid(coupon)) {
      throw new Error('Invalid coupon');
    }

    const total = order.getTotal();
    return total.multiply(coupon.discountPercentage);
  }

  calculateTax(order: Order, taxRate: Percentage): Money {
    const total = order.getTotal();
    return total.multiply(taxRate.value);
  }

  private isCouponValid(coupon: Coupon): boolean {
    return !coupon.expired && coupon.active;
  }
}
```

## Domain Events

### Domain Event Storming

A collaborative workshop to discover domain events and their relationships.

**Process**:
1. Gather domain experts and developers
2. Identify domain events (past tense, domain-relevant)
3. Map event sequences in time order
4. Identify aggregates (entities that generate events)
5. Discover relationships between aggregates
6. Document event payload and metadata

**Example**:
```typescript
// Domain Events (immutable, past tense)
class OrderCreated {
  constructor(
    public readonly orderId: OrderId,
    public readonly customerId: CustomerId,
    public readonly items: OrderItem[],
    public readonly createdAt: Date
  ) {}
}

class OrderItemAdded {
  constructor(
    public readonly orderId: OrderId,
    public readonly productId: ProductId,
    public readonly quantity: number,
    public readonly createdAt: Date
  ) {}
}

class OrderPaid {
  constructor(
    public readonly orderId: OrderId,
    public readonly paymentMethod: PaymentMethod,
    public readonly amount: Money,
    public readonly paidAt: Date
  ) {}
}

class OrderShipped {
  constructor(
    public readonly orderId: OrderId,
    public readonly shippingAddress: Address,
    public readonly trackingNumber: string,
    public readonly shippedAt: Date
  ) {}
}
```

### Domain Event Storytelling

A narrative approach to understanding domain event flows and business processes.

**Process**:
1. Create narrative timeline of events
2. Identify triggers and outcomes
3. Map event dependencies
4. Validate understanding with domain experts

**Example**:
```typescript
// Narrative: Order Processing Flow
//
// 1. Customer places order
//    → OrderCreated event
//
// 2. Order system processes order
//    → InventoryReserved event
//    → PaymentAuthorized event
//
// 3. Payment captured
//    → OrderPaid event
//
// 4. Order shipped
//    → OrderShipped event
//
// 5. Customer receives order
//    → OrderDelivered event
//
// 6. Order completed
//    → OrderCompleted event

class Order {
  place(customer: Customer, items: OrderItem[]): void {
    const order = new Order(
      OrderId.generate(),
      customer.id,
      items
    );

    this.events.push(
      new OrderCreated(order.id, customer.id, items, new Date())
    );
  }

  pay(payment: Payment): void {
    this.payment = payment;
    this.status = OrderStatus.PAID;

    this.events.push(
      new OrderPaid(this.id, payment.method, payment.amount, new Date())
    );
  }

  ship(shippingAddress: Address): void {
    this.shippingAddress = shippingAddress;
    this.trackingNumber = TrackingNumber.generate();
    this.status = OrderStatus.SHIPPED;

    this.events.push(
      new OrderShipped(this.id, shippingAddress, this.trackingNumber, new Date())
    );
  }
}
```

## DDD in Nexus

### Domain Model Creation

The Domain Architect Agent creates domain models using DDD principles:

```typescript
// Example: Blog with Posts and Comments

// Bounded Context: Blog
// Ubiquitous Language: Post, Comment, Category, Tag, Author

// Entities
class Post implements Entity<PostId> {
  constructor(
    private readonly id: PostId,
    private title: PostTitle,
    private content: PostContent,
    private author: Author,
    private status: PostStatus,
    private comments: Comment[],
    private tags: Tag[]
  ) {}

  publish(): void {
    if (this.status === PostStatus.PUBLISHED) {
      throw new Error('Post already published');
    }

    this.status = PostStatus.PUBLISHED;
    this.publishedAt = new Date();
  }

  addComment(comment: Comment): void {
    this.comments.push(comment);
  }
}

// Value Objects
class PostTitle {
  private constructor(public readonly value: string) {
    if (value.length < 5 || value.length > 200) {
      throw new Error('Title must be 5-200 characters');
    }
  }

  static of(value: string): PostTitle {
    return new PostTitle(value);
  }
}

// Domain Events
class PostPublished {
  constructor(
    public readonly postId: PostId,
    public readonly authorId: AuthorId,
    public readonly publishedAt: Date
  ) {}
}

class CommentAdded {
  constructor(
    public readonly postId: PostId,
    public readonly commentId: CommentId,
    public readonly authorId: AuthorId,
    public readonly addedAt: Date
  ) {}
}
```

## Best Practices

### ✅ DO

- Focus on core business domain
- Use ubiquitous language consistently
- Create bounded contexts with clear boundaries
- Make aggregates self-contained
- Use domain events for consistency
- Collaborate with domain experts

### ❌ DON'T

- Mix technical concerns with domain logic
- Share domain logic between bounded contexts
- Expose internal aggregate state
- Use anemic domain models (setters/getters only)
- Ignore invariant enforcement
- Create generic domain models

## References

- [docs/architecture/ARCHITECTURE.md](../architecture/ARCHITECTURE.md) - System architecture
- [docs/engineering/METHODOLOGIES.md](../METHODOLOGIES.md) - All methodologies
- [docs/engineering/patterns/PATTERNS.md](../patterns/PATTERNS.md) - Design patterns
- [CODING_AGENTS.md](../../CODING_AGENTS.md) - AI agent development guidelines
