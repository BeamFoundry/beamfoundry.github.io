---
title: "Rays Core"
weight: 1
---

# Rays Core

**Rays** is a powerful, enterprise-grade Dependency Injection (DI) and Inversion of Control (IoC)
framework for Go, built by **BeamFoundry**.

Inspired by mature ecosystems like Spring Boot, Rays brings structured lifecycles, annotation-driven
configuration, and robust dependency wiring to Go, while respecting Go's idiomatic use of composition
and interfaces. It eliminates boilerplate wiring, enforces clean boundaries, and makes your Go applications
highly testable.

## Links
* 📖 **[Read the Full Documentation](https://rays.bfdy.dev/rays/)**
* 💻 **[GitHub Repository](https://github.com/BeamFoundry/rays)**

## Key Features

1. **Annotation-Driven, but Type-Safe:** We use struct tags (`@:"Inject"`) for declaring dependencies,
  but rely on Go's strict typing to ensure the container wires the correct pointers.
2. **Encapsulation Matters:** Unlike manual DI which forces you to expose dependencies via public fields
  or constructor arguments, Rays natively injects into **private fields**. Your internal state remains protected.
3. **Two-Phase Bootstrapping:** To safely handle circular dependencies—a common pain point in large systems—Rays
  separates instantiation/injection from initialization.
4. **Context-Aware Scoping:** Singletons are great for services, but short-lived state requires a different approach.
  Rays introduces `Foundry[T]` to bridge the gap between global services and request-scoped prototypes.

## Example

```go
package main

import (
    "github.com/BeamFoundry/rays/pkg/core"
    "github.com/BeamFoundry/rays/pkg/lang"
    "github.com/BeamFoundry/rays"
)

// 1. Define a Repository
type Database struct {
    core.Component // Stereotype
    dsn string
}

func (db *Database) New() *Database {
    db.dsn = "postgres://localhost/mydb"
    return db
}

// 2. Define a Service
type UserService struct {
    core.Service //
    db     *Database    `@:"Inject"` // Private field injection!
    logger core.Logging `@:"Inject"`
}

func (s *UserService) PostConstruct() lang.Error {
    s.logger.Info("UserService ready!","connection", s.db.dsn)
    return nil
}

// 3. Define an Application, hosting a Run method.
type BatchUserApplication struct {
    core.Application
    userService *UserService `@:"Inject"`
    logger      core.Logging `@:"Inject"`
}

// 4. Define a Run method
func (b *BatchUserApplication) Run(wg *sync.WaitGroup) {
    wg.Add(1) // 1. Register this worker with the global WaitGroup

    b.logger.Info("BatchUserApplication Run started...")

    go func() {
        defer wg.Done() // 2. Ensure we decrement the WaitGroup when finished

        b.logger.Info("BatchUserApplication is running...")

        // Background loop logic goes here...
        for {
            time.Sleep(10 * time.Second)
        }
    }()
}

// 5. Boot the Container
func main() {
    ctx := rays.ApplicationContext()

    if err := ctx.Initialize(); err != nil {
        err.LogErrorWithStackTrace()
        return
    }

    ctx.Run() // Runs application lifecycle
}
```
