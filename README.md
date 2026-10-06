# Deep Intelligence Voice Agent

Research and design notes for a multi-use-case voice agent product.

- Backend: Python
- Client: Mobile app
- Current speech model: Amazon Nova 2 Sonic (AWS Bedrock)

## Contents

| File | What it covers |
|---|---|
| [docs/voice-agent-guide.md](docs/voice-agent-guide.md) | Voice agent basics, all agent types, model comparison, product design |
| [docs/top-voice-agent-platforms.md](docs/top-voice-agent-platforms.md) | Top 10 voice agent platforms and why they stand out |
| [docs/elevenlabs-agents.md](docs/elevenlabs-agents.md) | ElevenLabs Agents: sign-up, testing, and 21 capabilities |

## Status

Planning stage. No code yet.

## Next steps

1. Define the provider adapter interface (Python).
2. Define the tenant config schema (prompt, knowledge, tools, voice, language).
3. Benchmark Nova 2 Sonic, gpt-realtime, and Gemini Live on the same test calls.
4. Check latency from our main user regions.
