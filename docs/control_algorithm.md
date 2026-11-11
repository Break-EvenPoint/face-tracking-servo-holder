# 얼굴 위치 기반 제어 알고리즘

처음에는 PID 제어를 검토했다. 실제 시제품에서는 좌표 변화와 네트워크 지연, 기구 유격에 따른 움직임을 고려해 **비례형 각도 보정 + 데드존 + 명령당 이동각 제한**을 구현했다.

## 제어 입력

- `x`, `y`: 얼굴 중심과 카메라 화면 중심 사이의 위치 오차(px)
- `yaw`, `pitch`: 웹에서 계산한 얼굴 방향 관련 정규화 값
- `roll`: 웹에서 계산해 전달하지만 이 펌웨어의 자동 추적 보정에는 사용하지 않음

## 보정량 계산

`firmware/EC2_0512_ver/EC2_0512_ver.ino`에 구현된 관계는 다음과 같다.

```text
panCorrection  = (abs(x) > 25 ? 0.035 * x : 0) + 10.0 * yaw
tiltCorrection = (abs(y) > 20 ? 0.035 * y : 0) +  8.0 * pitch
```

입력이 유효하지 않으면 해당 항을 제외한다. 보정량을 현재 서보 각도에 더한 뒤 범위를 제한하고, 한 번의 명령에서 변하는 각도를 최대 4°로 제한한다.

```text
panTarget  = clamp(panCurrent  + round(panCorrection),  20, 160)
tiltTarget = clamp(tiltCurrent + round(tiltCorrection), 40, 140)

panNext  = moveToward(panCurrent,  panTarget,  4)
tiltNext = moveToward(tiltCurrent, tiltTarget, 4)
```

## 주요 설정값

| 항목 | 설정값 | 설계 의도 |
|---|---:|---|
| X축 위치 오차 데드존 | 25px | 미세한 수평 좌표 변화의 영향 감소 |
| Y축 위치 오차 데드존 | 20px | 미세한 수직 좌표 변화의 영향 감소 |
| 위치 오차 이득 | 0.035°/px | 좌표 오차를 회전 보정으로 변환 |
| yaw / pitch 이득 | 10.0 / 8.0 | 얼굴 방향 관련 값을 보정에 반영 |
| 한 번의 명령당 이동각 | 최대 4° | 급격한 모터 명령 제한 |

데드존은 `x`, `y`의 위치 오차 항에만 적용되므로, 그 범위 안에 들어와도 `yaw`, `pitch` 값에 따라 보정 명령이 발생할 수 있다. 또한 현재 저장된 코드는 적분·미분항을 사용하는 PID 제어가 아니다.

프로토타입에서는 이 방식으로 동작을 구현했다. 이후 개선 방향은 필터링, 전송 주기 조절, 제어 이득 튜닝과 실제 모터 응답의 계측이다.

[시스템 구성](architecture.md) · [실험 결과](results.md) · [펌웨어 코드](../firmware/EC2_0512_ver/EC2_0512_ver.ino)
