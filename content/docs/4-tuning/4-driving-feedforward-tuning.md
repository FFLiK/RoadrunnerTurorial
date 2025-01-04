---
title: "주행 피드포워드 조율 (Driving Feedforward Tuner)"
description: "Road Runner를 사용하기 전 로봇에 맞게 시스템을 조율하는 방법을 다룹니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 404
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

rectangle "<b>데드 휠 설정</b>" as s2
rectangle "<b>로컬라이제이션 테스트</b>" as s3
rectangle "엔코더 사용" { 
rectangle "<b>주행 속도 PID 조율</b>" as s4_y
}
rectangle "현재 단계" {
rectangle "엔코더 미사용" { 
rectangle "<b>주행 피드포워드 조율</b>" as s4_n
}
}
rectangle "<b>직진 테스트</b>" as s5

(s2) --> (s3)
(s3) --> (s4_y)
(s3) --> (s4_n)
(s4_y) --> (s5)
(s4_n) --> (s5)

```
현재 단계는 **주행 피드포워드 조율** 단계입니다.

피드포워드 컨트롤러를 정확히 튜닝하는 것은 경로를 따라 정확히 이동하기 위해 필수적입니다. 
피드포워드 컨트롤러의 튜닝이 부정확하면 이후 과정에서 오류가 발생할 수 있습니다. 
Roadrunner Quickstart에는 자동 튜너와 수동 튜너가 포함되어 있지만, 
많은 사용자가 자동 튜너가 최적의 결과를 제공하지 않는다고 느낍니다. 
특히 자동 튜너는 `kA` 값을 제대로 계산하지 못할 가능성이 큽니다. 
하지만 자동 튜너를 실행해 보고 얻은 수치를 사용하는 것은 개인의 자유이며, 
수동으로 값을 얻고자 할지라도, 자동 튜너로 수치를 얻고, 이후 수동으로 값을 조정하는 것을 추천합니다.

{{<callout context="tip" title="팁" icon="outline/rocket">}}
Xbox 및 Logitech 게임패드에서 **X 버튼**을, PS4 듀얼쇼크에서 **□ 버튼**을 눌러 튜닝 프로세스를 일시 중지하고 운전자 제어 모드로 전환할 수 있습니다.
Xbox 및 Logitech 게임패드에서 **A 버튼**을, PS4 듀얼쇼크에서 **X 버튼**을 눌러 다시 튜너에 제어권을 넘길 수 있습니다.
로봇이 경로를 벗어나면 운전자 제어 모드로 전환하여 로봇을 초기 위치로 되돌리십시오.
{{</callout>}}

---

## 조율 과정 (Tuning)
{{<callout context="caution" title="주의" icon="outline/alert-triangle">}}
튜닝 과정에서 로봇이 천천히 경로에서 벗어나는 것은 정상입니다. 이 문제는 이후에 헤딩 및 병진 PID 튜닝 과정에서 해결될 것입니다.
{{</callout>}}

1. RC(Robot Controller)를 통해 `ManualFeedforwardTuner` OpMode를 실행합니다.

2. RC의 Wi-Fi 네트워크에 연결합니다. 네트워크 비밀번호는 `Program and Manage` 메뉴에서 확인할 수 있습니다.

3. RC로 핸드폰을 사용하는 경우 `192.168.49.1:8080/dash`, Control Hub의 경우 `192.168.43.1:8080/dash`로 이동합니다.

페이지는 아래와 비슷하게 표시될 것입니다:

{{< figure
src="images/feedforward-tuning/example-dashboard-half.jpg"
alt="브라우저에서 FTC 대시보드를 표시하는 이미지"
caption="예시 대시보드 화면<br>(그래프 내용은 무시하세요. 페이지 레이아웃 예시입니다)"
>}}

4. OpMode를 실행합니다. 프로그램을 시작하기 전에는 그래프가 나타나지 않습니다.
    - 프로그램 실행 후 그래프 버튼을 클릭하십시오. 그래프 대신 여러 체크박스가 나타날 경우, `targetVelocity`와 `poseVelocity` 체크박스를 선택하면 됩니다.

5. 대시보드에서 오른쪽에 있는 `DriveConstants` 드롭다운을 찾습니다. 드롭다운을 열고, `kA`, `kV`, `kStatic` 변수를 찾으십시오. 곧 이 변수를 조정하게 될 것입니다.

{{<callout context="caution" title="주의" icon="outline/alert-triangle">}}
이 단계는 중요합니다. **속도 그래프에 안정기가 없는 경우 `kV`가 잘못된 것입니다.** 
이는 이후 정확도 문제로 이어질 수 있습니다. 안정기가 그래프에 나타나도록 거리를 늘리십시오.
{{</callout>}}

6. `ManualFeedforwardTuner` 드롭다운에서 `DISTANCE` 변수가 충분히 커서 `targetVelocity` 라인이 안정기를 가질 수 있도록 설정하십시오. 그래프가 삼각형 모양처럼 보일 경우, `DISTANCE` 값을 늘리십시오.
    {{<callout context="tip" title="팁" icon="outline/rocket">}}
    거리가 충분하지 않은 경우, Drive Constants에서 `MAX_ACCEL` 값을 늘리거나 `MAX_VEL` 값을 줄일 수 있습니다. 단, 이는 궤적의 속도/가속도에 영향을 미칠 수 있으므로 주의하십시오.
    {{</callout>}}

7. OpMode를 실행하면 로봇이 지정된 거리만큼 앞뒤로 이동할 것입니다. 목표는 `poseVelocity` 라인이 `targetVelocity` 라인과 일치하도록 하는 것입니다.

8. **추천 튜닝 과정**:

    1. `kV`를 초기 값으로 `1 / 최대 속도`로 설정하십시오. `poseVelocity` 라인이 `targetVelocity` 안정기 그래프의 상단에 닿아야 합니다. 그렇지 않으면 `kV`를 늘리십시오.
    2. `kA`를 증가시켜 `poseVelocity` 라인의 기울기가 `targetVelocity`와 일치하도록 조정하십시오.
    3. 아래의 참고 자료를 확인하여 각 게인의 역할을 시각적으로 이해하십시오.

       {{< figure
       src="images/feedforward-tuning/dawgma-tuning-guide.jpg"
       alt="피드포워드 튜닝의 다양한 그래프 예시"
       caption="튜닝 팁"
       >}}

       참고 자료는 FRC 팀 1712의 [Adaptive Pure Pursuit 논문](https://www.chiefdelphi.com/t/paper-implementation-of-the-adaptive-pure-pursuit-controller/166552)에서 발췌되었습니다.

    4. 이제 끝입니다! 적절히 튜닝된 피드포워드 컨트롤러의 예시는 아래를 참조하십시오.
    5. **대시보드에서 조정한 값은 반드시 `DriveConstants.java` 파일의 해당 변수에 복사해야 합니다.** 대시보드에서의 조정은 임시적이며, OpMode를 다시 시작하면 초기화됩니다.
    6. 튜닝 시뮬레이터를 사용하여 각 수치가 동작에 미치는 영향을 확인하십시오.
    7. **주의:** 그래프가 완벽할 필요는 없습니다. "충분히 괜찮은" 상태로 두십시오. 그래프를 완벽하게 만드는 데 무한한 시간을 소비할 수 있습니다. 또한, Rev Hub의 내부 모터 컨트롤러 특성상 감속 시 약간의 범프가 발생하며, 이를 제거하는 것은 불가능합니다.

튜닝된 피드포워드 컨트롤러의 예시 (팀 14320의 Deetz 제공):

{{< figure
src="images/feedforward-tuning/deetz-tuning-half.jpg"
alt="튜닝된 피드포워드 컨트롤러"
caption="튜닝된 피드포워드 컨트롤러"
>}}

감속 시의 비대칭성이 보일 수 있습니다. 기본 모터 컨트롤 모델로는 완벽한 속도 제어를 달성할 수 없습니다. 
감속 시 속도가 잘 추적되지 않는 것은 Rev Hub 모터 컨트롤러의 특성 때문일 가능성이 높습니다. 
자세한 내용이나 이 문제에 대한 해결책이 있는 경우 [FTC Discord](https://discord.gg/first-tech-challenge)에서 공유해 주세요.

{{<callout context="tip" title="팁" icon="outline/rocket">}}
REV Hub의 출력 전압은 배터리 레벨이 떨어지면서 감소합니다. 
따라서 피드포워드가 여러 경기에서 일관성을 보장하지 못할 수 있습니다. 
Road Runner에는 명시적인 포즈 속도 폐쇄 루프 제어가 없습니다. 
이를 보완하려면 병진 PID 컨트롤러의 `kD` 항을 추가하세요. 
자세한 내용은 [팔로워 PID 페이지](/follower-pid-tuning)에서 확인할 수 있습니다.
{{</callout>}}

{{<callout context="caution" title="주의" icon="outline/alert-triangle">}}
REV Hub의 모터 컨트롤러는 감속을 제대로 처리하지 못합니다. 
이로 인해 감속 단계에서 피드포워드를 정확히 튜닝하는 것이 불가능합니다. 
이는 매번 10% 정도의 오버슈트를 초래할 수 있으며, 이는 충분히 예상된 결과입니다.
따라서 **이 문제를 무시하고 다음 단계로 진행하십시오.** 
이 문제는 팔로워 PID 조율 과정에서 해결될 것입니다.
{{</callout>}}

---

## 문제 해결 (Troubleshooting)

1. `MaxVelocityTuner`가 역방향으로 움직이는 경우:
    - 바퀴 방향이 올바른지 확인하십시오. [goBILDA 메카넘 차트](/drive-constants.html#samplemecanumdrive-motor-direction)를 참고하세요.

2. `poseVelocity`가 반대로 움직이거나 `targetVelocity`를 따르지 않는 경우:
    - 로컬라이제이션에 문제가 있을 수 있습니다. `LocalizationTest`를 실행해 실제 위치와 일치하는지 확인하십시오.

3. `StraightTest` 또는 `ManualFeedforwardTuning` OpMode가 계속 과도하게 움직이고 `DriveConstants.java` 값을 조정해도 변하지 않는 경우:
    - `DriveConstants.java`에서 `MAX_VEL` 값을 낮춰 보십시오.

4. 기타 모터 방향 문제:
    - [모터 방향 반전](drive-constants.html#samplemecanumdrive-motor-direction)을 참고하십시오.
    - 모터 구성을 디버깅하는 데 어려움을 겪고 있다면 [모터 방향 디버거](https://github.com/acmerobotics/road-runner-quickstart/blob/quickstart1/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/drive/opmode/MotorDirectionDebugger.java)를 참조하세요. 모터 방향 디버거를 사용하면 모터를 하나씩 돌릴 수 있습니다. 41번 줄에서 @Disabled lin을 제거하고 opmode 주석의 지침을 따르십시오. 이를 사용하여 모터 구성 문제를 진단하고 적절하게 수정하세요.

---

## 피드포워드 조율 시뮬레이터
아래 링크로 들어가면 피드포워드 조율(Feedforward Tuning)을 시뮬레이션을 통해 연습해 볼 수 있습니다.
빠른 시일 내에 이 기능을 웹사이트의 기본 기능으로 추가하고자 합니다.

<a class="btn btn-primary btn-cta rounded-pill btn-lg my-3" href="https://learnroadrunner.com/feedforward-tuning.html#feedforward-tuning-simulator" role="button">시뮬레이션 바로가기</a>