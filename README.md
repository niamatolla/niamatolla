
<img width="1200" height="650" alt="Capture d’écran, le 2026-10-15 à 16 47 02" src="https://github.com/user-attachments/assets/b49dc7e6-2af4-484a-bafb-4fc9baf7e41f" />



## Featured Projects

### Netpulse: UDP Latency Load Tester with a C++ Measurement Core · [repo](https://github.com/niamatolla/Netpulse)

Pushes a network path with a controlled, rising stream of UDP packets and measures exactly *how and when* it degrades using tail latency, not averages. It finds the knee: the load where p99 and loss spike while p50 still looks fine.

**Pipeline:**

```
rate sweep  (100 → 5000 pps)
        │
        ▼
  Python orchestrator
  runs the C++ core once per rate, collects results
        │
        ▼
  C++ core  (open-loop load generator)
        │
        ├── send loop → UDP packets at a fixed rate → echo server
        │
        └── recv loop ← echoed packets, timestamped per packet
        │
        ▼
  per-packet RTT store + percentile math
  p50 / p95 / p99 · jitter (RFC 3550) · loss
        │
        ▼
  Python: plots + results writeup
  surfaces the knee across the whole sweep
```

**Highlights:**

- **Open-loop load generator**: sends at a fixed rate regardless of what comes back, so internal queues build and the path's breaking point (*the knee*) actually appears
- Records **every** round-trip time instead of averaging and reports true tail latency (p50 / p95 / p99), jitter (RFC 3550), and loss rate
- **Full rate sweep** (100 to 5000 pps) with generated plots that make the knee visible across load levels
- **C++ hot path for measurement fidelity**: a tight, predictable loop keeps runtime pauses from being logged as network latency
- Caught a real bug. The echo server's per-packet print sat inside the timed round trip, adding ~10 µs per packet (a third of loopback RTT); I removed it and verified the difference across runs
- **CMake build** for the C++ core. 

**Tech Stack:** C++17 • CMake • Python • matplotlib • UDP sockets • POSIX / pthreads

---

### Headiator: Face-Controlled Fighting Game with a Live AI Announcer · [repo](https://github.com/Starrymel/Headiator)

[![Watch the demo](https://img.youtube.com/vi/9tZU3SGazIs/0.jpg)](https://youtu.be/9tZU3SGazIs)

A two-player browser fighting game controlled by facial gestures (MediaPipe) and your **real heart rate & breathing**, read live from the webcam (not wearable). Built at Hack the Hill with a 4-person team · live at [headiator.club](https://headiator.club)

**My role: I owned the live AI announcer & post-match recap systems (end to end).**

**Pipeline:**

```
match event (punch / laser / KO)
      │
      ▼
commentary moment detector  →  Gemini line (Flash-Lite, minimal-latency)
      │                              └─ fallback: pre-generated lines
      ▼
announcer queue  →  ElevenLabs TTS (disk-cached by voice+model+text)
      │                              └─ fallback: browser voice
      ▼
plays in-game, live during the fight
      │
  round ends
      ▼
Gemini recap over the real logged events  →  dashboard (PostgreSQL, offline-safe)
```

**Highlights:**

- Live AI announcer: Gemini commentary proxy behind an announcer queue, with latency-first model selection and fallback lines when the API is slow or unavailable
- ElevenLabs TTS proxy with on-disk caching (keyed by voice + model + text) so repeated lines are instant plus pre-generated lines, API warm-up, and a browser-voice fallback
- Post-match recap: Gemini summarizes the *actual* logged match events into a structured recap, with a template fallback if the key is missing or rate-limited
- Match persistence + history dashboard that keeps working when the database is unreachable
- Fixed per-player heart-rate routing (Presage) in two laptop WebSocket netplay
- Priority-queued announcer with per-type cooldowns, staleness drops, and recent-line context to Gemini, so commentary never repeats or lags the fight
- 15 test files across commentary, AI, persistence, and biometrics

**Tech Stack:** Node.js • Express • Google Gemini API • ElevenLabs API • PostgreSQL • WebSocket (ws) • node:test . *(team stack also: MediaPipe • Canvas 2D • Presage/SmartSpectra • Render)*

---

###  Watchagent Weather Monitor:  Infrastructure Weather Monitoring Platform  · [repo](https://github.com/niamatolla/watchagent-weather-monitor)

A containerized monitoring platform that ingests live weather data, detects operationally relevant weather events, and exposes REST APIs for querying readings and alerts. Implemented Custom Cursor rules, review agent, and database analysis skill tailored to the codebase

**Pipeline:**
```
Open-Meteo API  (free, no key, updates hourly)
        │
        ▼
  weather_client.py
  fetches conditions for one city at a time
        │
        ▼
    poller.py
  runs every 15 min for all three cities
  checks (city, observed_at) before inserting
        │
        ├── duplicate → skip, no event detection
        │
        └── new reading → insert → commit
                │
                ▼
        event_detection/
          detectors.py   pure functions, no DB access
          cooldown.py    suppresses repeat events
          engine.py      runs detectors, checks cooldown, saves WeatherEvent
                │
                ▼
        SQLite  (weather_readings + weather_events)
        persisted in ./data/weather.db
                │
                ▼
        FastAPI
        /health   /readings   /events
```
**Highlights:**
 - Live weather ingestion for Ottawa, Toronto, and Vancouver
 - Signal-based event detection engine
 - Deduplication and cooldown safeguards
 - Docker & Docker Compose deployment
 - Automated unit testing and CI/CD
 - Custom Cursor rules, agent, and skill

**Tech Stack:** Python • FastAPI • SQLAlchemy • SQLite • Pydantic Settings • Docker • Docker Compose • Pytest • GitHub Actions • Cursor
 ElevenLabs API • PostgreSQL • WebSocket (ws) • node:test . *(team stack also: MediaPipe • Canvas 2D • Presage/SmartSpectra • Render)*

---

###  RAG Research Agent (v1) — Retrieval-Augmented Generation System · [repo](https://github.com/niamatolla/rag-research-agent)

[![Watch the demo](https://img.youtube.com/vi/h-ciobBha74/0.jpg)](https://youtu.be/h-ciobBha74)

End-to-end Retrieval-Augmented Generation system built with Azure

**Pipeline:**  
PDF ingestion → chunking → embeddings → vector search → grounded answers

**Highlights:**
- Azure AI Search (HNSW vector search)
- Azure OpenAI embeddings + GPT-4o
- Source-grounded responses
- Modular pipeline (ingestion / retrieval / generation)

**Tech:** Python • Azure OpenAI • Azure AI Search  

---

###  PapaPay: Full-Stack App · [repo](https://github.com/niamatolla/PapaPay)
React + Node.js + MySQL  
- Frontend: https://papa-pay.vercel.app
- Backend API: https://papapay.onrender.com

[![Watch the demo](https://img.youtube.com/vi/ebZYDfeHJ3w/0.jpg)](https://youtu.be/ebZYDfeHJ3w)

A request/approval workflow system with authentication and real-time state updates

**Highlights:**
- REST API with Express
- Authentication & session handling
- Dashboard with dynamic request state
- Full frontend ↔ backend integration

---

## Additional Projects

###  FPGA ALU (VHDL) · [repo](https://github.com/hibaahan/ALU-Project)
16-bit ALU with Booth multiplication, division, add/sub, and flags  
Tested on DE2-115 using structural VHDL  

###  MAX232 Loopback Test · [repo](https://github.com/niamatolla/MAX232-Breadboard-Loopback-Test) 
CMOS ↔ RS-232 conversion validated with oscilloscope + waveform generator  

---

## Tech Stack

**Languages**  
Python • C++ • Java • JavaScript • TypeScript • SQL • VHDL • PowerShell  

**AI / Inference**  
RAG Pipelines • Vector Search • Azure OpenAI • LLM Evaluation • Inference Benchmarking  

**Systems / Performance**  
C++17 • CMake • Multithreading (pthreads) • UDP Sockets • Tail-Latency and Percentile Analysis  

**Backend / Full-Stack**  
FastAPI • Node.js • Express • React • TailwindCSS • Vite  

**Data**  
PostgreSQL • MySQL • SQLite • SQLAlchemy  

**Cloud & DevOps**  
Docker • Docker Compose • Azure • GitHub Actions (CI/CD)  

**Tools**  
Git • GitHub • GitLab • Azure DevOps • Postman • Pytest • Vitest • Biome • Cursor  

**Hardware / Embedded**  
VHDL • Quartus II • Android Studio  

---

## Currently Learning

Model Serving & Batching • Inference Optimization • System Design • Distributed Systems





