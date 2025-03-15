# Circuit Breaker

## Architecture

### Core Components

1. **Circuit Breaker Core** 

   - State management
   - Failure tracking
   - Retry logic
   - Thread safety

2. **HTTP Middleware** 

   - Request interception
   - Response handling
   - Per-endpoint circuit breaker management

3. **Server Integration** 
   - API gateway setup
   - Route configuration
   - Circuit breaker initialization

## Configuration

The circuit breaker can be configured with the following parameters:

```go
type Config struct {
    FailureThreshold    int          
    ResetTimeout       time.Duration 
    MaxRetries         int         
    InitialBackoff     time.Duration 
    MaxBackoff         time.Duration
    BackoffMultiplier  float64     
}
```

### Default Configuration

```go
Config{
    FailureThreshold:   5,
    ResetTimeout:      10 * time.Second,
    MaxRetries:        3,
    InitialBackoff:    100 * time.Millisecond,
    MaxBackoff:        2 * time.Second,
    BackoffMultiplier: 2.0,
}
```

## Usage

### Basic Integration

```go
cbConfig := circuitbreaker.Config{
    FailureThreshold:   5,
    ResetTimeout:      10 * time.Second,
    MaxRetries:        3,
    InitialBackoff:    100 * time.Millisecond,
    MaxBackoff:        2 * time.Second,
    BackoffMultiplier: 2.0,
}

cbMiddleware := middleware.NewCircuitBreakerMiddleware(cbConfig)

router.Handle("/api/service", cbMiddleware.Middleware(yourHandler))
```

### States and Behavior

1. **Closed State**

   - Normal operation
   - Tracking failures
   - Switches to Open when FailureThreshold is reached

2. **Open State**

   - Fails fast
   - Returns 503 Service Unavailable
   - Transitions to Half-Open after ResetTimeout

3. **Half-Open State**
   - Allows test requests
   - Returns to Closed on success
   - Returns to Open on failure
