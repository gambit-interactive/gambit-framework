# 🚀 Gambit Framework

[![Go version](https://img.shields.io/badge/go-1.18%2B-blue.svg)](https://golang.org/dl/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Build Status](https://img.shields.io/badge/build-passing-brightgreen)](https://github.com/gambit-framework/gambit-framework/actions)

**Gambit** is a modern, opinionated web framework in Go, built on the pillars of Clean Architecture, Domain-Driven Design (DDD), and Test-Driven Development (TDD). It is designed to create robust, secure, scalable, and maintainable APIs, featuring a complete authentication and authorization system out-of-the-box.

## 🎯 Core Principles & Features

*   **Clean Architecture**: Strict separation between business rules, application logic, and infrastructure details.
*   **Domain-Driven Design (DDD)**: Focus on a rich and expressive domain model that reflects business logic.
*   **Security by Default**: A complete authentication and authorization system (JWT, Roles, OTP) is integrated into the core.
*   **Test-Driven Development (TDD)**: A structure that encourages and facilitates unit, integration, and E2E testing.
*   **Environment-based Configuration**: Adapt behavior (database, email/SMS sending) through environment variables.
*   **Multi-Database Support**: Use SQLite for rapid development or switch to PostgreSQL for production with a single `.env` change.
*   **Role-Based Access Control (RBAC)**: Granular access control with a flexible role-based middleware.
*   **Business Accounts & API Key Management**: Allow users to create their own applications with `public_key` and `secret_key` to consume your API.
*   **Device Tracking**: Identify requests from mobile clients (e.g., `User-Agent: Dart`) through custom headers.

## 🏗️ Architecture

Gambit strictly adheres to the principles of Clean Architecture, ensuring that dependencies always flow inwards, from the most concrete (Infrastructure) to the most abstract (Domain).

```
┌─────────────────────────────────────────────────────────┐
│                    Presentation Layer                   │
│      (HTTP Handlers, Middleware, Websockets, gRPC)      │
└─────────────────────┬───────────────────────────────────┘
                      │ (Invokes Use Cases with DTOs)
┌─────────────────────▼───────────────────────────────────┐
│                  Application Layer                      │
│      (Use Cases, DTOs, Application Services)            │
└─────────────────────┬───────────────────────────────────┘
                      │ (Uses Entities and Repositories)
┌─────────────────────▼───────────────────────────────────┐
│                   Domain Layer                          │
│  (Entities, Value Objects, Repositories, Domain Events) │
└─────────────────────┬───────────────────────────────────┘
                      │ (Implemented by Infrastructure)
┌─────────────────────▼───────────────────────────────────┐
│                Infrastructure Layer                     │
│   (Database, Notifiers, Caching, External APIs)         │
└─────────────────────────────────────────────────────────┘
```

## 📁 Project Structure

```
gambit-framework/
├── cmd/api/                  # Application entry point
├── config/                   # .env loading and validation
├── internal/
│   ├── domain/               # The core of your business (Entities, Repos)
│   ├── application/          # Application logic (Use Cases)
│   ├── infrastructure/       # External details (DB, HTTP, Notifiers)
├── pkg/                      # Public, reusable packages
├── tests/                    # Integration tests and mocks
├── .env.example              # Example configuration file
├── Makefile                  # Automation scripts
├── go.mod
└── README.md
```

## 🚀 Getting Started

Follow these steps to get a Gambit development environment up and running on your machine.

### Prerequisites

*   [Go](https://golang.org/dl/) (version 1.18 or higher)
*   [Docker](https://www.docker.com/products/docker-desktop) (Optional, for running Postgres/Redis)
*   [Air](https://github.com/cosmtrek/air) (Optional, for live-reloading)

### Installation

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/gambit-framework/gambit-framework.git
    cd gambit-framework
    ```

2.  **Install dependencies:**
    ```bash
    go mod tidy
    ```

3.  **Set up your environment variables:**
    Copy the example file and edit it as needed. For a quick start, the defaults (`sqlite` and `log` for notifications) work out-of-the-box with no extra setup.
    ```bash
    cp .env.example .env
    ```

4.  **Start the server:**
    ```bash
    make run
    ```
    Or for development with live-reloading:
    ```bash
    make dev
    ```

5.  **Verify it's running:**
    Open your browser or use `curl` to access the API. You should see a welcome message.
    ```bash
    curl http://localhost:8080
    # Expected response: {"message":"Welcome to Gambit Framework API!"}
    ```

## 🔧 Configuration (.env)

The `.env` file controls the application's behavior. Here are the most important variables:

```env
#-- Application Settings
APP_ENV=development # development, staging, production
SERVER_PORT=8080

#-- Database Settings
# For a quick start, the default is sqlite, which requires no server.
DB_DRIVER=sqlite
DB_DSN=./gambit.db # Path to the SQLite file

# Example for Postgres
# DB_DRIVER=postgres
# DB_DSN="host=localhost user=user password=pass dbname=gambit port=5432 sslmode=disable"

#-- JWT Settings
JWT_SECRET="your-super-secret-key-that-is-long"
JWT_ACCESS_TOKEN_EXP_MINUTES=15
JWT_REFRESH_TOKEN_EXP_HOURS=72

#-- Notification Settings
# DRIVER: "log" (for dev, prints to console), "smtp" (real email)
# In 'development', use 'log' to avoid real sends and costs.
EMAIL_NOTIFIER_DRIVER=log
SMS_NOTIFIER_DRIVER=log

# Disable completely if needed
EMAIL_ENABLED=true
SMS_ENABLED=false
```

## 💻 Usage & Concepts

### Defining Routes & Middleware

Routes are defined in `internal/infrastructure/http/routes/api.go` using [chi](https://github.com/go-chi/chi). Authentication and authorization middleware can be applied to route groups.

```go
// internal/infrastructure/http/routes/api.go
r.Group(func(r chi.Router) {
    // Apply the authentication middleware to all routes in this group
    r.Use(middleware.Auth()) 

    // A route that only requires authentication
    r.Get("/profile", userHandler.GetProfile)

    // A group of routes that require the "admin" role
    r.Group(func(r chi.Router) {
        r.Use(middleware.RequireRole("admin"))
        r.Get("/admin/dashboard", adminHandler.ShowDashboard)
    })
})
```

### Creating a Use Case

Use cases orchestrate business logic and are the heart of the application layer.

```go
// internal/application/usecases/auth/register_user.go
type RegisterUserUseCase struct {
    userRepo        repositories.UserRepository
    notificationSvc notifiers.NotificationService
    // ... other dependencies
}

func (uc *RegisterUserUseCase) Execute(ctx context.Context, input dto.RegisterUserInput) (*dto.RegisterUserOutput, error) {
    // 1. Validate if the user already exists
    // 2. Create the User entity
    // 3. Hash the password
    // 4. Persist the user in the database
    // 5. Generate an OTP (One-Time Password)
    // 6. Send a welcome notification (email/SMS)
    
    // ... implementation
    return &dto.RegisterUserOutput{UserID: user.ID.String()}, nil
}
```

## 🧪 Testing

Gambit is built with TDD in mind. To run all tests in the project:

```bash
make test
```

To view a test coverage report:

```bash
make test-coverage
```

### Use Case Test Example

Thanks to dependency injection and the use of interfaces, testing use cases is simple and fast.

```go
// internal/application/usecases/auth/register_user_test.go
func TestRegisterUserUseCase_Execute(t *testing.T) {
    // Arrange: Create mocks for all dependencies
    mockUserRepo := new(mocks.MockUserRepository)
    mockNotifier := new(mocks.MockNotificationService)

    // Configure the expected behavior of the mocks
    mockUserRepo.On("FindByEmail", mock.Anything, "test@example.com").Return(nil, errors.New("not found"))
    mockUserRepo.On("Create", mock.Anything, mock.AnythingOfType("*entities.User")).Return(nil)
    mockNotifier.On("SendEmail", mock.Anything, mock.Anything, mock.Anything, mock.Anything).Return(nil)

    useCase := auth.NewRegisterUserUseCase(mockUserRepo, mockNotifier /*...*/)
    input := dto.RegisterUserInput{ /*...*/ }

    // Act: Execute the use case
    output, err := useCase.Execute(context.Background(), input)

    // Assert: Check the results
    assert.NoError(t, err)
    assert.NotNil(t, output)
    mockUserRepo.AssertCalled(t, "Create", mock.Anything, mock.Anything)
}
```

## 📊 Useful Commands (Makefile)

| Command               | Description                                           |
| --------------------- | ----------------------------------------------------- |
| `make run`            | Starts the API server.                                |
| `make dev`            | Starts the server in development mode with hot-reload.|
| `make test`           | Runs all tests in the project.                        |
| `make test-coverage`  | Runs tests and generates an HTML coverage report.     |
| `make lint`           | Lints the codebase for errors and style issues.       |
| `make generate-mocks` | Generates mock implementations for domain interfaces. |
| `make migrate-up`     | Applies database migrations (for Postgres, etc.).     |
| `make migrate-down`   | Reverts the last applied migration.                   |
| `make build`          | Compiles the application binary for production.       |

## 🤝 Contributing

Contributions are welcome! Feel free to open an issue or submit a pull request.

1.  **Fork** the project.
2.  Create a new feature branch (`git checkout -b feat/new-feature`).
3.  Commit your changes (`git commit -m 'feat: Add new feature'`).
4.  Push to your branch (`git push origin feat/new-feature`).
5.  Open a **Pull Request**.

## 📄 License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.