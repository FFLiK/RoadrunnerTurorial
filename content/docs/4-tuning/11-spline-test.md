---
title: "스플라인 테스트 (Spline Test)"
description: "로드러너를 사용하기 전 로봇에 맞게 시스템을 조율하는 방법을 다룹니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 410
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
rectangle "<b>회전 테스트</b>" as s7
rectangle "<b>로컬라이제이션 테스트</b>" as s8
rectangle "<b>팔로워 PID 조율</b>" as s9
rectangle "현재 단계" {
rectangle "<b>스플라인 테스트</b>" as s10
}
rectangle "<b>완료</b>" as s11


(s4_y) --> (s5)
(s4_n) --> (s5)
(s5) --> (s6)
(s6) --> (s7)
(s7) --> (s8)
(s8) --> (s9)
(s9) --> (s10)
(s10) --> (s11)
```
현재 단계는 **스플라인 테스트** 단계입니다.

---

## 스플라인 테스트

1. 병진 이동과 회전 PID가 조정되었으면, Spline Test를 실행하여 모든 것이 제대로 작동하는지 확인하세요.
2. 대시보드를 열어 로봇이 올바르게 경로를 따라가는지 확인하세요. 로봇은 S자 형태의 경로를 따라가야 하며, 대시보드에서도 해당 경로가 올바르게 표시되어야 합니다.
3. 만약 경로를 따르는 동안 약간의 **진동(oscillation)**이 발생한다면, **속도 PID(velocity PID)** 또는 **병진 이동/회전 PID(translational/heading PID)**의 P 값이 너무 높을 가능성이 있습니다. 이 문제를 해결하려면 해당 PID 값을 다시 낮춰보며 조정하세요.
4. 문제가 발생하면 원인을 진단하기 위해 다시 돌아가 문제를 분석하세요. **[FTC Discord](https://discord.gg/first-tech-challenge)**에서 도움을 받을 수도 있습니다.


<small> 축하합니다! 모든 과정이 끝났습니다. 🎉 </small>

---

## 문제 진단 팁

- **대시보드를 열어 경로를 따라가는 동안 확인하세요.**
    - X, Y, Heading 에러를 관찰하며, 에러가 로컬라이저(Localizer) 때문인지 아니면 기본 경로 추적기(Base Path Follower) 때문인지 확인하세요.

- **병진 이동 / 회전 (Translational/Heading) PID를 끄기**
    - 병진 이동 / 회전 PID의 계수를 0으로 설정하세요.
    - 병진 이동 / 회전 PID를 껐을 때 경로 추적이 정상적으로 이루어진다면, 문제는 로컬라이저에 있습니다.
        - `LocalizationTest`를 실행하고 로컬라이저(대부분 `StandardTrackingWheelLocalizer`)를 수정하세요.
    - 병진 이동 / 회전 PID를 껐는데도 문제가 계속된다면, 문제는 구동 모터 방향이나 구동 상수(drive constants)에 있을 가능성이 높습니다.

- **모든 것을 차례로 끄면서 문제를 역추적하세요.**
    - 처음부터 하나씩 각 구성 요소를 확인하며 점진적으로 문제를 해결하면 더 빠르게 원인을 파악할 수 있습니다.

- **포즈 히스토리 제한 해제**
    - 대시보드의 필드에서 파란 선(포즈 히스토리, Pose History)이 사라지는 것을 방지하려면, `SampleMecanumDrive`에서 `POSE_HISTORY_LIMIT` 값을 `-1`로 설정하세요.
    - 이는 디버깅에 유용할 수 있습니다.

- **속도와 가속도 제한 낮추기**
    - `DriveConstants.java`에서 속도(Velocity)와 가속도(Acceleration) 제한을 낮추세요.
    - 실제 로봇이 달성할 수 있는 속도/가속도보다 높은 값을 설정하면, 로봇이 따라갈 수 없는 모션 프로파일이 생성되어 경로 추적이 실패할 수 있습니다.
    - 심각한 문제가 있다면 최대 속도와 가속도를 절반으로 줄이고, 각속도/각가속도 값을 약 **60도/초** 정도로 낮춰보세요.

---

## 튜닝 OpMode 숨기기

Road Runner Quickstart에는 여러 튜닝 OpMode가 포함되어 있습니다.  
사용자 정의 OpMode를 추가하면 RC의 OpMode 목록이 혼잡해질 수 있습니다.  
튜닝 과정이 끝났다면, OpMode 클래스 선언 위에 `@Disabled` 주석을 추가해 숨길 수 있습니다.

```java {hl_lines=[2]}
@Config
@Disabled
@Autonomous(group = "drive")
public class DriveVelocityPIDTuner extends LinearOpMode {
    public static double DISTANCE = 72; // in
```

위와 같은 방식으로 간단히 비활성화하세요.