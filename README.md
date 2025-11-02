# Media-UI — Real-Time Voice Agent Testing Platform

<div align="center">
  <br />
  <div>
    <img src="https://img.shields.io/badge/-Next.js_15-000000?style=for-the-badge&logo=next.js&logoColor=white" alt="Next.js" />
    <img src="https://img.shields.io/badge/-React_19-61DAFB?style=for-the-badge&logo=react&logoColor=black" alt="React" />
    <img src="https://img.shields.io/badge/-TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white" alt="TypeScript" />
    <img src="https://img.shields.io/badge/-Node.js-339933?style=for-the-badge&logo=node.js&logoColor=white" alt="Node.js" />
    <img src="https://img.shields.io/badge/-WebSocket-010101?style=for-the-badge&logo=socket.io&logoColor=white" alt="WebSocket" />
    <img src="https://img.shields.io/badge/-gRPC-244C5A?style=for-the-badge&logo=grpc&logoColor=white" alt="gRPC" />
    <img src="https://img.shields.io/badge/-TailwindCSS-38B2AC?style=for-the-badge&logo=tailwindcss&logoColor=white" alt="Tailwind CSS" />
  </div>
  <h3 align="center">Debug & Test Voice-Based Autonomous Agents</h3>
  <div align="center">
     Real-time STT → LLM → TTS testing with latency analytics, barge-in support, and conversation export
  </div>
  <br />
  
  <div align="center">
    <img src="docs/tool.png" alt="UI Screenshot" width="100%" style="max-width: 900px; border-radius: 8px; box-shadow: 0 4px 6px rgba(0,0,0,0.1);">
  </div>
  <br />
</div>

## 📋 Table of Contents

- [Introduction](#-introduction)
- [Tech Stack](#️-tech-stack)
- [Features](#-features)
- [Quick Start](#-quick-start)
- [Architecture](#-architecture)
- [Configuration](#️-configuration)
- [Project Structure](#-project-structure)
- [Development Guide](#-development-guide)

## 🚀 Introduction

**Media-UI** is a full-featured testing platform for voice-based autonomous agents, providing real-time audio streaming, speech recognition debugging, and comprehensive latency analytics.

**Built for:**

- ✅ **QA & Testing** – Validate STT accuracy, TTS quality, and agent responses
- ✅ **Performance Analysis** – Track latency metrics, silence gaps, and barge-in behavior
- ✅ **Debugging** – Export full conversation logs, recordings, and metrics
- ✅ **Demos & Presentations** – Clean chat UI with real-time agent interaction

> **⚠️ Note:** This is a **testing/debugging tool**, not a production voice application. Focus is on observability and developer experience.

## ⚙️ Tech Stack

### **Frontend** (Next.js App)

- **Next.js 15** – React framework with App Router
- **React 19** – Latest features with concurrent rendering
- **TypeScript 5** – Full type safety
- **Tailwind CSS 4** – Utility-first styling
- **Web Audio API** – AudioWorklet for microphone capture & TTS playback
- **Radix UI** – Accessible dialog, tooltip, switch components
- **Lucide Icons** – Clean, consistent iconography

### **Backend** (Node.js WebSocket Bridge)

- **WebSocket (ws)** – Real-time bidirectional communication
- **ConnectRPC** – gRPC-web protocol over WebSocket
- **Protocol Buffers** – Type-safe message serialization
- **ts-node** – Direct TypeScript execution for server

### **Audio Processing**

- **AudioWorklet** – Low-latency PCM capture (`pcm-processor.js`)
- **16-bit LINEAR16** @ 16kHz – High-quality audio encoding
- **µ-law decoding** – TTS playback from backend
- **WAV export** – Mixed recordings with real-time sync

### **Infrastructure**

- **Docker** – Multi-stage production builds
- **PM2** – Process management for Next.js + WebSocket server
- **Protocol Buffers** – Generated TypeScript types from `.proto` files

## ⚡ Features

### 🎤 **Real-Time Audio Streaming**

- Microphone capture via AudioWorklet (128-sample quantum)
- Buffered streaming with 40ms intervals
- Automatic AudioContext resume handling
- Device selection support

### 🧠 **Speech Recognition**

- Interim and final transcription results
- Start-of-input (SOI) and end-of-input (EOI) events
- Barge-in detection and handling
- Live text updates during speech

### 🔊 **Text-to-Speech Playback**

- Queue-based audio playback
- Interruptible during barge-in
- µ-law and WAV format support
- Chunk-level playback tracking

### 💬 **Chat Interface**

- Real-time message bubbles (user + agent)
- Millisecond-precision timestamps
- Connection status indicator
- Call duration timer

### 📊 **Latency Metrics**

- **Call-level**: Start latency, greeting playback time
- **Per-dialogue**:
  - First interim result latency
  - Customer utterance length
  - Prompt playback time
  - Silence gaps (pre/post agent response)
  - Barge-in latency
  - Audio chunks sent
- Expandable metrics panel with visual indicators

### 📤 **Export Capabilities**

- **Mixed Recording**: Caller + Agent audio synchronized
- **Backend Logs**: Full conversation with scrubbed audio payloads
- **Transcript**: HTML export with timestamps
- **Kibana Link**: Direct link to orchestrator logs

### 🛡️ **Error Handling**

- WebSocket reconnection logic
- gRPC stream error recovery
- User-friendly error messages
- Comprehensive client-side logging

## 🚀 Quick Start

### Prerequisites

- **Node.js 22.x** ([nvm](https://github.com/nvm-sh/nvm))
- **pnpm** (enable with `corepack enable`)

### Local Development

```bash
# 1. Install dependencies
pnpm install

# 2. Start WebSocket server (terminal 1)
pnpm dev:server
# Runs on ws://localhost:3001/ws

# 3. Start Next.js frontend (terminal 2)
pnpm dev
# Runs on http://localhost:3000

# Or start both concurrently:
pnpm dev:all
```

Visit **http://localhost:3000** → Configure connection → Start call

### Docker Deployment

```bash
# Build image
docker build -t media-ui .

# Run container
docker run -d \
  -p 3000:3000 \
  -p 3001:3001 \
  --name media-ui \
  media-ui

# Check logs
docker logs -f media-ui
```

**Services:**

- Frontend: http://localhost:3000
- WebSocket: ws://localhost:3001/ws

### Available Scripts

```bash
# Development
pnpm dev              # Next.js dev server (port 3000)
pnpm dev:server       # WebSocket server (port 3001)
pnpm dev:all          # Start both with concurrently

# Production
pnpm build            # Build Next.js app
pnpm start            # Start production server

# Utilities
pnpm lint             # ESLint checks
pnpm typecheck        # TypeScript validation
```

## 🏗️ Architecture

### High-Level Flow

```
                        WebSocket (JSON/Protobuf)           gRPC (Protobuf)
    ┌──────────────────────────────────────────┐    ┌──────────────────────────┐
    │                                          │    │                          │
    │                                          ▼    ▼                          │
┌───┴────────────┐                      ┌─────────────────┐              ┌────┴─────────────┐
│                │                      │                 │              │                  │
│    Next.js     │◀────────────────────▶│    Node.js      │◀────────────▶│    Universal     │
│    Frontend    │                      │  WebSocket      │              │     Harness      │
│                │   Bidirectional      │    Bridge       │ Bidirectional│    (Backend)     │
│  (Port 3000)   │   Streaming          │  (Port 3001)    │  Streaming   │                  │
│                │                      │                 │              │                  │
└────────┬───────┘                      └────────┬────────┘              └──────────────────┘
         │                                       │
         │ ┌─────────────────────────────────────┘
         │ │
         │ │  • Bearer Token (JWT)
         │ │  • Orchestrator Host URL
         │ │  • Org ID / Conversation ID
         │ │  • Language & Agent Config
         │ │
         ▼ ▼
    ┌─────────────────┐
    │  AudioWorklet   │
    │  PCM Processor  │
    ├─────────────────┤
    │  • 16-bit PCM   │
    │  • 16 kHz       │
    │  • 128 samples  │
    │  • 40ms buffer  │
    └─────────────────┘
         │
         ▼
    ┌─────────────────┐
    │   Microphone    │
    │   Hardware      │
    └─────────────────┘
```

### Call State Machine

```
IDLE
  ↓ startCall()
CALL_START (greeting)
  ↓ greeting received + played
AUDIO_STREAMING (duplex)
  ↓ user speaks → ASR → VA response
  ↓ loop until endCall()
CALL_END
  ↓ cleanup
ENDED
```

### Data Flow: Voice Interaction

```
1. User speaks → AudioWorklet captures PCM
2. UseMicrophone hook → sendAudioChunk()
3. CallStateMachine → buffers 40ms chunks
4. WebSocket → sends to Node.js bridge
5. Bridge → forwards to gRPC backend
6. Backend → ASR (interim/final) + VA response
7. WebSocket ← receives response with TTS audio
8. TTSPlayer → decodes µ-law → plays via Web Audio
9. UI updates with transcript + metrics
```

## ⚙️ Configuration

### Environment Variables

Create `.env.local`:

```bash
# WebSocket URL (auto-detected if not set)
NEXT_PUBLIC_WS_URL=ws://localhost:3001/ws
```

### Connection Settings

Configure via UI (stored in `localStorage`):

| Field              | Description                 | Example                                   |
| ------------------ | --------------------------- | ----------------------------------------- |
| **Host**           | Orchestrator gRPC endpoint  | `https://orchestrator.example.com`        |
| **Bearer Token**   | Authentication JWT          | `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...` |
| **Language**       | Speech recognition language | `en-US`, `en-IN`, `fr-FR`                 |
| **OrgId**          | Organization UUID           | `12345678-1234-1234-1234-123456789abc`    |
| **ConversationId** | Unique conversation UUID    | Auto-generated or manual                  |
| **VirtualAgentId** | Agent configuration ID      | `agent-abc123`                            |
| **WxCC ClusterId** | Cluster routing identifier  | `intgus1`                                 |
| **User Agent**     | Client identifier           | `web-ui`                                  |
| **Microphone**     | Audio input device          | Selected from browser enumeration         |

## 📁 Project Structure

```
media-ui/
├── src/
│   ├── app/
│   │   ├── page.tsx              # Main entry (ChatApp wrapper)
│   │   ├── layout.tsx            # Root layout with fonts
│   │   └── globals.css           # Tailwind directives
│   │
│   ├── components/
│   │   ├── ChatApp.tsx           # Top-level config + chat manager
│   │   ├── ChatBotUI.tsx         # Main chat interface
│   │   ├── ChatBubble.tsx        # Message display component
│   │   ├── ChatControls.tsx      # Start/stop/mic buttons
│   │   ├── ChatMetricsPanel.tsx  # Metrics sidebar
│   │   ├── ConfigScreen.tsx      # Connection configuration form
│   │   ├── ConnectionIndicator.tsx
│   │   ├── LatencyMetricsDisplay.tsx
│   │   ├── TranscriptExporter.tsx
│   │
│   │
│   ├── state/
│   │   ├── CallStateMachine.ts   # FSM orchestration
│   │   └── types.ts              # CallState enum + types
│   │
│   ├── grpc/
│   │   ├── bridgingClient.ts     # WebSocket ↔ gRPC bridge
│   │   ├── generated/            # Protobuf TypeScript files
│   │   │   ├── InsightInfer_pb.ts
│   │   │   ├── InsightInfer_connect.ts
│   │   │   ├── virtualagent_pb.ts
│   │   │
│   │   └── protos/               # .proto source files
│   │
│   ├── lib/
│   │   └── audio/
│   │       ├── TTSPlayer.ts      # TTS playback queue
│   │       ├── wavRecorder.ts    # WAV export utilities
│   │       ├── recordingBuilder.ts # Mixed audio timeline
│   │       └── recStore.ts       # IndexedDB storage
│   │
│   ├── hooks/
│   │   └── UseMicrophone.ts      # AudioWorklet integration
│   │
│   ├── server/
│   │   ├── wsServer.ts           # WebSocket server (port 3001)
│   │   ├── grpcTransport.ts      # gRPC client setup
│   │   ├── enumMapper.ts         # Protobuf enum conversions
│   │   ├── PushableStream.ts     # Async iterable stream
│   │   ├── utils.ts              # Base64 + logging helpers
│   │   └── logger.ts             # Structured logging
│   │
│   ├── config/
│   │   └── appProperties.ts      # Audio constants
│   │
│   └── scripts/
│       └── generate_protos.sh    # Protobuf codegen
│
├── public/
│   └── pcm-processor.js          # AudioWorklet processor
│
├── docs/
│   ├── tool.png                  # UI screenshot
│   ├── Class Diagram.png         # Architecture diagram
│   └── media-ui-sequence-diagram.png
│
├── Dockerfile                    # Multi-stage production build
├── ecosystem.config.js           # PM2 configuration
├── next.config.ts                # Next.js configuration
├── tsconfig.json                 # TypeScript config
├── tailwind.config.ts            # Tailwind setup
└── package.json                  # Dependencies + scripts
```

## 🛠️ Development Guide

### Generating Protobuf Files

```bash
# Install buf CLI (first time)
brew install bufbuild/buf/buf

# Generate TypeScript files from .proto
cd src/scripts
bash generate_protos.sh

# Or manually:
npx buf generate --path src/grpc/protos
```

### Adding a New Feature

**Example: Add "Call Recording Export to S3"**

```typescript
// 1. Update CallStateMachine.ts
public async endCall() {
  const recordings = await this.getRecordings();

  // New: Upload to S3
  if (recordings.mixed) {
    await uploadToS3(recordings.mixed, this.config.conversationId);
  }

  return recordings;
}

// 2. Create upload utility (lib/storage/s3.ts)
export async function uploadToS3(blob: Blob, convId: string) {
  const formData = new FormData();
  formData.append('file', blob, `${convId}.wav`);

  await fetch('/api/upload', {
    method: 'POST',
    body: formData
  });
}

// 3. Add API route (app/api/upload/route.ts)
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';

export async function POST(req: Request) {
  const formData = await req.formData();
  const file = formData.get('file') as File;

  // Upload to S3...
  return Response.json({ url: s3Url });
}
```

### Debugging Tips

#### WebSocket connection issues

```bash
# Check server is running
curl http://localhost:3001

# Test WebSocket with wscat
npm install -g wscat
wscat -c ws://localhost:3001/ws
> {"ping":1}

# Check browser console for connection errors
```

#### Audio not capturing

```bash
# Verify microphone permissions in browser
# Chrome: Settings → Privacy → Microphone

# Check AudioWorklet loading
# Browser console should show: "Microphone: Loaded PCM processor"

# Test with different sample rate
# Edit src/config/appProperties.ts:
FIXED_SAMPLE_RATE: 8000  # Try 8kHz instead of 16kHz
```

#### gRPC errors

```bash
# Check token expiration
# JWT decode: https://jwt.io

# Verify host URL format
# Must include https:// protocol

# Check backend logs for auth failures
```

### Common Issues

| Issue                    | Solution                                       |
| ------------------------ | ---------------------------------------------- |
| "No token provided"      | Enter valid bearer token in config screen      |
| "AudioContext suspended" | Click anywhere on page to trigger user gesture |
| "WebSocket closed"       | Restart ws-server: `pnpm dev:server`           |
| "VA greeting timeout"    | Check virtualAgentId is valid in config        |
| Choppy audio playback    | Reduce network latency or increase buffer size |
| Recording export fails   | Check browser IndexedDB quota (clear if full)  |

---

<div align="center">
  <br />
  <p><strong>⚠️ Testing Tool Disclaimer</strong></p>
  <p>This is a debugging and testing platform. For production voice applications:</p>
  <p>✓ Implement proper authentication &nbsp; ✓ Add rate limiting &nbsp; ✓ Secure WebSocket connections (WSS) &nbsp; ✓ Add monitoring/alerting</p>
  <br />
  <p>For architecture details and flow diagrams, see the <code>docs/</code> folder</p>
</div>
