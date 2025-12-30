# Specification by Example

## Overview

Specification by Example is a practice of using concrete examples to clarify abstract requirements. Nexus uses Specification by Example to ensure all team members have shared understanding of what should be built.

## Core Philosophy

### Concrete Examples Clarify Behavior

Abstract requirements lead to misunderstandings:

```
Abstract: "The system should handle user registration"
↓ Leads to:
- What fields are required?
- What validation rules apply?
- What happens on success?
- What happens on failure?
```

Concrete examples clarify behavior:

```
Example: "When user registers with valid email and password"
→ Then user account is created
→ Then verification email is sent
→ Then user can log in after verifying email

Example: "When user registers with existing email"
→ Then registration fails
→ Then error message "Email already registered" is shown
```

## Example Types

### 1. Valid Input Examples

Show what happens with valid inputs:

```typescript
describe('User Registration', () => {
  describe('Valid Inputs', () => {
    it('accepts standard email format', async () => {
      const result = await registerUser({
        email: 'user@example.com',
        password: 'SecurePass123!',
        name: 'John Doe'
      });

      expect(result.success).toBe(true);
      expect(result.user.email).toBe('user@example.com');
    });

    it('accepts email with plus sign', async () => {
      const result = await registerUser({
        email: 'user+tag@example.com',
        password: 'SecurePass123!',
        name: 'John Doe'
      });

      expect(result.success).toBe(true);
    });

    it('accepts password with special characters', async () => {
      const result = await registerUser({
        email: 'user@example.com',
        password: 'P@ssw0rd!#$',
        name: 'John Doe'
      });

      expect(result.success).toBe(true);
    });

    it('accepts name with spaces', async () => {
      const result = await registerUser({
        email: 'user@example.com',
        password: 'SecurePass123!',
        name: 'John Q. Public'
      });

      expect(result.success).toBe(true);
      expect(result.user.name).toBe('John Q. Public');
    });

    it('accepts minimum password length (8 characters)', async () => {
      const result = await registerUser({
        email: 'user@example.com',
        password: 'Secur1!',
        name: 'John Doe'
      });

      expect(result.success).toBe(true);
    });
  });
});
```

### 2. Invalid Input Examples

Show what happens with invalid inputs:

```typescript
describe('User Registration', () => {
  describe('Invalid Inputs', () => {
    it('rejects empty email', async () => {
      const result = await registerUser({
        email: '',
        password: 'SecurePass123!',
        name: 'John Doe'
      });

      expect(result.success).toBe(false);
      expect(result.error).toBe('INVALID_EMAIL');
      expect(result.message).toContain('Email is required');
    });

    it('rejects email without @ symbol', async () => {
      const result = await registerUser({
        email: 'invalidemail.com',
        password: 'SecurePass123!',
        name: 'John Doe'
      });

      expect(result.success).toBe(false);
      expect(result.error).toBe('INVALID_EMAIL_FORMAT');
    });

    it('rejects password without uppercase', async () => {
      const result = await registerUser({
        email: 'user@example.com',
        password: 'lowercase123!',
        name: 'John Doe'
      });

      expect(result.success).toBe(false);
      expect(result.error).toBe('PASSWORD_TOO_WEAK');
      expect(result.message).toContain('uppercase letter');
    });

    it('rejects password without lowercase', async () => {
      const result = await registerUser({
        email: 'user@example.com',
        password: 'UPPERCASE123!',
        name: 'John Doe'
      });

      expect(result.success).toBe(false);
      expect(result.error).toBe('PASSWORD_TOO_WEAK');
      expect(result.message).toContain('lowercase letter');
    });

    it('rejects password without number', async () => {
      const result = await registerUser({
        email: 'user@example.com',
        password: 'UpperLower!',
        name: 'John Doe'
      });

      expect(result.success).toBe(false);
      expect(result.error).toBe('PASSWORD_TOO_WEAK');
      expect(result.message).toContain('number');
    });

    it('rejects password without special character', async () => {
      const result = await registerUser({
        email: 'user@example.com',
        password: 'UpperLower123',
        name: 'John Doe'
      });

      expect(result.success).toBe(false);
      expect(result.error).toBe('PASSWORD_TOO_WEAK');
      expect(result.message).toContain('special character');
    });

    it('rejects password shorter than 8 characters', async () => {
      const result = await registerUser({
        email: 'user@example.com',
        password: 'Short1!',
        name: 'John Doe'
      });

      expect(result.success).toBe(false);
      expect(result.error).toBe('PASSWORD_TOO_WEAK');
      expect(result.message).toContain('at least 8 characters');
    });
  });
});
```

### 3. Edge Case Examples

Show behavior at boundaries:

```typescript
describe('User Registration', () => {
  describe('Edge Cases', () => {
    it('handles name with 255 characters (max)', async () => {
      const longName = 'A'.repeat(255);
      const result = await registerUser({
        email: 'user@example.com',
        password: 'SecurePass123!',
        name: longName
      });

      expect(result.success).toBe(true);
      expect(result.user.name).toBe(longName);
    });

    it('rejects name with 256 characters (over max)', async () => {
      const tooLongName = 'A'.repeat(256);
      const result = await registerUser({
        email: 'user@example.com',
        password: 'SecurePass123!',
        name: tooLongName
      });

      expect(result.success).toBe(false);
      expect(result.error).toBe('NAME_TOO_LONG');
    });

    it('handles duplicate email registration', async () => {
      // First registration
      await registerUser({
        email: 'duplicate@example.com',
        password: 'SecurePass123!',
        name: 'John Doe'
      });

      // Duplicate registration
      const result = await registerUser({
        email: 'duplicate@example.com',
        password: 'AnotherPass456!',
        name: 'Jane Doe'
      });

      expect(result.success).toBe(false);
      expect(result.error).toBe('EMAIL_ALREADY_EXISTS');
    });

    it('handles rapid duplicate registrations', async () => {
      const promises = [];
      for (let i = 0; i < 5; i++) {
        promises.push(registerUser({
          email: 'rapid@example.com',
          password: 'SecurePass123!',
          name: `User ${i}`
        }));
      }

      const results = await Promise.all(promises);
      const successes = results.filter(r => r.success);

      // Only one should succeed
      expect(successes.length).toBe(1);
    });
  });
});
```

### 4. Integration Scenario Examples

Show complete workflows:

```typescript
describe('User Registration Workflow', () => {
  @Test
  public async complete_registration_flow(): Promise<void> {
    // Step 1: User registers
    const registerResult = await registerUser({
      email: 'newuser@example.com',
      password: 'SecurePass123!',
      name: 'John Doe'
    });

    expect(registerResult.success).toBe(true);
    expect(registerResult.user.id).toBeDefined();
    expect(registerResult.verificationToken).toBeDefined();

    // Step 2: Verification email is sent
    expect(mockEmailSender.sendEmail).toHaveBeenCalledWith(
      'newuser@example.com',
      'Verify Your Email',
      expect.stringContaining('/verify/')
    );

    // Step 3: User verifies email
    const verifyResult = await verifyEmail(
      registerResult.verificationToken
    );

    expect(verifyResult.success).toBe(true);
    expect(verifyResult.user.verified).toBe(true);

    // Step 4: User can now log in
    const loginResult = await login({
      email: 'newuser@example.com',
      password: 'SecurePass123!'
    });

    expect(loginResult.success).toBe(true);
    expect(loginResult.token).toBeDefined();
    expect(loginResult.user).toBeDefined();
  }

  @Test
  public async user_cannot_login_without_verification(): Promise<void> {
    // User registers
    const registerResult = await registerUser({
      email: 'newuser@example.com',
      password: 'SecurePass123!',
      name: 'John Doe'
    });

    // User tries to login before verifying email
    const loginResult = await login({
      email: 'newuser@example.com',
      password: 'SecurePass123!'
    });

    expect(loginResult.success).toBe(false);
    expect(loginResult.error).toBe('EMAIL_NOT_VERIFIED');
  }
});
```

## Example Tables

### Example Mapping Table

Use tables to organize examples:

| Scenario | Given | When | Then |
|----------|--------|-------|-------|
| Valid registration | Email: user@example.com, Password: Secure123! | User registers | Account created, email sent |
| Duplicate email | Email already registered | User registers with same email | Error: EMAIL_ALREADY_EXISTS |
| Weak password | Password: weak | User registers | Error: PASSWORD_TOO_WEAK |
| Long name | Name: 256 characters | User registers | Error: NAME_TOO_LONG |

### Decision Table

Use tables for complex rules:

| Email Valid | Password Valid | Name Valid | Email Exists | Result |
|-------------|----------------|------------|-------------|--------|
| Yes | Yes | Yes | No | SUCCESS |
| No | Yes | Yes | No | INVALID_EMAIL |
| Yes | No | Yes | No | PASSWORD_TOO_WEAK |
| Yes | Yes | No | No | INVALID_NAME |
| Yes | Yes | Yes | Yes | EMAIL_ALREADY_EXISTS |

## Examples in Documentation

### README Examples

```markdown
## User Registration

### Valid Registration

```typescript
const result = await registerUser({
  email: 'newuser@example.com',
  password: 'SecurePass123!',
  name: 'John Doe'
});

console.log('User ID:', result.user.id);
console.log('Verification Token:', result.verificationToken);
```

### Error Handling

```typescript
// Duplicate email
const result = await registerUser({
  email: 'existing@example.com',
  password: 'SecurePass123!',
  name: 'Jane Doe'
});

if (!result.success) {
  console.error('Error:', result.error);
  console.error('Message:', result.message);
  // Output: Error: EMAIL_ALREADY_EXISTS
  // Output: Message: Email is already registered
}
```

### Password Requirements

Valid password must contain:
- At least 8 characters
- At least 1 uppercase letter
- At least 1 lowercase letter
- At least 1 number
- At least 1 special character

Examples:
- ✅ `SecurePass123!` - Valid
- ✅ `MyP@ssw0rd` - Valid
- ❌ `password` - Too simple (no uppercase, number, special)
- ❌ `PASSWORD123!` - No lowercase
- ❌ `password123!` - No uppercase
- ❌ `Password!` - No number
```

### API Documentation Examples

```typescript
/**
 * Registers a new user
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
 *   console.log('Verification email sent to:', result.user.email);
 * } else {
 *   console.error('Registration failed:', result.error);
 * }
 * ```
 */
export async function registerUser(
  input: RegisterUserInput
): Promise<RegisterUserOutput>
```

## Examples in Tests

### Behavior Specification

Tests serve as executable specifications:

```typescript
describe('User Registration', () => {
  // This is executable specification for registration behavior
  it('specifies registration flow with valid inputs', async () => {
    const result = await registerUser({
      email: 'valid@example.com',
      password: 'SecurePass123!',
      name: 'Valid Name'
    });

    // Specification: Account should be created
    expect(result.success).toBe(true);
    expect(result.user.id).toBeDefined();

    // Specification: Verification email should be sent
    expect(result.verificationToken).toBeDefined();
    expect(mockEmailSender.sendEmail).toHaveBeenCalledWith(
      'valid@example.com',
      expect.any(String),
      expect.any(String)
    );

    // Specification: User data should be correct
    expect(result.user.email).toBe('valid@example.com');
    expect(result.user.name).toBe('Valid Name');
    expect(result.user.verified).toBe(false);
    expect(result.user.createdAt).toBeInstanceOf(Date);
  });
});
```

### Edge Case Coverage

Examples cover all edge cases:

```typescript
describe('Email Validation Examples', () => {
  const validEmails = [
    'user@example.com',
    'user+tag@example.com',
    'user.name@example.com',
    'user123@example-domain.com'
  ];

  validEmails.forEach(email => {
    it(`accepts valid email: ${email}`, async () => {
      const result = await registerUser({
        email,
        password: 'SecurePass123!',
        name: 'Test User'
      });

      expect(result.success).toBe(true);
    });
  });

  const invalidEmails = [
    { email: '', expected: 'EMAIL_REQUIRED' },
    { email: 'no-at-symbol.com', expected: 'INVALID_EMAIL_FORMAT' },
    { email: 'no-domain@', expected: 'INVALID_EMAIL_FORMAT' },
    { email: 'two@@symbol.com', expected: 'INVALID_EMAIL_FORMAT' }
  ];

  invalidEmails.forEach(({ email, expected }) => {
    it(`rejects invalid email "${email}": ${expected}`, async () => {
      const result = await registerUser({
        email,
        password: 'SecurePass123!',
        name: 'Test User'
      });

      expect(result.success).toBe(false);
      expect(result.error).toBe(expected);
    });
  });
});
```

## Benefits

### Shared Understanding

- Examples are concrete and unambiguous
- All team members understand requirements
- No misinterpretation of abstract descriptions

### Test Coverage

- Examples directly translate to tests
- Edge cases identified upfront
- Comprehensive test coverage

### Documentation

- Examples serve as documentation
- Usage examples are always current
- API documentation is generated from tests

### Reduced Ambiguity

- Abstract requirements clarified
- Edge cases identified
- Behavior explicitly defined

## Best Practices

### ✅ DO

- Use concrete examples for requirements
- Document all valid inputs
- Document all invalid inputs
- Cover edge cases
- Use example tables for complex rules
- Make examples executable as tests
- Include examples in documentation
- Use Given-When-Then format

### ❌ DON'T

- Rely on abstract descriptions only
- Skip edge cases
- Assume behavior without examples
- Leave requirements ambiguous
- Ignore error cases
- Create examples that don't run
- Separate examples from tests
- Use vague examples

## Specification by Example in Nexus

### TDD Generator Agent

The TDD Generator Agent uses Specification by Example when creating tests:

```typescript
// Generated tests include comprehensive examples
describe('Feature Name', () => {
  describe('Valid Inputs', () => {
    // All valid input examples
  });

  describe('Invalid Inputs', () => {
    // All invalid input examples
  });

  describe('Edge Cases', () => {
    // All edge cases
  });

  describe('Integration Scenarios', () => {
    // Complete workflow examples
  });
});
```

### Living Documentation

Examples become part of living documentation:

```typescript
// Tests ARE the documentation
describe('User Registration', () => {
  // These examples are executable documentation
});
```

## References

- [docs/architecture/ARCHITECTURE.md](../architecture/ARCHITECTURE.md) - System architecture
- [docs/engineering/METHODOLOGIES.md](../METHODOLOGIES.md) - All methodologies
- [docs/engineering/test_driven/TEST_DRIVEN_DEVELOPMENT.md](../test_driven/TEST_DRIVEN_DEVELOPMENT.md) - TDD
- [docs/engineering/behavior_driven/BEHAVIOR_DRIVEN_DEVELOPMENT.md](../behavior_driven/BEHAVIOR_DRIVEN_DEVELOPMENT.md) - BDD
- [docs/engineering/living_documentation/LIVING_DOCUMENTATION.md](../living_documentation/LIVING_DOCUMENTATION.md) - Living Documentation
- [CODING_AGENTS.md](../../CODING_AGENTS.md) - AI agent development guidelines
