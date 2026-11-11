# 실행 방법

실험에 사용했던 **웹 → Flask → ESP32 → 서보모터** 연결 구조를 로컬 환경에서 실행하는 방법이다. 코드의 접속 주소와 모터 가동 범위는 사용하는 장치에 맞게 설정해야 한다.

## ESP32

Arduino IDE에 ESP32 보드 패키지와 `ESP32Servo` 라이브러리를 설치한다. `firmware/EC2_0512_ver/EC2_0512_ver.ino`를 연다.

같은 폴더의 `secrets.example.h`를 복사해 `secrets.h`를 만들고 Wi-Fi SSID와 비밀번호를 입력한다.

ESP32가 Wi-Fi에 연결되면 시리얼 모니터에 할당된 IP 주소가 표시된다. 모터는 제어기와 분리된 적정 전원을 사용하고, 두 장치의 GND는 공통으로 연결한다.

## Flask 중계 서버

```bash
python -m pip install -r requirements.txt
```

서버는 기본적으로 포트 `8443`에서 HTTPS를 사용한다. 로컬 실험에서는 인증서와 개인키를 `relay/cert.pem`, `relay/key.pem`에 준비하거나 환경변수 `RELAY_CERT`, `RELAY_KEY`로 경로를 지정한다.

PowerShell 예시:

```powershell
cd relay
$env:ESP32_URL = 'http://192.168.1.55/set'  # ESP32 실제 IP로 변경
python server.py
```

macOS / Linux 예시:

```bash
cd relay
export ESP32_URL='http://192.168.1.55/set'  # ESP32 실제 IP로 변경
python server.py
```

기본 설정: `RELAY_HOST=0.0.0.0`, `RELAY_PORT=8443`. HTTPS 연결에 사용할 인증서는 테스트 환경의 호스트명에 맞게 준비한다.

## 웹 인터페이스

`web/index.html`을 브라우저에서 연다. 카메라 접근이 가능한 보안 컨텍스트(HTTPS 또는 localhost)에서 실행해야 한다. 중계 서버 주소 입력란에는 **호스트:포트**만 넣는다. 웹 코드가 앞에 `https://`를 붙인다.

웹 코드의 MediaPipe 모델과 WASM은 실행 중 온라인 리소스에서 가져온다. 하드웨어·네트워크 환경에 따라 접속 및 인증서 설정이 필요하다.

## 저장된 코드의 범위

현재 소스는 개발 과정의 4서보 ESP32 펌웨어다. 브라우저에서는 `x, y, yaw, pitch, roll`을 전달하고, 자동 각도 보정은 Pan·Tilt에 적용한다. 개별 서보 직접 명령은 펌웨어에서 지원하지만 이 Flask 중계 서버의 `/set` 경로에는 포함되어 있지 않다.

ESP32 HTTP 서버에는 인증 기능이 없으므로 개발용 신뢰 가능한 내부 네트워크에서만 구동한다.
