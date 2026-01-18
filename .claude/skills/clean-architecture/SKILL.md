---
name: architecture-patterns
description: Implement proven architecture patterns including Clean Architecture, Hexagonal Architecture, and Domain-Driven Design. Use when architecting complex backend systems or refactoring existing applications for better maintainability.
---

# Architecture Patterns

Master proven architecture patterns including Clean Architecture, Hexagonal Architecture, and Domain-Driven Design to build maintainable, testable, and scalable systems.

## When to Use This Skill

- Designing new backend systems from scratch
- Refactoring monolithic applications for better maintainability
- Establishing architecture standards for your team
- Migrating from tightly coupled to loosely coupled architectures
- Implementing domain-driven design principles
- Creating testable and mockable codebases
- Planning microservices decomposition

## Core Concepts

### 1. Clean Architecture (Uncle Bob)

**Layers (dependency flows inward):**

- **Entities**: Core business models
- **Use Cases**: Application business rules
- **Interface Adapters**: Controllers, presenters, gateways
- **Frameworks & Drivers**: UI, database, external services

**Key Principles:**

- Dependencies point inward
- Inner layers know nothing about outer layers
- Business logic independent of frameworks
- Testable without UI, database, or external services

### 2. Hexagonal Architecture (Ports and Adapters)

**Components:**

- **Domain Core**: Business logic
- **Ports**: Interfaces defining interactions
- **Adapters**: Implementations of ports (database, REST, message queue)

**Benefits:**

- Swap implementations easily (mock for testing)
- Technology-agnostic core
- Clear separation of concerns

### 3. Domain-Driven Design (DDD)

**Strategic Patterns:**

- **Bounded Contexts**: Separate models for different domains
- **Context Mapping**: How contexts relate
- **Ubiquitous Language**: Shared terminology

**Tactical Patterns:**

- **Entities**: Objects with identity
- **Value Objects**: Immutable objects defined by attributes
- **Aggregates**: Consistency boundaries
- **Repositories**: Data access abstraction
- **Domain Events**: Things that happened

## Clean Architecture Pattern

### Directory Structure

```
app/
├── domain/           # Entities & business rules
│   ├── entities/
│   │   ├── user.ts
│   │   └── order.ts
│   ├── value_objects/
│   │   ├── email.ts
│   │   └── money.ts
│   └── interfaces/   # Abstract interfaces
│       ├── user_repository.ts
│       └── payment_gateway.ts
├── use_cases/        # Application business rules
│   ├── create_user.ts
│   ├── process_order.ts
│   └── send_notification.ts
├── adapters/         # Interface implementations
│   ├── repositories/
│   │   ├── postgres_user_repository.ts
│   │   └── redis_cache_repository.ts
│   ├── controllers/
│   │   └── user_controller.ts
│   └── gateways/
│       ├── stripe_payment_gateway.ts
│       └── sendgrid_email_gateway.ts
└── infrastructure/   # Framework & external concerns
    ├── database.ts
    ├── config.ts
    └── logging.ts
```

### Implementation Example

```typescript
// domain/entities/user.ts
export class User {
    /** Core user entity - no framework dependencies. */
    constructor(
        public id: string,
        public email: string,
        public name: string,
        public created_at: Date,
        public is_active: boolean = true
    ) {}

    /** Business rule: deactivating user. */
    deactivate(): void {
        this.is_active = false;
    }

    /** Business rule: active users can order. */
    can_place_order(): boolean {
        return this.is_active;
    }
}

// domain/interfaces/user_repository.ts
import { User } from '../entities/user';

export interface IUserRepository {
    /** Port: defines contract, no implementation. */
    find_by_id(user_id: string): Promise<User | null>;
    find_by_email(email: string): Promise<User | null>;
    save(user: User): Promise<User>;
    delete(user_id: string): Promise<boolean>;
}

// use_cases/create_user.ts
import { User } from '../domain/entities/user';
import { IUserRepository } from '../domain/interfaces/user_repository';
import { v4 as uuidv4 } from 'uuid';

export interface CreateUserRequest {
    email: string;
    name: string;
}

export interface CreateUserResponse {
    user: User | null;
    success: boolean;
    error?: string;
}

export class CreateUserUseCase {
    /** Use case: orchestrates business logic. */

    constructor(private user_repository: IUserRepository) {}

    async execute(request: CreateUserRequest): Promise<CreateUserResponse> {
        // Business validation
        const existing = await this.user_repository.find_by_email(request.email);
        if (existing) {
            return {
                user: null,
                success: false,
                error: "Email already exists"
            };
        }

        // Create entity
        const user = new User(
            uuidv4(),
            request.email,
            request.name,
            new Date(),
            true
        );

        // Persist
        const saved_user = await this.user_repository.save(user);

        return {
            user: saved_user,
            success: true
        };
    }
}

// adapters/repositories/postgres_user_repository.ts
import { IUserRepository } from '../../domain/interfaces/user_repository';
import { User } from '../../domain/entities/user';
import { Pool, PoolClient } from 'pg';

export class PostgresUserRepository implements IUserRepository {
    /** Adapter: PostgreSQL implementation. */

    constructor(private pool: Pool) {}

    async find_by_id(user_id: string): Promise<User | null> {
        const client = await this.pool.connect();
        try {
            const result = await client.query(
                'SELECT * FROM users WHERE id = $1',
                [user_id]
            );
            return result.rows.length > 0 ? this._to_entity(result.rows[0]) : null;
        } finally {
            client.release();
        }
    }

    async find_by_email(email: string): Promise<User | null> {
        const client = await this.pool.connect();
        try {
            const result = await client.query(
                'SELECT * FROM users WHERE email = $1',
                [email]
            );
            return result.rows.length > 0 ? this._to_entity(result.rows[0]) : null;
        } finally {
            client.release();
        }
    }

    async save(user: User): Promise<User> {
        const client = await this.pool.connect();
        try {
            await client.query(
                `
                INSERT INTO users (id, email, name, created_at, is_active)
                VALUES ($1, $2, $3, $4, $5)
                ON CONFLICT (id) DO UPDATE
                SET email = $2, name = $3, is_active = $5
                `,
                [user.id, user.email, user.name, user.created_at, user.is_active]
            );
            return user;
        } finally {
            client.release();
        }
    }

    async delete(user_id: string): Promise<boolean> {
        const client = await this.pool.connect();
        try {
            const result = await client.query(
                'DELETE FROM users WHERE id = $1',
                [user_id]
            );
            return result.rowCount === 1;
        } finally {
            client.release();
        }
    }

    private _to_entity(row: any): User {
        /** Map database row to entity. */
        return new User(
            row.id,
            row.email,
            row.name,
            new Date(row.created_at),
            row.is_active
        );
    }
}

// adapters/controllers/user_controller.ts
import { Router, Request, Response } from 'express';
import { CreateUserUseCase, CreateUserRequest } from '../../use_cases/create_user';

const router = Router();

interface CreateUserDTO {
    email: string;
    name: string;
}

// Dependency injection would typically be handled by a DI container
let createUserUseCase: CreateUserUseCase;

export function setCreateUserUseCase(useCase: CreateUserUseCase) {
    createUserUseCase = useCase;
}

router.post('/users', async (req: Request, res: Response) => {
    /** Controller: handles HTTP concerns only. */
    const dto: CreateUserDTO = req.body;

    const request: CreateUserRequest = {
        email: dto.email,
        name: dto.name
    };

    const response = await createUserUseCase.execute(request);

    if (!response.success) {
        return res.status(400).json({ error: response.error });
    }

    return res.status(201).json({ user: response.user });
});

export default router;
```

## Hexagonal Architecture Pattern

```typescript
// Core domain (hexagon center)
export class OrderService {
    /** Domain service - no infrastructure dependencies. */

    constructor(
        private orders: OrderRepositoryPort,
        private payments: PaymentGatewayPort,
        private notifications: NotificationPort
    ) {}

    async place_order(order: Order): Promise<OrderResult> {
        // Business logic
        if (!order.is_valid()) {
            return { success: false, error: "Invalid order" };
        }

        // Use ports (interfaces)
        const payment = await this.payments.charge(
            order.total,
            order.customer_id
        );

        if (!payment.success) {
            return { success: false, error: "Payment failed" };
        }

        order.mark_as_paid();
        const saved_order = await this.orders.save(order);

        await this.notifications.send(
            order.customer_email,
            "Order confirmed",
            `Order ${order.id} confirmed`
        );

        return { success: true, order: saved_order };
    }
}

// Ports (interfaces)
export interface OrderRepositoryPort {
    save(order: Order): Promise<Order>;
}

export interface PaymentGatewayPort {
    charge(amount: Money, customer: string): Promise<PaymentResult>;
}

export interface NotificationPort {
    send(to: string, subject: string, body: string): Promise<void>;
}

// Adapters (implementations)
export class StripePaymentAdapter implements PaymentGatewayPort {
    /** Primary adapter: connects to Stripe API. */

    constructor(private api_key: string) {
        // Initialize Stripe
    }

    async charge(amount: Money, customer: string): Promise<PaymentResult> {
        try {
            // Stripe API call would go here
            const charge = { id: 'stripe-charge-123' }; // Mock response
            return { success: true, transaction_id: charge.id };
        } catch (error) {
            return { success: false, error: error.message };
        }
    }
}

export class MockPaymentAdapter implements PaymentGatewayPort {
    /** Test adapter: no external dependencies. */

    async charge(amount: Money, customer: string): Promise<PaymentResult> {
        return { success: true, transaction_id: "mock-123" };
    }
}
```

## Domain-Driven Design Pattern

```typescript
// Value Objects (immutable)
export class Email {
    /** Value object: validated email. */
    constructor(public readonly value: string) {
        if (!this.value.includes('@')) {
            throw new Error('Invalid email');
        }
    }
}

export class Money {
    /** Value object: amount with currency. */
    constructor(
        public readonly amount: number, // cents
        public readonly currency: string
    ) {}

    add(other: Money): Money {
        if (this.currency !== other.currency) {
            throw new Error('Currency mismatch');
        }
        return new Money(this.amount + other.amount, this.currency);
    }
}

// Entities (with identity)
export class Order {
    /** Entity: has identity, mutable state. */
    public items: OrderItem[] = [];
    public status: OrderStatus = OrderStatus.PENDING;
    private _events: DomainEvent[] = [];

    constructor(
        public id: string,
        public customer: Customer
    ) {}

    add_item(product: Product, quantity: number): void {
        /** Business logic in entity. */
        const item = new OrderItem(product, quantity);
        this.items.push(item);
        this._events.push(new ItemAddedEvent(this.id, item));
    }

    total(): Money {
        /** Calculated property. */
        return this.items.reduce(
            (sum, item) => sum.add(item.subtotal()),
            new Money(0, 'USD')
        );
    }

    submit(): void {
        /** State transition with business rules. */
        if (this.items.length === 0) {
            throw new Error('Cannot submit empty order');
        }
        if (this.status !== OrderStatus.PENDING) {
            throw new Error('Order already submitted');
        }

        this.status = OrderStatus.SUBMITTED;
        this._events.push(new OrderSubmittedEvent(this.id));
    }
}

// Aggregates (consistency boundary)
export class Customer {
    /** Aggregate root: controls access to entities. */
    private _addresses: Address[] = [];
    private _orders: string[] = []; // Order IDs, not full objects

    constructor(
        public id: string,
        public email: Email
    ) {}

    add_address(address: Address): void {
        /** Aggregate enforces invariants. */
        if (this._addresses.length >= 5) {
            throw new Error('Maximum 5 addresses allowed');
        }
        this._addresses.push(address);
    }

    get primary_address(): Address | undefined {
        return this._addresses.find(a => a.is_primary);
    }
}

// Domain Events
export abstract class DomainEvent {
    constructor(public readonly occurred_at: Date = new Date()) {}
}

export class OrderSubmittedEvent extends DomainEvent {
    constructor(public readonly order_id: string) {
        super();
    }
}

// Repository (aggregate persistence)
export class OrderRepository {
    /** Repository: persist/retrieve aggregates. */

    async find_by_id(order_id: string): Promise<Order | null> {
        /** Reconstitute aggregate from storage. */
        // Implementation would go here
        return null;
    }

    async save(order: Order): Promise<void> {
        /** Persist aggregate and publish events. */
        await this._persist(order);
        await this._publish_events(order._events);
        order._events.length = 0; // Clear events
    }

    private async _persist(order: Order): Promise<void> {
        // Persistence logic
    }

    private async _publish_events(events: DomainEvent[]): Promise<void> {
        // Event publishing logic
    }
}
```


## Best Practices

1. **Dependency Rule**: Dependencies always point inward
2. **Interface Segregation**: Small, focused interfaces
3. **Business Logic in Domain**: Keep frameworks out of core
4. **Test Independence**: Core testable without infrastructure
5. **Bounded Contexts**: Clear domain boundaries
6. **Ubiquitous Language**: Consistent terminology
7. **Thin Controllers**: Delegate to use cases
8. **Rich Domain Models**: Behavior with data

## Common Pitfalls

- **Anemic Domain**: Entities with only data, no behavior
- **Framework Coupling**: Business logic depends on frameworks
- **Fat Controllers**: Business logic in controllers
- **Repository Leakage**: Exposing ORM objects
- **Missing Abstractions**: Concrete dependencies in core
- **Over-Engineering**: Clean architecture for simple CRUD
