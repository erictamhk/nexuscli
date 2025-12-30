# Behavior-Driven Development (BDD)

## Overview

Behavior-Driven Development (BDD) is an agile software development practice that encourages collaboration between developers, QA, and non-technical stakeholders. Nexus uses BDD to define features by behavior, not implementation details.

## Core Philosophy

### Features Defined by Behavior

Focus on **what** the system does, not **how** it does it.

**Example**:
```
❌ Implementation-focused:
- We need a database table for users
- We need an API endpoint for user registration
- We need a form for user input

✅ Behavior-focused:
- As a new user
- I want to register with email and password
- So that I can access the system
```

### Given-When-Then Format

Standard format for describing behavior:

```
GIVEN (context)
WHEN (action)
THEN (outcome)
```

**Example**:
```
GIVEN a registered user with valid credentials
WHEN user logs in with correct email and password
THEN user should receive authentication token
AND token should be valid for 24 hours
```

## User Stories

### Format

```
As a <role>
I want to <feature>
So that <benefit>
```

### Examples

**Registration Story**:
```
As a new user
I want to register with email and password
So that I can access the system
```

**Login Story**:
```
As a registered user
I want to log in with my credentials
So that I can access my account
```

**Password Reset Story**:
```
As a user who forgot my password
I want to reset it via email
So that I can regain access to my account
```

## Acceptance Criteria

### Definition

Acceptance criteria are specific conditions that must be met before a user story is considered complete. They are **executable tests** in BDD.

### Examples

**User Registration Acceptance Criteria**:

```typescript
describe('User Registration Feature', () => {
  describe('GIVEN a new user registers', () => {
    describe('WHEN all requirements are met', () => {
      it('THEN user account is created successfully', async () => {
        const result = await registerUser({
          email: 'newuser@example.com',
          password: 'SecurePass123!',
          name: 'John Doe'
        });

        expect(result.success).toBe(true);
        expect(result.user.id).toBeDefined();
      });

      it('THEN verification email is sent', async () => {
        const result = await registerUser({
          email: 'newuser@example.com',
          password: 'SecurePass123!',
          name: 'John Doe'
        });

        expect(result.verificationToken).toBeDefined();
        expect(result.emailSent).toBe(true);
      });

      it('THEN password is hashed securely', async () => {
        const result = await registerUser({
          email: 'newuser@example.com',
          password: 'SecurePass123!',
          name: 'John Doe'
        });

        const user = await findUserByEmail('newuser@example.com');
        expect(user.passwordHash).not.toBe('SecurePass123!');
        expect(bcrypt.compare('SecurePass123!', user.passwordHash)).toBe(true);
      });
    });

    describe('WHEN email already exists', () => {
      it('THEN registration fails with duplicate email error', async () => {
        await registerUser({
          email: 'existing@example.com',
          password: 'Password123!',
          name: 'Existing User'
        });

        const result = await registerUser({
          email: 'existing@example.com',
          password: 'AnotherPass456!',
          name: 'Another User'
        });

        expect(result.success).toBe(false);
        expect(result.error).toBe('EMAIL_ALREADY_EXISTS');
      });
    });

    describe('WHEN password is too weak', () => {
      it('THEN registration fails with password requirements error', async () => {
        const result = await registerUser({
          email: 'newuser@example.com',
          password: 'weak',
          name: 'John Doe'
        });

        expect(result.success).toBe(false);
        expect(result.error).toBe('PASSWORD_TOO_WEAK');
      });
    });
  });
});
```

## BDD Framework Integration

### Testing with BDD Style

Nexus uses Vitest with BDD-style describe/it:

```typescript
describe('Shopping Cart Feature', () => {
  describe('GIVEN an empty cart', () => {
    let cart: ShoppingCart;

    beforeEach(() => {
      cart = new ShoppingCart();
    });

    describe('WHEN user adds item', () => {
      beforeEach(async () => {
        await cart.addItem({
          productId: 'prod-123',
          name: 'Product Name',
          price: 29.99,
          quantity: 2
        });
      });

      it('THEN cart contains the item', () => {
        const items = cart.getItems();
        expect(items.length).toBe(1);
        expect(items[0].productId).toBe('prod-123');
      });

      it('THEN cart total is calculated correctly', () => {
        const total = cart.getTotal();
        expect(total).toBe(59.98); // 29.99 * 2
      });
    });
  });

  describe('GIVEN cart with items', () => {
    let cart: ShoppingCart;

    beforeEach(async () => {
      cart = new ShoppingCart();
      await cart.addItem({ productId: 'prod-1', name: 'Product 1', price: 10.00, quantity: 1 });
      await cart.addItem({ productId: 'prod-2', name: 'Product 2', price: 20.00, quantity: 1 });
    });

    describe('WHEN user removes item', () => {
      beforeEach(async () => {
        await cart.removeItem('prod-1');
      });

      it('THEN cart contains remaining items', () => {
        const items = cart.getItems();
        expect(items.length).toBe(1);
        expect(items[0].productId).toBe('prod-2');
      });

      it('THEN cart total is recalculated', () => {
        const total = cart.getTotal();
        expect(total).toBe(20.00);
      });
    });
  });
});
```

## Living Documentation

### Tests as Documentation

BDD tests serve as **executable specifications** and living documentation:

```typescript
// This test IS the documentation for "How to register a user"
describe('User Registration', () => {
  it('shows how to register a new user', async () => {
    const result = await registerUser({
      email: 'test@example.com',
      password: 'SecurePass123!',
      name: 'Test User'
    });

    // Expected behavior documented in test
    expect(result.success).toBe(true);
    expect(result.user.email).toBe('test@example.com');
    expect(result.user.name).toBe('Test User');
  });
});
```

### README References Test Suites

README files reference specific test suites:

```markdown
## User Registration

See [User Registration Tests](src/features/user/tests/integration/UserRegistration.test.ts) for complete behavior specification.

### Requirements

- Email must be unique
- Password must be at least 8 characters
- Password must contain uppercase, lowercase, number, and special character
- Verification email must be sent

### Examples

```typescript
const result = await registerUser({
  email: 'newuser@example.com',
  password: 'SecurePass123!',
  name: 'John Doe'
});
```
```

## Impact Mapping

### What Is Impact Mapping?

Impact mapping is a strategic planning technique that helps teams align on business value and prioritize features.

### Structure

```
Goal ← Outcome ← Output ← Activity
```

- **Goal**: Business objective
- **Outcome**: Desired change in behavior
- **Output**: Software feature
- **Activity**: Development work

### Example

**Feature**: Shopping Cart

```
Goal: Increase revenue by 20%

Outcome: Users make larger purchases

Output: Shopping cart functionality

Activities:
- Implement add to cart
- Implement cart persistence
- Implement cart total calculation
- Implement checkout from cart
```

## Example Mapping

### What Is Example Mapping?

Example mapping is a collaborative process that turns user stories into concrete examples.

### Process

1. **Write user story**
2. **Add acceptance criteria**
3. **Add examples**
4. **Add questions**
5. **Add rules**

### Example: Password Reset

**User Story**:
```
As a user who forgot password
I want to reset it via email
So that I can regain access
```

**Acceptance Criteria**:
- User receives email with reset link
- Reset link is valid for 1 hour
- User can set new password
- Old password no longer works

**Examples**:

| Scenario | Given | When | Then |
|----------|--------|-------|-------|
| Successful reset | User requests reset | User clicks email link and sets new password | User can log in with new password |
| Expired token | User requests reset | User clicks email link after 2 hours | Token is invalid, error shown |
| Invalid token | User enters wrong token | User clicks fake reset link | Token is invalid, error shown |

**Questions**:
- Should user be logged in before requesting reset?
- Can user request multiple resets?
- Should old password be invalidated immediately?

**Rules**:
- Reset token expires after 1 hour
- User can request new reset at any time
- Old password invalidated after successful reset

## BDD in Nexus

### Feature Planning with BDD

The Feature Planner Agent uses BDD to break down features:

```typescript
// Feature Planner Agent breaks down "Add Shopping Cart"
// into user stories and acceptance criteria

Feature: Shopping Cart

User Stories:
  1. As a customer
     I want to add items to my cart
     So that I can purchase them together

  2. As a customer
     I want to remove items from my cart
     So that I can change my mind before purchase

  3. As a customer
     I want to see cart total
     So that I know how much I'll pay

Acceptance Criteria:
  - Items can be added to cart
  - Items can be removed from cart
  - Cart persists across sessions
  - Total is calculated correctly
  - Discounts are applied correctly
```

### Test Generation with BDD

The TDD Generator Agent creates BDD-style tests:

```typescript
// TDD Generator creates executable specifications
describe('Shopping Cart', () => {
  describe('GIVEN a customer with an empty cart', () => {
    describe('WHEN customer adds item to cart', () => {
      it('THEN cart contains the item', async () => {
        // ...
      });

      it('THEN cart total includes item price', async () => {
        // ...
      });
    });
  });
});
```

## Best Practices

### ✅ DO

- Write features from user perspective
- Use Given-When-Then format
- Create executable acceptance criteria
- Collaborate with stakeholders
- Use concrete examples
- Make tests self-documenting
- Reference tests from documentation

### ❌ DON'T

- Focus on implementation details
- Write vague acceptance criteria
- Skip edge cases
- Ignore stakeholder input
- Create tests that aren't readable
- Write implementation-focused tests
- Separate documentation from tests

## Benefits

### Collaboration

- Shared language between technical and non-technical stakeholders
- Clear expectations from the start
- Reduced misunderstandings

### Test Coverage

- Acceptance criteria are executable tests
- Tests verify business requirements
- Requirements always stay current

### Documentation

- Tests are living documentation
- No outdated documentation
- Examples are always runnable

### Development Focus

- Focus on delivering value
- Clear acceptance criteria
- Prevents scope creep

## References

- [docs/architecture/ARCHITECTURE.md](../architecture/ARCHITECTURE.md) - System architecture
- [docs/engineering/METHODOLOGIES.md](../METHODOLOGIES.md) - All methodologies
- [docs/engineering/test_driven/TEST_DRIVEN_DEVELOPMENT.md](../test_driven/TEST_DRIVEN_DEVELOPMENT.md) - TDD
- [docs/engineering/example_mapping/EXAMPLE_MAPPING.md](../example_mapping/EXAMPLE_MAPPING.md) - Example Mapping
- [docs/engineering/specification_by_example/SPECIFICATION_BY_EXAMPLE.md](../specification_by_example/SPECIFICATION_BY_EXAMPLE.md) - Specification by Example
- [CODING_AGENTS.md](../../CODING_AGENTS.md) - AI agent development guidelines
