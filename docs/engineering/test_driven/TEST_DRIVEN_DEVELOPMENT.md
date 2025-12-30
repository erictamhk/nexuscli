# Test-Driven Development (TDD)

## Overview

Test-Driven Development (TDD) is a software development process that relies on very short development cycles: requirements are turned into specific test cases, then the software is improved to pass the new tests. Nexus strictly enforces TDD for all feature development.

## TDD Workflow

### The TDD Cycle

```
┌─────────────────────────────────────────┐
│  1. RED                            │
│     Write failing test                 │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  2. GREEN                          │
│     Write minimal code to pass test    │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  3. REFACTOR                        │
│     Improve code while tests pass       │
└──────────────┬──────────────────────┘
               │
               ▼
         (repeat cycle)
```

### Step 1: RED - Write Failing Test

Write a test that fails because the functionality doesn't exist yet.

**Purpose**:
- Define expected behavior
- Specify requirements
- Create executable specification

**Example**:
```typescript
// Post.test.ts
describe('Post Entity', () => {
  @Test
  public publish_changes_status_to_published(): void {
    // Arrange
    const post = new Post(
      PostId.generate(),
      PostTitle.of('Test Post'),
      PostContent.of('Test Content'),
      PostStatus.DRAFT
    );

    // Act
    post.publish();

    // Assert
    assertEquals(PostStatus.PUBLISHED, post.getStatus());
  }
});
```

**Result**: Test fails (RED) because `publish()` method doesn't exist.

### Step 2: GREEN - Write Minimal Code

Write just enough code to make the test pass.

**Purpose**:
- Implement simplest solution
- Focus on making test pass
- Don't over-engineer

**Example**:
```typescript
// Post.ts
export class Post implements Entity<PostId> {
  constructor(
    private readonly id: PostId,
    private title: PostTitle,
    private content: PostContent,
    private status: PostStatus
  ) {}

  publish(): void {
    this.status = PostStatus.PUBLISHED;
  }

  getStatus(): PostStatus {
    return this.status;
  }
}
```

**Result**: Test passes (GREEN).

### Step 3: REFACTOR - Improve Code

Improve code while tests continue to pass.

**Purpose**:
- Clean up code
- Remove duplication
- Improve design
- Maintain test coverage

**Example**:
```typescript
// Post.ts - After Refactor
export class Post implements Entity<PostId> {
  private status: PostStatus;

  constructor(
    private readonly id: PostId,
    private readonly title: PostTitle,
    private readonly content: PostContent,
    status: PostStatus
  ) {
    this.status = status;
  }

  publish(): void {
    if (this.status === PostStatus.PUBLISHED) {
      throw new Error('Post already published');
    }
    this.status = PostStatus.PUBLISHED;
  }

  getStatus(): PostStatus {
    return this.status;
  }
}

// Add new test for validation
@Test
public publish_already_published_post_throws_error(): void {
  const post = new Post(
    PostId.generate(),
    PostTitle.of('Test'),
    PostContent.of('Content'),
    PostStatus.PUBLISHED
  );

  expect(() => post.publish()).toThrow('Post already published');
}
```

## Test Structure

### Arrange-Act-Assert (AAA) Pattern

All tests follow AAA pattern for clarity:

```typescript
@Test
public descriptive_test_name_with_underscores(): void {
  // Arrange - Set up test data and dependencies
  const post = Post.create('Test Post');
  const repository = new InMemoryPostRepository();

  // Act - Execute code being tested
  repository.save(post);
  const found = repository.findById(post.getId());

  // Assert - Verify expected outcome
  assertNotNull(found);
  assertEquals(post.getId(), found.getId());
}
```

### Descriptive Test Names

Tests describe behavior, not implementation:

```typescript
// ✅ GOOD - Behavior-driven
@Test
public when_user_creates_post_then_status_is_draft(): void {
  // ...
}

@Test
public when_user_publishes_post_then_status_becomes_published(): void {
  // ...
}

@Test
public when_user_publishes_already_published_post_then_error_is_thrown(): void {
  // ...
}

// ❌ BAD - Implementation-focused
@Test
public test_post(): void {
  // ...
}

@Test
public publish_method(): void {
  // ...
}
```

## BDD Integration

TDD combines with **Behavior Driven Development** for behavior-focused tests:

```typescript
describe('Post Entity', () => {
  describe('Given a draft post', () => {
    const post = Post.create('Test Post');

    describe('When user publishes post', () => {
      it('Then status should be published', () => {
        post.publish();
        assertEquals(PostStatus.PUBLISHED, post.getStatus());
      });

      it('Then published date should be set', () => {
        const before = new Date();
        post.publish();
        const after = new Date();

        const publishedAt = post.getPublishedAt();
        assertNotNull(publishedAt);
        assertTrue(publishedAt >= before && publishedAt <= after);
      });
    });
  });
});
```

## Specification by Example

Use concrete examples in tests to clarify behavior:

```typescript
describe('Post Title Validation', () => {
  describe('Valid Titles', () => {
    it('accepts 5 characters', () => {
      const title = PostTitle.of('Hello');
      assertEquals('Hello', title.value);
    });

    it('accepts 200 characters', () => {
      const title = PostTitle.of('A'.repeat(200));
      assertEquals(200, title.value.length);
    });

    it('accepts standard text', () => {
      const title = PostTitle.of('My Blog Post Title');
      assertEquals('My Blog Post Title', title.value);
    });
  });

  describe('Invalid Titles', () => {
    it('rejects 4 characters', () => {
      expect(() => PostTitle.of('Test'))
        .toThrow('Title must be 5-200 characters');
    });

    it('rejects 201 characters', () => {
      expect(() => PostTitle.of('A'.repeat(201)))
        .toThrow('Title must be 5-200 characters');
    });

    it('rejects empty string', () => {
      expect(() => PostTitle.of(''))
        .toThrow('Title cannot be empty');
    });

    it('rejects null', () => {
      expect(() => PostTitle.of(null as any))
        .toThrow('Title cannot be null');
    });
  });
});
```

## Test Categories

### Unit Tests

Test individual components in isolation:

```typescript
describe('Post Entity', () => {
  @Test
  public publish_sets_status_to_published(): void {
    const post = new Post(/* ... */);
    post.publish();
    assertEquals(PostStatus.PUBLISHED, post.getStatus());
  }
});
```

### Integration Tests

Test components working together:

```typescript
describe('Publish Post Use Case', () => {
  @Test
  public async publish_post_saves_to_repository(): Promise<void> {
    const repository = new InMemoryPostRepository();
    const useCase = new PublishPostService(repository);

    const output = await useCase.execute({ postId: 'post-123' });

    assertTrue(output.success);
    const post = await repository.findById('post-123');
    assertEquals(PostStatus.PUBLISHED, post.getStatus());
  }
});
```

### E2E Tests

Test complete feature workflows:

```typescript
describe('Blog Feature E2E', () => {
  @Test
  public async user_creates_and_publishes_blog_post(): Promise<void> {
    // Create post
    const createOutput = await createPost({
      title: 'My First Post',
      content: 'Hello World'
    });
    assertTrue(createOutput.success);

    // Publish post
    const publishOutput = await publishPost({
      postId: createOutput.postId
    });
    assertTrue(publishOutput.success);

    // Verify post is published
    const post = await getPost(createOutput.postId);
    assertEquals('My First Post', post.title);
    assertEquals(PostStatus.PUBLISHED, post.status);
  }
});
```

## Test Doubles

### Mocks

Simulate dependencies with predefined behavior:

```typescript
describe('PublishPostService', () => {
  @Test
  public async publish_post_saves_to_repository(): Promise<void> {
    const mockRepository = {
      findById: jest.fn().mockResolvedValue(post),
      save: jest.fn().mockResolvedValue(undefined)
    } as jest.Mocked<IPostRepository>;

    const useCase = new PublishPostService(mockRepository);

    await useCase.execute({ postId: 'post-123' });

    expect(mockRepository.save).toHaveBeenCalledWith(post);
  }
});
```

### Stubs

Provide fixed responses for specific calls:

```typescript
describe('PostService', () => {
  @Test
  public async get_post_by_id_returns_post(): Promise<void> {
    const stubRepository = {
      findById: jest.fn()
        .mockResolvedValueOnce(post)
        .mockResolvedValueOnce(null)
    } as jest.Mocked<IPostRepository>;

    const service = new PostService(stubRepository);

    const found = await service.getPostById('post-123');
    assertEquals(post, found);

    const notFound = await service.getPostById('post-456');
    assertNull(notFound);
  }
});
```

### Fakes

Simple implementations for testing:

```typescript
class InMemoryPostRepository implements IPostRepository {
  private posts: Map<string, Post> = new Map();

  async findById(id: string): Promise<Post | null> {
    return this.posts.get(id) || null;
  }

  async save(post: Post): Promise<void> {
    this.posts.set(post.getId().value, post);
  }
}

describe('PostService', () => {
  @Test
  public async create_and_retrieve_post(): void {
    const repository = new InMemoryPostRepository();
    const service = new PostService(repository);

    const post = await service.createPost('Test', 'Content');

    const found = await repository.findById(post.getId().value);
    assertNotNull(found);
  }
});
```

## Test Isolation

### Independent Tests

Each test is independent:

```typescript
describe('Post Repository', () => {
  let repository: InMemoryPostRepository;

  beforeEach(() => {
    // Fresh repository for each test
    repository = new InMemoryPostRepository();
  });

  @Test
  public first_test_creates_post(): void {
    const post = Post.create('Post 1');
    repository.save(post);
    // ...
  }

  @Test
  public second_test_creates_post(): void {
    // This test is not affected by first_test
    const post = Post.create('Post 2');
    repository.save(post);
    assertEquals(1, repository.findAll().length);
  }
});
```

### Test Setup and Teardown

```typescript
describe('Database Tests', () => {
  let connection: DatabaseConnection;

  beforeAll(async () => {
    connection = await createTestDatabase();
  });

  afterAll(async () => {
    await connection.close();
  });

  beforeEach(async () => {
    await connection.migrate();
    await connection.seed('test-data');
  });

  afterEach(async () => {
    await connection.cleanDatabase();
  });
});
```

## Best Practices

### ✅ DO

- Write tests BEFORE code (TDD)
- Follow AAA pattern (Arrange-Act-Assert)
- Use descriptive test names
- Test behavior, not implementation
- Keep tests isolated and independent
- Use test doubles for external dependencies
- Keep tests fast and focused
- Aim for 85%+ coverage

### ❌ DON'T

- Write code then tests
- Test implementation details
- Use vague test names (test1, test2)
- Share state between tests
- Test multiple things in one test
- Skip edge cases
- Write slow integration tests for unit tests
- Ignore failing tests

## Nexus TDD Requirements

### Mandatory Rules

1. **Tests First**: Always write test before implementation
2. **AAA Pattern**: All tests follow Arrange-Act-Assert
3. **RED → GREEN → REFACTOR**: Complete the TDD cycle
4. **Descriptive Names**: Tests describe behavior
5. **Test Isolation**: Tests are independent
6. **Coverage Targets**: 85%+ overall, 100% for domain logic

### Test Coverage

- **Domain Logic**: 100% coverage
- **Use Cases**: 95%+ coverage
- **Adapters**: 80%+ coverage
- **Overall**: 85%+ coverage

### Executable Specifications

Tests serve as living documentation:
- Define expected behavior
- Provide usage examples
- Always stay current
- No outdated documentation possible

## References

- [docs/architecture/ARCHITECTURE.md](../architecture/ARCHITECTURE.md) - System architecture
- [docs/engineering/METHODOLOGIES.md](../METHODOLOGIES.md) - All methodologies
- [docs/engineering/conventions/TESTING_CONVENTIONS.md](../conventions/TESTING_CONVENTIONS.md) - Testing conventions
- [docs/engineering/behavior_driven/BEHAVIOR_DRIVEN_DEVELOPMENT.md](../behavior_driven/BEHAVIOR_DRIVEN_DEVELOPMENT.md) - BDD
- [CODING_AGENTS.md](../../CODING_AGENTS.md) - AI agent development guidelines
