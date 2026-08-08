# SMART HEALTH AI
## An Intelligent Healthcare Analytics & Contextual Medical Assistant Platform

**Major Project Report**  
Submitted in partial fulfillment of the requirements for the award of the degree of  
**MASTER OF COMPUTER APPLICATIONS (MCA)**  
*Osmania University, Hyderabad*

---

### Project Information
- **Project Title:** Smart Health AI
- **Domain:** Artificial Intelligence / Healthcare Informatics / Full-Stack Web Development
- **Technology Stack:** Next.js 15 (App Router, React 19), TypeScript, Prisma ORM, PostgreSQL (Neon Serverless), LangChain, Multi-LLM Orchestration (Groq, Gemini, OpenAI, Anthropic, Ollama), AES-256-CBC Encryption, NextAuth v5

---

## 1. Abstract
Personal health management is often fragmented across multiple laboratory reports, digital health records, physical checkup documents, and symptom logs. **Smart Health AI** is a state-of-the-art, privacy-first healthcare assistance system designed to consolidate, parse, and structure multimodal health data (PDF reports, blood test scans, clinical records, and symptom logs) into standardized JSON representations.

By integrating LangChain and modern Large Language Models (LLMs) with retrieval and contextual reasoning capabilities, Smart Health AI enables patients to interact with their aggregated clinical records in real-time through streaming conversational interfaces. Security is paramount: all sensitive configuration tokens and provider API keys are encrypted at rest using AES-256-CBC cryptographic primitives with unique initialization vectors (IVs).

---

## 2. System Architecture

```
+-------------------------------------------------------------------------+
|                              CLIENT LAYER                               |
|       Next.js 15 App Router / React 19 / Tailwind CSS / Radix UI        |
|  - Auth & Onboarding  - Health Data Uploader  - Streaming Chat Interface|
+------------------------------------+------------------------------------+
                                     |  HTTPS / REST / Server Actions
                                     v
+------------------------------------+------------------------------------+
|                           APPLICATION LAYER                             |
|                        Next.js API & Middleware                         |
|  - NextAuth v5 Credentials Provider (bcryptjs)                          |
|  - AES-256-CBC Encryption Engine (crypto)                               |
|  - Multi-Provider LLM Gateway (LangChain OpenAI / Anthropic / Google)   |
|  - Server-Sent Events (SSE) / ReadableStream Real-time Chat Engine      |
+------------------+---------------------------------+--------------------+
                   |                                 |
                   v                                 v
+------------------+---------------+  +---------------+--------------------+
|          DATABASE LAYER          |  |         AI & PARSER LAYER          |
|      Neon Serverless PostgreSQL  |  |  - Multimodal Vision Parsers       |
|  - Prisma ORM 6.5.0              |  |  - PDF OCR & Table Extraction      |
|  - 12 Relational Tables / Enums  |  |  - Groq / Gemini / Ollama / OpenAI |
+----------------------------------+  +------------------------------------+
```

---

## 3. Key Modules & Features

### 3.1. Authentication & Security
- **NextAuth v5 Beta**: JWT session strategy with credential-based authentication using hashed passwords (bcryptjs).
- **AES-256-CBC Data Encryption**: Secure storage for external LLM API keys. Every encryption generates a fresh 16-byte cryptographically secure random Initialization Vector (IV).

### 3.2. Multimodal Health Data Ingestion & Parsing
- **PDF & Image Processing**: Ingestion of blood test results, checkup PDFs, and imaging scans.
- **Vision-Assisted Data Extraction**: Extracts clinical biomarkers, blood pressure, lipid profiles, and liver function panels into structured schema definitions.
- **Aggregation Engine**: Aggregates disparate data sources into a unified clinical context.

### 3.3. Streaming Conversational AI Engine
- **LangChain Integration**: Dynamic prompt assembly incorporating system instructions, aggregated biomarkers, and past chat history.
- **Real-time Streaming**: Response generation utilizing `ReadableStream` and `TextDecoder` for low-latency interactive clinical consultations.
- **Multi-Model Support**: Dynamic switching between Google Gemini, Groq (OpenAI-compatible), Claude, OpenAI, and local Ollama instances.

---

## 4. Database Schema Design (Prisma)

The application utilizes 12 models with comprehensive referential integrity:
1. `User`: Manages user credentials, onboarding state, timestamps.
2. `HealthData`: Stores raw file references, MIME types, parsed JSON clinical structures, and completion statuses.
3. `ChatRoom`: Organizes multi-turn conversations associated with specific assistant modes and LLM configurations.
4. `ChatMessage`: Records conversational turns (`USER` / `ASSISTANT`) linked via foreign key relations with cascade deletion.
5. `AssistantMode` & `AssistantModeContext`: Defines tailored system prompts and clinical role settings.
6. `LLMProvider`: Manages provider configurations (endpoints, model IDs, encrypted API keys).
7. `OAuth2Client`, `OAuth2AuthorizationCode`, `OAuth2Token`: Standardized OAuth2 authentication provider infrastructure.
8. `RedditPost`, `RedditPostComment`, `RedditAccessToken`: Community integration support.

---

## 5. Local Setup & Execution Guide

### Prerequisites
- Node.js >= 18.x
- PostgreSQL / Neon Serverless DB
- GraphicsMagick & Ghostscript (for PDF processing)

### Environment Setup (`.env`)
```env
DATABASE_URL="postgresql://<user>:<password>@<neon-host>/<db>?sslmode=require"
AUTH_SECRET="<generated-base64-secret>"
ENCRYPTION_KEY="<32-byte-base64-key>"
NEXT_PUBLIC_URL="http://localhost:3000"
DEPLOYMENT_ENV="local"
```

### Installation Steps
```bash
# 1. Install dependencies
npm install

# 2. Generate Prisma Client & Sync Schema with Neon DB
npx prisma generate
npx prisma db push

# 3. Start Development Server
npm run dev
```

---

## 6. Viva Voce & Technical Defense Q&A

### Q1: How does streaming work in the chat module?
**Answer:** The chat route (`src/app/api/chat-rooms/[id]/messages/route.ts`) constructs a `ReadableStream` utilizing LangChain's `.stream()` interface. As tokens arrive from the LLM provider, they are enqueued in SSE format (`text/event-stream`), decoded by the client's `ReadableStreamDefaultReader`, and rendered incrementally to minimize Time-to-First-Token (TTFT).

### Q2: Why is AES-256-CBC used with random IVs?
**Answer:** AES-256-CBC ensures confidentiality for sensitive API keys at rest. A unique 16-byte IV is prepended to the ciphertext upon encryption, preventing deterministic ciphertext analysis across repeated encryptions of identical keys.

### Q3: How is clinical context passed to the LLM?
**Answer:** Prior to prompting the LLM, the system retrieves all completed `HealthData` records for the authenticated user, serializes the clinical parameters, and injects them alongside the specialized `AssistantMode` system prompt into the LangChain message payload.
