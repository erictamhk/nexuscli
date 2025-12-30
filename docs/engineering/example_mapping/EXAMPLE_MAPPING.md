# Example Mapping

## Overview

Example Mapping is a collaborative practice that turns abstract requirements into concrete, testable examples. Nexus uses Example Mapping to ensure all team members have shared understanding of requirements.

## What Is Example Mapping?

### Definition

Example Mapping is a facilitation technique where teams create concrete examples to clarify and understand user stories and acceptance criteria.

### Purpose

- Turn abstract stories into concrete examples
- Identify edge cases and rules
- Clarify ambiguity
- Create shared understanding
- Generate test scenarios

## Process

### Step 1: Prepare User Story

Start with a user story:

```
As a customer
I want to apply a coupon code to my order
So that I can get a discount
```

### Step 2: Map Examples

Create a table with examples:

| Scenario | Given | When | Then |
|----------|--------|-------|-------|
| Valid coupon | Customer has coupon code CUSTOMER10 | Customer applies coupon at checkout | Order total is reduced by 10% |
| Expired coupon | Coupon CUSTOMER20 has expired | Customer applies coupon at checkout | Error message shows "Coupon has expired" |
| Invalid code | Customer enters fake code FAKE123 | Customer applies coupon at checkout | Error message shows "Invalid coupon code" |
| Already used | Coupon USEDONCE already used by customer | Customer tries to reuse coupon | Error message shows "Coupon already used" |

### Step 3: Add Questions

Identify questions that need answering:

| Question | Answer |
|----------|---------|
| Can coupons be combined? | No, only one coupon per order |
| Is there a minimum order value? | Yes, minimum $50 to apply coupon |
| Can coupons be applied to sale items? | No, coupons don't apply to already discounted items |
| Do coupons expire? | Yes, coupons expire after 30 days |

### Step 4: Add Rules

Extract business rules:

| Rule | Description |
|-------|-------------|
| Single Coupon | Only one coupon can be applied per order |
| Minimum Value | Order must be $50+ to apply coupon |
| Sale Exclusion | Coupons don't apply to sale items |
| Expiration | Coupons expire after 30 days from creation |
| Single Use | Each coupon can only be used once per customer |

### Step 5: Identify Edge Cases

Find scenarios that need special handling:

| Edge Case | Description |
|------------|-------------|
| Zero order value | Free order with coupon (should be handled) |
| Maximum discount | Coupon gives 100% discount (should be prevented) |
| Negative order | Customer has returns causing negative total (should handle) |

## Complete Example: Password Reset

### User Story

```
As a user who forgot my password
I want to reset it via email
So that I can regain access to my account
```

### Example Mapping Table

| Scenario | Given | When | Then |
|----------|--------|-------|-------|
| Successful reset | User requests reset for email@example.com | User clicks email link and sets new password | User can log in with new password |
| Expired token | User requested reset 2 hours ago | User clicks email reset link | Error shows "Reset link has expired" |
| Invalid token | User manually enters token "fake-token" | User clicks fake link | Error shows "Invalid reset link" |
| Already used | User already used reset link | User clicks same link again | Error shows "Link already used" |
| No account | User requests reset for unknown@email.com | System processes request | Email shows "Account not found" or similar |

### Questions

| Question | Answer |
|----------|---------|
| How long is token valid? | 1 hour |
| Can user request multiple resets? | Yes, new token invalidates previous |
| Does new password need to be different from old? | No, can be same |
| Is user logged out after reset? | Yes, all sessions invalidated |

### Rules

| Rule | Description |
|-------|-------------|
| Token Expiration | Reset tokens expire after 1 hour |
| Single Use | Each reset token can only be used once |
| Previous Tokens | New token invalidates all previous tokens |
| Session Invalidation | All user sessions invalidated after reset |
| Password Requirements | New password must meet security requirements |

### Edge Cases

| Edge Case | Description |
|------------|-------------|
| Multiple requests | User requests reset 5 times in 1 minute |
| Same email different browser | User opens link on different browser/device |
| No internet | User has no internet when clicking link |

## Collaborative Process

### Who Participates?

- **Product Owner**: Provides user stories
- **Developers**: Provide technical perspective
- **QA**: Identify test scenarios
- **Stakeholders**: Provide domain knowledge
- **Facilitator**: Guides the mapping process

### Setup

1. Large table or whiteboard
2. Sticky notes for examples, questions, rules
3. Markers in different colors:
   - Yellow: Examples
   - Pink: Questions
   - Blue: Rules
   - Green: Edge cases

### Flow

```
1. Product Owner reads user story
   ↓
2. Team asks questions
   ↓
3. Team creates examples (sticky notes)
   ↓
4. Questions emerge (pink notes)
   ↓
5. Answers provided, rules identified (blue notes)
   ↓
6. Edge cases discussed (green notes)
   ↓
7. Repeat until understanding is clear
```

## From Examples to Tests

### Executable Specifications

Example mapping directly generates test cases:

```typescript
describe('Password Reset Feature', () => {
  describe('GIVEN user requests password reset', () => {
    describe('WHEN reset token is valid', () => {
      it('THEN user can set new password', async () => {
        const user = await createUser('user@example.com', 'OldPass123!');
        const token = await requestPasswordReset('user@example.com');

        const result = await resetPassword(token, 'NewPass456!');

        expect(result.success).toBe(true);
        const updatedUser = await findUser('user@example.com');
        expect(await bcrypt.compare('NewPass456!', updatedUser.passwordHash)).toBe(true);
      });

      it('THEN old password no longer works', async () => {
        const user = await createUser('user@example.com', 'OldPass123!');
        const token = await requestPasswordReset('user@example.com');

        await resetPassword(token, 'NewPass456!');

        const result = await login('user@example.com', 'OldPass123!');
        expect(result.success).toBe(false);
      });
    });

    describe('WHEN reset token has expired (> 1 hour)', () => {
      it('THEN error shows token expired', async () => {
        const user = await createUser('user@example.com', 'OldPass123!');
        const token = await requestPasswordReset('user@example.com');
        // Simulate token expiration
        expireToken(token);

        const result = await resetPassword(token, 'NewPass456!');

        expect(result.success).toBe(false);
        expect(result.error).toBe('RESET_TOKEN_EXPIRED');
      });
    });

    describe('WHEN reset token is invalid', () => {
      it('THEN error shows invalid token', async () => {
        const fakeToken = 'fake-reset-token-123';

        const result = await resetPassword(fakeToken, 'NewPass456!');

        expect(result.success).toBe(false);
        expect(result.error).toBe('INVALID_RESET_TOKEN');
      });
    });

    describe('WHEN reset token already used', () => {
      it('THEN error shows token already used', async () => {
        const user = await createUser('user@example.com', 'OldPass123!');
        const token = await requestPasswordReset('user@example.com');

        await resetPassword(token, 'NewPass456!');

        const result2 = await resetPassword(token, 'AnotherPass789!');

        expect(result2.success).toBe(false);
        expect(result2.error).toBe('RESET_TOKEN_ALREADY_USED');
      });
    });
  });
});
```

## Benefits

### Shared Understanding

- All team members understand requirements
- Questions are asked and answered
- Assumptions are exposed

### Better Tests

- Concrete examples directly translate to tests
- Edge cases identified upfront
- Test coverage is comprehensive

### Reduced Rework

- Ambiguity discovered early
- Misunderstandings prevented
- Development stays on track

### Documentation

- Examples serve as documentation
- Rules are explicit
- Test scenarios are documented

## Best Practices

### ✅ DO

- Facilitate with entire team
- Use physical space (whiteboard, table)
- Create concrete examples
- Ask questions early
- Identify rules and edge cases
- Collaborate on answers
- Document decisions
- Generate tests from examples

### ❌ DON'T

- Facilitate with only developers
- Use abstract examples
- Skip edge cases
- Ignore questions
- Make assumptions without asking
- Work in silos
- Forget to document decisions
- Write tests before examples

## Example Mapping in Nexus

### Feature Planner Agent

The Feature Planner Agent uses Example Mapping when breaking down features:

```typescript
// Example: Add Shopping Cart

// User Story
As a customer
I want to add items to my cart
So that I can purchase them together

// Example Mapping
| Scenario | Given | When | Then |
|----------|--------|-------|-------|
| Add item | Cart is empty | Customer adds product | Cart contains product |
| Add duplicate | Cart contains product | Customer adds same product | Quantity increases |
| Out of stock | Product has 0 inventory | Customer adds product | Error shows out of stock |
| Multiple items | Cart has 2 products | Customer adds 3rd | Cart contains 3 products |

// Rules Generated
- Each product in cart is unique
- Quantities add up for duplicates
- Cannot add out-of-stock items
- Cart persists across sessions

// Tests Generated
describe('Shopping Cart', () => {
  @Test
  public add_item_to_empty_cart_succeeds(): void {
    // ...
  }

  @Test
  public add_duplicate_increases_quantity(): void {
    // ...
  }

  @Test
  public add_out_of_stock_item_fails(): void {
    // ...
  }

  @Test
  public add_multiple_items_to_cart(): void {
    // ...
  }
});
```

## References

- [docs/architecture/ARCHITECTURE.md](../architecture/ARCHITECTURE.md) - System architecture
- [docs/engineering/METHODOLOGIES.md](../METHODOLOGIES.md) - All methodologies
- [docs/engineering/behavior_driven/BEHAVIOR_DRIVEN_DEVELOPMENT.md](../behavior_driven/BEHAVIOR_DRIVEN_DEVELOPMENT.md) - BDD
- [docs/engineering/test_driven/TEST_DRIVEN_DEVELOPMENT.md](../test_driven/TEST_DRIVEN_DEVELOPMENT.md) - TDD
- [docs/engineering/specification_by_example/SPECIFICATION_BY_EXAMPLE.md](../specification_by_example/SPECIFICATION_BY_EXAMPLE.md) - Specification by Example
- [CODING_AGENTS.md](../../CODING_AGENTS.md) - AI agent development guidelines
