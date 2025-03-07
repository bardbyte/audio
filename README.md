# Real-Time Voice Processing Framework – Roadmap 

## Introduction:

This document outlines the roadmap, responsibilities, and execution plan for building Amex’s real-time voice processing framework leading up to the April 4th demo event. The objective is to:

- Establish the core infrastructure for real-time voice streaming & processing.

- Ensure seamless integration of GPT-4o with WebSockets/WebRTC.Enabling Amex teams and other discussed use-cases to build voice-enabled LLM applications on top of this framework.

*This roadmap is structured based on the in-person discussion with Allwyn on March 6th, ensuring that our priorities align with Amex’s goals and requirements.*

## Project Goals & Phases

### **Phase 1**: Websocket Infrastructure Enablement (Current Focus)
**Objective**: Design and build the foundational real-time audio streaming pipeline using WebSockets/WebRTC and connect it to Azure GPT-4o endpoint
### Tasks:

- Set up WebSocket infrastructure for low-latency real-time audio streaming.
- Test & integrate WebSocket with GPT-4o’s real-time inference.
- Decide where hosting and where it will live for scalability and distribution either within Safechain etc. 

**Team**: Ben, Saheb, Shashi 

### **Phase 2**: Audio Redaction Integration (Team is Facilitating, Not Owning)
Objective: Work with John’s team to integrate audio redaction into the framework.

### Tasks:

- Ensure redaction happens before the model processes data.
- Validate if GPT-4o’s built-in redaction is sufficient or if local processing is needed.
- Keep redaction modular so future enhancements can be added.
Team: John’s Team (lead), Saheb (facilitator)

### **Phase 3**: Voice Identification & Authentication 
**Objective**: Develop a provision on top for multi-speaker detection and voice authentication.

### Tasks:

- Research speaker diarization & biometrics-based voice authentication.
- Compare GPT-4o’s multi-speaker processing against Gemini & other models.
- Identify the best approach for handling authentication in noisy environments.

Team: Vaibhav (lead research), Saheb (technical validation & integration)

### **Phase 4:** Multilingual Speech Processing 

***Objective**: Ensure 50+ languages are supported and transcriptions are accurate & consistent.

### Tasks:

- Verify accuracy of GPT-4o’s multilingual support for real-time transcription.
- Document all language capabilities in Confluence & GitHub Markdown.
- Run tests in multiple real-world scenarios.

Team: Shashi (lead), Saheb (technical validation)


### Task Breakdown based on in-person March 6th discussion with Allwyn and team

| Task Name    | Assignee    | Priority | 
| -------- | ------- | -------- | 
| Design system architecture for WebSocket streaming (from EAG to our framework and post-model availble)| Ben, Shashi, Vaibhav, Saheb | High (current focus)
| Set up WebSocket streaming infrastructure  | Ben, Shashi, Saheb | High (current focus)
| Test WebSocket connection with GPT-4o | Shashi, Saheb  | High
| Facilitate redaction integration with John's team| John, Saheb | Low
| Run latency benchmarks for WebSocket performance| Saheb | Medium
| Test GPT-4o's tool call functionality for Amex use cases| Saheb, Shashi, Ben | Medium 
| Research & document speaker authentication methods  | Vaibhav | Medium
| Research GPT-4o vs. Gemini & other voice models for multi-speaker handling | Vaibhav| High
| Validate multilingual transcription accuracy and document the research| Shashi | Medium 
| Integrate synthetic data into our use case for the demo| Abraham, Saheb| Medium
| Prepare towards demo'ing for April 4th event| Ben, Shashi, Vaibhav, Saheb | High

