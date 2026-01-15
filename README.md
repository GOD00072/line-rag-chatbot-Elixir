# Line RAG Chatbot (Elixir + C++ HNSW)

แพลตฟอร์ม Chatbot ประสิทธิภาพสูงที่ผสานการทำงานระหว่าง **LINE Messaging API** เข้ากับระบบ **RAG (Retrieval-Augmented Generation)** โดยมีหัวใจหลักเป็นระบบ Backend ที่เขียนด้วย **Elixir** และฐานข้อมูล Vector ประสิทธิภาพสูงที่พัฒนาขึ้นเองด้วย **C++**

## 🏗 สถาปัตยกรรมระบบ (System Architecture)

![System Architecture](https://img5.pic.in.th/file/secure-sv1/Screenshot-2026-01-14-at-14-30-56-System-Architecture---Line-RAG-Chatbot.png)

ระบบถูกออกแบบในลักษณะ **Modular Monolith** ควบคู่กับ Microservice เฉพาะทาง:

*   **Chat Service (Elixir/Phoenix):** ดูแล Business Logic ทั้งหมด, การจัดการ Webhook จาก LINE, ระบบคิวงาน (Job Queue), และหน้าจอ Admin
*   **Vector Service (C++):** เครื่องมือค้นหา Vector (Search Engine) ความเร็วสูง พัฒนาด้วย C++ 20 รองรับอัลกอริทึม HNSW และชุดคำสั่ง SIMD
*   **Infrastructure:** ใช้ PostgreSQL (เก็บข้อมูล), Redis (Cache/PubSub), และ RabbitMQ (รับรอง Webhook ปริมาณมหาศาล)


### 🚀 ฟีเจอร์เด่น (Key Features)

*   **RAG (Retrieval-Augmented Generation):** ตอบคำถามแม่นยำด้วยการค้นหาข้อมูลจากเอกสารของคุณเอง มาเสริมความฉลาดให้ AI
*   **High Throughput:** รองรับการ Scale รับข้อความมหาศาลด้วย `Broadway` + `RabbitMQ`
*   **Reliable Jobs:** งานเบื้องหลังเชื่อถือได้ด้วย `Oban` (มีระบบ Retry อัตโนมัติเมื่อล้มเหลว)
*   **Custom Vector DB:** ฐานข้อมูล Vector ที่ปรับจูนมาเฉพาะทาง (Optimized C++) ทำงานเร็วกว่า Database ทั่วไป
*   **Real-time Admin:** หน้าจอจัดการระบบที่แสดงผลแบบ Real-time ด้วย **Phoenix LiveView** (ไม่ต้องเขียน React แยก)

---

## 🛠 เทคโนโลยีที่ใช้ (Tech Stack)

### 1. Chat Service (Backend & UI)
*   **ภาษา:** Elixir (รันบน Erlang OTP)
*   **เฟรมเวิร์ก:** Phoenix Framework 1.7
*   **หน้าจอ UI:** Phoenix LiveView (Server-Side Rendering)
*   **ฐานข้อมูล:** PostgreSQL 15 (เชื่อมต่อผ่าน Ecto)
*   **ระบบคิวงาน:** Oban (เก็บสถานะงานลง PostgreSQL)
*   **การรับข้อมูล:** Broadway (ดึงข้อมูลจาก RabbitMQ)
*   **AI:** เชื่อมต่อ OpenAI API และ Google Gemini API

### 2. Vector Service (Search Engine)
*   **ภาษา:** C++ 20
*   **อัลกอริทึม:** HNSW (Hierarchical Navigable Small World)
*   **การเพิ่มประสิทธิภาพ:** SIMD (AVX2 / AVX-512)
*   **โปรโตคอล:** HTTP (REST) และ gRPC
*   **การจัดเก็บ:** บันทึกลง Disk (Persistent) + โหลด Index เข้า RAM

### 3. Infrastructure
*   **Message Broker:** RabbitMQ (สำหรับ Webhook Buffer)
*   **Cache/PubSub:** Redis 7
*   **Containerization:** Docker & Docker Compose

---

## 📦 การติดตั้งและใช้งาน (Installation)

### สิ่งที่ต้องมี (Prerequisites)
*   Docker และ Docker Compose
*   (ถ้าจะรันแบบ Manual) Elixir 1.15+ และ C++ Compiler (GCC/Clang)

### เริ่มต้นด่วน (Quick Start)

1.  **Clone โปรเจค**
    ```bash
    git clone https://github.com/your-username/line-rag-chatbot.git
    cd line-rag-chatbot
    ```

2.  **ตั้งค่า Environment**
    สร้างไฟล์ `.env` (หรือแก้ไขใน `docker-compose.yml`):
    ```env
    DATABASE_URL=ecto://postgres:postgres@postgres/line_chatbot
    ```

3.  **รันระบบ**
    ```bash
    docker-compose up -d --build
    ```

4.  **เข้าใช้งาน**
    *   **Admin Dashboard:** `http://localhost:8888`
    *   **Vector Service Health:** `http://localhost:50052/health`
    *   **RabbitMQ Management:** `http://localhost:15672` (user: guest / pass: guest)

### การติดตั้งแบบ Manual (Chat Service)

1.  **เข้าไปในโฟลเดอร์ Chat Service**
    ```bash
    cd chat_service
    ```

2.  **ติดตั้ง Dependencies**
    ```bash
    mix deps.get
    ```

3.  **ตั้งค่า Database**
    ```bash
    # สร้าง Database
    mix ecto.create

    # รัน Migrations
    mix ecto.migrate

    # (ถ้ามี) ใส่ข้อมูลเริ่มต้น
    mix ecto.seed
    ```

4.  **รัน Server**
    ```bash
    # รันแบบ Development
    mix phx.server
    ```

### การติดตั้งแบบ Manual (Vector Service - C++)

**สิ่งที่ต้องมี:**
- C++ Compiler (GCC 9+ หรือ Clang 10+)
- CMake 3.20+
- Protocol Buffers
- gRPC

**ขั้นตอนการติดตั้ง Dependencies (Ubuntu/Debian):**
```bash
sudo apt-get update && sudo apt-get install -y \
    build-essential \
    cmake \
    git \
    pkg-config \
    libprotobuf-dev \
    protobuf-compiler \
    libgrpc++-dev \
    protobuf-compiler-grpc
```

**การ Build:**

1.  **เข้าไปในโฟลเดอร์ Vector Service**
    ```bash
    cd vector_service
    ```

2.  **สร้างโฟลเดอร์ Build**
    ```bash
    mkdir -p build
    cd build
    ```

3.  **รัน CMake Configuration**
    ```bash
    cmake -DCMAKE_BUILD_TYPE=Release \
          -DUSE_AVX2=ON \
          -DBUILD_TESTS=OFF \
          ..
    ```

4.  **Compile โปรเจค**
    ```bash
    make -j$(nproc)
    ```

5.  **รัน Vector Server**
    ```bash
    ./vector_server
    ```

**ตัวเลือกการ Build:**
- `-DUSE_AVX2=ON` - เปิดใช้งาน AVX2 SIMD (แนะนำ)
- `-DUSE_AVX512=ON` - เปิดใช้งาน AVX-512 (สำหรับ CPU รุ่นใหม่)
- `-DBUILD_TESTS=ON` - Build พร้อม Test Suite

---

## 🔌 ช่องทางการเชื่อมต่อ (API Endpoints)

### Chat Service (Port 8888)
*   `POST /api/webhook/line` - Webhook หลักสำหรับรับข้อความจาก LINE
*   `GET /health` - ตรวจสอบสถานะระบบ

### Vector Service (Port 50052)
*   `POST /insert` - เพิ่มข้อมูล Vector
*   `POST /search` - ค้นหา Vector ที่ใกล้เคียง
*   `GET /stats/:collection` - ดูสถิติของ Collection

---

## 📂 โครงสร้างโปรเจค (Project Structure)

```text
.
├── chat_service/           # โค้ดส่วน Elixir Phoenix Application
│   ├── lib/
│   │   ├── chat_service/   # Business Logic หลัก
│   │   ├── chat_service_web/ # Web Controller และ LiveView
│   │   └── agents/         # ตรรกะการทำงานของ AI Agent
│   └── mix.exs
├── vector_service/         # โค้ดส่วน C++ Vector Database
│   ├── src/                # Source Code (HNSW, HTTP Server)
│   ├── include/            # Header files
│   └── Dockerfile
├── docker-compose.yml      # ไฟล์จัดการ Container
└── ARCHITECTURE_VISUALIZATION.html # แผนภาพระบบแบบจำลอง (Interactive)
```

---

## 🛡 ลิขสิทธิ์ (License)

โปรเจคนี้เผยแพร่ภายใต้ลิขสิทธิ์แบบ **MIT License**# line-rag-chatbot-Elixir
