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

Most of these issues come from codebases that have been extended, modified, and maintained over the years. In fact, the codebase might have seemed clean and simple at the time of the first commit. Maybe that 50 line config was just 5 lines two years ago. Maybe there was only one clean constructor back then too. Or maybe you just came over from Java and were new to Go back then, and now you're stuck with some design decisions you made five years ago. Now, with the advent of agentic coding, we have a whole new vector for these issues to surface: your friendly neighbourhood coding harness.

These situations could have been avoided in one simple way: _Using functional options_.

## What are _Functional Options_?

The _functional options_ pattern is a way of writing constructors with extensibility, readability, and usability in mind. The core of this pattern, as the name suggests, is the functional option. This is a closure making use of a core feature in Go: first-class functions. Each option is a closure that mutates the structure we are composing. That might be a server configuration, a complex response/request body, a data entry with many default fields, an enemy in a procedurally generated game, or whatever your mind desires.

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

In order to achieve this, we can define the constructor signature to accept an arbitrary amount of options:

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

Or a complex example where one change may cascade into multi-line changes due to the API spec.

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

Now, when the next set of features comes, the complexity of your project does not need to grow with it. With _functional options_, you have effectively managed that complexity creep and avoided the thing we most dread... opening a repository and having our eyes go blank.

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

Now, your PM comes back and tells you to add support for temperature and a variety of other fields that require default values. Well good news, because you used _functional options_, its as simple as creating an option and composing it into your constructor.

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

This is not only extensible, it is also readable. When you enter a codebase you can easily see exactly what is being set explicitly. Secondarily, this allows us to set defaults by bypassing the "unset values default to 0" issue that can occur with configuration using structs.

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

How do we determine if the user wants a default amount of retries or genuinely 0? There are workarounds, but the use of _functional options_ completely bypasses this concern:

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

Our default is explicitly defined within the constructor, and the user can explicitly use the functional option to change it. No more playing with zeros or extra flags.

Lastly, it is composable and reusable. When you need to create many of the same structure with small variations, this wins. You can create a slice of options to apply to a set of constructors then append to it when applying to another constructor easily:

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

When you need compile-time required fields, _functional options_ may be lacking. As their name suggests, they are options. In this case, we can use a hybrid approach with positional arguments for required fields and _functional options_ for others.

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

Furthermore, _functional options_ can introduce order dependent bugs. This risk is elevated when there are complex configuration requirements with dependencies between fields that are not easily separately composable. In this case, a config struct and a validation method may be the way to go.

The bottom line is, _functional options_ aren't a one size fits all solution. You still have to use that software engineering brain! However, in a lot of cases, this pattern does fit the bill, you just have to be careful when it doesn't.

## Teach Your AI

As we all know, a lot of code is being generated by LLMs nowadays. These default to their training, but also the context of your codebase. Sometimes, this can lead to a compounding of issues if the code quality in the codebase is already struggling. Other times, our AI assistants may try to "helpfully" refactor our codebase to our prompting, and its not until much later that we realize that was probably a mistake.

The tools we can use to fix all this already exist! Most harnesses we use for agentic coding such as Claude Code, OpenCode, or Github Copilot, support agent skills! These are pieces of context that are injected into our prompts or agentic loops that instruct our agents how to act.

I have created a simple skill that you can find here: [Skill File](https://github.com/riversheher/atypicaldeveloper/blob/master/resources/downloadable/Agent_Skill_Functional_Options_Pattern.md)! You can add this to your own harness! Another skill that I highly suggest is the golang-pro skill found in the popular claude-skills repository! You can find that here: [Github](https://github.com/Jeffallan/claude-skills/tree/main/skills/golang-pro)