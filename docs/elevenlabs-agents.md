# ElevenLabs Agents: Login, Testing, and Capabilities

Source: ElevenLabs public docs, summarized. Confirm exact names, limits, and models in the dashboard before relying on them.

## 1. Sign up and get an API key

1. Sign up or log in at https://elevenlabs.io/app/sign-up.
2. Create an API key at https://elevenlabs.io/app/settings/api-keys.
3. Store the key in a `.env` file. Never put it in code or chat.
   ```
   ELEVENLABS_API_KEY=your_key_here
   ```
   `.env` is in `.gitignore`.

## 2. Create an agent in the dashboard

1. Create a new agent with the **Blank template**.
2. **Agent tab:** set the first message. Example: "Hi, this is [name] from [company] support. How can I help you today?"
3. **Agent tab:** write the system prompt. It sets role, tone, and rules.
4. **Knowledge Base:** upload documents or add links.
5. **Voice tab:** choose a voice from the library.
6. Click **Test AI agent** to talk to it through your microphone.

## 3. Create an agent with code

Install the SDK:

```bash
pip install elevenlabs python-dotenv          # Python
npm install @elevenlabs/elevenlabs-js dotenv  # TypeScript
```

Set these fields:

| Field | Purpose |
|---|---|
| `name` | Agent name |
| `conversation_config.tts.voice_id` | Voice |
| `conversation_config.agent.first_message` | Opening line |
| `conversation_config.agent.prompt.prompt` | System prompt |

Run: `python create_agent.py` or `npx tsx createAgent.mts`.

## 4. Ways to test

| Method | How |
|---|---|
| Dashboard | Click Test AI agent |
| Website widget | Embed an HTML snippet with the agent ID |
| React SDK | Build a custom front end |
| Automated tests | Use the testing framework (capability 16) |

## 5. Capabilities

| # | Capability | Detail |
|---|---|---|
| 1 | Voice dialogue | Real-time spoken conversation with natural back-and-forth |
| 2 | Speech recognition | Fine-tuned ASR model for conversation |
| 3 | Language model layer | Choose from several LLMs, or bring your own. Check the dashboard for the list. |
| 4 | Text-to-speech | Low-latency engine. 5,000+ voices, 31 languages for agents. 70+ languages for TTS. |
| 5 | Voice customization | Choose and tune voices to fit your brand |
| 6 | Turn-taking | Proprietary model decides when the user has finished speaking |
| 7 | Prompt and persona | First message, system prompt, and rules in dashboard or API |
| 8 | Knowledge base (RAG) | Upload documents or links. The agent retrieves them while answering. |
| 9 | Tools and API connections | The agent calls your APIs to look up or update data |
| 10 | Visual workflow builder | Design multi-step flows with branches |
| 11 | Telephony | SIP trunk, native Twilio, and batch outbound calls |
| 12 | Web embedding | Embeddable widget, React SDK, and UI components (shadcn-based) |
| 13 | Mobile SDKs | Swift (iOS), Kotlin (Android), React Native |
| 14 | WebSocket API | Low-level access for custom builds |
| 15 | CLI | Create and manage agents as code |
| 16 | Testing and evaluation | Automated tests of scripted conversations before release |
| 17 | Experiments | A/B test two versions of an agent |
| 18 | Analytics | Keyword and semantic search of history, quality analysis, metrics dashboard |
| 19 | Real-time events | Subscribe to events during calls |
| 20 | Data controls | Retention policies and custom authentication |
| 21 | Cost monitoring | Track usage and spend per agent |

## Not verified

- Exact LLM list and pricing
- Call length, concurrency, and knowledge base limits
- Compliance certifications (SOC 2, HIPAA)
- Latency in our regions

## Sources

- [ElevenLabs Agents overview](https://elevenlabs.io/docs/eleven-agents/overview)
- [ElevenLabs Agents quickstart](https://elevenlabs.io/docs/eleven-agents/quickstart)
- [ElevenLabs sign-up](https://elevenlabs.io/app/sign-up)
- [ElevenLabs API keys](https://elevenlabs.io/app/settings/api-keys)
