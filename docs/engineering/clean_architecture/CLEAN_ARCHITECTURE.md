# Clean Architecture

## Overview

Clean Architecture is a software design philosophy that emphasizes separation of concerns and dependency inversion. Nexus implements a strict 4-layer Clean Architecture to ensure maintainability, testability, and business logic independence.

## Core Principles

### Dependency Rule

**Dependencies always point inward toward domain**:

```
┌─────────────────────────────────────┐
│   Frameworks & Drivers (Layer 4)   │  ← No dependencies
│      - Express, Database, UI          │
└──────────────┬──────────────────────┘
               │ depends on
┌──────────────▼──────────────────────┐
│   Adapters & Interfaces (Layer 3)     │
│      - Controllers, Repositories         │
└──────────────┬──────────────────────┘
               │ depends on
┌──────────────▼──────────────────────┐
│   Use Cases & Application (Layer 2)   │
│      - Business Orchestration          │
└──────────────┬──────────────────────┘
               │ depends on
┌──────────────▼──────────────────────┐
│   Domain & Entities (Layer 1)        │  ← No dependencies
│      - Business Logic                  │
└─────────────────────────────────────┘
```

### Key Points

- **Inner layers** don't know anything about outer layers
- **Outer layers** depend on interfaces defined in inner layers
- **Domain layer** contains business logic and rules
- **Use cases** orchestrate business logic
- **Adapters** handle external interactions
- **Frameworks** contain concrete implementations

## 4-Layer Architecture

### Layer 1: Domain (Core Business Logic)

**Purpose**: Pure business logic, no external dependencies

**Contains**:
- Entities
- Value Objects
- Domain Events
- Domain Services

**Rules**:
- NO dependencies on outer layers
- Pure business logic only
- Self-contained and testable
- No framework-specific code

**Example**:
```typescript
// domain/Post.ts
export class Post implements Entity<PostId> {
  constructor(
    private readonly id: PostId,
    private title: PostTitle,
    private content: PostContent,
    private status: PostStatus
  ) {}

  publish(): void {
    if (this.status === PostStatus.PUBLISHED) {
      throw new Error('Post already published');
    }
    this.status = PostStatus.PUBLISHED;
  }

  addComment(comment: Comment): void {
    // Business logic for adding comments
    this.comments.push(comment);
  }
}
```

### Layer 2: Use Cases (Application Logic)

**Purpose**: Orchestrate domain logic to fulfill user goals

**Contains**:
- Use Cases (Input Ports)
- Use Case Services (Implementations)
- DTOs (Data Transfer Objects)
- Output Ports (Repository, Presenter interfaces)

**Rules**:
- Depends ONLY on Layer 1 (Domain)
- Defines interfaces for Layer 3 (Adapters)
- Orchestrates business logic
- No business logic of its own

**Example**:
```typescript
// usecases/publish-post/PublishPostUseCase.ts
export interface PublishPostUseCase extends Command<PublishPostInput, PublishPostOutput> {}

// usecases/publish-post/PublishPostService.ts
export class PublishPostService implements PublishPostUseCase {
  constructor(
    private readonly postRepository: IPostRepository,
    private readonly eventPublisher: IEventPublisher
  ) {}

  async execute(input: PublishPostInput): Promise<PublishPostOutput> {
    const post = await this.postRepository.findById(input.postId);
    if (!post) {
      return PublishPostOutput.failure('Post not found');
    }

    post.publish();

    await this.postRepository.save(post);
    await this.eventPublisher.publish(new PostPublished(post.id, post.title, new Date()));

    return PublishPostOutput.success(post.id);
  }
}

// usecases/publish-post/PublishPostInput.ts
export interface PublishPostInput extends Input {
  postId: string;
}

// usecases/publish-post/PublishPostOutput.ts
export interface PublishPostOutput {
  success: boolean;
  postId?: string;
  message?: string;
}
```

### Layer 3: Adapters (External Interfaces)

**Purpose**: Handle external interactions and implementations

**Contains**:
- Controllers (REST, CLI)
- Presenters (JSON, HTML, Console)
- Repository Implementations
- Gateway Clients (External APIs)

**Rules**:
- Depends ONLY on Layer 1 and Layer 2
- Implements interfaces defined in Layer 2
- No business logic
- Handles technical concerns (HTTP, database, file I/O)

**Example**:
```typescript
// adapters/in/controllers/PublishPostWebController.ts
export class PublishPostWebController {
  constructor(private readonly publishPost: PublishPostUseCase) {}

  async handleRequest(req: Request, res: Response): Promise<void> {
    const input: PublishPostInput = {
      postId: req.params.id
    };

    const output = await this.publishPost.execute(input);

    if (output.success) {
      res.status(200).json({ success: true, postId: output.postId });
    } else {
      res.status(404).json({ success: false, message: output.message });
    }
  }
}

// adapters/out/repositories/SqlPostRepository.ts
export class SqlPostRepository implements IPostRepository {
  constructor(private readonly database: DatabaseConnection) {}

  async findById(id: string): Promise<Post | null> {
    const result = await this.database.query(
      'SELECT * FROM posts WHERE id = ?',
      [id]
    );
    return result ? PostMapper.toDomain(result) : null;
  }

  async save(post: Post): Promise<void> {
    const po = PostMapper.toPo(post);
    await this.database.query(
      'INSERT INTO posts (id, title, content, status) VALUES (?, ?, ?, ?)',
      [po.id, po.title, po.content, po.status]
    );
  }
}
```

### Layer 4: Frameworks (External Implementations)

**Purpose**: Concrete implementations of external systems

**Contains**:
- Express.js Setup
- Database Configuration (PostgreSQL, MySQL)
- Email Service (SMTP)
- File Storage (S3, Local)
- Authentication (JWT)

**Rules**:
- Depends ONLY on Layer 3 (Adapters)
- Contains concrete implementations
- No business logic
- Configured and wired together

**Example**:
```typescript
// frameworks/express/ExpressApp.ts
export class ExpressApp {
  constructor(
    private readonly publishPostController: PublishPostWebController,
    private readonly createPostController: CreatePostWebController
  ) {}

  setup(): Express {
    const app = express();

    app.use(express.json());

    app.post('/posts', (req, res) => this.createPostController.handleRequest(req, res));
    app.post('/posts/:id/publish', (req, res) => this.publishPostController.handleRequest(req, res));

    return app;
  }
}
```

## Vertical Slice Architecture

Nexus combines Clean Architecture with **Vertical Slice Architecture** - code organized by features, not technical layers.

### Feature-Based Structure

```
src/
├── features/
│   ├── post-management/
│   │   ├── domain/
│   │   │   ├── Post.ts
│   │   │   ├── PostId.ts
│   │   │   └── PostStatus.ts
│   │   ├── usecases/
│   │   │   ├── create-post/
│   │   │   │   ├── CreatePostUseCase.ts
│   │   │   │   ├── CreatePostService.ts
│   │   │   │   ├── CreatePostInput.ts
│   │   │   │   └── CreatePostOutput.ts
│   │   │   └── publish-post/
│   │   │       ├── PublishPostUseCase.ts
│   │   │       ├── PublishPostService.ts
│   │   │       ├── PublishPostInput.ts
│   │   │       └── PublishPostOutput.ts
│   │   ├── adapters/
│   │   │   ├── controllers/
│   │   │   │   ├── CreatePostWebController.ts
│   │   │   │   └── PublishPostWebController.ts
│   │   │   └── repositories/
│   │   │       └── SqlPostRepository.ts
│   │   └── tests/
│   │       ├── unit/domain/Post.test.ts
│   │       └── integration/usecases/CreatePostService.test.ts
│   └── user-management/
│       └── (same structure)
└── shared/
    ├── domain/
    ├── infrastructure/
    └── types/
```

### Benefits

- **Feature Co-location**: All code for feature together
- **Easy Navigation**: Find related code quickly
- **Independent Features**: Can add/remove features easily
- **Test Co-location**: Tests near code they test

## Design by Contract

Clean Architecture enforces **Design by Contract** - interfaces specify preconditions, postconditions, and invariants.

### Example

```typescript
// usecases/port/out/IPostRepository.ts
export interface IPostRepository {
  /**
   * @precondition postId must be a valid PostId
   * @postcondition Returns Post entity or throws PostNotFoundError
   * @invariant Post id, title remain unchanged
   */
  findById(postId: PostId): Promise<Post | null>;

  /**
   * @precondition Post must be valid (business rules satisfied)
   * @postcondition Post is persisted with generated ID if new
   * @invariant Post business invariants maintained
   */
  save(post: Post): Promise<void>;

  /**
   * @precondition Post must exist
   * @postcondition Post is removed from storage
   * @invariant No orphaned data left behind
   */
  delete(postId: PostId): Promise<void>;
}
```

## Dependency Rules

### STRICT - Never Violate

```typescript
// ❌ WRONG - Domain depends on Framework
import { ExpressRequest } from 'express';

class Post {
  publish(request: ExpressRequest): void {
    // Domain logic shouldn't depend on Express
  }
}

// ✅ CORRECT - Framework depends on Domain
class PublishPostController {
  constructor(private readonly publishPost: PublishPostUseCase) {}

  async handleRequest(request: ExpressRequest): Promise<void> {
    const input = { postId: request.params.id };
    await this.publishPost.execute(input);
  }
}
```

### STRICT - Never Skip Layers

```typescript
// ❌ WRONG - Controller directly calls Repository
class PublishPostController {
  constructor(private readonly repository: IPostRepository) {}

  async handleRequest(req: Request): Promise<void> {
    const post = await this.repository.findById(req.params.id);
    post.publish(); // Business logic in controller!
  }
}

// ✅ CORRECT - Controller → Use Case → Repository
class PublishPostController {
  constructor(private readonly publishPost: PublishPostUseCase) {}

  async handleRequest(req: Request): Promise<void> {
    const input = { postId: req.params.id };
    await this.publishPost.execute(input);
  }
}

class PublishPostService implements PublishPostUseCase {
  constructor(private readonly repository: IPostRepository) {}

  async execute(input: PublishPostInput): Promise<PublishPostOutput> {
    const post = await this.repository.findById(input.postId);
    post.publish(); // Business logic in use case
    await this.repository.save(post);
  }
}
```

## Benefits

### Testability

```typescript
// Easy to test - no dependencies on frameworks
describe('Post', () => {
  @Test
  public publish_changes_status_to_published(): void {
    const post = new Post(
      PostId.generate(),
      PostTitle.of('Test'),
      PostContent.of('Content'),
      PostStatus.DRAFT
    );

    post.publish();

    assertEquals(PostStatus.PUBLISHED, post.getStatus());
  }
});
```

### Maintainability

- **Independent layers**: Change framework without affecting business logic
- **Clear separation**: Business logic isolated from technical concerns
- **Easy to reason**: Each layer has single responsibility

### Flexibility

```typescript
// Can swap implementations easily
const developmentConfig = {
  postRepository: new InMemoryPostRepository()
};

const productionConfig = {
  postRepository: new SqlPostRepository(databaseUrl)
};

// Same use case works with both
const publishPost = new PublishPostService(config.postRepository);
```

## Best Practices

### ✅ DO

- Depend only on abstractions (interfaces)
- Keep domain layer pure (no external dependencies)
- Use dependency injection
- Follow dependency rule (inward only)
- Organize by features (vertical slices)
- Apply Design by Contract

### ❌ DON'T

- Depend on concrete classes outside your layer
- Skip layers (controller → repository)
- Put business logic in controllers
- Use framework types in domain layer
- Share domain logic between features
- Violate dependency direction

## References

- [docs/architecture/ARCHITECTURE.md](../architecture/ARCHITECTURE.md) - System architecture
- [docs/engineering/METHODOLOGIES.md](../METHODOLOGIES.md) - All methodologies
- [docs/engineering/patterns/PATTERNS.md](../patterns/PATTERNS.md) - Design patterns
- [docs/engineering/domain_driven/DOMAIN_DRIVEN_DESIGN.md](../domain_driven/DOMAIN_DRIVEN_DESIGN.md) - Domain-Driven Design
- [CODING_AGENTS.md](../../CODING_AGENTS.md) - AI agent development guidelines
