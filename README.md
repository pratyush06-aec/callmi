# ElevateBox Outbound Sales AI Agent (callmi)

An advanced, real-time outbound sales AI agent designed for ElevateBox (a web development and e-commerce agency). Powered by LiveKit Agents, Groq Llama 3.1, ElevenLabs TTS, and Deepgram STT, this system automatically dials leads, qualifies them, seamlessly switches languages (English, Hindi, Telugu), and sends intelligent WhatsApp follow-ups mid-call and post-call.

## 🚀 Key Features

*   **Ultra-Low Latency Voice Pipeline:** Uses Deepgram Nova-3 for real-time Speech-to-Text, Groq's fast Llama 3.1 8B model for intelligent dialogue and tool usage, and ElevenLabs for hyper-realistic Text-to-Speech.
*   **Proactive Outbound SIP Calling:** Fully capable of dialing real-world phone numbers using a Twilio SIP Trunk via the LiveKit API (`trigger.py`).
*   **Multilingual Code-Switching:** Naturally converses and dynamically code-switches between English, Hindi, and Telugu depending entirely on the user's spoken language.
*   **Dynamic Intent Classification & Actions:** The AI autonomously classifies leads into Hot, Warm, or Cold based on their budget, timeline, and requirements.
*   **WhatsApp Integrations (Mid-Call & Post-Call):** *(Note: Currently hardcoded to exclusively send messages to `+918688664337` for testing purposes).*
    *   **Hot Leads:** Instantly receive a mid-call WhatsApp message with an exclusive portfolio link without disconnecting the ongoing voice call.
    *   **Post-Call Summaries:** All leads automatically receive a post-call WhatsApp message containing a detailed summary of their requirements, a resume link, an architecture diagram link, and any dynamically scheduled callback times.
*   **Automated Callback Scheduling:** If a lead is busy or prefers to talk later, the agent saves the spoken requested time directly into the local database and includes it in the WhatsApp summary.
*   **Strict Domain Guardrails:** Enforced system prompt boundaries ensure the agent strictly refuses to discuss topics outside of web development and e-commerce.

---

## 🏗️ System Architecture

The project consists of a purely Python-based backend that handles SIP communication, LLM inference, and external API requests:

*   **`src/agent.py`**: The core LiveKit agent entrypoint. Contains the LLM system prompt, tool definitions (discovery, classification, scheduling), and WebRTC event hooks (like triggering the WhatsApp summary `on_disconnected`).
*   **`src/db.py`**: A lightweight SQLite wrapper for securely storing lead data, saving callback times, and persisting discovery details locally.
*   **`trigger.py`**: The CLI script responsible for dispatching the agent and initiating the outbound SIP call to Twilio.
*   **LiveKit Cloud**: Handles the WebRTC media streaming, VAD (Voice Activity Detection using Silero), and SIP Trunking.
*   **Twilio**: Acts as the PSTN bridge for outbound calling and provides the WhatsApp messaging API.

---

## 🛠️ Required API Dependencies & Credentials

This project heavily relies on cloud services. You will need to create a `.env.local` file inside the `backend/` directory with your own API keys. 

**Backend (`backend/.env.local`):**
```env
# LiveKit Cloud (https://cloud.livekit.io/)
LIVEKIT_URL=wss://<YOUR_PROJECT_URL>.livekit.cloud
LIVEKIT_API_KEY=<YOUR_LIVEKIT_API_KEY>
LIVEKIT_API_SECRET=<YOUR_LIVEKIT_API_SECRET>

# STT: Deepgram (https://deepgram.com)
DEEPGRAM_API_KEY=<YOUR_DEEPGRAM_API_KEY>

# LLM: Groq (https://console.groq.com/keys)
GROQ_API_KEY=<YOUR_GROQ_API_KEY>

# TTS: ElevenLabs (https://elevenlabs.io/)
ELEVEN_API_KEY=<YOUR_ELEVENLABS_API_KEY>

# SIP Telephony (Twilio / LiveKit SIP Trunk)
SIP_OUTBOUND_TRUNK_ID=<YOUR_SIP_TRUNK_ID>

# Twilio WhatsApp Messaging (https://www.twilio.com/)
TWILIO_ACCOUNT_SID=<YOUR_TWILIO_SID>
TWILIO_AUTH_TOKEN=<YOUR_TWILIO_AUTH_TOKEN>
TWILIO_WHATSAPP_NUMBER=whatsapp:<YOUR_TWILIO_WHATSAPP_NUMBER>
```

---

## 📦 Python Dependencies

Managed via `uv` in `backend/pyproject.toml`:
*   `livekit-agents` (Core framework)
*   `livekit-api` (For triggering SIP calls)
*   `livekit-plugins-groq` (LLM integration)
*   `livekit-plugins-elevenlabs` (TTS integration)
*   `livekit-plugins-noise-cancellation` (Audio cleanup)
*   `aiohttp` & `urllib` (For Twilio WhatsApp API requests)
*   `python-dotenv` (Environment management)
*   `pytest` (For testing)

---

## 🚀 Running the Agent

### 1. Install Dependencies
```bash
cd backend
uv sync
```

### 2. Start the Agent Worker
This will connect your local Python process to LiveKit Cloud and prepare it to receive agent dispatch requests.
```bash
uv run python src/agent.py dev
```

### 3. Trigger an Outbound Phone Call
In a separate terminal, use the trigger script to dial the target phone number. By default, this will dial the hardcoded test number (`+918688664337`).
```bash
cd backend
uv run python trigger.py
```
*(You can optionally provide a different number via `uv run python trigger.py +91XXXXXXXXXX`, but WhatsApp messages will still strictly route to `+918688664337`)*.
*The agent will immediately say the opening line in English as soon as the callee picks up!*
