# Design by Contract

## Overview

Design by Contract (DbC) is a software design methodology where software components specify **preconditions**, **postconditions**, and **invariants** for their operations. Nexus uses DbC to ensure components behave predictably and correctly.

## Core Concepts

### Preconditions

Conditions that must be true **before** an operation is called.

**Responsibility**: Caller must ensure preconditions hold

**Example**:
```typescript
interface IPostRepository {
  /**
   * @precondition postId must be a valid, non-empty string
   * @precondition postId must not be null or undefined
   */
  findById(postId: string): Promise<Post | null>;
}
```

### Postconditions

Conditions that must be true **after** an operation completes successfully.

**Responsibility**: Implementation must ensure postconditions hold

**Example**:
```typescript
interface IPostRepository {
  /**
   * @postcondition Returns Post entity if found
   * @postcondition Returns null if Post not found
   * @postcondition Returned Post contains complete data
   */
  findById(postId: string): Promise<Post | null>;
}
```

### Invariants

Conditions that must **always** be true for a component.

**Responsibility**: Implementation maintains invariants

**Example**:
```typescript
class Post implements Entity<PostId> {
  /**
   * @invariant Post id cannot be null
   * @invariant Post id cannot change
   * @invariant Post status is always valid
   */
  constructor(
    private readonly id: PostId,
    private title: PostTitle,
    private status: PostStatus
  ) {}

  // All methods maintain these invariants
}
```

## Contract Implementation

### 1. Interface with Contracts

Define contracts in interface:

```typescript
interface IPostRepository {
  /**
   * @precondition postId must be a valid PostId value
   * @precondition postId cannot be null or undefined
   *
   * @postcondition Returns Post entity if found
   * @postcondition Returns null if Post not found
   * @postcondition Post contains all fields if found
   *
   * @invariant Repository state unchanged (read-only operation)
   */
  findById(postId: PostId): Promise<Post | null>;

  /**
   * @precondition post must not be null
   * @precondition post must satisfy business invariants
   * @precondition post.id must be valid PostId
   *
   * @postcondition Post is persisted
   * @postcondition Post can be retrieved by findById
   * @postcondition Post.id is generated if not set
   *
   * @invariant Repository contains valid Post entities
   */
  save(post: Post): Promise<void>;

  /**
   * @precondition postId must be a valid PostId
   * @precondition Post with given ID must exist
   *
   * @postcondition Post is removed from repository
   * @postcondition findById returns null for removed Post
   *
   * @invariant No orphaned data remains
   */
  delete(postId: PostId): Promise<void>;
}
```

### 2. Implementation that Enforces Contracts

Check and enforce contracts in implementation:

```typescript
class SqlPostRepository implements IPostRepository {
  constructor(private readonly database: DatabaseConnection) {}

  async findById(postId: PostId): Promise<Post | null> {
    // Enforce preconditions
    if (!postId) {
      throw new Error('Precondition failed: postId cannot be null');
    }

    // Implementation
    const result = await this.database.query(
      'SELECT * FROM posts WHERE id = ?',
      [postId.value]
    );

    // Ensure postconditions
    if (result) {
      const post = PostMapper.toDomain(result);

      // Verify postcondition: Post contains all fields
      if (!post.id || !post.title || !post.content || !post.status) {
        throw new Error('Postcondition failed: Post missing fields');
      }

      return post;
    }

    // Postcondition: Returns null if not found
    return null;
  }

  async save(post: Post): Promise<void> {
    // Enforce preconditions
    if (!post) {
      throw new Error('Precondition failed: post cannot be null');
    }

    // Check business invariants
    if (!post.title.value || post.title.value.trim().length === 0) {
      throw new Error('Invariant failed: Post must have valid title');
    }

    // Implementation
    const po = PostMapper.toPo(post);

    if (!po.id) {
      // Generate new ID if not set
      po.id = PostId.generate().value;
    }

    await this.database.query(
      'INSERT INTO posts (id, title, content, status) VALUES (?, ?, ?, ?)',
      [po.id, po.title.value, po.content.value, po.status]
    );

    // Ensure postcondition: Post is persisted
    const saved = await this.findById(new PostId(po.id));
    if (!saved) {
      throw new Error('Postcondition failed: Post not persisted');
    }
  }

  async delete(postId: PostId): Promise<void> {
    // Enforce preconditions
    if (!postId) {
      throw new Error('Precondition failed: postId cannot be null');
    }

    // Check precondition: Post must exist
    const existing = await this.findById(postId);
    if (!existing) {
      throw new Error('Precondition failed: Post does not exist');
    }

    // Implementation
    await this.database.query(
      'DELETE FROM posts WHERE id = ?',
      [postId.value]
    );

    // Ensure postcondition: Post is removed
    const deleted = await this.findById(postId);
    if (deleted) {
      throw new Error('Postcondition failed: Post still exists');
    }
  }
}
```

### 3. Caller Checks Preconditions

Caller ensures preconditions before calling:

```typescript
class PublishPostService implements PublishPostUseCase {
  constructor(
    private readonly postRepository: IPostRepository
  ) {}

  async execute(input: PublishPostInput): Promise<PublishPostOutput> {
    // Check precondition: postId must be valid
    if (!input.postId || input.postId.trim().length === 0) {
      return PublishPostOutput.failure('postId is required');
    }

    const postId = new PostId(input.postId);

    // Call repository (which checks its own preconditions)
    const post = await this.postRepository.findById(postId);

    if (!post) {
      return PublishPostOutput.failure('Post not found');
    }

    post.publish();

    // Save (repository checks preconditions and ensures postconditions)
    await this.postRepository.save(post);

    return PublishPostOutput.success(post.id);
  }
}
```

## Contract Utilities

### Precondition Checking

```typescript
export function require(
  condition: boolean,
  message: string
): void {
  if (!condition) {
    throw new Error(`Precondition failed: ${message}`);
  }
}

// Usage
class User {
  setEmail(email: Email): void {
    require(email !== null, 'Email cannot be null');
    this.email = email;
  }
}
```

### Postcondition Validation

```typescript
export function ensure(
  condition: boolean,
  message: string
): void {
  if (!condition) {
    throw new Error(`Postcondition failed: ${message}`);
  }
}

// Usage
class UserRepository {
  async save(user: User): Promise<void> {
    // Save implementation...

    const saved = await this.findById(user.id);
    ensure(saved !== null, 'User must be saved');
    ensure(saved.id.equals(user.id), 'User ID must match');
  }
}
```

### Invariant Enforcement

```typescript
export function invariant(
  condition: boolean,
  message: string
): void {
  if (!condition) {
    throw new Error(`Invariant failed: ${message}`);
  }
}

// Usage
class ShoppingCart {
  private items: CartItem[] = [];

  addItem(item: CartItem): void {
    this.items.push(item);
    this.recalculateTotal();

    // Invariant: Total must be correct
    invariant(
      this.calculateTotal().equals(this.total),
      'Cart total must match sum of items'
    );
  }

  private recalculateTotal(): void {
    this.total = this.items.reduce((sum, item) =>
      sum.add(item.price.multiply(item.quantity)),
      Money.zero()
    );
  }
}
```

## Domain Contracts

### Entity Invariants

```typescript
class Post implements Entity<PostId> {
  private status: PostStatus;

  constructor(
    private readonly id: PostId,
    private readonly title: PostTitle,
    private readonly content: PostContent,
    status: PostStatus
  ) {
    // Invariant: Status is always valid
    invariant(
      Object.values(PostStatus).includes(status),
      'Post status must be valid'
    );
    this.status = status;
  }

  publish(): void {
    // Precondition: Post is not already published
    if (this.status === PostStatus.PUBLISHED) {
      throw new Error('Precondition failed: Post already published');
    }

    this.status = PostStatus.PUBLISHED;
    this.publishedAt = new Date();

    // Postcondition: Status is published
    invariant(
      this.status === PostStatus.PUBLISHED,
      'Post status must be published after publish()'
    );

    // Postcondition: Published date is set
    invariant(
      this.publishedAt !== null,
      'Post must have published date'
    );
  }
}
```

### Value Object Contracts

```typescript
class Email {
  private constructor(public readonly value: string) {
    // Invariant: Email is always valid format
    invariant(
      this.isValid(value),
      'Email must be in valid format'
    );
  }

  static of(value: string): Email {
    // Precondition: Value is not null
    require(value !== null, 'Email cannot be null');
    require(value.trim().length > 0, 'Email cannot be empty');

    return new Email(value);
  }

  private isValid(email: string): boolean {
    const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return regex.test(email);
  }
}
```

### Use Case Contracts

```typescript
interface PublishPostUseCase extends Command<PublishPostInput, PublishPostOutput> {}

class PublishPostService implements PublishPostUseCase {
  async execute(input: PublishPostInput): Promise<PublishPostOutput> {
    // Preconditions
    require(input.postId !== null, 'postId cannot be null');
    require(input.postId.trim().length > 0, 'postId cannot be empty');

    const postId = new PostId(input.postId);

    // Get post
    const post = await this.postRepository.findById(postId);

    // Postcondition: Post must exist
    ensure(post !== null, 'Post must exist to publish');

    // Publish
    post.publish();

    // Save
    await this.postRepository.save(post);

    // Postcondition: Post status is published
    const updated = await this.postRepository.findById(postId);
    ensure(
      updated.getStatus() === PostStatus.PUBLISHED,
      'Post must be published'
    );

    return PublishPostOutput.success(post.id);
  }
}
```

## Contract Testing

### Test Preconditions

```typescript
describe('Post Repository', () => {
  @Test
  public async findById_with_null_postId_throws_error(): Promise<void> {
    const repository = new InMemoryPostRepository();

    // Test precondition enforcement
    await expect(
      repository.findById(null as unknown as PostId)
    ).rejects.toThrow('Precondition failed: postId cannot be null');
  }
});
```

### Test Postconditions

```typescript
describe('Post Repository', () => {
  @Test
  public async save_persists_post(): Promise<void> {
    const repository = new InMemoryPostRepository();
    const post = Post.create('Test Post', 'Content');

    await repository.save(post);

    // Test postcondition: Post can be retrieved
    const found = await repository.findById(post.getId());
    expect(found).not.toBeNull();
    expect(found.getId()).toEqual(post.getId());
  }
});
```

### Test Invariants

```typescript
describe('Shopping Cart', () => {
  @Test
  public invariant_total_must_match_items(): void {
    const cart = new ShoppingCart();

    cart.addItem({ productId: '1', name: 'Product 1', price: 10.00, quantity: 2 });
    cart.addItem({ productId: '2', name: 'Product 2', price: 5.00, quantity: 1 });

    // Test invariant: Total matches sum of items
    const total = cart.getTotal();
    const expectedTotal = 10.00 * 2 + 5.00 * 1; // 25.00
    expect(total.value).toBe(expectedTotal);
  }
});
```

## Benefits

### Clear Expectations

- Interfaces specify behavior clearly
- Preconditions and postconditions documented
- Invariants explicit

### Fail Fast

- Errors caught early
- Violations detected immediately
- Debugging easier

### Better Documentation

- Contracts serve as documentation
- Behavior explicitly specified
- No ambiguity

### Easier Testing

- Contracts directly translated to tests
- Precondition tests
- Postcondition tests
- Invariant tests

### Safer Code

- Invalid state prevented
- Invariants maintained
- Predictable behavior

## Best Practices

### ✅ DO

- Define contracts in interfaces
- Check preconditions in implementation
- Ensure postconditions in implementation
- Maintain invariants throughout
- Use contract checking utilities
- Test all contracts
- Document contracts with JSDoc

### ❌ DON'T

- Skip preconditions for performance
- Assume preconditions without checking
- Ignore postconditions
- Break invariants
- Use complex logic in contracts
- Forget to test contracts
- Document contracts separately

## Design by Contract in Nexus

### All Interfaces Have Contracts

```typescript
// Every repository interface
interface IPostRepository {
  /**
   * @precondition ...
   * @postcondition ...
   * @invariant ...
   */
  findById(postId: PostId): Promise<Post | null>;
}

// Every use case interface
interface PublishPostUseCase extends Command<Input, Output> {
  /**
   * @precondition ...
   * @postcondition ...
   */
  execute(input: Input): Promise<Output>;
}
```

### Contracts Enforced Automatically

```typescript
// Contract checking middleware
export function withContracts<T extends any>(
  target: T
): T {
  return new Proxy(target, {
    apply: (target, thisArg, args) => {
      // Check preconditions
      // Call method
      // Ensure postconditions
    }
  }) as unknown as T;
}
```

## References

- [docs/architecture/ARCHITECTURE.md](../architecture/ARCHITECTURE.md) - System architecture
- [docs/engineering/METHODOLOGIES.md](../METHODOLOGIES.md) - All methodologies
- [docs/engineering/clean_architecture/CLEAN_ARCHITECTURE.md](../clean_architecture/CLEAN_ARCHITECTURE.md) - Clean Architecture
- [docs/engineering/development/DEVELOPMENT_GUIDELINES.md](../development/DEVELOPMENT_GUIDELINES.md) - Development guidelines
- [CODING_AGENTS.md](../../CODING_AGENTS.md) - AI agent development guidelines
