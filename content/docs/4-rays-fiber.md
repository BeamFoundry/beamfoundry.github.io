---
title: "Rays Fiber"
weight: 4
---

# Rays Fiber

**Rays Fiber** seamlessly integrates the popular, Express-inspired [Go Fiber](https://gofiber.io/)
web framework with the **Rays IoC Container**.

By bringing Fiber into the IoC ecosystem, you can build blazing-fast web APIs using clean,
object-oriented controllers, effortless dependency injection, and declarative configuration.

## Links
* 📖 **[Read the Full Documentation](https://rays.bfdy.dev/rays-fiber/)**
* 💻 **[GitHub Repository](https://github.com/BeamFoundry/rays-fiber)**

## Key Features

1. **Injected Controllers:** Define your HTTP handlers as methods on a struct, allowing you to seamlessly inject repositories,
  services, and configurations using the Rays `@:"Inject"` tag.
2. **Centralized Lifecycle:** The HTTP server lifecycle (startup and graceful shutdown) is automatically managed by the Rays
  application context.
3. **Declarative Routing:** Define endpoint HTTP methods and paths directly via struct tags, keeping your routing logic exactly
  where the handler code lives.

## Example

```go
package controllers

import (
    "github.com/gofiber/fiber/v3"
    . "github.com/BeamFoundry/rays-fiber/pkg/core"
    . "github.com/BeamFoundry/rays/pkg/core"
    . "github.com/BeamFoundry/rays/pkg/web"
)

type UserController struct {
    FiberController
    RequestMapping `Path:"/api/users"`
    
    // Dependencies are auto-wired
    userService *UserService `@:"Inject"`
}

func (c *UserController) GetUser(
    // Declarative endpoint mapping
    _ struct{ RequestMapping `Method:"GET" Path:"/:id"` },
    ctx *fiber.Ctx,
) error {
    id := ctx.Params("id")
    user := c.userService.FindByID(id)
    return ctx.JSON(user)
}

```
