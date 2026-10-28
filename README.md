# ARGUS 

Anomalous Request Gateway and URI Simulator - intelligent network proxy build in Go. Instead of passively blocking attackers or routing them to static and easily identifiable honeypot ARGUS intercepts malicious traffic and uses LLMs to synthesize realistic fake server responses on the fly. It traps attackers into an endless digital mirage, wasting their resources while safely capturing threat intelligence.

## File structure:

```
argus/
├── main.go
├── config/
│   └── config.go         # API keys, upstream target URLs, canary route list
├── proxy/
│   ├── router.go         # Reverse proxy handler logic
│   └── inspector.go      # Canary, regex, and IP threat scoring engine
├── honeypot/
│   ├── ai.go             # Gemini API integration
│   └── prompt.go         # System persona prompts and template rendering
├── cache/
│   └── memory.go         # Thread-safe in-memory cache for generated responses
│   └── sessions.go       # Session logic for LLM model
├── ui/ [OPTIONAL]
│   ├── dashboard.go      # rivo/tview 3-pane layout & render loop
│   └── event.go          # Telemetry struct definition & channel listener
└── app/
    └── server.go         # Simple dummy server serving "legitimate" app responses
```

## Deliverables

1. Proxy gateway
2. Threat inspector
3. Client state tracker
4. AI-deception engine
5. Response cache system
6. Upstream production forwarder

## Architecture

```mermaid
flowchart TD
    %% External Actors & APIs
    Client([Client / Attacker Traffic])
    GroqAPI[("External Groq API\n(OpenAI-compatible Chat API)")]

    %% Subgraph: Proxy Package
    subgraph ProxyPkg ["package proxy"]
        Router["Gateway Router\n(entrypoint + routing)"]
        ThreatEngine["Threat Engine\n(resolve, inspect, score)"]
        ThreatInspector["Threat Inspector\n(canaries + signatures)"]
        IPTracker["IP Threat Tracker\n(threshold + trap state)"]
        IPResolver["Client IP Resolver\n(peer / XFF trust)"]
    end

    %% Subgraph: Cache Package
    subgraph CachePkg ["package cache"]
        ResponseCache[("Response Cache\n(replay synthetic replies)")]
        SessionCache[("Session Memory Cache\n(attacker interaction history)")]
    end

    %% Subgraph: Honeypot Package
    subgraph HoneypotPkg ["package honeypot"]
        HoneypotHandler["Honeypot Response Handler\n(session + cache orchestration)"]
        AIEngine["AI Response Engine\n(LLM call + normalization)"]
        Prompting["Prompt Builder\n(stack-aware prompts)"]
        Fallback["Fallback Response Builder\n(static safe responses)"]
    end

    %% Subgraph: App Package
    subgraph AppPkg ["package app"]
        RealApp["Target Application\n(legitimate backend)"]
    end

    %% Subgraph: UI Package (Optional)
    subgraph UIPkg ["package ui (Optional)"]
        EventBus["event.go\n(Telemetry Channel Listener)"]
        Dashboard["dashboard.go\n(rivo/tview 3-Pane Terminal UI)"]
    end

    Reject["Safety Rejection\n(400 / 413 / 415)"]

    %% Execution Flow
    Client -->|"1. HTTP Request"| Router
    Router -->|"2. Resolve + Inspect + Score"| ThreatEngine
    ThreatEngine --> ThreatInspector
    ThreatEngine --> IPTracker
    ThreatEngine --> IPResolver

    %% Routing Decisions
    Router -->|"3a. Clean / below threshold"| RealApp
    Router -->|"3b. Trapped IP"| HoneypotHandler
    Router -->|"3c. Uninspectable body"| Reject

    %% Honeypot Pipeline
    HoneypotHandler -->|"4a. Session context"| SessionCache
    HoneypotHandler -->|"4b. Cache lookup/store"| ResponseCache
    HoneypotHandler -->|"4c. Generate synthetic response"| AIEngine
    AIEngine --> Prompting
    AIEngine --> GroqAPI
    AIEngine -->|"error path"| Fallback

    RealApp -->|"5. Legitimate response"| Client
    HoneypotHandler -->|"5. Deceptive response"| Client
    Reject -->|"5. Reject response"| Client

    %% Async Telemetry Stream (Optional)
    Router -.->|"Async Event"| EventBus
    EventBus -->|"Render Updates"| Dashboard

    %% Styling - Dark Mode & High Contrast Safe
    style Client fill:#2d3748,stroke:#cbd5e0,color:#ffffff,stroke-width:2px
    style GroqAPI fill:#1e3a8a,stroke:#60a5fa,color:#ffffff,stroke-width:2px
    style RealApp fill:#064e3b,stroke:#34d399,color:#ffffff,stroke-width:2px

    %% Optional Component Styling
    style UIPkg fill:#1e1b4b,stroke:#818cf8,color:#ffffff,stroke-width:2px,stroke-dasharray: 5 5
    style Dashboard fill:#431407,stroke:#fb923c,color:#ffffff,stroke-width:1px,stroke-dasharray: 3 3
    style EventBus fill:#431407,stroke:#fb923c,color:#ffffff,stroke-width:1px,stroke-dasharray: 3 3
```

