---
layout: default
title: Testing
parent: Development
nav_order: 2
---

# Testing Guide

This page describes the testing strategy, tooling, and conventions used across the project.

## Testing Pyramid

```mermaid
graph TD
    E2E["End-to-End Tests\n(Playwright)\n~20 tests"]
    INT["Integration Tests\n(Jest + Supertest)\n~150 tests"]
    UNIT["Unit Tests\n(Jest)\n~500 tests"]

    E2E --- INT --- UNIT

    style E2E  fill:#f28b82,color:#000
    style INT  fill:#fbbc04,color:#000
    style UNIT fill:#34a853,color:#fff
```

| Layer | Tool | Scope | Speed |
|-------|------|-------|-------|
| Unit | Jest | Single function / class in isolation | < 1 ms each |
| Integration | Jest + Supertest | Service + real DB (test schema) | ~50 ms each |
| E2E | Playwright | Full user journey through UI | ~5 s each |

## Running Tests

```bash
# All tests (unit + integration)
npm test

# Unit tests only
npm run test:unit

# Integration tests only
npm run test:integration

# E2E tests (requires running services)
npm run test:e2e

# Watch mode (unit only, great for TDD)
npm run test:watch

# Coverage report
npm run test:coverage
```

## Unit Test Example

```typescript
// services/core/src/services/__tests__/user.service.test.ts
import { UserService } from '../user.service';
import { UserRepository } from '../../repositories/user.repository';

jest.mock('../../repositories/user.repository');

describe('UserService', () => {
  let service: UserService;
  let repoMock: jest.Mocked<UserRepository>;

  beforeEach(() => {
    repoMock = new UserRepository() as jest.Mocked<UserRepository>;
    service  = new UserService(repoMock);
  });

  describe('findById', () => {
    it('returns the user when found', async () => {
      const mockUser = { id: '123', name: 'Jane', email: 'jane@example.com' };
      repoMock.findOne.mockResolvedValue(mockUser);

      const result = await service.findById('123');

      expect(repoMock.findOne).toHaveBeenCalledWith({ where: { id: '123' } });
      expect(result).toEqual(mockUser);
    });

    it('throws NotFoundException when user not found', async () => {
      repoMock.findOne.mockResolvedValue(null);

      await expect(service.findById('999')).rejects.toThrow('User not found');
    });
  });
});
```

## Integration Test Example

```typescript
// services/core/src/controllers/__tests__/user.controller.test.ts
import request from 'supertest';
import { app } from '../../app';
import { dataSource } from '../../database';

beforeAll(async () => {
  await dataSource.initialize();
  await dataSource.runMigrations();
});

afterAll(async () => {
  await dataSource.dropDatabase();
  await dataSource.destroy();
});

describe('GET /api/v1/users', () => {
  it('returns 401 when unauthenticated', async () => {
    const res = await request(app).get('/api/v1/users');
    expect(res.status).toBe(401);
  });

  it('returns paginated user list for admin', async () => {
    const token = await getAdminToken(); // test helper

    const res = await request(app)
      .get('/api/v1/users?page=1&limit=5')
      .set('Authorization', `Bearer ${token}`);

    expect(res.status).toBe(200);
    expect(res.body.data).toBeInstanceOf(Array);
    expect(res.body.meta).toMatchObject({ page: 1, limit: 5 });
  });
});
```

## Test Flow Diagram

```mermaid
flowchart TD
    Dev["Developer writes\nor modifies code"] --> UnitTest["Run unit tests\n(npm run test:unit)"]
    UnitTest --> Pass1{Pass?}
    Pass1 -- No --> Fix["Fix the code"]
    Fix --> UnitTest
    Pass1 -- Yes --> IntTest["Run integration tests\n(npm run test:integration)"]
    IntTest --> Pass2{Pass?}
    Pass2 -- No --> Fix
    Pass2 -- Yes --> PR["Open Pull Request"]
    PR --> CI["GitHub Actions CI\nruns full suite"]
    CI --> Pass3{Pass?}
    Pass3 -- No --> Fix
    Pass3 -- Yes --> Review["Code Review"]
    Review --> Merge["Merge to main"]
    Merge --> E2E["Nightly E2E run\n(Playwright)"]
```

## Coverage Requirements

| Layer | Minimum Coverage |
|-------|-----------------|
| Unit (lines) | 80% |
| Unit (branches) | 75% |
| Integration | Key happy-path + error cases covered |

Enforce coverage thresholds in `jest.config.ts`:

```typescript
// jest.config.ts
export default {
  coverageThreshold: {
    global: {
      lines:    80,
      branches: 75,
      functions: 80,
    },
  },
};
```

## Mocking Guidelines

```mermaid
flowchart TD
    Test["Test"] -->|"mock"| Ext["External services\n(SendGrid, Stripe, S3)"]
    Test -->|"real (test DB)"| DB["PostgreSQL\n(test schema, wiped after)"]
    Test -->|"real (test instance)"| Redis["Redis\n(test instance, FLUSHDB after)"]
    Test -->|"mock"| Clock["System clock\n(jest.useFakeTimers)"]
```

- **Always mock** third-party HTTP calls (use `nock` or `msw`).
- **Use a real test database** for integration tests – avoid mocking the data layer.
- **Use `jest.useFakeTimers()`** whenever time-based logic (expiry, retries) is tested.
