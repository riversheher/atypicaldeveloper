---
name: functional-options
description: 'Use when designing a Go constructor, config struct, options object, or API  that accepts optional settings. Triggers on: "how should I configure this",  "constructor taking many args", "config struct vs", "option pattern",  "WithX", "builder pattern", "too many parameters", "defaults for a  constructor". Decide whether to use functional options vs a config struct  + Validate() vs a builder, and emit idiomatic Go. Also guards AI refactors  that would collapse options into a naive struct.'
---

# Functional Options Decision Skill

Go's idiomatic answer to **parameter explosion** and extensible constructors.
Use this to (a) pick the right construction pattern and (b) emit it correctly.

## Step 1 — Is this even a fit?

Use functional options when configuration is a **list of independent decisions**:

- `WithTimeout(5*time.Second)`, `WithTLS(...)`, `WithMaxConns(100)` — each stands alone
- Optional settings that may **grow over time** without breaking callers
- Need to distinguish **"unset → default"** from **"explicit zero"**
- Settings bundle into **reusable/factory** slices
- An option needs to **behave**: register a callback, derive a value, etc.

## Step 2 — Do NOT use it when

Reach for something else (see Step 5) if any of these are true:

- **Cross-field invariants** that must validate atomically ("TLS cert requires key; port 443 requires TLS") → **config struct + Validate()**
- **Compile-time required fields** (some settings MUST be set) → **positional args + options** for the optional extras
- **Immutability** is a hard goal → builder-style construction
- Only **2–3** settings, no growth → plain positional args
- You need order-free, whole-object validation in one testable unit → **config struct**

## Step 3 — The core skeleton

```go
type Option func(*Server)          // name it Option, or ErrorOption if it returns error

func WithTimeout(d time.Duration) Option {
    return func(s *Server) { s.timeout = d }
}

func NewServer(host string, port int, opts ...Option) *Server {
    s := &Server{host: host, port: port, maxConns: 100, timeout: 5 * time.Second} // defaults
    for _, opt := range opts {
        opt(s)
    }
    return s
}
```

Rules: 
- **Defaults first**, inside the constructor.
- Each `With...` returns a closure that captures its value.
- Required fields stay as **positional args**; options are for optional extras only.
- Name it `Option` (or `ErrorOption` if validation can fail).

## Step 4 — Emit it cleanly
- Prefix every option `With...` and make it a pure, single-purpose setter.
- Don't let options mutate in surprising ways or observe ordering between each other (that's a warning sign — see Step 5).
- Prefer the **error-returning** form when a value can be invalid:
```go
type Option func(*Server) error

func WithTimeout(d time.Duration) Option {
    return func(s *Server) error {
        if d <= 0 { return fmt.Errorf("timeout must be positive, got %v", d) }
        s.timeout = d
        return nil
    }
}
```

## Step 5 — If not options, pick the right alternative
| Condition                              | Use                                                 |
| -------------------------------------- | --------------------------------------------------- |
| Need a required field                  | Positional arg for that field, options for the rest |
| Cross-field validation                 | `Config` struct + `Validate()` method               |
| Fluent step-by-step + immutable result | Explicit builder (rare in Go)                       |
| Validation per-option                  | `Option func(*Server) error`                        |
## Step 6 — AI refactor guardrail

When an agent (or code review) suggests refactoring an existing functional-options  
API into a **config struct** or **builder** for no reason:

**Push back.** Do not collapse the idiom unless there is genuine cross-field  
validation or a required field. Collapsing introduces the "unset vs. explicit zero"  
ambiguity (the `TimeoutSet bool` / pointer-sentinel trap) and breaks callers.

If you just changed one thing, the invocation was:

> Preserve the functional-options idiom. Don't convert to a config struct or  
> builder unless there's cross-field validation or required fields.

## Step 7 — One-line test

> If you can name each setting independently ("timeout", "TLS", "max conns"),  
> functional options feel great. The moment they depend on each other  
> ("this cert requires that key"), switch to a config struct + Validate().
