# Technical Design Document: AMEX Audio Framework

**Contributors**: Saheb Singh, Ben Laney, Shashi Matta, Vaibhav Gupta
**Created**: March 5, 2025  
**Last Updated**: March 12, 2025  
**Status**: Draft  
**Reviewers**: Allwyn Ponnudurai
**Target Audience**: AMEX Developers, GSE Team, Stakeholders  

---

## 1. Background

AMEX seeks to leverage real-time audio processing via the GPT-4o Realtime model (Azure endpoint pending EAG approval) to enhance customer experiences, such as authentication and inquiries. As of March 2025, this capability is in preview, requiring a framework to abstract its WebSocket-based connection for broader use across AMEX teams. This document proposes `amex-audio-framework`, a lightweight, reusable system to connect to GPT-4o Realtime, designed for extensibility to future audio endpoints and minimal redaction support, leaving use-case specifics (e.g., authentication) to downstream developers.

---

## 2. Problem Statement

AMEX developers need a standardized way to interface with GPT-4o Realtime’s WebSocket API without reinventing connection logic for each project. The framework must handle audio streaming efficiently, support basic redaction as a placeholder, and remain flexible for alternative audio websocket endpoints (e.g., OpenAI, Deepgram, Gemini Audio Models etc.) and advanced redaction layers (e.g., John’s team).

---

## 3. Goals

- **Core Functionality**: Provide a reusable WebSocket connection to Azure GPT-4o Realtime.  
- **Modularity**: Separate connection, redaction, and API layers for independent extension.  
- **Performance**: Achieve <500ms latency and 10+ concurrent streams for demo scale.  
- **Extensibility**: Support future WebSocket endpoints via configuration.  
- **Redaction**: Include a basic placeholder, pluggable for advanced implementations.  
- **Demo Readiness**: Enable simple audio-to-text transcription with synthetic audio.

---

## 4. Non-Goals

- **Business Logic**:  Authentication handled by IdaaS, tool calls and voice diarization is a future phase of this, or inquiry handling—left to AMEX devs.  
- **Production Deployment**: Focus on demo, not production hardening (e.g., no HA setup).  
- **Advanced Redaction**: Implementation of complex PII masking (e.g., InfoSec team layer) deferred.  
- **UI Development**: Frontend integration out of scope—API-only framework.

---

## 5. Proposed Design

### 5.1 System Overview
The `amex-audio-framework` connects synthetic audio inputs to GPT-4o Realtime via a modular, WebSocket-based architecture, exposed through a FastAPI endpoint.

#### High-Level Flow
- **Input**: Synthetic PCM16 WAVs uploaded by clients (e.g., curl or AMEX apps).  
- **API Layer**: FastAPI server ingests audio and delegates to the framework.  
- **Framework Core**: Manages WebSocket streaming to the endpoint, applies optional basic redaction.  
- **Output**: Returns transcribed text as JSON.

#### High Level Architecture

[Client Audio] --> [FastAPI Server] --> [Framework: AudioClient] --> [GPT-4o Realtime EAG Endpoint]


### 5.2 Components

#### 5.2.1 Directory Structure
- **`config/`**: YAML files for endpoint and session settings.  
- **`src/core/`**: WebSocket connection logic and endpoint abstraction.  
- **`src/redaction/`**: Basic redaction placeholder (extensible).  
- **`src/api/`**: FastAPI server for audio ingestion.  
- **`src/utils/`**: Configuration parsing utilities.  
- **`tests/`**: Unit tests for core functionality.

#### 5.2.2 Configuration
- File: `config/endpoint.yaml`.  
- Defines endpoint type (e.g., "azure", "openai"), URL, auth method, and session parameters (e.g., audio format).  
- Enables switching endpoints without code changes.

#### 5.2.3 Core Framework
- **Component**: `AudioClient` in `src/core/`.  
- **Function**: Establishes WebSocket connection, streams audio chunks, collects transcript deltas.  
- **Details**:  
  - Streams 100ms PCM16 chunks (24kHz) for low latency.  
  - Supports Azure Managed Identity or API key auth.  
  - Integrates optional redaction layer per delta.

#### 5.2.4 Endpoint Abstraction
- **Component**: `EndpointFactory` in `src/core/`.  
- **Function**: Creates `AudioClient` instances based on config (e.g., Azure, OpenAI).  
- **Details**: Extensible to new WebSocket endpoints via factory pattern.

#### 5.2.5 Redaction Layer
- **Component**: `src/redaction/`.  
- **Function**: Applies basic PII masking (e.g., 16-digit card numbers) as a placeholder.  
- **Details**:  
  - Default: Regex-based, lightweight.  
  - Pluggable: AMEX devs can inject custom redaction logic.

#### 5.2.6 API Server
- **Component**: `src/api/`.  
- **Function**: FastAPI endpoint (`/process_audio`) for audio uploads.  
- **Details**: Thin wrapper—passes audio to `AudioClient`, returns JSON response.

---

## 6. Alternatives Considered

### 6.1 Monolithic Script
- **Description**: Single script handling connection and API.  
- **Pros**: Quick to build for demo.  
- **Cons**: Not reusable, hard to extend—violates framework goal.  
- **Why Rejected**: Fails AMEX devs’ long-term needs.

### 6.2 Heavy Middleware (e.g., Kafka)
- **Description**: Queue-based system for audio processing.  
- **Pros**: High throughput for production.  
- **Cons**: Adds latency (>500ms), overkill for demo.  
- **Why Rejected**: Too complex for scope and timeline.

### 6.3 OpenAI-Only Design
- **Description**: Hardcode OpenAI endpoint, skip Azure.  
- **Pros**: Immediate access, simpler auth.  
- **Cons**: Misaligns with AMEX’s Azure preference.  
- **Why Rejected**: Strategic misalignment; config-driven approach better.

---

## 7. Technical Details

### 7.1 Performance
- **Latency**:  
  - Target: <500ms end-to-end.  
  - Expected: ~260ms for first delta (50ms connect + 200ms RTT + 10ms processing).  
  - Mitigation: 100ms chunked streaming, async I/O.  
- **Throughput**:  
  - Target: 10+ concurrent streams.  
  - Approach: Async WebSocket; pooling deferred to future phases.

### 7.2 Extensibility
- **New Endpoints**: Add to `EndpointFactory` and `endpoint.yaml` (e.g., Deepgram).  
- **Redaction**: Pass custom callable to `AudioClient` via factory.

### 7.3 Dependencies
- WebSocket library for streaming.  
- FastAPI and Uvicorn for API server.  
- Azure Identity for Managed Identity auth.  
- YAML parser for config.

### 7.4 Testing
- Unit tests for `AudioClient` and redaction in `tests/`.  
- Integration test with synthetic WAV.

---

## 8. Risks and Mitigations

 - **Risk**: Latency exceeds 500ms with large audio.  
  - **Mitigation**: Tune chunk size (100ms baseline); benchmark early.  
- **Risk**: Limited throughput for demo scale.  
  - **Mitigation**: Async design suffices; pooling if needed later.


---

## 9. Open Questions

- **EAG Timeline**: When will Azure GPT-4o endpoint be provisioned?  
- **Synthetic Audio**: What different audio scenarios we need to test the model against once this framework is up and running? 
- **Redaction Needs**: Any specific PII patterns beyond card numbers for basic layer?
- **Safechain integration**: Can this live and integrate within safechain or will it be an independent framework? 

---

## 10. Appendix

### 10.1 Glossary
- **GPT-4o Realtime**: OpenAI’s real-time audio/text model, Azure-hosted in preview.  
- **PCM16**: 16-bit Pulse Code Modulation audio format at 24kHz.  
- **EAG**: Enterprise Architecture Group—AMEX.

### 10.2 References
- Azure GPT-4o Realtime API docs - [official docs](https://learn.microsoft.com/en-us/azure/ai-services/openai/realtime-audio-quickstart?tabs=keyless%2Cmacos&pivots=ai-foundry-portal) - (internal link pending EAG).  
- OpenAI Realtime API: [platform.openai.com/docs](https://platform.openai.com/docs).

---

## 12. Feedback Requested
- Does this meet AMEX devs’ connection needs?   
- Thoughts on basic redaction scope?  

Let’s refine this together—your input shapes the framework!          