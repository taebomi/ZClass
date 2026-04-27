# ZClass — 웹 원격 강의 플랫폼

> 코로나 시기 온라인 강의의 불편한 출석 체크 문제를 해결하기 위해 기획한 웹 기반 화상회의 시스템

**인원:** 3인 협업 | **담당:** 머신러닝 제외 전 영역 (Node.js 백엔드, 프론트엔드, WebRTC, Flask 연동)

## 기술 스택
- **백엔드:** Node.js · Express · MongoDB · Socket.io
- **프론트엔드:** HTML · CSS · EJS
- **화상회의:** WebRTC (wrtc)
- **얼굴인식/졸음감지:** Python · Flask · FaceNet · YOLO · dlib (팀원 담당)

## 주요 구현
- 수십 명 규모를 가정, P2P 연결 폭증 문제를 고려해 Google Meet를 레퍼런스로 **SFU(Selective Forwarding Unit) 방식** 채택
- 팀원이 구현한 머신러닝 기반 얼굴인식·졸음감지 Python 모듈을 **Node.js ↔ Flask 간 HTTP 통신**으로 연동
- Socket.io 실시간 채팅 및 세션 기반 수업 입장 관리

## 핵심 코드

| 파일 | 설명 |
|------|------|
| [server.js](src/server.js) | WebRTC SFU 구현, Socket.io 이벤트, Flask 연동 |
| [class.js](src/public/js/3_class/class.js) | 클라이언트 WebRTC 연결, 채팅, 졸음감지 흐름 |
| [routes/class.js](src/routes/class.js) | 수업 입장, 출석 인증 세션 처리 |
| [models/attendances.js](src/models/attendances.js) | 출석 데이터 모델 |

## 시연 영상
[![ZClass 시연 영상](https://img.youtube.com/vi/GqfC0SP4rsg/0.jpg)](https://youtu.be/GqfC0SP4rsg)
