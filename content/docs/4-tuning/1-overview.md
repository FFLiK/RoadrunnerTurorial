---
title: "개요"
description: "로드러너를 사용하기 전 로봇에 맞게 시스템을 조율하는 방법을 다룹니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 401
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  robots: "" # custom robot tags (optional)
---

시작하기 전에, 각 단계에서 무엇을 하는지 이해하는 것이 중요합니다.

{{<callout context="tip" title="팁" icon="outline/rocket">}}
이 페이지는 Road Runner 공식 퀵스타트의 튜닝 가이드를 바탕으로 작성되었으며, 개인 경험을 바탕으로 한 추가적인 팁을 포함하고 있습니다. 보다 자세한 내용은 [공식 가이드](https://acme-robotics.gitbook.io/road-runner/quickstart/tuning)를 참고하세요.  
{{</callout>}}

Road Runner는 복잡한 수학적 계산을 통해 로봇의 동작을 제어하지만, 로봇이 매끄럽게 작동하려면 반드시 튜닝이 필요합니다. 모터, 무게 등 로봇마다 다른 특성이 구동 방식에 영향을 미치기 때문에, 튜닝 가이드를 따라 각 로봇에 맞게 구동 체계를 조정해야 합니다.

{{<callout context="caution" title="주의" icon="outline/alert-triangle">}}
로봇에 무거운 부품을 추가하는 등 큰 변화가 있을 경우, 반드시 재튜닝이 필요합니다. 튜닝 과정은 이전보다 빠르게 진행될 수 있지만, 일관된 성능을 위해 권장됩니다.  
{{</callout>}}

```kroki {type=PlantUML}

skinparam ranksep 30
skinparam dpi 125

rectangle "<b>드라이브 상수 설정</b>" as s1
rectangle "<b>데드 휠 설정</b>" as s2
rectangle "<b>로컬라이제이션 테스트</b>" as s3
rectangle "<b>직진 테스트</b>" as s5
rectangle "<b>경로 폭 조율</b>" as s6
rectangle "<b>회전 테스트</b>" as s7
rectangle "<b>로컬라이제이션 테스트</b>" as s8
rectangle "<b>팔로워 PID 조율</b>" as s9
rectangle "<b>스플라인 테스트</b>" as s10
rectangle "<b>완료</b>" as s11
rectangle "엔코더 사용" { 
rectangle "<b>주행 속도 PID 조율</b>" as s4_y
}
rectangle "엔코더 미사용" { 
rectangle "<b>주행 피드포워드 조율</b>" as s4_n
}

(s1) --> (s2)
(s2) --> (s3)
(s3) --> (s4_y)
(s3) --> (s4_n)
(s4_y) --> (s5)
(s4_n) --> (s5)
(s5) --> (s6)
(s6) --> (s7)
(s7) --> (s8)
(s8) --> (s9)
(s9) --> (s10)
(s10) --> (s11)
```

**각 단계를 완료한 후에 다음 단계로 진행하세요.**

---

### **Feedforward란 무엇인가요?**
Feedforward 속도 제어는 주어진 구동 특성을 기반으로 전압을 속도로 변환하는 함수입니다. 이 과정은 실시간 피드백이 없다는 단점이 있지만, PID 제어를 통해 오차를 보정합니다.

---

### **DriveVelocityPID는 어디에 있나요?**
DriveVelocityPID는 더 이상 사용되지 않으며, Feedforward 방식을 사용해야 합니다.  
Drive encoders나 odometry 여부와 관계없이 Feedforward를 사용하는 것이 권장됩니다.  
자세한 내용은 [여기](/drive-velocity-pid-tuning.html#why-is-drivevelocitypid-not-used)에서 확인하세요.

---

### **Drive Constants**
Drive Constants 파일은 로봇의 물리적 특성(모터 최대 RPM, 휠 반지름 등)을 포함합니다.  
튜닝 과정에서 발생하는 대부분의 큰 오류는 이 단계에서 나타납니다. 예를 들어, 로봇이 지정된 거리의 절반만 이동한다면, 이는 Drive Constants에 문제가 있을 가능성이 큽니다.

자세한 내용은 [Drive Constants 페이지](/drive-constants)에서 확인하세요.

---

### **Dead Wheels**
Dead Wheels가 있는 경우, Drive Constants를 편집한 후 Dead Wheels를 설정해야 합니다. Dead Wheels가 없다면 이 단계를 건너뛰세요.  
Dead Wheels의 개수(2개 또는 3개)에 따라 설정이 다릅니다. 이에 대한 차이는 [FAQ](/#what-is-the-difference-between-two-and-three-wheel-odometry)를 참고하세요.

자세한 내용은 [Dead Wheels 페이지](/dead-wheels)에서 확인하세요.

---

### **Localization Test\***
Dead Wheels를 사용하는 경우, Localization Test를 통해 Dead Wheels의 위치 추적 정확도를 확인하세요. Dead Wheels를 사용하지 않는 경우, 이 단계를 건너뛰고 이후에 Localization Test를 수행하세요.

자세한 내용은 [Dead Wheels 페이지](/dead-wheels)에서 확인하세요.

---

### **DriveFeedforwardTuner**
Feedforward 방식을 사용하는 경우, Feedforward 상수를 튜닝해야 합니다.  
자동 및 수동 튜닝 방식이 제공되며, 개인적으로는 수동 튜닝 방식을 권장합니다.

자세한 내용은 [Feedforward Tuning 페이지](/feedforward-tuning)에서 확인하세요.

---

### **Straight Test**
Straight Test는 Feedforward 및 속도 PID 튜닝의 효과를 확인하는 데 사용됩니다.  
로봇이 몇 번의 테스트에서 일관되게 동일한 거리(1~2인치 오차 범위 내)를 이동하면 튜닝이 성공적이었다고 볼 수 있습니다.

자세한 내용은 [Straight Test 페이지](/straight-test)에서 확인하세요.

---

### **TrackWidthTuner**
Track Width(휠 간 거리)는 실제 측정값과 달라질 수 있습니다. 이를 보정하기 위해 TrackWidthTuner를 실행하여 경험적 Track Width를 계산하세요.

자세한 내용은 [Track Width Tuning 페이지](/trackwidth-tuning)에서 확인하세요.

---

### **Turn Test**
Turn Test를 실행하여 Track Width가 정확한지 확인하세요.

자세한 내용은 [Turn Test Tuning 페이지](/turn-test)에서 확인하세요.

---

### **Localization Test\*\***
두 번째 Localization Test는 Drive Encoder Localization을 테스트하는 데 사용됩니다. Dead Wheels를 사용하여 Localization을 튜닝한 경우, 이 단계를 건너뛰세요.

자세한 내용은 [Localization Test 페이지](/localization-test-2)에서 확인하세요.

---

### **FollowerPIDTuner**
이 단계에서는 헤딩 PID와 이동 PID를 튜닝합니다. 이는 경로를 정확히 따라가기 위해 닫힌 루프 피드백 제어를 가능하게 합니다.

자세한 내용은 [Follower PID Tuning 페이지](/follower-pid-tuning)에서 확인하세요.

---

### **Spline Test**
모든 튜닝이 완료된 후, 로봇이 Spline 경로를 정확히 따라가는지 확인하세요. 문제가 발생하면 해당 단계로 돌아가 재튜닝하세요.

자세한 내용은 [Spline Test 페이지](/spline-test)에서 확인하세요.  