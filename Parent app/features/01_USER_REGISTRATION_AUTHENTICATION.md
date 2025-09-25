# Feature 01: User Registration & Authentication

## Overview
This feature handles user registration, login, password management, and session handling for the SafeGuard Parent App. It provides secure authentication mechanisms and user account management.

## Table of Contents
1. [Feature Requirements](#feature-requirements)
2. [System Architecture](#system-architecture)
3. [Sequence Diagrams](#sequence-diagrams)
4. [Activity Diagrams](#activity-diagrams)
5. [State Diagrams](#state-diagrams)
6. [API Specifications](#api-specifications)
7. [Database Design](#database-design)
8. [Frontend Implementation (Flutter)](#frontend-implementation-flutter)
9. [Backend Implementation (Spring Boot)](#backend-implementation-spring-boot)
10. [Security Considerations](#security-considerations)
11. [Testing Strategy](#testing-strategy)

---

## Feature Requirements

### Functional Requirements
- **FR-01**: Users can register with email or phone number
- **FR-02**: Users can login with email/phone and password
- **FR-03**: Users can reset forgotten passwords via email
- **FR-04**: Users can change their passwords when authenticated
- **FR-05**: Users can verify their email addresses
- **FR-06**: Users can logout and invalidate sessions

- **FR-08**: System validates password strength
- **FR-09**: System prevents duplicate account creation
- **FR-10**: System supports social login (Google, Apple)

### Non-Functional Requirements
- **NFR-01**: Authentication response time < 2 seconds
- **NFR-02**: Password hashing using BCrypt
- **NFR-03**: JWT tokens with 24-hour expiration
- **NFR-04**: Rate limiting: 5 login attempts per minute
- **NFR-05**: Password reset tokens expire in 15 minutes
- **NFR-06**: System supports 10,000+ concurrent users
- **NFR-07**: 99.9% uptime for authentication services

---

## System Architecture

### High-Level Architecture
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Flutter App   │    │  Spring Boot    │    │   PostgreSQL    │
│   (Frontend)    │◄──►│   (Backend)     │◄──►│   (Database)    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  SharedPrefs    │    │   Redis Cache   │    │   SMTP Server   │
│  (Local Storage)│    │   (Sessions)    │    │   (Email)       │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Component Diagram
```mermaid
graph TB
    subgraph "Flutter Frontend"
        A[Login Screen]
        B[Register Screen]
        C[Forgot Password Screen]
        D[Reset Password Screen]
        E[Auth Provider]
        F[API Client]
    end
    
    subgraph "Spring Boot Backend"
        G[Auth Controller]
        H[Auth Service]
        I[JWT Service]
        J[Password Service]
        K[Email Service]
        L[User Repository]
    end
    
    subgraph "Database Layer"
        M[PostgreSQL]
        N[Redis Cache]
    end
    
    A --> E
    B --> E
    C --> E
    D --> E
    E --> F
    F --> G
    
    G --> H
    H --> I
    H --> J
    H --> K
    H --> L
    
    L --> M
    H --> N
    K --> O[SMTP Server]
```

---

## Sequence Diagrams

### User Registration Flow
```mermaid
sequenceDiagram
    participant U as User
    participant F as Flutter App
    participant S as Spring Boot API
    participant D as PostgreSQL
    participant E as Email Service
    participant R as Redis Cache

    U->>F: Open Registration Screen
    F->>U: Display Registration Form
    
    U->>F: Enter Registration Details
    F->>F: Validate Form Data
    F->>F: Check Password Strength
    
    U->>F: Submit Registration
    F->>S: POST /api/auth/register
    Note over F,S: {fullName, email, password, termsAccepted}
    
    S->>S: Validate Input Data
    S->>D: Check if User Exists
    D-->>S: User Status
    
    alt User Already Exists
        S-->>F: 409 Conflict
        F-->>U: Show Error Message
    else User Does Not Exist
        S->>S: Hash Password (BCrypt)
        S->>D: Create User Record
        D-->>S: User Created
        
        S->>E: Send Welcome Email
        E-->>S: Email Sent
        
        S->>R: Store User Session
        R-->>S: Session Stored
        
        S-->>F: 201 Created + JWT Token
        F->>F: Store JWT Token
        F->>F: Navigate to Dashboard
        F-->>U: Show Success Message
    end
```

### User Login Flow
```mermaid
sequenceDiagram
    participant U as User
    participant F as Flutter App
    participant S as Spring Boot API
    participant D as PostgreSQL
    participant R as Redis Cache

    U->>F: Open Login Screen
    F->>U: Display Login Form
    
    U->>F: Enter Credentials
    F->>F: Validate Form Data
    
    U->>F: Click Sign In
    F->>S: POST /api/auth/login
    Note over F,S: {email, password, rememberMe}
    
    S->>S: Validate Input Data
    S->>D: Find User by Email
    D-->>S: User Data
    
    alt User Not Found
        S-->>F: 404 Not Found
        F-->>U: Show Invalid Credentials Error
    else User Found
        S->>S: Verify Password (BCrypt)
        
        alt Password Incorrect
            S-->>F: 401 Unauthorized
            F-->>U: Show Invalid Credentials Error
        else Password Correct
            S->>S: Generate JWT Token
            S->>R: Store Session
            R-->>S: Session Stored
            
            S-->>F: 200 OK + JWT Token + User Data
            F->>F: Store JWT Token
            F->>F: Navigate to Dashboard
            F-->>U: Show Welcome Message
        end
    end
```

### Forgot Password Flow
```mermaid
sequenceDiagram
    participant U as User
    participant F as Flutter App
    participant S as Spring Boot API
    participant D as PostgreSQL
    participant E as Email Service
    participant R as Redis Cache

    U->>F: Click "Forgot Password?"
    F->>U: Show Forgot Password Form
    
    U->>F: Enter Email Address
    U->>F: Click "Send Reset Link"
    
    F->>S: POST /api/auth/forgot-password
    Note over F,S: {email}
    
    S->>S: Validate Email Format
    S->>D: Find User by Email
    D-->>S: User Data
    
    alt User Not Found
        S-->>F: 404 Not Found
        F-->>U: Show "Email Not Found" Error
    else User Found
        S->>S: Generate Reset Token
        S->>R: Store Reset Token (15 min expiry)
        R-->>S: Token Stored
        
        S->>E: Send Password Reset Email
        E-->>S: Email Sent
        
        S-->>F: 200 OK + Success Message
        F-->>U: Show "Reset Link Sent" Message
        
        Note over U,F: User checks email and clicks reset link
        
        U->>F: Click Reset Link from Email
        F->>S: GET /api/auth/reset-password?token=xxx
        S->>R: Validate Reset Token
        R-->>S: Token Status
        
        alt Token Invalid/Expired
            S-->>F: 400 Bad Request
            F-->>U: Show "Invalid/Expired Link" Error
        else Token Valid
            F->>U: Show Reset Password Form
            U->>F: Enter New Password
            U->>F: Click "Reset Password"
            
            F->>S: POST /api/auth/reset-password
            Note over F,S: {token, newPassword}
            
            S->>S: Validate New Password
            S->>S: Hash New Password (BCrypt)
            S->>D: Update User Password
            D-->>S: Password Updated
            
            S->>R: Invalidate Reset Token
            R-->>S: Token Invalidated
            
            S-->>F: 200 OK + Success Message
            F->>F: Navigate to Login
            F-->>U: Show "Password Reset Successfully" Message
        end
    end
```

---

## Activity Diagrams

### Registration Process
```mermaid
flowchart TD
    A[Start Registration] --> B[Open Registration Form]
    B --> C[Enter Full Name]
    C --> D[Select Contact Type]
    D --> E{Email or Phone?}
    E -->|Email| F[Enter Email Address]
    E -->|Phone| G[Enter Phone Number]
    F --> H[Enter Password]
    G --> H
    H --> I[Confirm Password]
    I --> J[Accept Terms & Conditions]
    J --> K[Validate Form Data]
    K --> L{Validation Passed?}
    L -->|No| M[Show Validation Errors]
    M --> C
    L -->|Yes| N[Check Password Strength]
    N --> O{Password Strong Enough?}
    O -->|No| P[Show Password Requirements]
    P --> H
    O -->|Yes| Q[Submit Registration]
    Q --> R[Call Registration API]
    R --> S{User Already Exists?}
    S -->|Yes| T[Show User Exists Error]
    T --> C
    S -->|No| U[Create User Account]
    U --> V[Send Welcome Email]
    V --> W[Store User Session]
    W --> X[Generate JWT Token]
    X --> Y[Navigate to Dashboard]
    Y --> Z[End - Registration Complete]
```

### Login Process
```mermaid
flowchart TD
    A[Start Login] --> B[Open Login Form]
    B --> C[Select Login Type]
    C --> D{Email or Phone?}
    D -->|Email| E[Enter Email Address]
    D -->|Phone| F[Enter Phone Number]
    E --> G[Enter Password]
    F --> G
    G --> H[Toggle Remember Me]
    H --> I[Click Sign In]
    I --> J[Validate Form Data]
    J --> K{Validation Passed?}
    K -->|No| L[Show Validation Errors]
    L --> C
    K -->|Yes| M[Call Login API]
    M --> N[Find User in Database]
    N --> O{User Found?}
    O -->|No| P[Show Invalid Credentials]
    P --> C
    O -->|Yes| Q[Verify Password]
    Q --> R{Password Correct?}
    R -->|No| S[Show Invalid Credentials]
    S --> C
    R -->|Yes| T[Generate JWT Token]
    T --> U{Remember Me?}
    U -->|Yes| V[Store Long-term Session]
    U -->|No| W[Store Short-term Session]
    V --> X[Navigate to Dashboard]
    W --> X
    X --> Y[End - Login Complete]
```

---

## API Specifications

### Endpoints Table
| Method | Endpoint | Description | Request Body | Response | Status Codes | Auth Required |
|--------|----------|-------------|--------------|----------|--------------|---------------|
| POST | `/api/auth/register` | Register new user | `RegisterRequest` | `AuthResponse` | 201, 400, 409 | No |
| POST | `/api/auth/login` | User login | `LoginRequest` | `AuthResponse` | 200, 401, 400 | No |
| POST | `/api/auth/forgot-password` | Send reset email | `ForgotPasswordRequest` | `MessageResponse` | 200, 404, 400 | No |
| POST | `/api/auth/reset-password` | Reset password | `ResetPasswordRequest` | `MessageResponse` | 200, 400, 401 | No |
| POST | `/api/auth/refresh` | Refresh JWT token | `RefreshTokenRequest` | `TokenResponse` | 200, 401 | No |
| POST | `/api/auth/logout` | User logout | `LogoutRequest` | `MessageResponse` | 200, 401 | Yes |
| GET | `/api/auth/verify-email` | Verify email | Query: `token` | `MessageResponse` | 200, 400 | No |
| POST | `/api/auth/resend-verification` | Resend verification | `ResendVerificationRequest` | `MessageResponse` | 200, 400 | No |
| GET | `/api/auth/profile` | Get user profile | None | `UserProfileResponse` | 200, 401 | Yes |
| PUT | `/api/auth/profile` | Update profile | `UpdateProfileRequest` | `UserProfileResponse` | 200, 400, 401 | Yes |
| POST | `/api/auth/change-password` | Change password | `ChangePasswordRequest` | `MessageResponse` | 200, 400, 401 | Yes |

### Request/Response Models

#### RegisterRequest
```json
{
  "fullName": "string",
  "email": "string",
  "phone": "string",
  "password": "string",
  "contactType": "EMAIL | PHONE",
  "termsAccepted": boolean
}
```

#### LoginRequest
```json
{
  "email": "string",
  "phone": "string",
  "password": "string",
  "loginType": "EMAIL | PHONE",
  "rememberMe": boolean
}
```

#### AuthResponse
```json
{
  "success": boolean,
  "message": "string",
  "data": {
    "user": {
      "id": "string",
      "fullName": "string",
      "email": "string",
      "phone": "string",
      "isActive": boolean,
      "emailVerified": boolean,
      "phoneVerified": boolean,
      "createdAt": "datetime",
      "lastLoginAt": "datetime"
    },
    "token": "string",
    "refreshToken": "string",
    "expiresIn": number,
    "tokenType": "Bearer"
  }
}
```

---

## Database Design

### User Table
```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone VARCHAR(20) UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    contact_type VARCHAR(10) NOT NULL CHECK (contact_type IN ('EMAIL', 'PHONE')),
    is_active BOOLEAN NOT NULL DEFAULT true,
    email_verified BOOLEAN NOT NULL DEFAULT false,
    phone_verified BOOLEAN NOT NULL DEFAULT false,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    last_login_at TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_phone ON users(phone);
CREATE INDEX idx_users_active ON users(is_active);
```

### User Sessions Table
```sql
CREATE TABLE user_sessions (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    session_token VARCHAR(255) UNIQUE NOT NULL,
    refresh_token VARCHAR(255) UNIQUE NOT NULL,
    device_info TEXT,
    ip_address INET,
    is_active BOOLEAN NOT NULL DEFAULT true,
    expires_at TIMESTAMP NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    last_accessed_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_user_sessions_user_id ON user_sessions(user_id);
CREATE INDEX idx_user_sessions_token ON user_sessions(session_token);
CREATE INDEX idx_user_sessions_active ON user_sessions(is_active);
```

### Password Reset Tokens Table
```sql
CREATE TABLE password_reset_tokens (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token VARCHAR(255) UNIQUE NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    used BOOLEAN NOT NULL DEFAULT false,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_password_reset_tokens_token ON password_reset_tokens(token);
CREATE INDEX idx_password_reset_tokens_user_id ON password_reset_tokens(user_id);
CREATE INDEX idx_password_reset_tokens_expires ON password_reset_tokens(expires_at);
```

---

## Frontend Implementation (Flutter)

### Project Structure
```
lib/features/authentication/
├── data/
│   ├── datasources/
│   │   ├── auth_local_datasource.dart
│   │   └── auth_remote_datasource.dart
│   ├── models/
│   │   ├── user_model.dart
│   │   ├── auth_response_model.dart
│   │   └── login_request_model.dart
│   └── repositories/
│       └── auth_repository_impl.dart
├── domain/
│   ├── entities/
│   │   ├── user.dart
│   │   └── auth_response.dart
│   ├── repositories/
│   │   └── auth_repository.dart
│   └── usecases/
│       ├── login_usecase.dart
│       ├── register_usecase.dart
│       ├── forgot_password_usecase.dart
│       └── reset_password_usecase.dart
└── presentation/
    ├── pages/
    │   ├── login_page.dart
    │   ├── register_page.dart
    │   ├── forgot_password_page.dart
    │   └── reset_password_page.dart
    ├── widgets/
    │   ├── custom_text_field.dart
    │   ├── password_field.dart
    │   └── loading_button.dart
    └── providers/
        └── auth_provider.dart
```

### Key Dependencies
```yaml
dependencies:
  # State Management
  provider: ^6.0.5
  flutter_riverpod: ^2.4.0
  
  # HTTP Client
  dio: ^5.3.2
  retrofit: ^4.0.3
  
  # Local Storage
  shared_preferences: ^2.2.2
  secure_storage: ^9.0.0
  
  # Form Validation
  form_builder_validators: ^9.1.0
  flutter_form_builder: ^9.1.1
  
  # UI Components
  flutter_screenutil: ^5.9.0
  google_fonts: ^6.1.0
```


## Security Considerations

### Password Security
- **BCrypt Hashing**: Passwords are hashed using BCrypt with salt
- **Password Strength**: Minimum 8 characters, mixed case, numbers, special characters
- **Password History**: Prevent reuse of last 5 passwords
- **Account Lockout**: Lock account after 5 failed login attempts

### JWT Security
- **Secret Key**: Strong secret key (256-bit minimum)
- **Token Expiration**: 24 hours for access tokens, 7 days for refresh tokens
- **Token Rotation**: Refresh tokens are rotated on each use
- **Secure Storage**: Tokens stored securely on client side

### API Security
- **Rate Limiting**: 5 requests per minute per IP for auth endpoints
- **CORS Configuration**: Restricted to allowed origins
- **Input Validation**: All inputs validated and sanitized
- **SQL Injection Prevention**: Parameterized queries only

### Session Management
- **Session Invalidation**: Sessions invalidated on logout
- **Concurrent Sessions**: Limit to 5 active sessions per user
- **Session Timeout**: Inactive sessions expire after 30 minutes
- **Device Tracking**: Track and manage sessions by device

---

## Testing Strategy

### Unit Tests
- **Service Layer**: Test all business logic
- **Repository Layer**: Test data access methods
- **Utility Classes**: Test password hashing, JWT operations
- **Validation**: Test input validation logic

### Integration Tests
- **API Endpoints**: Test complete request/response cycles
- **Database Operations**: Test CRUD operations
- **Email Service**: Test email sending functionality
- **Cache Operations**: Test Redis integration

### Security Tests
- **Authentication**: Test login/logout flows
- **Authorization**: Test protected endpoints
- **Password Security**: Test password hashing and validation
- **JWT Security**: Test token generation and validation

### Performance Tests
- **Load Testing**: Test with 1000+ concurrent users
- **Response Time**: Ensure < 2 second response times
- **Database Performance**: Test query performance
- **Memory Usage**: Monitor memory consumption

---

## Implementation Checklist

### Frontend (Flutter)
- [ ] Set up project structure with clean architecture
- [ ] Implement authentication screens (Login, Register, Forgot Password)
- [ ] Create form validation and error handling
- [ ] Implement state management with Riverpod
- [ ] Set up API client with Dio and Retrofit
- [ ] Implement local storage for tokens
- [ ] Add loading states and user feedback
- [ ] Implement navigation between screens
- [ ] Add password strength indicator
- [ ] Implement social login (Google, Apple)

### Backend (Spring Boot)
- [ ] Set up Spring Boot project with required dependencies
- [ ] Configure database connection and JPA entities
- [ ] Implement JWT service for token management
- [ ] Create authentication service with business logic
- [ ] Implement password service with BCrypt
- [ ] Set up email service for notifications
- [ ] Configure Spring Security
- [ ] Implement rate limiting
- [ ] Add input validation and error handling
- [ ] Set up Redis for session management

### Database
- [ ] Create user table with proper indexes
- [ ] Create user sessions table
- [ ] Create password reset tokens table
- [ ] Set up database migrations
- [ ] Configure connection pooling
- [ ] Set up database monitoring

### Security
- [ ] Implement password hashing with BCrypt
- [ ] Set up JWT token management
- [ ] Configure CORS properly
- [ ] Implement rate limiting
- [ ] Add input validation
- [ ] Set up secure headers
- [ ] Implement session management
- [ ] Add audit logging

### Testing
- [ ] Write unit tests for all services
- [ ] Create integration tests for API endpoints
- [ ] Implement security tests
- [ ] Add performance tests
- [ ] Set up test data and fixtures
- [ ] Configure test environment
