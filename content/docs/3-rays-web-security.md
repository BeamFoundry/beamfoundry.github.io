---
title: "Rays Web Security"
weight: 3
---

# Rays Web Security

**Rays Web Security** brings the declarative power and robust architecture of Spring Security directly to Go.

Built natively on top of the Rays IoC container, this package provides authentication, authorization, and
protection against common exploits. It integrates seamlessly with web frameworks (like Rays Fiber) to secure
your HTTP endpoints using clean, fluent builder chains.

It deeply integrates with Rays Fiber via `HttpSecurity` configurations, allowing you to automatically protect
your endpoints using expressive, chainable security rules.

## Links
* 📖 **[Read the Full Documentation](https://rays.bfdy.dev/rays-web-security/)**
* 💻 **[GitHub Repository](https://github.com/BeamFoundry/rays-web-security)**

## Key Features

1. **Declarative Authorization:** Define endpoint security rules using a highly expressive `HttpSecurityBuilder`
  rather than imperative `if/else` blocks inside your handlers.
2. **Pluggable Authentication:** Easily swap out how users are authenticated by implementing the
  `AuthenticationProvider` or `UserDetailsService` interfaces.
3. **ThreadLocal Context:** Web Security utilizes `github.com/timandy/routine` to store the currently
  authenticated user in a ThreadLocal `SecurityContext`. Your deeper service layers can retrieve the
  user without needing the HTTP request object passed down.
4. **IoC Native Auto-Configuration:** The framework provides default security implementations out of
  the box, which you can easily override by defining your own Beams.

## Example

```go
package configurations

import (
    . "github.com/BeamFoundry/rays-web-security/pkg/config"
    . "github.com/BeamFoundry/rays/pkg/core"
)

type SecurityConfig struct {
    Configuration
}

// Construct the HttpSecurity rules using the Builder
func (this *SecurityConfig) HttpSecurity() *HttpSecurity {
    return (&HttpSecurityBuilder{}).
        AuthorizedHTTPRequests().
        Paths("/api/public/**", "/health").PermitAll().
        Paths("/api/admin/**").HasAuthority("ROLE_ADMIN").
        And().
        AnyRequest().Authenticated().
        Basic(). // Enables Basic Authentication by default
        Build()
}

```
