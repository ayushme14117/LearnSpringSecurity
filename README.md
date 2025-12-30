# LearnSpringSecurity

# Spring Security – Simple but Deep Explanation

This document explains **Spring Security** in **simple language**, but with **enough depth for a 7+ years experienced backend / Spring developer**.

The focus is on:
- Clear mental models
- How Spring Security actually works internally
- Real-world patterns used in production systems

---

## 1. What Spring Security Really Is

**Spring Security is a chain of servlet filters that sits in front of your application and decides:**

1. Who the user is (**Authentication**)
2. What the user can do (**Authorization**)
3. How requests are protected (**CSRF, CORS, sessions, headers, etc.**)

If a request does **not pass the Spring Security filter chain**, it never reaches your controller.

---

## 2. Core Mental Model (Most Important)

```
HTTP Request
   ↓
Spring Security Filter Chain
   ↓
DispatcherServlet
   ↓
Controller
```

Everything in Spring Security eventually boils down to **filters**.

---

## 3. Authentication – “Who Are You?”

Authentication answers:

> “Are you really who you claim to be?”

### Common Authentication Mechanisms
- Username + Password
- JWT Token
- OAuth2 Access Token

---

### Authentication Flow (Step by Step)

1. **Credentials arrive**
   - Login form
   - Authorization header (Bearer token)
   - OAuth2 callback

2. **Authentication object is created**
   ```java
   UsernamePasswordAuthenticationToken
   ```
   Initially unauthenticated.

3. **AuthenticationManager is invoked**
   ```java
   authenticationManager.authenticate(authentication)
   ```

4. **AuthenticationProvider validates**
   - Loads user (DB / LDAP / external service)
   - Verifies password or token
   - Returns authenticated Authentication

5. **SecurityContext is populated**
   ```java
   SecurityContextHolder.getContext().setAuthentication(auth);
   ```

From this point, Spring knows **who the user is**.

---

### Core Interfaces

| Interface | Responsibility |
|---------|----------------|
| Authentication | Holds principal and authorities |
| AuthenticationManager | Authentication entry point |
| AuthenticationProvider | Credential verification |
| UserDetailsService | Loads user data |
| SecurityContextHolder | Stores security context |

---

## 4. Authorization – “What Are You Allowed to Do?”

Authorization answers:

> “Now that I know who you are, can you access this resource?”

---

### URL-Based Authorization

```java
http.authorizeHttpRequests(auth -> auth
    .requestMatchers("/admin/**").hasRole("ADMIN")
    .anyRequest().authenticated()
);
```

---

### Method-Level Authorization (Recommended)

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser() {}
```

---

### Roles vs Authorities

- **Authority** → READ_USER
- **Role** → ROLE_ADMIN

Spring automatically prefixes roles with `ROLE_`.

---

## 5. Spring Security Filter Chain – The Real Engine

Spring Security is a **filter pipeline**.

### Simplified Filter Order

```
SecurityContextPersistenceFilter
↓
Authentication Filter (Form / JWT / OAuth2)
↓
AnonymousAuthenticationFilter
↓
ExceptionTranslationFilter
↓
FilterSecurityInterceptor
```

---

### What Each Filter Does

| Filter | Purpose |
|------|---------|
| SecurityContextPersistenceFilter | Loads and saves SecurityContext |
| Authentication Filter | Authenticates requests |
| AnonymousAuthenticationFilter | Assigns anonymous user |
| ExceptionTranslationFilter | Handles 401 / 403 |
| FilterSecurityInterceptor | Final authorization decision |

---

## 6. Stateful vs Stateless Security

### Stateful (Session-Based)

- Uses HTTP session
- Default for MVC apps

```java
http.sessionManagement()
    .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED);
```

---

### Stateless (JWT / REST APIs)

- No HTTP session
- Token sent on every request

```java
http.sessionManagement()
    .sessionCreationPolicy(SessionCreationPolicy.STATELESS);
```

Usually combined with:
- CSRF disabled
- Custom JWT filter

---

## 7. CSRF – When It Matters

CSRF protects **browser-based session authentication**.

| Scenario | CSRF |
|--------|------|
| Cookies + Session | ENABLE |
| JWT in Header | DISABLE |

```java
http.csrf().disable();
```

---

## 8. Exception Handling (401 vs 403)

| Situation | HTTP Status |
|---------|-------------|
| Not authenticated | 401 |
| Authenticated but forbidden | 403 |

Handled by:
- AuthenticationEntryPoint
- AccessDeniedHandler

---

## 9. Accessing Authenticated User

```java
@GetMapping("/me")
public UserDetails me(Authentication authentication) {
    return (UserDetails) authentication.getPrincipal();
}
```

Or:

```java
SecurityContextHolder.getContext().getAuthentication();
```

---

## 10. Typical REST API Architecture

```
Client
  ↓ (JWT)
Custom JWT Filter
  ↓
AuthenticationManager
  ↓
SecurityContextHolder
  ↓
Controller
```

---

## 11. OAuth2 – High-Level Flow

```
User → Authorization Server → Access Token
                                 ↓
                         Resource Server (Spring Security)
```

Spring Security:
- Validates token
- Maps scopes to authorities
- Applies authorization rules

---

# ADVANCED DETAILS

## 12. Why SecurityContextHolder Uses ThreadLocal

- One request = one thread
- ThreadLocal ensures isolation
- Context cleared after request completion

Async code requires:
```java
DelegatingSecurityContextRunnable
```

---

## 13. Authentication vs Identity

- Authentication = credential verification
- Identity = principal in SecurityContext

This separation allows re-authentication, impersonation, and token refresh.

---

## 14. FilterSecurityInterceptor – Final Gatekeeper

- Determines required permissions
- Compares with user authorities
- Throws AccessDeniedException if mismatch

Last security checkpoint before controller execution.

---

## 15. Custom JWT Authentication (Real World)

JWT filter responsibilities:
1. Extract token
2. Validate token
3. Build Authentication
4. Set SecurityContext

```java
SecurityContextHolder.getContext().setAuthentication(auth);
```

Nothing more.

---

## 16. Common Mistakes

❌ Mixing session auth with JWT  
❌ Disabling CSRF blindly  
❌ Putting business logic in filters  
❌ Ignoring filter order  
❌ Overusing SecurityContextHolder  

---

## 17. Spring Security 6 Notes

- WebSecurityConfigurerAdapter removed
- Lambda-based DSL
- Stronger defaults
- Explicit bean configuration

---

## 18. One-Paragraph Summary

Spring Security is a filter-based framework that authenticates requests, stores user identity in a thread-local security context, and authorizes access based on roles or authorities before the request reaches application code.

---

