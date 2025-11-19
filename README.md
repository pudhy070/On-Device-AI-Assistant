# Project Shittim Chest: 안드로이드 온디바이스 AI 어시스턴트

![Platform](https://img.shields.io/badge/Platform-Android%20Tablet-3DDC84)
![Rooting](https://img.shields.io/badge/Root-Required-red)
![Status](https://img.shields.io/badge/Status-In%20Development-orange)
![License](https://img.shields.io/badge/License-MIT-green)

> 블루 아카이브 "싯딤의 상자"에서 영감을 받아, 삼성 갤럭시 탭을 독립형 AI 어시스턴트 단말기로 변환하는 프로젝트입니다.

---

## 개요

Project Shittim Chest는 상용 안드로이드 태블릿을 완전한 오프라인 AI 기기로 변환하는 임베디드 AI 시스템입니다. 대규모 언어 모델(LLM), 자동 음성 인식(ASR), 음성 합성(TTS)을 클라우드 없이 기기 내에서 완전히 구동합니다.

**핵심 목표:**
- 🔒 **프라이버시 우선:** 모든 처리가 로컬에서 이루어지며 네트워크 전송 없음
- ⚡ **낮은 지연시간:** 하드웨어 직접 접근으로 클라우드 왕복 지연 제거
- 📡 **오프라인 동작:** 인터넷 연결 없이 완전한 기능 제공
- 🎨 **인터랙티브 UI:** 실시간 립싱크 애니메이션을 갖춘 3D 캐릭터 인터페이스

이 프로젝트는 모델 양자화, 시스템 레벨 최적화, 하드웨어 가속을 통해 리소스가 제한된 ARM 모바일 프로세서에 현대적인 AI 스택을 배포하는 과제를 해결합니다.

---

## 주요 기능

- ✅ **완전 오프라인 AI 파이프라인** - Snapdragon 모바일 프로세서에서 구동되는 완전한 AI 스택 (LLM + STT + TTS)
- ✅ **OS 레벨 커스터마이징** - 커스텀 부팅 애니메이션, 키오스크 런처 모드, 시스템 최적화
- ✅ **하이브리드 아키텍처** - Android 프론트엔드와 Linux 컨테이너 백엔드 간 WebSocket IPC
- ✅ **3D 캐릭터 렌더링** - 립싱크를 갖춘 실시간 VRM 모델 애니메이션
- ✅ **하드웨어 통합** - 자이로스코프, 배터리 모니터링, 네이티브 오디오 파이프라인
- ✅ **RAG 컨텍스트 시스템** - 장기 기억 및 컨텍스트 검색을 위한 벡터 데이터베이스

---

## 시스템 아키텍처

시스템은 프레젠테이션, 연산, 하드웨어 액세스를 분리하는 **3계층 아키텍처**를 구현합니다:

```
┌─────────────────────────────────────────────────────────────────┐
│                  레이어 1: 프레젠테이션                          │
│              (Android Native / Frontend)                        │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐   │
│  │   React UI   │  │  3D Renderer │  │  Hardware Access   │   │
│  │  Components  │  │  (Three.js)  │  │  (Sensors/Audio)   │   │
│  └──────────────┘  └──────────────┘  └────────────────────┘   │
│                                                                 │
│                    Capacitor Bridge                             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ WebSocket (localhost:8000)
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   레이어 2: 연산 처리                            │
│              (Termux Linux Container)                           │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐   │
│  │   FastAPI    │  │  llama.cpp   │  │   Sherpa-ONNX      │   │
│  │    Server    │  │    (LLM)     │  │   (STT/TTS)        │   │
│  └──────────────┘  └──────────────┘  └────────────────────┘   │
│                                                                 │
│  ┌──────────────┐  ┌──────────────────────────────────────┐   │
│  │   ChromaDB   │  │      Proot Ubuntu 22.04              │   │
│  │  (Vectors)   │  │      Python 3.10 Runtime             │   │
│  └──────────────┘  └──────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ System Calls
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                레이어 3: 커널 & 하드웨어                          │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐   │
│  │   Magisk     │  │  Snapdragon  │  │   Adreno GPU /     │   │
│  │   Root       │  │     CPU      │  │   Hexagon DSP      │   │
│  └──────────────┘  └──────────────┘  └────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 아키텍처 컴포넌트

| 레이어 | 컴포넌트 | 기술 | 역할 |
|-------|----------|------|------|
| **프레젠테이션** | Frontend UI | React + Capacitor | 사용자 인터페이스, 입력 처리 |
| | 3D 렌더러 | React Three Fiber | VRM 모델 렌더링, 애니메이션 |
| | 오디오 I/O | Android AudioRecord API | 마이크 캡처, 재생 |
| | 센서 | Capacitor Plugins | 자이로스코프, 배터리 모니터링 |
| **연산** | API 서버 | Python FastAPI | 요청 라우팅, 세션 관리 |
| | LLM 엔진 | llama.cpp (GGUF) | 텍스트 생성 추론 |
| | STT 엔진 | Sherpa-ONNX Zipformer | 음성-텍스트 변환 |
| | TTS 엔진 | Piper-TTS (VITS) | 텍스트-음성 합성 |
| | 벡터 DB | ChromaDB | RAG 컨텍스트 검색 |
| | 컨테이너 | Proot-Distro Ubuntu | 격리된 Linux 사용자 공간 |
| **하드웨어** | 루트 액세스 | Magisk | 시스템 레벨 권한 |
| | 프로세서 | Qualcomm Snapdragon | ARM64 연산 |
| | 가속기 | Adreno GPU, Hexagon DSP | 하드웨어 가속 |

---

## 데이터 처리 파이프라인

사용자 음성 입력부터 애니메이션 응답까지의 End-to-End 파이프라인:

```
사용자 음성 입력
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│  단계 1: 음성 활동 감지 (VAD)                               │
│  • Silero VAD로 무음 구간 필터링                            │
│  • AudioRecord API를 통한 16kHz PCM 오디오 캡처             │
└─────────────────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│  단계 2: 자동 음성 인식 (ASR)                               │
│  • Sherpa-ONNX Zipformer (Int8 양자화)                     │
│  • 스트리밍 텍스트 변환 (~200ms 지연시간)                   │
│  • 출력: 변환된 텍스트                                       │
└─────────────────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│  단계 3: 검색 증강 생성 (RAG)                               │
│  • ChromaDB에서 관련 컨텍스트 검색                          │
│  • Llama-3-8B-Instruct (Q4_K_M GGUF)                       │
│  • 출력: 생성된 응답 텍스트                                  │
└─────────────────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│  단계 4: 텍스트 음성 합성 (TTS)                             │
│  • Piper-TTS (VITS 뉴럴 보코더)                            │
│  • 음소 추출 → Viseme 타이밍                                │
│  • 출력: 오디오 파형 (22050Hz) + Viseme 데이터             │
└─────────────────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│  단계 5: 애니메이션 & 오디오 재생                           │
│  • VRM BlendShape 모프를 오디오와 동기화                    │
│  • Web Audio API 타이밍 (<16ms 정렬)                       │
│  • 출력: 동기화된 음성을 가진 애니메이션 캐릭터              │
└─────────────────────────────────────────────────────────────┘
```

### 파이프라인 성능

| 단계 | 모델/기술 | 양자화 | 지연시간 | 하드웨어 |
|------|----------|--------|---------|----------|
| VAD | Silero VAD | - | ~10ms | CPU |
| ASR | Sherpa-ONNX Zipformer | Int8 | ~200ms | CPU |
| LLM | Llama-3-8B-Instruct | Q4_K_M | ~3-5s | CPU/GPU |
| TTS | Piper-TTS (VITS) | - | ~500ms | CPU |
| 렌더링 | Three.js VRM | - | 16ms/프레임 | GPU |

---

## 기술 스택

### 프론트엔드 (Android 애플리케이션)

| 분류 | 기술 | 용도 |
|------|------|------|
| 프레임워크 | React 18 + Vite | UI 프레임워크 및 빌드 도구 |
| 모바일 브리지 | Capacitor 5 | Android 네이티브 API 접근 |
| 3D 렌더링 | React Three Fiber + Three.js | WebGL 기반 3D 렌더링 |
| 캐릭터 모델 | @pixiv/three-vrm | VRM 모델 로딩 및 애니메이션 |
| 상태 관리 | Zustand | 클라이언트 측 상태 관리 |
| 통신 | WebSocket API | 실시간 양방향 통신 |
| 오디오 | Web Audio API | 오디오 재생 및 타이밍 |

### 백엔드 (Termux Linux 컨테이너)

| 분류 | 기술 | 용도 |
|------|------|------|
| 환경 | Ubuntu 22.04 (Proot-Distro) | 격리된 Linux 사용자 공간 |
| 런타임 | Python 3.10+ | 백엔드 런타임 환경 |
| 웹 프레임워크 | FastAPI + Uvicorn | 비동기 API 서버 |
| LLM 엔진 | llama.cpp | GPU 가속을 지원하는 GGUF 모델 추론 |
| ASR 엔진 | Sherpa-ONNX | 스트리밍 음성 인식 |
| TTS 엔진 | Piper-TTS | 뉴럴 텍스트-음성 합성 |
| 벡터 데이터베이스 | ChromaDB | RAG를 위한 임베딩 저장소 |
| 임베딩 | sentence-transformers | 텍스트 임베딩 생성 |

### 시스템 수정

| 분류 | 기술 | 용도 |
|------|------|------|
| 루트 액세스 | Magisk | 시스템 레벨 권한 획득 |
| 부팅 애니메이션 | Custom bootanimation.zip | OEM 로고 교체 |
| 런처 | Custom HOME category | 부팅 시 키오스크 모드 |

---

## 프로젝트 구조

```
OnDeviceAI/
├── android_system_mod/          # 시스템 수정 리소스
│   ├── bootanimation.zip        # 커스텀 부팅 애니메이션
│   ├── magisk_modules/          # 시스템 패치용 Magisk 모듈
│   └── build.prop_tweaks        # 시스템 속성 최적화
│
├── frontend_launcher/           # Android 프론트엔드 애플리케이션
│   ├── src/
│   │   ├── assets/
│   │   │   └── models/          # VRM 캐릭터 모델
│   │   ├── components/          # React UI 컴포넌트
│   │   ├── core/
│   │   │   ├── audio/           # AudioWorklet 프로세서
│   │   │   ├── network/         # WebSocket 클라이언트
│   │   │   └── rendering/       # Three.js 씬 설정
│   │   └── hooks/               # 커스텀 React 훅
│   ├── android/                 # Capacitor Android 네이티브 설정
│   ├── capacitor.config.ts      # Capacitor 설정
│   ├── vite.config.ts           # Vite 빌드 설정
│   └── package.json
│
├── backend_core/                # AI 연산 백엔드
│   ├── app/
│   │   ├── main.py              # FastAPI 애플리케이션 진입점
│   │   ├── routers/             # API 라우트 핸들러
│   │   ├── services/            # 비즈니스 로직 (LLM, STT, TTS)
│   │   └── models/              # 데이터 모델 및 스키마
│   ├── models/                  # AI 모델 바이너리
│   │   ├── llm/                 # GGUF LLM 모델
│   │   ├── stt/                 # ONNX ASR 모델
│   │   └── tts/                 # TTS 음성 모델
│   ├── scripts/
│   │   ├── setup_env.sh         # Termux 환경 설정
│   │   └── boot_service.sh      # 부팅 시 자동 시작 서비스
│   ├── requirements.txt         # Python 의존성
│   └── config.yaml              # 백엔드 설정
│
└── README.md
```

---

## 설치 가이드

> ⚠️ **경고:** 이 프로젝트는 기기 루팅이 필요하며, 제조사 보증이 무효화되고 삼성 Knox가 손상됩니다.

### 사전 요구사항

- Samsung Galaxy Tab S7/S8/S9 (Snapdragon 변종 권장)
- USB 케이블 및 Odin이 설치된 PC (플래싱용)
- Android 루팅 및 Termux에 대한 기본 이해

### Phase 1: 기기 루팅 및 시스템 수정

1. **부트로더 언락**
   ```bash
   # 개발자 옵션에서 OEM 잠금 해제 활성화
   # 다운로드 모드로 부팅 (전원 + 볼륨 다운 + 볼륨 업)
   # 제조사별 언락 절차 따르기
   ```

2. **Magisk 설치**
   ```bash
   # 순정 펌웨어 AP 파일 다운로드
   # Magisk 앱으로 AP 파일 패치
   # Odin을 통해 패치된 AP 플래시
   ```

3. **커스텀 부팅 애니메이션 적용**
   ```bash
   # 루트 권한 필요
   adb push android_system_mod/bootanimation.zip /system/media/
   adb shell chmod 644 /system/media/bootanimation.zip
   ```

### Phase 2: 백엔드 설정 (Termux)

1. **Termux 설치**
   - [F-Droid](https://f-droid.org/packages/com.termux/)에서 다운로드
   - Termux:Boot 애드온 설치

2. **Ubuntu 컨테이너 설정**
   ```bash
   pkg update && pkg upgrade
   pkg install proot-distro
   proot-distro install ubuntu
   proot-distro login ubuntu
   ```

3. **AI 스택 설치**
   ```bash
   # Ubuntu 컨테이너 내부
   apt update && apt install python3.10 python3-pip cmake build-essential

   # 저장소 클론
   git clone <repository-url>
   cd backend_core

   # 의존성 설치
   pip install -r requirements.txt

   # 모델 다운로드 (예시)
   cd models/llm
   wget https://huggingface.co/.../llama-3-8b-instruct-q4_k_m.gguf
   ```

4. **자동 시작 설정**
   ```bash
   # Termux:Boot 설정
   mkdir -p ~/.termux/boot
   cp scripts/boot_service.sh ~/.termux/boot/
   chmod +x ~/.termux/boot/boot_service.sh
   ```

### Phase 3: 프론트엔드 배포

1. **APK 빌드**
   ```bash
   cd frontend_launcher
   npm install
   npm run build
   npx cap sync android
   npx cap build android
   ```

2. **런처 모드 설정**

   `android/app/src/main/AndroidManifest.xml` 수정:
   ```xml
   <intent-filter>
       <action android:name="android.intent.action.MAIN" />
       <category android:name="android.intent.category.LAUNCHER" />
       <category android:name="android.intent.category.HOME" />
       <category android:name="android.intent.category.DEFAULT" />
   </intent-filter>
   ```

3. **설치 및 기본 앱 설정**
   ```bash
   # APK 설치
   adb install app-release.apk

   # 설정 > 앱 > 기본 앱 > 홈 앱으로 이동
   # "Shittim Launcher" 선택
   ```

---

## 로드맵

- [ ] **Phase 1: 핵심 구현**
  - [x] 아키텍처 설계
  - [ ] 기본 WebSocket 통신
  - [ ] LLM 통합 (llama.cpp)
  - [ ] STT/TTS 통합 (Sherpa-ONNX, Piper)

- [ ] **Phase 2: UI/UX 개발**
  - [ ] VRM 캐릭터 렌더링
  - [ ] 립싱크 애니메이션 시스템
  - [ ] 터치 상호작용 핸들러
  - [ ] 키오스크 런처 모드

- [ ] **Phase 3: 시스템 최적화**
  - [ ] 모델 양자화 최적화
  - [ ] 발열 관리
  - [ ] 배터리 최적화
  - [ ] 부팅 서비스 자동화

- [ ] **Phase 4: 고급 기능**
  - [ ] NPU 가속 (Qualcomm SNPE SDK)
  - [ ] 멀티모달 상호작용 (비전)
  - [ ] IoT 허브 통합
  - [ ] 감정 인식

---

## 성능 고려사항

| 측면 | 고려사항 | 솔루션 |
|------|---------|--------|
| **메모리** | LLM 모델은 메모리를 많이 사용 | 4비트 양자화 (Q4_K_M) 사용 |
| **발열** | 지속적인 추론은 발열 유발 | 스로틀링 및 발열 모니터링 구현 |
| **배터리** | 온디바이스 AI는 배터리 소모 | 모델 크기 최적화, 가능 시 NPU 사용 |
| **저장공간** | 모델은 상당한 공간 필요 | 압축된 GGUF 포맷 사용, 선택적 모델 로딩 |
| **지연시간** | 사용자는 빠른 응답 기대 | 스트리밍 추론, 부팅 시 모델 사전 로드 |

---

## 기여

기여는 언제나 환영합니다! 이슈나 풀 리퀘스트를 자유롭게 제출해 주세요.

---

## 면책 조항

- 이 프로젝트는 NEXON Games에서 개발한 **블루 아카이브**에서 영감을 받은 팬 작품입니다.
- 기기 루팅은 보증을 무효화하며 되돌릴 수 없는 손상을 야기할 수 있습니다.
- 이 프로젝트는 연구 목적으로만 제작되었습니다.
- 모든 상표 및 저작권은 해당 소유자에게 있습니다.

---

## 문의

질문이나 협업을 원하시면 GitHub에서 이슈를 열어주세요.


---
---
---


# Project Shittim Chest: Android On-Device AI Assistant

![Platform](https://img.shields.io/badge/Platform-Android%20Tablet-3DDC84)
![Rooting](https://img.shields.io/badge/Root-Required-red)
![Status](https://img.shields.io/badge/Status-In%20Development-orange)
![License](https://img.shields.io/badge/License-MIT-green)

> Inspired by "Shittim Chest" from Blue Archive, this project transforms a Samsung Galaxy Tab into a standalone AI assistant terminal.

---

## Overview

Project Shittim Chest is an embedded AI system that transforms a commercial Android tablet into a fully offline AI device. It runs a Large Language Model (LLM), Automatic Speech Recognition (ASR), and Text-to-Speech (TTS) entirely on-device, without relying on the cloud.

**Core Goals:**
- 🔒 **Privacy-First:** All processing is done locally with no network transmission.
- ⚡ **Low Latency:** Eliminates cloud round-trip delays through direct hardware access.
- 📡 **Offline Operation:** Provides full functionality without an internet connection.
- 🎨 **Interactive UI:** A 3D character interface with real-time lip-sync animation.

This project addresses the challenge of deploying a modern AI stack on resource-constrained ARM mobile processors through model quantization, system-level optimizations, and hardware acceleration.

---

## Key Features

- ✅ **Fully Offline AI Pipeline** - A complete AI stack (LLM + STT + TTS) running on a Snapdragon mobile processor.
- ✅ **OS-Level Customization** - Custom boot animation, kiosk launcher mode, and system optimizations.
- ✅ **Hybrid Architecture** - WebSocket IPC between the Android frontend and a Linux container backend.
- ✅ **3D Character Rendering** - Real-time VRM model animation with lip-sync.
- ✅ **Hardware Integration** - Gyroscope, battery monitoring, and native audio pipeline.
- ✅ **RAG Context System** - Vector database for long-term memory and context retrieval.

---

## System Architecture

The system implements a **3-tier architecture** that separates presentation, computation, and hardware access:

```
┌─────────────────────────────────────────────────────────────────┐
│                  Layer 1: Presentation                          │
│              (Android Native / Frontend)                        │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐   │
│  │   React UI   │  │  3D Renderer │  │  Hardware Access   │   │
│  │  Components  │  │  (Three.js)  │  │  (Sensors/Audio)   │   │
│  └──────────────┘  └──────────────┘  └────────────────────┘   │
│                                                                 │
│                    Capacitor Bridge                             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ WebSocket (localhost:8000)
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   Layer 2: Computation                           │
│              (Termux Linux Container)                           │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐   │
│  │   FastAPI    │  │  llama.cpp   │  │   Sherpa-ONNX      │   │
│  │    Server    │  │    (LLM)     │  │   (STT/TTS)        │   │
│  └──────────────┘  └──────────────┘  └────────────────────┘   │
│                                                                 │
│  ┌──────────────┐  ┌──────────────────────────────────────┐   │
│  │   ChromaDB   │  │      Proot Ubuntu 22.04              │   │
│  │  (Vectors)   │  │      Python 3.10 Runtime             │   │
│  └──────────────┘  └──────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ System Calls
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                Layer 3: Kernel & Hardware                       │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐   │
│  │   Magisk     │  │  Snapdragon  │  │   Adreno GPU /     │   │
│  │   Root       │  │     CPU      │  │   Hexagon DSP      │   │
│  └──────────────┘  └──────────────┘  └────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Architecture Components

| Layer | Component | Technology | Role |
|---|---|---|---|
| **Presentation** | Frontend UI | React + Capacitor | User interface, input handling |
| | 3D Renderer | React Three Fiber | VRM model rendering, animation |
| | Audio I/O | Android AudioRecord API | Microphone capture, playback |
| | Sensors | Capacitor Plugins | Gyroscope, battery monitoring |
| **Computation** | API Server | Python FastAPI | Request routing, session management |
| | LLM Engine | llama.cpp (GGUF) | Text generation inference |
| | STT Engine | Sherpa-ONNX Zipformer | Speech-to-text conversion |
| | TTS Engine | Piper-TTS (VITS) | Text-to-speech synthesis |
| | Vector DB | ChromaDB | RAG context retrieval |
| | Container | Proot-Distro Ubuntu | Isolated Linux user space |
| **Hardware** | Root Access | Magisk | System-level permissions |
| | Processor | Qualcomm Snapdragon | ARM64 computation |
| | Accelerator | Adreno GPU, Hexagon DSP | Hardware acceleration |

---

## Data Processing Pipeline

The end-to-end pipeline from user voice input to animated response:

```
User Voice Input
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│  Step 1: Voice Activity Detection (VAD)                     │
│  • Filter silence with Silero VAD                           │
│  • Capture 16kHz PCM audio via AudioRecord API              │
└─────────────────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│  Step 2: Automatic Speech Recognition (ASR)                 │
│  • Sherpa-ONNX Zipformer (Int8 quantized)                   │
│  • Streaming text conversion (~200ms latency)               │
│  • Output: Transcribed text                                 │
└─────────────────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│  Step 3: Retrieval-Augmented Generation (RAG)               │
│  • Retrieve relevant context from ChromaDB                  │
│  • Llama-3-8B-Instruct (Q4_K_M GGUF)                        │
│  • Output: Generated response text                          │
└─────────────────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│  Step 4: Text-to-Speech (TTS)                               │
│  • Piper-TTS (VITS neural vocoder)                          │
│  • Phoneme extraction → Viseme timing                       │
│  • Output: Audio waveform (22050Hz) + Viseme data           │
└─────────────────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│  Step 5: Animation & Audio Playback                         │
│  • Sync VRM BlendShape morphs with audio                    │
│  • Web Audio API timing (<16ms alignment)                   │
│  • Output: Animated character with synchronized speech      │
└─────────────────────────────────────────────────────────────┘
```

### Pipeline Performance

| Step | Model/Technology | Quantization | Latency | Hardware |
|---|---|---|---|---|
| VAD | Silero VAD | - | ~10ms | CPU |
| ASR | Sherpa-ONNX Zipformer | Int8 | ~200ms | CPU |
| LLM | Llama-3-8B-Instruct | Q4_K_M | ~3-5s | CPU/GPU |
| TTS | Piper-TTS (VITS) | - | ~500ms | CPU |
| Rendering | Three.js VRM | - | 16ms/frame | GPU |

---

## Tech Stack

### Frontend (Android Application)

| Category | Technology | Purpose |
|---|---|---|
| Framework | React 18 + Vite | UI framework and build tool |
| Mobile Bridge | Capacitor 5 | Access to Android native APIs |
| 3D Rendering | React Three Fiber + Three.js | WebGL-based 3D rendering |
| Character Model | @pixiv/three-vrm | VRM model loading and animation |
| State Management | Zustand | Client-side state management |
| Communication | WebSocket API | Real-time bidirectional communication |
| Audio | Web Audio API | Audio playback and timing |

### Backend (Termux Linux Container)

| Category | Technology | Purpose |
|---|---|---|
| Environment | Ubuntu 22.04 (Proot-Distro) | Isolated Linux user space |
| Runtime | Python 3.10+ | Backend runtime environment |
| Web Framework | FastAPI + Uvicorn | Asynchronous API server |
| LLM Engine | llama.cpp | GGUF model inference with GPU acceleration support |
| ASR Engine | Sherpa-ONNX | Streaming speech recognition |
| TTS Engine | Piper-TTS | Neural text-to-speech synthesis |
| Vector Database | ChromaDB | Embedding storage for RAG |
| Embeddings | sentence-transformers | Text embedding generation |

### System Modifications

| Category | Technology | Purpose |
|---|---|---|
| Root Access | Magisk | Gaining system-level permissions |
| Boot Animation | Custom bootanimation.zip | Replacing OEM logo |
| Launcher | Custom HOME category | Kiosk mode on boot |

---

## Project Structure

```
OnDeviceAI/
├── android_system_mod/          # System modification resources
│   ├── bootanimation.zip        # Custom boot animation
│   ├── magisk_modules/          # Magisk modules for system patching
│   └── build.prop_tweaks        # System property optimizations
│
├── frontend_launcher/           # Android frontend application
│   ├── src/
│   │   ├── assets/
│   │   │   └── models/          # VRM character models
│   │   ├── components/          # React UI components
│   │   ├── core/
│   │   │   ├── audio/           # AudioWorklet processors
│   │   │   ├── network/         # WebSocket client
│   │   │   └── rendering/       # Three.js scene setup
│   │   └── hooks/               # Custom React hooks
│   ├── android/                 # Capacitor Android native configuration
│   ├── capacitor.config.ts      # Capacitor configuration
│   ├── vite.config.ts           # Vite build configuration
│   └── package.json
│
├── backend_core/                # AI computation backend
│   ├── app/
│   │   ├── main.py              # FastAPI application entry point
│   │   ├── routers/             # API route handlers
│   │   ├── services/            # Business logic (LLM, STT, TTS)
│   │   └── models/              # Data models and schemas
│   ├── models/                  # AI model binaries
│   │   ├── llm/                 # GGUF LLM models
│   │   ├── stt/                 # ONNX ASR models
│   │   └── tts/                 # TTS voice models
│   ├── scripts/
│   │   ├── setup_env.sh         # Termux environment setup
│   │   └── boot_service.sh      # Auto-start service on boot
│   ├── requirements.txt         # Python dependencies
│   └── config.yaml              # Backend configuration
│
└── README.md
```

---

## Installation Guide

> ⚠️ **Warning:** This project requires rooting the device, which will void the manufacturer's warranty and trip Samsung Knox.

### Prerequisites

- Samsung Galaxy Tab S7/S8/S9 (Snapdragon variant recommended)
- PC with USB cable and Odin installed (for flashing)
- Basic understanding of Android rooting and Termux

### Phase 1: Device Rooting and System Modification

1. **Unlock Bootloader**
   ```bash
   # Enable OEM unlocking in Developer Options
   # Boot into Download Mode (Power + Volume Down + Volume Up)
   # Follow manufacturer-specific unlock procedures
   ```

2. **Install Magisk**
   ```bash
   # Download the stock firmware AP file
   # Patch the AP file with the Magisk app
   # Flash the patched AP via Odin
   ```

3. **Apply Custom Boot Animation**
   ```bash
   # Root access required
   adb push android_system_mod/bootanimation.zip /system/media/
   adb shell chmod 644 /system/media/bootanimation.zip
   ```

### Phase 2: Backend Setup (Termux)

1. **Install Termux**
   - Download from [F-Droid](https://f-droid.org/packages/com.termux/)
   - Install the Termux:Boot add-on

2. **Set up Ubuntu Container**
   ```bash
   pkg update && pkg upgrade
   pkg install proot-distro
   proot-distro install ubuntu
   proot-distro login ubuntu
   ```

3. **Install AI Stack**
   ```bash
   # Inside the Ubuntu container
   apt update && apt install python3.10 python3-pip cmake build-essential

   # Clone the repository
   git clone <repository-url>
   cd backend_core

   # Install dependencies
   pip install -r requirements.txt

   # Download models (example)
   cd models/llm
   wget https://huggingface.co/.../llama-3-8b-instruct-q4_k_m.gguf
   ```

4. **Set up Auto-start**
   ```bash
   # Termux:Boot setup
   mkdir -p ~/.termux/boot
   cp scripts/boot_service.sh ~/.termux/boot/
   chmod +x ~/.termux/boot/boot_service.sh
   ```

### Phase 3: Frontend Deployment

1. **Build APK**
   ```bash
   cd frontend_launcher
   npm install
   npm run build
   npx cap sync android
   npx cap build android
   ```

2. **Set up Launcher Mode**

   Modify `android/app/src/main/AndroidManifest.xml`:
   ```xml
   <intent-filter>
       <action android:name="android.intent.action.MAIN" />
       <category android:name="android.intent.category.LAUNCHER" />
       <category android:name="android.intent.category.HOME" />
       <category android:name="android.intent.category.DEFAULT" />
   </intent-filter>
   ```

3. **Install and Set as Default**
   ```bash
   # Install the APK
   adb install app-release.apk

   # Go to Settings > Apps > Default apps > Home app
   # Select "Shittim Launcher"
   ```

---

## Roadmap

- [ ] **Phase 1: Core Implementation**
  - [x] Architecture design
  - [ ] Basic WebSocket communication
  - [ ] LLM integration (llama.cpp)
  - [ ] STT/TTS integration (Sherpa-ONNX, Piper)

- [ ] **Phase 2: UI/UX Development**
  - [ ] VRM character rendering
  - [ ] Lip-sync animation system
  - [ ] Touch interaction handler
  - [ ] Kiosk launcher mode

- [ ] **Phase 3: System Optimization**
  - [ ] Model quantization optimization
  - [ ] Thermal management
  - [ ] Battery optimization
  - [ ] Boot service automation

- [ ] **Phase 4: Advanced Features**
  - [ ] NPU acceleration (Qualcomm SNPE SDK)
  - [ ] Multimodal interaction (vision)
  - [ ] IoT hub integration
  - [ ] Emotion recognition

---

## Performance Considerations

| Aspect | Consideration | Solution |
|---|---|---|
| **Memory** | LLM models are memory-intensive | Use 4-bit quantization (Q4_K_M) |
| **Thermals** | Continuous inference causes heat | Implement throttling and thermal monitoring |
| **Battery** | On-device AI is power-hungry | Optimize model size, use NPU when possible |
| **Storage** | Models require significant space | Use compressed GGUF format, selective model loading |
| **Latency** | Users expect fast responses | Streaming inference, pre-load models on boot |

---

## Contributing

Contributions are always welcome! Feel free to submit issues or pull requests.

---

## Disclaimer

- This project is a fan work inspired by **Blue Archive**, developed by NEXON Games.
- Rooting your device will void your warranty and may cause irreversible damage.
- This project is intended for research purposes only.
- All trademarks and copyrights are the property of their respective owners.

---

## Contact

For questions or collaboration, please open an issue on GitHub.

---
---
---


# プロジェクト・シッテムチェスト：AndroidオンデバイスAIアシスタント

![Platform](https://img.shields.io/badge/Platform-Android%20Tablet-3DDC84)
![Rooting](https://img.shields.io/badge/Root-Required-red)
![Status](https://img.shields.io/badge/Status-In%20Development-orange)
![License](https://img.shields.io/badge/License-MIT-green)

> ブルーアーカイブの「シッテムの箱」からインスピレーションを受け、Samsung Galaxy TabをスタンドアロンAIアシスタント端末に変えるプロジェクトです。

---

## 概要

Project Shittim Chestは、市販のAndroidタブレットを完全なオフラインAIデバイスに変換する組み込みAIシステムです。大規模言語モデル(LLM)、自動音声認識(ASR)、音声合成(TTS)をクラウドなしでデバイス内で完全に実行します。

**主な目標:**
- 🔒 **プライバシー優先:** すべての処理がローカルで行われ、ネットワーク転送なし
- ⚡ **低遅延:** ハードウェアへの直接アクセスにより、クラウドの往復遅延を排除
- 📡 **オフライン動作:** インターネット接続なしで完全な機能を提供
- 🎨 **インタラクティブUI:** リアルタイムリップシンクアニメーションを備えた3Dキャラクターインターフェース

このプロジェクトは、モデルの量子化、システムレベルの最適化、ハードウェアアクセラレーションを通じて、リソースが制限されたARMモバイルプロセッサに現代的なAIスタックを展開する課題に取り組みます。

---

## 主な機能

- ✅ **完全オフラインAIパイプライン** - Snapdragonモバイルプロセッサで実行される完全なAIスタック (LLM + STT + TTS)
- ✅ **OSレベルのカスタマイズ** - カスタムブートアニメーション、キオスクランチャーモード、システム最適化
- ✅ **ハイブリッドアーキテクチャ** - AndroidフロントエンドとLinuxコンテナバックエンド間のWebSocket IPC
- ✅ **3Dキャラクターレンダリング** - リップシンクを備えたリアルタイムVRMモデルアニメーション
- ✅ **ハードウェア統合** - ジャイロスコープ、バッテリーモニタリング、ネイティブオーディオパイプライン
- ✅ **RAGコンテキストシステム** - 長期記憶とコンテキスト検索のためのベクトルデータベース

---

## システムアーキテクチャ

システムは、プレゼンテーション、演算、ハードウェアアクセスを分離する**3層アーキテクチャ**を実装します:

```
┌─────────────────────────────────────────────────────────────────┐
│                  レイヤー1: プレゼンテーション                       │
│              (Android Native / Frontend)                        │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐   │
│  │   React UI   │  │  3D Renderer │  │  Hardware Access   │   │
│  │  Components  │  │  (Three.js)  │  │  (Sensors/Audio)   │   │
│  └──────────────┘  └──────────────┘  └────────────────────┘   │
│                                                                 │
│                    Capacitor Bridge                             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ WebSocket (localhost:8000)
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   レイヤー2: 演算処理                            │
│              (Termux Linux Container)                           │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐   │
│  │   FastAPI    │  │  llama.cpp   │  │   Sherpa-ONNX      │   │
│  │    Server    │  │    (LLM)     │  │   (STT/TTS)        │   │
│  └──────────────┘  └──────────────┘  └────────────────────┘   │
│                                                                 │
│  ┌──────────────┐  ┌──────────────────────────────────────┐   │
│  │   ChromaDB   │  │      Proot Ubuntu 22.04              │   │
│  │  (Vectors)   │  │      Python 3.10 Runtime             │   │
│  └──────────────┘  └──────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ System Calls
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                レイヤー3: カーネル & ハードウェア                     │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐   │
│  │   Magisk     │  │  Snapdragon  │  │   Adreno GPU /     │   │
│  │   Root       │  │     CPU      │  │   Hexagon DSP      │   │
│  └──────────────┘  └──────────────┘  └────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### アーキテクチャコンポーネント

| レイヤー | コンポーネント | 技術 | 役割 |
|-------|----------|------|------|
| **プレゼンテーション** | Frontend UI | React + Capacitor | ユーザーインターフェース、入力処理 |
| | 3Dレンダラー | React Three Fiber | VRMモデルレンダリング、アニメーション |
| | オーディオI/O | Android AudioRecord API | マイクキャプチャ、再生 |
| | センサー | Capacitor Plugins | ジャイロスコープ、バッテリーモニタリング |
| **演算** | APIサーバー | Python FastAPI | リクエストルーティング、セッション管理 |
| | LLMエンジン | llama.cpp (GGUF) | テキスト生成推論 |
| | STTエンジン | Sherpa-ONNX Zipformer | 音声-テキスト変換 |
| | TTSエンジン | Piper-TTS (VITS) | テキスト-音声合成 |
| | ベクトルDB | ChromaDB | RAGコンテキスト検索 |
| | コンテナ | Proot-Distro Ubuntu | 隔離されたLinuxユーザー空間 |
| **ハードウェア** | ルートアクセス | Magisk | システムレベル権限 |
| | プロセッサ | Qualcomm Snapdragon | ARM64演算 |
| | アクセラレータ | Adreno GPU, Hexagon DSP | ハードウェアアクセラレーション |

---

## データ処理パイプライン

ユーザーの音声入力からアニメーション応答までのEnd-to-Endパイプライン:

```
ユーザーの音声入力
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│  ステップ1: 音声活動検出 (VAD)                               │
│  • Silero VADで無音区間をフィルタリング                     │
│  • AudioRecord APIによる16kHz PCMオーディオキャプチャ       │
└─────────────────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│  ステップ2: 自動音声認識 (ASR)                               │
│  • Sherpa-ONNX Zipformer (Int8量子化)                      │
│  • ストリーミングテキスト変換 (~200ms遅延)                  │
│  • 出力: 変換されたテキスト                                  │
└─────────────────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│  ステップ3: 検索拡張生成 (RAG)                               │
│  • ChromaDBから関連コンテキストを検索                       │
│  • Llama-3-8B-Instruct (Q4_K_M GGUF)                       │
│  • 出力: 生成された応答テキスト                              │
└─────────────────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│  ステップ4: テキスト音声合成 (TTS)                             │
│  • Piper-TTS (VITSニューラルボコーダー)                     │
│  • 音素抽出 → Visemeタイミング                               │
│  • 出力: オーディオ波形 (22050Hz) + Visemeデータ            │
└─────────────────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│  ステップ5: アニメーション & オーディオ再生                       │
│  • VRM BlendShapeモーフをオーディオと同期                   │
│  • Web Audio APIタイミング (<16msアライメント)                │
│  • 出力: 同期された音声を持つアニメーションキャラクター        │
└─────────────────────────────────────────────────────────────┘
```

### パイプライン性能

| ステップ | モデル/技術 | 量子化 | 遅延時間 | ハードウェア |
|------|----------|--------|---------|----------|
| VAD | Silero VAD | - | ~10ms | CPU |
| ASR | Sherpa-ONNX Zipformer | Int8 | ~200ms | CPU |
| LLM | Llama-3-8B-Instruct | Q4_K_M | ~3-5s | CPU/GPU |
| TTS | Piper-TTS (VITS) | - | ~500ms | CPU |
| レンダリング | Three.js VRM | - | 16ms/フレーム | GPU |

---

## 技術スタック

### フロントエンド (Androidアプリケーション)

| 分類 | 技術 | 用途 |
|------|------|------|
| フレームワーク | React 18 + Vite | UIフレームワーク 및 ビルドツール |
| モバイルブリッジ | Capacitor 5 | AndroidネイティブAPIアクセス |
| 3Dレンダリング | React Three Fiber + Three.js | WebGLベースの3Dレンダリング |
| キャラクターモデル | @pixiv/three-vrm | VRMモデルの読み込みとアニメーション |
| 状態管理 | Zustand | クライアントサイドの状態管理 |
| 通信 | WebSocket API | リアルタイム双方向通信 |
| オーディオ | Web Audio API | オーディオ再生とタイミング |

### バックエンド (Termux Linuxコンテナ)

| 分類 | 技術 | 用途 |
|------|------|------|
| 環境 | Ubuntu 22.04 (Proot-Distro) | 隔離されたLinuxユーザー空間 |
| ランタイム | Python 3.10+ | バックエンドランタイム環境 |
| Webフレームワーク | FastAPI + Uvicorn | 非同期APIサーバー |
| LLMエンジン | llama.cpp | GPUアクセラレーションをサポートするGGUFモデル推論 |
| ASRエンジン | Sherpa-ONNX | ストリーミング音声認識 |
| TTSエンジン | Piper-TTS | ニューラルテキスト音声合成 |
| ベクトルデータベース | ChromaDB | RAGのための埋め込みストレージ |
| 埋め込み | sentence-transformers | テキスト埋め込み生成 |

### システム修正

| 分類 | 技術 | 用途 |
|------|------|------|
| ルートアクセス | Magisk | システムレベルの権限取得 |
| ブートアニメーション | Custom bootanimation.zip | OEMロゴの置き換え |
| ランチャー | Custom HOME category | 起動時のキオスクモード |

---

## プロジェクト構造

```
OnDeviceAI/
├── android_system_mod/          # システム修正リソース
│   ├── bootanimation.zip        # カスタムブートアニメーション
│   ├── magisk_modules/          # システムパッチ用Magiskモジュール
│   └── build.prop_tweaks        # システムプロパティ最適化
│
├── frontend_launcher/           # Androidフロントエンドアプリケーション
│   ├── src/
│   │   ├── assets/
│   │   │   └── models/          # VRMキャラクターモデル
│   │   ├── components/          # React UIコンポーネント
│   │   ├── core/
│   │   │   ├── audio/           # AudioWorkletプロセッサ
│   │   │   ├── network/         # WebSocketクライアント
│   │   │   └── rendering/       # Three.jsシーン設定
│   │   └── hooks/               # カスタムReactフック
│   ├── android/                 # Capacitor Androidネイティブ設定
│   ├── capacitor.config.ts      # Capacitor設定
│   ├── vite.config.ts           # Viteビルド設定
│   └── package.json
│
├── backend_core/                # AI演算バックエンド
│   ├── app/
│   │   ├── main.py              # FastAPIアプリケーションエントリーポイント
│   │   ├── routers/             # APIルートハンドラ
│   │   ├── services/            # ビジネスロジック (LLM, STT, TTS)
│   │   └── models/              # データモデルとスキーマ
│   ├── models/                  # AIモデルバイナリ
│   │   ├── llm/                 # GGUF LLMモデル
│   │   ├── stt/                 # ONNX ASRモデル
│   │   └── tts/                 # TTS音声モデル
│   ├── scripts/
│   │   ├── setup_env.sh         # Termux環境設定
│   │   └── boot_service.sh      # 起動時自動開始サービス
│   ├── requirements.txt         # Python依存関係
│   └── config.yaml              # バックエンド設定
│
└── README.md
```

---

## インストールガイド

> ⚠️ **警告:** このプロジェクトはデバイスのroot化が必要であり、メーカー保証が無効になり、Samsung Knoxが破損します。

### 事前要件

- Samsung Galaxy Tab S7/S8/S9 (Snapdragonバリアント推奨)
- USBケーブルとOdinがインストールされたPC (フラッシュ用)
- Androidのroot化とTermuxに関する基本的な理解

### Phase 1: デバイスのroot化とシステム修正

1. **ブートローダーのアンロック**
   ```bash
   # 開発者オプションでOEMロック解除を有効化
   # ダウンロードモードで起動 (電源 + 音量ダウン + 音量アップ)
   # メーカー別のアンロック手順に従う
   ```

2. **Magiskのインストール**
   ```bash
   # 純正ファームウェアのAPファイルをダウンロード
   # MagiskアプリでAPファイルをパッチ
   # Odinを介してパッチされたAPをフラッシュ
   ```

3. **カスタムブートアニメーションの適用**
   ```bash
   # root権限が必要
   adb push android_system_mod/bootanimation.zip /system/media/
   adb shell chmod 644 /system/media/bootanimation.zip
   ```

### Phase 2: バックエンド設定 (Termux)

1. **Termuxのインストール**
   - [F-Droid](https://f-droid.org/packages/com.termux/)からダウンロード
   - Termux:Bootアドオンをインストール

2. **Ubuntuコンテナの設定**
   ```bash
   pkg update && pkg upgrade
   pkg install proot-distro
   proot-distro install ubuntu
   proot-distro login ubuntu
   ```

3. **AIスタックのインストール**
   ```bash
   # Ubuntuコンテナ内部
   apt update && apt install python3.10 python3-pip cmake build-essential

   # リポジトリをクローン
   git clone <repository-url>
   cd backend_core

   # 依存関係のインストール
   pip install -r requirements.txt

   # モデルのダウンロード (例)
   cd models/llm
   wget https://huggingface.co/.../llama-3-8b-instruct-q4_k_m.gguf
   ```

4. **自動開始設定**
   ```bash
   # Termux:Boot設定
   mkdir -p ~/.termux/boot
   cp scripts/boot_service.sh ~/.termux/boot/
   chmod +x ~/.termux/boot/boot_service.sh
   ```

### Phase 3: フロントエンドのデプロイ

1. **APKのビル드**
   ```bash
   cd frontend_launcher
   npm install
   npm run build
   npx cap sync android
   npx cap build android
   ```

2. **ランチャーモードの設定**

   `android/app/src/main/AndroidManifest.xml`を修正:
   ```xml
   <intent-filter>
       <action android:name="android.intent.action.MAIN" />
       <category android:name="android.intent.category.LAUNCHER" />
       <category android:name="android.intent.category.HOME" />
       <category android:name="android.intent.category.DEFAULT" />
   </intent-filter>
   ```

3. **インストールとデフォルトアプリの設定**
   ```bash
   # APKのインストール
   adb install app-release.apk

   # 設定 > アプリ > デフォルトアプリ > ホームアプリに移動
   # "Shittim Launcher"を選択
   ```

---

## ロードマップ

- [ ] **Phase 1: コア実装**
  - [x] アーキテクチャ設計
  - [ ] 基本的なWebSocket通信
  - [ ] LLM統合 (llama.cpp)
  - [ ] STT/TTS統合 (Sherpa-ONNX, Piper)

- [ ] **Phase 2: UI/UX開発**
  - [ ] VRMキャラクターレンダリング
  - [ ] リップシンクアニメーションシステム
  - [ ] タッチインタラクションハンドラ
  - [ ] キオスクランチャーモード

- [ ] **Phase 3: システム最適化**
  - [ ] モデル量子化の最適化
  - [ ] 発熱管理
  - [ ] バッテリー最適化
  - [ ] 起動サービスの自動化

- [ ] **Phase 4: 高度な機能**
  - [ ] NPUアクセラレーション (Qualcomm SNPE SDK)
  - [ ] マルチモーダルインタラクション (ビジョン)
  - [ ] IoTハブ統合
  - [ ] 感情認識

---

## パフォーマンスに関する考慮事項

| 側面 | 考慮事項 | 解決策 |
|------|---------|--------|
| **メモリ** | LLMモデルはメモリを大量に使用 | 4ビット量子化 (Q4_K_M) を使用 |
| **発熱** | 持続的な推論は発熱を引き起こす | スロットリングと発熱モニタリングを実装 |
| **バッテリー** | オンデバイスAIはバッテリーを消耗 | モデルサイズの最適化、可能な場合はNPUを使用 |
| **ストレージ** | モデルはかなりの容量が必要 | 圧縮されたGGUFフォーマットを使用、選択的モデル読み込み |
| **遅延時間** | ユーザーは迅速な応答を期待 | ストリーミング推論、起動時にモデルを事前読み込み |

---

## 貢献

貢献はいつでも歓迎します！イシューやプルリクエストを自由に提出してください。

---

## 免責事項

- このプロジェクトはNEXON Gamesが開発した**ブルーアーカイブ**からインスピレーションを受けたファン作品です。
- デバイスのroot化は保証を無効にし、元に戻せない損傷を引き起こす可能性があります。
- このプロジェクトは研究目的でのみ作成されました。
- すべての商標および著作権は、それぞれの所有者に帰属します。

---

## お問い合わせ

質問や共同作業をご希望の場合は、GitHubでイシューを開いてください。# On-Device-AI-Assistant
# On-Device-AI-Assistant
