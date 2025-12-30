# ADVANCED SPRING SECURITY

This document focuses on internals, diagrams, and real-world mental models.

---

## 1. Complete Request Flow Diagram

```
Client
  |
  |  Authorization: Bearer JWT
  |
  v
+------------------------------+
|  Spring Security FilterChain |
+------------------------------+
        |
        v
+------------------------------+
| JwtAuthenticationFilter     |
| - Extract token             |
| - Validate token            |
| - Set SecurityContext       |
+------------------------------+
        |
        v
+------------------------------+
| FilterSecurityInterceptor   |
| - Check roles/authorities   |
+------------------------------+
        |
        v
+------------------------------+
| Controller                  |
+------------------------------+
```

---

## 2. SecurityContext & ThreadLocal

```
Thread (Request)
   |
   |-- SecurityContextHolder
           |
           |-- Authentication
                 |
                 |-- Principal
                 |-- Authorities
```

Each request thread has its own SecurityContext.

---

## 3. Authentication vs Authorization Flow

```
Authentication Phase
--------------------
Credentials -> AuthenticationManager -> AuthenticationProvider
           -> SecurityContext populated

Authorization Phase
-------------------
Request -> Required Authorities
        -> Compare with User Authorities
        -> Allow / Deny
```

---

## 4. Filter Order (Simplified)

```
SecurityContextPersistenceFilter
↓
JwtAuthenticationFilter
↓
AnonymousAuthenticationFilter
↓
ExceptionTranslationFilter
↓
FilterSecurityInterceptor
```

---

## 5. Why FilterSecurityInterceptor Is Critical

- Final authorization decision
- Uses AccessDecisionManager
- Throws AccessDeniedException

Nothing after this filter is security-related.

---

## 6. Async & Security Context

Async execution requires context propagation:

```java
DelegatingSecurityContextRunnable
DelegatingSecurityContextExecutor
```

Without this, SecurityContext is lost.

---

## 7. Common Production Mistakes

❌ Storing JWT in localStorage for browser apps  
❌ Mixing sessions and JWT  
❌ Fat authentication filters  
❌ Ignoring exception handling  
❌ No token expiration strategy  

---

## 8. Microservices Security Pattern

```
API Gateway
   |
   |-- Validate JWT
   |
Downstream Services
   |
   |-- Trust Gateway / Validate Again
```

---

## 9. Final Mental Model

Spring Security =
Filters + Authentication + SecurityContext + Authorization

Everything else is configuration.
