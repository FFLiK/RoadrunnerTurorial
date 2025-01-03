---
title: "회전 테스트 조율 (Turn Test Tuning)"
description: "로드러너를 사용하기 전 로봇에 맞게 시스템을 조율하는 방법을 다룹니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 407
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  robots: "" # custom robot tags (optional)
---

```kroki {type=PlantUML}

skinparam ranksep 30
skinparam dpi 125
rectangle "<b>주행 속도 PID 조율</b>" as s4_y
rectangle "<b>주행 피드포워드 조율</b>" as s4_n
rectangle "<b>직진 테스트</b>" as s5
rectangle "<b>경로 폭 조율</b>" as s6
rectangle "현재 단계" {
rectangle "<b>회전 테스트</b>" as s7
}
rectangle "<b>로컬라이제이션 테스트</b>" as s8

(s4_y) --> (s5)
(s4_n) --> (s5)
(s5) --> (s6)
(s6) --> (s7)
(s7) --> (s8)
```
현재 단계는 **회전 테스트** 단계입니다.

---

### 회전 테스트 튜닝

회전 테스트는 경로 폭(Track Width)이 크게 잘못되지 않았는지 확인하는 단계입니다.

1. RC를 통해 `TurnTest` OpMode를 실행합니다.
2. 기본 설정에서 `TurnTest`는 로봇을 **90도 회전**시키도록 되어 있습니다.
3. 로봇이 정확히 90도 회전하지 않을 수도 있습니다. 이는 약간의 피드포워드(Feedforward) 차이로 인한 것일 수 있습니다.
4. 만약 정확히 회전하지 않는다면 Android Studio에서 `TurnTest.java` OpMode 파일로 이동하여 `ANGLE` 변수를 **180도**로 변경합니다.
5. OpMode를 다시 실행합니다. 경로 폭을 올바르게 튜닝했다면 로봇이 정확히 **180도 회전**해야 합니다.
6. 그렇지 않다면 경로 폭을 다시 튜닝하세요. 90도 회전 시 발생하는 오류는 걱정하지 마세요. 이 문제는 나중에 헤딩 PID(Heading PID)를 튜닝할 때 수정됩니다.
7. **주의:** `TurnTest`는 기본적으로 **반시계 방향으로 90도 회전**해야 합니다. 만약 기본 설정에서 로봇이 시계 방향으로 회전한다면, `SampleMecanumDrive.java` 파일에서 **왼쪽과 오른쪽 구동 모터의 방향 설정이 뒤바뀌었는지** 확인하세요.