# Authentication System Documentation

## Overview
Taiga implements a token-based authentication system with support for both public and private registration flows. The system uses JWT (JSON Web Tokens) for authentication and includes rate limiting to prevent abuse.

## Authentication Flow Sequence Diagram

```plantuml
@startuml
actor Client
participant "Auth API" as API
participant "Auth Service" as Service
database "Database" as DB

== Login Flow ==
Client -> API: POST /api/v1/auth\n{type: "normal", username, password}
activate API
API -> Service: get_token(data)
activate Service
Service -> DB: validate_credentials()
DB --> Service: user_data
Service -> Service: generate_jwt_token()
Service --> API: {access_token, refresh_token}
API --> Client: 200 OK + tokens
deactivate API
deactivate Service

== Token Refresh ==
Client -> API: POST /api/v1/auth/refresh\n{refresh_token}
activate API
API -> Service: refresh_token(data)
Service -> Service: validate_refresh_token()
Service -> Service: generate_new_access_token()
Service --> API: {access_token}
API --> Client: 200 OK + new token
deactivate API

== Registration Flow ==
Client -> API: POST /api/v1/auth/register\n{type: "public", user_data}
activate API
API -> Service: public_register(data)
activate Service
Service -> Service: validate_data()
Service -> DB: create_user()
DB --> Service: user
Service -> Service: generate_tokens()
Service --> API: {user, tokens}
API --> Client: 201 Created + user & tokens
deactivate Service
deactivate API
@enduml
```

## Activity Diagram - Registration Process

```plantuml
@startuml
start
:Client submits registration request;

if (Registration Type?) then (public)
  if (Public Registration Enabled?) then (yes)
    :Validate registration data;
    if (Terms Accepted?) then (yes)
      :Create new user;
      :Generate auth tokens;
      :Return user data and tokens;
    else (no)
      :Return error: Must accept terms;
      stop
    endif
  else (no)
    :Return error: Public registration disabled;
    stop
  endif
else (private)
  :Validate private registration data;
  if (Terms Accepted?) then (yes)
    :Create new user;
    :Generate auth tokens;
    :Return user data and tokens;
  else (no)
    :Return error: Must accept terms;
    stop
  endif
endif

stop
@enduml
```

## Key Components

### AuthViewSet
Main authentication controller that handles:
- Login (token generation)
- Token refresh
- Token verification (debug only)
- User registration (public and private)

### Rate Limiting
Two types of rate limiting are implemented:
- `LoginFailRateThrottle`: Limits failed login attempts
- `RegisterSuccessRateThrottle`: Limits successful registrations

### Authentication Methods
1. Normal Authentication
   - Username/password based
   - JWT token generation
   - Token refresh capability

2. Plugin-based Authentication
   - Supports multiple authentication backends
   - Configurable through auth plugins system

### Security Features
1. Token-based Authentication
   - Uses JWT tokens
   - Separate access and refresh tokens
   - Configurable token lifetime

2. Rate Limiting
   - Prevents brute force attacks
   - Limits registration spam

3. Terms Acceptance
   - Required for registration
   - Enforced at API level

## API Endpoints

| Endpoint | Method | Description |
|----------|---------|------------|
| `/api/v1/auth` | POST | Login endpoint for token generation |
| `/api/v1/auth/refresh` | POST | Refresh access token |
| `/api/v1/auth/verify` | POST | Verify token (debug only) |
| `/api/v1/auth/register` | POST | User registration endpoint |

## Error Handling

The authentication system handles several types of errors:
- Invalid credentials
- Rate limiting exceeded
- Invalid registration data
- Terms not accepted
- Public registration disabled
- Invalid token
- Token expired

Each error returns appropriate HTTP status codes and detailed error messages.