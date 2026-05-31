# ZClass — 웹 원격 강의 플랫폼

> 코로나 시기 온라인 강의의 불편한 출석 체크 문제를 해결하기 위해 기획한 웹 기반 화상회의 시스템

## 시연 영상
[![ZClass 시연 영상](https://img.youtube.com/vi/GqfC0SP4rsg/0.jpg)](https://youtu.be/GqfC0SP4rsg)

**인원:** 3인 협업 | **담당:** 머신러닝 제외 전 영역 (Node.js 백엔드, 프론트엔드, WebRTC, Flask 연동)

## 기술 스택
- **백엔드:** Node.js · Express · MongoDB · Socket.io
- **프론트엔드:** HTML · CSS · EJS
- **화상회의:** WebRTC (wrtc)
- **얼굴인식/졸음감지:** Python · Flask · FaceNet · YOLO · dlib (팀원 담당)

## 핵심 구현

| 파일 | 설명 |
|------|------|
| [server.js](src/server.js) | WebRTC SFU 구현, Socket.io 이벤트, Flask 연동 |
| [class.js](src/public/js/3_class/class.js) | 클라이언트 WebRTC 연결, 채팅, 졸음감지 흐름 |
| [routes/class.js](src/routes/class.js) | 수업 입장, 출석 인증 세션 처리 |
| [models/attendances.js](src/models/attendances.js) | 출석 데이터 모델 |

### SFU(Selective Forwarding Unit) 방식 WebRTC

수십 명 규모의 화상 수업을 가정했을 때, P2P 방식은 참가자 N명이 각자 N-1개의 연결을 맺어야 해 연결 수가 폭증합니다. Google Meet를 레퍼런스로 SFU 방식을 채택해 각 클라이언트가 서버에 스트림을 **1회만 송신(sendPC)** 하고, 다른 참가자 스트림은 서버로부터 **각각 수신(receivePC)** 하는 구조로 구현했습니다.

```javascript
// server.js — 신규 참가자 입장 시 처리 흐름
socket.receivePC.ontrack = (_data) => {
    userStreams[socket.id].addTrack(_data.track); // 서버가 스트림 보관

    // 기존 참가자들에게 새 참가자 알림 → 각자 receivePC 생성
    socket.to(socketSession.course_objectId).emit("newUserJoined", socket.id, socketSession.userInfo);

    // 새 참가자에게 기존 참가자 목록 전달 → 순차적으로 receivePC 생성
    sockets[socketSession.course_objectId]
        .filter((_socket) => _socket.id !== socket.id)
        .forEach((_socket) => {
            socket.emit("addOldUser", _socket.id, _socket.request.session.userInfo);
        });
    sockets[socketSession.course_objectId].push(socket);
};
```

### Node.js ↔ Flask 연동

팀원이 구현한 Python ML 모듈(FaceNet 얼굴인식, YOLO 마스크·모자 감지, dlib 졸음감지)을 Flask로 서빙하고, Node.js 서버에서 HTTP 요청으로 연동했습니다. 출석 체크 시 마스크·모자 착용 여부(YOLO) → 얼굴 인식(FaceNet) 순서로 순차 검증합니다.

```javascript
// server.js — 출석 체크 흐름 (Socket 이벤트)
socket.on('checkAttendance', async (_data) => {
    fs.writeFile(`python/data/face_pic/${userId}.png`, _data, ...); // 촬영 이미지 저장

    // 1단계: YOLO로 마스크·모자 착용 여부 검사
    const yoloResult = await axios.get(`http://127.0.0.1:5000/yolo?id=${userId}`);
    if (yoloResult.data === 0) {
        // 2단계: FaceNet으로 본인 여부 확인
        const faceResult = await axios.get(`http://127.0.0.1:5000/face_test?id=${userId}`);
        if (faceResult.data === userId) {
            socketSession.canEnter = true; // 세션에 입장 권한 부여
            socket.emit('checkAttendance', 10); // 인증 성공
        }
    }
});
```

---

## 핵심 코드 구조

```
src/
├── server.js                        # ★ SFU 구현 + Flask 연동 (핵심)
├── models/
│   ├── users.js                     # 사용자 모델
│   ├── courses.js                   # 강의 모델
│   └── attendances.js               # 출석 데이터 모델
├── routes/
│   ├── class.js                     # 수업 입장 + 출석 인증 세션 처리
│   ├── course.js                    # 강의 개설·관리
│   ├── waiting_room.js              # 대기실
│   └── main.js                      # 로그인·회원가입
└── public/js/
    ├── 3_class/
    │   ├── class.js                 # ★ 클라이언트 WebRTC + 졸음감지 흐름 (핵심)
    │   ├── class_init.js
    │   └── class_teacher.js
    └── 2_waiting_room/
        └── attendance.js            # 출석 체크 UI 흐름
```

