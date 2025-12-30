# Living Documentation

## Overview

Living Documentation is a documentation philosophy where documentation evolves with the codebase and always stays current. In Nexus, **tests ARE the documentation**, ensuring documentation never becomes outdated.

## Core Philosophy

### Tests ARE Documentation

Traditional documentation becomes outdated because:
- Documentation is written separately from code
- Developers forget to update docs
- Documentation and code drift apart

Living documentation solves this by:
- Tests serve as executable specifications
- Documentation is code itself
- Documentation cannot drift from implementation

### Always Current

Because tests must pass to commit:
- Documentation stays synchronized with implementation
- Outdated documentation causes test failures
- Developers must update tests when changing behavior

## Executable Specifications

### What Are Executable Specifications?

Tests that serve as documentation because they:
- Define expected behavior of system
- Run automatically to verify correctness
- Provide examples of how to use the code
- Keep documentation synchronized with implementation
- Serve as regression tests

### Example: User Registration

```typescript
// This test IS the documentation for "How to Register a User"

describe('User Registration Feature', () => {
  describe('GIVEN a new user registers', () => {
    const validEmail = 'newuser@example.com';
    const validPassword = 'SecurePass123!';

    describe('WHEN all requirements are met', () => {
      it('THEN user account is created successfully', async () => {
        // This IS the documentation for registration flow
        const result = await registerUser({
          email: validEmail,
          password: validPassword,
          name: 'John Doe'
        });

        // Documentation: Expected outcome
        expect(result.success).toBe(true);
        expect(result.user.id).toBeDefined();
        expect(result.user.email).toBe(validEmail);
        expect(result.user.name).toBe('John Doe');
        expect(result.user.createdAt).toBeDefined();
      });

      it('THEN verification email is sent', async () => {
        // This IS the documentation for email flow
        const result = await registerUser({
          email: validEmail,
          password: validPassword,
          name: 'John Doe'
        });

        // Documentation: Email verification flow
        expect(result.verificationToken).toBeDefined();
        expect(result.verificationSentAt).toBeDefined();
        expect(mockEmailSender.sendEmail).toHaveBeenCalledWith(
          validEmail,
          'Verify Your Email',
          expect.any(String) // Contains verification link
        );
      });

      it('THEN password is hashed securely', async () => {
        // This IS the documentation for security
        const result = await registerUser({
          email: validEmail,
          password: validPassword,
          name: 'John Doe'
        });

        const user = await findUserByEmail(validEmail);
        expect(user.passwordHash).not.toBe(validPassword); // Hashed
        expect(user.passwordHash).toMatch(/^\$2[aby]\$/); // bcrypt format
      });
    });

    describe('WHEN email already exists', () => {
      it('THEN registration fails with duplicate email error', async () => {
        // First registration
        await registerUser({
          email: validEmail,
          password: validPassword,
          name: 'John Doe'
        });

        // Duplicate registration - THIS DOCUMENTS ERROR CASES
        const result = await registerUser({
          email: validEmail,
          password: 'AnotherPass456!',
          name: 'Jane Doe'
        });

        expect(result.success).toBe(false);
        expect(result.error).toBe('EMAIL_ALREADY_EXISTS');
        expect(result.message).toContain('already registered');
      });
    });

    describe('WHEN password is too weak', () => {
      it('THEN registration fails with password requirements error', async () => {
        const result = await registerUser({
          email: validEmail,
          password: 'weak',
          name: 'John Doe'
        });

        expect(result.success).toBe(false);
        expect(result.error).toBe('PASSWORD_TOO_WEAK');
        expect(result.message).toContain('at least 8 characters');
        expect(result.message).toContain('uppercase');
        expect(result.message).toContain('lowercase');
        expect(result.message).toContain('number');
        expect(result.message).toContain('special character');
      });
    });

    describe('WHEN email is invalid format', () => {
      it('THEN registration fails with invalid email error', async () => {
        const result = await registerUser({
          email: 'invalid-email',
          password: validPassword,
          name: 'John Doe'
        });

        expect(result.success).toBe(false);
        expect(result.error).toBe('INVALID_EMAIL_FORMAT');
        expect(result.message).toContain('valid email address');
      });
    });
  });
});
```

### What This Provides

1. **Expected Behavior**: Clear documentation of what registration does
2. **Edge Cases**: All error cases documented with tests
3. **Code Examples**: Shows how to call the API
4. **Error Handling**: Documents error codes and messages
5. **Security**: Documents security measures (hashing)
6. **Always Current**: Fails if code behavior changes

## Documentation Structure

### README References Test Suites

README files link to specific test suites:

```markdown
## User Registration

Complete behavior specification and usage examples are in:
**[User Registration Tests](src/features/user/tests/integration/UserRegistration.test.ts)**

### Registration Flow

1. User provides email, password, and name
2. System validates input (see tests for requirements)
3. System hashes password securely
4. System creates user account
5. System sends verification email
6. System returns user ID and verification token

See tests for complete error handling and edge cases.
```

### API Documentation Generated from Tests

Tests generate API documentation:

```typescript
// Test serves as API documentation
describe('User Registration API', () => {
  @Test
  public post_users_registers_new_user(): void {
    // This documents POST /api/users endpoint
    const response = await request(app)
      .post('/api/users')
      .send({
        email: 'newuser@example.com',
        password: 'SecurePass123!',
        name: 'John Doe'
      });

    // Documents expected response
    expect(response.status).toBe(201);
    expect(response.body).toEqual({
      success: true,
      user: {
        id: expect.any(String),
        email: 'newuser@example.com',
        name: 'John Doe',
        createdAt: expect.any(String)
      },
      verificationToken: expect.any(String)
    });
  }

  @Test
  public post_users_with_duplicate_email_returns_409(): void {
    // Documents error response
    const response = await request(app)
      .post('/api/users')
      .send({
        email: 'existing@example.com',
        password: 'SecurePass123!',
        name: 'John Doe'
      });

    expect(response.status).toBe(409);
    expect(response.body).toEqual({
      success: false,
      error: 'EMAIL_ALREADY_EXISTS',
      message: expect.stringContaining('already registered')
    });
  }
});
```

## Documentation Types

### 1. Behavior Documentation

Tests document expected behavior:

```typescript
describe('Post Publishing', () => {
  @Test
  public when_publishing_draft_post_then_status_becomes_published(): void {
    // Documents: Publishing changes status
    const post = Post.create('Test Post', 'Content', PostStatus.DRAFT);
    post.publish();
    assertEquals(PostStatus.PUBLISHED, post.getStatus());
  }

  @Test
  public when_publishing_published_post_then_throws_error(): void {
    // Documents: Cannot publish already published post
    const post = Post.create('Test Post', 'Content', PostStatus.PUBLISHED);
    expect(() => post.publish()).toThrow('Post already published');
  }
});
```

### 2. Usage Documentation

Tests show how to use code:

```typescript
describe('User Authentication', () => {
  @Test
  public shows_how_to_authenticate_user(): void {
    // Documents: How to authenticate
    const user = User.create('test@example.com', 'Password123!');
    const authService = new AuthService();

    const result = authService.authenticate(
      'test@example.com',
      'Password123!'
    );

    // Shows expected result structure
    assertTrue(result.success);
    assertNotNull(result.token);
    assertNotNull(result.user);
  }
});
```

### 3. Error Documentation

Tests document error cases:

```typescript
describe('User Validation', () => {
  @Test
  public documents_email_validation_errors(): void {
    // Documents all email error cases
    expect(() => Email.of('')).toThrow('Email cannot be empty');
    expect(() => Email.of('invalid')).toThrow('Invalid email format');
    expect(() => Email.of('no-at-sign.com')).toThrow('Invalid email format');
    expect(() => Email.of('multiple@@at.com')).toThrow('Invalid email format');
  }

  @Test
  public documents_password_validation_errors(): void {
    // Documents all password error cases
    expect(() => Password.validate('short')).toThrow('at least 8 characters');
    expect(() => Password.validate('nouppercase')).toThrow('uppercase letter');
    expect(() => Password.validate('NOLOWERCASE')).toThrow('lowercase letter');
    expect(() => Password.validate('Nonumber')).toThrow('number');
    expect(() => Password.validate('Nospecial')).toThrow('special character');
  }
});
```

### 4. Edge Case Documentation

Tests document edge cases:

```typescript
describe('Shopping Cart', () => {
  @Test
  public handles_zero_quantity_gracefully(): void {
    // Documents edge case: Zero quantity
    const cart = new ShoppingCart();
    cart.addItem({ productId: '1', name: 'Product', price: 10.00, quantity: 0 });
    assertEquals(0, cart.getItemCount());
  }

  @Test
  public handles_negative_quantity_gracefully(): void {
    // Documents edge case: Negative quantity
    const cart = new ShoppingCart();
    expect(() => cart.addItem({ productId: '1', name: 'Product', price: 10.00, quantity: -1 }))
      .toThrow('Quantity must be positive');
  }

  @Test
  public handles_duplicate_items_correctly(): void {
    // Documents edge case: Adding same item twice
    const cart = new ShoppingCart();
    cart.addItem({ productId: '1', name: 'Product', price: 10.00, quantity: 2 });
    cart.addItem({ productId: '1', name: 'Product', price: 10.00, quantity: 3 });
    assertEquals(1, cart.getItemCount());
    assertEquals(5, cart.getItems()[0].quantity);
  }
});
```

## JSDoc + Tests

### Combining JSDoc with Tests

```typescript
/**
 * Registers a new user account.
 *
 * @example
 * ```typescript
 * const result = await registerUser({
 *   email: 'newuser@example.com',
 *   password: 'SecurePass123!',
 *   name: 'John Doe'
 * });
 *
 * if (result.success) {
 *   console.log('User registered:', result.user);
 * }
 * ```
 *
 * @see [User Registration Tests](src/features/user/tests/integration/UserRegistration.test.ts) for complete behavior specification
 */
export async function registerUser(
  input: RegisterUserInput
): Promise<RegisterUserOutput> {
  // Implementation...
}

// Test provides complete documentation
describe('User Registration', () => {
  // ... comprehensive tests
});
```

## Documentation Coverage

### Coverage Targets

- **All public APIs**: Have test examples
- **All error cases**: Documented with tests
- **All edge cases**: Covered in tests
- **All business rules**: Explicitly tested

### Verification

```bash
# Check test coverage
npm run test:coverage

# Verify all public APIs have tests
npm run test:coverage:api

# Check for undocumented APIs
npm run test:coverage:check
```

## Benefits

### No Outdated Documentation

- Tests fail if code changes
- Documentation stays synchronized
- Developers must update tests

### Examples Always Work

- Tests run on every commit
- Examples are verified
- No broken code examples

### Regression Protection

- Tests serve as regression tests
- Changes verified against expected behavior
- Prevents regressions

### Self-Documenting Code

- Tests are documentation
- No separate documentation drift
- Clear examples in code

## Best Practices

### ✅ DO

- Write tests as documentation
- Use descriptive test names
- Document all edge cases
- Document all error cases
- Provide usage examples
- Reference tests from README
- Use Given-When-Then format
- Keep tests current

### ❌ DON'T

- Write separate documentation that can drift
- Ignore edge cases in tests
- Use vague test names
- Skip error documentation
- Forget to update tests when code changes
- Create documentation separate from tests
- Assume behavior without tests

## Living Documentation in Nexus

### Test Structure

All features have comprehensive test suites:

```
src/features/xxx/tests/
├── unit/                 # Unit tests for domain logic
├── integration/          # Integration tests for use cases
└── e2e/                 # End-to-end tests for features
```

### README References

README files reference test suites:

```markdown
## User Management

**[Feature Documentation](src/features/user/README.md)**

**[Test Suite](src/features/user/tests/)**

### Behavior

See [User Registration Tests](src/features/user/tests/integration/UserRegistration.test.ts) for complete behavior specification.

### Examples

See [User Authentication Tests](src/features/user/tests/integration/UserAuthentication.test.ts) for usage examples.
```

## References

- [docs/architecture/ARCHITECTURE.md](../architecture/ARCHITECTURE.md) - System architecture
- [docs/engineering/METHODOLOGIES.md](../METHODOLOGIES.md) - All methodologies
- [docs/engineering/test_driven/TEST_DRIVEN_DEVELOPMENT.md](../test_driven/TEST_DRIVEN_DEVELOPMENT.md) - TDD
- [docs/engineering/behavior_driven/BEHAVIOR_DRIVEN_DEVELOPMENT.md](../behavior_driven/BEHAVIOR_DRIVEN_DEVELOPMENT.md) - BDD
- [docs/engineering/conventions/TESTING_CONVENTIONS.md](../conventions/TESTING_CONVENTIONS.md) - Testing conventions
- [CODING_AGENTS.md](../../CODING_AGENTS.md) - AI agent development guidelines
