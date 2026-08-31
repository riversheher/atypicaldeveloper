---
title: "Use Functional Options (and teach your AI to) "
description: Why you should teach your AI contributor one simple Go constructor idiom.
date: 2026-08-30T20:29:35-05:00
image: https://go.dev/doc/gopher/gopher5logo.jpg
math: 
license: 
tags: [
    "tech",
    "coding"
]
categories: [
    "Work"
]
comments: false
draft: false
---
You open a repository with a spark in your eye, ready to make your first contribution of the day. Your gaze lands on the first line of code, then the second, the third, and before you make it to the fourth, the laptop screen slams closed. Reality sinks in; this is not the repository you signed up for.

We've all been there. Code quality is in short supply, seemingly especially so at your workplace. Nod if you've seen these before: overloaded/telescoping constructors, endless configs with ambiguous defaults, or unidiomatic builder patterns.

```go
// 😵 the "eyes glaze over" special
func NewRequestSimple(model string, msg string) *Request {
    r := &Request{Model: model, Message: msg}
    r.Debug = false
    r.Temperature = 1.0
    r.Retries = 1
    r.Timeout = 100 * time.Second
    return r
}


func NewRequest(model string, msg string, debug bool) *Request {
    r := &Request{Model: model, Message: msg}
    r.Debug = debug
    r.Temperature = 1.0
    r.Retries = 1
    r.Timeout = 100 * time.Second
    return r
}


func NewRequestFull(model string, msg string, debug bool, temp float64, retries int, timeout int) *Request {
    r := &Request{Model: model, Message: msg}
    r.Debug = debug
    r.Temperature = temp
    r.Retries = retries
    r.Timeout = time.Duration(timeout) * time.Second
    return r
}


// what is `true`? what is `100`?
req := NewRequestFull("deepseek-v4-flash", "hi", true, 1.0, 1, 100)
```

Most of these issues come from codebases that have been extended, modified, and maintained over the years. In fact, the codebase might have seemed clean and simple at the time of the first commit. Maybe that 50 line config was just 5 lines two years ago. Maybe there was only one clean constructor back then too. Or maybe you just came over from Java and were new to Go back then — now you're stuck with some design decisions you made five years ago. With the rise of agentic coding, we have a whole new vector for these issues to surface: your friendly neighbourhood coding harness.

These situations could have been avoided in one simple way: _Using functional options_.

## What are _Functional Options_?

The _functional options_ pattern is a way of writing constructors with extensibility, readability, and usability in mind. The core of this pattern, as the name suggests, is the functional option. This is a closure, making use of one of Go's best features: first-class functions. Each option is a closure that mutates the structure we are composing. That might be a server configuration, a complex response/request body, a data entry with a pile of defaults, an enemy in a procedurally generated game, or whatever your mind desires.

The end product is a constructor that uses composition like this:

```go
// The end result — named decisions, not positional noise
req := NewChatRequest(
    WithModel("deepseek-v4-flash"),
    WithMessage("Explain _functional options_ in Go"),
)

// A variant with a couple of knobs turned
req := NewChatRequest(
    WithModel("glm-5.3-flash"),
    WithMessage("Write a haiku about closures"),
    WithTemperature(0.8),
    WithDebug(true),
)
```

It solves the issue of setting defaults without the need for complex configuration structs. It predicts future extension by baking it into design of the constructor on day 1. It allows for composability over many successive instances. All this while maintaining readability and simplicity. Your future self and your AI assistant will thank you.

To achieve this, we define the constructor signature to accept an arbitrary number of options:

```go
// The constructor accepts an arbitrary number of Option values
func NewChatRequest(opts ...Option) *ChatRequest {
    // ...
}
```

Then within the constructor, we iterate through the options, applying each one to the struct we are composing:

```go
func NewChatRequest(opts ...Option) *ChatRequest {
    req := &ChatRequest{
        Model:   "deepseek-v4-flash",   // sensible defaults
        Retries: 3,
        Timeout: 30 * time.Second,
    }
    for _, opt := range opts {
        opt(req)    // each closure mutates the request in place
    }
    return req
}
```

This allows us to define an option as a closure that mutates our desired struct. It can be something simple like setting a timeout:

```go
type Option func(*ChatRequest)

func WithTimeout(d time.Duration) Option {
    return func(req *ChatRequest) {
        req.Timeout = d
    }
}

func WithRetries(n int) Option {
    return func(req *ChatRequest) {
        req.Retries = n
    }
}
```

Or a complex example where one change cascades into a handful of others because of the API spec:

```go
// WithModel is more than a setter — it knows the API spec and
// sets several dependent fields at once.
func WithModel(m string) Option {
    return func(req *ChatRequest) {
        req.Model = m
        switch m {
        case "deepseek-v4-flash":
            req.MaxTokens = 8192
            req.Temperature = 0.7
        case "glm-5.3-flash":
            req.MaxTokens = 4096
            req.Temperature = 0.9
        }
    }
}
```

They can even return errors as a validation step:

```go
type Option func(*ChatRequest) error

func WithRetries(n int) Option {
    return func(req *ChatRequest) error {
        if n < 0 {
            return fmt.Errorf("retries must be >= 0, got %d", n)
        }
        req.Retries = n
        return nil
    }
}

func NewChatRequest(opts ...Option) (*ChatRequest, error) {
    req := &ChatRequest{Retries: 3}
    for _, opt := range opts {
        if err := opt(req); err != nil {
            return nil, err
        }
    }
    return req, nil
}
```

Now, when the next set of features rolls in, the complexity of your project doesn't have to grow with it. With functional options, you've managed the complexity creep and dodged the thing we all dread most... opening a repository and feeling your eyes go blank

## Why It Wins

There are a few key benefits worth taking a closer look at:
- Extensibility
- Readability
- Defaults
- Composability

Lets say you define a simple data structure to represent the request body of your new service. For example, in the beginning, all you needed to implement was: **model choice**, **input message**, and a **debug** flag that is defaulted to off:

```go
type ChatRequest struct {
    Model   string
    Message string
    Debug   bool
}

func NewChatRequest(opts ...Option) *ChatRequest {
    req := &ChatRequest{Model: "deepseek-v4-flash", Debug: false}
    for _, opt := range opts {
        opt(req)
    }
    return req
}
```

Now your PM comes back and tells you to add support for temperature and a whole host of other fields that need defaults. Well, good news! Because you used functional options, it's as simple as writing a new option and composing it into your constructor.

```go
type ChatRequest struct {
    Model       string
    Message     string
    Debug       bool
    Temperature float64
    MaxTokens   int
    Retries     int
    Timeout     time.Duration
}

func NewChatRequest(opts ...Option) *ChatRequest {
    req := &ChatRequest{
        Model:       "deepseek-v4-flash",
        Message:     "",
        Debug:       false,
        Temperature: 0.7,
        MaxTokens:   8192,
        Retries:     3,
        Timeout:     30 * time.Second,
    }
    for _, opt := range opts {
        opt(req)
    }
    return req
}

// Existing callers — unchanged. This is the extensibility payoff.
NewChatRequest(WithMessage("hi"))
NewChatRequest(WithModel("glm-5.3-flash"), WithTemperature(0.9))
```

This isn't just extensible, it's readable. When you land in a codebase, you can see exactly what's being set explicitly. And it lets us set defaults while sidestepping the "unset values default to 0" gotcha you hit with config structs.

Using a config struct, a user may set the model choice and message but choose to use the default for retry attempts:

```go
type Config struct {
    Model   string
    Message string
    Retries int   // 0 means... default retries? or zero retries?
}

cfg := Config{Model: "deepseek-v4-flash", Message: "hi"}
// Was the user relying on the default, or deliberately asking for 0?
// You can't tell — they're the same value.
```

How do we tell whether the user wants the default number of retries, or genuinely zero? There are workarounds, but functional options completely sidestep that concern:

```go
func NewChatRequest(opts ...Option) *ChatRequest {
    req := &ChatRequest{
        Model: "deepseek-v4-flash", 
        Retries: 3
    }   // the default lives here
    for _, opt := range opts {
        opt(req)
    }
    return req
}

// Using the default retries — nothing set
NewChatRequest(WithMessage("hi"))

// Explicit zero retries — genuinely different
NewChatRequest(WithMessage("hi"), WithRetries(0))
```

Our default lives right there in the constructor, and the user can call the option to change it. No more playing with zeros or extra flags.

Finally, it's composable and reusable. When you need to create a lot of the same thing with small variations, this really wins. You can build a slice of options, apply them to a set of constructors, then append to it for the next one:

```go
// A reusable bundle — every request gets these
var baseOpts = []Option{
    WithModel("deepseek-v4-flash"),
    WithTemperature(0.7),
    WithTimeout(30 * time.Second),
}

// Spread it in, then add extras on top
func newDefaultRequest(msg string) *ChatRequest {
    return NewChatRequest(append(baseOpts, WithMessage(msg))...)
}

func newCreativeRequest(msg string) *ChatRequest {
    return NewChatRequest(append(baseOpts, WithMessage(msg), WithTemperature(1.2))...)
}
```

## When It Wins

**When creating a library:**
- Used in major packages like http.Server, http.Client, log.Logger, OpenTelemetry and many other common dependencies.
- It is one of the best ways to allow developers to interface with complex internal systems.

**When configuring components that require multiple variants:**
- Workers, Handlers, Requests, etc.

**TESTS TESTS TESTS**
- Within tests, you're constantly constructing test data. With unit tests, this can mean a lot of default values turning one knob at a time. This is one of the best use cases for this pattern.

```go
// A small fixture factory — internal to the test package
type testUser struct {
    Name string
    Role Role
}

type userOption func(*testUser)

func withName(n string) userOption      { return func(u *testUser) { u.Name = n } }
func withRole(r Role)  userOption       { return func(u *testUser) { u.Role = r } }

func testUser(opts ...userOption) *testUser {
    u := &testUser{Name: "alice", Role: RoleUser}   // defaults
    for _, opt := range opts {
        opt(u)
    }
    return u
}

func TestAdminHasElevatedAccess(t *testing.T) {
    admin := testUser(withRole(RoleAdmin))          // everything else defaulted
    // ...
}

func TestUserCanExport(t *testing.T) {
    exporter := testUser(withRole(RoleViewer), withName("bob"))
    // ...
}
```

## When It's Not the Best Choice

When you need compile-time required fields, functional options can fall short. As the name says, they're options. In that case, you can use a hybrid: positional arguments for the required fields, and functional options for the rest.

```go
// Required fields are positional; options handle the optional extras
func NewChatRequest(model, message string, opts ...Option) (*ChatRequest, error) {
    if model == "" || message == "" {
        return nil, errors.New("model and message are required")
    }
    req := &ChatRequest{Model: model, Message: message, Retries: 3}
    for _, opt := range opts {
        if err := opt(req); err != nil {
            return nil, err
        }
    }
    return req, nil
}

// Usage
req, err := NewChatRequest("deepseek-v4-flash", "hi", WithTimeout(10*time.Second))
```

Furthermore, functional options can introduce order-dependent bugs. That risk climbs when you have complex requirements with dependencies between fields that don't compose cleanly on their own. In that case, a config struct with a validation method might be the way to go.

The bottom line: functional options aren't a one-size-fits-all solution. You still have to use that software engineering brain! But in a lot of cases this pattern does fit the bill; you just have to be careful when it doesn't.

## Teach Your AI

As we all know, a lot of code is generated by LLMs these days. They default to their training, but also to the context of your codebase. Sometimes that leads to a compounding of issues if the codebase's quality is already struggling. Other times, our AI assistants "helpfully" refactor our codebase in response to a prompt, and it's not until much later that we realize that was probably a mistake.

The tools to fix this already exist! Most agentic harnesses like Claude Code, OpenCode, and GitHub Copilot support agent skills. These are pieces of context injected into our prompts or agent loops that tell our agents how to act.

I've created a simple skill you can grab here: [Skill File](https://github.com/riversheher/atypicaldeveloper/blob/master/resources/downloadable/Agent_Skill_Functional_Options_Pattern.md)! You can add this to your own harness! dd it to your own harness! Another skill I highly recommend is the golang-pro skill from the popular claude-skills repo: [Github](https://github.com/Jeffallan/claude-skills/tree/main/skills/golang-pro)