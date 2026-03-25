# Authentication

The API uses **JWT (JSON Web Tokens)** for authentication. Short-lived access tokens are paired with longer-lived refresh tokens to balance security with convenience.

## Token Lifetimes

| Token | Lifetime | Storage recommendation |
|-------|----------|------------------------|
| Access token | 15 minutes | In-memory (not localStorage) |
| Refresh token | 7 days | HttpOnly cookie |

## Authentication Flow

```mermaid
sequenceDiagram
    participant C  as Client
    participant AS as Auth Service
    participant R  as Redis
    participant DB as PostgreSQL

    Note over C,AS: Login
    C->>AS: POST /auth/login {email, password}
    AS->>DB: Lookup user by email
    DB-->>AS: user {id, passwordHash, role}
    AS->>AS: bcrypt.compare(password, hash)
    AS->>R: SETEX session:{userId} 3600 "active"
    AS-->>C: 200 {accessToken, refreshToken}

    Note over C,AS: Authenticated Request
    C->>AS: GET /api/resource\nAuthorization: Bearer accessToken
    AS->>AS: jwt.verify(token, SECRET)
    AS->>R: GET session:{userId}
    R-->>AS: "active"
    AS-->>C: 200 OK {data}

    Note over C,AS: Token Refresh
    C->>AS: POST /auth/refresh {refreshToken}
    AS->>AS: jwt.verify(refreshToken, REFRESH_SECRET)
    AS-->>C: 200 {accessToken}

    Note over C,AS: Logout
    C->>AS: POST /auth/logout
    AS->>R: DEL session:{userId}
    AS-->>C: 204 No Content
```

## Endpoints

### `POST /auth/register`

Create a new user account.

**Request body**

```json
{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "password": "Sup3rS3cr3t!"
}
```

**Response `201 Created`**

```json
{
  "data": {
    "userId": "d4f2c1b0-...",
    "accessToken": "eyJhbGci...",
    "refreshToken": "eyJhbGci..."
  }
}
```

---

### `POST /auth/login`

Authenticate an existing user.

**Request body**

```json
{
  "email": "jane@example.com",
  "password": "Sup3rS3cr3t!"
}
```

**Response `200 OK`**

```json
{
  "data": {
    "accessToken": "eyJhbGci...",
    "refreshToken": "eyJhbGci...",
    "expiresIn": 900
  }
}
```

---

### `POST /auth/refresh`

Obtain a new access token using a valid refresh token.

**Request body**

```json
{
  "refreshToken": "eyJhbGci..."
}
```

**Response `200 OK`**

```json
{
  "data": {
    "accessToken": "eyJhbGci...",
    "expiresIn": 900
  }
}
```

---

### `POST /auth/logout`

Revoke the current session.

**Headers:** `Authorization: Bearer <accessToken>`

**Response `204 No Content`**

---

## Role-based Access Control (RBAC)

```mermaid
graph TD
    Admin["Role: admin"]
    Manager["Role: manager"]
    User["Role: user"]
    Guest["Role: guest"]

    Admin -->|inherits| Manager
    Manager -->|inherits| User
    User -->|inherits| Guest

    Admin --- PA["• Delete any resource\n• Manage users\n• View audit log"]
    Manager --- PM["• Create / update resources\n• View all team data"]
    User --- PU["• Create / update own resources\n• View own data"]
    Guest --- PG["• Read-only public endpoints"]
```

## Password Rules

- Minimum 8 characters
- At least one uppercase letter
- At least one digit
- At least one special character (`!@#$%^&*`)

## Security Notes

- Passwords are hashed with **bcrypt** (cost factor 12).
- Access tokens are signed with **RS256** (asymmetric keys).
- Refresh tokens are stored as an **HttpOnly**, **Secure**, **SameSite=Strict** cookie.
- Failed login attempts are rate-limited: **5 attempts per 15 minutes** per IP.
