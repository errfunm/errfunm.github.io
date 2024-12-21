---
title: Microservice Directory Structure Documentation
publish: true
---
# Microservice Folder Structure Documentation

This document explains the folder structure convention used in our company's microservices, including the purpose of each directory and file. Examples are provided to clarify their use.
kdsjslkl
---

## Table of Contents

1. [Folder Overview](#folder-overview)
2. [Directory and File Details](#directory-and-file-details)
    - [cmd/](#cmd)
    - [config/](#config)
    - [interface/api/grpc/](#interfaceapigrpc)
    - [internal/](#internal)
    - [testing/](#testing)
    - [.gitignore](#gitignore)
    - [.mockery.yaml](#.mockery.yaml)
    - [Makefile](#makefile)
    - [README.md](#readme.md)
    - [go.mod & go.sum](#gomod--gosum)
    - [tools.go](#tools.go)
3. [versioning](#versioning)

---

## Folder Overview

Here is an overview of the microservice folder structure:

```
microservice/
│
├── cmd/                      # Main entry points of the application
├── config/                   # Configuration files and settings
├── interface/api/grpc/       # API layer for gRPC communications
├── internal/                 # Private application logic
├── testing/                  # Test utilities
│
├── .gitignore                # Git ignore rules
├── .mockery.yaml             # Mockery tool configuration
├── Makefile                  # Automation commands for building/testing
├── README.md                 # Project documentation
├── go.mod & go.sum           # Go module dependency files
└── tools.go                  # Tools dependencies
```

---

## Directory and File Details

### `cmd/`

**Purpose:**  
Contains the application's entry points. The `cmd/` directory contains two services: `reader` and `writer`.

**Structure:**
```
cmd/
├── reader/
│   └── main.go
└── writer/
    └── main.go
```

**Example:**

`cmd/reader/main.go`
```go
package main

import (
	"context"
	"net/http"
	"time"

	"github.com/47monad/apin"
	"github.com/47monad/apin/initr"
	"github.com/47monad/apin/initropts"
	"github.com/47monad/apin/runner"
	"github.com/47monad/myMS/interface/api/sgrpc"
	"github.com/47monad/myMS/internal/app/svc"
	"github.com/47monad/myMS/internal/infra/persist"
	"github.com/47monad/myMS-go-genprotos/foopb"
	"github.com/47monad/be-tools-go/database/mongoutils"
	"github.com/47monad/be-tools-go/interface/grpckit"
	"github.com/47monad/be-tools-go/interface/grpcutils"
	"github.com/47monad/be-tools-go/interface/httputils"
	"google.golang.org/grpc"
)

func main() {
	app, err := apin.Load(apin.WithAppDir("reader"))
	if err != nil {
		panic(err)
	}
	apin.Must(app.InitZap(context.Background()))

	apin.Must(app.InitMongodb(context.Background()))
	defer initr.EnsureDisposed(app.MongodbShell)

	// instantiate repo
	repo := persist.NewMongoFooRepo(app.MongodbShell.Db)
	// instantiate service
	domainFooService := svc.NewFoorService(repo)
	// instantiate readerService
	reader := sgrpc.NewFooReaderService(domainFooService)

	apin.Must(app.InitPrometheus(context.Background()))

	apin.Must(app.InitGrpc(
		context.Background(),
		initropts.GrpcServer().
			AddInterceptor(grpcutils.InstanceIdInterceptor).
			AddInterceptor(grpckit.ErrorInterceptor).
			WithRunnable(func(server *grpc.Server) {
				foopb.RegisterFooReaderServer(server, reader)
			})))

	// Run the application
	r := runner.New(context.Background(), app, 3)

	r.AddGrpcServer(app.GrpcServerShell.Server)
	r.AddHttp(func(mux *http.ServeMux) {
		httputils.AttachPromHandler(mux, app.PrometheusShell.Registry)
	})
	r.AddHealthCheck(app.GrpcServerShell.HealthServer, 10*time.Second, func(ctx context.Context) bool {
		return mongoutils.IsHealthy(ctx, app.MongodbShell.Client)
	})

	if err := r.Run(); err != nil {
		panic(err)
	}
}
```

* The `cmd/writer/main.go` is going to be pretty the same.

---

### `config/`

**Purpose:**  
Holds configuration settings for the services such as environment variables and configuration files. Each service (`reader` and `writer`) has its own configuration.

**Structure:**
```
config/
├── reader/
│   ├── app.pkl
│   └── .env
├── writer/
│   ├── app.pkl
│   └── .env
└── main.go
```

**Example:**

`config/reader/.env`
```dotenv
APP_ENV=production
DBNAME=test
SERVER_PORT=8081
```

`config/reader/app.pkl`
```pkl
amends "https://raw.githubusercontent.com/47monad/sercon/main/pkl/ServiceConfig.pkl"

name = "ms_name Reader"
version = "0.5.0"

mongodb {
  enabled = true
}

prometheus {
  enabled = true
  useGrpcMetrics = true
}
```


---

### `interface/api/sgrpc/`

**Purpose:**  
Interface layer contains the handlers which are going to be invoked by outside. 
The `sgrpc` package Contains gRPC service handlers.
The gRPC service definitions are separate modules.

**Structure:**
```
interface/api/sgrpc/
   ├── foo_mapper.go
   ├── foo_reader.go
   ├── foo_reader_test.go
   ├── foo_writer.go
   └── foo_writer_test.go
```

**Example:**

`interface/api/sgrpc/foo_reader.go`
```go
package sgrpc

import (
	"context"

	"github.com/47monad/authril-accounts/internal/domain/msdom"
	"github.com/47monad/myMS-go-genprotos/mspb"
)

type fooReaderService struct {
	mspb.UnimplementedfooReaderServer
	srvc msdom.FooService
}

func (f fooReaderService) List(ctx context.Context, req *accountpb.ListOrganizationsReq) (*accountpb.ListOrganizationsResp, error) {
	panic("implement me")
}

func (f fooReaderService) Fetch(ctx context.Context, req *mspb.FetchFooReq) (*mspb.FetchFooResp, error) {
	panic("implement me")
}

func NewFooReaderService(srvc msdom.FooService) mspb.FooReaderServer {
	return &fooReaderService{
		srvc: srvc,
	}
}
```

---

### `internal/`

**Purpose:**  
Holds the core business logic, domain models, and infrastructure code. This directory is central to the service's architecture and is not exported outside the module, ensuring encapsulation and preventing unintentional dependencies.

- **`app/`**: Contains application-level services and orchestration logic. It is the entry point for coordinating domain and infrastructure layers.
- **`domain/`**: Represents the core business logic and domain models. It contains factories, repositories, and services that encapsulate business rules.
- **`infra/`**: Implements infrastructure-specific details like database models, repositories, and data mappers. This layer bridges the domain logic with external systems.

**Structure:**
```
internal/
├── app/
│   └── svc/
|      ├── foo_service.go
|      └── foo_service_test.go
├── domain/
│   ├── foo.go
│   ├── foo_factory.go
│   ├── foo_factory_test.go
│   ├── foo_repository.go
│   └── foo_service.go
└── infra/
    ├──  mongo_foo_model.go
    ├── mongo_foo_repo.go
    ├── mongo_foo_repo_test.go
    └── foo_mapper.go
```

`internal/app/svc/foo_service.go`
```go
package svc

type FooService struct {}

func (f *FooService) DoSomething() string {
    return "Foo Service Logic"
}
```

`internal/domain/foo.go`
```go
package domain

type Foo struct {
    ID   string
    Name string
}
```

`internal/infra/mongo_foo_repo.go`
```go
package infra

import "internal/domain"

type MongoFooRepository struct {}

func (m *MongoFooRepository) Save(foo domain.Foo) error {
    // MongoDB saving logic
    return nil
}
```

---
### `testing/`

**Purpose:**  
The `testing/` directory contains resources specifically designed to facilitate testing, such as mocks and testing utilities. It ensures tests are isolated, efficient, and easy to maintain. This directory includes the following subdirectories:

#### `mocks/`

Holds generated mocks for interfaces used throughout the application. These mocks are typically created using tools like `mockery` to simulate dependencies during testing.

**Example:**
`testing/mocks/my_service_mock.go`
```go
package mocks

type MyServiceMock struct {}

func (m *MyServiceMock) GetSomething() string {
    return "mocked value"
}
```

#### `fakes/`

Contains testing utilities, specifically:
1. **Fake Generators:** Functions or utilities to generate fake data or objects for testing purposes.
2. **In-Memory Repository Implementations:** Simplified, in-memory versions of repository layers used in cases where mocks are insufficient, such as testing complex interactions or verifying data integrity.

**Example:**
`testing/fakes/in_memory_repo.go`
```go
package fakes

import "myservice/internal/domain"

type InMemoryFooRepository struct {
    data map[string]domain.Foo
}

func NewInMemoryFooRepository() *InMemoryFooRepository {
    return &InMemoryFooRepository{data: make(map[string]domain.Foo)}
}

func (repo *InMemoryFooRepository) Save(foo domain.Foo) error {
    repo.data[foo.ID] = foo
    return nil
}

func (repo *InMemoryFooRepository) FindByID(id string) (domain.Foo, error) {
    foo, exists := repo.data[id]
    if !exists {
        return domain.Foo{}, errors.New("not found")
    }
    return foo, nil
}
```
---
### `.gitignore`

**Purpose:**  
Specifies files and directories to ignore in version control.

**Example:**
```plaintext
*.env

# Binaries for programs and plugins
*.exe
*.exe~
*.dll
*.so
*.dylib

# Test binary, built with `go test -c`
*.test

# Output of the go coverage tool, specifically when used with LiteIDE
*.out

# Dependency directories
vendor/

# Go workspace file
go.work

builds/

.idea/

.sercon/

.config/

docs/

bin/
```

---
### `.mockery.yaml`

**Purpose:**  
Holds configuration for the Mockery tool, used for generating mocks in Go.

**Example:**
```yaml
mockery:
  output: ./testing/mocks
  recursive: true
```

---

### `Makefile`

**Purpose:**  
Automates common tasks such as building, testing, formatting and cleaning the project.
A comprehensive and well-maintained Makefile version is already available separately and will be included in all microservices.
- source: https://github.com/47monad/gomake

---

### `README.md`

**Purpose:**  
Provides documentation for the project, including setup, usage, and contribution guidelines.

---

### `go.mod` & `go.sum`

**Purpose:**  
`go.mod` manages Go module dependencies. `go.sum` locks versions of the dependencies.

**Example:**

`go.mod`
```go
module github.com/company/myservice

go 1.21

require (
    github.com/some/library v1.2.3
)
```

---

### `tools.go`

**Purpose:**  
Tracks tool dependencies used in the project for consistency.

**Example:**
```go
//go:build tools

package tools

import (
	_ "github.com/47monad/sercon"
)
```
---
## versioning

| revision number | revision date | revised By  |
| --------------- | ------------- | ----------- |
| 1               | 2024/20/12    | Erfan Baghi |
