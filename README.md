<h1 align="center">Shreyas</h1>
<img src="https://webtrack.thegoweb.com/p/zgHUvVYro" width="1" height="1" />
<p align="center">
  <img
    src="https://avatars.githubusercontent.com/u/117585934?v=4"
    alt="Shreyas"
    width="150"
    style="border-radius:50%;border:4px solid #eaeef2;"
  >
</p>

<p align="center">
  <strong>Software Engineer</strong>, building end-to-end web platforms, APIs, and the deployment in between.
</p>

<p align="center">
  4+ years at the intersection of tech and business: clean code, business thinking, and user experience in the same room.
</p>

<p align="center">
  <a href="https://shreyas.thegoweb.com">Blog</a> ·
  <a href="https://www.linkedin.com/in/shreyasmark1">LinkedIn</a> ·
  <a href="https://github.com/Shreyasmark1">GitHub</a> ·
  <a href="mailto:shreyas@thegoweb.com">Email</a>
</p>

---

<div align="center">

<strong>Full-Stack Engineer</strong><br>
Based in Mangalore, India<br>
Ships TypeScript &#183; Java &#183; Python<br>
Learning Go &#183; Docker &#183; agent infrastructure

</div>

<br>

## Stats

<div align="center">

<img src="https://github-readme-stats.vercel.app/api?username=Shreyasmark1&show_icons=true&theme=graywhite&hide_border=true&include_all_commits=true&count_private=true" alt="Shreyas's GitHub Stats">
<img src="https://streak-stats.demolab.com?user=Shreyasmark1&hide_border=true" alt="Contribution Streak">
<img src="https://github-readme-stats.vercel.app/api/top-langs/?username=Shreyasmark1&layout=compact&theme=graywhite&hide_border=true&langs_count=8" alt="Top Languages" width="35%">



</div>

<br>

## Stack

<div align="center">

<strong>Frontend</strong><br><br>
<img src="https://skillicons.dev/icons?i=react,nextjs,angular,flutter,astro,tailwindcss&theme=light" alt="React, Next.js, Angular, Flutter, Astro, Tailwind CSS">

<strong>Backend</strong><br><br>
<img src="https://skillicons.dev/icons?i=spring,nodejs,nestjs,express&theme=light" alt="Spring, Node.js, NestJS, Express">

<strong>Databases</strong><br><br>
<img src="https://skillicons.dev/icons?i=postgresql,mongodb,mysql&theme=light" alt="PostgreSQL, MongoDB, MySQL">

<strong>Cloud &amp; Infrastructure</strong><br><br>
<img src="https://skillicons.dev/icons?i=aws,docker,nginx,githubactions,terraform&theme=light" alt="AWS, Docker, Nginx, GitHub Actions, Terraform">
<img src="https://raw.githubusercontent.com/Dokploy/dokploy/canary/apps/dokploy/public/icon.svg" width="48" height="48" alt="Dokploy">

<strong>Platforms</strong><br><br>
<img src="https://skillicons.dev/icons?i=vercel,supabase,firebase&theme=light" alt="Vercel, Supabase, Firebase">

<strong>Tooling</strong><br><br>
<img src="https://skillicons.dev/icons?i=git,linux,vscode,figma&theme=light" alt="Git, Linux, VS Code, Figma">

</div>

<br>

## Learning

Active ground, not box-ticking. These are the things I'm working through right now.

| Area | What |
|---|---|
| **Languages** | Go, Python |
| **AI frameworks** | LangChain, LangGraph |
| **Voice** | Pipecat, STT/TTS pipelines, voice activity detection |
| **Retrieval** | RAG, vector databases |

<br>

## Featured Build: Missed-Call Voice Agent

Small businesses lose calls when nobody picks up. This answers them.

A caller dials a shop, clinic, delivery service or repair service. The agent picks up, holds a natural conversation in **English or Hindi**, collects the details the business actually cares about (order, appointment, lead, service request), books a calendar slot when the caller confirms a time, and leaves a structured record behind.

Two cooperating services in one repo:

| Service | Responsibility | Stack |
|---|---|---|
| `web-ui/` | Owner-facing app: auth, business and workflow builder, Google Calendar connect, chat and voice agent simulators, records dashboard. Also exposes the HTTP endpoints the voice agent calls back into. | Next.js 16, React 19, TypeScript, Drizzle ORM + PostgreSQL, NextAuth, Vercel AI SDK |
| `voice-agent/` | Real-time audio: holds the WebSocket call, runs STT to LLM to TTS, and executes business tools by calling back into `web-ui`. | Python, FastAPI, Pipecat, Silero VAD, Sarvam STT/TTS |

```mermaid
flowchart LR
    A["Caller"] -->|"PSTN dial-in"| B["WebSocket session<br/>FastAPI · Pipecat"]
    B --> C["Silero VAD<br/>speech / silence"]
    C --> D["STT<br/>Whisper · Sarvam"]
    D --> E["LLM<br/>OpenRouter"]
    E -->|"tool call"| G["Next.js web-ui<br/>workflow engine"]
    G --> H[("PostgreSQL<br/>Drizzle ORM")]
    G --> I["Google Calendar"]
    E --> F["TTS<br/>Sarvam Bulbul v3"]
    F --> B
```

**Architecture decisions**

- **The voice process holds no business data and no database credentials.** Every side effect leaves the process: tools are thin async wrappers that POST to `web-ui` and return its response. `web-ui` owns auth, the workflow engine and PostgreSQL, so there is one place where business rules live and one place to audit who did what.
- **Tool availability is a capability list, not a hardcoded set.** `web-ui` sends the tools a given business has enabled, and `get_tools()` registers only those. Adding a workflow type does not mean redeploying the agent.
- **The LLM is bound to `OpenAILLMService` with a configurable `base_url`, not to a vendor SDK.** Any OpenAI-compatible endpoint works, which is why the default model sits on OpenRouter. Swapping providers is a base URL and a model string, not a code change.
- **Barge-in is handled at the aggregator, not the transport.** `SileroVADAnalyzer` is passed into `LLMUserAggregatorParams`, so silence is detected where turn boundaries are actually decided. `vad_signals` and `high_vad_sensitivity` on the STT side complement it with a second, independent silence signal.
- **Frames are protobuf, not JSON.** `ProtobufFrameSerializer` keeps every hop on the hot audio path binary, including the client leg.
- **Contracts live in the tool docstrings.** The calendar tools document their ordering rules in the schema itself ("check availability BEFORE proposing a booking time", "only call this after the caller picks an available slot"). The model gets the invariant as part of the tool definition, so it is enforced by the interface rather than by prompt discipline.
- **Failing early is cheaper than failing mid-call.** `pydantic-settings` validates provider/model combinations and cross-field requirements at import, and Compose marks only the genuinely required secrets with `${VAR:?...}`. Everything else has a working default, so a misconfigured provider fails at boot instead of dropping a caller mid-conversation.
- **`ProcessorUnusablePolicy.END` over silent continuation.** If a processor dies mid-pipeline the worker ends the call cleanly instead of leaving the caller on a dead line.
- **Bilingual is locale-driven, not prompt-driven.** An 11-entry map of Indian language codes feeds Pipecat's `Language` enum into STT, TTS and the session together, so the three always agree. Unmapped input falls back to `en-IN`.

<br>

## The Journey

The stack, year by year.

| Year | Focus |
|------|-------|
| **2022** | Java and Spring Boot, JWT, MySQL. Flutter and Angular alongside. |
| **2023** | React and Express. |
| **2024** | Next.js, React Native, PWA, and Go. |
| **2025** | Clean architecture. NestJS, PostgreSQL, Docker, AWS. |
| **2026** | Python and AI. Voice agents, multi-model AI, LangChain, LangGraph, vector databases, tool calling, Pipecat, FastAPI. |

<br>

## Writing

- [The Illusion of "Business Context" in AI Product Scoping](https://shreyas.thegoweb.com), July 2026
- More at [shreyas.thegoweb.com](https://shreyas.thegoweb.com)

<br>

## Currently

- Shipping the voice agent platform into real small-business deployments
- Learning Go, drawn to it by wanting backend fluency beyond the JVM
- Getting properly serious with Docker, Terraform and Dokploy
- Working through LangChain, LangGraph and RAG for retrieval-heavy agent work
- Exploring the shift from writing code to orchestrating agents that do the heavy lifting

<br>

## Let's Connect

<div align="center">

**Website** · [shreyas.thegoweb.com](https://shreyas.thegoweb.com)
**LinkedIn** · [linkedin.com/in/shreyasmark1](https://www.linkedin.com/in/shreyasmark1)
**Email** · [shreyas@thegoweb.com](mailto:shreyas@thegoweb.com)

<br>

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:249900,50:2DBF00,100:3DFF00&height=100" alt="" width="100%" style="display:block;width:100%;max-width:100%;border:0;">

</div>
