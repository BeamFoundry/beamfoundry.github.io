---
title: "Rays ORM"
weight: 2
---

# Rays ORM

**Rays ORM** is the official database integration package for the **BeamFoundry Rays** framework,
bringing the magic of Spring Data-style repositories to Go using [GORM](https://gorm.io/).

By bridging Rays' Inversion of Control (IoC) container with GORM and ThreadLocal storage,
this package completely eliminates database connection boilerplate, automatic repository method
generation, and context-passing for transactions.

## Links
* 📖 **[Read the Full Documentation](https://rays.bfdy.dev/rays-orm/)**
* 💻 **[GitHub Repository](https://github.com/BeamFoundry/rays-orm)**

## Key Features

1. **Auto-Generated Repository Methods:** By simply defining a function signature like `findByName func(string) *User`
  on your repository struct, Rays ORM will automatically generate the underlying GORM query logic and
  bind it to that field during the container's wiring phase.
2. **ThreadLocal Transactions:** Using `github.com/timandy/routine`, Rays ORM implements true ThreadLocal storage for Go.
  When a Service starts a transaction, the `*gorm.DB` transaction object is bound to the current goroutine.
  Downstream repositories transparently inherit this transaction without requiring you to pass it via function arguments.
3. **IoC Managed DB:** You define a standard `*gorm.DB` Beam, and the framework's `GormTxManager` handles the rest.
4. **Secure by Default (SQL Injection Prevention):** The auto-generated query builder does not concatenate raw strings.
It automatically translates your method names into **parameterized queries** using `?` placeholders, ensuring that all database
interactions are natively protected against SQL injection attacks without requiring manual sanitization.

## Example

```go
package repositories

import (
    "time"
    . "github.com/BeamFoundry/rays-orm"
    . "github.com/BeamFoundry/rays/pkg/core"
    . "github.com/BeamFoundry/rays/pkg/lang"
)

type User struct {
    ID        uint
    Name      string
    Age       uint
    CreatedAt time.Time
    UpdatedAt time.Time
}

// 1. Define the public contract
type UserRepositoryInterface interface {
    OrmRepositoryInterface[User, uint]

    FindByName(name string) *User
    FindActiveUsers(status string, minAge int) ([]*User, Error)
    FindByAgeRangeDynamic(min, max int, customFilters ...OrmQueryFunc) []*User
}

// 2. Define the implementation
type UserRepository struct {
    OrmRepository[User, uint]

    findByName                        func(string) *User
    findByStatusAndAgeGreaterThan     func(string, int) ([]*User, Error)
    findByAgeBetweenOrderByNameAscAndIdDesc func(int, int, ...OrmQueryFunc) []*User
}

// 3. Expose the generated functions
func (this *UserRepository) FindByName(name string) *User {
    return this.findByName(name)
}

func (this *UserRepository) FindActiveUsers(status string, minAge int) ([]*User, Error) {
    return this.findByStatusAndAgeGreaterThan(status, minAge)
}

func (this *UserRepository) FindByAgeRangeDynamic(min, max int, customFilters ...OrmQueryFunc) []*User {
    return this.findByAgeBetweenOrderByNameAscAndIdDesc(min, max, customFilters...)
}
```
