# Go — CLAUDE.md Template

> Copy this entire file into `.claude/CLAUDE.md` at your project root.
> Replace all `<!-- CUSTOMIZE -->` sections with your project-specific values.

---

```markdown
# CLAUDE.md

<!-- CUSTOMIZE: Replace with your project name and description -->
## Project: My Service
A Go microservice with HTTP API, PostgreSQL storage, and Docker deployment.

---

## Build & Run Commands

```bash
# Build
go build -o bin/server ./cmd/server          # Build server binary
go build ./...                                # Build all packages (verify compilation)

# Run
go run ./cmd/server                           # Run server locally
go run ./cmd/migrate                          # Run database migrations

# Test
go test ./...                                 # Run all tests
go test -v ./internal/user/...                # Verbose tests for one package
go test -run TestCreateUser ./internal/user   # Run a specific test
go test -race ./...                           # Run with race detector
go test -count=1 ./...                        # Disable test caching
go test -coverprofile=coverage.out ./...      # Generate coverage
go tool cover -html=coverage.out              # View coverage in browser

# Lint & Format
golangci-lint run                             # Run all linters
golangci-lint run --fix                       # Auto-fix where possible
gofmt -w .                                    # Format all Go files
go vet ./...                                  # Vet for suspicious constructs

# Dependencies
go mod tidy                                   # Clean up go.mod and go.sum
go mod download                               # Download dependencies
go mod verify                                 # Verify dependency integrity

# Code Generation
go generate ./...                             # Run all go:generate directives
```

---

## Code Conventions

### Go Style
- Follow [Effective Go](https://go.dev/doc/effective_go) and the [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)
- **Error handling is mandatory** — never ignore errors with `_`. If truly intentional, add a comment
- Keep functions short (under 40 lines preferred)
- Accept interfaces, return structs
- Use table-driven tests

### Naming
- Packages: short, lowercase, single word (`user`, `auth`, `store`) — never `utils`, `helpers`, `common`
- Interfaces: verb or verb-noun (`Reader`, `UserStore`, `Authenticator`) — not `IUser`
- Exported functions/types: `PascalCase`
- Unexported: `camelCase`
- Acronyms: all caps (`HTTPHandler`, `UserID`, `ParseJSON`)
- Test files: `*_test.go` in the same package (or `_test` package for black-box testing)
- Error variables: `ErrNotFound`, `ErrPermissionDenied`

```go
// GOOD: Clear interface with single responsibility
type UserStore interface {
    GetByID(ctx context.Context, id string) (*User, error)
    Create(ctx context.Context, user *User) error
    Update(ctx context.Context, user *User) error
    Delete(ctx context.Context, id string) error
}

// GOOD: Concrete implementation
type pgUserStore struct {
    db *pgxpool.Pool
}

func NewUserStore(db *pgxpool.Pool) UserStore {
    return &pgUserStore{db: db}
}
```

### Error Handling Patterns
```go
// GOOD: Wrap errors with context
user, err := store.GetByID(ctx, id)
if err != nil {
    return nil, fmt.Errorf("get user %s: %w", id, err)
}

// GOOD: Sentinel errors for expected conditions
var ErrNotFound = errors.New("not found")
var ErrConflict = errors.New("conflict")

// GOOD: Check sentinel errors
if errors.Is(err, ErrNotFound) {
    http.Error(w, "User not found", http.StatusNotFound)
    return
}
```

### Context
- Always pass `context.Context` as the first parameter
- Never store context in a struct
- Use context for cancellation, deadlines, and request-scoped values only

### Concurrency
- Prefer channels for communication, mutexes for state protection
- Always use `sync.WaitGroup` or `errgroup.Group` for goroutine lifecycle
- Never start goroutines without a clear shutdown mechanism
- Use `context.WithCancel` or `context.WithTimeout` for goroutine cancellation

```go
// GOOD: errgroup for concurrent operations with error handling
g, ctx := errgroup.WithContext(ctx)

g.Go(func() error {
    return fetchUserProfile(ctx, userID)
})
g.Go(func() error {
    return fetchUserOrders(ctx, userID)
})

if err := g.Wait(); err != nil {
    return fmt.Errorf("fetch user data: %w", err)
}
```

---

## Project Structure

<!-- CUSTOMIZE: Adjust to match your project layout -->

```
cmd/
  server/
    main.go                    # Entry point — wire up dependencies, start server
  migrate/
    main.go                    # Database migration runner
internal/
  user/
    handler.go                 # HTTP handlers
    handler_test.go            # Handler tests
    service.go                 # Business logic
    service_test.go            # Service tests
    store.go                   # Database queries (interface + implementation)
    store_test.go              # Store tests (integration)
    model.go                   # Domain types
  auth/
    handler.go
    middleware.go              # Auth middleware
    jwt.go                     # Token creation/validation
  middleware/
    logging.go                 # Request logging
    recovery.go                # Panic recovery
    cors.go                    # CORS configuration
  config/
    config.go                  # Configuration loading (env vars)
  database/
    postgres.go                # Connection pool setup
    migrations/                # SQL migration files
      001_create_users.up.sql
      001_create_users.down.sql
pkg/                           # Public packages (only if building a library)
  httputil/
    response.go                # Shared HTTP response helpers
api/
  openapi.yaml                 # API specification
scripts/
  seed.sh                      # Database seeding
Dockerfile
docker-compose.yml
Makefile                       # Build automation
.golangci.yml                  # Linter configuration
```

### Package Rules
- `internal/` — all application code lives here (not importable by external packages)
- `pkg/` — only for truly reusable packages (most projects don't need this)
- `cmd/` — one directory per binary
- Each domain package (`user/`, `auth/`) owns its own handler, service, store, and model
- No circular imports — dependencies flow inward: `handler -> service -> store`

---

## Testing

### Table-Driven Tests (Preferred Pattern)
```go
func TestValidateEmail(t *testing.T) {
    tests := []struct {
        name    string
        email   string
        wantErr bool
    }{
        {"valid email", "alice@example.com", false},
        {"missing @", "aliceexample.com", true},
        {"empty string", "", true},
        {"with plus", "alice+tag@example.com", false},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            err := ValidateEmail(tt.email)
            if (err != nil) != tt.wantErr {
                t.Errorf("ValidateEmail(%q) error = %v, wantErr %v", tt.email, err, tt.wantErr)
            }
        })
    }
}
```

### Integration Tests
- Use build tags: `//go:build integration`
- Run separately: `go test -tags=integration ./...`
- Use `testcontainers-go` for database tests

```go
//go:build integration

func TestUserStore_Create(t *testing.T) {
    ctx := context.Background()
    db := setupTestDB(t) // testcontainers PostgreSQL
    store := NewUserStore(db)

    user := &User{Name: "Alice", Email: "alice@test.com"}
    err := store.Create(ctx, user)
    require.NoError(t, err)
    require.NotEmpty(t, user.ID)

    got, err := store.GetByID(ctx, user.ID)
    require.NoError(t, err)
    assert.Equal(t, "Alice", got.Name)
}
```

### Test Helpers
- Use `t.Helper()` for helper functions
- Use `testify/assert` and `testify/require` for assertions
- Prefer `require` (fail immediately) for setup steps, `assert` for verifications

### Coverage Requirements
- Minimum 80% overall
- 100% for `auth/` and security-critical code
- Enforce in CI: `go test -coverprofile=c.out ./... && go tool cover -func=c.out`

---

## Linter Configuration (.golangci.yml)

```yaml
# .golangci.yml
run:
  timeout: 5m

linters:
  enable:
    - errcheck
    - gosimple
    - govet
    - ineffassign
    - staticcheck
    - unused
    - gofmt
    - goimports
    - misspell
    - unparam
    - gocritic
    - revive
    - gosec
    - prealloc
    - errorlint
    - exhaustive
    - noctx          # Finds http requests without context
    - bodyclose      # Checks response body is closed

linters-settings:
  revive:
    rules:
      - name: exported
        arguments: [checkPrivateReceivers]
      - name: unused-parameter
  gocritic:
    enabled-tags:
      - diagnostic
      - style
      - performance
  exhaustive:
    default-signifies-exhaustive: true

issues:
  exclude-rules:
    - path: _test\.go
      linters:
        - gosec
        - errcheck
```

---

## Deployment

### Docker
```dockerfile
# Dockerfile
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o /bin/server ./cmd/server

FROM alpine:3.19
RUN apk add --no-cache ca-certificates
COPY --from=builder /bin/server /bin/server
EXPOSE 8080
ENTRYPOINT ["/bin/server"]
```

### Makefile
```makefile
.PHONY: build test lint run migrate

build:
	go build -o bin/server ./cmd/server

test:
	go test -race ./...

lint:
	golangci-lint run

run:
	go run ./cmd/server

migrate:
	go run ./cmd/migrate

docker:
	docker build -t myservice .
```

---

## Common Tasks

### Add a new domain (e.g., "orders")
1. Create `internal/order/` with `model.go`, `store.go`, `service.go`, `handler.go`
2. Define the `OrderStore` interface in `store.go`
3. Implement PostgreSQL store
4. Wire up in `cmd/server/main.go`
5. Add migration in `internal/database/migrations/`
6. Write tests for each layer

### Add a new endpoint
1. Add handler method in the domain's `handler.go`
2. Register route in the domain's router setup
3. Write handler test with httptest
4. Update `api/openapi.yaml`

### Add middleware
1. Create file in `internal/middleware/`
2. Implement `func(next http.Handler) http.Handler` pattern
3. Chain in `cmd/server/main.go`
```
