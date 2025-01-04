---
title: "직진 테스트 (Straight Test)"
description: "Road Runner를 사용하기 전 로봇에 맞게 시스템을 조율하는 방법을 다룹니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 405
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

rectangle "<b>로컬라이제이션 테스트</b>" as s3
rectangle "엔코더 사용" { 
rectangle "<b>주행 속도 PID 조율</b>" as s4_y
}
rectangle "엔코더 미사용" { 
rectangle "<b>주행 피드포워드 조율</b>" as s4_n
}
rectangle "현재 단계" {
rectangle "<b>직진 테스트</b>" as s5
}
rectangle "<b>경로 폭 조율</b>" as s6

(s3) --> (s4_y)
(s3) --> (s4_n)
(s4_y) --> (s5)
(s4_n) --> (s5)
(s5) --> (s6)

```
현재 단계는 **직진 테스트** 단계입니다.

{{<callout context="caution" title="주의" icon="outline/alert-triangle">}}
구동 인코더 로컬라이제이션(데드 휠 미사용)을 사용하는 경우, 
아래 [구동 인코더 로컬라이제이션](#가로-이동-계수-조정---데드-휠을-사용하는-경우) 단계를 따라야 합니다. 
이 섹션은 구동 인코더 로컬라이제이션 조정에 필수적입니다.  
{{</callout>}}


속도 컨트롤러를 조정한 후, 모든 것이 제대로 작동하는지 확인하기 위해 간단한 직진 테스트를 실행해야 합니다.

---

### 기본 직진 테스트
1. RC를 사용해 `StraightTest` OpMode를 실행합니다.
2. 로봇이 이동한 거리를 측정합니다.
3. 기본 설정에서 로봇은 60인치를 이동해야 합니다.
    - 이 거리는 Dashboard 또는 OpMode 코드에서 변경할 수 있습니다.
        - Dashboard에서 변경하려면, 오른쪽 변수 설정 사이드바에서 `StraightTest` 드롭다운을 찾고 `DISTANCE` 값을 수정합니다.
        - OpMode에서 직접 변경하려면, `StraightTest.java` 파일에서 17번째 줄의 `DISTANCE` 값을 수정합니다.
4. `StraightTest` 작업 모드를 2~3회 실행하여, 로봇이 지정된 거리 내에서 1~3% 정도의 일관성을 유지하는지 확인합니다.
    - 로봇이 완전히 균일하게 움직이지 않을 수 있습니다. 이는 나중에 방향 및 폐쇄 루프 피드백이 활성화되면서 수정됩니다.
    - 오차가 큰 경우 앞선 단계에서의 속도 조절기가 좀 더 정확하게 조율되어야 합니다.
    - 로봇이 약간 치우쳐질 수 있습니다. 이는 후에 PID 조정 단계에서 수정될 것이기에 무시해도 됩니다.

{{< callout context="danger" title="경고" icon="outline/alert-octagon" >}}
**피드포워드 조율을 수행한 경우 주의사항:**  
로봇이 지정된 거리를 약 10~15% 초과할 가능성이 높습니다. 이 문제는 특히 조율이 빠르게 진행 되었을수록 더 심할 수 있습니다.
이는 REV Hub 모터 컨트롤러의 감속 제어 문제로 인해 발생합니다. 
이 단계에서는 이 문제를 무시할 수 있습니다. 
이후 추종 PID 조정 단계에서 `kD`와 `kP` 값을 조정하여 이 문제를 해결할 수 있습니다.
(하지만 PID가 이를 완전히 수정하지 못한다면 `kV`값을 다시 조정해야할 수 있습니다.)
{{< /callout >}}

5. 모든 것이 정상적으로 작동하면 다음 단계로 진행하십시오.

---

### 가로 이동 계수 조정 - 데드 휠을 사용하는 경우

{{<callout context="caution" title="주의" icon="outline/alert-triangle">}}
구동 인코더 로컬라이제이션(데드 휠 미사용)을 사용하는 경우 이 섹션을 건너뛰십시오.
{{</callout>}}

{{<callout context="tip" title="팁" icon="outline/rocket">}}
이는 **필수적인 단계는 아닙니다** (데드 휠을 사용하는 경우). 하지만 메카넘 구동의 운동학적 특성상, 메카넘 구동은 가로 이동(strafing) 시 낮은 토크를 나타냅니다. 
따라서 피드포워드 값에 약간의 보정이 필요할 수 있습니다.
이 단계를 건너뛴 경우라도, 변환 PID가 이러한 불일치를 대부분 보완하므로 문제를 알아차리지 못할 가능성이 높습니다. 
하지만 가로 이동 중 목표 거리보다 짧게 이동(undershooting)하는 현상이 발생하면 **가로 이동 계수(Lateral Multiplier)**를 적용하는 것이 좋습니다.

가로 이동의 비효율성은 [이 문서](https://www.chiefdelphi.com/t/paper-mecanum-and-omni-kinematic-and-force-analysis/106153)를 참고하세요.
{{</callout>}}

1. RC를 사용해 `StrafeTest` OpMode를 실행합니다.
2. OpMode를 실행하면 로봇이 오른쪽으로 지정된 거리만큼 이동합니다. (코드를 수정하여 왼쪽 이동으로 변경 가능)
3. 기본 설정에서 로봇은 60인치를 이동해야 합니다. 그러나 실제 이동 거리는 이보다 짧을 수 있기에, 실제 이동 거리도 측정합니다.
4. 목표 거리(기본값 60인치)를 실제 이동 거리로 나눕니다.
5. 이 값을 `SampleMecanumDrive.java` 파일(57번째 줄)의 `LATERAL_MULTIPLIER` 변수에 설정합니다.
6. `StrafeTest` 작업 모드를 다시 실행하여 수정된 값이 적용되었는지 확인합니다.

---

### 구동 인코더 로컬라이제이션

구동 인코더 로컬라이제이션(데드 휠 미사용)을 사용하는 경우, `StraightTest`와 `StrafeTest` 단계에서 로컬라이제이션을 조정해야 합니다.

### 직진 테스트
- `StraightTest` OpMode를 실행합니다. 실행이 완료되면 x와 y 이동 거리가 출력됩니다.
- 로봇이 한쪽으로 치우칠 수 있습니다. 이는 나중에 PID 조정이 활성화되면서 수정됩니다.
- 실제 이동 거리를 측정하고, 텔레메트리(Telemetry)에 표시된 `finalX` 값과 비교합니다.
- 두 값이 일치하지 않으면, `DriveConstants.java` 파일의 `GEAR_RATIO`를 `실제 거리 / 출력된 x 값`으로 곱하여 수정합니다.
- 이 과정을 여러 번 반복하여 정밀도를 높일 수 있습니다. 가능한 세밀하게 조절하는 것이 좋습니다.

### 가로 이동 테스트
- `StrafeTest` 작업 모드를 실행합니다. 실행이 완료되면 x와 y 이동 거리가 출력됩니다.
- 로봇이 전후로 치우칠 수 있습니다. 이는 나중에 PID 조정이 활성화되면서 수정됩니다.
- 실제 이동 거리를 측정하고, 텔레메트리(Telemetry)에 표시된 `finalY` 값과 비교합니다.
- 두 값이 일치하지 않으면, `SampleMecanumDrive.java` 파일의 `LATERAL_MULTIPLIER`를 `출력된 y 값 / 실제 거리`로 설정합니다.
- 이 과정을 여러 번 반복하여 정밀도를 높일 수 있습니다. 가능한 세밀하게 조절하는 것이 좋습니다.

---

### 문제 해결 (Troubleshooting)

1. 로봇이 직진 테스트에서 후진하거나 회전하는 경우:
모터 방향을 반대로 설정하십시오. [지침](drive-constants.html#samplemecanumdrive-motor-direction)을 참조하세요.

2. 직진 테스트가 일관적이지만 지정된 거리에 도달하지 못하는 경우:
`DriveConstants.java` 파일의 설정을 확인하세요.
- `TICKS_PER_REV`: 인코더의 사양을 확인
- `MAX_RPM`: 모터의 최대 RPM 확인
- `WHEEL_RADIUS`: 바퀴 반지름이 정확한지 확인
- `GEAR_RATIO`: 출력:입력 비율 확인

3. 로봇이 잘못된 방향으로 이동하는 경우:
   [goBILDA 메카넘 차트](/drive-constants.html#samplemecanumdrive-motor-direction)
위와 동일하게 모터 방향을 확인하고 수정하세요. 이 외에도 아래 내용을 참고할 수 있습니다.
   - [모터 방향 반전](drive-constants.html#samplemecanumdrive-motor-direction)을 참고하십시오.
   - 모터 구성을 디버깅하는 데 어려움을 겪고 있다면 [모터 방향 디버거](https://github.com/acmerobotics/road-runner-quickstart/blob/quickstart1/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/drive/opmode/MotorDirectionDebugger.java)를 참조하세요. 모터 방향 디버거를 사용하면 모터를 하나씩 돌릴 수 있습니다. 41번 줄에서 @Disabled lin을 제거하고 opmode 주석의 지침을 따르십시오. 이를 사용하여 모터 구성 문제를 진단하고 적절하게 수정하세요.


4. 10% 초과 문제:
   REV Hub의 모터 컨트롤러는 감속을 제대로 처리하지 못합니다.
   이로 인해 감속 단계에서 피드포워드를 정확히 튜닝하는 것이 불가능합니다.
   이는 매번 10% 정도의 오버슈트를 초래할 수 있으며, 이는 충분히 예상된 결과입니다.
   따라서 **이 문제를 무시하고 다음 단계로 진행하십시오.**
   이 문제는 팔로워 PID 조율 과정에서 해결될 것입니다.
   
{{<callout context="tip" title="팁" icon="outline/rocket">}}
**kV 값을 줄여 오버슈트를 감소시키는 경우:** 
이는 로봇이 지정된 모션 프로파일을 정확히 따르는 능력에 부정적인 영향을 미칠 수 있습니다. 
그러나 경로 전체를 따르는 정확성보다 **최종 지점의 정확도**가 더 중요하다는 점에서 이러한 부정확성을 어느 정도 용인할 수 있습니다. 
우리는 폐루프 피드백(closed-loop feedback)이 경로를 따르는 과정에서 발생할 수 있는 문제를 해결할 것으로 기대하고 있습니다. 
kV 값을 줄여 오버슈트를 보완하면 특히 **코스팅(coasting) 단계**에서 모션 프로파일 정확도가 저하될 수 있습니다. 
그러나 이 문제가 주로 빠른 기어비를 사용하는 시스템에서 나타나며, 이러한 경우 실제로 코스팅 단계에 많은 시간을 소비하지 않는 경우가 많습니다. 
따라서 가속 및 감속 단계가 더 중요하게 여겨지므로, **kV 값을 줄이는 것이 적절한 선택**이 될 수 있습니다.
{{</callout>}}