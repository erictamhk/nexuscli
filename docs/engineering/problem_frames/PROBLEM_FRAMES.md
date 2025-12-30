# Problem Frames Approach

## Overview

The Problem Frames Approach, created by Michael A. Jackson, is a software development methodology that focuses on **problem framing** before **problem solving**. Nexus uses this approach to ensure we solve the right problems in the right business context.

## Core Philosophy

### Solution is in Computer

Software provides solutions to problems, but **problems exist in the real world**, not in the computer.

### Problem is in the World

Problems involve people, organizations, relationships, and real-world phenomena. We must study the problem world, not just the software.

### Study the Problem World

Analyze problem domains deeply, travel away from the computer to understand the real problem context.

### Parallel Decomposition

Decompose user requirements in parallel, not hierarchically. This reveals independent problem domains.

## Key Concepts

### Problem Domains

Different parts of the real world involved in the problem:

**Examples**:
- **Users**: People who use the system
- **Products**: Physical or virtual products being sold
- **Orders**: Business transactions
- **Inventory**: Stock management
- **Payments**: Financial transactions

### Phenomena

Individuals, events, states, relationships, behaviors in domains:

**Types**:
- **Individuals**: Entities (User, Product, Order)
- **Events**: Actions taken or occurring (User logs in, Order placed)
- **States**: Conditions being true or false (User is active, Order is pending)
- **Relationships**: Connections between individuals (User has Orders)

### Shared Phenomena

Phenomena that exist in multiple domains:

**Example**:
```
Users Domain           Orders Domain          Products Domain
  │                        │                      │
  └── User ──────────────────┼──────────────────────> Product
                              │
                              └── Order

"User" exists in Users and Orders domains
"Product" exists in Orders and Products domains
"Order" exists in Users and Products domains
```

### Machine

The software system being built to solve the problem.

### Machine Interface

Where the software system meets and interacts with problem domains.

## Recognized Problem Frames

### 1. Required Behavior Frame

Control part of world to satisfy conditions.

**Example**: Thermostat
- Problem: Keep room at desired temperature
- Machine: Thermostat system
- Machine Interface: Temperature sensor, heating control

### 2. Commanded Behavior Frame

Accept commands from operator and control accordingly.

**Example**: Elevator control
- Problem: Move elevator to requested floor
- Machine: Elevator controller
- Machine Interface: Floor buttons, motor control

### 3. Information Display Frame

Obtain information from world and present it in required form.

**Example**: Stock ticker
- Problem: Display current stock prices
- Machine: Stock ticker system
- Machine Interface: Stock data feed, display device

### 4. Simple Workpieces Frame

Allow users to create and edit text/graphic objects.

**Example**: Word processor
- Problem: Create and edit documents
- Machine: Word processor software
- Machine Interface: Keyboard input, display output

### 5. Transformation Frame

Transform input data to produce required output according to rules.

**Example**: Data conversion tool
- Problem: Convert CSV to JSON
- Machine: Conversion software
- Machine Interface: File input, file output

## Problem Frames Process

### Step 1: Identify Problem Domains

Map out all parts of the real world involved in the problem.

**Example: E-commerce System**
```
Problem Domains:
- Customers (people who buy)
- Products (items for sale)
- Orders (purchases)
- Inventory (stock management)
- Payments (financial transactions)
- Shipping (delivery)
- Reviews (customer feedback)
```

### Step 2: Identify Phenomena

List all individuals, events, states in each domain.

**Example: Orders Domain**
```
Individuals:
- Order
- OrderItem
- OrderStatus

Events:
- Order created
- Item added to order
- Order placed
- Order paid
- Order shipped
- Order delivered
- Order cancelled

States:
- Order is pending
- Order is paid
- Order is shipped
- Order is cancelled
```

### Step 3: Find Shared Phenomena

Identify phenomena that exist in multiple domains.

**Example**
```
Customer ──────────────────┬────────────────────> Product
         │                         │
         └── Order ──────────────┴────────────────────> Inventory

Shared Phenomena:
- Customer: Exists in Customer and Orders domains
- Product: Exists in Products, Orders, and Inventory domains
- Order: Exists in Customer and Product domains
```

### Step 4: Create Context Diagram

Show problem domains and their relationships.

**Example: Shopping Cart System**
```
Customers  ←→  Products  ←→  Orders
   ↓              ↓              ↓
              Shopping Cart
   ↓              ↓              ↓
  Payments   ←→  Inventory  ←→  Shipping
```

### Step 5: Identify Problem Frame

Determine which recognized frame applies.

**Example: Shopping Cart System**
- **Simple Workpieces**: Allow users to add/remove items to cart
- **Transformation**: Calculate totals, apply discounts
- **Commanded Behavior**: Process checkout command
- **Information Display**: Show cart contents and totals

### Step 6: Create Problem Diagram

Show requirements and how they reference phenomena.

**Example: Display Shopping Cart**
```
Requirement:
  WHEN customer views cart
  THEN show all items with quantities and prices

Phenomena References:
  Customer (who is viewing)
  Cart (what is being viewed)
  CartItem (items in cart)
  CartItem.quantity (how many)
  CartItem.price (price each)
  Cart.total (sum of all items)

Specification Phenomena:
  CartService.displayCart(customerId)
  CartItemPresenter.present(cartItems)
```

## Nexus Application

### Before Generating Code

The Domain Architect Agent applies Problem Frames approach:

1. **Frame the problem**: Understand what real-world problem we're solving
2. **Identify problem domains**: Which parts of the real world are involved?
3. **Map phenomena**: What events, states, relationships exist?
4. **Find shared phenomena**: What do domains have in common?
5. **Create context diagram**: Visualize domains and their connections
6. **Identify the problem frame**: Which recognized frame applies?
7. **Only then**: Design the software solution (machine) to fit into this context

### Example: Blog with Posts and Comments

**Problem Frames Analysis**:

```
Problem Domains:
  - Blog Users (people who write/read posts)
  - Posts (content being created)
  - Comments (feedback on posts)
  - Categories (organizing posts)
  - Tags (labeling posts)

Shared Phenomena:
  - User: Exists in Blog Users, Posts (as author), Comments (as author)
  - Post: Exists in Posts, Comments (what comment is about)
  - Category: Exists in Categories, Posts (categorization)
  - Tag: Exists in Posts (labels)

Context Diagram:
  Blog Users ←→ Posts ←→ Comments
       ↓         ↓
    Categories   Tags

Problem Frames:
  - Simple Workpieces: Allow users to create/edit blog posts
  - Information Display: Display posts and comments to readers
  - Commanded Behavior: User publishes post, reader comments on post

Machine Interface:
  - BlogService API (posts CRUD)
  - CommentService API (comments CRUD)
  - UserService API (user management)
```

## Problem Frames in Practice

### Example 1: User Authentication

**Problem Domains**:
- Users (people wanting to access)
- Accounts (user credentials)
- Sessions (authentication state)

**Phenomena**:
- Events: User registers, User logs in, User logs out
- States: Account exists, Session active, Session expired

**Context Diagram**:
```
Users ←→ Accounts ←→ Sessions
```

**Problem Frame**:
- **Commanded Behavior**: User provides credentials, system grants access

**Machine Interface**:
- AuthenticationService.login(email, password)
- AuthenticationService.logout(sessionId)

### Example 2: Order Processing

**Problem Domains**:
- Customers (people buying)
- Products (items for sale)
- Orders (purchases)
- Inventory (stock management)
- Payments (financial transactions)
- Shipping (delivery)

**Phenomena**:
- Events: Order created, Payment processed, Order shipped
- States: Order pending, Order paid, Order shipped

**Context Diagram**:
```
Customers ←→ Products
    ↓          ↓
   Orders ←→ Inventory
    ↓          ↓
 Payments  ←→ Shipping
```

**Problem Frames**:
- **Simple Workpieces**: Allow customers to add/remove items to order
- **Commanded Behavior**: Customer places order, system processes payment
- **Transformation**: Calculate order total, apply discounts

**Machine Interface**:
- OrderService.createOrder(customerId, items)
- PaymentService.processPayment(orderId, paymentMethod)
- ShippingService.shipOrder(orderId, address)

## Best Practices

### ✅ DO

- Study the problem world first, before designing the machine
- Identify all problem domains involved
- Map phenomena in each domain
- Find shared phenomena between domains
- Create context diagrams to visualize relationships
- Identify the appropriate problem frame
- Design the machine to fit the problem context

### ❌ DON'T

- Start coding without understanding the problem
- Focus only on the machine (software)
- Ignore the problem world context
- Assume you understand the problem without study
- Skip creating context diagrams
- Apply technical solutions to business problems

## Problem Frames vs. Traditional Approaches

### Traditional Approach

```
1. Understand requirements
2. Design software architecture
3. Implement solution
4. Test against requirements
```

**Problem**: Solves technical problems, may miss business context

### Problem Frames Approach

```
1. Frame the problem (understand problem world)
2. Identify problem domains
3. Map phenomena
4. Find shared phenomena
5. Create context diagram
6. Identify problem frame
7. Design machine to fit context
8. Implement solution
9. Test against problem world
```

**Advantage**: Ensures software solves the right problem in the right context

## References

- [docs/architecture/ARCHITECTURE.md](../architecture/ARCHITECTURE.md) - System architecture
- [docs/engineering/METHODOLOGIES.md](../METHODOLOGIES.md) - All methodologies
- [docs/engineering/domain_driven/DOMAIN_DRIVEN_DESIGN.md](../domain_driven/DOMAIN_DRIVEN_DESIGN.md) - Domain-Driven Design
- [CODING_AGENTS.md](../../CODING_AGENTS.md) - AI agent development guidelines
