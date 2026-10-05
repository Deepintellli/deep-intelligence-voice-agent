# Voice Agents: Types, Models, and Product Design

## 1. Basics

A voice agent listens, understands, decides, acts, and speaks.

| Job | Meaning |
|---|---|
| Hear | Turn mic audio into data |
| Understand and decide | Work out what the user wants and what to do next |
| Act | Call tools, such as APIs or a database |
| Speak | Turn the reply into audio and play it |

### Two ways to build one

| Approach | Flow | Pros | Cons |
|---|---|---|---|
| Cascade | Mic → STT → text LLM → TTS → Speaker | Any text model. Easy to swap parts. | Slower. Loses tone and timing. |
| Speech-to-speech (S2S) | Mic → one model → Speaker | Low latency. Natural timing. Handles interruptions well. | Less control per step. Harder to swap models. |

Most new products use S2S.

### Core concepts

| Concept | Meaning |
|---|---|
| Audio chunks | Audio is sent in small pieces (about 20 to 100 ms). Common format: 16-bit PCM, mono, 16 or 24 kHz. |
| Full duplex | User and agent can speak at the same time. |
| VAD (voice activity detection) | Decides when the user starts and stops talking. |
| Barge-in | User interrupts. The agent stops, and the client stops playback. |
| Latency | Time from end of user speech to start of reply. A few hundred ms feels natural. Over about 1 s feels slow. |
| Transport | WebRTC for browser and mobile. WebSocket for server-to-server. |
| Tool calling | Model asks your code to run a function. You return the result. |
| System prompt | Sets role, tone, rules, and limits. Voice replies should be short. |

### Glossary

| Term | Meaning |
|---|---|
| STT | Speech-to-text |
| TTS | Text-to-speech |
| S2S | Speech-to-speech |
| VAD | Voice activity detection |
| PCM | Raw uncompressed audio samples |
| Turn | One user utterance plus the agent's reply |
| Transcript | Text record of the conversation |

### Build order

1. Text round-trip with the model (checks keys and SDK).
2. Audio in and out from a WAV file.
3. Live mic in a terminal, with a printed transcript.
4. Browser or mobile client, with playback and barge-in.
5. One tool (for example, a mock order lookup).
6. Guardrails, transcript logs, timeouts, error handling.
7. Evaluation on a fixed set of test calls.

### Common mistakes

- Waiting for full user audio before sending. Stream it.
- Not stopping playback on interrupt.
- Long, list-heavy replies. Keep voice replies short.
- API keys in client code.
- Testing only clean studio audio.
- No transcript logs.

---

## 2. All types of voice agents

### By architecture

| Type | How it works | Best for | Trade-off |
|---|---|---|---|
| Cascade | STT → text LLM → TTS | Full control of each step | Slower, loses tone |
| Speech-to-speech | One model, audio in and out | Natural, low-latency talk | Less per-step control |
| Hybrid | S2S for talk, text model for hard reasoning | Complex flows with natural voice | More parts to run |
| Turn-based (push-to-talk) | User records, then agent replies | Simple apps, low cost | Less natural |
| Full-duplex | Overlapping speech with barge-in | Real conversations | Needs careful turn-taking |
| Audio + video | Camera or screen stream plus voice | Visual help, inspection, demos | Higher cost, shorter sessions |

### By channel

| Channel | Transport | Typical use |
|---|---|---|
| Browser or mobile app | WebRTC (preferred) or WebSocket | In-app assistant |
| Server-to-server | WebSocket or bidirectional HTTP/2 stream | Backend-driven voice |
| Phone (PSTN) | Telephony bridge (Twilio, Vonage, Amazon Connect) | Support, booking |
| PBX / SIP | SIP trunk | Enterprise phone systems |
| Device | Embedded SDK | Kiosks, cars, wearables |

### By direction

| Direction | Example |
|---|---|
| Inbound (customer calls agent) | Support line |
| Outbound (agent calls customer) | Reminders, follow-ups |
| In-app (user talks inside app) | Mobile assistant |

### By autonomy

| Level | What it does |
|---|---|
| Scripted flow | Fixed steps, like an IVR menu |
| Prompt-driven | Follows a system prompt and knowledge |
| Tool-using | Reads and writes systems through functions |
| Multi-agent | Hands off between specialist agents |

### Features to design for

- Barge-in
- Turn-taking sensitivity
- Language switching
- Voice selection
- Tool calling (sync or async)
- Transcripts and logs
- Session resumption (sessions have time limits)
- Handoff to a human

---

## 3. Speech-to-speech models

| Model | Platform |
|---|---|
| Amazon Nova 2 Sonic (current) | AWS Bedrock |
| Amazon Nova Sonic (v1) | AWS Bedrock |
| OpenAI gpt-realtime | OpenAI Realtime API |
| Gemini Live API (native audio) | Google AI and Vertex AI |

Other options exist, such as Ultravox and Hume EVI. Not researched yet.

### Capabilities compared

Source: public docs, checked on the date of this note. Limits and language lists change. Confirm before committing.

| Capability | Nova 2 Sonic | gpt-realtime | Gemini Live API |
|---|---|---|---|
| Streaming voice in and out | Yes | Yes | Yes |
| Languages | Expanded set, incl. Portuguese and Hindi | Switches mid-sentence. Full list not verified. | 24 languages (native audio). Auto-switching. |
| Voices | Polyglot voices (one voice, many languages) | Cedar, Marin (new), others | 30 HD voices |
| Turn-taking control | Low, medium, high pause sensitivity | Server-side VAD settings | VAD |
| Tool calling | Yes. Async supported. | Yes. Remote MCP servers. | Yes. Sync only. |
| Text and voice in one session | Yes | Yes | Yes |
| Image input | Not stated | Yes | Video streaming |
| Phone and SIP | Amazon Connect, Twilio, Vonage, AudioCodes | SIP supported | Not stated |
| Open-source frameworks | LiveKit, Pipecat | Not stated | Not stated |
| Browser and mobile | Through your server | WebRTC recommended | Live API streaming |
| Context window | 1M tokens | Not stated | 128K in, 64K out |
| Session limits | Not stated | Not stated | Audio-only about 15 min. Audio + video about 2 min. Connection about 10 min. Warning 60 s before end. |
| Regions | US East (N. Virginia), US West (Oregon), Asia Pacific (Tokyo) | Not checked | Not checked |
| Grounding | Not stated | Not stated | Google Search |

Notes:

- Gemini limits come from a Firebase page. They may differ by model and platform.
- Nova 2 Sonic is not listed in Sydney (ap-southeast-2). Test latency from Tokyo or Oregon for Australian users.

---

## 4. Product design

The product must serve many use cases. Behavior comes from config and data, not from separate code.

### 4.1 Config per tenant or use case

| Field | Example |
|---|---|
| System prompt and persona | "You are a booking assistant for a clinic." |
| Knowledge sources | Documents for retrieval |
| Tools | Schemas for lookup and booking functions |
| Voice and language | Voice ID, language list |
| Turn-taking | Pause sensitivity |
| Escalation | When to hand off to a human |
| Compliance | Required disclaimers, forbidden statements |

A new use case is a new config. No redeploy.

### 4.2 Provider adapter (Python)

Define one interface:

```
VoiceSession
  start(config)
  send_audio(chunk)
  on_audio()
  send_tool_result(result)
  close()
```

Write one adapter per provider: `NovaSonicAdapter`, `GPTRealtimeAdapter`, `GeminiLiveAdapter`. App code uses only the interface. This allows:

- Nova 2 Sonic as the default
- Per-tenant or per-region provider choice
- Same test calls on every provider

### 4.3 Mobile architecture

```
Mobile app ⇄ (WebRTC or WebSocket) ⇄ Python server ⇄ Speech model
                                        ├── tools and APIs
                                        ├── config store
                                        └── transcripts and logs
```

- The phone never holds model or cloud keys.
- The server issues short-lived session tokens if the provider supports them.
- The app handles mic, playback, and stopping playback on interrupt.

### 4.4 Session limits

- Save session state to resume after a limit.
- Keep summaries of earlier turns.
- Tell the user when a session reconnects.

### 4.5 Hybrid for hard work

The speech model talks. A text model handles heavy reasoning and returns the result as a tool result. The speech model reads the answer aloud.

### 4.6 Evaluation

Test set should cover:

- Quiet and noisy audio, and speakerphone
- Accents and mixed languages
- Interruptions
- Tool calls with correct arguments
- Compliance phrases

Score each provider on latency, accuracy, and interruption handling.

---

## 5. Recommendation

1. Keep Nova 2 Sonic as the default.
2. Build the provider adapter now.
3. Benchmark gpt-realtime and Gemini Live on the same test set.
4. Check latency from main user regions before choosing a Bedrock region.

---

## Sources

- [Announcing Amazon Nova 2 Sonic](https://aws.amazon.com/about-aws/whats-new/2025/12/amazon-nova-2-sonic-real-time-conversational-ai)
- [Introducing Amazon Nova 2 Sonic (AWS News Blog)](https://aws.amazon.com/blogs/aws/introducing-amazon-nova-2-sonic-next-generation-speech-to-speech-model-for-conversational-ai)
- [Nova 2 Sonic model card, Amazon Bedrock](https://docs.aws.amazon.com/bedrock/latest/userguide/model-card-amazon-nova-2-sonic.md)
- [Introducing gpt-realtime (OpenAI)](https://openai.com/index/introducing-gpt-realtime/)
- [OpenAI Realtime API: A practical guide for voice agents (Dasha)](https://dasha.ai/blog/openai-realtime-api)
- [OpenAI upgrades Realtime API with phone calling and image support (CoAI)](https://getcoai.com/news/openai-upgrades-realtime-api-with-phone-calling-and-image-support)
- [Live API limits and specs (Firebase AI Logic)](https://firebase.google.com/docs/ai-logic/live-api/limits-and-specs)
- [Gemini 2.5 Flash with Gemini Live API (Google Cloud)](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/models/gemini/2-5-flash-live-api)
- [Gemini Live API key takeaways (Ry Walker)](https://rywalker.com/research/gemini-live-api)
