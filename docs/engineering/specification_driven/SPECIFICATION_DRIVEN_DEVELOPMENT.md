# Specification Driven Development

## Overview

Specification Driven Development (SDD) is a development process where specifications drive implementation, not the reverse. Nexus uses SDD to ensure code matches documented requirements.

## Core Philosophy

### Specifications Drive Implementation

**Traditional Approach**:
```
1. Write code
2. Write documentation
3. Hope they match
→ Result: Code and documentation drift apart
```

**Specification Driven Approach**:
```
1. Write specifications (tests)
2. Implement to meet specifications
3. Code matches specifications exactly
→ Result: Code always meets documented requirements
```

### Executable Specifications

Specifications are **executable tests** that:
- Define expected behavior
- Run automatically
- Serve as documentation
- Prevent code drift

## Specification Types

### 1. Acceptance Criteria

Conditions that must be met before feature is complete.

**Example**:
```typescript
describe('User Registration Feature', () => {
  // Acceptance Criteria: User can register with valid credentials
  @Test
  public async user_can_register_with_valid_credentials(): Promise<void> {
    const result = await registerUser({
      email: 'newuser@example.com',
      password: 'SecurePass123!',
      name: 'John Doe'
    });

    expect(result.success).toBe(true);
    expect(result.user.id).toBeDefined();
    expect(result.user.email).toBe('newuser@example.com');
  }

  // Acceptance Criteria: Email must be unique
  @Test
  public async duplicate_email_registration_fails(): Promise<void> {
    await registerUser({
      email: 'existing@example.com',
      password: 'SecurePass123!',
      name: 'User 1'
    });

    const result = await registerUser({
      email: 'existing@example.com',
      password: 'AnotherPass456!',
      name: 'User 2'
    });

    expect(result.success).toBe(false);
    expect(result.error).toBe('EMAIL_ALREADY_EXISTS');
  }

  // Acceptance Criteria: Password must meet security requirements
  @Test
  public async weak_password_registration_fails(): Promise<void> {
    const result = await registerUser({
      email: 'newuser@example.com',
      password: 'weak',
      name: 'John Doe'
    });

    expect(result.success).toBe(false);
    expect(result.error).toBe('PASSWORD_TOO_WEAK');
  }
});
```

### 2. Behavior Specifications

Define behavior of system components.

**Example**:
```typescript
describe('User Entity', () => {
  // Specification: User can change email
  @Test
  public user_can_change_email(): void {
    const user = User.create('old@example.com', 'password', 'John');
    user.changeEmail(Email.of('new@example.com'));

    expect(user.getEmail().value).toBe('new@example.com');
  }

  // Specification: User cannot change email to invalid format
  @Test
  public user_cannot_change_email_to_invalid_format(): void {
    const user = User.create('user@example.com', 'password', 'John');

    expect(() => user.changeEmail(Email.of('invalid-email')))
      .toThrow('Invalid email format');
  }

  // Specification: User can change name
  @Test
  public user_can_change_name(): void {
    const user = User.create('user@example.com', 'password', 'John');
    user.changeName('Jane Doe');

    expect(user.getName()).toBe('Jane Doe');
  }

  // Specification: User cannot change name to empty
  @Test
  public user_cannot_change_name_to_empty(): void {
    const user = User.create('user@example.com', 'password', 'John');

    expect(() => user.changeName(''))
      .toThrow('Name cannot be empty');
  }
});
```

### 3. Interface Specifications

Define behavior through interfaces.

**Example**:
```typescript
// Specification: IPostRepository contract
interface IPostRepository {
  /**
   * Finds post by ID
   *
   * @precondition postId must be valid, non-empty string
   * @postcondition Returns Post entity if found
   * @postcondition Returns null if Post not found
   */
  findById(postId: PostId): Promise<Post | null>;

  /**
   * Saves post to repository
   *
   * @precondition Post must be valid (business rules satisfied)
   * @precondition Post.id must be valid PostId
   * @postcondition Post is persisted
   * @postcondition Post can be retrieved by findById
   */
  save(post: Post): Promise<void>;

  /**
   * Deletes post from repository
   *
   * @precondition Post with given ID must exist
   * @postcondition Post is removed from repository
   * @postcondition findById returns null for removed Post
   */
  delete(postId: PostId): Promise<void>;
}

// Implementation must satisfy specification
class SqlPostRepository implements IPostRepository {
  async findById(postId: PostId): Promise<Post | null> {
    // Implementation must satisfy postconditions
  }

  async save(post: Post): Promise<void> {
    // Implementation must satisfy postconditions
  }

  async delete(postId: PostId): Promise<void> {
    // Implementation must satisfy postconditions
  }
}
```

## Specification Process

### Step 1: Write Specification

Create executable specification (test):

```typescript
// Specification: User authentication flow
describe('User Authentication', () => {
  @Test
  public async when_user_provides_valid_credentials_then_token_is_returned(): Promise<void> {
    const credentials = {
      email: 'user@example.com',
      password: 'ValidPass123!'
    };

    const result = await authenticateUser(credentials);

    expect(result.success).toBe(true);
    expect(result.token).toBeDefined();
    expect(result.token).toMatch(/^eyJ/); // JWT format
  }

  @Test
  public async when_user_provides_invalid_email_then_error_is_returned(): Promise<void> {
    const credentials = {
      email: 'nonexistent@example.com',
      password: 'SomePass456!'
    };

    const result = await authenticateUser(credentials);

    expect(result.success).toBe(false);
    expect(result.error).toBe('INVALID_CREDENTIALS');
  }

  @Test
  public async when_user_provides_invalid_password_then_error_is_returned(): Promise<void> {
    const credentials = {
      email: 'user@example.com',
      password: 'WrongPass789!'
    };

    const result = await authenticateUser(credentials);

    expect(result.success).toBe(false);
    expect(result.error).toBe('INVALID_CREDENTIALS');
  }
});
```

### Step 2: Run Specification

Verify specification fails (RED phase):

```bash
npm test -- UserAuthentication.test.ts

# Result: Tests fail because implementation doesn't exist yet
```

### Step 3: Implement to Meet Specification

Write code to pass tests (GREEN phase):

```typescript
class AuthService {
  constructor(
    private readonly userRepository: IUserRepository,
    private readonly tokenGenerator: ITokenGenerator
  ) {}

  async authenticateUser(credentials: LoginCredentials): Promise<LoginResult> {
    const user = await this.userRepository.findByEmail(credentials.email);

    if (!user) {
      return LoginResult.failure('INVALID_CREDENTIALS');
    }

    const isValid = await this.verifyPassword(
      credentials.password,
      user.passwordHash
    );

    if (!isValid) {
      return LoginResult.failure('INVALID_CREDENTIALS');
    }

    const token = await this.tokenGenerator.generate(user);

    return LoginResult.success(user.id.value, token);
  }

  private async verifyPassword(password: string, hash: string): Promise<boolean> {
    return await bcrypt.compare(password, hash);
  }
}
```

### Step 4: Verify Specification

Run tests to ensure specification met:

```bash
npm test -- UserAuthentication.test.ts

# Result: All tests pass (GREEN)
```

## Specification Documentation

### README References

README links to specifications:

```markdown
## User Authentication

Complete specification and behavior defined in:
**[User Authentication Tests](src/features/user/tests/integration/UserAuthentication.test.ts)**

### Specification

User can authenticate with valid email and password:
- Email must be registered
- Password must match stored hash
- JWT token is returned on success
- Token expires after 24 hours

Error cases:
- Invalid email (not registered)
- Invalid password (wrong password)
```

### JSDoc with References

JSDoc links to specifications:

```typescript
/**
 * Authenticates user with credentials
 *
 * @example
 * ```typescript
 * const result = await authenticateUser({
 *   email: 'user@example.com',
 *   password: 'Password123!'
 * });
 *
 * if (result.success) {
 *   console.log('Token:', result.token);
 * }
 * ```
 *
 * @see [User Authentication Tests](src/features/user/tests/integration/UserAuthentication.test.ts) for complete specification
 */
export async function authenticateUser(
  credentials: LoginCredentials
): Promise<LoginResult>
```

## Specification Types in Nexus

### Domain Specifications

Specify domain behavior:

```typescript
describe('Domain: Post', () => {
  @Test
  public post_can_be_published(): void {
    const post = Post.create('Test', 'Content');
    post.publish();

    expect(post.getStatus()).toBe(PostStatus.PUBLISHED);
  }

  @Test
  public published_post_cannot_be_published_again(): void {
    const post = Post.create('Test', 'Content');
    post.publish();

    expect(() => post.publish()).toThrow('Post already published');
  }
});
```

### Use Case Specifications

Specify use case behavior:

```typescript
describe('Use Case: Publish Post', () => {
  @Test
  public async publish_post_saves_to_repository(): Promise<void> {
    const input = { postId: 'post-123' };

    const output = await publishPostUseCase.execute(input);

    expect(output.success).toBe(true);
    const post = await postRepository.findById('post-123');
    expect(post.getStatus()).toBe(PostStatus.PUBLISHED);
  }
});
```

### Integration Specifications

Specify component integration:

```typescript
describe('Integration: User Registration', () => {
  @Test
  public async complete_registration_flow(): Promise<void> {
    // Register
    const registerResult = await registerUser({
      email: 'new@example.com',
      password: 'SecurePass123!',
      name: 'John Doe'
    });

    expect(registerResult.success).toBe(true);

    // Verify email sent
    expect(mockEmailSender.sendEmail).toHaveBeenCalled();

    // Verify email
    const user = await findUserByEmail('new@example.com');
    expect(user).not.toBeNull();
    expect(user.email.value).toBe('new@example.com');
  }
});
```

### E2E Specifications

Specify complete feature workflows:

```typescript
describe('E2E: Blog Feature', () => {
  @Test
  public async user_creates_publishes_and_reads_blog_post(): Promise<void> {
    // User logs in
    const loginResult = await loginUser('user@example.com', 'Password123!');
    expect(loginResult.success).toBe(true);

    // User creates post
    const createResult = await createPost({
      title: 'My First Post',
      content: 'Hello World'
    }, loginResult.token);

    expect(createResult.success).toBe(true);

    // User publishes post
    const publishResult = await publishPost(createResult.postId, loginResult.token);
    expect(publishResult.success).toBe(true);

    // User reads post
    const readResult = await readPost(createResult.postId);
    expect(readResult.success).toBe(true);
    expect(readResult.post.title).toBe('My First Post');
    expect(readResult.post.status).toBe(PostStatus.PUBLISHED);
  }
});
```

## Benefits

### Specifications Before Implementation

- **Clarity**: What to build is clear before coding
- **Validation**: Requirements validated through tests
- **No Drift**: Code always matches specifications
- **Living Documentation**: Tests are always current

### Executable Specifications

- **Automated Verification**: Specifications verified automatically
- **Regression Prevention**: Changes break tests if specifications violated
- **No Outdated Docs**: Code changes require test updates
- **Concrete Examples**: Specifications show exactly how to use code

### Documentation Synchronized

- **Single Source of Truth**: Tests ARE documentation
- **No Drift**: Cannot become outdated
- **Examples Included**: Specifications include usage examples
- **Always Current**: Tests must pass to commit

## Best Practices

### ✅ DO

- Write specifications first (before code)
- Make specifications executable (tests)
- Use descriptive names for specifications
- Include all edge cases in specifications
- Document acceptance criteria as tests
- Link to specifications from README
- Use JSDoc to reference specifications
- Update specifications when requirements change

### ❌ DON'T

- Write code before specifications
- Write separate documentation that can drift
- Skip edge cases in specifications
- Use vague specification names
- Forget to update specifications
- Separate documentation from specifications
- Assume behavior without specifications
- Ignore failing specifications

## Specification Driven Development in Nexus

### TDD Generator Agent

The TDD Generator Agent creates specifications first:

```typescript
// Generated test specifications
describe('Feature Name', () => {
  @Test
  public specification_1(): void {
    // Executable specification
  }

  @Test
  public specification_2(): void {
    // Executable specification
  }
});
```

### Living Documentation

Tests serve as living documentation:

```typescript
// This test IS documentation
describe('User Registration', () => {
  // Complete behavior specification
});
```

### README Integration

README links to specifications:

```markdown
## User Registration

Complete specification in:
**[User Registration Tests](src/features/user/tests/integration/UserRegistration.test.ts)**
```

## Comparison with Traditional Approaches

### Traditional Approach

```
1. Write requirements document
2. Write code
3. Write documentation
4. Hope they match

Problems:
- Code and documentation drift apart
- Requirements unclear
- Changes not reflected in docs
- Manual verification needed
```

### Specification Driven

```
1. Write executable specifications (tests)
2. Implement to meet specifications
3. Specifications always pass
4. Documentation IS specifications

Advantages:
- Code always matches specifications
- Requirements clear and testable
- Changes require specification updates
- Automatic verification
```

## References

- [docs/architecture/ARCHITECTURE.md](../architecture/ARCHITECTURE.md) - System architecture
- [docs/engineering/METHODOLOGIES.md](../METHODOLOGIES.md) - All methodologies
- [docs/engineering/test_driven/TEST_DRIVEN_DEVELOPMENT.md](../test_driven/TEST_DRIVEN_DEVELOPMENT.md) - TDD
- [docs/engineering/behavior_driven/BEHAVIOR_DRIVEN_DEVELOPMENT.md](../behavior_driven/BEHAVIOR_DRIVEN_DEVELOPMENT.md) - BDD
- [docs/engineering/specification_by_example/SPECIFICATION_BY_EXAMPLE.md](../specification_by_example/SPECIFICATION_BY_EXAMPLE.md) - Specification by Example
- [CODING_AGENTS.md](../../CODING_AGENTS.md) - AI agent development guidelines
