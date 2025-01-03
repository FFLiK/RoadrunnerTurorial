---
title: "경로 폭 조율 (Track Width Tuner)"
description: "로드러너를 사용하기 전 로봇에 맞게 시스템을 조율하는 방법을 다룹니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 406
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

rectangle "엔코더 사용" { 
rectangle "<b>주행 속도 PID 조율</b>" as s4_y
}
rectangle "엔코더 미사용" { 
rectangle "<b>주행 피드포워드 조율</b>" as s4_n
}

rectangle "<b>직진 테스트</b>" as s5
rectangle "현재 단계" {
rectangle "<b>경로 폭 조율</b>" as s6
}
rectangle "<b>회전 테스트</b>" as s7


(s4_y) --> (s5)
(s4_n) --> (s5)
(s5) --> (s6)
(s6) --> (s7)

```
현재 단계는 **경로 폭 조율** 단계입니다.

{{<callout context="caution" title="주의" icon="outline/alert-triangle">}}
트랙 너비를 조정하기 전에 로컬라이저의 방향 측정이 정확해야 합니다. 
3-오도메트리 로컬라이저를 사용하는 경우, 측면 거리(lateral distance)를 먼저 조정하세요. 
다른 경우에는 IMU를 통해 헤딩을 직접 얻어야 합니다.
{{</callout>}}

{{< callout context="note" title="노트" icon="outline/info-circle" >}}
"경로 폭"라는 용어는 문맥에 따라 혼란을 줄 수 있으므로, 이를 명확히 설명합니다.

**경로 폭**이란 두 개의 평행한 바퀴의 중심 간 거리입니다. 하지만, 이 용어는 상황에 따라 두 가지로 사용됩니다.

1. `DriveConstants.java`에서의 경로 폭는 **드라이브 트레인의 경로 폭**를 의미합니다. 
2. 3-휠 로컬라이저에서의 트랙 너비는 두 평행한 바퀴의 중심 간 거리로, 이는 `LATERAL_DISTANCE`와 동일합니다.

현재는 로컬라이저를 다루지 않으므로, 이 페이지에서 언급되는 모든 "트랙 너비"는 **드라이브 트레인의 트랙 너비**를 의미합니다.
{{< /callout >}}

{{< figure
src="images/drive-constants/wes-bot-edit-half.jpg"
alt="트랙 너비의 중심에서 중심까지 거리"
caption="2019/20 Skystone Bot (3658 Bosons)"
>}}

---

## 조율 과정 (Tuning Process)

1. **최대 각속도 측정**  
   경로 폭 튜너를 실행하기 전에 로봇이 유지할 수 있는 최대 각속도를 측정해야 합니다. 각속도가 너무 높으면 튜너가 제대로 작동하지 않습니다.
    - `MaxAngularVelocityTuner` OpMode를 실행합니다. 이 튜너는 로봇을 최대 속도로 몇 초 동안 회전시킵니다.
    - 완료 후 "Max Angular Velocity" 값이 출력됩니다. 이 값을 `DriveConstants.java`의 `MAX_ANG_VEL` 필드에 설정하세요.

2. **경로 폭 튜너 실행**
    - `TrackWidthTuner` OpMode를 실행합니다.
    - OpMode를 실행하면 로봇이 180도를 5회 회전할 것입니다. 조정 중 로봇을 건드리면 안 됩니다.
    - 실행 후, RC의 텔레메트리(Telemetry)에 "effective track width"가 출력됩니다.

3. **트랙 너비 값 설정**
    - 출력된 값이 실제 물리적 트랙 너비와 유사하다면(약간의 차이는 괜찮음), 이 값을 `DriveConstants.java`의 `TRACK_WIDTH`에 설정합니다.
    - FTC 대시보드가 열려 있다면, 오른쪽 변수 구성 사이드바에서 트랙 너비를 직접 변경하며 테스트할 수 있습니다. (초기 추정치를 설정한 후 테스트하세요.)
    - 로봇은 한 번 튜닝 할 때 마다 180도에 가까운 회전을 수행해야 합니다.
    - 문제가 발생한다면 수동 조율을 해야합니다. (180도를 돌지 않거나, 경로 폭이 합리적이지 않은 경우)

4. **수동 튜닝**
    - 튜닝 결과가 만족스럽지 않거나 효과적인 트랙 너비 값이 합리적이지 않은 경우, 수동으로 트랙 너비를 조정합니다.
    - 180도로 회전하도록 경로 폭를 조절하면 됩니다. 로봇이 180도보다 적게 회전하면 경로 폭를 늘리고, 180도보다 많이 회전하면 줄입니다.

{{< callout context="note" title="노트" icon="outline/info-circle" >}}
- 튜닝 결과는 완벽할 필요가 없습니다. 약 ±2~3도 이내의 정확도면 충분합니다.
- 현재 단계는 폐루프 피드백이 없는 피드포워드 모션 프로파일을 사용합니다. 이후 팔로워 PID 튜닝 단계에서 헤딩 PID를 활성화하면 원하는 각도로 정확히 회전할 수 있습니다.
{{< /callout >}}

---

## 문제 해결 (Troubleshooting)

1. **튜너 작동 불량**
    - `DriveConstants.java`의 `MAX_ANG_VEL` 값을 줄여보세요.