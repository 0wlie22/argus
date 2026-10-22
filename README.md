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
    GeminiAPI[("External Gemini API\n(Google GenAI SDK)")]

    %% Subgraph: Proxy Package
    subgraph ProxyPkg ["package proxy"]
        Router["router.go\n(httputil.ReverseProxy :8080)"]
        Inspector["inspector.go\n(Canary Traps & WAF Signatures)"]
    end

    %% Subgraph: Cache Package
    subgraph CachePkg ["package cache"]
        MemoryCache[("memory.go\n(IP Threat Scores & MD5 Payload Cache)")]
        SessionCache[("session.go\n(Attacker Context Ring Buffer)")]
    end

    %% Subgraph: Honeypot Package
    subgraph HoneypotPkg ["package honeypot"]
        AIEngine["ai.go\n(Async Gemini Client)"]
        PromptEng["prompt.go\n(Persona Prompts & Response Stripper)"]
    end

    %% Subgraph: App Package
    subgraph AppPkg ["package app"]
        RealApp["server.go\n(Legitimate App Target :8081)"]
    end

    %% Subgraph: UI Package (Optional)
    subgraph UIPkg ["package ui (Optional)"]
        EventBus["event.go\n(Telemetry Channel Listener)"]
        Dashboard["dashboard.go\n(rivo/tview 3-Pane Terminal UI)"]
    end

    %% Execution Flow
    Client -->|"1. HTTP Request"| Router
    Router -->|"2. Inspect Request"| Inspector
    Inspector <-->|"3. Lookup/Update IP Score"| MemoryCache

    %% Routing Decisions
    Inspector -->|"4a. Score < 50 (Legitimate)"| RealApp
    Inspector -->|"4b. Score >= 50 or Canary (Malicious)"| AIEngine

    %% Honeypot Pipeline
    AIEngine <-->|"5a. Check MD5 Cache"| MemoryCache
    AIEngine <-->|"5b. Fetch/Update Session Context"| SessionCache
    AIEngine -->|"6. Format Context"| PromptEng
    PromptEng -->|"7. Query LLM"| GeminiAPI
    GeminiAPI -->|"8. Synthetic Response"| Router
    Router -->|"9. Stream Decoy Payload"| Client

    %% Async Telemetry Stream (Optional)
    Router -.->|"Async Event"| EventBus
    EventBus -->|"Render Updates"| Dashboard

    %% Styling - Dark Mode & High Contrast Safe
    style Client fill:#2d3748,stroke:#cbd5e0,color:#ffffff,stroke-width:2px
    style GeminiAPI fill:#1e3a8a,stroke:#60a5fa,color:#ffffff,stroke-width:2px
    style RealApp fill:#064e3b,stroke:#34d399,color:#ffffff,stroke-width:2px
    
    %% Optional Component Styling
    style UIPkg fill:#1e1b4b,stroke:#818cf8,color:#ffffff,stroke-width:2px,stroke-dasharray: 5 5
    style Dashboard fill:#431407,stroke:#fb923c,color:#ffffff,stroke-width:1px,stroke-dasharray: 3 3
    style EventBus fill:#431407,stroke:#fb923c,color:#ffffff,stroke-width:1px,stroke-dasharray: 3 3
```
