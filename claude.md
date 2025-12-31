# Claude Development Notes

## Project Conventions

### Use Make for All Lifecycle Commands

**IMPORTANT:** Always use `make` commands instead of bare Go commands. The Makefile ensures proper environment setup and runs all necessary pre-steps.

#### Available Make Targets

**Development:**
- `make test` - Run tests (includes setup-envtest, coverage, formatting, vetting)
- `make build` - Build the manager binary
- `make fmt` - Format code with go fmt
- `make fmt-check` - Check if code is formatted
- `make vet` - Run go vet against code
- `make manifests` - Generate WebhookConfiguration, ClusterRole and CRD objects
- `make generate` - Generate DeepCopy, DeepCopyInto, and DeepCopyObject methods

**Build & Deploy:**
- `make docker-build` - Build Docker image
- `make docker-push` - Push Docker image
- `make docker-buildx` - Build and push for cross-platform support
- `make deploy` - Deploy controller to K8s cluster
- `make undeploy` - Remove controller from K8s cluster

**Installation:**
- `make install` - Install CRDs into K8s cluster
- `make uninstall` - Uninstall CRDs from K8s cluster

**Why use Make instead of bare commands:**
- Ensures KUBEBUILDER_ASSETS environment is properly configured
- Runs code generation, formatting, and vetting before tests/builds
- Downloads and manages tool versions (controller-gen, setup-envtest, kustomize)
- Maintains consistency across different development environments

## Development History

### 2025-12-31: Refactored to Use JFrog Go Client SDK

Refactored the codebase to use the official JFrog Go client library instead of manual HTTP client implementation.

#### Changes Made

**1. Dependencies** ([go.mod](go.mod))
- Added `github.com/jfrog/jfrog-client-go v1.55.0`
- All transitive dependencies automatically resolved via `go mod tidy`

**2. Controller Refactoring** ([internal/controller/serviceaccount_controller.go](internal/controller/serviceaccount_controller.go))
- **Before:** `DefaultJFrogClient` used manual HTTP client with custom request handling
- **After:** Uses SDK's `access.AccessServicesManager`
- Removed ~30 lines of manual HTTP request construction, URL encoding, and JSON parsing
- `ExchangeToken` method now uses SDK's `ExchangeOidcToken` API
- Properly handles SDK's `auth.OidcTokenResponseData` with embedded `CommonTokenParams`
- Handles SDK's pointer-based `ExpiresIn` field (`*uint`)

**3. Main Initialization** ([cmd/main.go](cmd/main.go))
- Created `AccessDetails` using `accessAuth.NewAccessDetails()`
- Built service config with `clientConfig.NewConfigBuilder()`
- Initialized Access Manager using `access.New(serviceConfig)`
- Removed manual HTTP client and TLS configuration (SDK handles this)

**4. Test Updates** ([internal/controller/serviceaccount_controller_test.go](internal/controller/serviceaccount_controller_test.go))
- Removed HTTP-level mock tests (SDK handles this internally)
- Kept high-level `MockJFrogClient` integration tests
- Tests pass with 72.1% code coverage

#### Benefits

- **Maintainability:** Using official SDK means automatic updates and bug fixes
- **Reliability:** SDK is tested and maintained by JFrog
- **Simplicity:** Removed manual HTTP/JSON handling code
- **Type Safety:** SDK provides proper type definitions and validation
- **Future-proof:** Easy to adopt new SDK features as they're released

#### Testing

All tests pass successfully:
```bash
make test
# ✅ internal/controller: 72.1% coverage
# ✅ All formatting and linting checks pass
```

#### SDK Documentation

- **GitHub:** https://github.com/jfrog/jfrog-client-go
- **Go Package:** https://pkg.go.dev/github.com/jfrog/jfrog-client-go
- **Official Docs:** https://docs.jfrog-applications.jfrog.io/ci-and-sdks/sdks/jfrog-go-client

#### Key SDK Types Used

- `access.AccessServicesManager` - Main SDK manager for Access API operations
- `services.CreateOidcTokenParams` - Parameters for OIDC token exchange
- `auth.OidcTokenResponseData` - Response from token exchange
- `auth.CommonTokenParams` - Common token fields (AccessToken, ExpiresIn, Scope, TokenType)

#### Migration Notes for Future Reference

When working with the SDK:
1. `ExpiresIn` is returned as `*uint` (pointer to unsigned int), convert to `int64` for our API
2. Response type is `auth.OidcTokenResponseData` with embedded `CommonTokenParams`
3. The SDK handles all HTTP-level details (TLS, retries, error handling)
4. No need to manually construct URLs or handle URL encoding - SDK does this
